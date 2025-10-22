---
title: 关于对拍的二三事
date: 2023-10-05 16:56:07 +0800
categories: [未分类]
tags: []
math: true
---

> UPD 2023/10/8：
>
> - 修复了关于 `shuffle` 函数描述的一个小问题；
> - 新增小节 `Chapter 2/在快速生成数据时提高数据密度`；
> - 新增小节 `Chapter 3/特殊题目的对拍方法`。
>
> UPD 2023/11/3：
>
> - 新增小节 `Chapter 2/简单提一句：关于随机数和 mt19937`；
> - 小修小补。

作为一位无数次因为没有对拍而挂题/对拍把题对挂了的蒟蒻，笔者根据经验把有关对拍的方式方法、实用技巧和注意事项等整理成本文，既是对自己的反思总结，也可供各位读者参考。

## Chapter 1：什么是对拍？对拍有什么用？

对于一道信息竞赛题目，大多数情况下都要编写多个解法。其中有的解法看起来很“蠢”，即效率低下，但正确性是显然的，也就是常说的暴力；其他解法有的是对暴力的进一步优化（比如暴力使用 DFS，加一处剪枝成为新的解法），也有的是从不同角度思考得到的新的解法。对于这些其他的解法，虽然很多时候它们比暴力要快，但我们很难甚至无法证明它一定是正确的。这时就可以**使用对拍来对解法的正确性进行验证**。

如果加入对拍，解一道题的大概步骤如下：

1. 写出暴力解法；
2. 尝试得到更优的解法，例如优化暴力或者使用其他算法；
3. 编写数据生成器和对拍器；
4. 通过对拍和/或 Hack 的方式找出更优解里的错误，并进行调试；
5. 直到找到一个满意的解法。

## Chapter 2：如何编写数据生成器？

数据生成器，顾名思义，就是可以生成满足题目要求的一组数据的程序。生成的数据一般需要保证**随机性**和**正确性**，即足够随机且能够覆盖所有情况，并且严格满足题目的输入数据要求。先来看一道简单的例题：

> [洛谷 P2392 / kkksc03 考前临时抱佛脚](https://www.luogu.com.cn/problem/P2392)
>
> 输入格式：
>
> - 第一行为四个整数 $s_1,s_2,s_3,s_4$；
> - 第二行为 $s_1$ 个整数 $A_1,A_2,...,A_{s_1}$；
> - 第三行为 $s_2$ 个整数 $B_1,B_2,...,B_{s_2}$；
> - 第四行为 $s_3$ 个整数 $C_1,C_2,...,C_{s_3}$；
> - 第五行为 $s_4$ 个整数 $D_1,D_2,...,D_{s_4}$。
>
> 数据范围：$1 \leq s_1,s_2,s_3,s_4 \leq 20,1 \leq A_i,B_i,C_i,D_i \leq 60$。

因为我们需要编写的是数据生成器，因此只要关心输入格式和数据范围就可以了。数据生成器本质上就是一个正常的 C++ 程序，一般来说不需要接收输入，同时把数据输出到文件内。

题意很明确，生成四个随机整数 $s_1,s_2,s_3,s_4$，再把它们分别作为数量生成 $A,B,C,D$ 四个数组。一般的生成随机数做法是使用 `stdlib` 里的 `rand` 函数，但是这个随机数函数的“质量”比较低，在极端情况下不能保证随机性。此处介绍另一种更常用的随机数生成方法：`mt19937`。具体用法请见下面代码：

<!--prettier-ignore-start-->
```c++
// 对于那些不喜欢用万能头的选手（比如我自己），请在开头引入<random>头文件，即：
// #include <random>
#include <bits/stdc++.h>
using namespace std;

// 创建一个“随机数生成器”的对象，名为rng
mt19937 rng;
// rng的本质是一个“仿函数”，即重载了括号运算符的结构体。你可以直接把它当作函数一样调用，比如：
// int num = rng();
// 此时rng()的返回值是一个[0,2147483647]（即int范围）内的随机整数。
// 如果需要指定最大最小值，可以把它封装成下面这样的函数：
inline int randInt(int l, int r)
{
    return rng() % (r - l + 1) + l; // 返回一个[l,r]范围内的随机整数
}

int s[5];

int main()
{
    freopen("data.in", "w", stdout);
    // rng是可以指定随机种子的，类似于srand(time(0))。我们需要在main函数开头加上这样一行：
    rng.seed(time(0)); // 将系统时间作为随机数种子

    for (int i = 1; i <= 4; i++)
    {
        s[i] = randInt(1, 20);
        printf("%d ", s[i]);
    }
    printf("\n");

    for (int i = 1; i <= 4; i++)
    {
        for (int j = 1; j <= s[i]; j++)
        {
            printf("%d ", randInt(1, 60));
        }
        printf("\n");
    }
    return 0;
}
```
{: file="compare.cpp" }
<!--prettier-ignore-end-->

不像 $s_1,s_2,s_3,s_4$，这里的 $A,B,C,D$ 数组在每个元素生成后直接输出，而不是先记录后输出。这是因为生成器后面并不需要用到这些数据，保存它们没有任何意义。

将源代码编译，每运行一次生成器程序，就可以得到一组输入数据。

### 常见特殊数据的生成套路

与真正解题一样，编写数据生成器需要具体问题具体分析，着重需要关注的是题目对输入数据的格式和性质要求。有一些很常用的特殊数据生成方法（随机排列、随机图等）可以当作模板记下来。

#### 1. 随机排列

`shuffle` 函数可以将一段序列内的元素进行随机打乱，可以利用它来生成给定元素集合的随机排列。这里以 $[1,n]$ 内所有整数的随机排列为例：

```c++
#include <bits/stdc++.h>
using namespace std;

mt19937 rng;
inline int randInt(int l, int r)
{
    return rng() % (r - l + 1) + l;
}

int arr[1003];

int main()
{
    freopen("data.in", "w", stdout);
    rng.seed(time(0));

    int n = randInt(1, 1000);
    printf("%d\n", n);
    for (int i = 1; i <= n; i++)
    {
        arr[i] = i;
    }
    // shuffle的前两个参数是两个指针，分别是序列的第一个元素所在位置和最后一个元素的后一个位置。
    // 把rng作为第三个参数传给shuffle作为随机数来源。
    shuffle(arr + 1, arr + 1 + n, rng);
    for (int i = 1; i <= n; i++)
    {
        printf("%d ", arr[i]);
    }
    printf("\n");
    return 0;
}
```

读者可能听说过或用过 `random_shuffle`，它的作用与 `shuffle` 是完全一致的，并且可以省略第三个参数（此时会使用 `cstdlib` 里的 `rand` 进行随机控制）。但是前者在 C++14 已经被标记为弃用，在 C++17 被正式移除标准库。笔者推荐以后者代替使用 ~~，毕竟少打 7 个字符有啥不好~~。

#### 2. 随机连通图/随机树

图是一种具有实际意义的数据结构，生成图就不能无脑输出一堆随机数了。可以采用构造的方法：一开始图上只有一个 $1$ 号点，接着依次取 $2$ 到 $n$ 号点加入图中，每添加一个点就让它和已有图内的随机若干个点连边，最终得到的图就是连通且随机的。

```c++
// 此处开始省略rng框架，只给出必要的函数实现代码

// 生成一个[1,n]的随机排列
inline vector<int> getRandomPerm(int n)
{
    vector<int> V;
    for (int i = 1; i <= n; i++)
        V.push_back(i);
    shuffle(V.begin(), V.end(), rng);
    return V;
}

// 图的边集，保存每一条边的两个端点。
// 至于图是有向还是无向取决于读入时的处理，生成数据时无需关心。
vector<pair<int, int>> G;

int main()
{
    freopen("data.in", "w", stdout);
    rng.seed(time(0));

    int n = randInt(1, 1000);
    int t = 5;

    // 依次取出2到n号点
    for (int u = 2; u <= n; u++)
    {
        // 随机选择已有的cnt个点连边。此处t是一个阈值，
        // 用来限制一个点最多连多少条边，可以避免图过于稠密的情况。
        // 当然并不是所有情况都要限制边数，需要具体问题具体分析。
        int cnt = min(t, randInt(1, u - 1));
        // 因为不能连出重边，所以需要生成1到u-1的随机排列，从中取出前cnt个点来
        vector<int> P = getRandomPerm(u - 1);
        for (int i = 0; i < cnt; i++)
        {
            int v = P[i];
            // 随机调换端点。如果这是一个有向图就相当于调转方向。
            if (randInt(0, 1) == 1)
                G.push_back({u, v});
            else
                G.push_back({u, v});
        }
    }
    printf("%d %d\n", n, G.size());
    // 将边打乱顺序后输出
    shuffle(G.begin(), G.end(), rng);
    for (auto p : G)
    {
        printf("%d %d\n", p.first, p.second);
    }
    return 0;
}
```

在实际运用中，题目对图往往还有诸多其他条件限制，比如需要附加点权/边权、要求有向无环图、允许图不连通、限制点的度数范围、允许重边、有向图允许双向边......一篇文章不可能将这些条件的实现全部收录，关键还是具体问题具体分析。

接下来讲一讲生成树。树作为一种特殊的图，可以套用上面生成图的方法，只不过每次新加入的点只需要选择 1 个已有点连边就可以了。

```c++
vector<pair<int, int>> G;

int main()
{
    freopen("data.in", "w", stdout);
    rng.seed(time(0));

    int n = randInt(1, 1000);

    for (int u = 2; u <= n; u++)
    {
        int v = randInt(1, u - 1);
        G.push_back({u, v});
    }
    // 如果明确了图是一棵树，一般不需要给出边的数量
    printf("%d\n", n);
    shuffle(G.begin(), G.end(), rng);
    for (auto p : G)
    {
        printf("%d %d\n", p.first, p.second);
    }
    return 0;
}
```

这样生成的树可能不是最理想的，但已经可以满足大部分情况下的需求了。有另一种基于 [Prufer 序列](https://oi-wiki.org/graph/prufer/)的更优秀的随机树生成算法，本文不再展开介绍，有兴趣的读者可自行学习。

如果输入树的形式不是一般的无向图形式，而是明确的父子关系形式（例如输入每个点的父亲，或者输入每个节点的儿子列表），则需要在已经生成的树上选择一个树根进行 DFS，求出父子关系后再输出。同样的，对于其他形式的条件限制或者输出格式，我们能做的只有具体问题具体分析。

### 在快速生成数据时提高数据密度

上面在初始化随机数种子时，我们使用的是目前的系统时间。但是这会导致和 `srand(time(0))` 一样的问题，那就是在同一秒内生成的数据是完全相同的。在程序一秒内多次调用数据生成器的情况下，这个问题就会导致实际对拍的样例数量比我们所预期的要少很多。比如对拍器一秒内可以调用 20 次数据生成器，设定对拍 1000 组样例，那么实际对拍的有效样例只有 $1000/20=50$ 组，剩下的 $950$ 组都是在作无意义的重复运算。

如何解决这一问题呢？我们可以让生成器从外部读入一个种子，而这个种子可以是上一次数据生成结束后留下的一个随机数。就像这样：

```c++
int main()
{
    freopen("data.in", "w", stdout);
    // 为了避免与重定向过的stdout冲突，需要新开一个文件流
    FILE *seed;
    seed = fopen("seed.txt","r");
    int s;
    fscanf(seed,"%d",&s);
    rng.seed(s);
    fclose(seed);

    /* 进行正常的数据生成操作 */

    // 把输入流改成输出流，因此开头段最后的fclose(seed)是必不可少的
    seed = fopen("seed.txt","w");
    fprintf(seed,"%ld",rng());
    fclose(seed);
    return 0;
}
```

这时种子会保存在 `seed.txt` 文件内。第一次运行生成器前，需要先创建这个文件，并且随意填入一个数字作为初始种子。这样每次生成数据时，都会继承上一次生成器留下的一个随机种子，从而让种子与时间脱离关系。

### 简单提一句：关于随机数和 `mt19937`

`mt19937` 是基于梅森旋转算法设计的随机数生成器，其拥有长度为 $2^{19937}-1$ 的循环节，因此几乎不需要担心碰到环，我们从而说它生成的随机数是“高质量”的。而 `stdlib` 里的 `rand()` 之所以说它质量低，就是因为它的循环节长度通常很小，如果再加上上一段提到的保存种子的技巧，那么出现重复种子的概率很高，导致对拍永远做无意义的重复运算。

`mt19937` 同样被广泛用于各种随机化算法的实现中，如模拟退火、爬山算法、随机哈希等等。

`mt19937_64` 是 `mt19937` 的兄弟，从名字也能猜出来，它可以生成 64 位随机整数（即 `long long` 范围）。两者的用法完全一致，此处不再赘述。

## Chapter 3：如何编写对拍器？

啰嗦了这么多，总算来到了本文的重点。网络上关于对拍编写的教程数不胜数，实现方法也是五花八门，例如使用 Python（NOI Linux 包含 Python 环境，只是不能拿来解题）、Bash 脚本（`.sh`）或者 Windows 批处理（`.bat`），但大多数人估计没有精力去专门为对拍去学习第二门语言，因此笔者认为 C++ 实现的对拍器才是最方便高效的。

**请注意，本文以 Linux 环境为准，Windows 下的对拍实现可能会略有区别。**

对拍器一共需要做四件事：编译程序、生成数据、运行程序、对比输出。利用 `system` 函数可以在 C++ 程序中调用系统命令，因此对拍器的过程有点类似于模拟手动调试。下面直接给出笔者常用的一个对拍器框架：

<!--prettier-ignore-start-->
```c++
#include <bits/stdc++.h>
using namespace std;

// 需要测试的样例数量
const int OKCount = 10;

int main()
{
    // 先把所有需要用到的程序编译好，这里的problem是实际的题目ID
    system("g++ -O2 -w generator.cpp -o generator");
    system("g++ -O2 -w problem.cpp -o problem");
    system("g++ -O2 -w problem-force.cpp -o problem-force");
    for (int Kase = 1; Kase <= OKCount; Kase++)
    {
        printf("Run Case #%d...\n", Kase);
        system("./generator");
        system("./problem");
        system("./problem-force");
        // Linux下的diff命令可以用于对比两个文件的差异，
        // 其中-ZB选项可以过滤所有的行尾空格和多余空行
        int sig = system("diff -ZB problem.out problem-force.out");
        // system函数的返回值就是所调用程序的返回值，diff会在文件出现差异时返回非0值
        if (sig != 0)
        {
            printf("Wrong Anwser\n");
            return 1;
        }
    }
    printf("Everything is OK\n");
    return 0;
}
```
{: file="compare.cpp" }
<!--prettier-ignore-end-->

如果对拍完毕后没有发现任何问题，对拍器就会输出 `Everything is OK`；如果在中途发现了答案不匹配，对拍器会停止对拍并输出 `Wrong Anwser`。这时就可以在目录下查看输入文件、暴力输出和正解输出，用来对正解进行调试。一般来说，令人信服的正确解法需要通过大约 1000~2000 个样例的对拍（当然是在生成的数据满足随机性和正确性的情况下）。

需要注意的是，如果在运行中的对拍器按 `Ctrl+C` 终止程序，只能终止目前 `system` 正在调用的程序，而不会终止对拍器本身的运行。有时候这个特性会导致对拍难以终止，具体的解决方案会在下文给出。

### 对拍器功能拓展

上面给出的对拍器只能够查出答案错误，对于可能出现的运行错误（尽管它往往也会导致答案错误）甚至编译错误则会忽略。针对这些情况，有必要拓展对拍器的功能。

#### 1. 检查编译错误

对于不能编译的程序，尝试运行它没有任何意义。`g++` 编译器会在编译失败时返回非 0 的值，只需要接收它的返回值并进行检查，提前退出即可：

```c++
int sig = 0;
sig |= system("g++ -O2 -w generator.cpp -o generator");
sig |= system("g++ -O2 -w problem.cpp -o problem");
sig |= system("g++ -O2 -w problem-force.cpp -o problem-force");
if (sig != 0)
{
    printf("Compile Error\n");
    return 1;
}
```

#### 2. 检查运行错误

一般的 C++ 程序在遇到比较“严重”的运行错误时（比如栈溢出、段错误等）会被系统强制终止并返回一个非 0 值。我们可以接受答案程序的返回值进行检查：

```c++
int sig = 0;
if (system("./problem") != 0)
    sig |= 1;
if (system("./problem-force") != 0)
    sig |= 2;
if (sig != 0)
{
    if (sig & 1)
        printf("std: Runtime Error\n");
    if (sig & 2)
        printf("force: Runtime Error\n");
    return 1;
}
```

这里没有直接保存程序的返回值，而是按位存储哪个程序出现了问题。这么做有两点好处：首先可以明确地看到是哪个程序出现了运行错误；同时，先运行的程序出现错误不会取消后面的程序的运行，以便得到一个可供调试时参考的输出。

一个有意思的小知识：当运行中的程序被 `Ctrl+C` 强制终止或者被 `kill` 杀死时，也会返回一个非 0 值。这也就意味着，如果在上面的对拍器运行过程中按 `Ctrl+C`，它也会输出 `Runtime Error` 并停止运行。

然而，很多时候，小幅度的数组越界并不会触发系统的段错误，例如在已经定义了 `int a[100];` 的情况下尝试访问 `a[-1]` 或 `a[105]`。这显然是不对的，那有没有什么方法可以查出来呢？编译选项 `-fsanitize=undefined` 闪亮登场：

```c++
system("g++ -O2 -w -fsanitize=undefined problem.cpp -o problem");
system("g++ -O2 -w -fsanitize=undefined problem-force.cpp -o problem-force");
```

我们需要在编译答案程序的时候加上这个编译选项。这样当程序运行的过程中，如果出现了任何的数组越界行为，程序都会往 `stderr` 输出对应的错误信息（但并不会停止程序运行，也不会改变程序的返回值）。此时就可以按 `Ctrl+C` 终止对拍了。具体的运行效果如下：

<!--prettier-ignore-start-->
```c++
#include <cstdio>
using namespace std;

int a[100];

int main()
{
    int x;
    scanf("%d", &x);
    a[x] = x;
    printf("%d\n", a[x]);
    return 0;
}
```
{: file="tmp.cpp" }

```terminal
# 编译选项：g++ tmp.cpp -o tmp
输入：
105
输出：
105
```

```terminal
# 编译选项：g++ -fsanitize=undefined tmp.cpp -o tmp
输入：
105
输出：
tmp.cpp:10:8: runtime error: index 105 out of bounds for type 'int [100]'
tmp.cpp:10:10: runtime error: store to address 0x556c1ba37344 with insufficient space for an object of type 'int'
0x556c1ba37344: note: pointer points here
  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
              ^
tmp.cpp:11:23: runtime error: index 105 out of bounds for type 'int [100]'
tmp.cpp:11:11: runtime error: load of address 0x556c1ba37344 with insufficient space for an object of type 'int'
0x556c1ba37344: note: pointer points here
  00 00 00 00 69 00 00 00  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
              ^
105
```
<!--prettier-ignore-end-->

需要注意的是，`-fsanitize=undefined` 不仅可以检查数组越界，还可以检查整型溢出（妈妈再也不用担心我爆 `int` 了！）。尽管很方便，但在一些特殊情况下这个特性并不是我们想要的，比如利用 `unsigned` 类型溢出自动取模的哈希算法。这就需要我们理智分析 `stderr` 里的输出信息。同时，`-fsanitize=undefined` 会导致程序运行时的时间常数增大很多，如果需要对程序进行时间复杂度估计，请不要在编译时加上这个选项。

#### 3. 记录正解程序用时

C++ 程序内部计时一般使用 `clock()` 函数，但可惜的是它不能够记录 `system` 调用外部程序所经过的时间。正确的做法是使用系统自带的 `time` 命令：

```c++
// 由于Bash的问题，带参数调用time命令需要使用完整目录。
// 这里的 -f \"%es\" 是控制time命令的输出格式，此处的格式会简单地输出一个运行时长秒数。
// time的格式控制功能很丰富，语法类似于printf，而且可以记录除时间以外的很多信息（例如内存占用等），
// 有兴趣的读者可以自己在机子上用 man time 命令查看说明文档。
system("/usr/bin/time -f \"%es\" ./problem");
```

这么做也有一个缺陷，就是对拍器内部无法记录时间，只能将结果直接打印在屏幕上。

### 特殊题目的对拍方法

上述的对拍方法只适用于进行文件比较的传统题。而对于需要特判（Special Judge）的题目或者是交互题（Interactive Problem），则需要改用其他的对拍方法。

#### 1. 特判题的对拍

特判题答案的验证需要一个校验器（Checker）程序。如果题目提供了校验器，那么可以直接使用，否则需要自己动手编写。

校验器的编写与正常解题是一样的，只不过可能需要从多个文件（输出文件和标准答案）中同时读入数据。可以采用多次调用 `freopen()` 的方法，或者使用 `FILE* + fopen + fprintf()/fscanf()` 或 `fstream` 的方式手动创建多个文件输入流。并且，如果发现输出文件的答案错误，校验器应该 `return 1`，只有在答案正确时才 `return 0`。之后只需要将一般的对拍器中调用 `diff` 命令修改为调用校验器即可。

对于一些更加特殊的构造题（例如[喵了个喵](https://www.luogu.com.cn/problem/P8866)、[移球游戏](https://www.luogu.com.cn/problem/P7115)等），并不存在标准答案，仅需验证答案可行性，对它们来说甚至可以省略掉暴力程序。

#### 2. 交互题的对拍

> 冷知识：在整个 NOI 历史上[只出现过 3 道交互题](https://www.luogu.com.cn/problem/list?keyword=&tag=83,82,342,343,77%7C103&page=1)。

交互题分为 IO 交互和函数交互两种。在 IO 交互中，选手的程序需要通过输入输出来与外界进行数据交换；而函数交互则看起来更加简单，一般要求选手将程序功能封装成一个函数，通过调用给出的库函数来进行数据交换。

IO 交互的实现很复杂，需要控制答案程序/交互器的输入输出顺序。不过我们可以借助 testlib 来比较方便地实现 IO 交互（事实上 testlib 还可以辅助实现数据生成器/校验器），具体可参阅 [OI Wiki](https://oi-wiki.org/tools/testlib/interactor/)。不过考场上一般不会给选手发 testlib，所以并没有什么实战价值 ~~（NOI 历史上也没有出现过 IO 交互题目）~~。

接下来说说更简单的函数交互。我们需要实现一个调用答案函数的主程序和提供的库函数，再编译运行主程序并验证答案，即相当于把单一的答案函数包装成一个可运行的、带标准输入输出的程序。或者，你也可以把这里的主程序理解成一种特殊的特判校验器。以[洛谷 P1947 / 猜数](https://www.luogu.com.cn/problem/P1947)为例，下面给出一个用于本地校验的主程序：

<!--prettier-ignore-start-->
```c++
#include <bits/stdc++.h>
// 为防止奇怪的命名空间问题，一般不使用using namespace std;
const int LIM = 3000000; // 防止卡交互机，设置次数上限

int k, cnt;

extern "C"
{
    // 答案函数的声明
    extern int Chtholly(int n, int c);

    // 实现库函数
    extern int Seniorious(int x)
    {
        cnt++;
        if (cnt > LIM)
            cnt = LIM;
        return (k < x) ? 1 : ((k == x) ? 0 : -1);
    }
}

int main()
{
    freopen("data.in", "r", stdin);

    int n, c;
    scanf("%d %d %d", &n, &c, &k); // 读入输入数据
    int res = Chtholly(n, c);      // 调用答案程序
    if (res != k || cnt >= c)      // 可以直接在主程序内完成答案校验，类似于Special Judge
        return 1;
    else
        return 0;
}
```
{: file="main.cpp" }
<!--prettier-ignore-end-->

假设答案程序文件名为 `guess.cpp`，那么编译时应该使用 `g++ main.cpp guess.cpp -o guess`（其他类似于 `-O2` 等标记同样可以加上去）一次性**把多个 C++ 源文件编译成一个可执行文件**。相应的对拍器如下：

<!--prettier-ignore-start-->
```c++
#include <bits/stdc++.h>
using namespace std;

const int OKCount = 10;

int main()
{
    // 数据生成器的编写与正常题目相同。
    system("g++ -O2 -w generator.cpp -o generator");
    system("g++ -O2 -w main.cpp guess.cpp -o guess");
    for (int Kase = 1; Kase <= OKCount; Kase++)
    {
        printf("Run Case #%d...\n", Kase);
        system("./generator");
        // 由于这里我们把答案校验嵌入了主程序，因此不需要另外再写checker了
        int sig = system("./guess");
        if (sig != 0)
        {
            printf("Wrong Anwser\n");
            return 1;
        }
    }
    printf("Everything is OK\n");
    return 0;
}
```
{: file="compare.cpp" }
<!--prettier-ignore-end-->

## Chapter 4：正确的对拍策略

前面的文字都讲的是对拍的技术，而技术终归还是需要人正确地用才能发挥价值。下面是一些对拍的注意事项：

- **解法千万条，暴力第一条。正解不对拍，爆零两行泪。** 有多少选手明明有写出正解的实力，却因为一个粗心大意的错误+没有对拍导致痛失一整道题的分数从而无缘 NOIP/省队资格/牌子？对拍不仅是对答案正确性的检验，还是一道检查工序，只有足够数量和质量的对拍才能构建起对答案的自信。【是笔者的真实经历（[#1]({{ "/posts/exam-review-noip-2021-simulated" | relative_url }}), [#2]({{ "/posts/exam-review-2023-9-27" | relative_url }})）。】
- **对拍的前提是确保暴力的正确性和可行性。** 正确性很好理解，拿一个答案错误的暴力来对拍没有任何意义；此外还要考虑暴力的可行性，比如假设暴力的时间复杂度是 $O(2^n)$ 的，那 $n$ 就不应当设置得过大（一般来说 $n \leq 24$ 是可以接受的），否则暴力的运行时间就会是一个天文数字。
- **对拍出错，要首先排除对拍过程本身的错误。** 对拍器也是人 ~~（你）~~ 写出来的，难免可能会出现错误，需要先把对拍过程中间的错误排除，才能再去考虑答案的错误。例如数据生成器开到 $n=2000$，而暴力中某个数组只开了 $1000$，又没有加 `-fsanitize=undefined` 导致以为是答案错误，于是调试的出发点就完全错掉了。（也是笔者的[真实经历]({{ "/posts/exam-review-nowcoder-oir1" | relative_url }})。）
- **对于解 Subtask（部分分）甚至骗分，对拍同样重要。** 对拍不仅适用于验证满分代码，同样可以验证 Subtask 的做法，只要能够生成满足对应性质的数据。就算是无脑骗分，对拍也可以让你心里对能骗多少分大概有个估计 ~~（[不可以，总司令](https://www.luogu.com.cn/discuss/525529)）~~。
- **学会定向对拍。** 就像上面一条所说，定向对拍就是专门针对某种特殊情况的输入数据进行对拍，可能是用来验证某一 Subtask 的正确性，也有可能是用来 Hack 一般情况。例如某些题目可能存在整型溢出的风险，但一般的数据很难达到溢出的情况，这时就可以生成极端大的数据进行定向对拍。（同样是笔者的[真实经历]({{ "/posts/exam-review-2023-10-2" | relative_url }})。）

## 总结

在 NOIP 的不带反馈的赛制模式下，对拍的重要性不亚于数学中的验算。尽管对拍看似属于应试技巧，它实际上也对选手的程序设计能力提出了更高的要求，因此对拍是 OI 赛制下不可不掌握的基本功之一。只有数量和质量都达标的对拍，才能有效保证答案的正确性，优化调试流程，从而在考试中稳定高分。
