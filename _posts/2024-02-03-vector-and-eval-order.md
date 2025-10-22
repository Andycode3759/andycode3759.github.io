---
title: 浅谈 `vector` 的内存机制和赋值语句计算顺序
date: 2024-02-03 18:45:06 +0800
categories: [未分类]
tags: []
math: true
---

前段时间在做可持久化数据结构题时遇到了一个奇怪的现象，同一份代码在不同 C++ 语言版本编译下运行结果不同：

[![C++14](https://cdn.luogu.com.cn/upload/image_hosting/6vw0wprb.png)](https://www.luogu.com.cn/record/145836136)

[![C++20](https://cdn.luogu.com.cn/upload/image_hosting/1m4pp10h.png)](https://www.luogu.com.cn/record/145836521)

这道题需要用到动态开点线段树，下面是建树部分的代码：

```c++
struct SegTreeNode
{
    SegTreeNode(int _sl = 0, int _sr = 0, int _v = 0) : sl(_sl), sr(_sr), val(_v) { ls = rs = -1; }
    int sl, sr;
    int ls, rs;
    int val;
};
vector<SegTreeNode> sgt;

int newNode(int sl, int sr, int val)
{
    sgt.push_back(SegTreeNode(sl, sr, val));
    return sgt.size() - 1;
}
int build(int l, int r)
{
    int t = newNode(l, r, 0);
    if (l == r)
    {
        sgt[t].val = arr[l];
        return t;
    }
    int mid = (l + r) >> 1;
    sgt[t].ls = build(l, mid);
    sgt[t].rs = build(mid + 1, r);
    return t;
}
```

因为之前就有过用 `vector` 存点导致出锅的经验，所以我把代码换成了用静态数组实现，并用 C++14 重新交了一遍：

[![C++14 with static array](https://cdn.luogu.com.cn/upload/image_hosting/v3iax8m7.png)](https://www.luogu.com.cn/record/145839911)

针对这一现象，我在 [CodeForces 上发帖](https://codeforces.com/blog/entry/125414)求助，很快就得到了大佬的回复：

![reply](https://cdn.luogu.com.cn/upload/image_hosting/3wmnx753.png)

> Generally speaking, **vector may apply for new memory when changing its size, which will cause the previous address to become invalid.** I think code like "sgt[t].ls = build(l, mid);" may be the culprit. **It first obtains an address in the vector, and then enters the build function**, which causes the vector to expand and the address Invalid. An error occurs, which explains why static arrays are correct.
>
> c++17 added a lot of extra sequence points in evaluation order (https://en.cppreference.com/w/cpp/language/eval_order — in this case specifically rule 19).
>
> In C++14 your program has undefined behaviour because there isn't any ordering requirement between evaluating where to store the result (sgt[t].ls) and evaluating the result to store there (build(l, mid)).

总结来说，造成这一问题的原因有二：

1. `vector` 在有新元素加入时会重新分配所有已储存元素的内存，如果在这之前已经保存了一个指向内部元素的指针，重新分配后这个指针就会变得无效；
2. C++14 和 C++17 中赋值语句左右两边的计算顺序存在差异。

下面我们通过几个简单的实验验证这两个说法。

## 实验一：`vector` 的内存重新分配

下面这段代码在不断往 `vector` 里加元素的同时输出第一个元素所在的地址：

```c++
#include <cstdio>
#include <vector>
using namespace std;

vector<int> vec;

int main()
{
    for (int i = 1; i <= 33; i++)
    {
        vec.push_back(i);
        printf("i=%d addr=%d\n", i, &vec[0]);
    }
    return 0;
}
```

运行结果如下：

```terminal
i=1 addr=-2068364624
i=2 addr=-2068363552
i=3 addr=-2068364624
i=4 addr=-2068364624
i=5 addr=-2068363520
i=6 addr=-2068363520
i=7 addr=-2068363520
i=8 addr=-2068363520
i=9 addr=-2068363472
i=10 addr=-2068363472
i=11 addr=-2068363472
i=12 addr=-2068363472
i=13 addr=-2068363472
i=14 addr=-2068363472
i=15 addr=-2068363472
i=16 addr=-2068363472
i=17 addr=-2068363392
i=18 addr=-2068363392
i=19 addr=-2068363392
i=20 addr=-2068363392
i=21 addr=-2068363392
i=22 addr=-2068363392
i=23 addr=-2068363392
i=24 addr=-2068363392
i=25 addr=-2068363392
i=26 addr=-2068363392
i=27 addr=-2068363392
i=28 addr=-2068363392
i=29 addr=-2068363392
i=30 addr=-2068363392
i=31 addr=-2068363392
i=32 addr=-2068363392
i=33 addr=-2068363248
```

可以看到，在 $i=1,2,3,5,9,17,33$，即 $i=2^n+1$ 的时候，`vector` 的内存发生了重新分配。也就是说，如果我们在内存重新分配前保存了一个指向 `vector` 内部元素的指针，那么重新分配后再去访问这个指针就会出错：

```c++
#include <cstdio>
#include <vector>
using namespace std;

vector<int> vec;

int main()
{
    vec.push_back(0);
    auto it = vec.begin();
    for (int i = 1; i <= 10; i++)
        vec.push_back(i);
    *it = 100;
    printf("%d\n", *vec.begin());
    return 0;
}
```

甚至连 `-fsanitize=undefined` 也无法检查出这一错误。

![](https://cdn.luogu.com.cn/upload/image_hosting/li1zsw6s.png)

## 实验二：赋值语句的计算顺序问题

等等，为什么赋值语句还会有计算顺序？赋值符号的左边不就是单个变量吗？

其实赋值符号的原理并没有看上去那么直观。赋值符号需要分别计算出左边表达式指向的内存地址，以及右边表达式的值，然后才能把右边的值分配给左边的地址。问题是在 C++14 及以前的标准中并没有规定要先算左边还是先算右边，而 GCC 的处理方式是先算左边。可以通过这份代码进行验证：

```c++
#include <cstdio>
using namespace std;

struct RefChecker
{
    int val;
    int &operator[](int idx)
    {
        printf("get address\n");
        return val;
    }
};

int getVal()
{
    printf("get value\n");
    return 616;
}

int main()
{
    RefChecker rc;
    rc[0] = getVal();
    return 0;
}
```

分别用 C++14 和 C++17 进行编译，运行结果如下：

![](https://cdn.luogu.com.cn/upload/image_hosting/4ow9y3nu.png)

由于在 [C++17 中规定](https://zh.cppreference.com/w/cpp/language/eval_order)了赋值语句必须先算右边的值、再算左边的内存地址，所以可以看到不同语言标准下输出的顺序是不一样的。

> 19\. 每个简单赋值表达式 E1 = E2 和每个复合赋值表达式 E1 @= E2 中，E2 的每个值计算和副作用都按顺序早于 E1 的每个值计算和副作用。

## 结论

在 C++14 下，执行 `sgt[t].ls = build(l, mid);` 这句话时，程序是这样做的：

1. 计算出 `sgt[t].ls` 所对应的内存地址；
2. 调用 `build` 函数，获得一个新节点的下标；
3. 把第二步算出的值分配给第一步获得的内存地址。

但问题是，`build` 函数本身会通过 `newNode` 开新点，从而可能导致 `sgt` 发生内存重新分配，于是在第一步计算出的内存地址就变成无效的了，从而导致第三步的赋值过程出错。

如果换成 C++17，那么顺序就变了：

1. 调用 `build` 函数，获得一个新节点的下标；
2. 计算出 `sgt[t].ls` 所对应的内存地址；
3. 把第一步算出的值分配给第二步获得的内存地址。

尽管 `build` 函数会导致 `sgt` 重新分配内存，但是在第二步获得的内存地址一定是准确的，这样赋值就没有问题。

如果一定要在 C++14 下改变前两步的顺序，解决方法是用临时变量先把右边的值存下来：

```c++
int res = build(l, mid);
sgt[t].ls = res;
res = build(mid + 1, r);
sgt[t].rs = res;
```
