---
title: Kuhn-Munkres Algorithm
date: 2020-05-10 14:17:51
tags:
---

1. 背景：带边权的二分图**最大权值完美匹配**。

2. 顶标概念：顶点权值 $L_x, L_y$。

3. 可行顶标：**所有**边 $e(x→y)$  ，都满足 $L_x+L_y\ge W_e$。

   **注：**此时还没有开始匹配，是对原图所有边的限制。

4. 相等子图：包含所有顶点；仅包含 $W_{x→y}=L_x+L_y$ 的边。

   问题即转化为求相等子图的完美匹配，此时边权和达到上限 $\sum L_x+ \sum L_y$。

5. 初始化：$L_y=0, L_x=\max_{j\in Y}{W_{x→j}}$ 使得初始方案为可行顶标。

6. 交替路：与匈牙利算法类似，在相等子图中搜索增广路时，如果找不到时我们称此时得到一条交替路。

   如果将**交替路**中 $X$ 集合的顶标都 $-d, d>0$，$Y$ 集合的顶标都 $+d$，有以下结论：

   - 如果 $x→y$ 在相等子图中，$L_x+L_y$ 无变化，即该边仍在相等子图中。

   - 如果 $x→y$ 不在相等子图中，**当 $x$ 在交替路而 $y$ 不在交替路时**，$L_x+L_y$ 将减少 $d$，意味着该边有可能加入相等子图中。注意 $d$ 不能过大，否则会破坏**可行顶标**这个前提，因此
     $$
     d=\min\{L_x+L_y-W_{x→y}|x在交替路\and y不在交替路\}
     $$

   **注：**相等子图中顶标之和 $\sum L_x + \sum L_y$ 越来越小，以此来加入更多的边使得边权和更大。

7. [代码模板](https://github.com/McGinn7/Template-for-ICPC/blob/master/template/km.cpp)

```c++
// find a perfect matching with maximum total cost
// in bipartite graph G=(X,Y;E).
// if you want the total cost minimum, just negative
// cost.
template<typename T>
struct KuhnMunkres {
	int n, m, vis[N], pre[N], mth[N];
	T INF, Lx[N], Ly[N], slack[N], w[N][N];

	void init(int _n, int _m, T _INF) {
		n = _n, m = _m, INF = _INF;
		// modify weight of non-edge if need
		rep(i, 0, n) rep(j, 0, m) w[i][j] = -INF;
	}

	inline void addEdge(int u, int v, int wgh) {
		w[u][v] = wgh;
	}

	void find(int v) {
		rep(i, 0, m + 1) vis[i] = 0, slack[i] = INF;
		int u = m;
		for (mth[u] = v; mth[u] != -1; u = v) {
			vis[u] = 1;
			T d = INF;
			rep(i, 0, m) if (!vis[i]) {
				T att = Lx[mth[u]] + Ly[i] - w[mth[u]][i];
				// maintain feasible solution
				if (att < slack[i]) slack[i] = att, pre[i] = u;
				// find minimum d
				if (slack[i] < d) d = slack[v = i];
			}
			rep(i, 0, m + 1)
				if (vis[i]) Lx[mth[i]] -= d, Ly[i] += d;
				else slack[i] -= d;
		}
		// find a augmented path
		for(; u != m; u = pre[u]) mth[u] = mth[pre[u]];
	}

	T solve() {
		bool iswap = false;
		if (n > m) {
			rep(i, 0, n) rep(j, 0, min(i, m))
				swap(w[i][j], w[j][i]);
			iswap = true, swap(n, m);
		}
		fill(Lx, Lx + n, 0);
		fill(Ly, Ly + m, 0);
		fill(mth, mth + m, -1);
		rep(i, 0, n) find(i);

		T ret = 0;
		rep(i, 0, n) ret += Lx[i];
		rep(i, 0, m) ret += Ly[i];
		if (iswap)
			per(i, 0, max(n, m)) swap(Lx[i], Ly[i]);
		return ret;
	}
};
```

