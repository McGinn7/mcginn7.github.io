---
title: 回文分解算法
date: 2019-11-20 19:30:36
tags:
---

内容基本是翻译自论文《A Subquadratic Algorithm for Minimum Palindromic Factorization》，主要对文章进行翻译，力图简化算法的证明过程并给出相应的结论。简化证明过程可能存在不严谨的地方，如有需要可自行查看参考资料中的论文原文。

## 太长不看版

给定一字符串，对于每个右端点为 $r$ 的回文子串，将左端点记为 $l_1\lt l_2\lt \dots \lt l_k$，记间距 $Δ_i=l_i-l_{i-1}$，结论有：

1. 间距 $Δ_i$ 构成单调递减序列（严格来说，是单调非增序列），即 $\forall i>1,Δ_i\ge Δ_{i-1}$。
2. 间距 $Δ_i$ 不超过 $\log(r)$ 种。

基于上述 2 点结论，可以在 $\log(r)$ 时间内利用端点 $r-1$ 的信息计算得到 $r$ 的回文左端点，进而对字符串进行回文分解。 

时间复杂度 $O(N\log N)$，空间复杂度 $O(N)$。

## 问题背景

给定长为 $N$  的字符串 $S$，在 $O(N\log N)$ 时间内对字符串 $S$ 分解为最少数量的回文子串，即最小回文分解：
$$
S = S_1S_2\dots S_l
$$
其中，$S_i$ 都是**回文串**。

## 论文摘录

### 引言

首先，定义 $PL(S)$ 表示字符串 $S$ 的最小回文分解的子串数量。如 $PL(abaab)=2$，拆分成 $a$ 和 $baab$ 两个子回文串。

通过**动态规划**的思想，可以在 $O(N^2)$ 时间内计算 $PL(S)$：
$$
PL(S[1..j])=\min _i \{PL(S[1..i-1])+1 : i\le j, S[i..j]\ is\ a\ palindrome\}
$$
具体实现为：对于每个右端点 $j$，维护一个左端点集合 $P_j$，对于每个 $i\in P_j$ 都有 $S[i..j]$ 是回文串。基于集合 $P_j$ 可以枚举计算得到 $P_{j+1}$ 集合：如果 $S[i..j]$ 是回文串且 $S[i-1]=S[j+1]$，则 $i-1\in P_{j+1}$。

该论文的算法主要改进左端点 $P_j$ 的表示，利用回文串的性质将左端点划分成 $O(\log j)$ 个**等差数列**的集合 $G_j$，在 $O(\log j)$ 时间内从 $G_{j-1}$ 转移计算 $G_j$。

---

**Border**：串 $y$ 是串 $x$ 的 border 表示 $y$ 既是 $x$ 的前缀也是 $x$ 的后缀。

1. $x$ 也是其本身的 border。

2. 当 $y \neq x$ 时，称其为 **proper border**。

---

### 引理1 

记串 $y$ 为回文串 $x$ 的后缀，则 $y$ 是 $x$ 的 border 当且仅当 $y$ 是回文串。

### 引理2 

记串 $y$ 为串 $x$ 的 border 并且 $|x|\le 2|y|$，那么 $x$ 是回文串当且仅当 $y$ 是回文串。

### 引理3 

记串 $y$ 为回文串 $x$ 的**真后缀（proper suffix）**，那么 $|x|-|y|$ 是 $x$ 的周期（period）当且仅当 $y$ 是回文串。特别地，$|x|-|y|$ 是 $x$ 最小周期当且仅当 $y$ 是 $x$ 的最长回文真后缀。

- 部分网上博文将 period 译为循环节是不太准确的。例如 $x=aba$ 和 $y=a$，此时 $|x|-|y|=2$，并不能整除 $|x|=3$。因此，这里我译为周期。

### 引理4

记 $x$ 为回文串，$y$ 是 $x$ 的最长回文真后缀，$z$ 是 $y$ 的最长回文真后缀，以及 $u, v$ 满足 $x=uy$ 和 $y=vz$，则有：

(1) $|u|\ge |v|$;

(2) 如果 $|u|\gt |v|$，则 $|u| \gt |z|$;

(3) 如果 $|u|=|v|$，则 $u=v$。

**证明**

(1) 根据**引理3**，$|u|=|x|-|y|$ 是 $x$ 的最小周期，$|v|=|y|-|z|$ 是 $y$ 的最小周期。根据 $|y|$ 的长度分两种情况考虑：

1. 当 $2|y|\le |x|$ 时，有 $|u|\ge|y|>|v|$。
2. 当 $2|y|>|x|$ 时，$u$ 是 $x$ 的周期同时也是 $y$ 的周期。由于 $v$ 是 $y$ 的最小周期，因此 $|u| \ge |v|$。

(2) 显然，根据 $x,y$ 均是回文串，可知 $v$ 是 $x$ 的前缀，记 $x=vw$。根据回文串性质，如图 1 所示， $z$ 是 $w$ 的 border 且 $|w|=|zu|$（注意，这里仅仅是长度相等，并不是说 $w=zu$）。根据条件 $|u|\gt |v|$，可得 $|w|>|y|$。这里使用**反证法**，假设结果不成立即 $|u|\le |z|$，那么 $|w|=|zu|\le 2|z|$，而根据**引理2**的结论 $w$ 是回文串，与 $y$ 是 $x$ 的最长回文真后缀相矛盾，因此假设不成立。

{% asset_img image-20191120213959957.png %}

<center>图 1. 反证法的结果图</center>
(3) 上述证明过程中可知 $v$ 是 $x$ 的前缀，并且 $u$ 也是 $x$ 的前缀。在 $|u|=|v|$ 的条件下，显然 $u=v$。

### 引理5

回文左端点集合 $P_j$ （有序）的点间距是非增的，并且最多有 $O(\log j)$ 种间距。

**证明** 

对于任意 $i\in [2..|P_j|-1]$，记 $x = S[p_{i-1}..j], y=S[p_i..j],z=S[p_{i+1}..j]$，则间距有 $|u|=|x|-|y|$ 和 $|v|=|y|-|z|$。

根据**引理4(1)**有 $|u|\ge|v|$，从而间距非增。一旦间距发生变化即 $|u|\gt |v|$，根据**引理4(2)**有 $|u|>|z|$，进而 $|x|>|u|+|z|>2|z|$，长度至少翻倍，因而发生变化的次数不超过 $O(\log j)$，也就是说只有 $O(\log j)$ 种间距。

---

对于任意正整数间距 Δ，定义
$$
P_{j,Δ}=\{p_i:1<i\le m, p_i-p_{i-1}=Δ\}
$$
特别地，定义 $P_{j,∞}=\{p_1\}$。

对于每个非空 $P_{j,Δ}$ 可以使用三元组 $(\min P_{j,Δ},Δ,|P_{j,Δ}|)$ 表示，同时定义 $G_j$ 为按 Δ 降序的三元组列表，其大小为 $O(\log j)$。

---

### 引理6

记 $p_i$ 和 $p_{i+1}$ 为集合 $P_{j-1,Δ}$ 的两个**连续**元素，则 $p_i-1\in P_j$ 当且仅当 $p_{i+1}-1\in P_j$。

**证明**

根据定义有 $p_{i+1}-p_i=Δ$ 和 $p_i - p_{i-1}=Δ$，根据**引理4(3)**可得 $S[p_i-1]=S[p_{i+1}-1]=c$。当 $S[j]=c$，显然有 $p_i -1\in P_j \Leftrightarrow p_{i+1}-1\in P_j$，也就是说 $p_i-1$ 和 $p_{i+1}-1$ 都能和右端点 $j$ 构成回文串。

利用**引理6**，对于集合 $P_{j, Δ}$ 可以常数时间内更新，也就是可以在 $O(\log j)$ 时间内利用 $G_{j-1}$ 计算得到 $G'_j$：
$$
G'_j =\{(i-1, Δ,k):(i, Δ,k)\in G_{j-1},i>1,and\ S[i-1]=S[j]\}
$$
注意从 $P_{j-1}$ 转移得到 $P_j$ 的过程中，部分左端点 $p_i-1$ 不能够与 $j$ 形成回文串而被剔除，因此 $P_j$ 中的间距会发生改变，需要进一步调整得到正确的三元组列表 $G_j$。

考虑 $P_{j-1,Δ}$ 中的三元组 $(p_i,Δ,k)$，当 $S[p_i-1]=S[j]$ 时，三元组 $(p_i-1,Δ,k)$ 插入到 $G_j$ 中。根据 $P_{j,Δ}$ 的定义，$p_i$ 的前一个元素 $p_{i-1}=p_i-Δ$，左端点 $p_{i-1}$ 可能不与 $j$ 形成新回文串而被剔除，此时 $P_j$ 中的 $p_i-1$ 的前一个元素不为 $p_{i-1}-1$，即新间距 $Δ'\gt Δ$。另一方面，旧端点的剔除只会影响每个组的首元素，因此只需将 $(p_i,Δ,k)$ 拆分成 $(p_i-1, Δ',1)$ 和 $(p_i-1+Δ,Δ,k-1)$ 插入到 $G_j$ 中即可。

因为 $G_j$ 是有序的，所以具体实现过程中可实时记录前一个左端点的位置，以此计算新的间距 $Δ'$。此外，由于 $Δ'>Δ$ 且间距是非增的，所以 $(p_i-1,Δ',1)$ 可能与前一三元组具有相同的间距。通过合并相同间距的三元组，最终得到列表 $G_j$。

---

### 引理8

对于 $k\ge 2$，如果 $(i,Δ,k)\in G_j$，那么 $(i,Δ,k-1)\in G_{j-Δ}$。

**证明**

根据定义，$(i,Δ,k)\in G_j$ 等价于 $P_{j, Δ}=\{i,i+Δ,\dots, i+(k-1)Δ\}$。

要证明 $(i, Δ,k-1)\in G_{j-Δ}$，相当于要证明以下两点：

1. $P_{j-Δ,Δ}\cap [i-Δ+1..j-Δ]=\{i,i+Δ,\dots, i+(k-2)Δ\}$；
2. $P_{j-Δ,Δ}\cap [1..i-Δ]=\emptyset$。

记 $x=S[i-Δ..j],y=S[i..j]$。因为 $x,y$ 均是回文串，因此有 $S[i-Δ..j-Δ]=S[i..j]$，如图 2所示。因为两个串相等，显然 $\forall l\in [i..j]$，都有 $l\in P_j\Leftrightarrow l-Δ\in P_{j-Δ}$。更进一步地，两者的左端点间距也是相同的，即 $\forall l\in (i..j]$，有 $l \in P_{j,Δ}\Leftrightarrow l-Δ\in P_{j-Δ,Δ}$（**注意：**区间是左开右闭的）。因为 $\min P_{j,Δ} = i$，所以 $i-Δ\notin [i-Δ+1..j-Δ]$，第 1 点得证。

{% asset_img image-20191121212552874.png %}

<center>图 2. y是x的border</center>
接下来证明第 2 点。要证明 $P_{j-Δ,Δ}\cap [1..i-Δ]=\emptyset $，只需要证明 $i-2Δ\notin P_{j-Δ}$，即串 $S[i-2Δ..j-Δ]$ 不是回文串。

使用**反证法**证明：假设 $S[i-2Δ..j-Δ]$ 是回文串。记 $w=S[i-2Δ..i-Δ-1]$，$w^R$ 表示串 $w$ 的反转串，如图 3所示。

因为 $S[i-2Δ..j-Δ]$ 是回文串，所以 $S[j-Δ+1..j-Δ]=w^R$。

因为 $S[i-Δ..j-Δ]$ 和 $S[i-Δ..j]$ 是回文串，所以有 $S[i-Δ..i-1]=w$ 和 $S[j-Δ+1..j]=w^R$，所以 $S[i-2Δ..j]=wS[i-Δ..j-Δ]w^R$ 也是回文串，即 $i-Δ\in P_{j,Δ}$。这与定义的 $i=\min P_{j,Δ}$ 相矛盾，所以假设不成立，即 $i-2Δ \notin P_{j-Δ}$。

{% asset_img image-20191121215159983.png %}

<center>图 3. S[i-2Δ..j-Δ]为回文串的示意图</center>
**引理8**阐述了一个事实：当 $|P_{j,Δ}|\ge 2$ 时，$P_{j,Δ} = P_{j-Δ,Δ}\cup\{\max P_{j,Δ}\}$，即 $P_{j,Δ}$ 仅多了一个元素而已。因此在计算 $PL_{j,Δ} = \min \{PL[i-1]+1:i\in P_{j,Δ}\}$ 时，只需考虑多出来的那个元素，维护 $PL_{j-Δ,Δ}$ 的信息即可加速计算。

那么在具体实现中如何维护 $PL_{j-Δ,Δ}$ 呢？暴力做法直接套 *map*，这样的空间复杂度为 $O(N\log N)$。接下来的**引理9**可将空间复杂度降至 $O(N)$。

---

### 引理9

记 $m=\min P_{j,Δ}-Δ$，则 $\forall l\in [j-Δ+1..j-1]$，有 $m\notin P_l$。

**证明**

**反证法**：假设存在 $l\in [j-Δ+1..j-1]$，意味着 $S[m..l]$ 是回文串。

记 $h=l-(j-Δ)$，则 $S[m+h..l-h]$ 也是回文串，并且 $m<m+h<m+Δ$。

然而 $l-h$ 实际上等于 $j-Δ$，所以 $m+d\in P_{j-Δ,Δ}$。并且根据定义有 $m+Δ=\min P_{j-Δ,Δ}$，而 $m+h$ 则介于 $m$ 和 $m+Δ$ 之间，与定义 $P_{j-Δ,Δ}$ 相矛盾，即 $m+Δ$ 的前一个元素为 $m+h$ 而非 $m$ ，因此假设不成立。

{% asset_img image-20191122105243688.png %}

<center>图 4. 假设成立的结果图</center>
**引理9**说明位置 $m$ 在右端点范围 $[j-Δ,j]$ 中只会被 $j-Δ$ 和 $j$ 更新和使用，因此可将 $PL_{j-Δ,Δ}$ 的结果存放在 $GPL[m]$ 中，从而将空间复杂度降低至 $O(N)$。

## 题目

### Palindrome Partition

**来源**  [Codeforces 932G](codeforces.com/problemset/problem/932/G)

**题意**  给一字符串 $s(2\le s\le10^6)$，且 $|s|$ 是偶数。现要将串 $s$ 划分成**偶数**个子串 $(p_1,p_2,\dots,p_{2k})$，满足 $\forall i\in [1,2k], p_i=p_{2k-i+1}$。求满足条件的方案数，结果对 $10^9+7$ 取模。

**分析**  串长 $|s| $ 是偶数，子串数量也是偶数，因此将串 $s$ 折半并按 $s_1s_ns_2s_{n-1}...s_{n/2}s_{n/2+1}$ 构造新串，便将原问题转换成将原串分解为若干偶数长度子回文串的方案数。

---

### Reverses

**来源**  [Codeforces 906E](codeforces.com/problemset/problem/906/E)

**题意**   给定两个字符串 $s, t(1\le |s|=|t| \le 5\times 10^5)$，允许翻转串 $t$ 若干**不相交**子串，使得翻转后串 $t$ 等于 $s$，求最少需要子串个数，并给出任一方案。

**分析**  首先构造串 $str=s_1t_1s_2t_2\dots s_nt_n$。如果子串 $t[i..j]$ 翻转后与串 $s[i..j]$ 相等，则子串 $s_it_i..s_jt_j$ 是回文串。而如果子串 $t[i..j]=s[i..j]$，则 $s_it_i..s_jt_j$ 最坏情况下会分解成若干长度为 2 的子回文串。于是原问题转化成对串 $str$ 进行回文分解，在状态转移过程中长度为 2 的回文串在原问题中不属于翻转串，因此其代价为 0。

## 参考代码

### 回文分解模板

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef double db;
typedef long long ll;
typedef vector<int> vi;
typedef pair<int, int> pii;
#define fi first
#define se second
#define pb push_back
#define mp make_pair
#define sz(x) ((int)(x).size())
#define all(x) begin(x),end(x)
#define rep(i,l,r) for(int i=(l);i<(r);++i)
#define per(i,l,r) for(int i=(r)-1;i>=(l);--i)
#define dd(x) cout << #x << "=" << x << ", "
#define de(x) cout << #x << "=" << x << endl
//-------

const int N = 1e5 + 7;
const int M = 100; // Ensure M = k*log(N)
int PL[N], GPL[N];
tuple<int, int, int> g[M], G[M];

// Ensure str[|str|] = '\0'
void Palindromic(char *str) {
	const char *S = str - 1;	

	int n = strlen(str);
	int G_size = 0;	
	PL[0] = 0;
	rep(j, 1, n + 1) {
		int i, d, k, g_size = 0;
		swap(g_size, G_size);
		rep(_, 0, g_size) g[_] = G[_];	
		
		rep(_, 0, g_size) {
			tie(i, d, k) = g[_];
			if (i > 1 && S[i - 1] == S[j])
				G[G_size++] = {i - 1, d, k};
		}

		g_size = 0;
		int r = -j;
		rep(_, 0, G_size) {
			tie(i, d, k) = G[_];
			if (i - r != d) {
				g[g_size++] = {i, i - r, 1};
				if (k > 1)
					g[g_size++] = {i + d, d, k - 1};
			} else 
				g[g_size++] = {i, d, k};
			r = i + (k - 1) * d;
		}
		if (j > 1 && S[j - 1] == S[j]) {
			g[g_size++] = {j - 1, j - 1 - r, 1};
			r = j - 1;
		}
		g[g_size++] = {j, j - r, 1};

		G_size = 0;		
		tie(i, d, k) = g[0]; 
		rep(_, 1, g_size) {
			if (get<1>(g[_]) == d)
				k += get<2>(g[_]);
			else {
				G[G_size++] = {i, d, k};
				tie(i, d, k) = g[_];
			}
		}
		G[G_size++] = {i, d, k};
		
		PL[j] = j;	
		rep(_, 0, G_size) {
			tie(i, d, k) = G[_];
			r = i + (k - 1) * d;
			int m = PL[r - 1] + 1;
			if (k > 1) m = min(m, GPL[i - d]);
			if (d <= i) GPL[i - d] = m;	
			PL[j] = min(PL[j], m);
		}
	}
}

char str[N];

int main() {
	scanf(" %s", str);
	Palindromic(str);
	return 0;
}
```

### Palindrome Partition

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef double db;
typedef pair<int, int> pii;
typedef vector<int> vi;
#define fi first
#define se second
#define mp make_pair
#define pb push_back
#define sz(x) ((int)(x).size())
#define all(x) (x).begin(),(x).end()
#define rep(i,l,r) for(int i=(l);i<(r);++i)
#define per(i,l,r) for(int i=(r)-1;i>=(l);--i)
#define dd(x) cout << #x << " = " << x << ", "
#define de(x) cout << #x << " = " << x << endl
//-------

const int N = 1e6 + 7;
const int P = 1e9 + 7;
const int M = 100;
int n;
char s[N], t[N];

int PL[N], GPL[N];
tuple<int, int, int> g[M], G[M];

inline void inc(int &x, int y) {
	if ((x += y) >= P) x -= P;
}

void Palindromic(char *str) {
	const char *S = str - 1;
	int n = strlen(str);
	int G_size = 0;
	PL[0] = 1;
	rep(j, 1, n + 1) {
		int i, d, k, g_size = 0;
		swap(g_size, G_size);
		rep(_, 0, g_size) g[_] = G[_];

		rep(_, 0, g_size) {
			tie(i, d, k) = g[_];
			if (i > 1 && S[i - 1] == S[j])
				G[G_size++] = {i - 1, d, k};
		}

		g_size = 0;
		int r = -j;
		rep(_, 0, G_size) {
			tie(i, d, k) = G[_];
			if (i - r != d) {
				g[g_size++] = {i, i - r, 1};
				if (k > 1)
					g[g_size++] = {i + d, d, k - 1};
			} else 
				g[g_size++] = {i, d, k};
			r = i + (k - 1) * d;
		}
		if (j > 1 && S[j - 1] == S[j]) {
			g[g_size++] = {j - 1, j - 1 - r, 1};
			r = j - 1;
		}
		g[g_size++] = {j, j - r, 1};
		
		G_size = 0;
		tie(i, d, k) = g[0];
		rep(_, 1, g_size) {
			if (get<1>(g[_]) == d)
				k += get<2>(g[_]);
			else {
				G[G_size++] = {i, d, k};
				tie(i, d, k) = g[_];
			}
		}
		G[G_size++] = {i, d, k};

		PL[j] = 0;
		rep(_, 0, G_size) {
			tie(i, d, k) = G[_];	
			r = i + (k - 1) * d;
			int m = PL[r - 1];
			if (k > 1) inc(m, GPL[i - d]);
			if (d <= i) GPL[i - d] = m;	
			if (~j & 1) inc(PL[j], m);
		}
	}
}

int main() {
	scanf(" %s", s);
	n = strlen(s);
	rep(i, 0, n / 2) {
		t[2 * i] = s[i];
		t[2 * i + 1] = s[n - 1 - i];	
	}
	t[n] = 0;

	Palindromic(t);
	printf("%d\n", PL[n]);
	return 0;
}
```

### Reverses

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef double db;
typedef long long ll;
typedef vector<int> vi;
typedef pair<int, int> pii;
#define fi first
#define se second
#define pb push_back
#define mp make_pair
#define sz(x) ((int)(x).size())
#define all(x) begin(x),end(x)
#define rep(i,l,r) for(int i=(l);i<(r);++i)
#define per(i,l,r) for(int i=(r)-1;i>=(l);--i)
#define dd(x) cout << #x << "=" << x << ", "
#define de(x) cout << #x << "=" << x << endl
//-------

const int N = 5e5 + 7;
const int M = 100; // Ensure M = log(N)
char s[N], t[N], str[2 * N];

pair<int, int> PL[2 * N], GPL[2 * N];
tuple<int, int, int> g[M], G[M];

void Palindromic(char *str) {
	const char *S = str - 1;

	int n = strlen(str);
	int G_size = 0;	
	PL[0] = {0, 0};
	rep(j, 1, n + 1) {
		int i, d, k, g_size = 0;
		swap(g_size, G_size);
		rep(_, 0, g_size) g[_] = G[_];	
		
		rep(_, 0, g_size) {
			tie(i, d, k) = g[_];
			if (i > 1 && S[i - 1] == S[j])
				G[G_size++] = {i - 1, d, k};
		}

		g_size = 0;
		int r = -j;
		rep(_, 0, G_size) {
			tie(i, d, k) = G[_];
			if (i - r != d) {
				g[g_size++] = {i, i - r, 1};
				if (k > 1)
					g[g_size++] = {i + d, d, k - 1};
			} else 
				g[g_size++] = {i, d, k};
			r = i + (k - 1) * d;
		}
		if (j > 1 && S[j - 1] == S[j]) {
			g[g_size++] = {j - 1, j - 1 - r, 1};
			r = j - 1;
		}
		g[g_size++] = {j, j - r, 1};

		G_size = 0;		
		tie(i, d, k) = g[0]; 
		rep(_, 1, g_size) {
			if (get<1>(g[_]) == d)
				k += get<2>(g[_]);
			else {
				G[G_size++] = {i, d, k};
				tie(i, d, k) = g[_];
			}
		}
		G[G_size++] = {i, d, k};
	
		PL[j] = {n + 1, 0};
		if (j % 2 == 0 && S[j - 1] == S[j])
			PL[j] = min(PL[j], 
						make_pair(PL[j - 2].first, j - 2));
		rep(_, 0, G_size) {
			tie(i, d, k) = G[_];
			r = i + (k - 1) * d;
			// int m = PL[r - 1] + 1;
			pair<int, int> m = {PL[r - 1].first + 1, r - 1};
			if (k > 1) m = min(m, GPL[i - d]);
			if (d <= i) GPL[i - d] = m;	
			if (~j & 1) PL[j] = min(PL[j], m);
		}
	}
}

bool same(int l, int r, const char *s, const char *t) {
	rep(i, l, r + 1) if (s[i] != t[i])
		return false;
	return true;
}

int main() {
	scanf(" %s %s", s, t);	
	int n = strlen(s);
	rep(i, 0, n) {
		str[2 * i] = s[i];
		str[2 * i + 1] = t[i];
	}
	str[2 * n] = 0;	

	Palindromic(str);

	if (PL[2 * n].first > n) {
		puts("-1");
	} else {
		vector<pair<int, int> > ans;			
		for (int r = 2 * n; r > 0; r = PL[r].second) {
			int L = PL[r].second / 2;	
			int R = (r - 1) / 2;
			if (!same(L, R, s, t))
				ans.emplace_back(make_pair(L + 1, R + 1));
		}
		printf("%d\n", ans.size());
		for (auto p : ans)
			printf("%d %d\n", p.first, p.second);
	}

	return 0;
}
```



## 参考资料

- [Fici G, Gagie T, Karkkainen J, et al. A subquadratic algorithm for minimum palindromic factorization[J]. Journal of Discrete Algorithms, 2014: 41-48.]( https://arxiv.org/abs/1403.2431)

  