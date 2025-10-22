---
title: 考试总结 - CSP-S 2020 模拟测试
date: 2023-08-28 17:08:05 +0800
categories: [考试总结]
tags: [csp-s]
math: true
---

## 回顾

开题时先把四道题完整读了一遍，没急着动手。之后再依次给每道题写了暴力，再挑有思路的题目下手。

T1 应该算模拟，代码量有点大，最后由于很奇怪的 bug 调了半个多小时 ~~（但依然没有完全调对）（至少比挂掉好一点）~~ 导致没有时间钻研其他题目。本来 T2 不至于只拿暴力分的，诈骗其实相当明显。

分析近几次模拟赛，有一个共同问题就是时间不够。时间不够的主要原因在于暴力写太慢，以及在真正思维过程中又有点过分追求严谨、举棋不定。比赛写代码应该紧紧围绕“多拿分”这一个目的，不能被一些没必要考虑的细节所分散注意力。例如 T1 如果追求一定要用通用方法全部分类讨论算出来，那么代码就会极其冗长、难调试还容易出错；更好的做法是把儒略历的部分暴力预处理出来，格里高利历的部分再用倍增或者周期除余加速算，就可以避免繁琐的分类讨论。

竞赛讲究一个能力，比如思维能力、代码能力等等，这些能力都应该在做题和总结中不断提高。学知识和模板只是工具，关键是要明白这些工具要怎么灵活运用。

还有一个方面就是读题需要仔细。T2 因为一句小字“所有的 $q_i$ 互不相同”就决定了其诈骗性质，忽略这一点也是 T2 没做出来的原因之一。

## `julian` - 儒略日

“回顾”里已经说的比较多了，写出来发现倍增反而会超时，按照类似进制的思想一级一级算（400 年->1 年->1 天）会更快。

```c++
using ll = long long;
const int MONTH[13] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
const int MAX_LOG = 40;
const int TURN = 2299239; // -4712.1.1 -> 1583.1.1
const int ROUND = 146097; // 格里高利历中任意连续的400年都是这么多天

struct Date
{
    Date(int _y = 0, int _m = 0, int _d = 0) : y(_y), m(_m), d(_d)
    {
    }
    int y, m, d;
    void print()
    {
        printf("%d %d", d, m);
        if (y > 0)
            printf(" %d\n", y);
        else
            printf(" %d BC\n", (-y) + 1);
    }
};
Date mem[TURN + 3];

inline bool isGapYear(int y)    // 是否是闰年
{
    if (y <= 1582)
    {
        return y % 4 == 0;
    }
    else
    {
        if (y % 4 != 0)
            return false;
        if (y % 400 == 0)
            return true;
        if (y % 100 == 0)
            return false;
        return true;
    }
}
inline int dayInYaer(int y) // 一年中的天数
{
    return isGapYear(y) ? 366 : 365;
}
inline int dayInMonth(int y, int m) // 某月中的天数
{
    if (isGapYear(y) && m == 2)
        return MONTH[m] + 1;
    else
        return MONTH[m];
}
inline void addDay(int &y, int &m, int &d)  // 前进1天
{
    if (y == 1582 && m == 10 && d == 4)
    {
        d = 15;
        return;
    }
    d++;
    if (d > dayInMonth(y, m))
    {
        d -= dayInMonth(y, m), m++;
        if (m > 12)
            y++, m -= 12;
    }
}

void initMem()  // 1583.1.1及以前 暴力预处理
{
    Date cur(-4712, 1, 1);
    for (int n = 0; n <= TURN; n++)
    {
        mem[n] = cur;
        addDay(cur.y, cur.m, cur.d);
    }
}
void solve(ll n)
{
    int y = 1583, m = 1, d = 1;
    //400年一周期
    y += (n / ROUND) * 400;
    n %= ROUND;
    //按年跳
    while (n >= 367)
    {
        n -= dayInYaer(y);
        y++;
    }
    //按天跳
    for (; n > 0; n--)
    {
        addDay(y, m, d);
    }
    Date(y, m, d).print();
}

int main()
{
    initMem();

    int q;
    scanf("%d", &q);
    while (q--)
    {
        ll n;
        scanf("%lld", &n);
        if (n <= TURN)
            mem[n].print();
        else
            solve(n - TURN);
    }
    return 0;
}
```

顺便把倍增的实现方法贴在这里：

```c++
inline int howManyGaps(int r)   // 1582至r有多少闰年
{
    int res = (r - 1580) / 4;
    if (r <= 1600)
        return res;
    int err = (r - 1600) / 100 - (r - 1600) / 400;
    return res - err;
}
void solveJump(ll n)
{
    int y = 1583, m = 1, d = 1;
    for (int lg = MAX_LOG - 1; lg >= 0; lg--)
    {
        while (n >= jump[lg])   // jump[lg] = 365*(1<<lg)
        {
            n -= jump[lg];
            y += (1 << lg);
        }
    }
    n -= howManyGaps(y - 1);    // 少算的闰年要补回去
    while (n < 0)   // 可能会减到负数，需要把多余的年退掉
    {
        n += dayInYaer(y - 1);
        y--;
    }
    for (; n > 0; n--)  // 剩余部分按天跳
    {
        addDay(y, m, d);
    }
    Date(y, m, d).print();
}
```

## `zoo` - 动物园

因为“所有的 $q_i$ 都不相同”，所以只需要计算两个东西：对购买饲料有贡献的位 $t$，和没有 $p_i$ 涉及的位（即不可能触发饲料购买的位）$P$。这些位统称为“宽容位”，新增加的动物编号在“宽容位”上可以随意取值（$0$ 或 $1$），剩下的位全部为 $0$ 即可。答案即为 `1<<(count(t|P))-n`（`count` 函数为取二进制中 $1$ 的个数），减去 $n$ 是因为所有本来饲养了的动物要么对饲料购买有贡献，要么没有触发饲料购买，一定满足上述条件。

最后有个小细节：$k=64$ 的点会让 `unsigned long long` 溢出，需要开 `int128`。

```c++
using ull = __int128;
int n, m, c, k;
ull tot, P, t;

int main()
{
    scanf("%d %d %d %d", &n, &m, &c, &k);
    unsigned long long id;
    for (int i = 1; i <= n; i++)
    {
        scanf("%llu", &id);
        tot |= id;
    }
    for (int i = 1; i <= m; i++)
    {
        int p, q;
        scanf("%d %d", &p, &q);
        P |= ((__int128)1 << p);
    }

    t = (tot & P) & (((__int128)1 << k) - 1);
    P = (~P) & (((__int128)1 << k) - 1);
    int cnt = popcount(t | P);

    printInt128(((__int128)1 << cnt) - n);
    return 0;
}
```

## `call` - 函数调用

### Subtask 1：暴力（20%）

把所有操作模拟一遍。

### Subtask 2：优化暴力（70%）

暴力的主要时间负担在于每个操作 2 都是 $O(n)$ 的，特别浪费时间。联想到线段树打标记，能不能给它优化掉呢？答案是肯定的。但是似乎只开一个乘法和一个加法标记又不够用，因为乘和加会交替出现，维护标记本身又会需要 $O(n)$ 时间。这也好办，只需要开一个全局乘法标记和加法标记数组，再把所有的调用函数的操作**倒序**模拟就行了。最终答案就是 `a[i]*mult+mark[i]`。

下面简单证明一下这种操作的合理性。我们先把所有位置的初值看作 $0$，在调用函数前加上 $n$ 个对应的加法操作。一个加法操作可能会被后面的乘法操作放大，放大的倍数是从当前操作开始、后面所有乘法操作的值的乘积。如果我们从后往前执行函数调用操作，那么全局乘积标记也就同时记录了往后的所有乘法操作的影响倍数。因此当遇到一个加法操作时，它的值一定会在最终被放大 $mult$ 倍。

代码实现也很简单，跟纯暴力几乎一样。

```c++
struct Func
{
    int type;
    int p, v;
    vector<int> call;
};

int n, m, q;
int val[MAXN];
Func fc[MAXN];

long long mult = 1;
long long mark[MAXN];

int ask[MAXN];

inline void doWork(Func &f)
{
    switch (f.type)
    {
    case 1:
        mark[f.p] = (mark[f.p] + (1LL * f.v * mult) % MOD) % MOD;
        break;
    case 2:
        mult = 1LL * mult * f.v % MOD;
        break;
    case 3:
        for (int i = f.call.size() - 1; i >= 0; i--)
            doWork(fc[f.call[i]]);
        break;
    }
}

int main()
{
    n = read();
    for (int i = 1; i <= n; i++)
        val[i] = read();

    m = read();
    for (int i = 1; i <= m; i++)
    {
        int t = read();
        fc[i].type = t;
        int c;
        switch (t)
        {
        case 1:
            fc[i].p = read();
            fc[i].v = read();
            break;
        case 2:
            fc[i].v = read();
            break;
        case 3:
            c = read();
            for (int j = 1; j <= c; j++)
                fc[i].call.push_back(read());
            break;
        }
    }

    q = read();
    for (int i = 1; i <= q; i++)
        ask[i] = read();

    for (int i = q; i >= 1; i--)
        doWork(fc[ask[i]]);

    for (int i = 1; i <= n; i++)
        printf("%d ", (mult * val[i] % MOD + mark[i]) % MOD);
    printf("\n");
    return 0;
}
```

## `snakes` - 贪吃蛇

### Subtask 1：$n=3$（20%）

让第三条蛇考虑要不要吃第一条蛇，如果吃完后战斗力小于第二条蛇则不吃，答案为 $3$；否则答案为 $1$。

### Subtask 2：暴力，$n \leq 10$（40%）

首先尝试让所有蛇都“降智”，即不考虑自己能不能活，贪心地尝试吃所有能吃的蛇。那么很大概率会有笨蛇因为吃了别的蛇而导致自己被吃了，这是我们不希望发生的事情，因此如果出现了这种情况就需要排除掉。

对于蛇老大，先让它尝试吃掉最小的蛇，再以吃掉后的状态进行一次递归的模拟；如果蛇老大在未来能够活下来，那么它肯定会选择吃；否则它就不吃了。

```c++
// State[i] 表示第i条蛇是否存活
using State = bitset<2005>;
int hp[MAXN];
int snk[MAXN];
bool snkComp(int a, int b)
{
    return hp[a] == hp[b] ? a < b : hp[a] < hp[b];
}
// 以S为初始状态，返回模拟之后的结果状态
State dfs(State S)
{
    // 仅有1条蛇存活时它肯定不会吃自己
    if (S.count() <= 1)
        return S;
    // 按照战力排序
    sort(snk + 1, snk + 1 + n, snkComp);
    int l = 1, r = n;
    while (!S[snk[l]])
        l++;
    while (!S[snk[r]])
        r--;
    // 找到最弱和最强的存活的蛇
    int mm = snk[l], mx = snk[r];
    // 尝试吃
    hp[mx] -= hp[mm];
    S[mm] = false;
    // 递归模拟，预测未来
    State nxt = dfs(S);
    // 回溯，回到现实
    S[mm] = true;
    hp[mx] += hp[mm];
    // 如果自己能在未来活下来就选择吃
    if (nxt[mx])
        return nxt;
    else
        return S;
}
```

### Subtask 3：$n \leq 5 \times 10^4$（70%）

### Subtask 4：AC（100%）
