---
title: 一些数学命题的优美证明
date: 2024-01-06 15:45:37 +0800
categories: [未分类]
tags: [数学]
math: true
---

## `1`

**命题：** 凸 $n (n \geq 4)$ 边形有 ${ {n(n-3)} \over 2}$ 条对角线。

**证明：** 在平面内任意做凸 $n$ 边形，可得到其 $n$ 个顶点。这些顶点两两任意连线，没有线段共线，可得到共 $C_{n}^{2}$ 条线段，其中有 $n$ 条是凸 $n$ 边形的边，剩下的 $C_{n}^{2}-n={ {n(n-3)} \over 2}$ 条线段就是其对角线。证毕。

## `2`

**命题：** 已知两个不相关的离散性有穷随机变量 $X$ 和 $Y$，它们的期望分别为 $E(X)$ 和 $E(Y)$，方差分别为 $D(X)$ 和 $D(Y)$，则 $E(X+Y)=E(X)+E(Y)$，$D(X+Y)=D(X)+D(Y)$。（随机变量期望/方差的线性性）

**证明：**

(i) 设 $X$ 和 $Y$ 的分布列分别为 $P(X=x_i)=p_i(1 \leq i \leq n),P(Y=y_i)=q_i(1 \leq i \leq m)$，则

$$
E(X) = \sum_{i=1}^{n}p_ix_i, ~ E(Y) = \sum_{i=1}^{m}q_iy_i
$$

显然有

$$
\sum_{i=1}^{n}p_i=1, ~ \sum_{i=1}^{m}q_i=1,
$$

于是

$$
\begin{align*}
E(X+Y) &= \sum_{i=1}^{n} \sum_{j=1}^{m}p_iq_j(x_i+y_j) \\
       &= \sum_{i=1}^{n} p_i(x_i\sum_{j=1}^{m}q_j + \sum_{j=1}^{m}q_jy_j) \\
       &= \sum_{i=1}^{n} p_i(x_i + E(Y)) \\
       &= \sum_{i=1}^{n} p_ix_i + E(Y) \\
       &= E(X) + E(Y)
\end{align*}
$$

(ii) 方便起见，记 $dx_i = x_i-E(X), dy_i = y_i-E(Y)$。首先需要一个引理：

$$
\sum_{i=1}^{n}p_idx_i=0
$$

（其实换成 $Y$ 也一样。）证明很简单：

$$
\begin{align*}
\sum_{i=1}^{n}p_idx_i &= \sum_{i=1}^{n}p_i(x_i-E(X)) \\
                      &= \sum_{i=1}^{n}p_ix_i - E(X)\sum_{i=1}^{n}p_i \\
                      &= E(X) - E(X) \\
                      &= 0
\end{align*}
$$

由于

$$
D(X) = \sum_{i=1}^{n}p_i(x_i-E(X))^2 = \sum_{i=1}^{n}p_idx_i^2 \\
D(Y) = \sum_{i=1}^{n}q_i(y_i-E(Y))^2 = \sum_{i=1}^{m}q_idy_i^2
$$

所以

$$
\begin{align*}
D(X+Y) &= \sum_{i=1}^{n} \sum_{j=1}^{m}p_iq_j(x_i+y_j-E(X+Y))^2 \\
       &= \sum_{i=1}^{n} \sum_{j=1}^{m}p_iq_j(dx_i+dy_j)^2 \\
       &= \sum_{i=1}^{n} p_i\sum_{j=1}^{m}(q_jdx_i^2 + 2q_jdx_idy_j + q_jdy_j^2) \\
       &= \sum_{i=1}^{n} p_i (dx_i^2\sum_{j=1}^{m}q_j + 2dx_i\sum_{j=1}^{m}q_jdy_j + \sum_{j=1}^{m}q_jdy_j^2) \\
       &= \sum_{i=1}^{n} p_i (dx_i^2 + 0 + D(Y)) \\
       &= \sum_{i=1}^{n} p_idx_i^2 + D(Y) \\
       &= D(X) + D(Y)
\end{align*}
$$

证毕。

## `3`

**命题：** $\forall n \in \mathbb{N}^{+}, 1^2+2^2+3^2+ \dots +n^2 = \sum_{k=1}^{n}k^2 = {1 \over 6}n(n+1)(2n+1)$。（前 $n$ 个正整数平方和公式）

**证明：** 注意到：

$$
\begin{align*}
1^2 &= 1 \\
2^2 &= 1+3 \\
3^2 &= 1+3+5 \\
\dots \\
n^2 &= 1+3+5+ \dots +(2n-1)
\end{align*}
$$

写出 $n$ 个这样的等式，全部相加，等式左边就是 $1^2+2^2+\dots+n^2$，等式右边是：$n$ 个 $1$ 相加，$(n-1)$ 个 $3$ 相加，$(n-2)$ 个 $5$ 相加，……，$(n-k+1)$ 个 $(2k-1)$ 相加，……，$1$ 个 $(2n-1)$ 相加。于是就可以得到：

$$
\sum_{k=1}^{n}k^2 = \sum_{k=1}^{n}(n-k+1)(2k-1)
$$

化简等式右边：

$$
\begin{align*}
\sum_{k=1}^{n}(n-k+1)(2k-1) &= \sum_{k=1}^{n}(2nk-2k^2+3k-n-1) \\
                            &= \sum_{k=1}^{n}(2nk-2k^2+3k)-n(n+1) \\
                            &= (2n+3)\sum_{k=1}^{n}k - 2\sum_{k=1}^{n}k^2 - n(n+1) \\
                            &= {n(n+1)(2n+3) \over 2} - n(n+1) - 2\sum_{k=1}^{n}k^2 \\
                            &= n(n+1)({2n+3 \over 2}-1) - 2\sum_{k=1}^{n}k^2 \\
                            &= {n(n+1)(2n+1) \over 2} - 2\sum_{k=1}^{n}k^2
\end{align*}
$$

移项可得：

$$
\begin{align*}
3 \sum_{k=1}^{n}k^2 &= {n(n+1)(2n+1) \over 2} \\
  \sum_{k=1}^{n}k^2 &= {n(n+1)(2n+1) \over 6}
\end{align*}
$$

证毕。
