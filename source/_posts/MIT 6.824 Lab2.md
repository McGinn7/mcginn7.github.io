---
title: "MIT 6.824 Lab 2: Key/Value Server"
date: 2024-05-04 22:03:00
---
## 要求
- 实现单机 key/value 存储服务，一个 server 和若干 clients。
- 支持 3 种函数
	- Put(key, value)：保存和替换 (key, value) 对。
	- Append(key, arg)：将 arg 补充到 key's value 上并返回旧值 value。
	- Get(key)：返回 key's value，若不存在 key 则返回空字符串。
- 注意事项
	- 每个操作需要保持顺序执行。
	- 每个操作可以是线性复杂度，便于观察并发问题。
	- 考虑信息丢失，当 client 不断重试需要保证对应的操作只执行一次。可以假设一个请求不会同时出现。