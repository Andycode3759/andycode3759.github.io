---
title: 考试总结 - NOI 2023 春季测试模拟赛（重赛）
date: 2023-08-28 21:51:03 +0800
categories: [考试总结]
tags: [noi春测]
math: true
---

前情提要：[2023.3]({{ "/posts/exam-review-noi-2023-springtest" | relative_url }})

一模一样的题目。**再不订正就没面子了。**

## `power` - 幂次

### Subtask -1：朴素暴力（20%）

枚举 $x$，对于每个 $x$ 找到最小的 $a$ 满足 $a^b=x$ 且 $b \geq k$。找因子需要 $O(\sqrt n)$，拆解计算指数需要 $O(\sqrt n)$，总时间为 $O(n^2)$。

### Subtask 1：（稍微好一点的）暴力，$n \leq 10^8$（40%~65%）

考虑底数 $a$，从 $k$ 开始枚举所有可能的指数 $b$，直到 $a^b \geq n$ 退出并换到下一个 $a$。至于去重，把所有 $a^b$ 都打上标记，答案即为标记数。时间近似于多项式级别，瓶颈主要在于标记空间，`bitset` 压位后最多可以开到 $10^8$。使用 `set` 可以忽略这个问题（牺牲时间作为代价）。

实测 `bitset` 有 40 分，`set` 有 65 分。

```c++
long long n;
int k;

set<long long> vis;

int main()
{
    scanf("%lld %d", &n, &k);
    if (k == 1) // k=1记得特判
    {
        printf("%lld\n", n);
        return 0;
    }
    long long lim = powl(n, (long double)1.0 / k); // a不可能超过n^(1/k)，因为大于它的数的k次方一定会超过n
    vis.insert(1);
    for (long long a = 2; a <= lim; a++)
    {
        for (int b = k;; b++)
        {
            long long res = quickPow(a, b); // 其实这里用不用快速幂都无所谓，暴力乘的得分是一样的
            if (res <= n)
                vis.insert(res);
            if (res >= n)
                break;
        }
    }

    printf("%ld\n", vis.size());
    return 0;
}
```

### Subtask 2：AC（100%）

换一种思路：现在不考虑底数，转而考虑指数 $b$。对于一个确定的 $b$，所有的满足 $a^b \leq n$ 的 $a$ 的数量，我们记为 $g(b)$。不难发现，$g(b)=n^{1 \over b}$。那么答案就是 $\sum_{i}^{i \geq k}g(i)$ ，即所有 $g(i)$ 相加吗？并不是，因为一个数可能有多种不同的表示方式，对应了不同的 $a$ 和 $b$，例如 $2^4=4^2$。显然这些 $b$ 会成倍数关系。那么，在所有数都不超过 $n$ 的情况下，我们可以指定一个数只能被最小的那个 $b$ 表示，在统计答案时把 $b$ 的倍数所对应的 $g$ 值全部减去就好了。

```c++
long long ans = 1;  // 包括了1^k这个数
for (int b = MAXK - 1; b >= k; b--)
{
    f[b] = powl(n, (long double)1.0 / b) - 1; // 要把a=1的情况减掉
    for (int m = b * 2; m < MAXK; m += b)
        f[b] -= f[m];
    ans += f[b];
}
```

## `tree` - 圣诞树

### Subtask 1：暴力，$n \leq 9$（30%）

孩子终于会用 `next_permutation` 了，可喜可贺。

### Subtask 2：特殊性质 B（+10%）

所有点呈单调的“下坡”形，可以证明从最高点一路连到最低点一定是最优方案。按顺序输出 $1$ 到 $n$ 即可。

### Subtask 3：稍微好一点的暴力，$n \leq 18$（55%）

通过三角形不等式可以证明，最短线路一定不会交叉。那么可以这样考虑：以最高点和最低点的连线把整个图形分成两部分，从最高点开始，每一步有两个选择，要么往下连一个点，要么跑道对面连一个最高的还没连的点。于是就可以写 `dfs` 了，时间为 $O(2^n)$，期望得分 60。需要注意某一边连到最低处时还可以考虑连到对面最低点的选择。

```c++
using Point = pair<double, double>;
const int MAXN = 1003;
const double EPS = 1e-10;

inline double distance(Point a, Point b)
{
    return sqrt((a.first - b.first) * (a.first - b.first) + (a.second - b.second) * (a.second - b.second));
}

Point pt[MAXN];
int n, ans[MAXN], perm[MAXN], k, low;
bool pointCmp(int a, int b)
{
    if (abs(pt[a].second - pt[b].second) <= EPS)
        return a < b;
    else
        return pt[a].second > pt[b].second;
}

int optL[MAXN], optR[MAXN];
int cntL = 0, cntR = 0;
double best = 1e100;

void dfs(int l, int r, double sum)
{
    int step = l + r + 1;

    // 往左走
    if (l < cntL)
    {
        perm[step + 1] = optL[l + 1];
        dfs(l + 1, r, sum + distance(pt[perm[step]], pt[perm[step + 1]]));
    }
    // 往右走
    if (r < cntR)
    {
        perm[step + 1] = optR[r + 1];
        dfs(l, r + 1, sum + distance(pt[perm[step]], pt[perm[step + 1]]));
    }

    if (l == cntL && r == cntR) // 连完了全部的点
    {
        if (sum < best)
        {
            best = sum;
            for (int i = 2; i <= n; i++)
                ans[i] = perm[i];
        }
        return;
    }
    else if (l == cntL) // 左边的连完了，考虑连右边最下面的点
    {
        step++;
        for (int i = cntR; i > r; i--, step++)
        {
            perm[step] = optR[i];
            sum += distance(pt[perm[step - 1]], pt[perm[step]]);
        }
        if (sum < best)
        {
            best = sum;
            for (int i = 2; i <= n; i++)
                ans[i] = perm[i];
        }
        return;
    }
    else if (r == cntR) // 右边的连完了，考虑连左边最下面的点
    {
        step++;
        for (int i = cntL; i > l; i--, step++)
        {
            perm[step] = optL[i];
            sum += distance(pt[perm[step - 1]], pt[perm[step]]);
        }
        if (sum < best)
        {
            best = sum;
            for (int i = 2; i <= n; i++)
                ans[i] = perm[i];
        }
        return;
    }
}

int main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
    {
        scanf("%lf %lf", &pt[i].first, &pt[i].second);
        perm[i] = i;
    }

    sort(perm + 1, perm + 1 + n, pointCmp);
    // 最高点和最低点
    k = perm[1], low = perm[n];
    ans[1] = k;

    // 生成左边点和右边点的选择序列
    // 注意1号点不一定是最低点，输入仅能保证1到n顺时针排列
    int i;
    for (i = k - 1; i >= 1 && i >= low; i--)
        optL[++cntL] = i;
    if (i == 0)
        for (i = n; i >= 1 && i >= low; i--)
            optL[++cntL] = i;
    for (i = k + 1; i <= n; i++)
        optR[++cntR] = i;
    if (i == n + 1)
        for (i = 1; i <= n && i < low; i++)
            optR[++cntR] = i;

    dfs(0, 0, 0);

    for (int i = 1; i <= n; i++)
        printf("%d ", ans[i]);
    printf("\n");

    return 0;
}
```

或许是某些细节出了问题，实际得分只有 55。

### Subtask 4：AC（100%）

有决策的地方就有 DP。设 $dp(l,r,ex)$ 表示已经向左取了 $l$ 个点、向右取了 $r$ 个点，当前位于左（$ex=0$）/右（$ex=1$）侧的最佳方案。每一个位置有四种情况、两种决策：

- 位于左边，从上面的一个点拉下来（$dp(l,r,0) \to dp(l+1,r,0)$）；
- 位于左边，从对面的一个点拉过来（$dp(l,r,1) \to dp(l+1,r,0)$）；
- 位于右边，从上面的一个点拉下来（$dp(l,r,1) \to dp(l,r+1,1)$）；
- 位于右边，从对面的一个点拉过来（$dp(l,r,0) \to dp(l,r+1,1)$）；

用区间 DP 的方法枚举长度和起点即可求解。

```c++
for (int i = 0; i <= n; ++i)
{
    for (int j = 0; j <= n; ++j)
    {
        dp[0][i][j] = dp[1][i][j] = INF;
    }
}
dp[0][0][0] = dp[1][0][0] = 0;

for (int len = 1; len < n; ++len)
{
    for (int l = 0; l <= len; ++l)
    {
        int r = len - l;
        double t;
        // 左边
        if (l != 0)
        {
            // 从上面一个点拉过来
            t = dp[0][l - 1][r] + distance(pt[pointMod(k + l)], pt[pointMod(k + l - 1)]);
            if (dp[0][l][r] > t)
            {
                dp[0][l][r] = t;
                fa[0][l][r] = Node(l - 1, r, 0);
            }
            // 从对面的上一个点拉过来
            t = dp[1][l - 1][r] + distance(pt[pointMod(k + l)], pt[pointMod(k - r)]);
            if (dp[0][l][r] > t)
            {
                dp[0][l][r] = t;
                fa[0][l][r] = Node(l - 1, r, 1);
            }
        }
        // 右边
        if (r != 0)
        {
            // 从对面的上一个点拉过来
            t = dp[0][l][r - 1] + distance(pt[pointMod(k - r)], pt[pointMod(k + l)]);
            if (dp[1][l][r] > t)
            {
                dp[1][l][r] = t;
                fa[1][l][r] = Node(l, r - 1, 0);
            }
            // 从上面一个点拉过来
            t = dp[1][l][r - 1] + distance(pt[pointMod(k - r)], pt[pointMod(k - r + 1)]);
            if (dp[1][l][r] > t)
            {
                dp[1][l][r] = t;
                fa[1][l][r] = Node(l, r - 1, 1);
            }
        }
    }
}

// 寻找最优方案的头和尾
Node head;
double best = INF;
for (int i = 0; i < n; i++)
{
    if (dp[0][i][n - i - 1] < best)
    {
        best = dp[0][i][n - i - 1];
        head = Node(i, n - i - 1, 0);
    }
    if (dp[1][i][n - i - 1] < best)
    {
        best = dp[1][i][n - i - 1];
        head = Node(i, n - i - 1, 1);
    }
}
ans[1] = k;
// 记录路径
for (int i = n; i > 1; i--)
{
    ans[i] = (head.ex == 0) ? pointMod(k + head.l) : pointMod(k - head.r);
    head = fa[head.ex][head.l][head.r];
}

for (int i = 1; i <= n; i++)
    printf("%d ", ans[i]);
printf("\n");
```

### 类似题目

- [[ABC273F] Hammer 2](https://www.luogu.com.cn/problem/AT_abc273_f)

## `lock` - 密码锁

### Subtask 1：$k \leq 2$（30%）

- $k=1$：只有一行，直接算就行。
- $k=2$：既然要让极差最小化，那就把较小的数放上面，较大的数放下面。

### Subtask 2：暴力，$n \leq 10$（10%）

枚举每一个拨圈的拨动格数，时间 $O(k^n)$。

### Subtask 3：$k=3$（25%）

### Subtask 4：$k=4$（35%）
