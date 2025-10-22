---
title: 题解 - ABC342F Black Jack 黑杰克
date: 2024-02-25 19:40:38 +0800
categories: [题解]
tags: [atcoder, abc]
math: true
---

链接：[ABC342 F](https://atcoder.jp/contests/abc342/tasks/abc342_f)

## 题目大意

有一个 $D$ 面的骰子，可以从 $1$ 到 $D$ 中等概率抽取一个整数。你和庄家玩一次游戏，流程如下：

- 你可以丢任意次骰子，且随时可以决定继续或停止丢骰子。把所有投到的点数相加，记作 $x$；
- 庄家以这样的策略丢骰子：记目前已经丢到的点数之和为 $y$，如果 $y<L$（$L$ 是给定的常数）则继续丢骰子，否则停止。

你能获胜当且仅当 $x \leq N$ 且满足下列两个条件之一：

1. $y > N$；
2. $x > y$。

给定 $N,L,D \leq 2 \times 10^5$，求你在最佳策略下获胜的概率。

## 解析

庄家的策略是固定的，可以先算出庄家停在某个点数上的概率，记作 $dp_i$。转移是简单的。

接着我们设 $win_i$ 表示目前点数之和为 $i$ 的情况下，最终获胜的总概率。显然对于 $i > N,win_i=0$。每一步有两个决策：停止或继续，取两个决策中获胜概率更高的一个。

- 如果继续，那么从 $i$ 转移到 $i+1 \sim i+1+d$ 是等概率的，即 $\sum \limits_{j=i+1}^{i+1+d}win_j/d$；
- 如果停止，那么胜率只与庄家的状态有关，即当前点数比庄家大或者庄家已经爆出 $N$，即 $\sum \limits_{j=n+1}^{\infty}dp_j+\sum \limits_{j=0}^{i-1}dp_j$，化简一下变成 $1-\sum \limits_{j=i}^{n}dp_j$（也即用 $1$ 减去庄家获胜的概率）。

最终答案就是 $win_0$。

代码实现方面需要注意：

- 计算 $dp$ 的时候要用数据结构加速（暴力加是 $O(nd)$ 肯定超时），可以差分树状数组，也可以线段树区间加 + 单点查询。
- 计算 $win$ 的时候要从后往前，同时维护一个后缀和。
- 注意对 $\leq 0$ 的下标的特殊处理。

技能点：概率论，数据结构优化 DP。

## 参考代码

```c++
#include <algorithm>
#include <cstdio>
using namespace std;
const int MAXN = 200005;

int n, l, d;
double dp[MAXN * 2], dpSum[MAXN * 2], win[2 * MAXN], winSum[2 * MAXN];

double fw[MAXN * 2];
inline void fwModify(int pos, double x)
{
    for (; pos < MAXN * 2; pos += (pos & (-pos)))
        fw[pos] += x;
}
inline double fwQuery(int pos)
{
    if (pos < 0)
        return 0;
    double res = fw[0];
    for (; pos > 0; pos -= (pos & (-pos)))
        res += fw[pos];
    return res;
}

int main()
{
    scanf("%d %d %d", &n, &l, &d);

    fwModify(1, 1.0 / d);
    fwModify(d + 1, -1.0 / d);

    for (int i = 1; i < l; i++)
    {
        double t = fwQuery(i);
        fwModify(i + 1, t / d);
        fwModify(i + 1 + d, -t / d);
    }
    for (int i = l; i <= l + d; i++)
        dp[i] = fwQuery(i);
    for (int i = l; i <= n; i++)
        dpSum[i] = dpSum[i - 1] + dp[i];

    for (int i = n; i >= 0; i--)
    {
        win[i] = max(i == 0 ? 1 - dpSum[n] : 1 - (dpSum[n] - dpSum[i - 1]), (winSum[i + 1] - winSum[i + d + 1]) / d);
        winSum[i] = winSum[i + 1] + win[i];
    }

    printf("%lf\n", win[0]);
    return 0;
}
```
