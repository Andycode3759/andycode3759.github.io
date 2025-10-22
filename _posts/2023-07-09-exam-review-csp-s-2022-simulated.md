---
title: 考试总结 - CPS-S 2022 模拟测试
date: 2023-07-09 19:32:05 +0800
categories: [考试总结]
tags: [csp-s]
math: true
---

## holiday - 假期计划

先解决 $k=0$ 的情况：由于 $N$ 比较小，可以直接用矩阵存图，然后用四层循环枚举四个景点求答案最大值即可。

```c++
long long ans = -1;
for (int d1 = 2; d1 <= n; d1++)
{
    for (int d2 = 2; d2 <= n; d2++)
    {
        if (d2 == d1)
            continue;
        for (int d3 = 2; d3 <= n; d3++)
        {
            if (d3 == d2 || d3 == d1)
                continue;
            for (int d4 = 2; d4 <= n; d4++)
            {
                if (d4 == d3 || d4 == d2 || d4 == d1)
                    continue;

                if (graph[1][d1] == 1 && graph[d1][d2] == 1 && graph[d2][d3] == 1 && graph[d3][d4] == 1 &&
                    graph[d4][1] == 1)
                {
                    ans = max(ans, score[d1] + score[d2] + score[d3] + score[d4]);
                }
            }
        }
    }
}
return ans;
```

接下来是 $k>0$ 的情况，先用 Floyd 求出任意两个地点之间最少转车几次可以到达（$N$ 比较小时速度比较快），然后把判断条件改成“转车次数是否在$k$次以内”即可。

```c++
for (int k = 1; k <= n; k++)
{
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= n; j++)
        {
            if (graph[i][k] + graph[k][j] < graph[i][j])
                graph[i][j] = graph[i][k] + graph[k][j];
        }
    }
}
long long ans = -1;
for (int d1 = 2; d1 <= n; d1++)
{
    for (int d2 = 2; d2 <= n; d2++)
    {
        if (d2 == d1)
            continue;
        for (int d3 = 2; d3 <= n; d3++)
        {
            if (d3 == d2 || d3 == d1)
                continue;
            for (int d4 = 2; d4 <= n; d4++)
            {
                if (d4 == d3 || d4 == d2 || d4 == d1)
                    continue;

                // 这里graph[a][b]的实际含义是转车次数+1
                if (graph[1][d1] <= k + 1 && graph[d1][d2] <= k + 1 && graph[d2][d3] <= k + 1 &&
                    graph[d3][d4] <= k + 1 && graph[d4][1] <= k + 1)
                {
                    ans = max(ans, score[d1] + score[d2] + score[d3] + score[d4]);
                }
            }
        }
    }
}
return ans;
```

上面的做法对于稍大的 $N$ 都会超时，因为枚举实在太费时间了。接下来把枚举改成搜索，可以多个 10 分左右：

```c++
long long ans = -1;
bool vis[MAXN];
void dfs(int cur, int step, long long sum)
{
    if (step > 4)
    {
        if (graph[cur][1] <= k + 1) // 此时走到了D，还要判断和1是否联通
            ans = max(ans, sum);
        return;
    }
    for (int i = 2; i <= n; i++)
    {
        if (vis[i] || graph[cur][i] > k + 1)
            continue;
        vis[i] = true;
        dfs(i, step + 1, sum + score[i]);
        vis[i] = false;
    }
}

int main()
{
    ...
    // Floyd
    for (int k = 1; k <= n; k++)
    {
        for (int i = 1; i <= n; i++)
        {
            for (int j = 1; j <= n; j++)
            {
                if (graph[i][k] + graph[k][j] < graph[i][j])
                    graph[i][j] = graph[i][k] + graph[k][j];
            }
        }
    }

    dfs(1, 1, 0);
    printf("%lld\n", ans);
}
```

更进一步，由于 $m$ 比较小，图整体来说是稀疏的，邻接矩阵存图比较浪费，可以改成 `vector` 存图。这时就会发现 Floyd 的速度开始拖后腿了，可以改成用 BFS 重新构图，把 $k$ 步之内能到达的点也连上边。注意 BFS 实现时有很多细节需要注意。

```c++
int n, m, k;
long long score[MAXN];
int graph[MAXN][MAXN];
vector<int> outs1[MAXN], outs2[MAXN];
// outs1是原图，graph是bfs时过渡用的邻接矩阵存的图，outs2是转化后的最终图

long long ans = -1;
bool vis[MAXN];

inline void dfs(int cur, int step, long long sum)
{
    if (step > 4)
    {
        if (graph[cur][1] <= k + 1)
            ans = max(ans, sum);
        return;
    }
    int len = outs2[cur].size();
    for (int i = 0; i < len; i++)
    {
        int v = outs2[cur][i];
        if (vis[v])
            continue;
        vis[v] = true;
        dfs(v, step + 1, sum + score[v]);
        vis[v] = false;
    }
}

inline void bfs(int st)
{
    queue<int> Q;
    Q.push(st);
    vis[st] = true;
    while (!Q.empty())
    {
        int t = Q.front();
        Q.pop();
        int len = outs1[t].size();
        for (int i = 0; i < len; i++)
        {
            int v = outs1[t][i];
            if (!vis[v])
            {
                graph[st][v] = graph[st][t] + 1;
                if (graph[st][v] < k + 1)
                {
                    Q.push(v);
                }
                vis[v] = true;
            }
        }
    }
}

int main()
{
    // 顾名思义，readInt和readLL是快读用的函数
    n = readInt();
    m = readInt();
    k = readInt();
    for (int i = 2; i <= n; i++)
    {
        score[i] = readLL();
    }
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= n; j++)
        {
            graph[i][j] = (i == j) ? 0 : INF;
        }
    }
    for (int i = 1; i <= m; i++)
    {
        int u = readInt(), v = readInt();
        outs1[u].push_back(v);
        outs1[v].push_back(u);
    }

    // 初始化并对每个点开始进行一次bfs
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= n; j++)
        {
            graph[i][j] = (i == j) ? 0 : INF;
            vis[j] = false;
        }
        bfs(i);
    }

    // 把邻接矩阵图转化为vector图
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= n; j++)
        {
            if (i != j && graph[i][j] <= k + 1)
                outs2[i].push_back(j);
        }
    }

    // 主算法依然是dfs
    for (int i = 2; i <= n; i++)
        vis[i] = false;
    vis[1] = true;
    dfs(1, 1, 0);

    printf("%lld\n", ans);
}
```

最大的那些点还是过不了，此时需要想到 DFS 中的一个剪枝操作。可以记录在第 $s$ 步走到第 $i$ 个点时已经得到的最大积分，如果目前的得分情况不如有记录的最好情况，那么可以直接剪枝。

```c++
long long bestScore[6][MAXN]; // 第step步走在i点上时，目前能够获得的最大得分

inline void dfs(int cur, int step, long long sum)
{
    if (step > 4)
    {
        if (graph[cur][1] <= k + 1)
            ans = max(ans, sum);
        return;
    }
    if (sum < bestScore[step][cur]) // 剪枝操作
        return;
    bestScore[step][cur] = sum; // 维护最大分数
    int len = outs2[cur].size();
    for (int i = 0; i < len; i++)
    {
        int v = outs2[cur][i];
        if (vis[v])
            continue;
        vis[v] = true;
        dfs(v, step + 1, sum + score[v]);
        vis[v] = false;
    }
}
```

最后再加上一点点卡常技巧，比如快读和内联函数之类的，就这样，在避开一切正解做法的情况下，成功地解决 ~~（卡过）~~ 了这道题。（Edit：过官方数据足够了，但洛谷上某个 Hack 数据点还是过不了）

- 有时不会打正解也不影响某道题拿高分甚至满分，可以尝试一步步优化+卡常，但同时要注意确保解法的正确性（或者至少正确率比较高）。
  - 具体地，枚举可以改成搜索，对于稀疏图可以把邻接矩阵改 `vector`，DFS 尽量剪枝，都能大大加快速度。

## game - 策略游戏

首先看特殊性质 1，两个数列中的数全为正数，那么 L 可以直接无脑选最大值、Q 无脑选最小值。区间最值可以用 ST 表预处理出来。

```c++
int getMinA(int l, int r)
{
    int lg = lg2[r - l + 1];
    return min(stMinA[lg][r], stMinA[lg][l + (1 << lg) - 1]);
}
int getMaxA(int l, int r)
{
    int lg = lg2[r - l + 1];
    return max(stMaxA[lg][r], stMaxA[lg][l + (1 << lg) - 1]);
}
int getMinB(int l, int r)
{
    int lg = lg2[r - l + 1];
    return min(stMinB[lg][r], stMinB[lg][l + (1 << lg) - 1]);
}
int getMaxB(int l, int r)
{
    int lg = lg2[r - l + 1];
    return max(stMaxB[lg][r], stMaxB[lg][l + (1 << lg) - 1]);
}
void solveSP1() // 解法主函数在这
{
    long long res = 0;
    for (int i = 1; i <= q; i++)
    {
        Round &r = rounds[i];
        int maxA = getMaxA(r.l1, r.r1);
        int minB = getMinB(r.l2, r.r2);
        res = 1LL * maxA * minB;
        printf("%lld\n", res);
    }
}
void initST()
{
    for (int lg = 1, len = (1 << lg); len <= n; lg++, len = (1 << lg))
    {
        for (int i = n; i > (1 << (lg - 1)); i--)
        {
            stMaxA[lg][i] = max(stMaxA[lg - 1][i], stMaxA[lg - 1][i - (1 << (lg - 1))]);
            stMinA[lg][i] = min(stMinA[lg - 1][i], stMinA[lg - 1][i - (1 << (lg - 1))]);
        }
    }
    for (int lg = 1, len = (1 << lg); len <= m; lg++, len = (1 << lg))
    {
        for (int i = m; i > (1 << (lg - 1)); i--)
        {
            stMaxB[lg][i] = max(stMaxB[lg - 1][i], stMaxB[lg - 1][i - (1 << (lg - 1))]);
            stMinB[lg][i] = min(stMinB[lg - 1][i], stMinB[lg - 1][i - (1 << (lg - 1))]);
        }
    }
}
```

再看特殊性质 2，两方中有一方莫得选择，那就看这个固定的数是正还是负。对于 L 来说，如果 Q 的数是正数就选最大值，如果是负数就选最小值；反之亦然，对于 Q 来说，如果 L 的数是正数就选最小值，是负数就选最大值。

```c++
void solveSP2()
{
    long long res = 0;
    for (int i = 1; i <= q; i++)
    {
        Round &r = rounds[i];
        if (r.l1 == r.r1)
        {
            if (arrA[r.l1] == 0)
                res = 0;
            else if (arrA[r.l1] > 0)
                res = 1LL * getMinB(r.l2, r.r2) * arrA[r.r1];
            else // arrA[r.l1] < 0
                res = 1LL * getMaxB(r.l2, r.r2) * arrA[r.r1];
        }
        else // r.l2 == r.r2
        {
            if (arrB[r.l2] == 0)
                res = 0;
            else if (arrB[r.l2] > 0)
                res = 1LL * getMaxA(r.l1, r.r1) * arrB[r.r2];
            else // arrA[r.l2] < 0
                res = 1LL * getMinA(r.l1, r.r1) * arrB[r.r2];
        }
        printf("%lld\n", res);
    }
}
```

最后就是一般情况，需要考虑两个人的具体策略。先考虑 Q 的策略，他无非只能从可选的数中要么选最大、要么选最小，因此结果主要取决于 L 的选择。把 L 的选择按正负分类讨论：

- 如果 L 要选一个正数，那么 Q 就会选最小的数，看 Q 手里最小的数（记为 $min_Q$）的正负：
  - 若 $min_Q \geq 0$，那么 L 选最大的数准没错；
  - 若 $min_Q < 0$，那么 L 得选一个绝对值最小的正数。
- 反之，如果 L 要选一个负数，那么 Q 就会选最大的数，看 Q 手里最大的数（记为 $max_Q$）的正负：
  - 若 $max_Q \leq 0$，那么 L 就选绝对值最大的负数（即最小的数）；
  - 若 $max_Q > 0$，那么 L 就需要选绝对值最小的负数。

```c++
inline void solveGeneral()
{
    for (int i = 1; i <= q; i++)
    {
        long long res = -INF_LL;
        Round &r = rounds[i];
        int minl = getMinA(r.l1, r.r1);
        int maxl = getMaxA(r.l1, r.r1);
        int minPosL = getMinPosA(r.l1, r.r1);
        int maxNegL = getMaxNegA(r.l1, r.r1);
        int minq = getMinB(r.l2, r.r2);
        int maxq = getMaxB(r.l2, r.r2);
        res = max(res, 1LL * maxl * (maxl >= 0 ? minq : maxq));
        res = max(res, 1LL * minl * (minl >= 0 ? minq : maxq));
        if (minPosL != INF)
            res = max(res, 1LL * minPosL * (minPosL >= 0 ? minq : maxq));
        if (maxNegL != -INF)
            res = max(res, 1LL * maxNegL * (maxNegL >= 0 ? minq : maxq));
        printf("%lld\n", res);
    }
}
```

这道题的难点主要在于，L 和 Q 的策略是不同的，并且有负数的出现，需要找到同时满足两人策略的最优情况。如果这道题改成直接求最大/最小，或者只有特殊性质的数据，难度直接降到普及组。此类题无非需要能够清楚地推理，把所有情况都分类讨论出来，再用代码实现。

最后的最后，考试时特殊性质 2 的解法有个地方粗心写错了，至少少拿了 20 分：

```c++
...
else // r.l2 == r.r2
{
    if (arrA[r.l2] == 0)    // 这里应该是arrB!!!
        res = 0;
    else if (arrA[r.l2] > 0)    // 这里也应该是arrB!!!
        res = 1LL * getMaxA(r.l1, r.r1) * arrB[r.r2];
    else // arrA[r.l2] < 0
        res = 1LL * getMinA(r.l1, r.r1) * arrB[r.r2];
}
```

**写代码时眼睛要看显示屏，警钟撅烂。**

## galaxy - 星战

~~写了 2KB 多的代码只得 25 分 全部输出 NO 都能得 45 分 强烈谴责造数据的出题人（（~~

题目的本意是：有 4 种操作，分别是断开/连接某条边、断开/连接某个点的所有入边，每次操作后判断图是否满足以下性质：

1. 所有点有且仅有一条出边；
2. 从任意一点出发都能形成环。

实际上由性质一可以直接推出性质二，所以考场上写的 DFS 判环甚至塔尖缩点等等都是白写哒！（尽管如此，实际比赛时写了也问题不大（前提是写对），无非就是发现这条性质后去掉这个操作可以稍微快一点。）

最朴素的做法就是邻接矩阵存图再维护出边数量：

```c++
bool graph[MAXN][MAXN];
bool exist[MAXN][MAXN]; // 注意还需要记录一条边是否存在，供操作2和4判断
// 考试时就是因为没有考虑这个而少得了15分，警钟撅烂x2
int outCnt[MAXN];
int n, m, q;

scanf("%d%d", &n, &m);
int op, u, v;
for (int i = 1; i <= m; i++)
{
    scanf("%d%d", &u, &v);
    exist[u][v] = true;
    graph[u][v] = true;
    outCnt[u]++;
}
scanf("%d", &q);
for (int i = 1; i <= q; i++)
{
    scanf("%d", &op);
    if (op == 1 || op == 3)
    {
        scanf("%d%d", &u, &v);
        if (op == 1)
            graph[u][v] = false, outCnt[u]--;
        else // op == 3
            graph[u][v] = true, outCnt[u]++;
    }
    else
    {
        scanf("%d", &u);
        if (op == 2)
        {
            for (int i = 1; i <= n; i++)
            {
                if (!exist[i][u])   // 不存在的边肯定不能拆/连
                    continue;
                if (graph[i][u])
                    outCnt[i]--;
                graph[i][u] = false;
            }
        }
        else // op == 4
        {
            for (int i = 1; i <= n; i++)
            {
                if (!exist[i][u])
                    continue;
                if (!graph[i][u])
                    outCnt[i]++;
                graph[i][u] = true;
            }
        }
    }
    bool flag = false;
    for (int i = 1; i <= n; i++)
    {
        if (outCnt[i] != 1)
        {
            printf("NO\n");
            flag = true;
            break;
        }
    }
    if (!flag)
        printf("YES\n");
}
return 0;
```

这样做只能过掉 $N$ 比较小的数据，因为邻接矩阵实在是太太太浪费内存了。一个或许更好的方法是把二维数组改成 `map`：

```c++
struct Edge
{
    Edge(int u, int v) : src(u), dest(v)
    {
    }
    int src, dest;

    const bool operator<(const Edge &a) const   // 要定义这个才能用map
    {
        return src == a.src ? dest < a.dest : src < a.src;
    }
};

map<Edge, int> edgeStats; // 0为不存在，1为可用，-1为被摧毁
int outCnt[MAXN];
int n, m, q;

scanf("%d%d", &n, &m);
int op, u, v;
for (int i = 1; i <= m; i++)
{
    scanf("%d%d", &u, &v);
    edgeStats[Edge(u, v)] = 1;
    outCnt[u]++;
}
scanf("%d", &q);
for (int i = 1; i <= q; i++)
{
    scanf("%d", &op);
    if (op == 1 || op == 3)
    {
        scanf("%d%d", &u, &v);
        if (op == 1)
            edgeStats[Edge(u, v)] = -1, outCnt[u]--;
        else // op == 3
            edgeStats[Edge(u, v)] = 1, outCnt[u]++;
    }
    else
    {
        scanf("%d", &u);
        if (op == 2)
        {
            for (int i = 1; i <= n; i++)
            {
                if (edgeStats[Edge(i, u)] == 1)
                {
                    edgeStats[Edge(i, u)] = -1;
                    outCnt[i]--;
                }
            }
        }
        else // op == 4
        {
            for (int i = 1; i <= n; i++)
            {
                if (edgeStats[Edge(i, u)] == -1)
                {
                    edgeStats[Edge(i, u)] = 1;
                    outCnt[i]++;
                }
            }
        }
    }
    bool flag = false;
    for (int i = 1; i <= n; i++)
    {
        if (outCnt[i] != 1)
        {
            printf("NO\n");
            flag = true;
            break;
        }
    }
    if (!flag)
        printf("YES\n");
}
return 0;
```

这时就会发现然并卵，原先 RE 的只是变成了 TLE 而已，因为 `map` 也实在是太太太慢了。

还有一种操作就是用 `bitset` 替换 `bool` 数组进行压位，实际上也是然并卵，因为压位并不能压进数据范围限制内。

无所谓了，卡时间总比卡内存容易。有一个比较“愚蠢”的方法是：判定时不再判断每个点的出度是否都为 1，而是判断**入度**之和是否为 $n$。同时发现，操作 2 和 4 是按照一个点的所有入边来操作的，这样维护入度也可以方便不少 ~~（甚至图都可以不用存）~~。

```c++
int inCnt[MAXN], inCntExist[MAXN], sum;
int n, m, q;

scanf("%d%d", &n, &m);
int op, u, v;
for (int i = 1; i <= m; i++)
{
    scanf("%d%d", &u, &v);
    inCnt[u]++, inCntExist[u]++;    // 要把原始数据备份一下
    sum += inCnt[u];
    // 为什么这里是加上inCnt[u]而不是sum++？因为sum的实际含义
    // 其实并不是入度之和，具体原理讲起来有点绕，只可意会不可言传。
    // 总之sum==n时一定能保证入度之和为n，而且大多数情况下这是一个充要条件。
}
scanf("%d", &q);
for (int i = 1; i <= q; i++)
{
    scanf("%d", &op);
    if (op == 1 || op == 3)
    {
        scanf("%d%d", &u, &v);
        if (op == 1)
            inCnt[u]--, sum--;
        else // op == 3
            inCnt[u]++, sum++;
    }
    else
    {
        scanf("%d", &u);
        if (op == 2)
        {
            if (inCnt[u] > 0)
            {
                sum -= inCnt[u];
                inCnt[u] = 0;
            }
        }
        else // op == 4
        {
            inCnt[u] = inCntExist[u];
            sum += inCnt[u];
        }
    }
    printf(sum == n ? "YES\n" : "NO\n");
}
return 0;
```

此时就会惊恐地发现居然连样例都过不了了，但是丝毫不影响多拿 10 分。

## transmit - 数据传输
