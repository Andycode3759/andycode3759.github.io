---
title: 考试总结 - Public NOIP Round#3 Div.1
date: 2023-05-06 17:06:24 +0800
categories: [考试总结]
tags: [pjudge]
math: true
---

## A - 移除石子

- 30 分（特殊性质 1）：石子全部等距摆成一条直线，直接排序后从左下角开始两个两个拿就可以。
- 另外 20 分（特殊性质 2）：石子全部摆在 $y$ 轴上，也是直接排序从下往上两个两个拿，记得正方形另一顶点的坐标需要通过边长计算一下。
- 100 分：从上面骗分的过程中不难想到应该要先给石子全部排序，再来按顺序考虑。我们假设排序后，最“靠近右上角”的三个点从后往前分别是 $X,Y,Z$（不一定是离右上角的距离最近，只是按坐标排序的优先级最低）：
  1. $X$ 与 $Y$ 的横坐标相等，那么靠右上角放一个正方形就可以消去 $X$ 和 $Y$；
  2. $X$ 的横坐标大于 $Y$ 的横坐标：
     2.1. 如果 $Y$ 的纵坐标小于等于 $X$ 的纵坐标，可以直接用正方形将两者框起来，因为可以确保中间没有其他石子；
     2.2. 如果 $Y$ 的纵坐标大于 $X$ 的纵坐标，那就考虑 $Z$，如果 $Y$ 和 $Z$ 的横坐标相同则可以先消去（记得正方形的横坐标要偏移 $0.5$ 个单位，避免“误伤” $X$）；
     2.3. 以上情况都不是：那就不考虑 $Z$，直接将 $X$ 和 $Y$ 框起来消去。

```c++
void solveSP1() // 特殊情况：所有石子的横纵坐标相等
{
    printf("Yes\n");
    sort(stones + 1, stones + 1 + n, StoneCmpY());
    for (int i = 1; i < n; i += 2)
    {
        printf("%d %d %d %d\n", stones[i].x, stones[i].y, stones[i + 1].x, stones[i + 1].y);
    }
}
void solveSP2() // 特殊情况：所有石子都在Y轴上
{
    printf("Yes\n");
    sort(stones + 1, stones + 1 + n, StoneCmpY());
    for (int i = 1; i < n; i += 2)
    {
        printf("0 %d %d %d\n", stones[i].y, stones[i + 1].y - stones[i].y, stones[i + 1].y);
    }
}
void solveGeneral() // 一般情况
{
    printf("Yes\n");
    sort(stones + 1, stones + 1 + n, StoneCmp());
    for (int x = n; x >= 1;)
    {
        if (vis[x])
        {
            --x;
            continue;
        }
        int y = x - 1;
        while (vis[y])
            --y;
        Stone &sx = stones[x], &sy = stones[y];

        if (sx.x == sy.x)
        {
            printf("%d %d %d %d\n", sx.x, sx.y, sx.x + sx.y - sy.y, sy.y);
            vis[x] = true;
            vis[y] = true;
        }
        else if (sx.y >= sy.y)
        {
            int len = max(sx.x - sy.x, sx.y - sy.y);
            printf("%d %d %d %d\n", sy.x, sy.y, sy.x + len, sy.y + len);
            vis[x] = true;
            vis[y] = true;
        }
        else
        {
            int z = y - 1;
            Stone &sz = stones[z];
            if (z == 0 || sz.x < sy.x || sz.y < sx.y)
            {
                // Z可能不存在
                int len = max(sx.x - sy.x, sy.y - sx.y);
                printf("%d %d %d %d\n", sy.x, sx.y, sy.x + len, sx.y + len);
                vis[x] = true;
                vis[y] = true;
            }
            else if (sz.y > sx.y)
            {
                printf("%d %d %d %d\n", sz.x, sz.y, sz.x + (sy.y - sz.y), sy.y);
                vis[z] = true;
                vis[y] = true;
            }
            else if (sy.y - sz.y <= sx.x - sz.x)
            {
                printf("%.2lf %d %.2lf %d\n", sy.x - 0.5, sy.y, sy.x - 0.5 + sy.y - sz.y, sz.y);
                vis[y] = true;
                vis[z] = true;
            }
            else
            {
                printf("%d %d %d %d\n", sz.x, sz.y, sx.x, sz.y + (sx.x - sz.x));
                vis[x] = true;
                vis[z] = true;
            }
        }
    }
}
```

## B - 抓内鬼

之前考过的原题，见[D 题]({{ "/posts/exam-review-2023-3-11" | relative_url }})。

~~考过的原题都做不出来是不是要给自己两个大 B 兜~~

## C - 异或序列

首先考虑 DP，定义答案 $f(i,j,k)$ 为长度为 $i$、以 $j,k$ 结尾的好的序列的数量。这样来算的话需要 $O(n^4)$ 的时间，显然太慢，能不能减掉一维呢？实际上序列的长度和答案无关，因此可以减去 $i$ 这一维。

```c++
for (int i = 1; i <= n; i++)
{
    for (int j = i + 1; j <= n; j++)
    {
        f[i][j] = 1;
    }
}

for (int i = 1; i <= n; i++)
{
    for (int j = i + 1; j <= n; j++)
    {
        for (int x = j + 1; x <= n; x++)
        {
            if (x != (i ^ j))
            {
                f[j][x] += f[i][j];
                f[j][x] %= mod;
            }
        }
    }
}

int ans = n;    // 答案还需要加上n（长度为1的序列个数）

for (int i = 1; i <= n; i++)
{
    for (int j = i + 1; j <= n; j++)
    {
        ans += f[i][j];
        ans %= mod;
    }
}
```

40 分已经是 DP 算法的极限了，想要拿到满分还需要换一种思路。这种序列存在这样一种性质：如果某个任意的序列 $a$ 在连续的三个位置 $a_i,a_{i+1},a_{i+2}$ 上的元素异或和为 $0$，那么再往前挪一个位置 $a_{i-1},a_i,a_{i+1}$ 的三个元素异或和一定不为 $0$。如此，可以设 $f(n)$ 表示以 $n$ 结尾的合法序列的个数，$g(n)$ 为在 $n$ 处第一次满足异或和为 $0$ 的序列数，那么

$$ f(n)=1+\sum\_{i<n}f(i) - g(n) $$

如何计算 $g(n)$ 呢？如果某个以 $n$ 结尾的序列的倒数第 2、3 个元素是 $x,y$，可以枚举可能的 $y$，如果有 $x=(y\ xor\ n)$ 满足 $x<y$，那么 $g(n)$ 就是对于所有满足条件的 $y$，$f(x)$ 的总和。复杂度 $O(n^2)$，大概有 60 分。

```c++
f[0] = f[1] = 1;
for (int y = 2; y <= n; y++)
{
    int sum = 0;
    for (int x = 1; x <= y - 1; x++)
    {
        sum += f[x];
        sum %= mod;
    }
    int g = 0;
    for (int x = 1; x <= y - 1; x++)
    {
        if ((x ^ y) < x)
        {
            g += f[x ^ y];
            g %= mod;
        }
    }
    f[y] = (1 + sum - g + mod) % mod;
}

int ans = 0;
for (int i = 1; i <= n; i++)
{
    ans += f[i];
    ans %= mod;
}
printf("%d\n", ans % mod);
```

## D - 数圈圈

最暴力的算法就是四维枚举起点和终点的 $x,y$ 坐标，当然可以提前判断相邻的两个字符是否相同，会稍微快一点。

矩形的本质是两对横竖线上连续相同的字符，那么可以考虑算出各个位置的横向、纵向的字符串的哈希值，判断其是否为哈希基数的倍数（一列或一行相同字符）再判断行列是否分别相等（四边由同一种字符组成），就可以快速地判断矩形而无需 $O(n)$ 枚举。

```c++
const int MAXN = 2008;
const int MOD = 23;

int modPow[MAXN];
char canvas[MAXN][MAXN];
int n, m;
bool mark[30];
int hashX[MAXN][MAXN], hashY[MAXN][MAXN];

int findX(int ax, int ay, int bx, int by)
{
    return hashX[bx][by] - hashX[ax][ay - 1] * modPow[by - ay + 1];
}
int findY(int ax, int ay, int bx, int by)
{
    return hashY[bx][by] - hashY[ax - 1][ay] * modPow[bx - ax + 1];
}

int main()
{
    scanf("%d%d", &n, &m);

    modPow[0] = 1;
    for (int i = 1; i <= max(n, m); i++)    // 预处理出MOD的次方
    {
        modPow[i] = modPow[i - 1] * MOD;
    }

    int lc = 0;
    for (int i = 1; i <= n; i++)
    {
        scanf("%s", canvas[i] + 1);
        for (int j = 1; j <= m; j++)
        {
            int c = canvas[i][j] - 'a';
            if (!mark[c])
                lc++;
            mark[c] = true;

            hashX[i][j] = hashX[i][j - 1] * MOD + c + 1;    // 计算哈希
            hashY[i][j] = hashY[i - 1][j] * MOD + c + 1;
        }
    }
    if (lc == 1)
    {
        // 子任务3的特殊情况
        printf("%lld\n", 1LL * m * n * (m - 1) * (n - 1) / 4);
        return 0;
    }

    long long ans = 0;

    for (int ax = 1; ax < n; ax++)
    {
        for (int ay = 1; ay < m; ay++)
        {
            for (int bx = ax + 1; bx <= n; bx++)
            {
                if (canvas[bx][ay] != canvas[bx - 1][ay])
                    break;  // 提前判断相邻位置
                for (int by = ay + 1; by <= m; by++)
                {
                    if (canvas[ax][by] != canvas[ax][by - 1])
                        break;
                    if (findX(ax, ay, ax, by) == findX(bx, ay, bx, by) &&
                        findY(ax, ay, bx, ay) == findY(ax, by, bx, by))
                        ans++;
                }
            }
        }
    }

    printf("%lld\n", ans);
    return 0;
}

```
