---
title: DSP 3 Fourier Transform
date: 2025-06-29 11:27:30
tags:
- note
categories:
- Digital Signal Processing
---

## 傅里叶变换

傅里叶变换是一个重要的数学工具，它可以把一个**非周期函数**表示为一组正弦和余弦函数的**连续积分**。

对于一个周期信号 $f(t)$ ，设其周期为 $T$ ，则其角频率为 $\omega = \frac{2\pi}{T}$ ，此时 $f(t)$ 包含的频率分量为 $\omega$ 的整数倍，在频谱上表现为一个个离散的谱线。当 $T \to \infty$ 时，$\omega \to 0$ ，频率分量变为连续的，频谱上表现为一个个连续的谱线。同时，当 $T \to \infty$ 时，$f(t)$ 变为一个**非周期信号**。

$f(t)$ 的谱系数（即各个频率分量的系数）为：

$$
F(n\omega) = \frac{1}{T} \int_{t_0}^{t_0 + T} f(t) e^{-jn\omega t} \, dt 
$$

显然，当 $T \to \infty$ 时，$F(n\omega)$ 趋于 0 。所以，如果我们对非周期函数 $f(t)$ 进行傅里叶变换，能得到连续谱，但其幅度无限小。虽然各个频率分量的幅度无限小，但它们之间仍存在相对大小。所以我们引入**频谱密度函数**。

我们考虑：

$$
T F(n\omega) = \int_{t_0}^{t_0 + T} f(t) e^{-jn\omega t} \, dt
$$

而且，

$$
T F(n\omega) = \frac{F(n\omega)}{\frac{1}{T}} = \frac{F(n\omega)}{f}
$$

由此可见，$T F(n\omega)$ 表征着单位频带上的频谱值。因此我们可以把 $T F(n\omega)$ 看作是一个**频谱密度函数**，并记 $F(jn\omega) = T F(n\omega)$。

由傅里叶级数的定义，我们可以得到：

$$
\begin{aligned}
f(t) &= \sum_{n=-\infty}^{\infty} F(n\omega) e^{jn\omega t}\\
&= \sum_{n=-\infty}^{\infty} \frac{F(jn\omega)}{T} e^{jn\omega t}\\
\end{aligned}
$$

当 $T \to \infty$ 时，$\Delta(n\omega) = \omega \to d\omega$ ，$(n\omega) \to \omega$ ，所以：

$$
\begin{aligned}
\lim_{T \to \infty} f(t) &= \lim_{T \to \infty} \sum_{n=-\infty}^{\infty} F(jn\omega) \frac{\omega}{2\pi} e^{jn\omega t}\\
&= \frac{1}{2\pi} \int_{-\infty}^{\infty} F(j\omega) e^{j\omega t} d\omega\\
\end{aligned}
$$

至此，我们就推导出了**傅里叶变换**的公式：

$$
F(jw) = \int_{-\infty}^{\infty} f(t) e^{-j\omega t} dt
$$

**傅里叶逆变换**的公式为：

$$
f(t) = \frac{1}{2\pi} \int_{-\infty}^{\infty} F(j\omega) e^{j\omega t} d\omega
$$

$F(j\omega)$ 有时也简写为 $F(\omega)$ 。

---

## 傅里叶变换的物理意义

由于 $f(t)$ 是实函数，所以 $F(\omega)$ 满足共轭对称性 $F(-\omega)=\overline{F(\omega)}$，即 $|F(\omega)|$ 关于 $\omega$ 为偶函数、$\phi(\omega)$ 为奇函数。于是 $f(t)$ 的被积函数虚部关于 $\omega$ 为奇函数，在整个实轴上积分为零，因此可以略去。

$$
\begin{aligned}
f(t) &= \frac{1}{2\pi} \int_{-\infty}^{\infty} F(\omega) e^{j\omega t} d\omega\\
&= \frac{1}{2\pi} \int_{-\infty}^{\infty} |F(\omega)| cos(\omega t + \phi(\omega)) d\omega + j \frac{1}{2\pi} \int_{-\infty}^{\infty} |F(\omega)| sin(\omega t + \phi(\omega)) d\omega\\
&= \frac{1}{2\pi} \int_{-\infty}^{\infty} |F(\omega)| cos(\omega t + \phi(\omega)) d\omega
\end{aligned}
$$

所以我们得到：

![](https://ref.xht03.online/202504121030427.png)

这表明：一个非周期信号 $f(t)$ 可以看作是无穷多个振幅无穷小的连续余弦信号之和。

---

## 常见信号的傅里叶变换

### 矩形信号

$$
\begin{aligned}
F(\omega) &= \int_{-\frac{\tau}{2}}^{\frac{\tau}{2}} E e^{-j\omega t} dt\\
&= \frac{E}{-j\omega} e^{-j\omega t} \bigg|_{-\frac{\tau}{2}}^{\frac{\tau}{2}}\\
&= \frac{E}{-j\omega} \left( e^{j\frac{-\omega\tau}{2}} - e^{j\frac{\omega\tau}{2}} \right)\\
&= \frac{E}{j\omega} \cdot 2j \sin\left( \frac{\omega\tau}{2} \right)\\
&= \frac{2E}{\omega} \sin\left( \frac{\omega\tau}{2} \right)\\
&= E\tau \cdot \frac{\sin\left( \frac{\omega\tau}{2} \right)}{\frac{\omega\tau}{2}}\\
&= E\tau \cdot Sa(\frac{\omega\tau}{2})
\end{aligned}
$$

其中 $Sa(x) = \frac{\sin(x)}{x}$ 。

![](https://ref.xht03.online/202504121050823.png)

---

### 单位冲激信号

$$
F(\omega) = \int_{-\infty}^{\infty} \delta(t) e^{-j\omega t} dt = 1
$$

实际上，$\delta(t)$ 可以看作 $\tau \times \frac{1}{\tau}$ 的矩形脉冲，且 $\tau \to 0$ 。

![](https://ref.xht03.online/202504121036688.png)

---

### 直流信号

直流信号可以看作是一个矩形脉冲，只不过 $\tau \to \infty$ 。

$$
\begin{aligned}
F(\omega) &= \int_{-\infty}^{\infty} E \delta(t) e^{-j\omega t} dt\\
&= \lim_{\tau \to \infty} \int_{-\tau}^{tau} E e^{-j\omega t} dt\\
&= E \lim_{\tau \to \infty} \frac{2sin(\omega\tau)}{\omega}\\
&= 2 \pi E \delta(\omega)
\end{aligned}
$$

这里我们用到了 $\lim_{\tau \to \infty} \frac{\sin(\omega \tau)}{\pi \omega} = \delta(\omega)$ ，这里我们就不加证明的使用了。

![](https://ref.xht03.online/202504121107115.png)

---

## 傅里叶变换的性质

当 $f(t)$ 为**实函数**时：

$$
\begin{aligned}
F(\omega) &= \int_{-\infty}^{\infty} f(t) e^{-j\omega t} dt\\
&= \int_{-\infty}^{\infty} f(t) \left( \cos(\omega t) - j\sin(\omega t) \right) dt\\
&= R(\omega) - jI(\omega)\\
\end{aligned}
$$

我们记：$F(\omega) = |F(\omega)| e^{j\phi(\omega)}$ 。

那么不难得到：

- $|F(\omega)|$ 是关于 $\omega$ 的偶函数。

- $\phi(\omega) = -arctan\frac{I(\omega)}{R(\omega)}$ 是关于 $\omega$ 的奇函数。

除此以外，傅里叶变换还具有以下基本性质：

- 线性性质

    $$
    a_1f_1(t) + a_2f_2(t) \leftrightarrow a_1F_1(\omega) + a_2F_2(\omega)
    $$

- 时移性质

    若 $f(t) \leftrightarrow F(\omega)$ ，则：$f(t - t_0) \leftrightarrow e^{-j\omega t_0}F(\omega)$

- 频移性质

    若 $f(t) \leftrightarrow F(\omega)$ ，则：$f(t)e^{j\omega_0 t} \leftrightarrow F(\omega - \omega_0)$

- 频域缩放性质

    若 $f(t) \leftrightarrow F(\omega)$ ，则：$f(at) \leftrightarrow \frac{1}{|a|}F\left( \frac{\omega}{a} \right)$

- **卷积定理**

    若 $f_1(t) \leftrightarrow F_1(\omega)$ ，$f_2(t) \leftrightarrow F_2(\omega)$ ，则：

    $$
    f_1(t) * f_2(t) \leftrightarrow F_1(\omega) \cdot F_2(\omega)
    $$

    $$
    f_1(t) \cdot f_2(t) \leftrightarrow \frac{1}{2\pi}F_1(\omega) * F_2(\omega) 
    $$

---

## 离散时间傅里叶变换

上述考虑的是连续时间下的傅里叶变换。我们现在考虑**离散时间**下的傅里叶变换。

对于**非周期信号** $x(n)$ 的离散时间傅里叶变换（DTFT）为：

$$
X(e^{j\omega}) = \sum_{n=-\infty}^{\infty} x(n) e^{-j\omega n}
$$

这里 $X(e^{j\omega})$ 是一个复数函数，称为**频谱**。它是一个周期为 $2\pi$ 的周期函数。

离散时间傅里叶变换的**逆变换**为：

$$
x(n) = \frac{1}{2\pi} \int_{-\pi}^{\pi} X(e^{j\omega}) e^{j\omega n} d\omega
$$

---

## DTFT 性质

离散时间傅里叶变换具有以下性质（与连续时间傅里叶变换类似）：

- 线性性质

$$
a_1x_1(n) + a_2x_2(n) \leftrightarrow a_1X_1(e^{j\omega}) + a_2X_2(e^{j\omega})
$$

- 时移性质

    若 $x(n) \leftrightarrow X(e^{j\omega})$ ，则：$x(n - n_0) \leftrightarrow e^{-j\omega n_0}X(e^{j\omega})$

- 频移性质

    若 $x(n) \leftrightarrow X(e^{j\omega})$ ，则：$x(n)e^{j\omega_0 n} \leftrightarrow X(e^{j(\omega - \omega_0)})$

- 卷积性质

    若 $x_1(n) \leftrightarrow X_1(e^{j\omega})$ ，$x_2(n) \leftrightarrow X_2(e^{j\omega})$ ，则：

$$
x_1(n) * x_2(n) \leftrightarrow X_1(e^{j\omega}) \cdot X_2(e^{j\omega})
$$

$$
x_1(n) \cdot x_2(n) \leftrightarrow \frac{1}{2\pi}X_1(e^{j\omega}) * X_2(e^{j\omega})
$$

---

## 离散傅里叶变换

当我们使用计算机进行数字信号处理时：

- 计算机只能处理有限个数据点。

- 计算机只能处理有限个频率分量。

所以，在实际应用中，我们一定是对**离散时间**，且**有限长度**的信号进行傅里叶变换。我们称之为**离散傅里叶变换**（DFT）。

需要先指出的是，离散傅里叶变换（DFT）是对离散时间傅里叶变换（DTFT）的一个近似。DFT 是对 DTFT 的一个有限采样。

DTFT 是一个连续的函数，而无限多个频率分量无法被计算机存储。而 DFT 是一个离散的函数。

---

DFT 的定义为：

1. 将有限长序列 $x(n)$ ，$n = 0, 1, \cdots, N - 1$ ，拓展成**周期函数** $\tilde{x}(n)$ 。

$$
\tilde{x}(n) = \sum_{r=-\infty}^{\infty} x(n+rN) = x((n))_{N}\ , \quad ((n))_{N} = n \mod N
$$

2. 计算 DFT：

$$
\begin{aligned}
X[k] &= \sum_{n=0}^{N-1} x(n) e^{-j\frac{2\pi}{N}nk} \quad (k = 0, 1, \cdots, N - 1)\\
&= \sum_{n=0}^{N-1} x(n) W_N^{nk} \quad (W_N = e^{-j\frac{2\pi}{N}}\ \text{为N阶单位根})
\end{aligned}
$$

3. 计算 IDFT：

$$
\begin{aligned}
x(n) &= \frac{1}{N} \sum_{k=0}^{N-1} X[k] e^{j\frac{2\pi}{N}nk} \quad (n = 0, 1, \cdots, N - 1)\\
&= \frac{1}{N} \sum_{k=0}^{N-1} X[k] W_N^{-nk}
\end{aligned}
$$

---

## DFT 性质

DFT 具有以下性质：

1. 线性

$$
ax_1(n) + bx_2(n) \leftrightarrow aX_1[k] + bX_2[k]
$$

2. 对称性

设 $x(n)$ 是长度为 $N$ 的是序列，则：

$$
X[k] = X^*[N - k] \quad (k = 0, 1, \cdots, N - 1)
$$

其中 $X^*$ 是 $X$ 的共轭。

3. 圆周移位

$$
x((n - n_0))_N \leftrightarrow X[k] W_N^{-kn_0} \quad (n_0 = 0, 1, \cdots, N - 1)
$$

![](https://ref.xht03.online/202504131104011.png)

4. 圆周卷积

设 $x(n)$ 和 $h(n)$ 是长度为 $N$ 的序列，则：

$$
y(n) = x(n) * h(n) = \left[\sum_{m=0}^{N-1} \tilde{h}(m) \tilde{x}(n - m)\right] R_N(n)
$$

其中 $R_N(n)$ 是长度为 $N$ 的矩形波。

---

## 总结

在此，我们对各种傅里叶级数、变换做个总结。

- 傅里叶级数用于处理**周期信号**，可以分为

  - 连续时间傅里叶级数（CTFS）

  - 离散时间傅里叶级数（DTFS）

- 傅里叶变换用于处理**非周期信号**，可以分为

    - 连续时间傅里叶变换（CTFT）
    
    - 离散时间傅里叶变换（DTFT）

那么，聪明如你可能会问：离散傅里叶变换（DFT）是什么呢？

离散傅里叶变换（DFT）在延拓后是一个周期信号，所以 **DFT 本质上是 DFS** 。

由于：周期函数分解的正弦波线性组合，各个频率分量都是 $\omega$ 的整数倍，所以：

- 傅里叶级数的结果是**离散的**。（或者说，周期函数的频谱是离散的）

- 傅里叶变换的结果是**连续的**。（或者说，非周期函数的频谱是连续的）


---

## 频率响应

先看一个最简单的输入，复指数 $x(n)=e^{j\omega n}$，我们试问：这个复指数经过 LTI 系统后，输出如何？把 $x(n)=e^{j\omega n}$ 代入 $y=x*h$

$$
\begin{aligned}
y(n)
&=\sum_{k=-\infty}^{+\infty}h(k)\,x(n-k)\\
&=\sum_{k=-\infty}^{+\infty}h(k)\,e^{j\omega(n-k)}\\
&=e^{j\omega n}\sum_{k=-\infty}^{+\infty}h(k)\,e^{-j\omega k}
\end{aligned}
$$

注意到最后一行：

- 前面那个因子 $e^{j\omega n}$ 就是输入 $x(n)$

- 后面那串 $\sum_{k=-\infty}^{+\infty}h(k)\,e^{-j\omega k}$ **和 $n$ 一点关系都没有**，它只取决于频率 $\omega$。

我们记

$$
H(e^{j\omega}) = \sum_{k=-\infty}^{+\infty}h(k)\,e^{-j\omega k}
$$

那么

$$
y(n)=H(e^{j\omega})\,e^{j\omega n}=H(e^{j\omega})\,x(n)
$$

**也就是：复指数输入，输出仍然是同一个复指数，频率 $\omega$ 不变，只是被乘上了一个复数 $H(e^{j\omega})$**。对于一个给定的频率 $\omega$，$H(e^{j\omega})$ 是常数。换言之，**复指数经过 LTI 系统，频率不变，只是幅值和相位变化了。**

---

那推广到任意信号 $x(n)$，经过 LTI 系统后，输出如何呢？根据 DTFT 的逆变换，任意信号 $x(n)$ 可以写成复指数的"连续加权和"：

$$
x(n)=\frac{1}{2\pi}\int_{-\pi}^{\pi}X(e^{j\omega})\,e^{j\omega n}\,d\omega
$$

这个式子的含义是：**$x(n)$ 由无穷多个不同频率 $\omega$ 的复指数 $e^{j\omega n}$ 叠加而成**。而 

$$
\frac{1}{2\pi}X(e^{j\omega})d\omega$$

就是频率 $\omega$ 那一个分量的权重。因为系统是线性的，总输出就是把所有这些分量（对所有 $\omega$）的输出叠加起来。上面复指数的例子告诉我们，复指数 $e^{j\omega n}$ 经系统只是乘 $H(e^{j\omega})$，则其输出为

$$
\left[\frac{1}{2\pi}X(e^{j\omega})\,d\omega\;\right]H(e^{j\omega})e^{j\omega n}
$$

我们把各个频率分量 $\omega$ 叠加起来，也即是求积分：

$$
y(n)=\frac{1}{2\pi}\int_{-\pi}^{\pi}X(e^{j\omega})H(e^{j\omega})\,e^{j\omega n}\,d\omega
$$

这个式子的结构正好就是 DTFT 的逆变换 $\frac{1}{2\pi}\int_{-\pi}^{\pi}(\cdot)\,e^{j\omega n}d\omega$。也就是说，**$X(e^{j\omega})H(e^{j\omega})$ 正是 $y(n)$ 的频谱**：

$$
Y(e^{j\omega})=X(e^{j\omega})H(e^{j\omega})
$$

这就是卷积定理。

> **一个时域的无穷卷积 $y=x*h$，在频域只是一次乘法**。

**为什么频域更省事**？时域要算逐项相乘再求和的无穷卷积；频域只要——把 $x(n)$、$h(n)$ 分别做 DTFT 得 $X$、$H$，两者相乘得 $Y$，再对 $Y$ 做一次逆变换，就还原出 $y(n)$。三步都是熟面孔，运算也简单。

---

卷积定理还有一种更代数的证明：直接从 $y(n)$ 的频谱定义出发，代入卷积、交换求和。

$$
\begin{aligned}
Y(e^{j\omega})
&=\sum_{n=-\infty}^{+\infty}y(n)\,e^{-j\omega n}\\
&=\sum_{n=-\infty}^{+\infty}\left[\sum_{k=-\infty}^{+\infty}h(k)\,x(n-k)\right]e^{-j\omega n} \quad \text{let } m = n-k\\
&=\left[\sum_{k=-\infty}^{+\infty}h(k)e^{-j\omega k}\right]\left[\sum_{m=-\infty}^{+\infty}x(m)e^{-j\omega m}\right]\\
&=H(e^{j\omega})\,X(e^{j\omega})
\end{aligned}
$$

殊途同归，都是 $Y=XH$。

---

上面那个"和 $n$ 无关、只取决于 $\omega$"的常数 $H(e^{j\omega})$，因为它决定了系统对**每个频率分量**分别做多大的缩放和相移，我们给它一个名字：**频率响应**。

$$
H(e^{j\omega})=\sum_{k=-\infty}^{+\infty}h(k)\,e^{-j\omega k}
$$

注意右式，它就是冲激响应 $h(n)$ 的**离散时间傅里叶变换（DTFT）**。所以：

> **频率响应，就是冲激响应的频谱。** 
 
于是，时域和频域其实是**同一个 LTI 系统的两种等价描述**：

| 角度 | 用哪个量 | 回答的问题 |
|---|---|---|
| 时域 | 冲激响应 $h(n)$ | "给一个脉冲，系统怎么响应" |
| 频域 | 频率响应 $H(e^{j\omega})$ | "对各个频率分量，系统分别怎么缩放/相移" |

把两种角度串起来的，正是这座桥：

$$
H(e^{j\omega})=\mathrm{DTFT}\{h(n)\}
$$