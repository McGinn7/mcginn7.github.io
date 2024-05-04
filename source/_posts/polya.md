---
title: Pólya计数法
date: 2019-10-31 19:54:39
tags:
- polya
---

# 题目

## Necklace of Beads

**来源** [POJ 1286]( http://poj.org/problem?id=1286)

**题意** 使用 RGB 三种颜色对长为 $0\le n\le 23$ 的项链染色，求本质不同的方案数。一种方案如果经过**旋转**或**翻转**得到另一种方案，两种方案视为同一种。

**注意** $n=0$ 在本题中会出现，测试数据输出 0。

**分析** 使用 Polya 计数公式求解，即对于正 $n$ 边形的顶点对称群
$$
 \{ρ_n^0=\iota,ρ_n,\dots, ρ_n^{n-1}, τ_1, τ_2, \dots, τ_n \} 
$$
的循环因子分解。

1. 旋转置换 $ρ_n^i,i=0,1,\dots n-1$ 的循环个数为
   $$
   \gcd(n, i)
   $$
   **证明** 旋转置换中的每个元素在一个有向圈 $s+ki=s(\%\ n)$ ，其中 $k$ 是最小正整数。那么 $n|ki$，令 $ki=lcm(n, i)=ni/\gcd(n, i)$，则 $k=n/\gcd(n, i)$。所以有向圈的长度为 $n/\gcd(n, i)$，个数为 $n/k=\gcd(n,i)$ 个。

   根据定理 3 可得，
   $$
   |C(ρ_n^i)|=3^{\gcd(n, i)}
   $$
   
2. 反射置换 $τ_i$ 需要根据 $n$ 的奇偶性考虑。

   当 $n$ 为奇数时，有 $n$ 个关于角点与其对边中点的连线的反射，每个反射置换的型为
   $$
   (1, \frac{n-1}{2}, 0, \dots, 0)
   $$
   根据定理 3 可得
   $$
   |C(τ_i)|=3^{\frac{n+1}{2}}
   $$
   当 $n$ 为偶数时，有 $n/2$ 个关于对角点的反射和 $n/2$ 个关于对边中点连线的反射，两种置换的型分别为
   $$
   (2, \frac n2-1, 0, \dots, 0)\\
   (0, \frac n 2, 0, \dots, 0)
   $$
   
   根据置换的型和定理 3 可求得 $|C(f)|$，最后使用 Burnside 定理即可求解不同着色的方案数。

## Color

**来源** [POJ 2154]( http://poj.org/problem?id=2154)，[LG 4980]( https://www.luogu.org/problem/P4980)

**题意** 给定长为 $N\le 10^9$ 的项链，使用至多 $N$ 种颜色对项链上的珠子着色，考虑在**旋转**条件下的不同着色方案数，结果对 $1\le P \le 3\times 10^4$ 取模。

**分析** 旋转置换 $ρ_n^i,i=0,1,\dots n-1$ 的循环个数为
$$
\gcd(n, i)
$$
根据 Polya 定理，总方案数为
$$
Answer = \frac 1 n \sum_{0\le i \lt n} |C(\rho_n^i)|
= \frac 1 n \sum_{0\le i \lt n} n^{\gcd(n, i)} \\
= \frac 1 n \sum_{d|n}n^d\sum_{0\le i \lt n}[\gcd(n, i) =d] \\
= \frac 1 n \sum_{d|n}n^d\sum_{0\le i \lt n}[\gcd(n/d, i/d) =1] \\
= \frac 1 n \sum_{d|n}n^d\sum_{0\le i \lt n/d}[\gcd(n/d, i) =1] \\
= \frac 1 n \sum_{d|n}n^d\phi(n/d) 
= \sum_{d|n}n^{d-1}\phi(n/d) \\
$$
$O(\sqrt n)$ 枚举 $n$ 的所有约数 $d$（因为前 10 个质数乘积大于 $10^9$，因此约数个数最多只有 $2^{10}=1024$ 个），快速幂求 $n^{d-1}$，$O(\sqrt {n/d})$ 求 $\phi(n/d)$。

## Magic Bracelet

**来源** [POJ 2888]( http://poj.org/problem?id=2888)

**题意** 给定长为 $1\le n \le 10^9$  的项链，使用 $1\le m \le 10$ 种颜色对项链上的珠子着色，考虑旋转同构的不同着色方案数，结果对 9973 取模。此外，限制部分颜色对不能着色于相邻的珠子上。数据保证 $gcd(n, 9973)=1$。

**分析** 已知循环置换 $ρ_n^i$ 的循环个数为 $\gcd(n, i)$。对于一个循环圈可表示为 $j+ki(\mod n)$，则在同一个循环的元素 $x, y$ 要满足 $x \mod \gcd(n, i) = y \mod  \gcd(n, i)$，即循环节大小为 $\gcd(n, i)$。因为相邻珠子不能着限制颜色对，此时保证前 $\gcd(n, i) + 1$ 个元素不出现禁止的颜色对即可，问题转化成求解 $k$ 个珠子的项链不出现禁止颜色对的方案数。

转化后的问题可以使用动态规划解决，$dp(i, j)$ 表示前 $i$ 个元素中最后一个元素的颜色为 $j$ 的方案数。记不能相邻的颜色对集合为 $E$，则转移方程为
$$
dp(i, j) = \sum_{(k,j)\notin E}dp(i-1,k)
$$
由于 $n$ 很大而 $m$ 很小，因此可以使用矩阵 + 快速幂解决。另一方面，项链首尾相接的问题可以通过枚举第一个元素的颜色解决。

## Birthday Toy

**来源** [HDU 2865]( http://acm.hdu.edu.cn/showproblem.php?pid=2865)

**题意** 使用 $4\le k \le 10^9$ 种颜色对形如图 1的项链着色，项链包括 $3\le N \le 10^9$ 个小珠子以及一个中心的大珠子。要求相邻珠子不能同色，求旋转同构的不同着色方案数，结果对 $10^9+7$ 取模。

{% asset_img image-20191102224432497.png %}

<center>图 1. 特殊形状的项链</center>
**分析** 由于中心点的大珠子与所有小珠子均相邻，因此可从 $k$ 种颜色中先选一种颜色对大珠子着色，使得问题转化为使用 $k-1$ 种颜色对普通项链着色。

进而本问题与 **POJ 2888 Magic Bracelet** 类似，问题进一步转化成长为 $L$ 的项链相邻珠子不同构的着色方案数。使用动态规划思想解决，$dp(i, j)$ 表示前 $i$ 个珠子最后一个颜色为 $j$ 的方案数，转移方程为
$$
dp(i, j) = \sum_k{dp(i-1,k)} - dp(i-1, j)
$$
项链需要解决首尾相接问题，此时通过 $O(k)$ 枚举第一个元素的颜色解决。由于颜色数 $k$ 很大，因此不能使用线性递推的动态规划做法。

在枚举第一个元素的颜色时，不失一般性，假设第一个元素的颜色为 1，那么颜色 2,...,k 是等价的。可以理解为只用 2 种颜色着色，只是第 2 种颜色有 $k - 1$ 种替换， ​因此动态规划的状态只有 $dp(i, 1)$ 和 $dp(i, c), c=2,\dots, k$，因此转移方程可写成
$$
dp(i, 1) = dp(i-1, 1)+(k-1)dp(i, c) - dp(i-1,1) \\
dp(i, c) = dp(i-1,1)+(k-1)dp(i, c)-dp(i-1,c) = dp(i-1,1)+(k-2)dp(i-2,c)
$$
写成矩阵形式
$$
\begin{pmatrix}
0&k-1\\
1&k-2
\end{pmatrix}
\begin{pmatrix}
dp(i-1,1)\\
dp(i-1,c)
\end{pmatrix}
=
\begin{pmatrix}
dp(i,1)\\
dp(i,c)
\end{pmatrix}
$$
使用快速幂优优化计划 $dp(i, 1)$，进而方案数为
$$
Answer = \frac 1 n \sum_{d|n} dp(d, 1)φ(n / d)
$$

# 《组合数学》摘录

## 置换群与对称群

### 置换

**定义** 设 $X$ 是一个有限集 $\{1, 2, \dots , n\}$，$X$ 的每个置换 $f$
$$
f(1)=i_1, f(2)=i_2, \dots , f(n)=i_n
$$
可视为到其自身定义的一对一函数 $f:X\rightarrow X$，用 $2\times n $ 阵列表示：
$$
\begin{pmatrix}
1&2&\dots&n\\
i_1&i_2&\dots&i_n
\end{pmatrix} \tag1
$$
将 $\{1, 2, \dots, n\}$ 的所有 $n!$ 个置换构成的集合记为 $S_n$。

**运算** 置换的合成运算“$\circ$”满足**结合律**，但不满足交换律：
$$
(f\circ g)\circ h=f\circ(g\circ h) \tag 2
$$
**恒等置换** 各数对应到自身的置换 $\iota$：
$$
\iota = \begin{pmatrix}
1&2&\dots&n \\
1&2&\dots&n \\
\end{pmatrix} \tag 3
$$
**逆函数** 如果 $f(s)=k$，那么 $f^{-1}(k)=s$。

### 置换群

**定义** 若 $S_n$ 的非空子集 $G$ 为 $X$ 的一个**置换群**，则满足：

1. $\forall f, g\in G, f\circ g \in G$，即合成运算的封闭性。
2. 恒等置换 $\iota \in G$，即包含单位元。
3. $\forall f \in G，f^{-1}\in G$，即逆元的封闭性。

**特殊** $X=\{1, 2, \dots, n\}$ 的所有置换的集合 $S_n$ 是一个置换群，记为 $n$ **阶对称群**。集合 $G=\{\iota\}$ 也是一个置换群。

**性质** 置换群满足**消去律**：若 $f\circ g=f\circ h$，那么 $g=h$。

## Burnside定理

计算集合 $X$ 的不等价着色数。

设 $G$ 是 $X$ 的一个置换群，$C$ 是一个着色集合，使着色 $c$ 保持不变的集合：
$$
G(c)=\{f:f\in G, f\ast c=c\} \tag{4}
$$
集合 $G(c)$ 称为 $c$ 的**稳定核**，任何着色的稳定核是一个置换群。

在 $f$ 作用下使着色 $c$ 保持不变的 $G$ 中所有着色的集合：
$$
C(f)=\{c:c\in C, f\ast c=c\} \tag 5
$$
**定理1** 对于每一种着色 $c$，$c$ 的稳定核 $G(c)$ 是一个置换群，且对 $G$ 中任意置换 $f,g$, $g\ast c=f\ast c$ 当且仅当 $f^{-1}\circ g$ 属于 $G(c)$。

**推论1** 设 $c$ 为 $C$ 中的一种着色，那么与 $c$ **等价的着色数**等于 $G$ 中的置换个数除以 $c$ 的稳定核中的置换个数
$$
|\{f\ast c:f\in G\}|=\frac{|G|}{|G(c)|} \tag 6
$$
**证明** 对于 $h\in G(c)$，有 $(f\circ h)\ast c=f\ast (h\ast c)=f\ast c$，从而对于每个置换 $f$，恰好存在 $|G(c)|$ 个置换，这些置换作用在 $c$ 上跟 $f$ 有同样的效果。

**定理2** 设 $G$ 是 $X$ 的一个置换群，$C$ 是 $X$ 的一个着色集并且使得对于 $G$ 中的任意 $f$ 与 $C$ 中的任意 $c$，$f\ast c\in C$，则 $C$ 中不等价的着色数 $N(G, C)$ 为
$$
N(G, C)=\frac1 {|G|} \sum_{f\in G}|C(f)| \tag 7
$$
换言之，$C$ 中不等价的着色数等于使着色通过 $G$ 中的置换保持不变的着色的平均数。

**证明** 计数满足 $f\ast c=c$ 的 $(f, c)$ 个数。使用两种不同的方式计数，然后使计数相等。

一种从置换 $f$ 考察，根据**等价着色的定义**计数结果为
$$
\sum_{f\in G} |C(f)| \tag 8
$$
一种从着色 $c$ 考察，每个 $c$ 对结果的贡献为
$$
|G(c)|=\frac{|G|}{(与 c 等价的着色数)} \tag 9
$$
计数结果为
$$
\sum_{c \in C} \frac{|G|}{(与c等价的着色数)} \tag {10}
$$
按等价类（根据置换群的性值，等价具有传递性）将着色归类，每个等价类的总贡献为 $|G|$，等价类的个数就是不等价类的着色数 $N(G, C)$，因此公式(10) 等于
$$
N(G,C)\times |G| \tag {11}
$$
联立公式(8) 和公式(11) 得到
$$
\sum_{f\in G}|C(f)|=N(G, C)\times |G| \tag {12}
$$

## Pólya计数公式

通过考虑置换的循环结构，计算可变得容易简便。

设 $f$ 是 $X=\{1, 2, \dots, n\}$ 的一个置换，$D_f=(X, A_f)$ 是顶点集为 $X$ 且弧集为
$$
A_f = \{(i, f(i)):i\in X\}
$$
的有向图。该有向图有 $n$ 个顶点与 $n$ 条弧，各顶点的入度和出度等于1，因此弧集 $A_f$ 被划分为若干个有向圈，且每个顶点恰好只属于一个有向圈。

如果某些元素以循环的方式被置换且余下元素保持不变，那么称这样的置换为**循环置换**或简称**循环**。如果循环中的元素个数为 $k$，则称它为 $k-$循环。

设 $f$ 是集合 $X$ 的任意置换，关于合成运算 $f$ 有化成循环的因子分解
$$
f=[i_1\ i_2\ \dots\ i_p]\circ[j_1\ j_2\ \dots\ j_q]\circ\dots\circ[l_1\ l_2\ \dots\ l_r] \tag {13}
$$
公式(13) 称为 $f$ 循环因子分解。

对于 $f$ 分解中的每个循环，该循环中的所有元素着色相同，因此着色方案数与循环阶数无关，而与循环个数有关。置换 $f$ 的循环因子分解中的循环个数记为
$$
\#(f)
$$
**定理3** 设 $f$ 是集合 $X$ 的一个置换，假如用 $k$ 种颜色对 $X$ 的元素进行着色，令 $C$ 是 $X$ 的所有着色的集合，则 $f$ 保持 $C$ 中着色不变的着色数为
$$
|C(f)|=k^{\#(f)} 
$$
假设 $f$ 的循环因子分解有 $e_i$ 个 $i$-循环，因 $X$ 的各元素在 $f$ 循环因子分解中恰好出现在一个训话中，所以 $e_i$ 是非负整数且满足
$$
\sum_{i=1}^{n}ie_i=n \tag {14}
$$
称 $n$ 元组 $(e_1, e_2, \dots, e_n)$ 是置换 $f$ 的**型**，记为
$$
type(f)=(e_1, e_2, \dots, e_n)
$$
循环数为
$$
\#(f)=e_1+e_2+\dots+e_n
$$
因为置换的型仅取决于循环因子分解中循环的阶数，所以不同置换可以有相同的型，我们可引进 $n$ 个不定元
$$
z_1, z_2, \dots, z_n
$$
其中，$z_k$ 对应一个 $k$ 阶循环（$k=1, 2, \dots, n$）。对于具有 $type(f)=(e_1, e_2, \dots, e_n)$ 的每个置换 $f$，定义 $f$ 的**单项式**为
$$
mon(f)=z_1^{e_1}z_2^{e_2}\dots z_n^{e_n}
$$
设 $G$ 是 $X$ 的一个置换群。对 $G$ 中每个置换 $f$ 的单项式求和，得到关于 $G$ 中的置换按照型的生成函数
$$
\sum_{f\in G} mon(f)=\sum_{f\in G}z_1^{e_1}z_2^{e_2}\dots z_n^{e_n} \tag{15}
$$
合并公式(15) 中的同类型，$z_1^{e_1}z_2^{e_2}\dots z_n^{e_n}$ 的系数等于型为 $(e_1, e_2,\dots, e_n)$ 的 $G$ 中的置换个数。

$G$ 的**循环指数**定义为该生成函数除以 $G$ 中的置换个数 $G$，即
$$
P_G(z_1, z_2, \dots , z_n) = \frac 1 {|G|} \sum_{f\in G} z_1^{e_1}z_2^{e_2}\dots z_n^{e_n}
$$
**定理4** 设 $X$ 是有 $n$ 个元素的一个集合，假设有 $k$ 种可用的颜色集可用来对 $X$ 的元素进行着色。令 $C$ 是 $X$ 的所有 $k^n$ 种着色的集合，$G$ 是 $X$ 的一个置换群。则不等价的着色数是用 $z_i=k(i=1,2,\dots, n)$ 带入 $G$ 的循环指数中而得到的数，即
$$
N(G,C)=P_G(k,k,\dots, k)
$$
**定理5（Polya定理）** 设 $X$ 是一个元素集合，$G$ 是 $X$ 的一个置换群，$\{u_1, u_2, \dots, u_k\}$ 是 $k$ 种颜色的一个集合，$C$ 是 $X$ 的任意着色集并且 $G$ 为 $C$ 上的一个置换群，那么根据各颜色的数目，$C$ 的不等价着色数的生成函数是由循环指数 $P_G(z_1, z_2, \dots, z_n)$ 通过做变量代换
$$
z_j=u_1^j+\dots+u_k^j\ (j=1, 2, \dots, n)
$$
而得到的表达式
$$
P_G(u_1+\dots+u_k, u_1^2+\dots+u_k^2,\dots, u_1^n+\dots+u_k^n) \tag{16}
$$
换言之，公式(16) 中
$$
u_1^{p_1}u_2^{p_2}\dots u_k^{p_k}
$$
的系数等于 $X$ 中的 $p_i$ 个元素着颜色 $u_i$ 的 $C$ 中不等价的着色数。

# 参考代码

## Necklace of Beads

```cpp
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<iostream>
#include<vector>
#include<queue>
#include<set>
#include<map>
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

inline int __gcd(int x, int y) {
	return !x ? y : __gcd(y % x, x);	
}

ll kpow(ll a, ll b) {
	ll r = 1;
	while (b > 0) {
		if (b & 1) r = r * a;
		a = a * a, b >>= 1;
	}
	return r;
}

int main() {	
	int n;
	while (~scanf("%d", &n) && ~n) {
		if (n == 0) {
			puts("0");
			continue;
		}
		ll ans = 0; 
		// rotation
		rep(i, 0, n) {
			ll circ = __gcd(n, i);	
			ans += kpow(3, circ);	
		}
		// relection
		ans += (2 + (n & 1)) * n * kpow(3, n / 2);
		ans /= 2 * n;
		printf("%I64d\n", ans);
	}
	return 0;
}
```

## Color

```cpp
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<iostream>
#include<vector>
#include<queue>
#include<set>
#include<map>
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

const int N = 31625;

int n, p;
vector<int> primes;
vector<bool> isprime;

int kpow(int a, int b, int mod) {
	a %= mod;
	int r = 1;
	while (b > 0) {
		if (b & 1) r = r * a % mod;
		a = a * a % mod, b >>= 1;
	}
	return r;
}

void init() {
	isprime.resize(N, true);
	isprime[0] = isprime[1] = false;
	rep(i, 2, N) if (isprime[i]) {
		primes.push_back(i);
		for (int j = i * i; j < N; j += i)
			isprime[j] = false;
	}
}

int phi(int n) {
	int ret = n;
	rep(i, 0, sz(primes)) if (primes[i] <= n) {
		const int &p = primes[i];
		if (n % p == 0) {
			ret -= ret / p;
			while (n % p == 0) n /= p;
		}
	} else
		break;
	if (n > 1)
		ret -= ret / n;
	return ret;	
}

int main() {
	init();

	int cases; scanf("%d", &cases);
	rep(casei, 0, cases) {	
		int ans = 0;
		scanf("%d%d", &n, &p);
		for (int d = 1; d * d <= n; ++d) {
			if (n % d) continue;
			ans += 1ll * kpow(n, d - 1, p) * phi(n / d) % p;		
			if (d * d != n) 
				ans += 1ll * kpow(n, n / d - 1, p) * phi(d) % p;
			ans %= p;	
		}
		printf("%d\n", ans);

	}
	return 0;
}
```

## Magic Bracelet

```cpp
#include<cstdio>
#include<cstring>
#include<algorithm>
#include<iostream>
#include<vector>
#include<queue>
#include<set>
#include<map>
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

const int P = 9973;
const int M = 10;
const int N = 31625;
int n, m, k;
vector<int> prime;
bool isprime[N];

struct Matrix {	
	int a[M][M];
	void init(int x) {
		rep(i, 0, m) rep(j, 0, m)
			a[i][j] = i == j ? x : 0;
	}
	void fill(int x) {
		rep(i, 0, m) rep(j, 0, m)
			a[i][j] = x;
	}
	int trace() {
		int ret = 0;	
		rep(i, 0, m) ret += a[i][i];
		return ret % P;
	}
	Matrix operator*(const Matrix &mat) const {
		Matrix r; r.init(0);
		rep(i, 0, m) rep(j, 0, m) {
			rep(k, 0, m) r.a[i][j] += a[i][k] * mat.a[k][j];		
			r.a[i][j] %= P;
		}
		return r;
	}
	Matrix operator^(int n) {
		Matrix r, a = *this;
		r.init(1);
		while (n > 0) {
			if (n & 1) r = r * a;
			a = a * a, n >>= 1;
		}
		return r;
	}
};

inline void inc(int &x, int y) {
	if ((x += y) >= P) x -= P;
}

void initPrime() {
	memset(isprime, 1, sizeof(isprime));
	isprime[0] = isprime[1] = false;
	rep(i, 2, N) {
		if (isprime[i]) prime.push_back(i);
		for (int j = 0; i * prime[j] < N; ++j) {
			isprime[i * prime[j]] = false;
			if (i % prime[j] == 0) break;	
		}
	}
}

int phi(int n) {
	int ret = n;
	rep(i, 0, sz(prime)) {
		if (n < prime[i]) break;
		if (n % prime[i] == 0) {
			ret -= ret / prime[i];
			while (n % prime[i] == 0)
				n /= prime[i];
		}
	}
	if (n > 1) ret -= ret / n;
	return ret;
}

int f(int n, int d, Matrix &a) {
	int fd = (a ^ d).trace();
	return phi(n / d) % P * fd % P;
}

void exgcd(int a, int b, int &x, int &y) {
	if (!b) x = 1, y = 0;
	else {
		exgcd(b, a % b, y, x);
		y -= a / b * x;
	}
}

int inv(int n) {
	int x, y;
	exgcd(n, P, x, y);
	x = (x % P + P) % P;	
	return x;
}

int main() {
	initPrime();

	int cases; scanf("%d", &cases);
	while (cases-- > 0) {
		scanf("%d%d%d", &n, &m, &k);
		Matrix a; a.fill(1);
		rep(_k, 0, k) {
			int x, y; scanf("%d%d", &x, &y);
			--x, --y;
			a.a[x][y] = a.a[y][x] = 0;
		}
		int ans = 0;
		for (int d = 1; d * d <= n; ++d) {
			if (n % d) continue;
			inc(ans, f(n, d, a));
			if (d * d != n) 
				inc(ans, f(n, n / d, a));
		}
		ans = ans * inv(n) % P;
		printf("%d\n", ans);
	}
	return 0;
}
```

## Birthday Toy

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

const int P = 1e9 + 7;
const int M = 2;
const int N = 31625;

int n, k;
vector<int> prime;
bool isprime[N];

inline void inc(int &x, int y) {
	if ((x += y) >= P) x -= P;
}

int kpow(int a, int b) {
	a %= P;
	int r = 1;
	while (b > 0) {
		if (b & 1) r = 1ll * r * a % P;
		a = 1ll * a * a % P, b >>= 1;
	}
	return r;
}

struct Matrix {
	int a[M][M];
	void init(int x) {
		rep(i, 0, M) rep(j, 0, M)
			a[i][j] = i == j ? x : 0;
	}
	Matrix operator*(const Matrix &mat) const {
		Matrix r; r.init(0);
		rep(i, 0, M) rep(j, 0, M) rep(k, 0, M)
			inc(r.a[i][j], 1ll * a[i][k] * mat.a[k][j] % P);
		return r;
	}
	Matrix operator^(int n) {
		Matrix r, a = *this;
		r.init(1);
		while (n > 0) {
			if (n & 1) r = r * a;
			a = a * a, n >>= 1;
		}
		return r;
	}
};

void initPrime() {
	memset(isprime, true, sizeof(isprime));
	isprime[0] = isprime[1] = false;
	rep(i, 2, N) {
		if (isprime[i]) prime.push_back(i);
		for (int j = 0; i * prime[j] < N; ++j) {
			isprime[i * prime[j]] = false;
			if (i % prime[j] == 0) break;
		}
	}
}

int phi(int n) {
	int ret = n;
	rep(i, 0, sz(prime)) {
		if (n < prime[i]) break;
		if (n % prime[i] == 0) {
			ret -= ret / prime[i];
			while (n % prime[i] == 0)
				n /= prime[i];
		}
	}
	if (n > 1) ret -= ret / n;
	return ret;
}

int f(int n, int m, int d, Matrix &a) {
	int fd = 1ll * m * (a ^ d).a[0][0] % P;
	return 1ll * fd * phi(n / d) % P;
}

int main() {
	initPrime();
	while (~scanf("%d%d", &n, &k)) {
		int ans = 0;
		int m = k - 1; // #color for small beads
		Matrix a;
		a.a[0][0] = 0, a.a[0][1] = m - 1;
		a.a[1][0] = 1, a.a[1][1] = m - 2;
		for (int d = 1; d * d <= n; ++d) {
			if (n % d) continue;		
			inc(ans, f(n, m, d, a));		
			if (d * d != n)
				inc(ans, f(n, m, n / d, a));
		}
		ans = 1ll * ans * kpow(n, P - 2) % P;	
		ans = 1ll * ans * k % P;		
		printf("%d\n", ans);
	}
	return 0;
}
```

# 参考资料

- Richard A. Brualdi. 组合数学 [M].  机械工业出版社, 2012