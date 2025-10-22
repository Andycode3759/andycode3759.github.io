---
title: 考试总结 - 2023 牛客 OI 赛前集训营-提高组（第三场）
date: 2023-10-09 17:09:01 +0800
categories: [考试总结]
tags: [牛客]
math: true
---

题目偏简单，T1 终于给了一道真正的签到题，T2 T3 也偏数据结构套路，只有 T4 具有一定的区分度。但是实际结果反映出，T2 T3 得分偏低，主要就是因为数据结构相关的基本功、二分答案的基本套路还有欠缺。

## `A` - 填数游戏

非常简单的一个贪心。将所有空位的权值排序，分成正数和负数，分别按绝对值从大到小处理；如果是正数，那就在可选数里选一个最大的正数或绝对值最小的负数；如果是负数，那就在可选数里选一个绝对值最大的负数或绝对值最小的正数。

考试里第一眼看成 DP 了，导致浪费了过多时间。这些时间本来可以花在 T2 和 T3 上拿更多分的。

## `B` - 摆渡车

### Subtask 1：暴力（50%）

显然每次选人坐车时优先选重量最小的。用一个数组记录目前经过的所有人的体重，并且用暴力插入的方法维护其单调性，这样这个数组就能和优先队列一样用了，统计答案直接暴力计算即可。

```c++
int arr[MAXN];
inline int getCnt(int x, int vol)
{
    for (int i = 1; i < x; i++)
    {
        vol -= arr[i];
        if (vol < 0)
            return i;
    }
    return x;
}
inline void solve1()
{
    printf("0 ");
    arr[1] = wei[1];
    for (int i = 2; i <= n; i++)
    {
        printf("%d ", i - getCnt(i, m - wei[i]));
        arr[i] = wei[i];
        int j = i;
        while (j > 1 && arr[j - 1] > arr[j])
            swap(arr[j - 1], arr[j]), j--;
    }
    printf("\n");
}
```

如果不考虑贪心，那么硬套一个 01 背包上去也是可以拿到前 50 分的：

```c++
int dp[MAXN];
inline void solve2()
{
    for (int i = 1; i <= m; i++)
        dp[i] = 0;
    printf("0 ");
    for (int i = 1; i < n; i++)
    {
        for (int j = m; j >= wei[i]; j--)
        {
            dp[j] = max(dp[j], dp[j - wei[i]] + 1);
        }
        printf("%d ", i - dp[m - wei[i + 1]]);
    }
    printf("\n");
}
```

### Subtask 2：更好的暴力（70%）

取最小值？这不就是优先队列/`set` 擅长干的吗？只需要把上面做法的暴力维护改成用一个 `multiset` 就可以直接多 20 分了：

```c++
multiset<int> S;
inline int getCnt(int vol)
{
    int cnt = S.size();
    for (int p : S)
    {
        if (vol >= p)
            vol -= p, cnt--;
        else
            break;
    }
    return cnt;
}
inline void solve1()
{
    S.clear();
    for (int i = 1; i <= n; i++)
    {
        printf("%d ", getCnt(m - wei[i]));
        S.insert(wei[i]);
    }
    printf("\n");
}
```

### Subtask 3：AC（100%）

解题的关键就在于怎么优化 `getCnt` 这个操作，即快速算出 $[1,x-1]$ 区间中至多多少个元素之和小于等于 $m-w_x$。可以使用权值线段树，每个节点保存区间内数字个数和它们的总和，然后就可以加快查询和更新过程了：

```c++
int n, m;
int wei[MAXN], wei2[MAXN];
int tot;

struct SegTreeNode
{
    int cnt;
    long long sum;
};
SegTreeNode stree[MAXN << 2];
// 上传信息
inline void pushUp(int nd)
{
    stree[nd].cnt = stree[nd << 1].cnt + stree[(nd << 1) | 1].cnt;
    stree[nd].sum = stree[nd << 1].sum + stree[(nd << 1) | 1].sum;
}
// 与其说是建树，不如说是清空线段树信息
void build(int nd, int l, int r)
{
    stree[nd].cnt = stree[nd].sum = 0;
    if (l == r)
        return;
    int mid = (l + r) >> 1;
    build(nd << 1, l, mid);
    build((nd << 1) | 1, mid + 1, r);
    pushUp(nd);
}
// 添加一个数x
void insert(int nd, int l, int r, int x)
{
    if (l == r)
    {
        stree[nd].cnt++;
        stree[nd].sum += wei2[x];
        return;
    }
    int mid = (l + r) >> 1;
    if (x <= mid)
        insert(nd << 1, l, mid, x);
    else
        insert((nd << 1) | 1, mid + 1, r, x);
    pushUp(nd);
}
// 查询l到r中有最多有多少个数的总和小于等于x
int query(int nd, int l, int r, int x)
{
    if (l == r)
    {
        // 对于叶子节点，x%wei2[l]==0恒成立，
        // 即要么把x撑满，要么cnt个数全上
        return min(x / wei2[l], stree[nd].cnt);
    }
    int mid = (l + r) >> 1;
    // 因为单调性，左儿子的和肯定更小，且方案会更优
    // 所以如果x比左儿子的和还小就去左儿子查
    if (x <= stree[nd << 1].sum)
        return query(nd << 1, l, mid, x);
    // 否则可以把左儿子全部拿上，剩下的值再放到右儿子里查
    else
        return stree[nd << 1].cnt + query((nd << 1) | 1, mid + 1, r, x - stree[nd << 1].sum);
}
inline void solve1()
{
    // 离散化，不过因为实际的wei值还是需要用到，所以需要复制一份
    sort(wei2 + 1, wei2 + 1 + n);
    tot = unique(wei2 + 1, wei2 + 1 + n) - wei2 - 1;
    for (int i = 1; i <= n; i++)
    {
        wei[i] = lower_bound(wei2 + 1, wei2 + 1 + tot, wei[i]) - wei2;
    }

    build(1, 1, tot);
    for (int i = 1; i <= n; i++)
    {
        printf("%d ", i - 1 - query(1, 1, tot, m - wei2[wei[i]]));
        insert(1, 1, tot, wei[i]);
    }
    printf("\n");
}
```

## `C` - 分糖果

### Subtask 1：$K=1$（20%）

送分，直接取前缀和的最小值即可。

### Subtask 2：$N \leq 100$（50%）

这种二分答案 + DP 检验的题目应该算是比较套路了，考验问题建模的基本功。二分答案就是检验最大值，即验证一个 $x$ 能否使得在子段和不超过 $x$ 的情况下在最前面分出 $K$ 个子段。但是具体取前多少个数是不确定的，所以需要进行 DP：设 $dp(i)$ 表示前 $i$ 个数在子段和不超过 $x$ 的情况下，最多可以分成多少段。如果**存在** $dp(i) \geq k$ 那么这个 $x$ 就是可行的。考虑暴力转移，对于每一个 $i$ 都 $O(n)$ 枚举最后一个子段，取最大值（没有可行方案为 $0$）：

```c++
int dp[MAXN];
inline bool check(ll lim)
{
    for (int i = 1; i <= n; i++)
    {
        dp[i] = 0;
        for (int j = 0; j < i; j++)
        {
            // j==0 即只分一段，要求dp[j]>0即存在以j结尾的方案
            // 这一段的子段和在lim以内(sum[i]-sum[j]<=lim)就可以分一段出来
            if ((j == 0 || dp[j] > 0) && sum[i] - sum[j] <= lim)
                dp[i] = max(dp[i], dp[j] + 1);
        }
        if (dp[i] >= k)
            return true;
    }
    return false;
}

inline void solve1()
{
    ll l = -INF, r = INF, mid = (l + r) >> 1, ans = 0;
    while (l <= r)
    {
        mid = (l + r) >> 1;
        if (check(mid))
        {
            ans = mid;
            r = mid - 1;
        }
        else
        {
            l = mid + 1;
        }
    }
    printf("%lld\n", ans);
}
```

仔细观察一下，满足条件的状态都需要满足一个式子：$sum_i-sum_j \leq lim$。对于确定的一个 $i$，$sum_i$ 和 $lim$ 都是确定的，所以把这个式子变下形就得到了：$sum_j \geq sum_i-lim$，即找到 $sum$ 数组排序后大于等于 $sum_i-lim$ 的部分。这很自然地让人联想到 `multiset`：

```c++
using Node = pair<ll, int>;
int dp[MAXN];
multiset<Node, greater<Node>> S;
inline bool check(ll lim)
{
    S.clear();
    S.insert({0, 0});
    for (int i = 1; i <= n; i++)
    {
        dp[i] = 0;
        for (auto p : S)
        {
            if (p.first < sum[i] - lim)
                break;
            if ((p.second == 0 || dp[p.second] > 0))
                dp[i] = max(dp[i], dp[p.second] + 1);
        }
        if (dp[i] >= k)
            return true;
        S.insert({sum[i], i});
    }
    return false;
}
```

但可惜基于平衡树的 `multiset` 还是太慢，还是只有 50 分。拿到满分需要找到更快的状态转移方法。

### Subtask 3：AC（100%）

再考虑状态转移的过程，无论用不用 `multiset` 优化，其本质都是 $O(n)$ 的。而且，在 `dp[i]=max(dp[i],dp[j]-1)` 这一过程中实际上做了很多无用功，因为只需要找到一个最大的 `dp[j]` 再加上 $1$ 就能直接得到新状态了。那么可以考虑用某种数据结构来维护 $dp$ 数组，支持查询 $dp$ 的最大值。

我们先把前缀和数组复制一份，排个序，这样每次转移时的有效 $j$ 就是 $[某个点,n]$ 的一段区间，这里的 $某个点$ 可以用 `lower_bound` 查 $sum_i-lim$ 直接查出来。我们的目标就是在这段区间对应的 $dp$ 数组找到一个最大值。注意到这个区间一定是以 $n$ 结尾的，可以用后缀树状数组来维护最大值，方便快捷：

```c++
ll fw[MAXN];
inline void fwUpdate(int pos, ll x)
{
    for (int i = pos; i > 0; i -= (i & (-i)))
        fw[i] = max(fw[i], x);
}
inline ll fwQuery(int pos)
{
    ll res = -INF;
    for (int i = pos; i <= n; i += (i & (-i)))
        res = max(res, fw[i]);
    return res;
}
inline void fwClear()
{
    for (int i = 1; i <= n; i++)
        fw[i] = 0;
}

int dp[MAXN];
inline bool check(ll lim)
{
    fwClear();
    for (int i = 1; i <= n; i++)
    {
        // bd就是待查区间的起点
        int bd = lower_bound(sum2 + 1, sum2 + 1 + n, sum[i] - lim) - sum2;
        // 如果没找到，那就只能[1,i]单独分一段（对应暴力里的j=0）
        if (bd < 1 || bd > n)
            dp[i] = sum[i] <= lim ? 1 : 0;
        else
        {
            // 直接取出已知dp数组里的最大值，加个1
            ll mx = fwQuery(bd);
            if (mx != 0)
                dp[i] = mx + 1;
            else
                // 如果之前都没有可行的方案，那还是只能[1,i]单独分一段
                dp[i] = sum[i] <= lim ? 1 : 0;
        }
        if (dp[i] >= k)
            return true;
        // 更新dp信息，插入这个sum[i]所对应的dp值
        fwUpdate(arr[i], dp[i]);
    }
    return false;
}

inline void solve1()
{
    sort(sum2 + 1, sum2 + 1 + n);
    // 后续不需要用到原始数据了，这里拿arr来保存sum值离散化后对应的下标
    for (int i = 1; i <= n; i++)
        arr[i] = lower_bound(sum2 + 1, sum2 + 1 + n, sum[i]) - sum2;
    ll l = -INF, r = INF, mid = (l + r) >> 1, ans = 0;
    while (l <= r)
    {
        mid = (l + r) >> 1;
        if (check(mid))
        {
            ans = mid;
            r = mid - 1;
        }
        else
        {
            l = mid + 1;
        }
    }
    printf("%lld\n", ans);
}
```

这题的关键就在于，理解 $sum$ 到下标、下标到 $dp$ 的映射关系，从而建立 $sum$ 到 $dp$ 的一一对应关系，并以此为基础设计数据结构。当然，一切的一切都建立在构建出最基础的二分答案 + DP 检验的解题套路，而实际考试中也是因为第一步没走好而吃了不少亏。

## `D` - 宝石加工

### Subtask 1：暴力（20%）

暴力处理每个询问，DP 过程非常容易构建：设 $dp(i,j)$ 为左边处理到第 $i$ 个、右边处理到第 $j$ 个的最大获利，每一步有卖左边、卖右边、合成两个原料共三种决策：

```c++
int n, m, q;
int valA[MAXN], valB[MAXN];
int ptA[MAXN], ptB[MAXN];
// cost需要动态开，不然会爆内存
vector<int> cost[MAXN];

ll dp[1003][1003];
inline ll solveSub(int la, int ra, int lb, int rb)
{
    memset(dp, 0, sizeof dp);
    // 两个状态都需要从l-1开始，因为要考虑最开头的一个单独卖掉的决策
    for (int i = la - 1; i <= ra; i++)
    {
        for (int j = lb - 1; j <= rb; j++)
        {
            ll res = 0;
            if (i >= la) // 卖左边
                res = max(res, dp[i - 1][j] + ptA[i]);
            if (j >= lb) // 卖右边
                res = max(res, dp[i][j - 1] + ptB[j]);
            if (i >= la && j >= lb) // 合成
                res = max(res, dp[i - 1][j - 1] + valA[i] + valB[j] - cost[i][j]);
            dp[i][j] = res;
        }
    }
    return dp[ra][rb];
}
inline void solve1()
{
    while (q--)
    {
        int la, ra, lb, rb;
        la = fastRead(), ra = fastRead(), lb = fastRead(), rb = fastRead();
        printf("%lld\n", solveSub(la, ra, lb, rb));
    }
}
```

### Subtask 2：$n=1$（+24%）

只需要考虑左边的一个原料是放上去合成还是单独卖掉就好了。先对于右边的每个原料，处理出其与左边原料合成能够**增加**多少获利（也有可能是亏损），然后对于每个询问取增利最大的那个原料，再和单独卖掉左边原料的增利相比较。区间询问最值自然用 ST 表：

```c++
int lg2[MAXN];
ll sum[MAXN];
ll stMax[17][MAXN];
inline ll stQuery(int l, int r)
{
    int lg = lg2[r - l + 1];
    return max(stMax[lg][l], stMax[lg][r - (1 << lg) + 1]);
}
inline void solve2()
{
    // 预处理lg2并初始化st表，顺便求一个前缀和
    lg2[1] = 0;
    for (int i = 2; i < MAXN; i++)
        lg2[i] = lg2[i >> 1] + 1;
    for (int i = 1; i <= m; i++)
    {
        sum[i] = sum[i - 1] + ptB[i];
        stMax[0][i] = valA[1] + valB[i] - cost[1][i] - ptB[i];
    }
    for (int lg = 1; lg < 17; lg++)
    {
        for (int i = 1; i + (1 << (lg - 1)) <= m; i++)
        {
            stMax[lg][i] = max(stMax[lg - 1][i], stMax[lg - 1][i + (1 << (lg - 1))]);
        }
    }
    // 实际回答询问的部分，其实非常简单
    while (q--)
    {
        int la, ra, lb, rb;
        la = fastRead(), ra = fastRead(), lb = fastRead(), rb = fastRead();
        ll ans = sum[rb] - sum[lb - 1] + max(stQuery(lb, rb), (ll)ptA[1]);
        printf("%lld\n", ans);
    }
}
```

### Subtask 3：暴力优化（64%）

其实，只要把暴力做法的 DP 过程稍微优化一下就可以多 20 分了。原始的 DP 过程之所以慢，就是因为在循环中用了三个 `if` 语句，而这也是会给运算带来负担的。我们需要把所有的 `if` 语句去掉，可以把 $dp(l_1-1,j)$ 和 $dp(i,l_2-1)$ 的初状态全部算出来，再直接 DP 就不需要用 `if` 判断了：

```c++
for (int j = lb; j <= rb; j++)
    dp[la - 1][j] = dp[la - 1][j - 1] + ptB[j];
for (int i = la; i <= ra; i++)
{
    dp[i][lb - 1] = dp[i - 1][lb - 1] + ptA[i];
    for (int j = lb; j <= rb; j++)
    {
        dp[i][j] = dp[i - 1][j - 1] + valA[i] + valB[j] - cost[i][j];
        dp[i][j] = max(dp[i][j], dp[i - 1][j] + ptA[i]);
        dp[i][j] = max(dp[i][j], dp[i][j - 1] + ptB[j]);
    }
}
return dp[ra][rb];
```

### ~~Subtask -1：骗分（+8%）~~

~~不考虑合成，把两列原料全部卖掉，可以多得 8 分。（造数据出题人背锅）~~
