---
title: 考试总结 - 云斗 NOIP 2023 赛前模拟赛（vp）
date: 2023-11-15 19:04:13 +0800
categories: [考试总结]
tags: [云斗学院]
math: true
---

没能参加在线比赛，线下 vp 时跳过了前两道 Bonus 题。

## `C` - 超正经的树上问题

想到了一种很不正经的解法，详见[题解](https://www.luogu.com.cn/blog/andycode3759/solution-ydoip2023a)。

## `D` - 论翻越坚果的艺术

考试时搞 $T_1$ 花了两个多小时导致没时间思考 $T_2$，应该算比较遗憾。是个比较套路的二分答案+贪心题。

所有坚果的高度和是不变的，最大化剩下的坚果高度和，可以转化为最小化被跳过的坚果高度和。考虑二分答案并检验，是否可以摆放坚果使得任意长度为 $k$ 的区间内坚果高度和都不超过 $lim$。

贪心放的过程中遵循尽量靠前放的原则。对于一个坚果 $x$，求它最远能和前面第几个坚果打包（被一个长度为 $k$ 的区间覆盖，高度和不超过 $lim$），设找到的是 $y$，如果 $y=0$ 说明可以原地放（$pos_x=x$），否则就放在 $y$ 的后面 $k$ 个单位的位置、或者和前一个坚果贴贴（$pos_x=\max(pos_{i-1}+1,pos_y+k)$）。找 $y$ 可以直接在前缀和里 `lower_bound`。如果发现 $pos > m$ 就说明放不下了，该 $lim$ 不可行。

二分答案的下界是 $\max(h_i)$，因为显然如果有坚果自己的高度大于 $lim$ 的话，该方案一定是不可行的。

```c++
int n, m, k;
int hei[MAXN];
ll sum[MAXN];

int pos[MAXN];
inline bool check(ll lim)
{
    for (int i = 1; i <= n; i++)
        pos[i] = 0;
    pos[1] = 1;
    for (int i = 2; i <= n; i++)
    {
        int y = lower_bound(sum, sum + i, sum[i] - lim) - sum;
        if (y == 0)
            pos[i] = i;
        else
            pos[i] = max(pos[i - 1] + 1, pos[y] + k);
        if (pos[i] > m)
            return false;
    }
    return true;
}

int main()
{
    scanf("%d %d %d", &n, &m, &k);
    ll l = 0, r = INF, ans = 1;
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", hei + i);
        l = max(l, (ll)hei[i]);
        sum[i] = sum[i - 1] + hei[i];
    }

    while (l <= r)
    {
        ll mid = (l + r) >> 1;
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
    printf("%lld\n", sum[n] - ans);
    return 0;
}
```

## `E` - Yet Another Yummy Problem

看数据范围应该是状压 DP，但是想不出来怎么压就把这题正解放掉了。实际上是数位 DP 和状压 DP 的结合。

## `F` - 好梦一日游

这道题的关键在于转化题意：给定两个等长数组 $a,b$，两个数组之间存在一种一一对应关系，这个对应关系的代价为每一对 $a,b$ 里最大值的和，求最小代价。

### Subtask 1

把所有对应关系都试一遍，即暴力枚举 $b$ 的全排列，求 $\sum_{i=1}^{n}\max (a_i,b_i)$，然后记录最小答案即可。

```c++
namespace Solve1
{
    int perm[MAXN];
    inline void solveSub()
    {
        for (int i = 1; i <= n; i++)
            perm[i] = i;
        ll ans = INF;
        do
        {
            ll sum = 0;
            for (int i = 1; i <= n; i++)
                sum += max(a[i], b[perm[i]]);
            ans = min(ans, sum);
        } while (next_permutation(perm + 1, perm + 1 + n));
        printf("%lld\n", ans);
    }
    inline void main()
    {
        for (int i = 1; i <= n; i++)
            a[i] = max(a[i], 100);
        solveSub();
        while (q--)
        {
            int x, y;
            scanf("%d %d", &x, &y);
            a[x] += y;
            solveSub();
        }
    }
}
```

### Subtask 2

单次询问可以在 $O(n \log n)$ 内贪心解决：先把 $a$ 从大到小排序，依次让每一个 $a$ 找到配对的 $b$，如果有不超过 $a$ 的 $b$ 就尽量选最大，否则尽量选最小。

```c++
namespace Solve2
{
    multiset<int> S;
    int sa[MAXN];
    inline void solveSub()
    {
        S.clear();
        for (int i = 1; i <= n; i++)
            S.insert(b[i]), sa[i] = a[i];
        sort(sa + 1, sa + 1 + n, greater<int>());
        ll ans = 0;
        for (int i = 1; i <= n; i++)
        {
            auto it = S.lower_bound(sa[i]);
            if (it == S.end())
                it--;
            ans += max(sa[i], *it);
            S.erase(it);
        }
        printf("%lld\n", ans);
    }
}
```

## 最后的模拟赛

距离 NOIP 2023 只有不到 60 个小时了，与其继续卷题，不如好好思考一下正式比赛时应该有个怎么样的心理准备。

综合近几次模拟赛结果，根据个人的水平和对 NOIP 难度的预期，正式赛的感觉大概会是这样：

- $T_1$ 应该是唯一一道有能力做出来的签到题，但是不会像 CSP-S 2023 $T_1$ 那样一眼秒，多多少少会有点思维难度，而且比较花时间。很有可能要跳过去再回头思考两三次才能解决。对于签到题的策略就是保证细节不出错，能对拍一定要对拍，争取切掉。
- $T_2$ 属于中档左右的题目，大概率 A 不了，但是部分分会很容易拿。拿到该拿的 Subtask 后可以多思考一下能不能突破，如果想出来了就是意外收获，想不出来也属于正常。
- $T_3$ 应该是中等偏上的题目，一定 A 不了，那就尽可能挖掘可做的 Subtask，如果能骗分就尽量骗。正解想不出来直接放弃。
- $T_4$ 属于不可做题，送出来的分拿到后就不要继续浪费时间。

目标 150+，在 JX 拿省一绰绰有余，或许还能给省选铺路。总之，只要不再犯以前犯过的错误（没对拍、对拍对错、不开 `long long`、切错题、模板默写不出来），能拿下的分全部解决，保持良好的心态，那么 NOIP 就可以是不留遗憾的一次比赛。
