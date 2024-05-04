---
title: Nowcoder-出题人的数组
date: 2019-03-31 20:11:49
tags: 
- ICPC
- 贪心
---

链接：<https://ac.nowcoder.com/acm/contest/545/C>
来源：牛客网

#### 题目描述

出题人有两个数组 $A, B$，请你把两个数组归并起来使得 $Cost=∑i∗C_i$ 最小，要求两个原数组的顺序在新数组中保持不变。 

#### 输入描述

第一行输入两个正整数 $n,m$，分别表示数组 $A, B$ 的长度。
第二行输入 $n$ 个正整数，表示数组 $A$。
第二行输入 $m$ 个正整数，表示数组 $B$ 。

#### 输出描述

一个正整数，表示最小代价 $Cost$。

#### 示例 1

<table style="border-collapse:collapse;border-spacing:0;border-color:#999" class="tg"><tr><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 10px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:#fd6864;color:#fff;background-color:#fd6864;text-align:left">输入</th><th style="font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 10px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:#fd6864;color:#fff;background-color:#fd6864;text-align:left">输出</th></tr><tr><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 10px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:#fd6864;color:#444;background-color:#F7FDFA;text-align:left">3 3<br>1 3 5<br>2 6 4</td><td style="font-family:Arial, sans-serif;font-size:14px;padding:10px 10px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;border-color:#fd6864;color:#444;background-color:#F7FDFA;text-align:left">75</td></tr></table>

#### 备注

$n, m \le 100000$

$A_i, B_i \le 100000$



#### 解题思路

$O(nm)$ 动态规划很容易想到，但是复杂度太高且没有方法优化，那么就考虑贪心解法。

显然，合并后的数组 $C$ 格式为 $\dots ABABA\dots$，即一段 $A$ 接一段 $B$ 。

常见的贪心策略为，考虑相邻元素的交换是否会导致更优的结果。由于不能打乱原先的顺序，故总是后段的前缀替换前段的后缀，不失一般性，我们可以假设前段为 $A$，后段为 $B$。

记 $Cost(A) = \sum_{i = 1} ^ {|A|}i\times A_i$，则原先的贡献值为 $Cost(A)+Cost(B)+|A|\times Sum(B)$，交换后的贡献值为 $Cost(A)+Cost(B)+|B|\times Sum(A)$，则当 $|B|\times Sum(A) \lt |A|\times Sum(B)$，即
$$
\frac{Sum(A)}{|A|} \lt \frac{Sum(B)}{|B|}
$$
也就是说，**均值越大的段需要优先选择**。

剩下的就是如何构造这些段，我们假设串 $A=A_1A_2$，当 $Average(A_1)\lt Average(A_2)$ 时，在数组 $C$ 中总会合并成一段，根据这一性质在原数组中利用**单调栈**即可构造初始的段，之后就是从数组 $A,B​$ 贪心选择均值较大的段。

```c++
#include<bits/stdc++.h>
using namespace std;
typedef double db;
typedef long long ll;
typedef vector<int> vi;
typedef pair<int, int> pii;
#define fi first
#define se second
#define pb push_back
#define sz(x) ((int)(x).size())
#define all(x) begin(x),end(x)
#define rep(i,l,r) for(int i=(l);i<(r);++i)
#define per(i,l,r) for(int i=(r)-1;i>=(l);--i)
#define dd(x) cout << #x << "=" << x << ", "
#define de(x) cout << #x << "=" << x << endl
//-------
const int N = 1e5 + 7;
int n, m, A[N], B[N];
struct Node {
    int num; ll sum;
    Node() {}
    Node(int _num, ll _sum) {
        num = _num, sum = _sum;
    }
    Node operator+(const Node &p) const {
        return Node(num + p.num, sum + p.sum);
    }
    bool operator<(const Node &p) const {
        return sum * p.num < p.sum * num; 
    }
} a[N], b[N];
int gao(int n, int *r, Node *a) {
	int top = 0;
    rep(i, 0, n) {
        scanf("%d", r + i);
        Node v(1, r[i]);    
        while (top > 0 && a[top - 1] < v)
            v = a[--top] + v;
        a[top++] = v;
    }
    return top;
}
int main() {
    scanf("%d%d", &n, &m);
	int la = gao(n, A, a), lb = gao(m, B, b);
    n = m = 0;
    ll ans = 0;
    for (int i = 0, j = 0; i < la || j < lb; ) {
        while (i < la && (j == lb || !(a[i] < b[j]))) {
            rep(k, n, n + a[i].num) ans += 1ll * (k + m + 1) * A[k];    
            n += a[i++].num;
        }
        while (j < lb && (i == la || !(b[j] < a[i]))) {
            rep(k, m, m + b[j].num) ans += 1ll * (k + n + 1) * B[k];
            m += b[j++].num;
        }
    }
    printf("%lld\n", ans);
    return 0;
}
```

