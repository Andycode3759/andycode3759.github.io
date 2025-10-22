---
title: 考试总结 - 信友队 CSP-S 2023 复赛模拟赛
date: 2023-10-19 17:31:31 +0800
categories: [考试总结]
tags: [信友队]
math: true
---

[Link](https://contest.xinyoudui.com/contest/145)

~~我超，源！~~

## `firstsnow` - First Snow

考虑枚举一个正方形的外接正方形，大概长这样：

![](https://cdn.luogu.com.cn/upload/image_hosting/4goavpk9.png)

那么算到的真实正方形就是 $(x,y+d),(x+d,y+a),(x+a,y+a-d),(x+a-d,y)$。依次枚举 $x,y,a,d$，时间复杂度 $O(n^4)$。

```c++
inline void solve()
{
    ll cnt = 0;
    for (int x = 1; x <= n; ++x)
    {
        for (int y = 1; y <= n; ++y)
        {
            int m = max(x, y);
            for (int a = 1; a <= n - m; ++a)
            {
                for (int d = 0; d < a; ++d)
                {
                    if (G[x][y + d] && G[x + d][y + a] && G[x + a][y + a - d] && G[x + a - d][y])
                        ++cnt;
                }
            }
        }
    }
    printf("%lld\n", cnt);
}
```

当时就在想有没有更优的类似 $O(n^3)$ 的做法，比如针对 $a$ 或 $d$ 层的枚举进行预处理，但是推了半天没推出来。其实正解的本质还是 $O(n^4)$，而且也用到了预处理掉一层枚举的思想，只不过处理的是 $y$ 层。用 `bitset` 储存矩阵，再把从每一个点开始的往右的后继数列保存下来，这样就可以不必枚举 $y$ 了，相当于在最左边枚举后利用位运算一次性处理一行。$y$ 层的枚举就变成了接近常数的位运算操作，时间降为 $O(n^3)$（严格来说应该是 $O({ {n^4} \over w})$）。

（注：位运算中 `>>` 和 `<<` 应该分别理解为“向低位移动”和“向高位移动”，如果只记“右移”和“左移”的话很容易把 `bitset` 的位移搞混！）

```c++
for (int x = 1; x <= n; x++)
{
    for (int y = 1; y <= n; y++)
    {
        tail[x][y] = G[x] >> y;
    }
}

inline void solve()
{
    ll cnt = 0;
    for (int x = 1; x <= n; ++x)
    {
        for (int a = 1; a <= n - x; ++a)
        {
            for (int d = 0; d < a; ++d)
            {
                cnt += (tail[x][d + 1] & tail[x + d][a + 1] & tail[x + a][a - d + 1] & tail[x + a - d][1]).count();
            }
        }
    }
    printf("%lld\n", cnt);
}
```

其实赛时想法已经非常接近正解了，但是在优化时只拘泥于考虑 $a$ 和 $d$ 层的优化，有时做题确实需要跳出定式思考。

顺便说一下 $o=4$ 的部分分推法。考虑最原始的暴力做法，发现最里面的 `if` 是无关紧要的（无论如何一定会 `cnt++`），那么也就等价于求

$$
\sum_{x=1}^{n}\sum_{y=1}^{n}\sum_{a=1}^{n-max\{x,y\} }\sum_{d=0}^{a-1}1
$$

把最里面一层化简就是

$$
\sum_{x=1}^{n}\sum_{y=1}^{n}\sum_{a=1}^{n-max\{x,y\} } {a(a+1) \over 2}
$$

我们取比较小的几个 $n$，再把对于每个 $(x,y)$ 的 $a$ 的最大值写出来，形成一个矩阵 $M_n$，大概长这样：

![](https://cdn.luogu.com.cn/upload/image_hosting/jkq9p6te.png)

容易发现会形成从左上角开始一层层扩散递减的形状。根据扩散的过程可以得到：

$$
\sum{M_n} = \sum_{i=1}^{n-1}(2i-1)(n-i)
$$

记答案为 $f(n)$，那么 $f(n)$ 就相当于 $i$ 取 $[2,n]$ 时所有矩阵 $M_i$ 的所有元素之和，即 $f(n)=\sum_{i=2}^{n}\sum{M_i}$。直接代入计算即可，时间为 $O(n^2)$。

## `paradise` - Paradise

### Subtask 1：40%

可以直接套一个完全背包模板。出题人说有 40 分，但是我直接写了 Subtask 2，所以就相信他说的吧。

### Subtask 2：60%

解法一：根据“$a_{i}$ 是 $a_{i+1}$ 的因数”这一条件，可以把相同的 $a$ 全部去重，只取 $b$ 最小的；之后再逐一检查，对于 $a_{i+1}$ 的性价比不如 $a_{i}$ 的方案也可以舍去。如此处理一番之后再套背包就有 60 分。

解法二：还是按照上面的过程预处理，但是对于每一个 $n$ 贪心计算：先取最大的 $a_{x}$，取不下了再取次大的 $a_{x-1}$，直到刚好取完。因为预处理已经保证了 $a$ 与性价比是成正相关的，因此贪心正确。时间只需要 $O(n)$，比解法一优秀一点。

### Subtask 3：100%

对于 $a,b \leq 10^{18}$，背包做法已经走到头了，考虑继续优化解法二。解法二的瓶颈无非在于求和（如果只是单点询问的话已经可以过了），我们先考虑能不能优化单个的 $f(n)$ 的计算过程。假设剩下的最优选法有 $m$ 个，实际上 $f(n)$ 是可以写出解析式的：

$$
f(n)=b_m \lfloor {n \over a_m} \rfloor + \sum_{j=1}^{m-1}b_j \lfloor { {n \bmod a_{j+1} } \over {a_j} } \rfloor
$$

这也很好理解，取商品的过程就是不断取余数的过程。接下来再稍微发挥一下数学功底：

![](https://cdn.luogu.com.cn/upload/image_hosting/b7rbv1d7.png)

这串式子太长了，我们记 $g(x)=b_x - [x>1]{ {b_{x-1}a_x} \over { a_{x-1} } }$，对 $f(n)$ 进行化简：

$$
f(n)=\sum_{j=1}^{m}g(j) \lfloor {n \over {a_j} } \rfloor
$$

然后求答案：

$$
ans = \sum_{i=0}^{n-1}f(i) = \sum_{j=1}^{m}g(j)\sum_{i=0}^{n-1} \lfloor {n \over {a_j} } \rfloor
$$

之后就是最关键的步骤，需要用到这个性质：

$$
\sum_{i=0}^{n-1} \lfloor {n \over {a_j} } \rfloor = { {\lfloor {n / a} \rfloor (2n - a \lfloor n/a \rfloor - a)} \over 2}
$$

这样就少了一层求和，时间就只与 $m$ 有关了。

顺便一提，本题~~非常恶心地~~要求对 $2^{64}$ 取模，但是如果用 `unsigned long long` 自然溢出就挂了，正确做法应该是在除法时特殊处理一下（好在除数是 $2$ 的幂，只需要判断奇偶就行，不需要算逆元），或者用 `__int128` 并在输出答案时手动取余（尤其是对于背包做法只能这样做，否则取余后不能保证单调性）。

## `neokosmo` - nέο κόsmo

## `nirvluce` - Nirv lucE
