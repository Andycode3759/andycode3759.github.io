---
title: 考试总结 - NOIP 2022 模拟测试
date: 2023-07-17 20:10:00 +0800
categories: [考试总结]
tags: [noip]
math: true
---

## `plant` - 种花

思路一：字母 `C` 和 `F` 都是两横加一竖构成的，于是先预处理出所有长度不小于 2 的横和长度不小于 3 的竖，再枚举配对并计算数量。

```c++
struct Segment
{
    Segment(int _p, int _l, int _r) : p(_p), l(_l), r(_r)
    {
    }
    int p, l, r;
};
vector<Segment> rows, cols;

int id;
bool land[MAXN][MAXN];
int n, m, doC, doF;

void preProcess()
{
    for (int i = 1; i <= n; i++)
    {
        int st = 1, len = 0;
        for (int j = 1; j <= m; j++)
        {
            if (land[i][j])
            {
                if (len >= 2)
                    rows.push_back(Segment(i, st, j - 1));
                st = j + 1;
                len = 0;
            }
            else
                len++;
        }
        if (len >= 2)
            rows.push_back(Segment(i, st, m));
    }
    for (int j = 1; j <= m; j++)
    {
        int st = 1, len = 0;
        for (int i = 1; i <= n; i++)
        {
            if (land[i][j])
            {
                if (len >= 3)
                    cols.push_back(Segment(j, st, i - 1));
                st = i + 1;
                len = 0;
            }
            else
                len++;
        }
        if (len >= 3)
            cols.push_back(Segment(j, st, n));
    }
}
inline void solve()
{
    int cntR = rows.size(), cntC = cols.size();
    int resC = 0, resF = 0;
    for (int ra = 0; ra < cntR; ra++)
    {
        Segment &rowA = rows[ra];
        for (int rb = ra + 1; rb < cntR; rb++)
        {
            Segment &rowB = rows[rb];
            if (rowB.p - rowA.p <= 1 || rowA.r - rowB.l <= 0)
                continue;
            for (int c = 0; c < cntC; c++)
            {
                Segment &col = cols[c];

                // 三条笔画需要符合的条件：
                // - 两横之间间距至少一格；
                // - 竖需要穿过两条横。
                if (col.p < rowA.l || col.p > rowA.r || col.p < rowB.l || col.p > rowB.r || col.r < rowB.p ||
                    col.l > rowA.p)
                    continue;
                if (doC)
                    resC = (resC + 1LL * (rowA.r - col.p) * (rowB.r - col.p)) % MOD;
                if (doF)
                    resF = (resF + 1LL * (rowA.r - col.p) * (rowB.r - col.p) * (col.r - rowB.p)) % MOD;
            }
        }
    }
    printf("%d %d\n", resC, resF);
}
```

这个方法对于 1 非常稠密的图理论上会更快，因为预处理得到的笔画数量比较少。但对于实际的数据来说很大一部分还是会超时。

思路二：对于每一个点，计算出其向右延伸和向下延伸的空地长度（即反向求前缀和，遇到坑就归零），再对于每一个 C 或 F 枚举其两个拐点，就能比较方便地计算数量。

```c++
int id;
int land[MAXN][MAXN];
int n, m, doC, doF;
int row[MAXN][MAXN], col[MAXN][MAXN];

inline void solve()
{
    int resC = 0, resF = 0;

    for (int h = 2; h < n; h++)
    {
        for (int i = 1; i + h <= n; i++)
        {
            for (int j = 1; j <= m; j++)
            {
                //(x1,y1) (x2,y2)即为两个拐点
                int x1 = i, x2 = i + h, y1 = j, y2 = j;
                if (row[x1][y1] <= 1 || row[x2][y2] <= 1 || col[x1][y1] - col[x2][y2] < h)
                    continue;
                if (doC)
                    resC += (row[x1][y1] - 1) * (row[x2][y2] - 1);
                if (doF)
                    resF += (row[x1][y1] - 1) * (row[x2][y2] - 1) * (col[x2][y2] - 1);
            }
        }
    }
    printf("%d %d\n", resC, resF);
}

// 为了方便起见，输入时把0和1反转
void preProcess()
{
    for (int i = 1; i <= n; i++)
    {
        for (int j = m; j >= 1; j--)
        {
            row[i][j] = row[i][j + 1] + land[i][j];
            if (land[i][j] == 0)
                row[i][j] = 0;
        }
    }
    for (int j = 1; j <= m; j++)
    {
        for (int i = n; i >= 1; i--)
        {
            col[i][j] = col[i + 1][j] + land[i][j];
            if (land[i][j] == 0)
                col[i][j] = 0;
        }
    }
}
```

分析以后发现，其实不需要枚举两个拐点，一个点就够了。可以把 `row` 数组（值需减一）按纵向求前缀和，就能得到对于每一个点可用的“C 的下半部分”的数量；同理，将 `(row-1)*(col-1)` 的值按纵向求前缀和，可以算出可用的“F 的下半部分”的数量。

```c++
typedef unsigned long long ulong;
int id;
int land[MAXN][MAXN];
int n, m, doC, doF;
// 防爆
ulong row[MAXN][MAXN], col[MAXN][MAXN];
ulong rowSum[MAXN][MAXN], fCnt[MAXN][MAXN];

void preProcess()
{
    for (int i = 1; i <= n; i++)
    {
        for (int j = m; j >= 1; j--)
        {
            if (land[i][j] == 0)
                row[i][j] = 0;
            else
                row[i][j] = row[i][j + 1] + land[i][j];
        }
    }
    for (int j = 1; j <= m; j++)
    {
        for (int i = n; i >= 1; i--)
        {
            if (land[i][j] == 0)
                col[i][j] = 0;
            else
                col[i][j] = col[i + 1][j] + land[i][j];
        }
    }
    for (int j = 1; j <= m; j++)
    {
        for (int i = 1; i <= n; i++)
        {
            // C的竖必须是连续的，但对于F来说每一横不一定都有对应的一竖作为“尾巴”，因此不必连续
            if (row[i][j] == 0)
                rowSum[i][j] = 0;
            else
                rowSum[i][j] = rowSum[i - 1][j] + row[i][j] - 1;

            fCnt[i][j] = (col[i][j] - 1) * (row[i][j] - 1) + fCnt[i - 1][j];
        }
    }
}
inline void solve()
{
    ulong resC = 0, resF = 0;

    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (row[i][j] <= 1 || col[i][j] <= 2)
                continue;
            // 可以根据col数组定位C的一竖最大可以往下延伸到何处
            // C的数量 = 当前点最长一横的长度 * 可用的“C的下半部分”数量
            resC += (row[i][j] - 1) * (rowSum[i + col[i][j] - 1][j] - rowSum[i + 1][j]);
            // F的数量 = 当前点最长一横的长度 * 可用的“F的下半部分”数量
            resF += (row[i][j] - 1) * (fCnt[i + col[i][j] - 2][j] - fCnt[i + 1][j]);
        }
    }

    printf("%llu %llu\n", resC * doC % MOD, resF * doF % MOD);
}
```

这道题难度不是很高，在 NOI 比赛范畴里应该属于签到题，但考试时交了思路一的代码导致超时只有 50 分。为什么没有整出思路二？因为考试时先做了第二题，浪费了不少时间（大约 90min）还一分没拿，导致第一题没有时间细想。

由此复盘一下可以发现，考试策略方面还是存在问题。对于没有把握拿分的题目不应该死磕，也不应该对任何题目都怀有“应该会送点分，空着感觉过不去”的错觉。考试时所有题目的分数都是平等的，简单题和难题的一分都是一分，因此应该先把有把握拿到的分都拿到手后，再去思考有难度的题目。而不是一上来所有题目全都硬着头皮写个暴力，然后一个一个慢慢调（这应该是普及组的做题思路），很有可能被卡到一分不剩。同时应该要学会判断题目的难易程度，能否准确地判断一道题是否在自己的能力范围内，对于考试策略的影响是很大的。

## `meow` - 喵了个喵

对于 $k=2n-2$ 的情况：$n-1$ 的两倍刚好是 $k$~~（废话）~~，那么可以考虑把 $1$ 到 $k$ 号牌都平均分配给前 $n-1$ 个栈，即一个栈管两种牌。如果一张牌其对应的栈是空的，或者栈顶与它相同，那就直接放在栈顶上；否则就借助最后面空出来的 $n$ 号栈消去栈底的相同牌。

对于 $k=2n-1$ 的情况：比第一种情况多了一种牌，最棘手的问题就是怎么处理多余的这种牌。

## `barrack` - 建造军营

暴力做法：对于 $n \leq 16$ 的情况，可以用二进制法枚举所有点的可能状态（建/不建军营）。对于每一种点的状态，有一种边的状态与其对应（即一条边是必须联通的，或是可有可无的）。用 DFS 把这个边的状态给求出来，数出里面可有可无的边的数量（记为 $s$），答案累加上 $2^s$ 即可。暴力的思路看似简单，但实际的程序不是很好写，有很多细节需要注意，稍不注意就会 WA/TLE。

特殊性质 A：所有的点呈线性排列，可以直接用数学方法算出来。

- 只建 $1$ 个军营：所有的边都是可有可无的，答案贡献为 $n \cdot 2^m$；
- 建 $2$ 个以上的军营：只需要考虑最远的两个军营的距离即可，因为中间的军营都是可有可无的；对于边则相反，中间的边必须联通，外部的边可有可无。对于最远端军营相距为 $h$ 的情况，内部有 $h-2$ 个可有可无的军营，外部有 $n-h$ 条可有可无的边，总共有 $n-h+1$ 种情况，故对答案的贡献为 $(n-h+1) \cdot 2^{n-2}$。
- 最终答案为 $n \cdot 2^m +2^{n-2} \cdot \sum_{h=2}^{n}{(n-h+1)}$。虽然这个式子后面看起来是个等差数列可以展开，但是展开来用 $O(1)$ 算就错了，用 $O(n)$ 枚举求和反而是对的，原因未知。~~（玄学）~~

记得预处理出所有 $2$ 的幂。还有暴力做法里少用 `memset`，效率非常低。

```c++
const int MAXN = 500005;
const int MAXM = 1000006;
const int MOD = 1000000007;

// 链式前向星存图
struct Edge
{
    int dest, next;
};
Edge edges[MAXM * 2];
int tot = 0;
int fir[MAXN];
inline void addEdge(int src, int dest)
{
    Edge e;
    e.dest = dest, e.next = fir[src];
    edges[++tot] = e;
    fir[src] = tot;
}
bool mark[MAXM * 2];    // 断边标记

int n, m;
unsigned long long pow2[MAXM];

inline int solveSP1() // 特殊性质A
{
    unsigned long long res = n * pow2[n - 1] % MOD;
    for (int h = 2; h <= n; h++)
    {
        res = (res + (n - h + 1) * pow2[n - 2]) % MOD;
    }
    return res % MOD;
}

// 暴力做法
bool vis[MAXN];
void dfs(int cur)   // DFS走遍所有点
{
    if (vis[cur])
        return;
    vis[cur] = true;
    for (int e = fir[cur]; e != -1; e = edges[e].next)
    {
        int v = edges[e].dest;
        if (mark[e])
            continue;
        dfs(v);
    }
}
vector<int> G;
inline bool checkVis()
{
    for (int i = 0; i < (int)G.size(); i++)
    {
        if (!vis[G[i]])
        {
            return false;
        }
    }
    return true;
}
inline int solveGeneral()
{
    if (n > 16 || m > 25)
        return time(0) % MOD;   // 超出暴力范围 输出一个随机数意思一下

    unsigned long long res = 0;
    int maxState = (1 << n) - 1;
    for (int state = 1; state <= maxState; state++)
    {
        vector<int>().swap(G);
        for (int i = 1; i <= m; i++)    // 把二进制状态转化为vector点集
        {
            if (state & (1 << (i - 1)))
            {
                G.push_back(i);
            }
        }
        int safeEdge = 0;
        for (int i = 1; i <= m; i++)
        {
            // 不要用memset，不要用memset，不要用memset
            // memset(vis, false, sizeof vis);
            for (int j = 1; j <= n; j++)
                vis[j] = false;

            mark[i * 2 - 1] = true;
            mark[i * 2] = true;
            dfs(G[0]);
            mark[i * 2 - 1] = false;
            mark[i * 2] = false;
            if (checkVis())
                safeEdge++;
        }
        res = (res + pow2[safeEdge]) % MOD;
    }
    return res % MOD;
}

int main()
{
    freopen("barrack.in", "r", stdin);
    freopen("barrack.out", "w", stdout);
    memset(fir, -1, sizeof fir);
    scanf("%d%d", &n, &m);
    pow2[0] = 1;
    for (int i = 1; i <= max(n, m); i++)
    {
        pow2[i] = (pow2[i - 1] << 1) % MOD;
    }
    bool sp1 = m == n - 1;
    int u, v;
    for (int i = 1; i <= m; i++)
    {
        scanf("%d%d", &u, &v);
        if (u - v != 1 && v - u != 1)
            sp1 = false;
        addEdge(u, v);
        addEdge(v, u);
    }
    if (sp1)
        printf("%d\n", solveSP1());
    else
        printf("%d\n", solveGeneral());
    return 0;
}
```

## `match` - 比赛

前面 8 分是送的，直接枚举即可。搞 ST 表纯属自作多情，该超时的还是得超时（因为 $Q$ 太大了）。

20 分相对比较好拿，无非空间换时间，把所有 $[l,r]\ 1 \leq l \leq r \leq n$ 的区间的小答案全都预处理出来放进一个二维数组，查询时求和即可。空间和时间都是 $O(n^2)$（若 $n$ 和 $q$ 同阶）。

```c++
for (int l = 1; l <= n; l++)
{
    for (int r = l; r <= n; r++)
    {
        // 确定左端点，右端点答案取前缀和
        // getMaxA和getMaxB都是ST表加速的取最大值
        sum[l][r] = sum[l][r - 1] + getMaxA(l, r) * getMaxB(l, r);
    }
}

for (int i = 1; i <= q; i++)
{
    unsigned long long res = 0;
    int l = ql[i], r = qr[i];
    for (int i = l; i <= r; i++)
    {
        // 枚举左端点求和
        res += sum[i][r];
    }
    printf("%llu\n", res);
}
```
