---
title: 倍增法计算后缀数组的代码实现
date: 2024-01-10 21:28:16 +0800
categories: [未分类]
tags: [后缀数组]
math: true
---

算法原理讲解请参阅 [OI Wiki / 后缀数组简介 #$O(n \log ^2n)$ 做法](https://oi-wiki.org/string/sa/#onlog2n-%E5%81%9A%E6%B3%95)，此处仅给出代码。

主要思想：在对双关键字 $(rk_i, rk_{ i+2^{lg} })$ （相当于 `std::pair`）进行排序时，将数据对转化为一个整数：$(rk_i, rk_{ i+2^{lg} }) \to rk_i \times \texttt{MAXN} + rk_{ i+2^{lg} }$，然后再用 `std::pair` 将这个整数和下标捆绑，直接排序，就可以得到新的 $rk$ 数组。

- 优点：不需要另外手写结构体/Comparer；内存读写量更小（不需要 `memcpy` 或 `memset`）；~~个人认为~~更好理解；
- 缺点：重度依赖 STL，常数较大；主体算法只操作 $rk$ 数组，结束后要从 $rk$ 再推一遍才能得到 $sa$。

题目：[P3809 【模板】后缀排序](https://www.luogu.com.cn/problem/P3809)

```c++
#include <algorithm>
#include <cstring>
#include <cstdio>
#include <utility>
using namespace std;
using ll = long long;
const int MAXN = 1000006;
const int MAX_LOG = 21;

int n;
char str[MAXN];
int rk[MAXN], sa[MAXN];
pair<ll, int> pr[MAXN];

int main()
{
    scanf("%s", str + 1);
    n = strlen(str + 1);

    for (int i = 1; i <= n; i++)
        pr[i] = {str[i], i};
    sort(pr + 1, pr + 1 + n);
    for (int i = 1, cr = 1; i <= n; i++) // 相同值排序后得到的rk序号是相同的，cr记录目前出现了几个不同的值
    {
        if (i >= 2 && pr[i].first != pr[i - 1].first)
            cr++;
        rk[pr[i].second] = cr;
    }

    for (int lg = 0; (1 << lg) < n && lg < MAX_LOG; lg++)
    {
        for (int i = 1; i <= n; i++)
            pr[i] = {(ll)rk[i] * MAXN, i};
        for (int i = 1; i + (1 << lg) <= n; i++)
            pr[i].first += rk[i + (1 << lg)]; // 第二关键字如果越界就认为是0，即不加第二维
        sort(pr + 1, pr + 1 + n);
        for (int i = 1, cr = 1; i <= n; i++)
        {
            if (i >= 2 && pr[i].first != pr[i - 1].first)
                cr++;
            rk[pr[i].second] = cr;
        }
    }
    for (int i = 1; i <= n; i++)
        sa[rk[i]] = i;
    for (int i = 1; i <= n; i++)
        printf("%d ", sa[i]);
    printf("\n");
    return 0;
}
```
