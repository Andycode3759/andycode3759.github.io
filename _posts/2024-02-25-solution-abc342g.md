---
title: 题解 - ABC342G Retroactive Range Chmax 可撤销区间最值
date: 2024-02-25 20:59:44 +0800
categories: [题解]
tags: [atcoder, abc]
math: true
---

链接：[ABC342 G](https://atcoder.jp/contests/abc342/tasks/abc342_g)

## 题目大意

维护一个数列 $A$，支持如下操作：

1. 将区间 $i \in [l,r]$ 内的所有 $A_i$ 更新为 $\max (A_i,x)$；
2. 撤销某次类型 1 的操作；
3. 查询目前的某个 $A_i$。

## 解析

假设不考虑撤销，显然直接用区间修改 + 单点查询线段树即可。如果加了撤销也很简单：线段树上的每个节点不再维护值 + 标记，改成用一个 `multiset` 维护这个区间内所有已知的可能最大值。撤销直接递归到对应节点 `erase()` 即可。查询时先查找目前节点可能的最大取值，然后再递归到相应儿子里查找，最终结果取 max。

这么做是对的，因为（感性认知）：

- 修改时给某些节点插入了 $x$，撤销时对于相同的操作区间也一定会递归到相同的节点。因为没有标记的上传/下传，所以这些 $x$ 永远都在原来的地方，因此撤销的正确性是对的；
- 单点查询只要 $O(\log n)$，即线段树层数；
- 区间修改和撤销的复杂度和标准的线段树上操作是一致的，近似 $O(\log n)$。综上，时间复杂度也是对的。

技能点：数据结构设计。

## 参考代码

```c++
#include <cstdio>
#include <set>
using namespace std;
const int MAXN = 200005;

struct Operation
{
    Operation(int _l = 0, int _r = 0, int _x = 0) : l(_l), r(_r), x(_x) {}
    int l, r, x;
};
Operation qry[MAXN];

int n, q;
int arr[MAXN];

namespace SegTree
{
    multiset<int> sgt[MAXN * 4];
    void modify(int idx, int sl, int sr, int l, int r, int x)
    {
        if (l <= sl && sr <= r)
        {
            sgt[idx].insert(x);
            return;
        }
        int mid = (sl + sr) >> 1;
        int ls = idx << 1, rs = ls | 1;
        if (l <= mid)
            modify(ls, sl, mid, l, r, x);
        if (r > mid)
            modify(rs, mid + 1, sr, l, r, x);
    }
    int query(int idx, int sl, int sr, int p)
    {
        int ans = 0;
        if (!sgt[idx].empty())
            ans = *sgt[idx].rbegin();
        if (sl == sr)
            return ans;
        int mid = (sl + sr) >> 1;
        int ls = idx << 1, rs = ls | 1;
        if (p <= mid)
            ans = max(ans, query(ls, sl, mid, p));
        else
            ans = max(ans, query(rs, mid + 1, sr, p));
        return ans;
    }
    void cancel(int idx, int sl, int sr, int l, int r, int x)
    {
        if (l <= sl && sr <= r)
        {
            sgt[idx].erase(sgt[idx].lower_bound(x));
            return;
        }
        int mid = (sl + sr) >> 1;
        int ls = idx << 1, rs = ls | 1;
        if (l <= mid)
            cancel(ls, sl, mid, l, r, x);
        if (r > mid)
            cancel(rs, mid + 1, sr, l, r, x);
    }
}

int main()
{
    scanf("%d", &n);
    for (int i = 1; i <= n; i++)
    {
        scanf("%d", arr + i);
        SegTree::modify(1, 1, n, i, i, arr[i]);
    }
    scanf("%d", &q);
    for (int i = 1; i <= q; i++)
    {
        int op;
        scanf("%d", &op);
        if (op == 1)
        {
            int l, r, x;
            scanf("%d %d %d", &l, &r, &x);
            qry[i] = Operation(l, r, x);
            SegTree::modify(1, 1, n, l, r, x);
        }
        else if (op == 2)
        {
            int x;
            scanf("%d", &x);
            SegTree::cancel(1, 1, n, qry[x].l, qry[x].r, qry[x].x);
        }
        else if (op == 3)
        {
            int x;
            scanf("%d", &x);
            printf("%d\n", SegTree::query(1, 1, n, x));
        }
    }
    return 0;
}
```
