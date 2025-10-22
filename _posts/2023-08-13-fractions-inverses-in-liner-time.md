---
title: 小技巧 - 线性时间求阶乘逆元
date: 2023-08-13 14:35:23 +0800
categories: [未分类]
tags: []
math: true
---

```c++
// How to get Fractions (and their inverses) in liner time
// Stop using getInv(frac[x])!
#include <cstdio>
constexpr int MAXN = 1000006;
constexpr int MOD = 998244353;

int frac[MAXN], fracInv[MAXN];

inline int quickPow(int a, int x)
{
    int res = 1;
    while (x > 0)
    {
        if ((x & 1) == 1)
            res = 1LL * res * a % MOD;
        a = 1LL * a * a % MOD;
        x = x >> 1;
    }
    return res;
}

inline void init(int n)
{
    // Nothing new here...
    frac[0] = 1;
    for (int i = 1; i <= n; i++)
    {
        frac[i] = 1LL * frac[i - 1] * i % MOD;
    }

    // Here goes magic!
    fracInv[n] = quickPow(frac[n], MOD - 2);
    for (int i = n; i >= 1; i--)
    {
        fracInv[i - 1] = 1LL * fracInv[i] * i % MOD;
    }
    /*
    Why? Because:
            fracInv[n] = 1 / frac[n]   ('/' is in modular sense)
                       = 1 / (1 * 2 * ... * n)
    and:
          fracInv[n-1] = 1 / frac[n-1]
                       = 1 / (1 * 2 * ... * (n-1))
    so:
        fracInv[n] * n = fracInv[n-1]
    */
}

int main()
{
    int n;
    printf("Fraction (and their inverse) calculator\n");
    printf("Input n: ");
    scanf("%d", &n);
    init(n);
    printf("%d! = %d (mod %d)\n", n, frac[n], MOD);
    printf("1 / %d! = %d (mod %d)\n", n, fracInv[n], MOD);
    return 0;
}
```
