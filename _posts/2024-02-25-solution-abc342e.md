---
title: 题解 - ABC342E Last Train 最后的火车
date: 2024-02-25 16:24:28 +0800
categories: [题解]
tags: [atcoder, abc]
math: true
---

链接：[ABC342 E](https://atcoder.jp/contests/abc342/tasks/abc342_e)

## 题目大意

有 $n$ 个车站和 $m$ 条铁路，每条铁路有 6 个参数 $l,d,k,c,A,B$，表示这条铁路上的列车从 $A$ 站驶向 $B$ 站，从时刻 $l$ 开始每隔 $d$ 时间发一趟车，总共有 $k$ 趟，列车行驶需要耗费 $c$ 时间。对于每一个 $i = 1,2,\dots,n-1$，求从 $i$ 站到达 $n$ 站的最晚时间，如果无法到达输出 `Unreachable`。

## 解析

把时间看成路程，下文所说的“最长路”和“最短路”分别指最晚/最早到站时间。

直接算从 $i$ 到 $n$ 的最长路不仅慢，还无法顾及列车发车数量等限制，就算 DP 空间也开不下。考虑把“从 $i$ 到 $n$ 的最长路”转化为“从 $n$ 到 $i$ 的最短路”，把所有边（即铁路）的方向调转后，再把时间映射到负数。这样一来“从 $A$ 驶向 $B$，从 $l$ 时刻开始共发车 $k$ 趟，每趟间隔 $d$，行驶耗时 $c$”的铁路，就转化为了“从 $B$ 驶向 $A$，从 $-l-(k-1)d-c$ 时刻开始共发车 $k$ 趟，每趟间隔 $d$，行驶耗时 $c$” 的铁路。

这样转化满足局部最优性，因为从同一个点早出发一定比晚出发要更优（早来了可以等车，晚来了就不行），于是就可以套单源最短路算法来求解了，比如 Dijkstra。在处理出边的时候要注意选择这条铁路上 $dis_u$ 时刻及以后的第一趟列车，且如果 $dis_u$ 已经比最后一趟列车还晚了就要忽略这条边。

技能点：构造，单源最短路径。

## 参考代码

```c++
#include <utility>
#include <queue>
#include <cstdio>
#include <vector>
using namespace std;
using ll = long long;
const int MAXN = 200005;
const ll INF = (1LL << 60) - 1;

struct Edge
{
    Edge(ll _l, ll _d, ll _k, ll _c, int _dest)
    {
        l = _l;
        d = _d;
        k = _k;
        c = _c;
        dest = _dest;
    }
    ll l;     // 发车时间
    ll d;     // 发车间隔
    ll k;     // 车次数量
    ll c;     // 行驶时间
    int dest; // 目的地
};

struct DijNode
{
    DijNode(ll _d, int _u) : dis(_d), u(_u) {}
    ll dis;
    int u;
    const bool operator<(const DijNode &d) const
    {
        return dis > d.dis;
    }
};

int n, m;
vector<Edge> G[MAXN];

ll dis[MAXN];
priority_queue<DijNode> Q;
bool vis[MAXN];

int main()
{
    scanf("%d %d", &n, &m);
    for (int i = 1; i <= m; i++)
    {
        int l, d, k, c, a, b;
        scanf("%d %d %d %d %d %d", &l, &d, &k, &c, &a, &b);
        // 转化边
        G[b].push_back(Edge(-l - (ll)(k - 1) * d - c, d, k, c, a));
    }

    for (int i = 1; i <= n; i++)
        dis[i] = INF;
    dis[n] = -INF;
    Q.push(DijNode(0, n));
    while (!Q.empty())
    {
        DijNode t = Q.top();
        Q.pop();
        int u = t.u;
        if (vis[u])
            continue;
        vis[u] = true;
        for (Edge e : G[u])
        {
            ll nd = max(dis[u], e.l);
            // dis[u]已经比最后一趟车还晚了就忽略
            if (nd > e.l + (e.k - 1) * e.d)
                continue;
            // dis[u]及之后第一趟车到达目的地e.dest的时间
            nd = e.l + ((nd - e.l + e.d - 1) / e.d) * e.d + e.c;
            if (nd < dis[e.dest])
            {
                dis[e.dest] = nd;
                Q.push(DijNode(nd, e.dest));
            }
        }
    }
    for (int i = 1; i <= n - 1; i++)
    {
        if (dis[i] >= INF)
            printf("Unreachable\n");
        else
            printf("%lld\n", -dis[i]);
    }
    return 0;
}
```
