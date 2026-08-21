---
title: DSP 6 Z Transform
date: 2025-07-07 11:34:50
tags:
- note
categories:
- Digital Signal Processing
---

## Z变换

一个序列 $x(n)$ 的离散时间傅里叶变换（DTFT）为：

$$
X(e^{j\omega}) = \sum_{n=-\infty}^{+\infty} x(n)e^{-j\omega n}
$$

上式收敛条件之一是序列 $x(n)$ 必须是绝对可和的，即：

$$
\sum_{n=-\infty}^{+\infty} |x(n)| < +\infty
$$

然而在实际应用中，许多序列不是绝对可求和的，因此不能进行离散时间傅里叶变换。为此，我们引入了 Z 变换，用以处理非绝对可求和的序列。

给定 $x(n)$ ，其 Z 变换定义如下：

- 双边 Z 变换：

  $$
  X(z) = \sum_{n=-\infty}^{+\infty} x(n)z^{-n}
  $$

- 单边 Z 变换：

  $$
  X(z) = \sum_{n=0}^{+\infty} x(n)z^{-n}
  $$

令 $z = re^{j\omega}$ ，那么：

$$
X(z) = \sum_{n=-\infty}^{+\infty} x(n)r^{-n}e^{-j\omega n} = \sum_{n=-\infty}^{+\infty} [x(n)r^{-n}]e^{-j\omega n}
$$

我们可以将 $x(n)r^{-n}$ 看作一个新的序列 $y(n)$ ，则原序列 $x(n)$ 的 Z 变换可以看作是序列 $y(n)$ 的离散时间傅里叶变换（DTFT）。

**由 DTFT 的收敛条件可知，$X(z)$ 收敛的条件是序列 $y(n)=x(n)r^{-n}$ 必须是绝对可和的。**

---

## 收敛域

对任意给定序列 $x(n)$ ，使其 Z 变换收敛的所有 Z 值的结合称为收敛域（ROC）。

Z 变换存在的条件是**级数绝对可和**，即：

$$
\sum_{n=-\infty}^{+\infty} |x(n)z^{-n}| < +\infty
$$

满足上述不等式的 $z$ 的取值范围称为收敛域（ROC）。

通常来说：

- 对于因果序列，其 Z 变换的收敛域 ROC 为某个圆外区域，含 $\infty$ ，不含0。

- 对于反因果序列，其 Z 变换的收敛域 ROC 为某个圆内区域，含0（即原点），不含 $\infty$ 。

- 对于双边序列，其 Z 变换的收敛域为环状区域，不含 $\infty$ ，不含0。

在之后的例子中会体现这些结论。

---

## 零点与极点

常用的 Z 变换是有理函数，我们可以将其写为两个多项式之比：

$$
X(z) = \frac{P(z)}{Q(z)}
$$

- 零点：$P(z)$ 的根。

- 极点：$Q(z)$ 的根。

我们注意到，在极点处 Z 变换不存在，因此收敛域不包括极点，收敛域总是用极点限定其边界的。

---

**例 1**

考虑如下序列的 Z 变换及其收敛域：

$$
x[n] = \alpha^n u[n]
$$

则 Z 变换为：

$$
X(z) = \sum_{n=0}^{+\infty} \alpha^n z^{-n} = \sum_{n=0}^{+\infty} (\frac{\alpha}{z})^n
$$

当 $|\frac{\alpha}{z}| < 1$ 时，级数收敛，$X(z) = \frac{1}{1-\alpha z^{-1}} $ ，收敛域为 $|z| > |\alpha|$ 。

![](https://ref.xht03.online/202504211816989.png)

---

**例 2**

考虑如下序列的 Z 变换及其收敛域：

$$
x[n] = -\alpha^n u[-n-1]
$$

则 Z 变换为：

$$
X(z) = -\sum_{n=-\infty}^{-1} \alpha^n z^{-n} = -\alpha^{-1} z \sum_{n=0}^{+\infty} \alpha^{-n} z^{n} 
$$

当 $|\alpha^{-1}z| < 1$ 时，级数收敛，

$$
X(z) = -\frac{-\alpha^{-1}z}{1-\alpha^{-1}z} = \frac{1}{1-\alpha z^{-1}}
$$

收敛域为 $|z| < |\alpha|$ 。

![](https://ref.xht03.online/202504211829908.png)

从例 1 和例 2 中我们可以看出，不同的 $x(n)$ 可能有相同的 Z 变换，但收敛域不同。为了保证由逆 Z 变换求出的序列是唯一的，我们必须指明收敛域。

---

**例 3**

考虑如下序列的 Z 变换及其收敛域：

$$
x(n) = \left(\frac{1}{2}\right)^n u[n] - 2^{n} u[-n-1]
$$

则 Z 变换为：

$$
\begin{aligned}
X(z) &= \sum_{n=0}^{+\infty} \left(\frac{1}{2}\right)^n z^{-n} - \sum_{n=-\infty}^{-1} 2^{n} z^{-n}\\
&= \frac{1}{1-\frac{1}{2}z^{-1}} + \frac{1}{1-2z^{-1}}\\
\end{aligned}
$$

收敛域为 $\frac{1}{2} < |z| < 2$ 。

一般情况下，双边序列 $X(z)$ 的 ROC 是 z 平面上一个以原点为中心的圆环。

---

## 常用 Z 变换

- $\delta[n]$

  $$
  X(z) = \sum_{n=-\infty}^{+\infty} \delta[n]z^{-n} = 1
  $$

- $u[n]$

  $$
  X(z) = \sum_{n=0}^{+\infty} z^{-n} = \frac{1}{1-z^{-1}} = \frac{1}{1-z^{-1}} \quad (|z| > 1)
  $$

- $nu[n]$

  $$
  X(z) = \sum_{n=0}^{+\infty} n z^{-n} = \frac{z^{-1}}{(1-z^{-1})^2} \quad (|z| > 1)
  $$

- $\alpha^n u[n]$

  $$
  X(z) = \sum_{n=0}^{+\infty} \alpha^n z^{-n} = \frac{1}{1-\alpha z^{-1}} \quad (|z| > |\alpha|)
  $$

- $n \alpha^n u[n]$

  $$
  X(z) = \sum_{n=0}^{+\infty} n \alpha^n z^{-n} = \frac{\alpha z^{-1}}{(1-\alpha z^{-1})^2} \quad (|z| > |\alpha|)
  $$

- $u[-n-1]$

  $$
  X(z) = \sum_{n=-\infty}^{-1} z^{-n} = \frac{1}{1-z^{-1}} \quad (|z| < 1)
  $$

- $-\alpha^{n} u[-n-1]$

  $$
  X(z) = \sum_{n=-\infty}^{-1} -\alpha^{n} z^{-n} = \frac{1}{1-\alpha z^{-1}} \quad (|z| < |\alpha|)
  $$

---

## Z 反变换

Z 反变换是将 Z 变换的结果 $X(z)$ 和 ROC 转换为时域序列 $x(n)$ 的过程。

Z 反变换的有如下三种常见计算方法：

1. 幂级数法
  
   通过将 $X(z)$ 展开为幂级数来求解。

   $$
   X(Z) = a_0 + a_1 z^{-1} + a_2 z^{-2} + \cdots
   $$

   显然，系数 ${a_0, a_1, a_2, \cdots}$ 就是序列 $x(n)$ 。

2. 部分分式法
  
   将 $X(z)$ 化为部分分式的形式，然后利用已知的 Z 变换表来求解。

   $$
   X(z) = \sum_{i} \frac{A_i}{1-a_i z^{-1}}
   $$

3. 留数法

    留数法是利用复变函数的留数定理来求解 Z 反变换。
  
    $$
    x(n) = \frac{1}{2\pi j} \oint_{C} X(z) z^{n-1} dz
    $$
  
    其中 $C$ 是包含 ROC 的闭合曲线。

---

**例 4**

考虑如下 Z 变换：

$$
X(z) = \frac{3-\frac{5}{6}z^{-1}}{(1-\frac{1}{4}z^{-1})(1-\frac{1}{3}z^{-1})} \quad \frac{1}{4} < |z| < \frac{1}{3}
$$

将 $X(z)$ 进行裂项：

$$
X(z) = \frac{1}{1-\frac{1}{4}z^{-1}} + \frac{2}{1-\frac{1}{3}z^{-1}}
$$

第一个分式的 ROC 为 $|z| > \frac{1}{4}$ ，第二个分式的 ROC 为 $|z| > \frac{1}{3}$ 。那么：

$$
x(n) = \frac{1}{4^n} u[n] - 2 \cdot \frac{1}{3^n} u[-n-1]
$$
