---
title: CSAPP - cachelab
date: 2020-02-23 11:32:46
tags:
---



##  题目

### Part A. Cache Simuator

#### 要求

将 `valgrind` 存储跟踪作为输入，在 csim.c 中实现一个缓存模拟器，结果输出：命中，不命中和驱逐的数量（hit, miss, eviction）。需要支持以下参数输入：

- -s：用于组索引的位数。
- -E：每组行数。
- -b：用于块的位数。
- -t：保存 valgrind 跟踪记录的文件。

Valgrind 输出格式为：

```
I 0400d7d4,8
 M 0421c7f0,4
 L 04f6b868,8
 S 7ff0005c8,8
```

每行格式为 `[space]operation address,size`，L 表示加载数据，S 表示存储数据，M 表示修改数据（加载后存储）。指令 M 的存储必然命中，因此其结果只有两种：2 命中或 1 未命中 1 命中。

size 表示内存访问的字节数。测试数据保证内存访问不会出现跨越缓存块的边界，因此 size 在该实验中可忽略。

在该实验中只关注**数据缓存**，因此操作 I（表示指令加载）可忽略。

####  做法

因为只需要输出 (hit, miss, eviction) 三元组，所以实际实现过程中不必要完全复刻 cache 的过程。

对于参数 (s, E, b, t) 输入，使用 `getopt` 指令即可。

构造二维数组 cache\[1 << s\]\[E\] 记录（标记，有效位）。因为组索引的位数 s 必然大于 0，所以有效位和标记位可以压入一个 64 位整数中。

在要驱逐缓存行时，使用 LRU 策略，因此还需要构造二维数组 last_used_time\[1 << s\]\[E\] 记录每个缓存行上次使用时间。

```c
#include "cachelab.h"
#include <stdio.h>
#include <string.h>
#include <getopt.h>
#include <stdlib.h>
#include <unistd.h>
#include <limits.h>

#define BUFFER_SIZE 50
char buf[BUFFER_SIZE];

int s;	  // Number of set index bits
int E;	  // Associativity, number of lines per set
int b;	  // Number of block bits
char *t;	// Name of the valgrind trace to replay

int cache_size;
long long *cache;       // format: tag | valid

unsigned time_stamp;
unsigned *last_used_time;

int hit_count;
int miss_count;
int eviction_count;

void memoryAccess(long long addr) {
	++time_stamp;

	int set_idx = (addr >> b) & ((1 << s) - 1);
	long long tag = addr >> (b + s);

	long long *set_cache = cache + set_idx * E;
	unsigned *set_used_time = last_used_time + set_idx * E;

	int i;
	unsigned LRU_i = -1;
	unsigned LRU_valid;
	unsigned LRU_time;
	for (i = 0; i < E; ++i) {
		unsigned valid = set_cache[i] & 1;
		long long tag_i = set_cache[i] >> 1;
		if (valid > 0 && tag == tag_i) {
			++hit_count;
			set_used_time[i] = time_stamp;
			return ;
		} else {
			if (LRU_i == -1
			  || valid < LRU_valid
			  || (valid == LRU_valid&&
			      set_used_time[i] < LRU_time)) {
				LRU_i = i;
				LRU_valid = valid;
				LRU_time = set_used_time[i];
			}
		}
	}

	++miss_count;
	eviction_count += LRU_valid;
	set_used_time[LRU_i] = time_stamp;
	set_cache[LRU_i] = tag << 1 | 1;
}

int main(int argc, char *argv[]) {
	char *optString = "s:E:b:t:";
	int opt = getopt(argc, argv, optString);
	while (~opt) {
		switch (opt) {
			case 's': s = atoi(optarg); break;
			case 'E': E = atoi(optarg); break;
			case 'b': b = atoi(optarg); break;
			case 't': t = optarg; break;
		}
		opt = getopt(argc, argv, optString);
	}

	time_stamp = 0;
	hit_count = miss_count = eviction_count = 0;
	cache_size = E << s;
	cache = (long long *) malloc(sizeof(*cache) * cache_size);
	last_used_time = (unsigned *) malloc(sizeof(*last_used_time) * cache_size);
	memset(cache, 0, sizeof(*cache) * cache_size);

	FILE *fp = fopen(t, "r");
	while (fgets(buf, BUFFER_SIZE, fp) != NULL) {
		int len = strlen(buf);
		if (len <= 2 || buf[0] != ' ') continue;
		char op = buf[1];
		if (!(op == 'L' || op == 'S' || op == 'M'))
			continue;
		int i;
		for (i = 0; i < len; ++i)
			if (buf[i] == ',') {	    // replace ',' with '\0'
				buf[i] = '\0';
				break;
			}
		buf[1] = '0', buf[2] = 'x';
		long long addr;
		sscanf(buf + 1, "%llx", &addr);
		printf("op = %c, addr = %llx, ", op, addr);
		memoryAccess(addr);
		if (op == 'M') ++hit_count;
	}
	printSummary(hit_count, miss_count, eviction_count);

	fclose(fp);
	return 0;
}
```


### Part B. Optimizing Matrix Transpose

#### 要求

实现函数 `void trans(int M, int N, int A[N][M], int B[M][N])` 转置矩阵 A 并存储到矩阵 B 中，限制：

- 缓存参数为：s = 5, E = 1, b = 5。
- 最多能够定义 12 个 int 类型的局部变量。
- 不允许修改矩阵 A，但能任意修改矩阵 B。

#### 做法

b = 5 表示每个块缓存 $2^5=32$ 字节，即每 8 个 int 元素组索引加一，每个缓存行缓存 8 个地址连续的 int 元素。

s = 5 表示高速缓存中有 $2^5 =32$ 个组，则组索引的循环节大小为 32×8 = 256，即每 256 个 int 元素组索引开始重复。

利用**局部性原理**，尝试 8×8 分块转置：

```c
void transpose_submit(int M, int N, int A[N][M], int B[M][N]) {
	#define min(a,b) ((a)<(b)?(a):(b))
	#define rep(i,l,r) for(i=(l);i<(r);++i)
	#define repd(i,l,r,d) for(i=(l);i<(r);i+=(d))
	#define per(i,l,r) for(i=(r)-1;i>=(l);--i)
	#define perd(i,l,r,d) for(i=(r)-1;i>=(l);i-=(d))

	unsigned x, y, i, j;
	repd(x, 0, N, 8)
		repd(y, 0, M, 8) {
			rep(i, x, min(N, x + 8))
				rep(j, y, min(M, y + 8))
					B[j][i] = A[i][j];
		}
}
```

使用 `make && ./driver.py` 测试：

```
Cache Lab summary:
                        Points   Max pts      Misses
Csim correctness          27.0        27
Trans perf 32x32           6.9         8         343
Trans perf 64x64           0.0         8        4723
Trans perf 61x67           8.8        10        2118
          Total points    42.7        53
```

在 32×32 和 61×67 效果还行，但是在 64×64 上 miss 很多。

使用以下指令查看具体的内存跟踪记录：

```bash
./test-trans -N 64 -M 64
./csim-ref -v -s 5 -E 1 -b 5 -t ./trace.f0
...
L 30df6c,4 miss eviction
S 34dc78,4 miss eviction
L 30df70,4 hit
S 34dd78,4 miss eviction
L 30df74,4 hit
S 34de78,4 miss eviction
L 30df78,4 hit
S 34df78,4 miss eviction
L 30df7c,4 miss eviction
S 34e078,4 miss eviction
L 30e060,4 miss eviction
S 34d97c,4 miss eviction
L 30e064,4 hit
S 34da7c,4 miss eviction
...
```

可以看到，每次在加载（L）后的存储（S）指令就会 miss，并驱逐一条缓存。

当 $SetIndex(A[i][j])=SetIndex(B[i][j])$ 时，8×8 分块中的组索引分布如下：

{% asset_img auad.png %}

前 4 行定义为 $A_u$ 和 $B_u$，后 4 行定义为 $A_d$ 和 $B_d$。

读取 $A[0][j]$ 时，依次存储 $B[j][0]$。注意到，组 0 每次都加载 A 后，均因为 B 也有相同的组索引 0 而被覆盖掉，从而丢失了 A 的空间局部性。

对于这种情况，可以开辟 8 个 int 的局部变量将 $A[0][j]$ 全读入，再更新 $B[j][0]$。因为循环变量已使用了 $x, y, i, j$，因此额外的 8 个 int 是限制上限。

```c
void transpose_submit(int M, int N, int A[N][M], int B[M][N]) {
	#define min(a,b) ((a)<(b)?(a):(b))
	#define rep(i,l,r) for(i=(l);i<(r);++i)
	#define repd(i,l,r,d) for(i=(l);i<(r);i+=(d))
	#define per(i,l,r) for(i=(r)-1;i>=(l);--i)
	#define perd(i,l,r,d) for(i=(r)-1;i>=(l);i-=(d))

	unsigned x, y, i, j;
	int a[8];
	repd(x, 0, N, 8)
		repd(y, 0, M, 8)
			rep(i, x, min(N, x + 8)) {
				rep(j, y, min(M, y + 8)) a[j - y] = A[i][j];
				rep(j, y, min(M, y + 8)) B[j][i] = a[j - y];
			}
}
```

使用 `make && ./driver.py` 测试：

```
Cache Lab summary:
                        Points   Max pts      Misses
Csim correctness          27.0        27
Trans perf 32x32           8.0         8         289
Trans perf 64x64           0.0         8        4613
Trans perf 61x67          10.0        10        1999
          Total points    45.0        53
```

64×64 的 miss 几乎不变，而 32×32 和 61×67 已满分。

目前为止，已经充分利用了 A 的空间局部性，然而对于 B 而言在 64×64 却非如此。

{% asset_img auad.png %}

在读取 $A[i][j]$ 后，先存储 $B_u(0, 8, 16, 24)$，然后存储 $B_d(0, 8, 16, 24)$，即后 4 行的缓存覆盖掉了前 4 行的缓存，从而丢失了 B 的空间局部性。

那么解决方法自然而然是先写 B 的前 4 行再写后 4 行。

```c
void transpose_submit(int M, int N, int A[N][M], int B[M][N]) {
	#define min(a,b) ((a)<(b)?(a):(b))
	#define rep(i,l,r) for(i=(l);i<(r);++i)
	#define repd(i,l,r,d) for(i=(l);i<(r);i+=(d))
	#define per(i,l,r) for(i=(r)-1;i>=(l);--i)
	#define perd(i,l,r,d) for(i=(r)-1;i>=(l);i-=(d))

	unsigned x, y, i, j;
	int a[8];
	repd(x, 0, N, 8)
		repd(y, 0, M, 8) {
			rep(i, x, min(N, x + 8)) {
				rep(j, y, min(M, y + 4)) a[j - y] = A[i][j];
				rep(j, y, min(M, y + 4)) B[j][i] = a[j - y];
			}
			rep(i, x, min(N, x + 8)) {
				rep(j, y + 4, min(M, y + 8)) a[j - y] = A[i][j];
				rep(j, y + 4, min(M, y + 8)) B[j][i] = a[j - y];
			}
		}
}
```

使用 `make && ./driver.py` 测试：

```
Cache Lab summary:
                        Points   Max pts      Misses
Csim correctness          27.0        27
Trans perf 32x32           7.4         8         321
Trans perf 64x64           4.0         8        1653
Trans perf 61x67          10.0        10        1953
          Total points    48.4        53
```

32×32 的 miss 略微提升，但是 64×64 的 miss 已显著下降（4613→1653）。

 进一步的优化需要打印出 64×64 的组索引数值数组（这里本文假设了 $A[i][j]$ 和 $B[i][j]$ 的组索引相同，直觉上是冲突最多的，未经过严格证明）。

```
00 00 00 00 00 00 00 00 01 01 01 01 01 01 01 01 02 ...
08 08 08 08 08 08 08 08 09 09 09 09 09 09 09 09 0a ...
10 10 10 10 10 10 10 10 11 11 11 11 11 11 11 11 12 ...
18 18 18 18 18 18 18 18 19 19 19 19 19 19 19 19 1a ...
00 00 00 00 00 00 00 00 01 01 01 01 01 01 01 01 02 ...
08 08 08 08 08 08 08 08 09 09 09 09 09 09 09 09 0a ...
10 10 10 10 10 10 10 10 11 11 11 11 11 11 11 11 12 ...
18 18 18 18 18 18 18 18 19 19 19 19 19 19 19 19 1a ...
00 00 00 00 00 00 00 00 01 01 01 01 01 01 01 01 02 ...
...
```

观察可以得出以下结论：

1. 当 8×8 块的左上角坐标**不在**对角线上时，$A[i][j]$ 的组索引和 $B[j][i]$ 的组索引没有交集。
2. 当 8×8 块的左上角坐标**在**对角线上时，此时冲突最为严重。

针对观察 1 的结论：在读完 A 的前 4 列后，A 的最后 4 行已被缓存而不会被覆盖。然而，在上一实现中，处理后 4 列时又从前 4 行开始读 A，浪费了已缓存的后 4 行。

所以，这里对于处理 A 的后 4 列时，从后往前读 A 的行，充分利用缓存。过程转变如下图所示：

{% asset_img traversals.png %}

此外，由于每次只读 A 每行的 4 列，实际上用于缓存 A 的局部变量仅需 4 个。但这样的话会浪费上限为 12 的限制。因此，对于额外的 4 个局部变量可缓存 8×8 块**第一行**的后 4 列。这样充分利用了高速缓存的 8 个 int，从而在处理第一行的后 4 列时不用再从内存读取而避免了 miss 。

```c
void transpose_submit(int M, int N, int A[N][M], int B[M][N]) {
	#define min(a,b) ((a)<(b)?(a):(b))
	#define rep(i,l,r) for(i=(l);i<(r);++i)
	#define repd(i,l,r,d) for(i=(l);i<(r);i+=(d))
	#define per(i,l,r) for(i=(r)-1;i>=(l);--i)
	#define perd(i,l,r,d) for(i=(r)-1;i>=(l);i-=(d))

	int x, y, i, j, a[8];
	repd(x, 0, N, 8)
		repd(y, 0, M, 8) {
			rep(j, y, min(M, y + 8)) a[j - y] = A[x][j];
			rep(j, y, min(M, y + 4)) B[j][x] = a[j - y];
			rep(i, x + 1, min(N, x + 8)) {
				rep(j, y, min(M, y + 4)) a[j - y] = A[i][j];
				rep(j, y, min(M, y + 4)) B[j][i] = a[j - y];
			}
			per(i, x + 1, min(N, x + 8)) {
				rep(j, y + 4, min(M, y + 8)) a[j - y - 4] = A[i][j];
				rep(j, y + 4, min(M, y + 8)) B[j][i] = a[j - y - 4];
			}
			rep(j, y + 4, min(M, y + 8)) B[j][x] = a[j - y];
		}
}
```

使用 `make && ./driver.py` 测试：

```
Cache Lab summary:
                        Points   Max pts      Misses
Csim correctness          27.0        27
Trans perf 32x32           7.8         8         309
Trans perf 64x64           7.3         8        1357
Trans perf 61x67          10.0        10        1940
          Total points    52.1        53
```

剩余优化在于处于对角线的 8×8 分块，其冲突主要在于 A 的缓存组索引和 B 的缓存组索引相同。这里只需要针对这种情况构造一种冲突较少的方案即可。

本文构造的方案为：

1. 将 A 的前 4 行依次存储到 B 的前 4 行（**不转置**），此时缓存中均是 $B_u $。转置 B 前 4 行的两个 4×4 子矩阵。由于缓存的存在，转置子矩阵不会导致任何的 miss。
2. 同样处理后 4 行。注意，此时缓存的是 $B_d$。
3. 剩下的就是交换 $B_u$ 的右子矩阵和 $B_d$ 的左子矩阵，利用 8 个局部变量容易做到。

{% asset_img pipeline.png %}

```c
void transpose_submit(int M, int N, int A[N][M], int B[M][N]) {
    #define min(a,b) ((a)<(b)?(a):(b))
    #define rep(i,l,r) for(i=(l);i<(r);++i)
    #define repd(i,l,r,d) for(i=(l);i<(r);i+=(d))
    #define per(i,l,r) for(i=(r)-1;i>=(l);--i)
    #define perd(i,l,r,d) for(i=(r)-1;i>=(l);i-=(d))

    int x, y, i, j, a[8];
    repd(x, 0, N, 8) repd(y, 0, M, 8)
        if (x != y || x + 8 > min(N, M) || y + 8 > min(N, M)) {
            rep(j, y, min(M, y + 8)) a[j - y] = A[x][j];
            rep(j, y, min(M, y + 4)) B[j][x] = a[j - y];

            rep(i, x + 1, min(N, x + 8)) {
                rep(j, y, min(M, y + 4)) a[j - y] = A[i][j];
                rep(j, y, min(M, y + 4)) B[j][i] = a[j - y];
            }
            y += 4;
            per(i, x + 1, min(N, x + 8)) {
                rep(j, y, min(M, y + 4)) a[j - y] = A[i][j];
                rep(j, y, min(M, y + 4)) B[j][i] = a[j - y];
            }
            rep(j, y, min(M, y + 4)) B[j][x] = a[4 + j - y];
            y -= 4;
        } else {
            a[0] = x; x = y; y = a[0];
            // Upper part
            rep(i, 0, 4) {
                rep(j, 0, 8) a[j] = A[y + i][x + j];
                rep(j, 0, 8) B[x + i][y + j] = a[j];
            }
            // trans(0, 0)
            rep(i, 0, 4) rep(j, i + 1, 4) {
                a[0] = B[x + i][y + j];
                B[x + i][y + j] = B[x + j][y + i];
                B[x + j][y + i] = a[0];
            }
            // trans(0, 4)
            y += 4;
            rep(i, 0, 4) rep(j, i + 1, 4) {
                a[0] = B[x + i][y + j];
                B[x + i][y + j] = B[x + j][y + i];
                B[x + j][y + i] = a[0];
            }
            y -= 4;
            // Lower part
            rep(i, 4, 8) {
                rep(j, 0, 8) a[j] = A[y + i][x + j];
                rep(j, 0, 8) B[x + i][y + j] = a[j];
            }
            // trans(4, 0)
            x += 4;
            rep(i, 0, 4) rep(j, i + 1, 4) {
                a[0] = B[x + i][y + j];
                B[x + i][y + j] = B[x + j][y + i];
                B[x + j][y + i] = a[0];
            }
            // trans(4, 4)
            y += 4;
            rep(i, 0, 4) rep(j, i + 1, 4) {
                a[0] = B[x + i][y + j];
                B[x + i][y + j] = B[x + j][y + i];
                B[x + j][y + i] = a[0];
            }
            y -= 4;
            x -= 4;
            // swap Bur, Bdl
            rep(i, 0, 4) {
                rep(j, 0, 4) a[j] = B[x + 4 + i][y + j];
                rep(j, 4, 8) a[j] = B[x + i][y + j];
                rep(j, 4, 8) B[x + i][y + j] = a[j - 4];
                rep(j, 0, 4) B[x + 4 + i][y + j] = a[j + 4];
            }
            a[0] = x; x = y; y = a[0];
        }
}
```

使用 `make && ./driver.py` 测试：

```
Cache Lab summary:
                        Points   Max pts      Misses
Csim correctness          27.0        27
Trans perf 32x32           8.0         8         261
Trans perf 64x64           8.0         8        1261
Trans perf 61x67          10.0        10        1966
          Total points    53.0        53
```



