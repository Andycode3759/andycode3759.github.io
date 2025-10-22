---
title: 考试总结 - 2022 牛客 OI 赛前集训营-提高组（第四场）（vp）
date: 2023-10-31 20:54:22 +0800
categories: [考试总结]
tags: [牛客]
math: true
---

许昊然在分享自己的考试经验时说：不要让自己长时间陷入某项工作（思考、调试）中。本次考试贯彻了这一思想理念，保证没有连续对着一道题思考/调程序超过半小时，压力果然小了很多，心态也不一样。在没有拿下任何一道题的情况下 ~~（顺便吐槽出题人都不放个签到题）~~ 还圆满完成了 200+ 的任务，可见最终效果也非常显著。

与此形成鲜明对比的是，CSP-S 2023 中 T3 只拿了可怜的 10 分，考后自己订正只花了不到 2 小时的时间就从零开始调到满分。根本原因还是做题环境和心态不一样。

**考试中有时学会给自己减轻压力，才能更大地释放出潜力。** 当然，考完后不会做的题也还是得及时补上的。

## `A` - 博弈

每一点都想到了（推导获胜条件 + 减不合法方案 + 离散化 + XOR-Hash），就是没想到维护根节点到每个节点的和。当时在考虑怎么处理所有点对哈希的树上前缀差的时候考虑了好久，因为似乎不能把线性的做法照搬，链剖又好像不太对，思来想去最后还是用了 $O(n^2)$ 找 LCA 暴力统计。

另外，如果是 `mt19937` 随机大数哈希的话，32 位的空间太小，很容易撞（只有 42 分），改用 64 位的 `mt19937_64` 可以缓解这一问题。

```c++
using ll = long long;
const int MAXN = 500005;

mt19937_64 rng(time(0)); // 用常规的mt19937就挂了

struct Edge
{
    Edge(int _t = 0, int _i = 0) : to(_t), idx(_i) {}
    int to, idx;
};

int n;
vector<Edge> G[MAXN];
int val[MAXN];
vector<int> V;
int tot;

ll hsh[MAXN];
unordered_map<ll, int> cnt;

void dfs(int fa, int u, ll sum)
{
    cnt[sum]++;
    for (Edge e : G[u])
    {
        int v = e.to;
        if (v == fa)
            continue;
        // 把从根到每个节点的hash sum出现次数统计出来
        dfs(u, v, sum ^ hsh[val[e.idx]]);
    }
}

int main()
{
    int T;
    scanf("%d", &T);
    while (T--)
    {
        for (int i = 1; i <= n; i++)
            G[i].clear();
        V.clear();
        cnt.clear();

        scanf("%d", &n);
        for (int i = 1; i <= n - 1; i++)
        {
            int u, v, w;
            scanf("%d %d %d", &u, &v, &w);
            G[u].push_back(Edge(v, i));
            G[v].push_back(Edge(u, i));
            val[i] = w;
            V.push_back(w);
        }
        // 离散化+哈希生成
        sort(V.begin(), V.end());
        tot = unique(V.begin(), V.end()) - V.begin();
        V.resize(tot);
        for (int i = 1; i <= n - 1; i++)
            val[i] = lower_bound(V.begin(), V.end(), val[i]) - V.begin() + 1;
        for (int i = 1; i <= tot; i++)
            hsh[i] = rng();
        // 统计答案
        ll ans = (ll)n * (n - 1) / 2;
        dfs(0, 1, 0);
        for (auto p : cnt)
            ans -= (ll)p.second * (p.second - 1) / 2;
        printf("%lld\n", ans);
    }
    return 0;
}
```

## `B` - 跳跃

### Subtask 1：50%

$O(n^2 k)$ 的 DP 还是比较好想的：设 $dp(x,i)$ 为方向为 $x \in \{0,1\}$，当前位置为 $i$ 时的最大得分，枚举起点和终点进行转移，共转移 $k$ 次。刚好可以利用 $x$ 那一维滚动数组。

```c++
ll dp[2][MAXN];
inline void solve1()
{
    for (int i = 1; i <= n; i++)
        dp[0][i] = dp[1][i] = -INF;
    dp[0][1] = 0;
    for (int i = 1; i <= k; i++)
    {
        int cur = i & 1, prev = cur ^ 1;
        for (int j = 1; j <= n; j++)
        {
            if (cur) // right
            {
                for (int t = j; t <= n; t++)
                {
                    if (dp[prev][j] + sum[t] - sum[j - 1] >= 0)
                        dp[cur][t] = max(dp[cur][t], dp[prev][j] + sum[t] - sum[j - 1]);
                }
            }
            else // left
            {
                for (int t = j; t >= 1; t--)
                {
                    if (dp[prev][j] + sum[j] - sum[t - 1] >= 0)
                        dp[cur][t] = max(dp[cur][t], dp[prev][j] + sum[j] - sum[t - 1]);
                }
            }
        }
    }
    int cur = k & 1;
    ll ans = 0;
    for (int i = 1; i <= n; i++)
        ans = max(ans, dp[cur][i]);
    printf("%lld\n", ans);
}
```

### Subtask 2：???

注意到 $k \leq 10^9$，肯定不能一直转移 $k$ 次。手玩一下就会发现，似乎从某个时间点开始，DP 转移只会发生在区间和最大的一段区间的两端，即反复横跳。可以把这个区间和最大的区域求出来（都 $n \leq 1000$ 了，$O(n^2)$ 找也问题不大），在 DP 时如果发现最大值开始在这两端间转移了，就直接退出，剩下多少次就加上多少倍的区间和。

```c++
int ml, mr;
ll mxSum;
inline void solve2()
{
    mxSum = -INF;
    for (int l = 1; l <= n; l++)
    {
        for (int r = l; r <= n; r++)
        {
            if (sum[r] - sum[l - 1] > mxSum)
            {
                mxSum = sum[r] - sum[l - 1];
                ml = l, mr = r;
            }
        }
    }

    for (int i = 1; i <= n; i++)
        dp[0][i] = dp[1][i] = -INF;
    dp[0][1] = 0;
    int lazy = 0;
    ll ans = 0;
    for (int i = 1; i <= k; i++)
    {
        int cur = i & 1, prev = cur ^ 1;
        for (int j = 1; j <= n; j++)
        {
            if (cur) // right
            {
                ll mxDp = -INF;
                int h = 0;
                for (int t = j; t <= n; t++)
                {
                    if (dp[prev][j] + sum[t] - sum[j - 1] >= 0)
                        dp[cur][t] = max(dp[cur][t], dp[prev][j] + sum[t] - sum[j - 1]);
                    if (dp[cur][t] > mxDp)
                        mxDp = dp[cur][t], h = t;
                }
                if (i < k && mxDp > 0 && j == ml && h == mr)
                {
                    lazy = i;
                    ans = mxDp;
                    break;
                }
            }
            else // left
            {
                ll mxDp = -INF;
                int h = 0;
                for (int t = j; t >= 1; t--)
                {
                    if (dp[prev][j] + sum[j] - sum[t - 1] >= 0)
                        dp[cur][t] = max(dp[cur][t], dp[prev][j] + sum[j] - sum[t - 1]);
                    if (dp[cur][t] > mxDp)
                        mxDp = dp[cur][t], h = t;
                }
                if (i < k && mxDp > 0 && j == mr && h == ml)
                {
                    lazy = i;
                    ans = mxDp;
                    break;
                }
            }
        }
        if (lazy != 0)
            break;
    }
    if (lazy == 0)
    {
        int cur = k & 1;
        for (int i = 1; i <= n; i++)
            ans = max(ans, dp[cur][i]);
    }
    else
    {
        ans += (k - lazy) * mxSum;
    }
    printf("%lld\n", ans);
}
```

考试时对拍发现这个做法并不是永远正确，偶尔会有答案错误，所以加了一点奇奇怪怪的条件判断（比如 `i < k`，`if (lazy == 0)` 等等）来卡答案。但是无论如何，配合切 Subtask 可以骗得 90 分的好成绩。（Edit: 挂掉的一个点是因为 TLE）

[@Wilson_Lee](https://www.luogu.com.cn/user/513900) 指出 $k$ 的转移似乎最多不会超过 $5000$ 次（[ref](https://blog.nowcoder.net/n/2ed7fa2120ea4fe7b08dc9e5f66a6dfd)），所以可以转移 $5000$ 次后直接进入最后阶段的反复横跳。这当然不是永远成立的，只能怪出题人没造好数据。

### Subtask 3：100%

其实反复横跳的思想已经非常接近正解了，只不过需要把目光放长远，不要仅限于最后一个阶段。分析发现，其实每一次跳跃本质上都是在一个目前最优的区间里反复横跳。那为什么不是所有的 $k$ 次跳跃都是直接在最大和的区间两端跳呢？因为有限制：分数时刻都不能低于 $0$。这样就会导致某些时刻的某些区域是到达不了的，因为到达最大和的区间两端可能需要迈过一段扣分的“门槛”。

这时需要换一个 DP 的切入点。

## `C` - 大陆

### Subtask 1：???

一道构造题，不过条件很松。

考试时想了一个非常不正确的贪心做法。分析一下题目发现，所有的省份肯定会连成一个连通块，而这个省会要么在连通块内部，要么在邻近连通块的某一个点上。尝试简化条件，假设每一个省都是一棵子树，子树的树根是它的省会。初始时所有点都是一个省，省会在树根上，显然会超出城市大小限制，那么就对所有子树扫一遍，有子树大小大于 $B$ 的就让它独立成省、减轻父亲负担，然后递归 + 循环。当然，这样做不仅可以很轻松地 Hack，还很容易卡死循环（父亲省大于 $3B$ 但没有子树的大小达到了 $B$，比如菊花图），不过依然可以骗得 42 分的好成绩。

```c++
int n, b;
vector<int> G[MAXN];
int siz[MAXN];

int rt[MAXN];
int bel[MAXN], cnt[MAXN];
int tot = 1;
// 算子树大小
void dfsSiz(int fa, int u)
{
    siz[u] = 1;
    for (int v : G[u])
    {
        if (v == fa)
            continue;
        dfsSiz(u, v);
        siz[u] += siz[v];
    }
}
// 更新省和点的相关信息
void dfsColor(int fa, int u, int p)
{
    cnt[bel[u]]--;
    bel[u] = p;
    cnt[p]++;
    for (int v : G[u])
    {
        if (v == fa)
            continue;
        dfsColor(u, v, p);
    }
}
// 递归分裂省
void dfsSplit(int fa, int u)
{
    while (cnt[bel[u]] > 3 * b)
    {
        for (int v : G[u])
        {
            if (v == fa)
                continue;
            if (bel[v] == bel[u] && siz[v] >= b)
            {
                rt[++tot] = v;
                dfsColor(u, v, tot);
                dfsSplit(u, v);
                break;
            }
        }
    }
}

int main()
{
    scanf("%d %d", &n, &b);
    for (int i = 1; i <= n - 1; i++)
    {
        int u, v;
        scanf("%d %d", &u, &v);
        G[u].push_back(v);
        G[v].push_back(u);
    }
    // 初始时只有一个省，尝试分裂
    dfsSiz(0, 1);
    rt[1] = 1;
    dfsColor(0, 1, 1);
    dfsSplit(0, 1);

    printf("%d\n", tot);
    for (int i = 1; i <= n; i++)
        printf("%d ", bel[i]);
    printf("\n");
    for (int i = 1; i <= tot; i++)
        printf("%d ", rt[i]);
    printf("\n");

    return 0;
}
```

### Subtask 2：100%

官方题解给出的一种构造方法：初始时每个点都不属于任何省，对于每个节点记录一个点集 $S_x$，表示 $x$ 的子树里有哪些点还没有被分配到省。从下往上合并 $S$，父节点的 $S$ 是所有儿子节点的 $S$ 取并集。如果在合并儿子的过程中，碰到自己的 $S$ 大小刚好大于等于 $B$（其实可以不用考虑上界），就把现有的这些点组织成一个省，省会为 $x$，再继续合并（这一过程对于一个点来说可以发生多次），剩下的点再和自己一起上传给父亲。

实际实现时，由于内存限制很紧，因此要把 $n$ 个集合压缩到一个栈空间内。以及最后 $1$ 号点会留下一些孤立的点，随便找一个儿子的省合并就好了。

```c++
int rt[MAXN];
int bel[MAXN], cnt[MAXN];
int tot = 0;

stack<int> S;
void dfs(int fa, int u)
{
    int sz = S.size();
    for (int v : G[u])
    {
        if (v == fa)
            continue;
        dfs(u, v);
        if (S.size() - sz >= b)
        {
            tot++;
            rt[tot] = u;
            cnt[tot] = S.size() - sz;
            while (S.size() > sz)
            {
                int x = S.top();
                S.pop();
                bel[x] = tot;
            }
        }
    }
    S.push(u);
}
```

## `D` - 排列

### Subtask 1：40%

暴力模拟循环右移 + 计算 LIS，出现大于等于 $3$ 的 LIS 长度就输出 `YES`。时间 $O(n^2 q)$。

```c++
int n, q;
int arr[MAXN];
int buf[MAXN];
int lis[MAXN];
inline bool check()
{
    for (int i = 1; i <= n; i++)
    {
        lis[i] = 1;
        for (int j = 1; j < i; j++)
        {
            if (arr[j] < arr[i])
                lis[i] = max(lis[i], lis[j] + 1);
        }
        if (lis[i] >= 3)
            return true;
    }
    return false;
}
inline void solve1()
{
    while (q--)
    {
        int l, r, k;
        scanf("%d %d %d", &l, &r, &k);
        int len = r - l + 1;
        for (int i = 0; i < len; i++)
            buf[i] = arr[i + l];
        for (int i = 0; i < len; i++)
            arr[l + (i + k) % len] = buf[i];
        if (check())
            printf("YES\n");
        else
            printf("NO\n");
    }
}
```
