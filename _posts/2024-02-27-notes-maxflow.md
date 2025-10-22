---
title: 学习笔记 - 网络最大流
date: 2024-02-27 22:29:15 +0800
categories: [学习笔记]
tags: [网络流]
math: true
---

## 网络流模型

**网络（Network）** 是一张有一个入度为 $0$ 的 **源点（Source）** 和一个出度为 $0$ 的 **汇点（Sink）** 的带边权的有向图。这张图可以不必是 DAG（可以带环），但源点到汇点必须联通。

网络上的边权又称为边的 **容量（Capacity）**。我们把边想象成水管，边权表示这条水管可以流经的最大水流量。**最大流（Max Flow）** 问题是指，通过这张网络上的水管，最多可以把多少流量的水从源点输送到汇点。

形式化地，我们要给每条边分配一个 **流量（Flow）** 值，使得：

- 流量值不超过这条边的容量；
- 源点出边的流量总和等于汇点入边的流量总和，且越大越好；
- 对于非源/汇点的点，入边的流量等于出边的流量。

## 朴素算法

下文记源点为 $s$，汇点为 $t$。

我们先尝试用最朴素的方法来解决这个问题。在网络上找一条从 $s$ 到 $t$ 的路径，往这条路径里输送尽可能多的水（即让这条路径上每条边的流量都等于容量最小边的容量），然后再找另一条路径重复操作，直到 $s$ 和 $t$ 之间不存在还能增加流量的路径。

为了方便算法的实现，我们再定义一个 **余量（Residual）**，对于每条边都有「余量=容量-流量」。

把原图的边权视作余量，即这条边还能流多少水。实现一下我们的想法：

1. 在图上随便找一条从 $s$ 到 $t$ 的路径；
2. 记录这条路径上的最小余量 $r$；
3. 往这条路径里输入流量为 $r$ 的水，即所有边的余量减去 $r$，最大流答案加上 $r$；
4. 显然如果有一条边的余量变成了 $0$ 那它就不能再输送更多水了，这条边不再发挥作用，可以把它从图上删去；
5. 重复这一流程，直到 $s$ 和 $t$ 不再联通。

下面是用这个算法实现的 [洛谷 P3376](https://www.luogu.com.cn/problem/P3376) 的程序：

```c++
#include <vector>
#include <bitset>
#include <cstdio>
#include <unordered_set>
using namespace std;
using ll = long long;
const int MAXN = 203;
const ll INF = (1LL << 60) - 1;

int n, m, s, t;
unordered_set<int> G[MAXN]; // 因为要支持删边，所以用set存图更方便
ll resd[MAXN][MAXN];    // resd[u][v] 表示从u到v的余量

vector<int> path;
bitset<MAXN> vis;
bool findPath(int u, ll &bot)   // 尝试找一条u到t的路径，找不到返回false
{                               // bot记录“瓶颈边”，即路径上最小的余量
    path.push_back(u);
    if (u == t)
        return true;
    vis[u] = true;
    ll ob = bot;
    for (int v : G[u])
    {
        if (vis[v])
            continue;
        bot = min(ob, resd[u][v]);
        if (findPath(v, bot))
            return true;
    }
    path.pop_back();
    return false;
}

int main()
{
    scanf("%d %d %d %d", &n, &m, &s, &t);
    for (int i = 1; i <= m; i++)
    {
        int u, v, w;
        scanf("%d %d %d", &u, &v, &w);
        G[u].insert(v);
        resd[u][v] += w;
    }

    ll bot = INF;
    ll ans = 0;
    while (findPath(s, bot))
    {
        ans += bot;
        for (int i = 0; i < path.size() - 1; i++)
        {
            int u = path[i], v = path[i + 1];
            resd[u][v] -= bot;
            if (resd[u][v] == 0)
                G[u].erase(v);
        }

        bot = INF;
        vis.reset();
        path.clear();
    }
    printf("%lld\n", ans);
    return 0;
}
```

写出来提交后就会高兴地发现爆 WA 了。这一算法不一定是正确的。我们可以随手举个反例：

![wrong-case](https://ooo.0x0.ooo/2024/02/27/OyZXSg.png)

正确答案是 $100$，流量分配如下：

![wc-anwser](https://ooo.0x0.ooo/2024/02/27/OyZ3ns.png)

但是我们的朴素算法可能在一开始选择了 $1 \to 3 \to 2 \to 4$ 这条路径，导致在增加了 $99$ 的流量后图不联通了：

![wc-wrong](https://ooo.0x0.ooo/2024/02/27/OyZCDK.png)

## Ford–Fulkerson 增广

朴素算法之所以可能出错，是因为在找路径的时候是随意找的，我们不能保证每次都找到最优的那条路径。有没有方法修正呢？答案是肯定的：每次找到一条路径并增加流量后，再连一条反向的、容量等于最小余量的路径。这样做可以保证，当我们找到了一条比之前更优的路径时，可以有反悔的余地。

拿上面的反例来说，假如我们一开始选的是 $1 \to 3 \to 2 \to 4$ 这条路径，那么操作后就会产生一条反向路径（图中边上数字为余量）：

![reverse](https://ooo.0x0.ooo/2024/02/27/OyZw8a.png)

于是就会又多出一条 $1 \to 2 \to 3 \to 4$ 的路径，就可以把我们的答案给修正了。

代码实现也很简单，只要在上面朴素算法的基础上多一个加反向路径的操作：

```c++
for (int i = path.size() - 1; i >= 1; i--)
{
    int u = path[i], v = path[i - 1];
    G[u].insert(v);
    resd[u][v] += bot;
}
```

需要注意的是，如果产生了重边，需要把它们进行合并，余量相加。这也是为什么程序中使用二位数组来储存余量 + `set` 保存图形态，这样可以方便进行合并操作。

## Edmonds–Karp 算法

为什么上面一节要叫“增广”而这一节开始又叫“算法”了呢？因为 Ford–Fulkerson 单指「连反向路径」这一操作的思想，具体的算法实现可以有很多种类。上面的实现中我们还是随便选择从 $s$ 到 $t$ 的路径，我们暂且把它叫做「朴素 Ford–Fulkerson 算法」。

朴素 FF 算法虽然能保证正确性，但是效率很低。比如这张图：

![slow](https://ooo.0x0.ooo/2024/02/27/Oyv55L.png)

如果程序每次都选择 $2$ 和 $3$ 中间的那条边，那么流量每次都只会增加 $1$，一共需要选择 $198$ 次路径程序才会结束。由此可以看出朴素 FF 算法的时间复杂度和流量大小是相关的，为 $O(nf)$（记 $n$ 为点数，$m$ 为边数，$f$ 为最大流的值，下同）。

有没有改进的方法呢？思来想去，改进的方向只有一个，那就是选路径的流程。我们可以把随便选路径改成选择从 $s$ 到 $t$ 的 **最短路径**，这样就得到了 Edmonds–Karp 算法。

注意选择最短路径时我们需要忽略边权，即选择经过点数最少的那条路径。实现是简单的，直接用 BFS 即可：

```c++
vector<int> path;
bitset<MAXN> vis;
ll dis[MAXN];   // 记录路径长度，虽然没啥用
int prv[MAXN];  // 记录路径上的前一个点
bool findPath(ll &bot)
{
    for (int i = 1; i <= n; i++)
        dis[i] = INF, prv[i] = 0;
    dis[s] = 0;
    vis.reset();
    queue<int> Q;
    Q.push(s);
    vis[s] = true;
    while (!Q.empty())
    {
        int u = Q.front();
        Q.pop();
        for (int v : G[u])
        {
            if (!vis[v])
            {
                dis[v] = dis[u] + 1;
                prv[v] = u;
                Q.push(v);
                vis[v] = true;
            }
        }
    }
    // s与t不联通
    if (dis[t] >= INF)
        return false;
    // 通过prv把路径反向还原出来，再反转一下就得到了原路径
    vector<int> tmp;
    int x = t;
    do
    {
        if (prv[x] != 0)
            bot = min(bot, resd[prv[x]][x]);
        tmp.push_back(x);
        x = prv[x];
    } while (x != 0);
    for (auto it = tmp.rbegin(); it != tmp.rend(); it++)
        path.push_back(*it);
    return true;
}
```

Edmonds–Karp 算法的时间复杂度是 $O(nm^2)$。详细证明可以参阅 [OI Wiki](https://oi-wiki.org/graph/flow/max-flow/#%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6%E5%88%86%E6%9E%90_1)，这里就不展开了。

## Dinic 算法

Edmonds–Karp 算法在稀疏图上表现优秀，但是在 $m$ 与 $n^2$ 同阶的稠密图中依然很慢。Dinic 算法就是一种对稠密图友好的算法，它依然基于 Ford–Fulkerson 增广的思想。

考虑 EK 算法中 BFS 找最短路的过程，其时间最坏情况下是 $O(m)$ 的。Dinic 算法的思路与 EK 算法差不多，只不过在 BFS 的过程中不再遍历每条边，而是预先处理出一张 **分层图（Level Graph）**。我们把所有图上点按照离 $s$ 的最短距离来进行分层，再把除了从 $k$ 到 $k+1$ 层点之外的所有边都删掉，就得到了原图的分层图。不难发现分层图一定是一张 DAG。我们可以在分层图上随便找出一条 $s$ 到 $t$ 的路径，再按照 Ford–Fulkerson 增广的流程操作即可。

```c++
vector<int> path;
queue<int> Q;
bitset<MAXN> vis;
int dep[MAXN];  // 点所处的层数
bool dfs(int u) // 在分层图上找路径
{
    path.push_back(u);
    if (u == t)
        return true;
    for (int v : G[u])
    {
        if (dep[v] != dep[u] + 1) // 只能从k层到k+1层
            continue;
        if (dfs(v))
            return true;
    }
    path.pop_back();
    return false;
}
bool findPath()
{
    path.clear();
    queue<int>().swap(Q);
    vis.reset();
    Q.push(s);
    vis[s] = true;
    for (int i = 1; i <= n; i++)
        dep[i] = 0;
    dep[s] = 1;
    while (!Q.empty())
    {
        int u = Q.front();
        Q.pop();
        for (int v : G[u])
        {
            if (vis[v])
                continue;
            dep[v] = dep[u] + 1;
            vis[v] = true;
            Q.push(v);
        }
    }
    if (!vis[t])
        return false;
    return dfs(s);
}
```

Dinic 建分层图的过程是 $O(n)$ 的 ~~，但是如果你和我一样是个憨憨写成了标准 BFS 的话就又退化回了 $O(m)$，~~ 因此 Dinic 算法的总复杂度上界为 $O(n^2m)$。证明详见 [OI Wiki](https://oi-wiki.org/graph/flow/max-flow/#%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6%E5%88%86%E6%9E%90_2)。

## 参考资料

[「网络流问题基础」系列视频（By ShusenWang）](https://www.bilibili.com/video/BV1K64y1C7Do/)
