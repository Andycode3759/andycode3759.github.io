---
title: 考试总结 - NOIP 2018 自测
date: 2023-09-25 17:34:04 +0800
categories: [考试总结]
tags: [noip]
math: true
---

这次检测暴露出来一个很严重的问题，就是**分析问题**的能力不够。按理来说 Day1 的题目应该比 Day2 简单，但是 Day2 的得分要高得多，原因是 Day2 T1 比较送分而且 T2 可以用暴力打表猜出结论拿下 65 分，剩下的带有思维性质的题目几乎没怎么拿分。同时很多细碎的 Subtask（5~10 分的那种）也没有引起足够重视，导致浪费了很多时间去推自己推不出来的分。

## `money` - 货币系统

如果考虑构造 $b$ 数组，或者判断能表达出来的金额是否等价，那这题就没法做了，因为“任意金额”可以取遍所有正整数而没有上限。正确的想法是，考虑面值之间能否互相合并，或者说删掉多余的面值。结论其实很简单：如果一个面值能被比它更小的面值给表示出来，那么这个面值就是多余的；再加上每种面值可以取无限多张，于是就变成了一个完全背包问题。

```c++
const int MAXN = 102;
const int MAXA = 25005;
int n, a[MAXN];
bitset<MAXA> sack;

inline void solve1()
{
    sack.reset();
    sack[0] = true;
    sort(a + 1, a + 1 + n);
    int ans = 0;
    for (int i = 1; i <= n; i++)
    {
        if (!sack[a[i]])
            ans++;
        for (int j = a[i]; j < MAXA; j++)
        {
            sack[j] = sack[j] | sack[j - a[i]];
        }
    }
    printf("%d\n", ans);
}
```

## `track` - 赛道修建

这题按理来说部分分是给的很足的，前三个 Subtask 加起来一共就有 55 分，但是考试时只拿到了第一个。当时忽略了 $a=1$（菊花图）这个 Subtask，转而去搞 $b=a+1$（链状）而且还没搞出来，事实证明前者应该是更简单的。

### Subtask 1：$m=1$（20%）

只修一条赛道，那就要求最长的可能的赛道长度，很明显就是求树的直径。二次 DFS 的写法是最简单的。

### Subtask 2：$a=1$（15%）

由于题目只规定“每条道路至多被一条赛道经过”，但是对路口能经过的次数没有限制，因此一条道路可以与**任意**的最多一条其他道路合并，形成一条更长的赛道。换句话说，所有的赛道覆盖的道路数要么是 $1$ 要么是 $2$。我们需要分类讨论：

- **Case 1**：$m \leq \lfloor { {n-1}\over{2} } \rfloor $：

  最多只会用到 $2m$ 条道路。考虑贪心，既然要让长度最小的赛道长度最大，那么就要取最小的 $2m$ 条道路，让它们两两拼接。配对方式很简单，每次取长度最小和最大的两条道路拼成一条赛道即可。（注意是**前 $2m$ 小**的道路中的最小/最大，不是所有 $n-1$ 条！）

- **Case 2**：$m > \lfloor { {n-1}\over{2} } \rfloor $：

  $n-1$ 条道路不够两两配对了，那就一定会有多余的道路。依然是贪心，还是需要让长度最小的那一批道路两两拼接，至于后面的一批长度大的道路，可以放心地让它们变成光棍。拼接方式和 Case 1 里的一致。

综合两种情况，可以得到一种统一的解法：把所有边长从大到小排序，如果 $n-1 < 2m$ 那就在列表末尾补 $2m-n+1$ 个 $0$（相当于列表中至少要有 $2m$ 个元素，此时前 $2m-n+1$ 个元素就变成了光棍）。然后令 $l_1$ 和 $l_{2m}$ 配对，$l_2$ 和 $l_{2m-1}$ 配对……$l_m$ 和 $l_{m+1}$ 配对，再取所有配对的最小值。更加形式化的说，求 $\min_{i=1}^{m}{l_{i}+l_{2m-i+1} }$ 的值即可。

### Subtask 3：$b=a+1$（20%）

图变成了一条链。根据贪心可以知道，最优解一定会覆盖所有的道路，那么问题就转化成了把一个数组切成 $m$ 段，求最小的子段和的最大值。很明显不能够简单地直接贪心了，既然是求最小值最大，可以考虑二分答案+检验。

假设能够求出在子段和不小于 $k$ 的情况下最多能够把数组切成 $c$ 段，如果 $c \geq m$，那么这个 $k$ 就是合法的。显然对于越小的 $k$，实现难度是越低的，即存在单调性；实现这一检查也不难，直接 $O(n)$ 贪心地划分即可。注意需要先把边权的线性排列求出来，放在一个单独的数组里。

```c++
int len2[MAXN];
void dfsLine(int fa, int u)
{
    for (int i = 0; i < G[u].size(); i++)
    {
        int v = G[u][i].p, idx = G[u][i].idx;
        if (v == fa)
            continue;
        len2[u] = len[idx];
        dfsLine(u, v);
    }
}
bool check(int k)
{
    int c = 0, sum = 0;
    for (int i = 1; i <= n - 1; i++)
    {
        sum += len2[i];
        if (sum >= k)
            c++, sum = 0;
    }
    return c >= m;
}
inline void solve3()
{
    dfsLine(0, 1);
    int r = 0;
    for (int i = 1; i <= n - 1; i++)
        r += len2[i];
    int l = 1, mid = (l + r) >> 1, ans = 1;
    while (l <= r)
    {
        mid = (l + r) >> 1;
        if (check(mid))
        {
            ans = mid;
            l = mid + 1;
        }
        else
        {
            r = mid - 1;
        }
    }
    printf("%d\n", ans);
}
```

### Subtask 4：AC（100%）

其实就是上述三个 Subtask 的合体。首先随便选一个根节点，把无根树变成一棵有根树。主框架依然考虑二分答案+检验，假设能够求出最短赛道长度不小于 $k$ 的情况下最多能建 $c$ 条赛道，如果 $c \geq m$，那么这个 $k$ 就是合法的。根据链状做法的检验思路，从叶子节点处开始，向上累加边权。说地形象一点，就是从所有叶子节点开始往上修赛道。那么赛道修到一半，一定会在 LCA 的地方相遇，应该如何解决冲突呢？换句话说，一个节点 $d$ 可能会收到自己多个儿子上传上来的边权和，记作可重集 $S$，我们需要计算它对 $c$ 能产生多少贡献，以及 $d$ 给父节点 $f$ 上传的边权和是多少。把 $S$ 按照 $k$ 划分成两个子集 $low$ 和 $up$，满足 $up=\{x \in S \vert x \geq k\}$，$low=\{x \in S \vert x < k\}$，把它们分开处理：

- **Case 1**：$up$：

  对于每一个 $x \in up$，它们都会对 $c$ 产生 $+1$ 的贡献，且不可能上传给 $f$。换句话说，这些赛道修到 $d$ 就已经长度超过 $k$ 了，于是就停在了 $d$ 节点。

- **Case 2**：$low$：

  这些赛道的处境比较尴尬，本身长度既没到 $k$，也不好决定把谁上传给 $f$。解决方法比较简单粗暴：以 $d$ 为转折点，把赛道两两配对拼接起来，最后从没有拼接的赛道中选出最长的那一根光棍上传给父亲。

  是不是联想到了 $a=1$（菊花图）的做法？唯一的不同之处在于，需要保证两条赛道拼接起来必须长度不小于 $k$。这也好办：每次选出最小的 $x$，查找到它的灵魂伴侣 $y$ 满足 $x+y \geq k$ 且 $y$ 尽量小，把 $x$ 和 $y$ 从 $S$ 中删去，重复这一过程直到 $x$ 找不到灵魂伴侣或者剩余的元素数量小于等于 $1$，此时把剩下的最大元素上传即可（如果没有元素剩余就上传 $0$）。

对了，是不是还有 Subtask 1 的思路没有用到？可以直接用它求一遍树的直径当作二分答案的上界。

```c++
int cnt;
int dfsBuild(int fa, int u, ll k)
{
    multiset<ll> S;
    for (auto e : G[u])
    {
        int v = e.p, l = len[e.idx];
        if (v == fa)
            continue;
        ll res = dfsBuild(u, v, k) + l;
        if (res >= k)   // 有超过k的道路直接停下来，累加贡献
            cnt++;
        else
            S.insert(res);
    }

    ll mx = 0;
    while (S.size() > 1)
    {
        auto x = S.begin();
        S.erase(x);
        auto y = S.lower_bound(k - *x);
        // 这里有个小坑点，全局的lower_bound函数
        // 比multiset自带的lower_bound成员函数要慢得多，所以写成
        // lower_bound(S.begin(),S.end(),k-*x)就超时了
        if (y == S.end())
        {
            mx = max(mx, *x);
        }
        else
        {
            S.erase(y);
            cnt++;
        }
    }
    if (!S.empty())
        mx = max(mx, *S.rbegin());
    return mx;
}
inline bool checkTree(ll k)
{
    cnt = 0;
    dfsBuild(0, 1, k);
    return cnt >= m;
}
inline void solve4()
{
    mxDis = -1;
    dis[1] = 0;
    dfs(0, 1);
    mxDis = -1;
    dis[mxp] = 0;
    dfs(0, mxp);
    ll l = 1, r = mxDis, mid = (l + r) >> 1, ans = 1;
    while (l <= r)
    {
        mid = (l + r) >> 1;
        if (checkTree(mid))
        {
            ans = mid;
            l = mid + 1;
        }
        else
        {
            r = mid - 1;
        }
    }
    printf("%lld\n", ans);
}
```

综上，考试时忽略掉 $a=1$ 这个部分分是很吃亏的，不仅是因为少拿了这 15 分，更是因为它是得到正解的最大的一个突破口。

## `travel` - 旅行

### Subtask 1：$m=n-1$（60%）

很明显图的结构是一棵树。分析一下题面发现，旅行过程中可以“沿着第一次访问该城市时经过的道路后退到上一个城市”：这不就是 DFS 中的回溯操作吗？于是问题就转化为了求一颗树的字典序最小的 DFS 序。解法很简单，给所有点的儿子排个序，从 $1$ 号点开始直接走下去就 OK 了。

```c++
int n, m;
struct Edge
{
    Edge(int _p, int _i) : p(_p), idx(_i) {}
    int p, idx;
    const bool operator<(const Edge &b) const
    {
        return p < b.p;
    }
};
vector<Edge> G[MAXN];

vector<int> seq;
void dfs(int fa, int u)
{
    seq.push_back(u);
    for (int i = 0; i < G[u].size(); i++)
    {
        int v = G[u][i].p;
        if (v == fa)
            continue;
        dfs(u, v);
    }
}
inline void solve1()
{
    for (int i = 1; i <= n; i++)
        sort(G[i].begin(), G[i].end());
    dfs(0, 1);
    for (int i = 0; i < seq.size(); i++)
        printf("%d ", seq[i]);
    printf("\n");
}
```

### Subtask 2：$m=n$（40%）

发现是一棵树加了一条边，那么图中就会出现一个环。换句话说，把环上的某一条边删掉，图就又变成了一棵树。一种很暴力的做法是，枚举环上的每一条边把它删掉，然后套一遍 Subtask 1 的做法，取字典序最小的答案。如果不会判环也不要紧，直接枚举所有的边，发现走出了环或者图不连通就直接退出去。时间应该是 $O(nm) \sim O(n^2)$ 的，$n$ 最大也只有 $5000$，卡卡常开个 O2 就能过了。

```c++
bool vis[MAXN];
inline void clearVis()
{
    for (int i = 1; i <= n; i++)
        vis[i] = false;
}
int ban;    // 当前被删掉的边的idx
vector<int> seq, ans;
bool dfs(int fa, int u) // 需要判环，所以需要返回一个值
{
    seq.push_back(u);
    vis[u] = true;
    for (int i = 0; i < G[u].size(); i++)
    {
        if (G[u][i].idx == ban)
            continue;
        int v = G[u][i].p;
        if (v == fa)
            continue;
        if (vis[v] || !dfs(u, v))
            return false;
    }
    return true;
}
inline void tryUpd()    // 尝试比较字典序，更新目前最优解
{
    if (ans.size() == 0)
    {
        ans = seq;
        return;
    }
    bool flag = false;
    for (int i = 0; i < n; i++)
    {
        if (seq[i] < ans[i])
        {
            flag = true;
            break;
        }
        if (seq[i] > ans[i])
            break;
    }
    if (flag)
        ans = seq;
}
inline void solve2()
{
    for (int i = 1; i <= n; i++)
        sort(G[i].begin(), G[i].end());
    for (int i = 1; i <= m; i++)
    {
        seq.clear();
        clearVis();
        ban = i;
        if (!dfs(0, 1)) // 走出环
            continue;
        if (seq.size() < n) // 图不连通
            continue;
        tryUpd();
    }
    for (int i = 0; i < ans.size(); i++)
        printf("%d ", ans[i]);
    printf("\n");
}
```

### Subtask Extra：$n \leq 5 \times 10^5$

> 见[洛谷 P5049『旅行 加强版』](https://www.luogu.com.cn/problem/P5049)。

那万一 $n$ 很大呢？显然不能再枚举边了，可以考虑直接在带环图上走，在合适的时候选择回溯，这样就跳过了一条边。[未完待续]

## `game` - 填数游戏

### Subtask 1：$n,m \leq 3$（20%）

我会 DFS！先枚举所有的填数方案，再按字典序从小到大枚举 $w$（其本质就是一个 $n-1$ 个 $\texttt{D}$ 和 $m-1$ 个 $\texttt{R}$ 构成的排列），检验得到的 $s$ 是否是单调不上升的。如何按字典序从小到大枚举排列呢？其实 `next_permutation` 本身的定义就是这个功能。

二维状态就没必要用二进制压缩了，还要多个编码解码的过程，直接 DFS 更加清晰。时间复杂度为 $O(2^{mn} \cdot { {(m+n)}!\over{m!n!} })$。

```c++
int n, m, ans, s[5][5], w[20], p;
inline void doState()   // 检验方案
{
    for (int i = 1; i <= n - 1; i++)
        w[i] = 0;
    for (int i = n; i <= n + m - 2; i++)
        w[i] = 1;
    int lst = INF;
    bool flag = false;
    do
    {
        p = s[0][0];
        int x = 0, y = 0;
        for (int i = 1; i <= n + m - 2; i++)
        {
            if (w[i] == 0)
                x++;
            if (w[i] == 1)
                y++;
            p = (p << 1) | s[x][y];
        }
        if (p > lst)
        {
            flag = true;
            break;
        }
        lst = p;
    } while (next_permutation(w + 1, w - 1 + n + m));
    if (!flag)
        ans++;
}
void dfsState(int x, int y) // 枚举填数方案
{
    if (x == n)
    {
        doState();
        return;
    }
    int nx = x, ny = y + 1;
    if (ny == m)
        nx++, ny = 0;
    s[x][y] = 0;
    dfsState(nx, ny);
    s[x][y] = 1;
    dfsState(nx, ny);
}
inline void solve1()
{
    dfsState(0, 0);
    printf("%d\n", ans);
}
```

### Subtask 2：$n \leq 3,m \leq 10^6$（35%）

$n$ 这么小，可以考虑按 $n$ 的情况分类讨论。分析一下暴力的时间复杂度，$n \leq 3,m \leq 8$ 的时候都是可以做的，打个表就能发现下列结论：

- $n=1$：答案为 $2^m$；
- $n=2$：答案为 $4 \cdot 3^{m-1}$；
- $n=3$：答案为 $112 \cdot 3^{m-3}$。

后两个结论在 $m \leq 3$ 的时候并不正确，但是不要紧，切 Subtask 的时候交给暴力就可以了。于是写个快速幂就能稳拿 65 分。

## `defense` - 保卫王国

同 `track`，这题部分分也给的很足，就算是暴力也能得不少分，拿 0 分属实不应该。

### Subtask 1：暴力（44%）

**审题很重要。**“由道路直接连接的两座城市中至少要有一座城市驻扎军队”与“每座城市至少要有一个相邻城市驻扎军队”两句话意思是不一样的。如果看成后面的意思那就没法搞了。

其实暴力很好打，就是[这道题](https://www.luogu.com.cn/problem/P1352)一模一样的模型，加了个初状态赋值罢了。

```c++
int n, m;
char type[5];
int val[MAXN];
vector<int> G[MAXN];

int a, x, b, y;
ll dp[2][MAXN];
void dfsDP(int fa, int u)
{
    dp[1][u] += val[u];
    for (int v : G[u])
    {
        if (v == fa)
            continue;
        dfsDP(u, v);
        dp[0][u] += dp[1][v];
        dp[1][u] += min(dp[0][v], dp[1][v]);
    }
}
inline void solve1()
{
    while (m--)
    {
        memset(dp, 0, sizeof dp);
        scanf("%d %d %d %d", &a, &x, &b, &y);
        dp[x ^ 1][a] = INF, dp[y ^ 1][b] = INF;
        dfsDP(0, 1);
        ll res = min(dp[0][1], dp[1][1]);
        printf("%lld\n", res >= INF ? -1 : res);
    }
}
```

### Subtask 2：`type=X1`（$a=1,x=1$）（+24%）

显然不能对所有询问都跑一遍 DFS，需要预处理出答案。既然已经固定了一个端点 $a=1$，那么可以先利用这个条件跑一遍 `dfsDP` 把所有子树对应的答案求出来。此时，$dp(y,b)$ 就是 $b$ 及其子树对答案的贡献。那么如何计算剩下部分呢？

反其道而行之，设 $dp_2(x,u)$ 表示节点 $u$ 的状态为 $x$ 时除去 $u$ 及其子树的最小费用。那么 $dp(y,b)+dp_2(y,b)$ 就是要求的答案。

求 $dp_2$ 和求 $dp$ 的过程大致是一样的，只不过由依赖儿子节点换成了依赖父节点，求 $dp_2(x,u)$ 的过程中肯定要从 $dp_2(x,fa)$ 转移过来，$fa$ 为 $u$ 的父节点。至于剩下的部分，就是 $fa$ 去掉儿子 $u$ 的其他儿子们（或者说 $u$ 的兄弟）对答案的贡献了，记作 $brt(x)$，可以根据 $dp$ 的值反推出来。之后就和正常 $dp$ 的过程一样，$x=1$ 那父节点可选可不选，取最小值；$x=0$ 那父节点必须选。

```c++
ll dp2[2][MAXN]; //[i][j] => j的状态为i时除去j及其子树的费用
void dfsOut(int fa, int u)
{
    if (u > 1)
    {
        ll brt0 = dp[0][fa] - dp[1][u];
        ll brt1 = dp[1][fa] - min(dp[0][u], dp[1][u]);
        dp2[0][u] = dp2[1][fa] + brt1;
        dp2[1][u] = min(dp2[0][fa] + brt0, dp2[1][fa] + brt1);
    }
    for (int v : G[u])
    {
        if (v == fa)
            continue;
        dfsOut(u, v);
    }
}
inline void solve2()
{
    dp[0][1] = INF;
    dfsDP(0, 1);
    dfsOut(0, 1);
    while (m--)
    {
        scanf("%d %d %d %d", &a, &x, &b, &y);
        printf("%lld\n", dp[y][b] + dp2[y][b]);
    }
}
```
