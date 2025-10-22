---
title: 小技巧 - 一种很新的带模运算模板
date: 2023-10-26 20:38:38 +0800
categories: [未分类]
tags: []
math: true
---

你是否还在为带模运算要处处 `(long long)`、`%MOD` 而感到头痛和焦虑，甚至因此丢过分数？

实际上可以把带模运算的数抽象成结构体，再重定义运算符，就可以让运算式简单、清晰很多。

## 板子

```c++
struct ModedNum
{
    const static int MOD = 1e9 + 7; // 这个常数按需修改，如果别处要用也可以放外面
    // 注意：仅支持模质数，因为用到了费马小定理来计算逆元

    static int quickPow(int x, int p) // 如果别的地方也要用快速幂可以放外面
    {
        int res = 1;
        for (; p > 0; p >>= 1)
        {
            if (p & 1)
                res = (long long)res * x % MOD;
            x = (long long)x * x % MOD;
        }
        return res;
    }

    ModedNum(long long x = 0) : val(x) {}
    long long val;

    const ModedNum operator+(const ModedNum &b) const
    {
        return ModedNum((val + b.val) % MOD);
    }
    const ModedNum operator-(const ModedNum &b) const
    {
        return ModedNum((val - b.val + MOD) % MOD);
    }
    const ModedNum operator*(const ModedNum &b) const
    {
        return ModedNum(val * b.val % MOD);
    }
    const ModedNum operator/(const ModedNum &b) const
    {
        return ModedNum(val * quickPow(b.val, MOD - 2) % MOD);
    }
    const ModedNum operator=(const long long &b)
    {
        val = b % MOD;
        return *this;
    }
    const ModedNum operator+(const long long &b) const
    {
        return *this + ModedNum(b);
    }
    const ModedNum operator-(const long long &b) const
    {
        return *this - ModedNum(b);
    }
    const ModedNum operator*(const long long &b) const
    {
        return *this * ModedNum(b);
    }
    const ModedNum operator/(const long long &b) const
    {
        return *this / ModedNum(b);
    }
};
```

## 使用例

[P5323 [BJOI2019] 光线](https://www.luogu.com.cn/problem/P5323)

正常写法（仅展示计算表达式部分）：

```c++
const int MOD = 1e9 + 7;
const int INV_OF_100 = 570000004;
int n;
int a[MAXN], b[MAXN];

inline void solve()
{
    for (int i = 1; i <= n; i++)
        a[i] = (long long)a[i] * INV_OF_100 % MOD, b[i] = (long long)b[i] * INV_OF_100 % MOD;
    int A = a[1], B = b[1];
    for (int i = 2; i <= n; i++)
    {
        int X = MOD - (long long)B * b[i] % MOD + 1;
        A = (long long)A * a[i] % MOD * quickPow(X, MOD - 2) % MOD;
        B = (b[i] + ((long long)B * a[i] % MOD * a[i] % MOD) * quickPow(MOD - (long long)B * b[i] % MOD + 1, MOD - 2) % MOD) % MOD;
    }
    printf("%d\n", A);
}
```

带板子写法：

```c++
int n;
ModedNum a[MAXN], b[MAXN];

inline void solve()
{
    for (int i = 1; i <= n; i++)
        a[i] = a[i] / 100, b[i] = b[i] / 100;
    ModedNum A = a[1], B = b[1];
    for (int i = 2; i <= n; i++)
    {
        ModedNum X = ModedNum(1) - B * b[i];
        A = (A * a[i]) / X;
        B = b[i] + a[i] * a[i] * B / X;
    }
    printf("%lld\n", A.val);
}
```

是不是感觉眼前一亮？
