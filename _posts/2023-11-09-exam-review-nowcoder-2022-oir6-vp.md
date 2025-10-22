---
title: 考试总结 - 2022 牛客 OI 赛前集训营-提高组 Round 6 (vp)
date: 2023-11-09 20:44:28 +0800
categories: [考试总结]
tags: [牛客]
math: true
---

## `A`

这种题目和[种花](https://www.luogu.com.cn/problem/P8865)之类的题目属于同一类，几乎没有算法，考察分析问题的能力和代码基本功，但是根据近期表现来看这类题目做得不是很理想。

把所有的三元组想象成一个 $A \times B \times C$ 个方块组成的长方体，每一次删除相当于以 $(1,1,1),(u,v,w)$ 为两个端点确定一个小长方体，把这个小长方体里的方块删掉。

根据生成器可以看出 $u,v,w$ 三维至少有一个会达到上限，而且有高达 $1 \over 9$ 的概率会出现 $u=A,v=B$ 同时满足的情况。可以把这些操作找出来，维护一个 $w$ 的最大值，记作 $hei$，表示高度为 $hei$ 的这一半可以直接砍掉。

接着处理剩下的操作，又因为数据随机，剩下一块的高度肯定会比较小，考虑 $O(n)$ 处理每一层。从下往上枚举 $z$ 轴坐标（即从 $C$ 递减到 $hei+1$，以最左上方靠里的点为 $(1,1,1)$），找到 $w=z$ 的操作，对于每个 $x$ 轴坐标维护最右端被删到了哪一列，记作 $len_x$；又因为长方体一定是从里往外扩的，对 $len_x$ 数组求一个后缀最大值后，全部求和就是这一层被删掉的方块数。因为越靠近原点，被删的面积一定单调不降，即 $w$ 大的操作会影响 $w$ 小的，所以 $len_x$ 数组可以直接留给下一个 $z$。

最后要记得开 `__int128`。

```c++
void printInt128(__int128 x)
{
    if (x >= 10)
        printInt128(x / 10);
    printf("%d", (int)(x % 10));
}
int hei, len[MAXN];
int main()
{
    DataGen::GetData();

    for (int i = 1; i <= n; i++)
    {
        if (u[i] == A && v[i] == B)
            hei = max(hei, w[i]);
    }
    __int128 ans = (__int128)A * B * hei;
    for (int h = C; h > hei; h--)
    {
        for (int i = 1; i <= n; ++i)
            if (w[i] == h)
                len[u[i]] = max(len[u[i]], v[i]);
        for (int i = A - 1; i >= 1; i--)
            len[i] = max(len[i], len[i + 1]);
        for (int i = 1; i <= A; i++)
            ans += len[i];
    }
    printInt128(ans);
    return 0;
}
```

（%%% [@Wilson_Lee](https://www.luogu.com.cn/user/513900) 学长，给出的解法比官方解法简单明了很多）

## `B`

概率和期望这一块学得很少，要补上。

另外考试时理解错题意了：“第 $i$ 轮工序的粉碎概率是 $p_i$”搞成了“质量为 $i$ 的物品的粉碎概率是 $p_i$”，导致把问题想得很复杂，实际上要简单得多。

深刻教训：发现一道题不可做的时候，首先应该多读两遍题确认没有理解错题意。

### Subtask 1：状压 DP

算期望往往可以转化为算概率，考虑计算 $n$ 个物品各自有多大的概率能够存活。设 $dp(s)$ 表示目前存活物品的状态的二进制表示为 $s$，出现这一状态的概率（在所有 $\operatorname*{popcount}(s)$ 相同的状态之中）。可以用刷表法：在已知当前状态为 $s$ 时选择一个物品 $j$ 粉碎，选中这个物品的概率为 $x$，就有 $dp(s \operatorname*{xor} (1<<j)) \operatorname*{+=} dp(s) \cdot x$。对于每个状态 $s$ 要把所有现存物品的 $x$ 预处理出来。答案就是 $\sum_{i=0}^{n-1}dp(1 << i) \times (i+1)$。

```c++
int n;
int p[MAXN];

int f[23]; // 粉碎物品i的概率
inline void calcF(int t, int s)
{
    vector<int> V;
    for (int i = 0; i < n; i++)
        if (s & (1 << i))
            V.push_back(i + 1);

    int roll = 1;
    for (int i = 0; i < V.size() - 1; i++)
    {
        f[V[i]] = (long long)p[t] * roll % MOD;
        roll = (long long)roll * (1 - p[t] + MOD) % MOD;
    }
    f[V.back()] = roll;
}

int dp[(1 << 20) + 5]; // 出现s状态的概率
vector<int> S[23];
inline void solve1()
{
    for (int s = 1; s < (1 << n); s++)
    {
        S[__builtin_popcount(s)].push_back(s);
    }
    dp[(1 << n) - 1] = 1;
    for (int i = n; i > 1; i--)
    {
        for (auto s : S[i])
        {
            int t = n - i + 1; // 当前是第t步工序
            calcF(t, s);
            for (int j = 0; j < n; j++)
            {
                if (s & (1 << j))
                    dp[s ^ (1 << j)] = (dp[s ^ (1 << j)] + (long long)dp[s] * f[j + 1] % MOD) % MOD;
            }
        }
    }
    int ans = 0;
    for (int i = 0; i < n; i++)
    {
        ans = (ans + (long long)dp[1 << i] * (i + 1) % MOD) % MOD;
    }
    printf("%d\n", ans);
}
```

### Subtask 2：AC

把每一个物品分开计算，假设目前关心的物品是 $a$，设 $dp(i,j)$ 表示在第 $i$ 道工序时 $a$ 的排名为 $j$ 的概率。

在工序推进（即 $i$ 增加）的过程中观察 $a$ 排名的变化，中途可能粉碎的是 $a$ 前面或者 $a$ 后面的物品。如果粉碎的数在 $a$ 的前面，当且仅当 $a$ 前面的数存在一个没能成功逃脱的，即 $dp(i+1,j-1) \operatorname*{+=} dp(i,j) \times (1-(1-p_i)^{j-1})$（所有情况减去 $a$ 前面的数全部成功逃脱的情况）；如果粉碎的物品在 $a$ 后面，当且仅当 $a$ 和前面的数都成功逃脱，即 $dp(i+1,j) \operatorname*{+=} dp(i,j) \times (1-p_i)^j$。

枚举 $a \in [1,n]$，初状态 $dp(1,a)=1$，累加 $dp(n,1) \cdot a$ 即为答案。

注意到幂次都是和 $j$ 有关的，没必要用快速幂，边算边乘即可。

```c++
int ans = 0;
for (int a = 1; a <= n; a++)
{
    for (int i = 1; i <= n; i++)
        for (int j = 1; j <= n; j++)
            dp[i][j] = 0;
    dp[1][a] = 1;
    for (int i = 1; i < n; i++)
    {
        int pPow = 1;
        for (int j = 1; j <= n; j++)
        {
            if (j > 1)
                dp[i + 1][j - 1] += (long long)dp[i][j] * (1 - pPow + MOD) % MOD;
            dp[i + 1][j] += (long long)dp[i][j] * pPow % MOD * (1 - p[i] + MOD) % MOD;
            pPow = (long long)pPow * (1 - p[i] + MOD) % MOD;
        }
    }
    ans = (ans + (long long)dp[n][1] * a % MOD) % MOD;
}
printf("%d\n", ans);
return 0;
```

## `C`

### Subtask 1：暴力

$O(n)$ 枚举 $k$，对于每个 $k$ 再 $O(n)$ 检查每个子区间（利用滑动窗口优化就不需要 $O(n^2)$ 扫）。总时间复杂度 $O(n^2)$。

### Subtask 2：非常奇妙但不知道对不对的做法

首先考虑完整的区间 $a[1,n]$，统计所有数字的出现个数。比较最低的次数和 $b_n$，如果大于等于，那么答案直接就是 $n$；否则考虑缩小区间。

因为要让所有数字出现次数都大于等于 $b_k$，那么那些出现次数小于 $b_k$ 的数就是不需要的。假设目前区间的两个端点为 $l,r$，一种很不正确的贪心做法就是比较两个端点上数字的出现次数，哪一端少就缩小哪一端（即 `l++` 或 `r--`），如果一样多就比较里面一层（$l+1$ 和 $r-1$），依次类推。如果比完了发现都是一样多，那就随便缩一端。缩完后再次检查，满足条件就输出答案 $k=r-l+1$，不满足就继续缩，如果缩完了还没发现合法区间那答案就是 $0$。

记录每个数字的出现次数可以直接用 `map`，但是这样检查的时候最坏会多一层 $O(n)$。可以改用支持单点修改、求全局最小的线段树维护，能够大大降低时间复杂度。

```c++

int n;
int arr[MAXN], brr[MAXN];

int sgtVal[MAXN * 4];
// 直接储存数字的出现个数，避免单点查询还要加个O(logn)
int cnt[MAXN];
// 单点修改
void sgtModify(int idx, int nl, int nr, int p, int x)
{
    if (nl == nr)
    {
        sgtVal[idx] += x;
        cnt[p] += x;
        return;
    }
    int mid = (nl + nr) >> 1;
    if (p <= mid)
        sgtModify(idx << 1, nl, mid, p, x);
    else
        sgtModify((idx << 1) | 1, mid + 1, nr, p, x);

    int vl = sgtVal[idx << 1];
    int vr = sgtVal[(idx << 1) | 1];
    // 上传信息时稍微处理一下，忽略0
    if (vl == 0 && vr != 0)
        sgtVal[idx] = vr;
    if (vl != 0 && vr == 0)
        sgtVal[idx] = vl;
    if (vl != 0 && vr != 0)
        sgtVal[idx] = min(vl, vr);
    if (vl == 0 && vr == 0)
        sgtVal[idx] = 0;
}

// 比较两端谁更优
inline bool endComp(int l, int r)
{
    while (l < r)
    {
        int vl = cnt[arr[l]], vr = cnt[arr[r]];
        if (vl < vr)
            return true;
        if (vl > vr)
            return false;
        l++, r--;
    }
    return true;
}
inline void solve2()
{
    // 注意a里面可能会出现0
    for (int i = 1; i <= n; i++)
        sgtModify(1, 0, n, arr[i], 1);
    int l = 1, r = n;
    while (l <= r)
    {
        // 全局最小值就是1号节点
        if (sgtVal[1] >= brr[r - l + 1])
        {
            printf("%d\n", r - l + 1);
            return;
        }
        if (endComp(l, r))
            sgtModify(1, 0, n, arr[l], -1), l++;
        else
            sgtModify(1, 0, n, arr[r], -1), r--;
    }
    printf("0\n");
}
```

这份代码有 90 分，因为比较两端的时候往里扫会导致最坏 $O(n^2)$。

至于正确性，暂时没想到任何合理的解释，对拍约 4000 组样例平均仅有 1 个答案错误 ~~（也有可能是因为数据太弱）~~。

## `D`
