---
title: Random Variables
date: 2019-11-10 16:08:23
tags:
- 数据分析
- math
mathjax: true
---

本文将展示一个随机变量及其相关的概率分布是如何被统计学描述的，比如均值和方差。

<!-- more -->

## 随机变量
许多情况下，随机变量可以定义为其他随机变量之和。这些情况中最重要的是从样本均值估计总体均值。因此，我们需要一些关于随机变量的性质。

### 期望（Expected Value）

离散随机变量和连续随机变量计算公式：
- 离散变量$E(X) = \sum xP(x)$
- 连续变量$E(X) = \int_{- \infty}^{ + \infty} xf(x)dx$


几个随机变量加在一起的平均值就是它们的平均值之和，即使不同的变量在统计学上不独立。
$$E(x)=ax+b$$
对于离散情况和连续情况，证明都相当简单。唯一需要了解的是求和（或积分）的顺序可以互换和在证明的中途出现的边缘分布函数。

离散情况证明：

$$
\begin{equation}\begin{split} 
  E(X + Y) &= \sum_x \sum_y (x+y) P_{XY}(x,y) \\
  &= \sum\limits_x \sum\limits_y x P_{XY}(x,y) + \sum\limits_y \sum\limits_x y P_{XY}(x,y) \\
  &= \sum\limits_x x P_X (x) + y P_Y (y) \\
  &= E(X) + E(Y)
\end{split}\end{equation}
$$


连续变量证明:

$$
\begin{equation}\begin{split}
  E(X + Y) &= \int_x \int_y (x+y) f_{XY}(x,y) \mathrm{d}x \\\\
  &= \int_x \int_y x f_{XY}(x,y) \mathrm{d}y \mathrm{d}x + \int_y \int_x y f_{XY}(x,y) \mathrm{d}x \mathrm{d}y \\\\
  &= \int_x x f_X (x) \mathrm{d}x + \int_y y f_Y (y) \mathrm{d}y \\\\
  &= E(X) + E(Y)
\end{split}\end{equation}
$$
