---
title: "MIT 6.824 Lab 1: MapReduce"
date: 2023-07-18 23:10:19
tags:
---

## 要求 // Rules
1. Reduce 任务数量为给定的 $nReduce$，即 mapper 会生产出 $nReduce$ 个中间文件用于后续的 reduce 任务。
2. `mr-out-X` 文件每行包含一个 reduce 函数输出，每行格式为 `%v %v` 分别表示 key 和 value。
3. mapper 输出文件放在当前目录下，以便后续 reducer 作为输入读取。
4. 需要实现 `Done()` 函数用于表明整个 MapReduce 任务完成。
5. 当整个任务都完成时，需要让 worker 进程也应该退出。一种简单的方案是改造 `call()` 函数：当 worker 联系不上 coordinator 时，就可以假定 coordinator 因为任务完成而退出了，因此 worker 也可以退出了。

## 提示 // Hints
1. 一个入手的途径是去修改 `mr/workers.go` 中的 `Worker()` 函数，发送 RPC 到 coordinator 请求任务下发。
2. reduce 任务必须输出至 `mr-out-X`
3. 当修改了 `mr/` 文件夹下的任意内容时，最好 re-build 所有 MapReduce 插件以确认生效。
4. 对于中间文件的命名，一种合理的方案是 `mr-X-Y` ，其中 `X` 表示 map 的任务 id，`Y` 表示 reduce 的任务 id。
5. map 任务需要将生产的 key-value 对存储到中间文件上，一种可行的方案使用 Go 语言的 `encoding/json` 包。
```go
  // 存储
  enc := json.NewEncoder(file)
  for _, kv := ... {
	  err := enc.Encode(&kv)
  }
  // 读取
  dec := json.NewDecoder(file)
  for {
    var kv KeyValue
    if err := dec.Decode(&kv); err != nil {
      break
    }
    kva = append(kva, kv)
  }
```
5. coordinator 作为 RPC 服务器，是并发的，因此记得给数据上锁。
6. 运行 `go run -race` 使用 Go 语言的竞争检测器。
7. Worker 有时是需要等待的，比如 reduce 任务必须等最后一个 map 任务完成后才能开始。
	1. 一种方案是 worker 周期性地向 coordinator 请求任务。
	2. 一种方案是 coordinator 中对应的 RPC handler 使用 `time.Sleep()` 或 `sync.Cond` 进行循环等待。因为 Go 语言中的每个 handler 都有自己的线程，所以 handler 等待状态不会影响 coordinator 处理其他请求。
8. 因为 coordinator 不能判断 worker 是宕机还是处理速度过慢，因此最好给 worker 设置超时时间（10s），当 worker 超时后将任务重新分配给其他 worker 去完成。
9. 若要测试宕机恢复，可以使用 `mrapps/crash.go` 插件，该插件会在 map 和 reduce 函数中随机退出。
10. `test-mr.sh` 处理文件在 `mr-tmp` 文件夹中，若程序有问题可在该目录下查看输出文件。
11. Go RPC 只处理大写字母开头的字段，子数据结构的字段名也必须是大写开头的。
12. 当调用 RPC `call()` 函数时，返回结构体必须包含所有的默认值，否则可能返回错误值。

## 思路 // Idea
- MapReduce 整体分成 2 阶段
	- Map：从原始输入生产 key-value 对
	- Reduce：从 key-value 对生产最终结果
- Worker 采用轮询机制不断向 Coordinator 请求任务下发直至整个任务完成，请求任务的同时向 Coordinator 透传已完成的任务信息。
- Coordinator 维护两个 channel，一个 chan 用于分配任务，一个 chan 用于定时检测 worker 是否超时（判定为宕机）。

## 具体实现 // Implementation

完整代码实现：[MIT6.824/src/mr at master · McGinn7/MIT6.824](https://github.com/McGinn7/MIT6.824/tree/master/src/mr)

### rpc.go
- 结构定义
```go
type Task struct {
    Id       int
    Type     string
    Filepath string
    TTL      time.Time  // 超时时间点
}

type TaskRequest struct {
    WorkerId     int
    LastTaskId   int
    LastTaskType string
}

type TaskResponse struct {
    TaskId   int
    TaskType string
    Filepath string
    NMap     int
    NReduce  int
}
```
- 文件命名规则
```go
func mapTempFilename(mapId int, reduceId int, workerId int) string {
    return fmt.Sprintf("mr-map-%d-%d-%d", mapId, reduceId, workerId)
}

func mapOutputFilename(mapId int, reduceId int) string {
    return fmt.Sprintf("mr-map-%d-%d", mapId, reduceId)
}

func finishMapTask(workerId int, mapId int, nReduce int) {
    for reduceId := 0; reduceId < nReduce; reduceId++ {
        os.Rename(
            mapTempFilename(mapId, reduceId, workerId),
            mapOutputFilename(mapId, reduceId))
    }
}

func reduceTempFilename(reduceId int, workerId int) string {
    return fmt.Sprintf("mr-reduce-%d-%d", reduceId, workerId)
}

func reduceOutputFilename(reduceId int) string {
    return fmt.Sprintf("mr-out-%d", reduceId)
}

func finishReduceTask(workerId int, reduceId int) {
    os.Rename(
        reduceTempFilename(reduceId, workerId),
        reduceOutputFilename(reduceId))
}
```

### worker.go
- 循环请求任务
```go
func Worker(mapf func(string, string) []KeyValue,
    reducef func(string, []string) string) {
    id := os.Getpid()
    lastTaskId := -1
    lastTaskType := ""
    for {
        req := TaskRequest{
            WorkerId:     id,
            LastTaskId:   lastTaskId,
            LastTaskType: lastTaskType,
        }
        rsp := TaskResponse{}
        call("Coordinator.GetTask", &req, &rsp)
        switch rsp.TaskType {
        case "MAP":
            _map(id, rsp.TaskId, rsp.NReduce, rsp.Filepath, mapf)
        case "REDUCE":
            _reduce(id, rsp.TaskId, rsp.NMap, reducef)
        default:
            return
        }
        lastTaskId = rsp.TaskId
        lastTaskType = rsp.TaskType
    }
}
```
- map 任务处理
```go
func _map(workerId int, mapId int, nReduce int, filepath string, mapf func(string, string) []KeyValue) {
	file, _ := os.Open(filepath)
	content, _ := io.ReadAll(file)
	file.Close()

	kva := mapf(filepath, string(content))
	result := make(map[int][]KeyValue)
	for _, kv := range kva {
		reduceId := ihash(kv.Key) % nReduce
		result[reduceId] = append(result[reduceId], kv)
	}
	for reduceId, kvs := range result {
		outFile, _ := os.Create(mapTempFilename(mapId, reduceId, workerId))
		for _, kv := range kvs {
			fmt.Fprintf(outFile, "%v\t%v\n", kv.Key, kv.Value)
		}
		outFile.Close()
	}
}
```
- reduce 任务处理
```go
func _reduce(workerId int, reduceId int, nMap int, reducef func(string, []string) string) {
	var lines []string
	for mapId := 0; mapId < nMap; mapId++ {
		file, err := os.Open(mapOutputFilename(mapId, reduceId))
		if err != nil {
			continue
		}
		content, err := io.ReadAll(file)
		if err != nil {
			continue
		}
		file.Close()
		lines = append(lines, strings.Split(string(content), "\n")...)
	}

	result := make(map[string][]string)
	for _, line := range lines {
		if strings.TrimSpace(line) == "" {
			continue
		}
		split := strings.Split(line, "\t")
		key := split[0]
		value := split[1]
		result[key] = append(result[key], value)
	}

	keys := make([]string, 0)
	for key := range result {
		keys = append(keys, key)
	}
	sort.Strings(keys)
	outFile, _ := os.Create(reduceTempFilename(reduceId, workerId))
	for _, key := range keys {
		fmt.Fprintf(outFile, "%v %v\n", key, reducef(key, result[key]))
	}
	outFile.Close()
}
```

### coordinator.go
- 初始化
```go
type Coordinator struct {
	// Your definitions here.
	nMap             int
	nReduce          int
	stage            string
	taskStatus       map[int]bool
	taskList         chan Task
	reviewList       chan Task
	timeout_duration time.Duration
}
func MakeCoordinator(files []string, nReduce int) *Coordinator {
	// Your code here.
	bufferSize := len(files)
	if bufferSize < nReduce {
		bufferSize = nReduce
	}
	c := Coordinator{
		nMap:             len(files),
		nReduce:          nReduce,
		stage:            "MAP",
		taskStatus:       make(map[int]bool),
		taskList:         make(chan Task, bufferSize),
		reviewList:       make(chan Task, bufferSize),
		timeout_duration: 10 * time.Second,
	}
	for i, file := range files {
		task := Task{
			Id:       i,
			Type:     "MAP",
			Filepath: file,
			TTL:      time.Now().Add(c.timeout_duration),
		}
		c.taskList <- task
	}

	c.server()

	go func() {
		for task := range c.reviewList {
			time.Sleep(time.Until(task.TTL))
			if finished, ok := c.taskStatus[task.Id]; !ok || !finished {
				task.TTL = time.Now().Add(c.timeout_duration)
				c.taskList <- task
			}
		}
	}()
	return &c
}
```
- 任务请求处理
```go
func (c *Coordinator) GetTask(req *TaskRequest, rsp *TaskResponse) error {

	if req.LastTaskType == c.stage {
		switch req.LastTaskType {
		case "MAP":
			finishMapTask(req.WorkerId, req.LastTaskId, c.nReduce)
			c.taskStatus[req.LastTaskId] = true
			if len(c.taskStatus) == c.nMap {
				c.stage = "REDUCE"
				c.taskStatus = make(map[int]bool)
				for i := 0; i < c.nReduce; i++ {
					task := Task{
						Id:       i,
						Type:     "REDUCE",
						Filepath: "",
						TTL:      time.Now().Add(c.timeout_duration),
					}
					c.taskList <- task
				}
			}

		case "REDUCE":
			finishReduceTask(req.WorkerId, req.LastTaskId)
			c.taskStatus[req.LastTaskId] = true
			if len(c.taskStatus) == c.nReduce {
				c.stage = "DONE"
				close(c.taskList)
				close(c.reviewList)
			}
		}
	}

	for {
		task, ok := <-c.taskList
		if !ok {
			return nil
		}
		if finished, ok := c.taskStatus[task.Id]; ok && finished {
			continue
		} else {
			rsp.TaskId = task.Id
			rsp.TaskType = task.Type
			rsp.Filepath = task.Filepath
			rsp.NMap = c.nMap
			rsp.NReduce = c.nReduce

			task.TTL = time.Now().Add(c.timeout_duration)
			c.reviewList <- task
			return nil
		}
	}
}
```

### 其他事项
- 每次完成修改，在 `src/mr` 目录使用 `go build -buildmode=plugin ../mrapps/wc.go` 重新编译，之后运行 `main/test-mr.sh` 进行测试。