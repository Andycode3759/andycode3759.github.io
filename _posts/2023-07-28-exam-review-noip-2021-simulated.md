---
title: 考试总结 - NOIP 2021 模拟测试
date: 2023-07-28 17:21:02 +0800
categories: [考试总结]
tags: [noip]
math: true
---

## `number` - 报数（以及为什么第一题就挂了）

1. 基本功不够：由于竞赛入门前的大部分知识都是粗糙地自学，以及数论相关题目做的偏少，导致质因数分解这类基础算法都掌握不牢靠，从而有“畏惧心理”。**数论方面知识需要进行一次系统的查缺补漏。**
2. 对拍方式有问题：考试的时候对拍只用了大样例 4 里的前 10000 个数，不知是机缘巧合还是 bug，用 VSCode 自带的对比文件没能查出问题，就以为自己写对了。但实际上样例都过不了。**今后对拍不能只对固定的一组数据下手，一定要自己写造数据程序，要用熟悉的可靠的文本对比方式。**
3. 码风问题：由于在高中搞竞赛前学编程一直都是以“实用主义”至上，故码风和思维模式等方面都比较倾向于实际工程，比如“冗长”的变量名和“优雅细腻”的函数调用方法等。但**竞赛是竞赛，工程是工程，竞赛代码就是用来做题的，好不好看无所谓，能出答案的就是好代码。**但为什么学了快一年竞赛还是没有改变这个习惯？其根本原因还是**平时训练量不够，做题做少了。**

**以上三点，痛定思痛，保证以后不再犯相同的低级错误。**

首先最无脑的做法就是枚举一个数的全部因数，然后对每一个因数判断是否含 7 来决定这个数能不能报。能骗到 50 分。

```c++
bool check7(int x)
{
    while(x>0)
    {
        if(x%10==7) return true;
        x/=10;
    }
    return false;
}

bool isOk(int x)
{
    if(check7(x)) return false;
    for(int i=2;i<=x/2;i++)
    {
        if(x%i==0 && check7(i)) return false;
    }
    return true;
}

int main()
{
    int T;
    scanf("%d",&T);
    while(T--)
    {
        int x;
        scanf("%d",&x);
        if(!isOk(x))
        {
            printf("-1\n");
            continue;
        }
        do x++; while(!isOk(x));
        printf("%d\n",x);
    }
}
```

稍微分析一下，从满足特定数的倍数关系这个条件，应该联想到埃氏筛法。可以把所有带 7 的数给判断出来，然后把它们的倍数都打上标记，表示这个数不能报。询问时直接逐个枚举查询即可。

```c++
const int MAXN = 10000007;

bool isBad[MAXN];   // 为true表示这个数不能报

bool check7(int x)  // 一个数里是否含有7
{
    while(x>0)
    {
        if(x%10==7) return true;
        x/=10;
    }
    return false;
}

int n;

int main()
{
    for(int i=7;i<MAXN;i++) // 预处理
    {
        if(check7(i))
        {
            for(int j=1;j*i<MAXN;j++)
            {
                isBad[i*j]=true;
            }
        }
    }

    int T;
    scanf("%d", &T);
    while (T--)
    {
        scanf("%d",&n);
        if(isBad[n])
        {
            printf("-1\n");
            continue;
        }
        do n++; while(isBad[n]);
        printf("%d\n",n);
    }

    return 0;
}
```

这样做就已经有 70 分了。最后超时的点是因为中间 `do while` 语句跑得太慢，即每次都要枚举才能找出“大于某个数的能报的数”。可以考虑在处理询问时，顺便把这个信息记录下来，方便直接调用。

```c++
const int MAXN = 10000007;

bool isBad[MAXN];
int nxt[MAXN];  // nxt[i]为大于i的第一个能报的数（需要保证isBad[i]为false）

bool check7(int x)
{
    while(x>0)
    {
        if(x%10==7) return true;
        x/=10;
    }
    return false;
}

int n;

int main()
{
    for(int i=7;i<MAXN;i++)
    {
        if(check7(i))
        {
            for(int j=1;j*i<MAXN;j++)
            {
                isBad[i*j]=true;
            }
        }
    }

    int T;
    scanf("%d", &T);
    while (T--)
    {
        scanf("%d",&n);
        if(isBad[n])
        {
            printf("-1\n");
            continue;
        }
        if(nxt[n]!=0)   // 有信息记录就直接查
        {
            printf("%d\n",nxt[n]);
            continue;
        }
        int t=n;
        do n++; while(isBad[n]);
        nxt[t]=n;   // 记录
        printf("%d\n",n);
    }
    return 0;
}
```

至此，（按道理来说应该）就可以毫无痛点地把这道题 AC。

## `sequence` - 数列

~~（题意比较抽象，考试时没有 100% 看懂，或许考试时的程序逻辑和题目根本就对不上）~~

如果读懂了题目，可以写一个暴力，枚举 $a$ 的所有可能状态累加答案。

```c++
typedef long long ll;
const int MAXN = 105;
const ll MOD = 998244353;

int n,m,k;
int v[MAXN];
int a[MAXN];
ll ans;

int count1(ll x)    //数出二进制里1的个数
{
	int res=0;
	while(x>0)
	{
		x-=(x&(-x));
		res++;
	}
	return res;
}

void dfs(int step,ll s,ll val)  //放第几个数，目前的S的值，目前的权值
{
	if(step>=n+1)
	{
		int cnt=count1(s);
		if(cnt<=k) ans=(ans+val)%MOD;
		return;
	}
	for(int i=0;i<=m;i++)
	{
		a[step]=i;
		dfs(step+1,s+(1<<i),val*v[i]%MOD);
	}
}
int main()
{
	scanf("%d%d%d",&n,&m,&k);
	for(int i=0;i<=m;i++)
	{
        scanf("%d",v+i);
    }
	dfs(1,0,1);
	printf("%lld\n",ans);
	return 0;
}
```

## `variance` - 方差

有这样一个事实：对于一个给定的数列，经过任意次操作后能够达到的数列状态数量一定是有限的。基于这一点，比较暴力的思路就是用搜索的方法把这些状态全部找出来，然后对所有的状态都求一次答案取最小即可。这一份代码专门针对 $n=4$ 的情况（居然还有 12 分），但思路与此一致：

```c++
const int MAXN = 10004;

int n;
int arr[MAXN];

int ans;
bool vis[40010];
void dfs(int state)
{
    if (vis[state])
        return;
    vis[state] = true;
    int arr1[6], arr2[6];
    for (int i = 1; i <= 4; i++)    //解码状态
    {
        arr2[i] = arr1[i] = state % 11;
        state /= 11;
    }
    arr1[2] = arr1[1] + arr1[3] - arr1[2];  //在i=2处操作
    arr2[3] = arr2[2] + arr2[4] - arr2[3];  //在i=3处操作
    int state1 = 0, state2 = 0;
    for (int i = 4; i >= 1; i--)    //编码状态
    {
        state1 = state1 * 11 + arr1[i];
        state2 = state2 * 11 + arr2[i];
    }
    dfs(state1);
    dfs(state2);
}

int main()
{
    arr[0] = 1;
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", arr + i);
    }
    if (n > 4)  // 只能处理n=4的情况，超出范围就随便输出个数意思一下
    {
        printf("%d\n", n * n);
        return 0;
    }
    int state = 0;
    for (int i = 4; i >= 1; i--)    //把初状态编码
    {
        state = state * 11 + arr[i];
    }
    dfs(state);

    int ans = (1 << 30) - 1;
    for (int i = 0; i < 40000; i++) //枚举所有可能的状态
    {
        if (!vis[i])
            continue;
        int s = i;
        for (int j = 1; j <= 4; j++)    //将状态解码，还原成数组，方便求答案
        {
            arr[j] = s % 11;
            s /= 11;
        }
        double avg = (arr[1] + arr[2] + arr[3] + arr[4]) / 4.0;
        double sum = 0;
        for (int i = 1; i <= 4; i++)
        {
            sum += (arr[i] - avg) * (arr[i] - avg);
        }
        sum *= n;
        ans = min(ans, (int)round(sum));
    }
    printf("%d\n", ans);

    return 0;
}
```

## `chess` - 棋局

一道大搜索题，需要处理的信息和细节非常多。最朴素的做法就是直接在二维数组里面 DFS：

```c++
const int MAXN = 5005;
const int DIR[][2] = { {1, 0}, {-1, 0}, {0, 1}, {0, -1} };

int n, m, q;
int stateRow[MAXN][MAXN], stateCol[MAXN][MAXN];
int colAt[MAXN][MAXN], lvAt[MAXN][MAXN];

inline void init()
{
    memset(stateRow, 0, sizeof stateRow);
    memset(stateCol, 0, sizeof stateCol);
    memset(colAt, -1, sizeof colAt);
    memset(lvAt, 0, sizeof lvAt);
}

bool vis[MAXN][MAXN], ok[MAXN][MAXN];
void dfs(int x, int y, int state, int col, int lv, int nrmLeft, int lstDir)
{
    /*
    x,y: 当前棋子的坐标
    state: 上一次经过的网格线的状态，-1表示未确定（起始）
    col,lv: 棋子的颜色和等级
    nrmLeft: 剩余的走普通道路的次数
    lstDir: 上一次走的方向，-1表示未确定（起始）
    */
    ok[x][y] = true;    // 最后的答案就是ok的点的数量

    /*
    无论如何都不能继续的情况：
    - 已经走了一次普通道路；
    - 已经吃了一个子。
    */
    if (nrmLeft == 0 || colAt[x][y] == (col ^ 1))
        return;
    for (int d = 0; d < 4; d++)
    {
        int dx = x + DIR[d][0];
        int dy = y + DIR[d][1];
        int ns;
        switch (d)
        {
        case 0:
            ns = stateCol[x][y];
            break;
        case 1:
            ns = stateCol[dx][y];
            break;
        case 2:
            ns = stateRow[x][y];
            break;
        case 3:
            ns = stateRow[x][dy];
            break;
        }

        /*
        下一个点不能走的情况：
        - 坐标越界；
        - 已经走过这个点；
        - 到下一个点的网格线状态为0；
        - 下一个点已经有了一个不能吃的棋子（同色或更高级）。
        */
        if (dx <= 0 || dy <= 0 || dx > n || dy > m || vis[dx][dy] || ns == 0 || colAt[dx][dy] == col ||
            lv < lvAt[dx][dy])
            continue;
        if (state == -1 || state == ns)
        {
            if (ns == 1)
            {
                vis[dx][dy] = true;
                dfs(dx, dy, ns, col, lv, nrmLeft - 1, d);
                vis[dx][dy] = false;
            }
            if (ns == 2 && (lstDir == -1 || d == lstDir)) //直行道路需要判断通向才能走
            {
                vis[dx][dy] = true;
                dfs(dx, dy, ns, col, lv, nrmLeft, d);
                vis[dx][dy] = false;
            }
            if (ns == 3)
            {
                vis[dx][dy] = true;
                dfs(dx, dy, ns, col, lv, nrmLeft, d);
                vis[dx][dy] = false;
            }
        }
    }
}
inline void clearState(bool dest[][MAXN])
{
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            dest[i][j] = false;
        }
    }
}

int main()
{
    int T;
    scanf("%d", &T);
    while (T--)
    {
        init(); // 数组这么多，一定一定要记得清空
        scanf("%d%d%d", &n, &m, &q);
        for (int i = 1; i <= n; i++)
        {
            for (int j = 1; j <= m - 1; j++)
            {
                scanf("%1d", &stateRow[i][j]);
            }
        }
        for (int i = 1; i <= n - 1; i++)
        {
            for (int j = 1; j <= m; j++)
            {
                scanf("%1d", &stateCol[i][j]);
            }
        }

        int col, lv, x, y;
        while (q--)
        {
            scanf("%d%d%d%d", &col, &lv, &x, &y);
            int ans = 0;
            clearState(ok);
            lvAt[x][y] = lv;
            colAt[x][y] = col;

            clearState(vis);
            vis[x][y] = true;
            dfs(x, y, -1, col, lv, 1, -1);
            ok[x][y] = false;

            for (int i = 1; i <= n; i++)
            {
                for (int j = 1; j <= m; j++)
                {
                    if (ok[i][j])
                        ans++;
                }
            }
            printf("%d\n", ans);
        }
    }
    return 0;
}
```

由于会爆空间，因此二维数组开不下 100% 的数据范围。而且，只要数据规模稍微大一点点，此方法就会超时。由此可知，想要多拿分，目前的 DFS 已经不行了，要另外开辟思路。

~~（貌似改成构图 BFS 可以忽略数据范围，但去重+统答案极难调试）~~

~~（实在不行就考虑特殊性质吧...但似乎也不是很好处理的样子）~~
