---
title: CSAPP - archlab
date: 2020-02-21 21:17:36
tags:
---

## 知识点

- 学习流水线化的 Y86-64 处理器的设计和实现，深刻理解**流水线冒险**。
- 学习**循环展开**，理解其对程序性能的影响。

##  题目

### Part A

#### 要求

使用 Y86-64 指令集，编写关于 sum_list, rsum_list 和 copy_block 函数的 Y86-64 程序。

#### 做法

参考书本**图 4-7** 即可，实现也较为简单。

### Part B

#### 要求

使用硬件控制语言（HCL）修改 sim/seq/seq-full.hcl，使得处理器支持 iaddq 指令。

#### 做法

iaddq 指令将立即数和寄存器相加，并将结果写回寄存器。

具体参考 irmovq 指令的实现即可。

### Part C

#### 要求

在 ncopy.ys 中用 Y86-64 指令实现函数 ncopy：复制 src 到 dst，同时统计并返回其中正数的数量。

满分要求达到 CPE < 7.50。

```c
/* 
 * ncopy - copy src to dst, returning number of positive ints
 * contained in src array.
 */
word_t ncopy(word_t *src, word_t *dst, word_t len) {
	word_t count = 0;
    word_t val;
    while (len > 0) {
		val = *src++;
        *dst = val;
        if (val > 0) 
            count++;
        len--;
    }
    return count;    
}
```

#### 做法

首先实现最朴素的循环体：

```assembly
# %rdi = src, %rsi = dst, %rdx = len
ncopy:
	xorq %rax,%rax
	andq %rdx,%rdx				# len > 0?
	jle Done
Loop:
	mrmovq (%rdi), %r10			# get *src
	rmmovq %r10, (%rsi)			# set *dst
	andq %r10, %r10				# val > 0?
	jle Npos
	iaddq $1, %rax				# count++
Npos:
	iaddq $8, %rdi				# src++
	iaddq $8, %rsi				# dst++
	iaddq $-1, %rdx				# len--
	jne Loop
```

该实现使用 `./benchmark.pl` 测试， CPE = 11.70。该实现需要经过优化来达到满分要求。

先简单介绍**循环展开（Loop Unrolling）**，通过增加每次迭代计算的元素数量，从而减少循环的迭代次数：

1. 减少循环不直接有助于计算的操作数量，比如循环索引。

   不严谨地说，就是关键操作（比如乘法操作）数量不变，但是循环变量的加法减少了，这样就可以减少不必要的操作数量。

2. 循环展开可以改变代码，有可能减少流水线冒险。

在朴素版本中，`mrmovq (%rdi), %r10` 和 `rmmovq %r10, (%rsi)` 存在数据相关：需要先将 \*src 读取到寄存器 %r10 中，进而才能读取 %r10 值保存到 \*dst。处理器通过暂停指令来处理该情况，参考下图情形：

{% asset_img  bubble.png 流水线暂停 %}

由于有**转发**机制，流水线处理器可以减少暂停的周期数，而不是图上的 3 个周期。

下面实现 2×1 循环展开：

```assembly
ncopy:
	xorq %rax, %rax
	iaddq $-2, %rdx			# len - 2 >= 0?
	jl Rem
Loop:
	mrmovq (%rdi), %r8		# get *src
	mrmovq 8(%rdi), %r9		# get *(src + 1)
	rmmovq %r8, (%rsi)		# set *dst
	rmmovq %r9, 8(%rsi)		# set *(dst + 1)
	andq %r8, %r8			# val > 0?
	jle LNP8
	iaddq $1, %rax			# count++
LNP8:
	andq %r9, %r9			# val > 0?
	jle LNP9
	iaddq $1, %rax			# count++
LNP9:
	iaddq $16, %rdi			# src += 2
	iaddq $16, %rsi			# dst += 2
	iaddq $-2, %rdx			# len -= 2
	jge Loop

Rem:
	iaddq $2, %rdx
	jle Done

	mrmovq (%rdi), %r8
	rmmovq %r8, (%rsi)
	andq %r8, %r8
	jle Done
	iaddq $1, %rax
```

该实现使用 `./benchmark.pl` 测试， CPE = 8.87。

注意，此处将余数放到最后处理，可以去掉 %rdi, %rsi 和 %rdx 的加法操作，从而提高 CPE。

其中，将 `iaddq $16, %rdi` 置于 `mrmovq 8(%rdi), %r9`  和 `rmmovq %r8, (%rsi)` 之间，CPE 不会提升。此外，插入到 `andq` 和 `jle` 之间也不会提升 CPE。

因此，可得出结论 `mrmovq` 和 `rmmovq` 如果存在数据相关，只会**暂停一个周期**（即插入一个 bubble）。

进一步的优化考虑使用更多的寄存器来展开循环。因为 Y86-64 指令集仅支持 15 个寄存器，去掉已使用的寄存器和栈寄存器，剩余 10 个寄存器可用。所以最多能够编写 10×1 循环展开程序。

参考 2×1 展开，余数部分**手动处理**而不依赖循环结构，可以进一步减少循环索引和循环指针的加法操作。手动处理的话，需要知道余数大小，然后跳转到相应的位置进行处理。这里可以使用**类似二分查找**思想：条件跳转指令有 `jl`, `je` 和 `jg`，可以确定 2 个区间和 1 个值。

知道余数大小后，类似 C 语言中 switch 选择语句依次处理剩余部分：

```c
switch (Remainder) {
    case 9: val=*src++; *dst++=val; count += val>0;
    case 8: val=*src++; *dst++=val; count += val>0;
       ...
    case 1: val=*src++; *dst++=val; count += val>0;
}
```

10×1 循环展开+余数手动处理版本

```assembly
ncopy:
	iaddq $-10, %rdx
	jl Rem

Loop:
	mrmovq (%rdi), %r8
	mrmovq 8(%rdi), %r9
	mrmovq 16(%rdi), %r10
	mrmovq 24(%rdi), %r11
	mrmovq 32(%rdi), %r12
	mrmovq 40(%rdi), %r13
	mrmovq 48(%rdi), %r14
	mrmovq 56(%rdi), %rcx
	mrmovq 64(%rdi), %rbx
	mrmovq 72(%rdi), %rbp
	rmmovq %r8, (%rsi)
	rmmovq %r9, 8(%rsi)
	rmmovq %r10, 16(%rsi)
	rmmovq %r11, 24(%rsi)
	rmmovq %r12, 32(%rsi)
	rmmovq %r13, 40(%rsi)
	rmmovq %r14, 48(%rsi)
	rmmovq %rcx, 56(%rsi)
	rmmovq %rbx, 64(%rsi)
	rmmovq %rbp, 72(%rsi)
	andq %r8, %r8
	jle R10N8
	iaddq $1, %rax
R10N8:
	andq %r9, %r9
	jle R10N9
	iaddq $1, %rax
R10N9:
	andq %r10, %r10
	jle R10N10
	iaddq $1, %rax
R10N10:
	andq %r11, %r11
	jle R10N11
	iaddq $1, %rax
R10N11:
	andq %r12, %r12
	jle R10N12
	iaddq $1, %rax
R10N12:
	andq %r13, %r13
	jle R10N13
	iaddq $1, %rax
R10N13:
	andq %r14, %r14
	jle R10N14
	iaddq $1, %rax
R10N14:
	andq %rcx, %rcx
	jle R10N15
	iaddq $1, %rax
R10N15:
	andq %rbx, %rbx
	jle R10N16
	iaddq $1, %rax
R10N16:
	andq %rbp, %rbp
	jle R10N17
	iaddq $1, %rax
R10N17:
	iaddq $80, %rdi
	iaddq $80, %rsi
	iaddq $-10, %rdx
	jge Loop

Rem:
	iaddq $10, %rdx
	jle Done
	iaddq $-4, %rdx

	jge GE4
	iaddq $2, %rdx
	jl R1
	je R2
	jmp R3

GE4:
	je R4
	iaddq $-2, %rdx
	jl R5
	je R6

	iaddq $-2, %rdx
	jl R7
	je R8

R9:
	mrmovq 64(%rdi), %r8
	rmmovq %r8, 64(%rsi)
	andq %r8, %r8
	jle R8
	iaddq $1, %rax
R8:
	mrmovq 56(%rdi), %r8
	rmmovq %r8, 56(%rsi)
	andq %r8, %r8
	jle R7
	iaddq $1, %rax
R7:
	mrmovq 48(%rdi), %r8
	rmmovq %r8, 48(%rsi)
	andq %r8, %r8
	jle R6
	iaddq $1, %rax
R6:
	mrmovq 40(%rdi), %r8
	rmmovq %r8, 40(%rsi)
	andq %r8, %r8
	jle R5
	iaddq $1, %rax
R5:
	mrmovq 32(%rdi), %r8
	rmmovq %r8, 32(%rsi)
	andq %r8, %r8
	jle R4
	iaddq $1, %rax
R4:
	mrmovq 24(%rdi), %r8
	rmmovq %r8, 24(%rsi)
	andq %r8, %r8
	jle R3
	iaddq $1, %rax
R3:
	mrmovq 16(%rdi), %r8
	rmmovq %r8, 16(%rsi)
	andq %r8, %r8
	jle R2
	iaddq $1, %rax
R2:
	mrmovq 8(%rdi), %r8
	rmmovq %r8, 8(%rsi)
	andq %r8, %r8
	jle R1
	iaddq $1, %rax
R1:
	mrmovq (%rdi), %r8
	rmmovq %r8, (%rsi)
	andq %r8, %r8
	jle Done
	iaddq $1, %rax
```

该实现使用 `./benchmark.pl` 测试， CPE = 8.02。

继续观察上述代码，还有 2 处可以优化：

1. 每个 case 下，`mrmovq` 和 `rmmovq` 存在数据相关。
2. 余数为 0 时，单独特判。假设每个余数等概率出现，那么很大概率这个条件跳转不会发生，从而增加了 CPE。所以，余数二分查找时要把 0 考虑进去。

针对 1 的优化，从 `mrmovq` 和 `rmmovq` 不会设置**条件码**入手，将这两条指令插入到 `andq %r8, %r8` 和 `jle` 之间，从而避免流水线暂停（这两条指令相邻时暂停一周期）。解决方法是：将前一个数的正负判断延迟到当前模块处理。具体实现为：

```assembly
Rn:
	andq %r8, %r8				# %r8=src[n - 1]
	mrmovq 8n(%rdi), %r8		# 加载src[n]到%r8
	jle EnNP
	iaddq $1, %rax
EnNP:
	rmmovq %r8, 8n(%rsi)		# 设置dst[n]=%r8
```

该做法要求进入一个模块前，需要先加载一个数到寄存器 %r8 中。除了余数为 0 时，其他情形都需要 src[0]，所以考虑加载 src[0] 到 %r8 中。

针对 2 的优化，要注意二分查找的分界点，该过程可通过动态规划来计算最少的指令数：

```c++
#include<bits/stdc++.h>
using namespace std;
#define mp make_pair
#define rep(i,l,r) for(int i=(l);i<(r);++i)
#define per(i,l,r) for(int i=(r)-1;i>=(l);--i)
#define dd(x) cout << #x << " = " << x << ", "
#define de(x) cout << #x << " = " << x << endl
//-------

map<pair<int,int>, int> dp;

int dfs(pair<int, int> ij) {
	if (ij.first >= ij.second) return 0;
	if (dp.count(ij)) return dp[ij];
	int l, r;
	tie(l, r) = ij;
	pair<int, int> ret(INT_MAX, INT_MAX);
	rep(m, l, r + 1) {
		int sum = 1 + 1;	// test; je
		if (l <= m - 1) 	// jl, 左分支
			sum += (l < m - 1) + (m - l) + dfs(mp(l, m - 1));
		if (m + 1 <= r)		// jg, 右分支
			sum += (m + 1 < r) + (r - m) + dfs(mp(m + 1, r));
		ret = min(ret, mp(sum, m));
	}
	dd(l), dd(r), dd(ret.first), de(ret.second);
	return dp[ij] = ret.first;
}

int main() {
	int l, r; cin >> l >> r;
	dd(l), de(r);
	int ans = dfs(mp(l, r));
	de(ans);
	return 0;
}

```

在区间为 [0, 9] 时，关键输出为：

```
l = 7, r = 9, ret.first = 4, ret.second = 8
l = 4, r = 5, ret.first = 3, ret.second = 4
l = 4, r = 9, ret.first = 16, ret.second = 6
l = 0, r = 2, ret.first = 4, ret.second = 1
l = 0, r = 9, ret.first = 33, ret.second = 3
```

查找的关键点为：1，3，4，6，8。

在实现过程反复测试中，发现处理器更倾向于总是跳转，结合具体的查找实现，最终的关键点定位：1，3，5，7， 8。

{% asset_img  binarysearch.png 二分查找 %}

优化后的版本：

```assembly
ncopy:
	iaddq $-10, %rdx
	jl Rem

Loop:
	mrmovq (%rdi), %r8
	mrmovq 8(%rdi), %r9
	mrmovq 16(%rdi), %r10
	mrmovq 24(%rdi), %r11
	mrmovq 32(%rdi), %r12
	mrmovq 40(%rdi), %r13
	mrmovq 48(%rdi), %r14
	mrmovq 56(%rdi), %rcx
	mrmovq 64(%rdi), %rbx
	mrmovq 72(%rdi), %rbp
	rmmovq %r8, (%rsi)
	rmmovq %r9, 8(%rsi)
	rmmovq %r10, 16(%rsi)
	rmmovq %r11, 24(%rsi)
	rmmovq %r12, 32(%rsi)
	rmmovq %r13, 40(%rsi)
	rmmovq %r14, 48(%rsi)
	rmmovq %rcx, 56(%rsi)
	rmmovq %rbx, 64(%rsi)
	rmmovq %rbp, 72(%rsi)
	andq %r8, %r8
	jle R10N8
	iaddq $1, %rax
R10N8:
	andq %r9, %r9
	jle R10N9
	iaddq $1, %rax
R10N9:
	andq %r10, %r10
	jle R10N10
	iaddq $1, %rax
R10N10:
	andq %r11, %r11
	jle R10N11
	iaddq $1, %rax
R10N11:
	andq %r12, %r12
	jle R10N12
	iaddq $1, %rax
R10N12:
	andq %r13, %r13
	jle R10N13
	iaddq $1, %rax
R10N13:
	andq %r14, %r14
	jle R10N14
	iaddq $1, %rax
R10N14:
	andq %rcx, %rcx
	jle R10N15
	iaddq $1, %rax
R10N15:
	andq %rbx, %rbx
	jle R10N16
	iaddq $1, %rax
R10N16:
	andq %rbp, %rbp
	jle R10N17
	iaddq $1, %rax
R10N17:
	iaddq $80, %rdi
	iaddq $80, %rsi
	iaddq $-10, %rdx
	jge Loop

Rem:
	mrmovq (%rdi), %r8
	iaddq $7, %rdx
	jge RGE3
R02:
	iaddq $2, %rdx
	jl Done
	rmmovq %r8, (%rsi)
	je R1
	jmp R2

R46:
	iaddq $2, %rdx
	jl R4
	je R5
	jmp R6

RGE3:
	rmmovq %r8, (%rsi)
	je R3

R49:	
	iaddq $-4, %rdx
	jl R46
	je R7

R89:
	iaddq $-1, %rdx
	je R8

R9:
	andq %r8, %r8
	mrmovq 64(%rdi), %r8
	jle R9NP
	iaddq $1, %rax
	R9NP:
	rmmovq %r8, 64(%rsi)
R8:
	andq %r8, %r8
	mrmovq 56(%rdi), %r8
	jle R8NP
	iaddq $1, %rax
	R8NP:
	rmmovq %r8, 56(%rsi)
R7:
	andq %r8, %r8
	mrmovq 48(%rdi), %r8
	jle R7NP
	iaddq $1, %rax
	R7NP:
	rmmovq %r8, 48(%rsi)
R6:
	andq %r8, %r8
	mrmovq 40(%rdi), %r8
	jle R6NP
	iaddq $1, %rax
	R6NP:
	rmmovq %r8, 40(%rsi)
R5:
	andq %r8, %r8
	mrmovq 32(%rdi), %r8
	jle R5NP
	iaddq $1, %rax
	R5NP:
	rmmovq %r8, 32(%rsi)
R4:
	andq %r8, %r8
	mrmovq 24(%rdi), %r8
	jle R4NP
	iaddq $1, %rax
	R4NP:
	rmmovq %r8, 24(%rsi)
R3:
	andq %r8, %r8
	mrmovq 16(%rdi), %r8
	jle R3NP
	iaddq $1, %rax
	R3NP:
	rmmovq %r8, 16(%rsi)
R2:
	andq %r8, %r8
	mrmovq 8(%rdi), %r8
	jle R2NP
	iaddq $1, %rax
	R2NP:
	rmmovq %r8, 8(%rsi)
R1:
	andq %r8, %r8
	jle Done
	iaddq $1, %rax
```

注意，该版本去掉了 `xorq %rax, %rax`。

测试结果：

{% asset_img  result.png 正确性结果 %}

{% asset_img  CPE.png CPE %}

## 参考

1. [csapp archlab Part C](https://zhuanlan.zhihu.com/p/33751460)
2. [csapp archlab 60分解答](https://zhuanlan.zhihu.com/p/77072339)

