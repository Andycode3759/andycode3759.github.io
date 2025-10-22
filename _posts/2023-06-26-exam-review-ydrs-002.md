---
title: 比赛总结 - YDRS#002 云斗杯六月 Silver 组模拟赛
date: 2023-06-26 17:21:56 +0800
categories: [考试总结]
tags: [云斗学院]
math: true
---

## A - [签到题也要好好做](https://yundouxueyuan.com/p/5238)

首先肯定是打个暴力，把序列复制一份成环，从 $2$ 到 $n$ 枚举区间长度，再从 $1$ 到 $n$ 枚举区间起点，区间内扫一遍数出奇偶个数即可。时间复杂度 $O(n^3)$。

接着从以前刷到过的[这道题](https://www.luogu.com.cn/problem/P1114)获得灵感，可以把奇数处理成 $-1$，偶数处理成 $1$，对新数组取前缀和，在判断区间是否符合条件时只需要求 `sum[r]-sum[l-1]` 是否为 $0$ 即可。这样可以优化到 $O(n^2)$。

思考一下发现，求答案的过程本质上就是求有序点对 $(i,j),sum_{i-1}=sum_j,i<j$ 的过程。可以一遍扫过 `sum` 数组，同时记录 `sum[i]` 的数值目前为止出现了多少次，之前这个数出现的次数就是这个点对答案的贡献。注意 $n+1$ 到 $2n$ 区间比较特殊，这段区间之内不能有匹配（否则就和 $1$ 到 $n$ 内的重复了），同时区间长度不可能超过 $n$，因此数到后面需要对处理过的区间进行收缩，并且只在前一半区间记录次数。这样说感觉很抽象，还是代码来的清晰：

```c++
int n;
int arr[MAXN * 2];  // 原数组，大小*2是为了复制一份成环
int sum[MAXN * 2];  // 按照奇-1偶1处理后的前缀和
int cnt[MAXN * 2];  // 记录sum在目前能够匹配的左端点区间里出现过的次数

scanf("%d", &n);
for (int i = 1; i <= n; i++)
{
    scanf("%d", arr + i);
    arr[n + i] = arr[i];    // 复制数组
}
int minSum = 0;
for (int i = 1; i <= n * 2; i++)
{
    sum[i] = (arr[i] % 2 == 0) ? 1 : -1;    // 对原数进行处理，同时求前缀和
    sum[i] += sum[i - 1];
    minSum = min(minSum, sum[i]);
}
for (int i = 0; i <= n * 2; i++)
{
    sum[i] -= minSum;   // 这样做是为了保证所有的sum都大于等于0
}

long long ans = 0;
for (int i = 1; i <= n * 2; i++)
{
    if (i > n + 1) // 区间长度不能超过n，因此到后一半区间时要从左端开始消去前面的数的影响
    {
        cnt[sum[i - n - 1]]--;
    }
    ans += cnt[sum[i]]; // 如果在[1,i-1]区间内出现了sum[i]这个数，那么出现的次数就是对答案的贡献
    if (i <= n) // 只有i<=n才能累加次数，否则会出现[n+1,2*n]区间内部的匹配，就和[1,n]内的重复了
        cnt[sum[i]]++;  // 要先累加答案再更新次数，否则会把[i,i-1]这个不合法的区间也算进去
}

printf("%lld\n", ans);
return 0;
```

这道题程序实现确实不难，但有很多细节需要注意，稍不注意就极有可能爆零。因此写过的暴力不应该丢掉，要拿来对拍。同时在赛场上，大部分的细节其实并不需要严格证明保证正确，可以通过感性的方式朝着“认为正确”的方向实现 ~~（又称瞎蒙）~~，对拍后答案对的上就足够说明程序的正确性了。

还有一点，之前刷过的题可以为比赛时解题提供灵感甚至模型，虽然不太可能考到能复制粘贴的原题，但还是要学会知识点的灵活运用。

## B - [比赛也需要懂套路](https://yundouxueyuan.com/p/5236)

暴力做法：枚举循环节长度 $ans$（显然 $ans$ 必须是 $n$ 的因子），然后把原字符串拆分成 $n \over ans$ 个小节，判断每个小节里面相对位置一样的字符是否相同。时间大概是 $O(qn^2)$。

事实上，并不需要判断*所有*小节是否相同也能够判定答案的合法性。对于任意一个字符串，分别去掉它的第一个和最后一个循环节，得到的两个新字符串，对齐开头位置后一定是相等的。设 $S_i$ 表示第 $i$ 个小节，这样做等价于判断一个有 $n$ 个小节的字符串，$\forall i \in [1,n-1],S_i=S_{i+1}$ 是否成立（因为在“对齐”的过程中会把字符串偏移一个小节的长度）。

```c++
bool strcmp(char *s1, char *s2, int len)
{
    for (int i = 0; i < len; i++)
    {
        if (*s1 != *s2)
            return false;
        s1++, s2++;
    }
    return true;
}

int solve(int l, int r)
{
    int len = r - l + 1;
    for (int ans = 1; ans <= len; ans++)
    {
        if (len % ans != 0)
            continue;
        if (strcmp(str + l, str + l + ans, len - ans))
            return ans;
    }
    return len;
}
```

现在主要时间压力来到了字符串比较。这一过程可以利用字符串哈希加速，即算出整个字符串的哈希后求子串哈希并比较。

```c++
bool strcmp(int l1, int r1, int l2, int r2)
{
    // strHash和basePow都需要在主函数预处理出来
    unsigned long long hash1 = strHash[r1] - strHash[l1 - 1] * basePow[r1 - l1 + 1];
    unsigned long long hash2 = strHash[r2] - strHash[l2 - 1] * basePow[r2 - l2 + 1];
    return hash1 == hash2;
}
```

然而这样搞依然会超时，因为在枚举答案长度时一个个检查是否为 $len$ 的约数的过程很费时间。有一个技巧可以加速这一过程，先将长度 $len$ 进行质因数分解，写成形如 $len=p_1^{v_1}p_2^{v_2}...p_n^{v_n}=\prod^{n}_{i=1}p_i^{v_i}$，其中 $p_i$ 均为质数的形式。然后再对于每一个 $p_i$，对其进行 $v_i$ 次尝试，每次尝试先判断 $ans$（初值为 $len$）是否合法，若合法就从 $ans$ 中除去 $p_i$，否则转到下一个 $p_{i+1}$。最后的 $ans$ 即为最小的合法长度。

最后的最后，如果在分解质因数时直接使用埃氏筛法还是会超时，所以需要先把所有 $n$ 以内的数的最小质因数都处理出来，用以加速质因数分解的过程。

```c++
const int MAXN = 500005;
const int BASE = 29;    // 哈希进制
char str[MAXN];
unsigned long long strHash[MAXN];   // 原字符串哈希
unsigned long long basePow[MAXN];   // 哈希位权，用于算子串哈希
int minF[MAXN]; // minF[i]为i的最小质因数
int n, q;

struct Factor
{
    int val = 0, pow = 0;
};
vector<Factor> factors;

int solve(int l, int r)
{
    int len = r - l + 1;

    // 质因数分解
    factors.clear();
    Factor fa;
    int lt = len;
    while (lt > 1)
    {
        fa.val = minF[lt], fa.pow = 0;
        while (lt % fa.val == 0)
        {
            fa.pow++;
            lt /= fa.val;
        }
        factors.push_back(fa);
    }

    // 求出答案
    int ans = len;
    for (int i = 0; i < factors.size(); i++)
    {
        while (factors[i].pow > 0)
        {
            int newLen = ans / factors[i].val;
            unsigned long long hash1 = strHash[r - newLen] - strHash[l - 1] * basePow[r - newLen - l + 1];
            unsigned long long hash2 = strHash[r] - strHash[l + newLen - 1] * basePow[r - l - newLen + 1];
            if (hash1 == hash2)
            {
                factors[i].pow--;
                ans /= factors[i].val;
            }
            else
                break;
        }
    }
    return ans;
}

int main()
{
    scanf("%d", &n);
    // 预处理minF
    for (int i = 2; i <= n; i++)
    {
        if (minF[i] > 0)
            continue;
        for (int j = 1; j * i <= n; j++)
        {
            if (minF[i * j] == 0)
                minF[i * j] = i;
        }
    }
    scanf("%s", str + 1);
    // 计算哈希
    basePow[0] = 1;
    for (int i = 1; i <= n; i++)
    {
        basePow[i] = basePow[i - 1] * BASE;
        strHash[i] = strHash[i - 1] * BASE + str[i] - 'a';
    }
    scanf("%d", &q);
    while (q--)
    {
        int l, r;
        scanf("%d%d", &l, &r);
        printf("%d\n", solve(l, r));
    }
    return 0;
}
```

综上所述，这道题整体上还是很套路的，一个技巧接一个技巧，想要拿满分需要做到滴水不漏，难度比较大。但是拿到较高的部分分还是比较简单的，尤其是在发现快速判断长度合法性的结论并加上哈希比较后。如果最后想不到质因数分解的加速策略，用埃氏筛其实也有 95 分，跟满分差距并不大，所以在这种类型的其他题目上没必要浪费太多的时间追求完美。

## C - [有人不会起题目名](https://yundouxueyuan.com/p/5237)

## D - [出题人准备的礼物](https://yundouxueyuan.com/p/5235)
