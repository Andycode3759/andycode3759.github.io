---
title: 考试总结 - 2023 牛客 OI 赛前集训营-提高组（第四场）
date: 2023-10-15 17:24:42 +0800
categories: [考试总结]
tags: [牛客]
math: true
---

## `A` - 同色三角形

### Subtask 1：$n \leq 5$（30%）

构图后二进制枚举边的状态，或者手算后分类讨论皆可。

### Subtask 2：AC（100%）

正难则反，计算异色三角形的数量。对于每一个点，设有 $a$ 条红边和 $b$ 条绿边，那么红绿搭配一定可以组成一个异色三角形；每一个异色三角形会被两个顶点计算两次，所以总数就是 $\sum_{i=1}^{n}{a_ib_i \over 2}$，再用总数 $C_{n}^{3}$ 减掉就是答案了。

其实这道题已经有很多地方在暗示正解思路了，例如点的顺序和编号都是无关紧要的，那么答案极有可能只和 $a$ 和 $b$ 的数量有关。没做出来还是因为第一眼看到题目没思路导致心里很慌，或者说做题时格局打开得还不够。

## `B` - 任务

### Subtask 1：25%

可以证明（或者写暴力程序实验得出），构造一个特定长度的字符串使得 $S$ 是其子序列的方案数，与 $S$ 的内容是无关的，只与 $S$ 的长度有关。可以设 $f(x,y)$ 为构造长度为 $y$ 的字符串，使其有一个指定的长度为 $x$ 的子序列的方案数。通过打表和大眼观察法可以得到以下递推式：$f(x,y)=25f(x,y-1)+f(x-1,y-1)$，边界是：

$$
f(x,y)=
\begin{cases}
26^y& \text{x=0}\\
1& \text{x=y}\\
0& \text{x>y}
\end{cases}
$$

直接用这个式子暴力计算有 25 分。

### Subtask 2：50%

可以把上述二维递推式转成一维的：$f(x,y)=25^{y-x} \cdot C_{y-1}^{x-1} + 26 \cdot f(x,y-1)$ ~~（至于怎么转...数学推导来得更快，反正作为数学渣渣考试时花了三小时愣是没看出规律来）~~。虽然直接用这个式子算还是只有 25 分，但是根据 $1 \leq n \leq 10^5,\sum \lvert s_i \rvert \leq 3 \times 10^5$ 可以猜测，有很多长度相同的 $s$。于是可以利用 `map` 把所有出现过的 $\lvert s_i \rvert$ 所对应的 $f(\lvert s_i \rvert,y),y \in [0,10^5]$ 当作数列预处理出来，需要的时候直接查询即可。

```c++
// quickPow, C（组合数）等函数如字面意思，模板实现略
int n, q;
int arr[MAXN];
int task[MAXN];

char buf[300005];

map<int, vector<int>> memF;
int makeF(vector<int> &mem, int x, int y)
{
    int res;
    if (x == 0)
        res = quickPow(26, y);
    else if (x == y)
        res = 1;
    else
        res = ((long long)C(y - 1, x - 1) * quickPow(25, y - x) % MOD + (long long)makeF(mem, x, y - 1) * 26 % MOD) % MOD;
    return mem[y] = res;
}

int main()
{
    init(); // 预处理阶乘用于算组合数，具体实现略
    scanf("%d %d", &n, &q);
    for (int i = 1; i <= n; i++)
    {
        scanf("%s", buf);
        task[i] = strlen(buf);
        if (memF.count(task[i]) == 0)
        {
            vector<int> t;
            t.resize(MAXN);
            makeF(t, task[i], 100000);
            memF[task[i]] = t;
        }
    }
    for (int i = 1; i <= n; i++)
        scanf("%d", arr + i);
    while (q--)
    {
        int op, l, r;
        scanf("%d %d %d", &op, &l, &r);
        if (op == 0)
            arr[l] = r;
        else if (op == 1)
        {
            int tot = 1;
            for (int i = l; i <= r; i++)
                tot = (long long)tot * memF[task[i]][arr[i]] % MOD;
            printf("%d\n", tot);
        }
    }
    return 0;
}
```

### Subtask 3：100%

再仔细分析一下查询的过程，发现其本质就是一个单点修改、区间求积。与区间求和同理，可以使用线段树或树状数组加速。同时预处理 $f$ 的过程还可以优化，将递归改成递推，避免大量调用 `quickPow` 导致浪费时间。

```c++
// 树状数组板子，但是改成区间求积
int fw[MAXN];
inline void fwInit()
{
    for (int i = 0; i <= n; i++)
        fw[i] = 1;
}
inline void fwModify(int pos, int x)
{
    for (; pos <= n; pos += (pos & (-pos)))
        fw[pos] = (long long)fw[pos] * x % MOD;
}
inline int fwGet(int pos)
{
    int res = 1;
    for (; pos > 0; pos -= (pos & (-pos)))
        res = (long long)res * fw[pos] % MOD;
    return res;
}

// 递推预处理
inline void makeF(vector<int> &mem, int x)
{
    int mult = 1;
    mem[x - 1] = 0;
    for (int i = x; i <= 100000; i++)
    {
        mem[i] = ((long long)C(i - 1, x - 1) * mult % MOD + (long long)mem[i - 1] * 26 % MOD) % MOD;
        mult = (long long)mult * 25 % MOD;
    }
}


int main()
{
    init();
    scanf("%d %d", &n, &q);
    fwInit();
    for (int i = 1; i <= n; i++)
    {
        scanf("%s", buf);
        task[i] = strlen(buf);
        if (memF.count(task[i]) == 0)
        {
            vector<int> t;
            t.resize(MAXN);
            makeF(t, task[i]);
            memF[task[i]] = t;
        }
    }
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", arr + i);
        fwModify(i, memF[task[i]][arr[i]]);
    }
    while (q--)
    {
        int op, l, r;
        scanf("%d %d %d", &op, &l, &r);
        if (op == 0)
        {
            arr[l] = r;
            // 查询单点值
            // quickPow(x,MOD-2)相当于求逆元（做除法）
            int t = (long long)fwGet(l) * quickPow(fwGet(l - 1), MOD - 2) % MOD;
            // 把原始值除掉，乘上新值
            fwModify(l, quickPow(t, MOD - 2));
            fwModify(l, memF[task[l]][r]);
        }
        else if (op == 1)
        {
            int tot = (long long)fwGet(r) * quickPow(fwGet(l - 1), MOD - 2) % MOD;
            printf("%d\n", tot);
        }
    }
    return 0;
}
```

总之，是一道集数学、思维、数据结构于一体的好题。

## `C` - 加法方案

这题的思路和 `A` 题有相似之处：枚举全部的选数方案显然是不可能的，那就计算每个数位对答案的贡献。为了叙述方便，读入时把 $a$ 数组反转一下，用 $a_{n-1}$ 表示最高位、$a_0$ 表示最低位。

对于一个数 $a_i$，它的贡献取决于自己位于选出来的数的哪一位上，设它的前面（靠低位方向）有 $j$ 个数，那么贡献就是 $a_i \times 10^j$；比它更高位的数对它的贡献是没有影响的，而高位数的选取方案共有 $2^{n-i-1}$ 种，因此一个数对答案的总贡献为 $2^{n-i-1} (\sum_{j=0}^{i} C_{i}^{j} \times a_i \times 10^j)$。总答案还要乘 $2$，因为分出来的另一半也有贡献，但是还要减去一个 $N$ 本身（因为不能取 $0$ 个）。得到这个式子就可以 $O(n^2)$ 枚举求和拿下前 50 分了。

```c++
scanf("%s", input);
n = strlen(input);
ll mult = 1;
for (int i = 0; i < n; i++)
{
    num[i] = input[n - i - 1] - '0';
    tot = (tot + num[i] * mult % MOD) % MOD;
    mult = mult * 10 % MOD;
}
for (int i = 0; i < n; i++)
{
    for (int j = 0; j <= i; j++)
    {
        sum = (sum + quickPow(2, n - i - 1) * C(i, j) % MOD * num[i] % MOD * quickPow(10, j) % MOD) % MOD;
    }
}
sum = sum * 2 % MOD;
sum = (sum - tot + MOD) % MOD;
printf("%lld\n", sum);
return 0;
```

如果要拿满分，无非就是简化掉式子中的求和符号。通过数学推导或打表可以发现 $\sum_{j=0}^{i} C_{i}^{j} \times 10^j = 11^i$，这样贡献的计算式就能简化为 $2^{n-i-1} \times 11^i \times a_i$。

```c++
for (int i = 0; i < n; i++)
    sum = (sum + quickPow(2, n - i - 1) * quickPow(11, i) % MOD * num[i] % MOD) % MOD;
```

满分代码加上快速幂不到 50 行，基本不存在程序实现方面的困难，并且思维方面难度也并不是很大，关键就在于第一步能不能想到计算贡献的思路，而不是第一眼看到数据范围没给暴力分就认为这道题完全不可做而放弃。

## `D` - 打标签
