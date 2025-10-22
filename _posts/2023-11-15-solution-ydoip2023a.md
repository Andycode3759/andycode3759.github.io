---
title: 题解 - YDOIp2023A 超正经的树上问题
date: 2023-11-15 16:43:19 +0800
categories: [题解]
tags: [云斗学院]
math: true
---

因为没有发现结论（所有合法节点都在一条链上），所以把这道树上问题打成了 RMQ 问题（

（或许可以顺便解释一下那些写线段树合并和 Treap 选手的新奇思路）

首先对树进行一次先序遍历，根据遍历顺序把节点编号拍成一条链，把这个链叫做 $DFN$ 数组。这样一颗子树就对应于 $DFN$ 数组上的一段区间。记以节点 $i$ 为根的子树对应的区间为 $[lb_i,rb_i]$。

这样问题就转化成了，给定一个由 $1$ 到 $n$ 的排列组成的数组，每个节点对应一段区间，求有哪些节点 $i$ 满足在 $[1,lb_i-1] \cup [rb_i+1,n]$ 区间中不存在两个差的绝对值小于等于 $k$ 的数。然后我们就来解这道题。

把与 $x$ 的差的绝对值小于等于 $k$ 的数称作 $x$ 的不好的数。稍微转化一下条件，一个满足条件的节点 $i$，可以保证对于任意数 $a_x(x \in [1,lb_i-1] \cup [rb_i+1,n])$，$a_x$ 的不好的数都落在区间 $[lb_i,rb_i]$ 里。那么我们可以对任意一个数 $a$ 都求出它不好的数的最小分布区间。这样如果对于一个节点 $i$ 满足所有在 $[1,lb_i-1] \cup [rb_i+1,n]$ 内的数的不好的数的分布区间的并集是 $[lb_i,rb_i]$ 的子集，那么这个 $i$ 就是合法的答案。~~（这长难句确实有点难理解...）~~

于是我们可以记录每一个数在 $DFN$ 数组上出现的位置 $pos_i$（其实根据约定，$pos$ 数组就是 $lb$），然后对 $pos$ 数组打一个 ST 表求区间最大/最小，就可以对于 $DFN$ 上的每一个位置的数求出它不好的数的最小分布区间。之后再根据 $DFN$ 数组的顺序，对不好的数的分布区间求前缀并集和后缀并集，就可以在 $O(1)$ 内判断一个节点是否符合条件。

时间复杂度：求 DFS 序 $O(n)$，打 ST 表 $O(n \log n)$，预处理各种信息 $O(n)$，处理答案 $O(n)$。总复杂度 $O(n \log n)$，瓶颈在 RMQ。

```c++
const int MAXN = 300005;
const int MAX_LOG = 19;
const int INF = (1 << 30) - 1;

int lg2[MAXN];
int n, k;
int fa[MAXN];
vector<int> G[MAXN];
int dfn[MAXN], lb[MAXN], rb[MAXN]; // lb=>pos
int cur = 0;

// 求dfn
void dfs(int u)
{
    dfn[++cur] = u;
    lb[u] = cur;
    for (int v : G[u])
    {
        dfs(v);
    }
    rb[u] = cur;
}

// ST表
int stMx[MAX_LOG][MAXN], stMn[MAX_LOG][MAXN];
inline void stInit()
{
    lg2[1] = 0;
    for (int i = 2; i < MAXN; i++)
        lg2[i] = lg2[i >> 1] + 1;
    for (int i = 1; i <= n; i++)
        stMx[0][i] = stMn[0][i] = lb[i];
    for (int lg = 1; lg < MAX_LOG; lg++)
    {
        for (int i = 1; i + (1 << (lg - 1)) <= n; i++)
        {
            stMx[lg][i] = max(stMx[lg - 1][i], stMx[lg - 1][i + (1 << (lg - 1))]);
            stMn[lg][i] = min(stMn[lg - 1][i], stMn[lg - 1][i + (1 << (lg - 1))]);
        }
    }
}
inline int stQueryMx(int l, int r)
{
    l = max(l, 1);
    r = min(r, n);
    if (l > r)
        return -1;
    int lg = lg2[r - l + 1];
    return max(stMx[lg][l], stMx[lg][r - (1 << lg) + 1]);
}
inline int stQueryMn(int l, int r)
{
    l = max(l, 1);
    r = min(r, n);
    if (l > r)
        return INF;
    int lg = lg2[r - l + 1];
    return min(stMn[lg][l], stMn[lg][r - (1 << lg) + 1]);
}

// 区间并集
inline pair<int, int> segMax(pair<int, int> a, pair<int, int> b)
{
    return {min(a.first, b.first), max(a.second, b.second)};
}
// 判断子区间
inline bool isSubSeg(pair<int, int> a, pair<int, int> b)
{
    return a.first >= b.first && a.second <= b.second;
}

// nty[i] 表示dfs序为i的数的不好的数的分布区间
pair<int, int> nty[MAXN];
// nty在dfs序上的前缀交集/后缀交集
pair<int, int> mxr[MAXN], mxl[MAXN];

int root;
int main()
{
    scanf("%d %d", &n, &k);
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", &fa[i]);
        if (fa[i] == 0)
            root = i;
        G[fa[i]].push_back(i);
    }
    dfs(root);

    // 预处理
    stInit();
    for (int i = 1; i <= n; i++)
    {
        int x = dfn[i];
        nty[i] = {min(stQueryMn(x - k, x - 1), stQueryMn(x + 1, x + k)), max(stQueryMx(x - k, x - 1), stQueryMx(x + 1, x + k))};
    }
    mxr[1] = nty[1];
    for (int i = 2; i <= n; i++)
    {
        mxr[i] = segMax(nty[i], mxr[i - 1]);
    }
    mxl[n] = nty[n];
    for (int i = n - 1; i >= 1; i--)
    {
        mxl[i] = segMax(nty[i], mxl[i + 1]);
    }

    // 直接出答案
    for (int i = 1; i <= n; i++)
    {
        int x = lb[i] - 1, y = rb[i] + 1;
        if ((x == 0 || isSubSeg(mxr[x], {lb[i], rb[i]})) && (y > n || isSubSeg(mxl[y], {lb[i], rb[i]})))
            printf("%d ", i);
    }
    printf("\n");
    return 0;
}
```
