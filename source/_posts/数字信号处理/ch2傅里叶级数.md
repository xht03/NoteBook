---
title: DSP 2 Fourier Series
date: 2025-06-28 10:16:30
tags:
- note
categories:
- Digital Signal Processing
---

## 正交向量

在介绍正交函数之前，我们先回顾一下正交向量。这有助于我们理解正交函数。

如果两个向量 $\vec{A_1}$ 和 $\vec{A_2}$ 相互垂直，则称 $\vec{A_1}$ 和 $\vec{A_2}$ 为正交向量，记作 $\vec{A_1} \perp \vec{A_2}$ 。

接着，我们考虑平面上任意两个向量 $\vec{A_1}$ 和 $\vec{A_2}$ ，当我们想用 $\vec{A_2}$ 来近似表示 $\vec{A_1}$ 时，我们就只能用 $\vec{A_1}$ 在 $\vec{A_2}$ 上的投影来表示，但这不可避免地会带来误差。

设它们的夹角为 $\theta$ ，$\vec{A_1}$ 在 $\vec{A_2}$ 上的投影为 $c_{12}\vec{A_2}$ 。则误差为：

$$
\vec{E} = \vec{A_1} - c_{12}\vec{A_2}
$$

我们期望误差 $\vec{E}$ 最小化，即：$\|\vec{E}\|^2$ 最小化。为取到最小值，我们可以计算出$c_{12}$ 的值：

$$
\frac{d|\vec{E}|^2}{dc_{12}} = 0 \implies c_{12} = \frac{\vec{A_1} \cdot \vec{A_2}}{\|\vec{A_2}\|^2}
$$

所以，系数 $c_{12}$ 表征着 两个向量 $\vec{A_1}$ 和 $\vec{A_2}$ 的相似程度。

如果 $\vec{A_1}$ 和 $\vec{A_2}$ 是正交的，则它们的夹角为 $90^{\circ}$ ，此时 $c_{12} = 0$ ，即 $\vec{A_1}$ 和 $\vec{A_2}$ 之间最不相似。

![](https://ref.xht03.online/202504011027962.png)

---

## 正交函数

正交函数与正交向量类似。我们可以**把函数看作是一个向量空间中的向量**：一个函数在时间轴上是一个向量，函数的值是这个向量的坐标。

我们考虑两个函数 $f_1(t)$ 和 $f_2(t)$ ，在时间区间 $[t_1, t_2]$ 上。

我们希望用 $f_2(t)$ 来近似表示 $f_1(t)$ ，即：

$$
f_1(t) \approx c_{12}f_2(t) \quad t_1 < t < t_2
$$

类似的，我们可以定义误差函数为：

$$
\epsilon(t) = f_1(t) - c_{12}f_2(t)
$$

为了使 $f_1(t)$ 和 $f_2(t)$ 之间的误差最小化，我们使用**均方误差**：

$$
\overline{\epsilon^2(t)} = \frac{1}{t_2 - t_1} \int_{t_1}^{t_2} \epsilon^2(t) dt = \frac{1}{t_2 - t_1} \int_{t_1}^{t_2} [f_1(t) - c_{12}f_2(t)]^2 dt
$$

我们对 $c_{12}$ 求导数并令其为零：

$$
\frac{d\overline{\epsilon^2(t)}}{dc_{12}} = 0 \implies c_{12} = \frac{\int_{t_1}^{t_2} f_1(t)f_2(t)dt}{\int_{t_1}^{t_2} f_2^2(t)dt}
$$

与正交向量类似，系数 $c_{12}$ 表征着两个函数 $f_1(t)$ 和 $f_2(t)$ 的相似程度，称为 $f_1(t)$ 和 $f_2(t)$ 的**相关系数**。

当 $c_{12} = 0$ 时，函数 $f_1(t)$ 和 $f_2(t)$ 是**正交**的。

所以，正交函数的严格定义如下：

在区间 $[t_1, t_2]$ 上，两个函数 $f_1(t)$ 和 $f_2(t)$ 是正交的，当且仅当：

$$
\int_{t_1}^{t_2} f_1(t)f_2(t)dt = 0
$$

这里我们也可以看出，正交函数与正交向量的相似。如果把时间轴看作一个连续的“基底”（类似于向量空间中的坐标轴），那么一个函数 $f(t)$ 可以被看作在这个“基底”上定义的一个向量。在每个时间点 $t$ 上，$f(t)$ 的值可以看作这个“向量”的一个分量。因为时间轴是连续的，这个“向量”的分量是无穷多的（对应于 $t$ 的每个可能取值），而不是有限个。

---

## 正交函数集

在区间 $[t_1, t_2]$ 上的 $n$ 个函数 $f_1(t)$、$f_2(t)$、$\cdots$、$f_n(t)$ 称为正交函数集，当且仅当：

$$
\left\{
\begin{aligned}
&\int_{t_1}^{t_2} f_i(t)f_j(t)dt = 0 \quad (i \neq j) \\
&\int_{t_1}^{t_2} f_i^2(t)dt = k_i \quad (i = 1, 2, \cdots, n)
\end{aligned}
\right.
$$

这里 $k_i \neq 0$ 是常数。

---

## 正交函数的线性组合

任意一个函数 $g(t)$ 在区间 $[t_1, t_2]$ 上都可以用 $n$ 个正交函数的线性组合来近似表示：

$$
g(t) \approx c_1f_1(t) + c_2f_2(t) + \cdots + c_nf_n(t)
$$

在均方误差最小的情况下，可求解系数 $c_i$ ：

$$
\left\{
\begin{aligned}
&\overline{\epsilon^2(t)} = \frac{1}{t_2 - t_1} \int_{t_1}^{t_2} [g(t) -\sum_{i=1}^nc_if_i(t)]^2 dt \\
&\frac{d\overline{\epsilon^2(t)}}{dc_i} = 0 \quad (i = 1, 2, \cdots, n)
\end{aligned}
\right.
$$

则系数 $c_i$ 为：

$$
c_i = \frac{\int_{t_1}^{t_2} g(t)f_i(t)dt}{\int_{t_1}^{t_2} f_i^2(t)dt} = \frac{1}{k_i}\int_{t_1}^{t_2} g(t)f_i(t)dt
$$
这里 $c_i$ 是函数 $g(t)$ 在正交函数集 $\{f_1(t), f_2(t), \cdots, f_n(t)\}$ 上的投影系数。

---

## 完备正交函数集

在区间 $[t_1, t_2]$ 上，正交函数集 $\{f_1(t), f_2(t), \cdots, f_n(t)\}$ 近似表示任意一个函数 $g(t)$，如果：

$$
\lim_{n \to \infty} \overline{\epsilon^2(t)} = 0
$$

则称 $\{f_1(t), f_2(t), \cdots, f_n(t)\}$ 为**完备正交函数集**。其中 $\overline{\epsilon^2(t)}$ 是均方误差。

---

## 正交分解

上面我们介绍了完备正交函数集。所谓完备是指对任意函数 $g(t)$ 都可以用一个无穷级数表示：

$$
g(t) = \sum_{i=1}^{\infty} c_if_i(t)
$$

这个级数收敛于 $g(t)$ ，上式也就是 $g(t)$ 的正交分解。

在实际应用中，我们通常只需要有限个正交函数就可以近似表示任意函数 $g(t)$。这就是正交分解的意义所在。

---

## 三角函数集

三角函数集是一个常用的完备正交函数集。

$$
\{cos(n\omega t) , \ sin(n\omega t) \mid n = 0, 1, 2, \cdots\}
$$

在区间 $[t_0, t_0 + T]$ 上，上述无限个三角函数组成一个完备正交函数集。这里 $T$ 是周期，$T=\frac{2\pi}{\omega}$ 。

---

证明：

1. 正交性：我们下面证明，

- 当 $i \neq j$ 时，$\int_{t_0}^{t_0 + T} f_i(t) f_j(t) \, dt = 0$

- 当 $i = j$ 时，$\int_{t_0}^{t_0 + T} f_i^2(t) \, dt \neq 0$

我们考虑（其余情况类似）：

$$
\int_{t_0}^{t_0 + T} \cos(n\omega t) \cos(m\omega t) \, dt
$$

使用和差化积公式：

$$
\int_{t_0}^{t_0 + T} \cos(n\omega t) \cos(m\omega t) \, dt = \frac{1}{2} \int_{t_0}^{t_0 + T} \left[ \cos((n + m)\omega t) + \cos((n - m)\omega t) \right] \, dt
$$

又有：

$$
\int_{t_0}^{t_0 + T} \cos((n + m)\omega t) \, dt = 0, \quad \int_{t_0}^{t_0 + T} \cos((n - m)\omega t) \, dt = 0 \quad (n \neq m)
$$

因此：

$$
\int_{t_0}^{t_0 + T} \cos(n\omega t) \cos(m\omega t) \, dt = 0 \quad (n \neq m)
$$

且不难证明，当 $n = m$ 时：

$$
\int_{t_0}^{t_0 + T} \cos^2(n\omega t) \, dt = \frac{T}{2}
$$

2. 完备性

完备性是指：对于任意**平方可积**函数 $f(t)$ （即 $\int_{t_0}^{t_0 + T} |f(t)|^2 \, dt < \infty$ ），都可以用三角函数集的线性组合来表示。

我们需要证明任意平方可积的 $f(t)$ 可以写成：

$$
f(t) = \frac{a_0}{2} + \sum_{n=1}^\infty \left[ a_n \cos(n\omega t) + b_n \sin(n\omega t) \right]
$$

接下来，我们试着确定 $f(t)$ 系数 $a_n$ 和 $b_n$ 的值。

$$
\int_{t_0}^{t_0 + T} f(t) \, dt = \frac{a_0}{2} \cdot T + \sum_{n=1}^N \left[ a_n \cdot \int_{t_0}^{t_0 + T} \cos(n\omega t) \, dt + b_n \cdot \int_{t_0}^{t_0 + T} \sin(n\omega t) \, dt \right]
$$

由于 $\int_{t_0}^{t_0 + T} \cos(n\omega t) \, dt = 0$ 和 $\int_{t_0}^{t_0 + T} \sin(n\omega t) \, dt = 0$ ，所以：

$$
a_0 = \frac{2}{T} \int_{t_0}^{t_0 + T} f(t) \, dt 
$$

类似的，我们可以求出 $a_n$ 和 $b_n$ 的值：

$$
\begin{aligned}
&\int_{t_0}^{t_0 + T} f(t) \cos(n\omega t) \, dt\\
&= \frac{a_0}{2} \cdot \int_{t_0}^{t_0 + T} \cos(n\omega t) \, dt + \sum_{m=1}^N \left[ a_m \cdot \int_{t_0}^{t_0 + T} \cos(n\omega t) \cos(m\omega t) \, dt + b_m \cdot \int_{t_0}^{t_0 + T} \cos(n\omega t) \sin(m\omega t) \, dt \right]\\
&= a_n \int_{t_0}^{t_0 + T} \cos^2(n\omega t) \, dt\\
&= \frac{T}{2} a_n
\end{aligned}
$$

则有：

$$
a_n = \frac{2}{T} \int_{t_0}^{t_0 + T} f(t) \cos(n\omega t) \, dt
$$

类似的，我们有：

$$
b_n = \frac{2}{T} \int_{t_0}^{t_0 + T} f(t) \sin(n\omega t) \, dt
$$

这里得到的 $f(t)$ 的展开式，其实就是之后我们要介绍的**傅里叶级数**。

设函数 $s_N(t)$ 为：

$$
s_N(t) = \frac{a_0}{2} + \sum_{n=1}^N [ a_n \cos(n\omega t) + b_n \sin(n\omega t)]
$$

考虑 $f(t)$ 和 $s_N(t)$ 的平方误差：

$$
\int_{t_0}^{t_0 + T} \left| f(t) - s_N(t) \right|^2 \, dt
$$

展开后：

$$
\int_{t_0}^{t_0 + T} \left| f(t) - s_N(t) \right|^2 \, dt = \int_{t_0}^{t_0 + T} f^2(t) \, dt - 2 \int_{t_0}^{t_0 + T} f(t) s_N(t) \, dt + \int_{t_0}^{t_0 + T} s_N^2(t) \, dt
$$

第一项：$\int_{t_0}^{t_0 + T} f^2(t) \, dt$ 是 $f(t)$ 的 $L^2$ 范数的平方。

第二项：

$$
\begin{aligned}
\int_{t_0}^{t_0 + T} f(t) s_N(t) \, dt &= \int_{t_0}^{t_0 + T} f(t) \left( \frac{a_0}{2} + \sum_{n=1}^N \left[ a_n \cos(n\omega t) + b_n \sin(n\omega t) \right] \right) \, dt\\
&= \frac{a_0}{2} \int_{t_0}^{t_0 + T} f(t) \, dt + \sum_{n=1}^N \left[ a_n \cdot \int _{t_0}^{t_0 + T} f(t) \cos(n\omega t) \, dt + b_n \cdot \int_{t_0}^{t_0 + T} f(t) \sin(n\omega t) \, dt \right]\\
&= \frac{T}{4} a_0^2 + \frac{T}{2} \sum_{n=1}^N (a_n^2 + b_n^2)
\end{aligned}
$$

第三项：注意到，只有平方项积分后的结果不为零。

$$
\begin{aligned}
\int_{t_0}^{t_0 + T} s_N^2(t) \, dt &= \int_{t_0}^{t_0 + T} \left( \frac{a_0}{2} + \sum_{n=1}^N [ a_n \cos(n\omega t) + b_n \sin(n\omega t)] \right)^2 dt\\
&= \frac{T}{4} a_0^2 + \frac{T}{2} \sum_{n=1}^N (a_n^2 + b_n^2)\\
\end{aligned}
$$

综上，我们得到：

$$
\int_{t_0}^{t_0 + T} \left| f(t) - s_N(t) \right|^2 \, dt = \int_{t_0}^{t_0 + T} f^2(t) \, dt - \left( \frac{T}{4} a_0^2 + \frac{T}{2} \sum_{n=1}^N (a_n^2 + b_n^2) \right)
$$

注意到，上式的左边本质是一个平方求和，是非负的。因此，我们可以得到 Bessel 不等式：

$$
\int_{t_0}^{t_0 + T} f^2(t) \, dt \geq \frac{T}{4} a_0^2 + \frac{T}{2} \sum_{n=1}^N (a_n^2 + b_n^2)
$$

回到我们的证明，我们还需要证明：

$$
\lim_{N \to \infty} \int_{t_0}^{t_0 + T} \left| f(t) - s_N(t) \right|^2 \, dt = 0
$$

由于 $f(t)$ 是平方可积的，所以：$\int_{t_0}^{t_0 + T} f^2(t) \, dt$ 是有极限的。而随着 $N$ 的增大，$\frac{T}{4} a_0^2 + \frac{T}{2} \sum_{n=1}^N (a_n^2 + b_n^2)$ 也会增大。由于：单调有界，必有极限，所以得证。

---

## 傅里叶级数

傅里叶级数是一个重要的数学工具，它可以把一个**周期函数**表示为一组正弦和余弦函数的**线性组合**。

傅里叶级数的形式为：

$$
f(t) = \frac{a_0}{2} + \sum_{n=1}^{\infty} \left[ a_n \cos(n\omega t) + b_n \sin(n\omega t) \right]
$$

这里 $a_0$、$a_n$ 和 $b_n$ 是傅里叶系数，$\omega$ 是角频率。其中：

$$
\begin{aligned}
a_n &= \frac{2}{T} \int_{t_0}^{t_0 + T} f(t) \cos(n\omega t) \, dt \\
b_n &= \frac{2}{T} \int_{t_0}^{t_0 + T} f(t) \sin(n\omega t) \, dt
\end{aligned}
$$

---

根据欧拉公式：

$$
e^{j\theta} = \cos(\theta) + j\sin(\theta)
$$

我们可以把傅里叶级数写成**复数形式**：

$$
f(t) = \sum_{n=-\infty}^{\infty} F_n e^{jn\omega t}
$$

其中：

$$
F_n = \frac{1}{T} \int_{t_0}^{t_0 + T} f(t) e^{-jn\omega t} \, dt
$$

这里，复指数函数集 $\{e^{jn\omega t} \mid n = 0, \pm 1, \pm 2, \cdots\}$ 是一个完备正交函数集。我们就不加证明了。
