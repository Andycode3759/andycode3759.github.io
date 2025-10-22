---
title: 考试总结 - NOIP 2020 模拟测试
date: 2023-09-03 17:31:50 +0800
categories: [考试总结]
tags: [noip]
math: true
---

## `water` - 排水系统

直接 BFS 模拟排水的过程，队列内数据点记录两个值，点的编号和水量。需要实现一个分数结构体及其加法、除法运算。理论上时间复杂度是不符合要求的，但数据比较水，可以过。

以及，分数运算过程中分母分子可能会非常大，甚至爆 `long long`，需要开 `__int128`。

```c++
using ll = __int128;

void printInt128(ll x)
{
    if (x >= 10)
        printInt128(x / 10);
    printf("%d", x % 10);
    return;
}

ll gcd(ll a, ll b)
{
    if (b == 0)
        return a;
    else
        return gcd(b, a % b);
}

struct Fraction
{
    Fraction(ll _p = 0, ll _q = 1) : p(_p), q(_q)
    {
    }
    ll p, q;

    Fraction operator+(Fraction &f) const
    {
        Fraction res(p * f.q + f.p * q, q * f.q);
        ll g = gcd(res.p, res.q);
        res.p /= g, res.q /= g;
        return res;
    }
    Fraction operator/(int x) const
    {
        Fraction res(p, q * x);
        ll g = gcd(res.p, res.q);
        res.p /= g, res.q /= g;
        return res;
    }
};

vector<int> G[MAXN];

int n, m;
int deg[MAXN]; // deg[u]=0 -> isIn[u]=true
bool isOut[MAXN];

Fraction water[MAXN];

using Node = pair<int, Fraction>;
queue<Node> Q;

int main()
{
    n = fastRead(), m = fastRead();
    for (int i = 1; i <= n; i++)
    {
        int c = fastRead();
        if (c == 0)
            isOut[i] = true;
        for (int j = 1; j <= c; j++)
        {
            int v = fastRead();
            G[i].push_back(v);
            deg[v]++;
        }
    }

    for (int i = 1; i <= n; i++)
    {
        if (deg[i] == 0)
        {
            Q.push({i, Fraction(1, 1)});
        }
    }

    while (!Q.empty())
    {
        Node t = Q.front();
        Q.pop();
        if (isOut[t.first])
        {
            water[t.first] = water[t.first] + t.second;
        }
        else
        {
            for (int i = 0, len = G[t.first].size(); i < len; i++)
            {
                int v = G[t.first][i];
                Q.push({v, t.second / len});
            }
        }
    }

    for (int i = 1; i <= n; i++)
    {
        if (isOut[i])
        {
            printInt128(water[i].p);
            printf(" ");
            printInt128(water[i].q);
            printf("\n");
        }
    }

    return 0;
}
```

## `string` - 字符串匹配

### Subtask 1：暴力（32%~48%）

枚举 $C$ 的长度，切掉一个尾巴之后字符串前面就变成了 $AB$ 循环若干次。

可以用 $O(nlogn)$ 的方式枚举一个字符串所有的循环节长度（[source](https://yundouxueyuan.com/p/5236)）：首先循环节长度 $r$ 必须是总长度 $n$ 的因子，之后将字符串分别切掉前面 $r$ 个字符、后面 $r$ 个字符，得到两个长度相同的子串，首尾对齐后匹配，如果一致说明 $r$ 是原字符串的一个循环节长度。

对于一个有效的 $r$ 再枚举 $A$ 和 $B$ 之间的切分位置，之后只要比较 $F(A)$ 和 $F(C)$ 就行了，一个前缀一个后缀，可以分别从前往后、从后往前预处理出来。总时间复杂度约等于 $O(n^2log^2n)$。

检查循环节时匹配可以直接暴力比较，试过了用哈希，区别不大。

```c++
memset(mark, false, sizeof mark);
int fc = 0;
for (int i = 1; i <= len; i++)
{
    if (mark[str[i]])
        fc--, mark[str[i]] = false;
    else
        fc++, mark[str[i]] = true;
    preF[i] = fc;
}
memset(mark, false, sizeof mark);
fc = 0;
for (int i = len; i >= 1; i--)
{
    if (mark[str[i]])
        fc--, mark[str[i]] = false;
    else
        fc++, mark[str[i]] = true;
    repF[i] = fc;
}

long long ans = 0;
for (int n = len - 1; n >= 2; n--)
{
    int c = len - n;
    int fc = repF[n + 1];
    for (int i = 2; i <= n / 2; i++)
    {
        if (n % i != 0)
            continue;
        if (customStrcmp(1, 1 + i, n - i))
        {
            // [1,i] = AB
            for (int j = 1; j < i; j++)
            {
                // [1,j] = A
                // [j+1,i] = B
                int fa = preF[j];
                if (fa <= fc)
                    ans++;
            }
        }
    }
    // [1,n] = AB
    for (int j = 1; j < n; j++)
    {
        // [1,j] = A
        // [j+1,n] = B
        int fa = preF[j];
        if (fa <= fc)
            ans++;
    }
}
printf("%lld\n", ans);
```

### Subtask 2：一种字符（+8%）

依然是枚举 $C$ 并枚举前面 $AB$ 的循环节。既然只有一种字符那么可以跳过匹配检查，只需要考虑字符个数即可。如果 $\lvert C \rvert$ 是偶数，那么 $F(C)=0$，如果是奇数，那么 $F(C)=1$；分别对应 $F(A)$ 必须是 $0$ 即 $\lvert A \rvert$ 是偶数，以及 $\lvert A \rvert$ 是奇是偶都可以。具体数学式子写在了代码里，按照公式直接计算方案数即可免去枚举 $A$ 的长度。时间复杂度 $O(nlogn)$。

```c++
inline long long solve1_GetG(int x, int c)
{
    return (c & 1) == 1 ? x - 1 : (x - 1) >> 1;
}
inline long long solve1_GetH(int x, int c)
{
    long long res = 0;
    for (int i = 1; i * i <= x; i++)
    {
        if (x % i != 0)
            continue;
        res += solve1_GetG(i, c);
        if (i * i != x)
            res += solve1_GetG(x / i, c);
    }
    return res;
}
inline void solve1()
{
    long long ans = 0;
    for (int n = 2; n < len; n++)
    {
        ans += solve1_GetH(n, len - n);
    }
    printf("%lld\n", ans);
}
```

### Subtask 3：AC（100%）

换个角度考虑问题：不枚举 $C$，而是枚举 $AB$，再看 $AB$ 最多可以往后复制多少次，剩下的部分当作 $C$ 即可。比较复制的部分是否一样依旧可以用哈希。

核心问题来到统答案。如果还是枚举 $A$ 和 $B$ 的切分点再一个个累加，那么和暴力做法无异。仔细思考就会发现，$F$ 的值域为 $[0,26]$，可以用类似于权值前缀和的方法，开 $27$ 个数组（或者数状数组），用 $sumCnt(i,j)$ 表示前 $j$ 个位置中 $F(x)=i$ 出现了多少次。答案累加就能够快很多。

```c++
for (int i = 1; i <= len; i++)
{
    sumCnt[preF[i]][i] += 1;
}
for (int i = 0; i < 27; i++)
{
    for (int j = 1; j <= len; j++)
    {
        sumCnt[i][j] += sumCnt[i][j - 1];
    }
}

// a -> AB的长度
for (int a = 2; a < len; a++)
{
    // t -> AB复制的次数
    for (int t = 1; t * a < len; t++)
    {
        ull h1 = Hash[a];
        ull h2 = Hash[a * t] - Hash[a * (t - 1)] * PPow[a];
        // 当前这节是一致的
        if (h1 == h2)
        {
            for (int b = 0; b <= repF[a * t + 1]; b++)
            {
                ans += sumCnt[b][a - 1];
            }
        }
        // 否则后面的就不用看了
        else
            break;
    }
}
```

上述代码只能得 84 分。拿满分很简单，对 $sumCnt(i,j)$ 再关于 $i$ 做一次前缀和（相当于二维前缀和）即可省掉里面关于 $b$ 的一层循环。

```c++
for (int i = 1; i <= len; i++)
{
    sumCnt[preF[i]][i] += 1;
}
for (int i = 0; i < 27; i++)
{
    for (int j = 1; j <= len; j++)
    {
        sumCnt[i][j] += sumCnt[i][j - 1];
    }
}
for (int i = 1; i < 27; i++)
{
    for (int j = 1; j <= len; j++)
    {
        sumCnt[i][j] += sumCnt[i - 1][j];
    }
}
ull ans = 0;
for (int a = 2; a < len; a++)
{
    for (int t = 1; t * a < len; t++)
    {
        ull h1 = Hash[a];
        ull h2 = Hash[a * t] - Hash[a * (t - 1)] * PPow[a];
        if (h1 == h2)
            ans += sumCnt[repF[a * t + 1]][a - 1];
        else
            break;
    }
}
```

这道题似乎常数比较大，在洛谷上交的时候 20 号点刚好处于 1s 左右的临界时间 ~~（能不能 AC 看运气）~~。在服务器不那么好的学校内部 OJ 则会超时 0.2s，依然只能拿 84 分。加个[火车头](https://www.luogu.com.cn/paste/pojcu4ev)即可解决一切常数问题（

## `ball` - 移球游戏

### Subtask 1：$n=2$（10%）

暴力并不是很好写，毕竟无法提前确定需要多少步才能结束。这时就需要对它使用迭代加深：

```c++
int n, m;
int pile[MAXN][404];
int top[MAXN];

using Action = pair<int, int>;
vector<Action> Q;

inline void move(int x, int y)
{
    pile[y][++top[y]] = pile[x][top[x]--];
    Q.push_back({x, y});
}
inline void undoMove()
{
    Action lst = Q.back();
    Q.pop_back();
    pile[lst.first][++top[lst.first]] = pile[lst.second][top[lst.second]--];
}

inline bool check()
{
    for (int i = 1; i <= n + 1; i++)
    {
        if (top[i] == 0)
            continue;
        if (top[i] != m)
            return false;
        for (int j = 2; j <= m; j++)
        {
            if (pile[i][j] != pile[i][j - 1])
                return false;
        }
    }
    return true;
}

bool idasDfs(int step, int maxStep)
{
    if (step > maxStep)
    {
        return check();
    }
    for (int i = 1; i <= n + 1; i++)
    {
        for (int j = 1; j <= n + 1; j++)
        {
            if (i == j || top[i] == 0 || top[j] == m)
                continue;
            if (!Q.empty() && i == Q.back().second && j == Q.back().first)
                continue;
            move(i, j);
            if (idasDfs(step + 1, maxStep))
                return true;
            undoMove();
        }
    }
    return false;
}

inline void solve()
{
    for (int i = 1; i <= 820000; i++)
    {
        if (idasDfs(1, i))
            break;
    }
    printf("%d\n", Q.size());
    for (int i = 0; i < Q.size(); i++)
    {
        printf("%d %d\n", Q[i].first, Q[i].second);
    }
}
```

时间瓶颈关键在于 `check` 函数的实现，如果暴力扫描的话显然会超时，只能拿 5 分。 ~~（突然忘记 10 分要怎么写了）~~

### Subtask 2：40%

### Subtask 3：70%

### Subtask 4：AC（100%）

## `walk` - 微信步数

### Subtask 1：暴力（25%~35%）

纯模拟，DFS 枚举所有的起点再直接按照方案走即可。

```c++
int n, k;
int w[12];
using Step = pair<int, int>;
Step st[MAXN];

int pos[12], pos2[12];
long long sum = 0;
void dfs(int c)
{
    if (c > k)
    {
        for (int i = 1; i <= k; i++)
            pos2[i] = pos[i];

        long long ans = 0;
        for (int i = 0;; i = (i + 1) % n)
        {
            bool flag = false;
            for (int j = 1; j <= k; j++)
            {
                if (pos2[j] > w[j] || pos2[j] < 1)
                {
                    flag = true;
                    break;
                }
            }
            if (flag)
                break;
            pos2[st[i].first] += st[i].second;
            ans++;
        }
        sum = (sum + ans) % MOD;
        return;
    }
    for (int i = 1; i <= w[c]; i++)
    {
        pos[c] = i;
        dfs(c + 1);
    }
}
inline void solveForce()
{
    dfs(1);
    printf("%lld\n", sum);
}
```

此方法只能拿到 25 分，原因是没有处理输出 $-1$ 的情况，会陷入死循环。判重也比较简单，可以用类似于哈希的方法对坐标进行编码。

```c++
const int HASH_MOD = 499979;
const int HASH_P = 241;

int n, k;
int w[12];
using Step = pair<int, int>;
Step st[MAXN];

bool vis[HASH_MOD];

int pos[12], pos2[12];
long long sum = 0;
void dfs(int c)
{
    if (c > k)
    {
        for (int i = 1; i <= k; i++)
            pos2[i] = pos[i];
        memset(vis, false, sizeof vis);
        long long ans = 0;
        while (true)
        {
            int code = 0;
            for (int i = 1; i <= k; i++)
            {
                code = (code * HASH_P % HASH_MOD + pos2[i]) % HASH_MOD;
            }
            if (vis[code])
            {
                printf("-1\n");
                exit(0);
            }
            vis[code] = true;
            bool flag = false;
            for (int i = 0; i < n; i++)
            {
                for (int j = 1; j <= k; j++)
                {
                    if (pos2[j] > w[j] || pos2[j] < 1)
                    {
                        flag = true;
                        break;
                    }
                }
                if (flag)
                    break;
                pos2[st[i].first] += st[i].second;
                ans++;
            }
            if (flag)
                break;
        }
        sum = (sum + ans) % MOD;
        return;
    }
    for (int i = 1; i <= w[c]; i++)
    {
        pos[c] = i;
        dfs(c + 1);
    }
}
inline void solveForce()
{
    dfs(1);
    printf("%lld\n", sum);
}
```

这样就可以拿到 35 分了。

### Subtask 2：$k=1$（+10%）

### Subtask 3：AC（100%）
