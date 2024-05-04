---
title: Knuth-Morris-Pratt Algorithm
date: 2019-05-24 23:39:27
tags:
- KMP
---

KMP 算法解决在文本串（text）快速找出单词（word）的所有出现位置。

暴力匹配的时间复杂度为 $O(|T||W|)$，而 KMP 算法通过引入**最长前后缀**，将检索的时间复杂度降至线性。

## 最长前后缀

> lps indicates longest proper prefix which is also suffix.

最长前后缀（**LPS, Longest proper Prefix and Suffix**）表示既是原串 $S$ 的真前缀也是后缀的最长子串 $T$，其中 $|T|\lt |S|$。
$$
LPS(aaa) = aa \\
LPS(abcdab)=ab
$$

## 检索过程

假设已知**单词串**的每个前缀 $W[0: i]$ 的最长前后缀长度 $lps(i)$，且已经匹配 $T[i - j: i] = W[0:j]$。继续匹配有两种情形：

1. $T[i+1]=W[j + 1]$，则匹配长度 +1。
2. $T[i+ 1] \neq W[j+1]$，此时显然要重新找单词串的一个**最长**前缀 $W[0:k], k\lt j$，使得 $T[i-k:i]=W[0:k]$ 且 $T[i + 1]=W[k+ 1]$，继续与 $i+1$ 结尾的文本串匹配。

{% asset_img 1555835282553.png 情形 2 示意图 %}

<center>图 1. 情形 2 示意图。虚线框表示相同部分。</center>
此时 $W[0:k]$ 与 $W[0:j]$ 的后缀相同，同时其本身是前缀。

令 $k=lps(j)$，若 $T[i+1]=W[k+1]$，则继续匹配。否则将 $k$ 视为新的 $j$，则转化成情形 2 相同的子问题。

**时间复杂度**：匹配成功的复杂度是线性的。而匹配失败时会减小单词串的前缀长度，减一长度**至少**对应一次的成功匹配，此时时间复杂度也是线性的。故算法总的时间复杂度是线性的。

## 预处理

对于单词串的最长前后缀 $lps(i)$，本质上是单词串的自我匹配，即**此时文本串为单词串**。对应于检索过程中的两种情形，可以很容易地完成 $lps(i)$ 的构造。

## 参考代码

```c++
// n = |T|, m = |W|, index = [0, n)
lps[0] = -1;
for (int i = 1, j = -1; i < m; ++i) {
	while (j >= 0 && W[i] != W[j + 1])
		j = lps[j];
	j += W[i] == W[j + 1];
	lps[i] = j;
}
for (int i = 0, j = -1; i < n; ++i) {
	while (j >= 0 && T[i] != W[j + 1])
		j = lps[j];
	j += T[i] == W[j + 1];
	if (j == m - 1) {
		// match successfully
		j = lps[j];
	}
}
```