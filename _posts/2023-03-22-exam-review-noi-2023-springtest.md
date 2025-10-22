---
title: 考试总结 - NOI 2023 春季测试模拟赛
date: 2023-03-22 16:27:11 +0800
categories: [考试总结]
tags: [noi春测]
math: true
---

下午花了~~一坤~~两个半小时做了做春测的题目...正常考试时间有 4.5 小时，所以只拿了 140 分合情合理 ~~（菜是原罪）~~

另外还是第一次用 OI/NOIP 赛制的模拟测试（原先一直为 IOI/ACM 赛制，即提交后可见结果），本来有点怕不适应，实际上也没什么。只要对自己的水平有较清晰的认识，赛制的区别可以成为能够忽略的小细节。

自从吸取了[上次翻车的经验]({{ "/posts/exam-review-2023-3-11" | relative_url }})后决定再也不抢最优解了，老老实实先打暴力，把该拿的分拿到。

## A - [涂色游戏 paint][t1]

模拟就完事了：

```c++
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1005;

int canvas[MAXN][MAXN];

int main()
{
    freopen("paint.in", "r", stdin);
    freopen("paint.out", "w", stdout);

    int T;
    scanf("%d", &T);
    while (T--)
    {
        memset(canvas, 0, sizeof(canvas));
        int n, m, q;
        // n行m列
        scanf("%d%d%d", &n, &m, &q);

        int opt, x, c;
        // opt=0 行
        // opt=1 列
        for (int j = 0; j < q; j++)
        {
            scanf("%d%d%d", &opt, &x, &c);

            if (opt == 0)
            {
                for (int i = 1; i <= m; i++)
                {
                    canvas[x][i] = c;
                }
            }
            if (opt == 1)
            {
                for (int i = 1; i <= n; i++)
                {
                    canvas[i][x] = c;
                }
            }
        }

        for (int x = 1; x <= n; x++)
        {
            for (int y = 1; y <= m; y++)
            {
                printf("%d ", canvas[x][y]);
            }
            printf("\n");
        }
    }
    return 0;
}
```

即使不考虑效率问题，二维数组也开不下 $10^{10}$ 的空间，因此理论上可以有 70 分左右。接着观察到 15~16 号样例有个奇妙的性质：保证 $opt$ 为 $1$，那么对于这些测试点可以新加一个一维数组特殊处理（重复部分的代码省略）：

```c++
...
const int MAXSN = 100006;
int spCanvas[MAXSN];

int main()
{
    ...
    while (T--)
    {
        memset(spCanvas, 0, sizeof(spCanvas));
        bool spCase = false;
        ...

        if (m >= MAXN)
            spCase = true;  // 特殊测试点

        for (int j = 0; j < q; j++)
        {
            scanf("%d%d%d", &opt, &x, &c);
            if (spCase)
            {
                spCanvas[x] = c;
            }
            else    // 一般情况，同上
            {
                ...
            }
        }

        if (spCase)
        {
            for (int j = 1; j <= n; j++)    // 记得一共有n行
            {
                for (int i = 1; i <= m; i++)
                {
                    printf("%d ", spCanvas[i]);
                }
                printf("\n");
            }
        }
        else    // 一般情况，同上
        {
            ...
        }
    }
    return 0;
}
```

理论上可以解决 2 个测试点多拿 10 分，可惜由于码力不足导致程序有些特性，考完判了下依然是 65 分 ~~（没掉分就不错了）~~

接下来便是搞正解。作为 NOI 出题的传统，第一题一般都不难，仔细思考都能满分。这题的模型有如下几个特点：

1. 一个点的颜色可能会被横画的操作影响，也可能会被纵画的操作影响；
2. 相同 $x$ 的横画（或纵画）会互相覆盖，后面的会覆盖前面的。

这两句话单独拎出来可能是废话，却是解题的关键。根据 2，只需要记录某一列（或某一行）**最后一次**的操作即可（因为前面的操作都被覆盖掉了），再根据 1 在输出时依据记录的操作来还原颜色，判断**影响到这一行的操作**和**影响到这一列的操作**谁**来得更晚**即可。

```c++
// 正解
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 100005;

struct Operate
{
    Operate(int _c, int _z)
    {
        color = _c;
        zidx = _z;
    }
    int color;
    int zidx;   // 操作的顺序（或优先级），zidx大的会盖过小的
};
vector<Operate> opts; // 所有操作
int affectRow[MAXN];  // 最后一次影响到第i行的操作在opts中的下标
int affectCol[MAXN];  // 最后一次影响到第i列的操作在opts中的下标

int main()
{
    freopen("paint.in", "r", stdin);
    freopen("paint.out", "w", stdout);

    int T;
    scanf("%d", &T);
    while (T--)
    {
        vector<Operate>().swap(opts);
        memset(affectRow, -1, sizeof(affectRow));
        memset(affectCol, -1, sizeof(affectCol));
        int n, m, q;
        // n行m列
        scanf("%d%d%d", &n, &m, &q);

        int opt, x, c;
        for (int j = 0; j < q; j++)
        {
            scanf("%d%d%d", &opt, &x, &c);
            opts.push_back(Operate(c, j));
            if (opt == 0)   // 影响到了第x行
            {
                affectRow[x] = opts.size() - 1;
            }
            if (opt == 1)   // 影响到了第x列
            {
                affectCol[x] = opts.size() - 1;
            }
        }

        for (int x = 1; x <= n; x++)    // 行
        {
            for (int y = 1; y <= m; y++)    // 列
            {
                // 取出 最后一次影响x行的操作 和 最后一次影响y行的操作，
                // 比较两者顺序决定最后的颜色
                Operate r = affectRow[x] < 0 ? Operate(0, -1) : opts[affectRow[x]];
                Operate c = affectCol[y] < 0 ? Operate(0, -1) : opts[affectCol[y]];
                // 注意此行或列可能一笔都没画上，需要特殊判断

                // 此处可以保证r.zidx和c.zidx不相等，所以不用考虑等号的问题
                int ans = r.zidx > c.zidx ? r.color : c.color;
                printf("%d ", ans);
            }
            printf("\n");
        }
    }
    return 0;
}
```

## B - [幂次 power][t2]

暴力枚举：

```c++
// 暴力v1
// 得分 10/100
#include <bits/stdc++.h>
using namespace std;

int main()
{
    freopen("power.in", "r", stdin);
    freopen("power.out", "w", stdout);

    int n, k;
    scanf("%d%d", &n, &k);
    int ans = 0;
    for (int x = 1; x <= n; x++)
    {
        for (int a = 1; a <= x; a++)
        {
            bool flag = false;
            for (int b = k; b <= n; b++)
            {
                if (pow(a, b) == x)
                {
                    ans++;
                    flag = true;
                    break;
                }
            }
            if (flag)
                break;
        }
    }

    printf("%d\n", ans);

    return 0;
}
```

正解需要用到容斥原理。设 $f(i)$ 表示所有可表示为 $a^i$ 且小于等于 $n$ 的 $i$ 不同的数的个数（即$f(i)=\sum_{x=a^i}^{x \leq n, x_i \not = x_j} 1$），设 $g(i)$ 表示所有可以表示为 $a^i$ 的小于等于 $n$ 的数的个数（即$g(i)=\sum_{x=a^i}^{x \leq n} 1$）。推导可得 $g(i)=f(i)+f(2i)+f(3i)+\dots+f(ki)$，且$g(i)=\sqrt[i] n$，由容斥原理可得 $f(i)=g(i)-f(2i)-f(3i)-\dots-f(ki)$。

```c++
// Insert Code Here
```

### C - [圣诞树 tree][t3]

标准 DFS 都不会写的蒟蒻选择手动模拟枚举排列 ~~，居然拿了 30 分可喜可贺~~（~~其实本来是只想拿 10 分的~~）：

```c++
// 暴力
// 得分 30/100
#include <bits/stdc++.h>
using namespace std;
const int MAXN = 1006;
const double EPS = 1e-11;

struct Point
{
    Point(double _x = 0, double _y = 0, int _i = 0)
    {
        x = _x;
        y = _y;
        index = _i;
    }

    double x, y;
    int index;
};
struct PointCmp
{
    bool operator()(Point a, Point b)
    {
        if (abs(a.y - b.y) <= EPS)
        {
            return a.index < b.index;
        }
        else
        {
            return a.y > b.y;
        }
    }
};
struct BFSNode
{
    vector<int> perm;   // 记录原始排列，就是这么简单粗暴
    double distance;
};

Point points[MAXN];

int n;
bool book[MAXN];

double getDistance(Point a, Point b)
{
    return sqrt(pow(a.x - b.x, 2) + pow(a.y - b.y, 2));
}
bool contains(vector<int> &v, int obj)  // 顺便吐槽一下 为什么STL的容器没有C#那样的Contains子函数
{
    for (vector<int>::iterator i = v.begin(); i != v.end(); i++)
    {
        if (*i == obj)
            return true;
    }
    return false;
}

int main()
{
    freopen("tree.in", "r", stdin);
    freopen("tree.out", "w", stdout);

    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
    {
        double x, y;
        scanf("%lf%lf", &x, &y);
        points[i] = Point(x, y, i);
    }
    sort(points + 1, points + 1 + n, PointCmp());   // 找出开头的点，顺便进行一个离散化（虽然我也不知道意义何在（

    vector<int> ansPerm;
    double minDis = 1e15;
    queue<BFSNode> Q;
    BFSNode start;
    start.perm.push_back(1);
    start.distance = 0;
    Q.push(start);

    while (!Q.empty())
    {
        BFSNode p = Q.front();
        Q.pop();
        if (p.perm.size() >= n)
        {
            if (p.distance < minDis)
            {
                ansPerm = p.perm;
                minDis = p.distance;
            }
            continue;
        }
        for (int i = 1; i <= n; i++)
        {
            if (contains(p.perm, i))
                continue;
            BFSNode ex;
            ex.perm = p.perm;
            Point a = points[*(ex.perm.end() - 1)];
            ex.perm.push_back(i);
            Point b = points[*(ex.perm.end() - 1)];
            ex.distance = p.distance + getDistance(a, b);
            Q.push(ex);
        }
    }

    for (vector<int>::iterator i = ansPerm.begin(); i != ansPerm.end(); i++)
    {
        printf("%d ", points[*i].index);
    }
    printf("\n");
    return 0;
}
```

对于正解思路，首先想到的是 DP，定义 `minDis[i][j]` 为从第 $i$ 个灯连到第 $j$ 个灯的最小距离，那么对于每一段区间都有连左边或连右边两种决策，自然就可以这样转移：

$$minDis(l,r)=min\{minDis(l+1,r)+\lvert P_{l}P_{l+1} \rvert, minDis(l,r-1)+\lvert P_{r}P_{r-1} \rvert\}$$

但此方法只能求出最短距离，求路径还需要记录每一步的决策，用 `decision[i][j]` 记录在计算 `minDis[i][j]` 时最后一次连的是几号灯。最后递归输出答案就好了。

_...对吧？_ 直到开始对大样例时结果就出错了，当时一直以为是输出答案时的递归顺序不对，因此尝试过了不下三种 `printAns()` 函数的实现，但是均只解决了单个样例的特殊情况：

```c++
void printAns(int left, int right)
{
    int &d = decision[left][right];
    printf("%d ", points[d].index);

    if (left != right)
    {
        if (d == left)
        {
            printAns(left + 1, right);
        }
        if (d == right)
        {
            printAns(left, right - 1);
        }
    }
}
```

```c++
vector<int> ansPerm;    // 输出时逆序输出
void printAns(int left, int right)
{
    int &d = decision[left][right];
    ansPerm.push_back(points[d].index);

    if (left != right)
    {
        if (d == left)
        {
            printAns(left + 1, right);
        }
        if (d == right)
        {
            printAns(left, right - 1);
        }
    }
}
```

```c++
vector<int> ansPerm;
void printAns(int left, int right)
{
    int &d = decision[left][right];


    if (left != right)
    {
        if (d == left)
        {
            printAns(left + 1, right);
        }
    }
    ansPerm.push_back(points[d].index);
    if (left != right)
    {
        if (d == right)
        {
            printAns(left, right - 1);
        }
    }
}
```

事实上并非输出答案顺序的问题，这种思路本身就不符合题意。真正的解法应该如何呢？其实 DP 的大体思路是正确的，只不过还要多加一维，同时修改 `minDis` 数组的含义：`minDis[l][r][d]`表示从最高点 $k$ 开始，向左接 $l$ 个灯，向右接 $r$ 个灯，并且当前位置处于左边（$k=0$）或右边（$k=1$）时的最短线长。同时由于整个图构成了一个环，还需要倍长序列。

```c++
// Insert Code Here
```

## D - [密码锁 lock][t4]

考试时在 C 题上浪费了太多时间，导致看 D 题时心态有点慌，没有深入理解题意就交了个白卷。反思后发现这其实是不应该的，最起码应该拿到 30 分：$k=1$ 的情况下只需要算最大值和最小值的差，$k=2$ 的情况下让每一个拨圈小数在上、大数在下即可。

```c++
// Insert Code Here
```

## 如果能多 2 个小时...

如果按照考试时的真实心理状态，很有可能会继续花时间在 C 题上，但有很大概率效果不理想。合理的思路应该是写出 D 题的 30 分代码，再继续从 B 题里多得分。

[t1]: https://www.luogu.com.cn/problem/P9117
[t2]: https://www.luogu.com.cn/problem/P9118
[t3]: https://www.luogu.com.cn/problem/P9119
[t4]: https://www.luogu.com.cn/problem/P9120
