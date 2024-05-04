---
title: AhoCorasick Algorithm
date: 2019-05-31 10:23:33
tags:
- AhoCorasick
- AC自动机
---

##  简介

AhoCorasick 算法（简称 AC 自动机），解决多模式串的字符匹配问题，即给定若干个单词串 $W_i$，求在文本串 $T$ 中的出现位置。KMP 算法解决单模式串的字符匹配，所以 AC 自动机可认为是 KMP 算法的扩展。

## 预备知识

- 字典树（Trie）：树上任意节点到根的路径所构成的子串，记为 $S(u)$，都是某个插入串的**前缀**。
- KMP 算法：利用最长前后缀完成线性匹配。

## 核心思想

AhoCorasick 本质上与 KMP 算法是一样的，都是通过相同前后缀减少重复计算问题，只是数据结构不同。

对应于 KMP，AC 自动机需要构建**最长公共前后缀**（LCPS，Longese Common proper Prefix and Suffix），即对于树上任意节点 $u$，找出**最大树深**的节点 $v$，满足 $S(v)$ 是 $S(u)$ 的**真后缀**。因为字典树上的任意节点 $x$ 所表示的 $S(x)$ 都是前缀，故起名最长公共前后缀。

通常将节点 $v$ 记为 $fail(u)$，表示串 $S(u)$ 失配时的跳转节点，出于可读性的考虑，本文记为 $lcps(u)$。

$lcps(u)$  的**构建过程**：记节点 $u$ 的父节点为 $f(u)$，与其连边的字符为 $c$。若 $lcps(f(u))$ 存在 $c$ 的出边，则 $lcps(u)=trans(lcps(f(u)), c)$。否则继续找 $lcps(lcps(f(u)))$，直至找到或到达根节点（说明未找到）。

![](20190601092923.png)

<center>图 1. lcps(u) 的构建</center>

## 检索过程

假设已知 $lcps(u)$，且字典树节点 $u$ 与文本串 $T[0:i]$ 匹配，即 $T[i-|S(u)|+1:i] = S(u)$。继续匹配有两种情形：

1. $trans(u, T[i+1]) \neq NULL$，则匹配长度 +1。

2. $trans(u, T[i+1])=NULL$，与 KMP 类似，在字典树中找出最大深度（即最长前缀）的节点 $v$，满足 $S(v)$ 是 $S(u)$ 的真后缀，同时 $trans(v, T[i+1])\neq NULL$。

   令 $v = lcps(u)$，判断是否能匹配，否则继续判断 $lcps(lcps(u))$。

## 参考代码

```c++
const int N = 5e5 + 7; // sum(|Wi|)
const int E = 26; // character set size
struct AhoCorasick {
    int root = 0;
    int n, lcps[N], trans[N][E];
    int end[N]; // end[u] > 0 : S(u) = Wi
    int new_node() {
        memset(trans[n], 0, E * sizeof(int));    
        lcps[n] = root, end[n] = 0;
        return n++;
    }
    void init() {
        n = 0;
        root = new_node();
    }
    void insert(const char *str, int len) {
        int u = root; 
        for (int i = 0; i < len; ++i) {
            int c = str[i] - 'a';
            if (!trans[u][c])
                trans[u][c] = new_node();
            u = trans[u][c];
        }
        ++end[u];
    }
    void LCPS() {
        queue<int> Q({root});
        while (!Q.empty()) {
            int u = Q.front(); Q.pop();
            for (int c = 0; c < E; ++c) {
                if (trans[u][c]) {
                    int v = lcps[u];    
                    while (v != root && !trans[v][c])
                        v = lcps[v];
                    lcps[trans[u][c]] = u == root ? 
                        root : trans[v][c]; 
                    Q.push(trans[u][c]);
                } else
                    trans[u][c] = trans[lcps[u]][c];
            }
        }
    }
};
```



