---
title: 题解 - ABC342D Square Pair 平方对
date: 2024-02-25 16:24:27 +0800
categories: [题解]
tags: [atcoder, abc]
math: true
---

链接：[ABC342 D](https://atcoder.jp/contests/abc342/tasks/abc342_d)

## 题目大意

给定一个 $n$ 个自然数组成的数列 $A$，求有多少对不同位置的数相乘可得到平方数（可以表示成自然数的平方的数）。所有数的值域 $\leq 2 \times 10^5$。

## 解析

基本思路都是把每一个数的平方素因子都除去，转化为统计有多少对数相同的问题。

暴力分解素因子什么的都太 low 了，观察到值域很小，可以在埃氏筛筛素数的时候顺便处理出每一个数的最小平方素因子，对于每一个 $A_i$ 再用一个循环处理一下就行了。速度远远快于 $O(n \log V)$。

一开始写的哈希但是挂了，其实哈希本身没写错，是对于 $0$ 的处理出错了……非 $0$ 数的贡献和 $0$ 要分开来算，对于第 $i$ 个出现的 $0$ 其对答案有 $n-i$ 的贡献。

技能点：数论，埃氏筛的应用。

**Extra**：本题有加强版 「SNOI2024」平方数（[洛谷](https://www.luogu.com.cn/problem/P10063) / [LibreOJ](https://loj.ac/p/4041)），值域为 $\leq 10^{36}$（没错，`long long` 都存不下 ~~（但是可以用 `__int128`）~~）。

## 参考代码

```c++
#include <cstdio>
#include <map>
using namespace std;
using ll = long long;
const int MAXN = 200005;
const int MAXP = 18000;

int mp[MAXN];
bool np[MAXN];
map<int, int> ncnt;

int n;
int arr[MAXN];

inline void primeInit()
{
    np[1] = true;
    for (int i = 2; i * i < MAXN; i++)
    {
        if (np[i])
            continue;
        for (int j = i + i; j < MAXN; j += i)
        {
            np[j] = true;
            if (mp[j] == 0 && j % (i * i) == 0) // 最小平方素因子
                mp[j] = i * i;
        }
    }
}

int main()
{
    primeInit();
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
        scanf("%d", arr + i);
    for (int i = 1; i <= n; i++)
    {
        while (mp[arr[i]] > 0)
            arr[i] /= mp[arr[i]];
    }
    ll ans = 0;
    int zcnt = 0;
    for (int i = 1; i <= n; i++)
    {
        if (arr[i] == 0)
            zcnt++, ans += n - zcnt;
        else
            ncnt[arr[i]]++;
    }
    for (auto p : ncnt)
        ans += (ll)p.second * (p.second - 1) / 2;   // C(k,2)求和
    printf("%lld\n", ans);
    return 0;
}
```
