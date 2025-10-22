---
title: 题解 - 谱面设计（Hard Ver.）
date: 2024-02-19 20:00:34 +0800
categories: [题解]
tags: [原创题]
math: true
---

题面见此：[U389482 谱面设计（Hard Ver.）](https://www.luogu.com.cn/problem/U389482)

idea：2022 清华大学强基计划招生（的某一道题），见于 2023 版高中必刷题（北师大数学选修一）第五章刷素养部分。

为什么要标一个 Hard Version 呢？因为本题原本的 100% 数据范围只有 $n \leq 10^4$，后面想到了 $O(n)$ 的做法后就把数据加强了。这道题的 Eazy Version 在[济南]({{ "/posts/journal-biodo-oi-jinan" | relative_url }})的时候给同学做过（感谢 [@qianxinyu](https://www.luogu.com.cn/user/715225) 提供了一种 $O(n)$ 的做法，应该和这里的满分做法是等价的）。

为了方便，下文称放 Note 为给盒子涂色，红表示 Drag，蓝表示 Tap（其实这也是原题的表述方法）。

## Subtask 1：$n \leq 20$（20%）

二进制枚举状态，$0$ 表示红，$1$ 表示蓝，然后直接 check 即可。复杂度 $O(n \cdot 2^n)$。

```c++
int ans = 0;
for (int s = 0; s < (1 << n); s++)
{
    int combo = 0;
    bool flag = true;
    for (int i = 0; i < n; i++)
    {
        if (s & (1 << i))
            combo++;
        else
            combo = 0;
        if (combo > m)
        {
            flag = false;
            break;
        }
    }
    if (flag)
        ans++;
}
printf("%d\n", ans);
```

## Subtask 2：$n \leq 2000$（50%）

二维暴力 DP。设 $dp(i,j)$ 表示涂到第 $i$ 个盒子时，有 $j$ 个连续的蓝色盒子后缀（$j=0$ 表示第 $i$ 个盒子是红色）。

每一个位置有两个决策，涂红或涂蓝。涂红只能转移到 $dp(i,0)$，此时就是把前面 $i-1$ 个盒子的所有方案累加；涂蓝就会让后缀 $+1$，即从 $(i,j)$ 转移到 $(i+1,j+1)$。由此不难得到转移方程：

$$
\begin{align*}
dp(i,0) &= \sum_{k=0}^{m}dp(i-1,k)\\
dp(i,j) &= dp(i-1,j-1)\ (j>0)
\end{align*}
$$

时间和空间均为 $O(n^2)$。

```c++
int dp[MAXN][MAXN];

for (int i = 1; i <= m; i++)
    dp[i][i] = 1;
dp[0][1] = 1;
for (int i = 2; i <= n; i++)
{
    for (int k = 0; k <= m; k++)
    {
        dp[0][i] = (dp[0][i] + dp[k][i - 1]) % MOD;
    }
    for (int j = 1; j <= m; j++)
    {
        if (i == j)
            continue;
        dp[j][i] = (dp[j][i] + dp[j - 1][i - 1]) % MOD;
    }
}

int ans = 0;
for (int j = 0; j <= m; j++)
    ans = (ans + dp[j][n]) % MOD;
printf("%d\n", ans);
```

## Subtask 3：$n \leq 10^7$（100%）

设 $f_i$ 为涂到第 $i$ 个盒子时的合法方案数。按照 $i$ 的大小分类讨论：

1. 当 $i \leq m$ 时：显然怎么涂都可以，有 $f_i = 2^i\ (i \leq m)$；
2. 当 $i=m+1$ 时：有且仅有一种不合法的方案，就是 $m+1$ 个盒子全涂蓝，有 $f_{m+1}=2^{m+1}-1$；
3. 当 $i>m+1$ 时：考虑递推，利用 $f_{i-1}$ 求出 $f_i$。首先该位置有两种决策，涂红或涂蓝，但是涂蓝可能会出问题，因为在 $f_{i-1}$ 中存在后缀有连续 $m$ 个蓝盒子的方案，我们需要把这些方案减去。要减多少呢？如果规定后缀必须有连续 $m$ 个蓝盒子的话，那么倒数第 $m+1$ 个盒子也一定是红的，即需要减去 $f_{i-m-2}$ 的方案。所以有 $f_i=2 \cdot f_{i-1} - f_{i-m-2}\ (i>m+1)$。

根据以上规律直接递推计算即可，复杂度 $O(n)$。别忘了 $f_0=1$。

```c++
int dp[MAXN];

dp[0] = 1;
for (int i = 1; i <= n; i++)
{
    dp[i] = 2 * dp[i - 1] % MOD;
    if (i == m + 1)
        dp[i]--;
    else if (i > m + 1)
        dp[i] = (dp[i] - dp[i - m - 2] + MOD) % MOD;
}
printf("%d\n", dp[n]);
```
