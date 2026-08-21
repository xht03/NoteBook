---
title: DSP 5 Sampling
date: 2025-07-05 19:41:20
tags:
- note
categories:
- Digital Signal Processing
---

## 采样

我们以声音信号处理为例。当我们说话后，声音信号会在空气中传播，经过麦克风转换为电信号。这个电信号是一个连续的模拟信号。模拟信号在时间和幅度上都是连续的。计算机无法处理连续值，只能处理有限个离散值。因此，我们需要将模拟信号转换为数字信号，也就是**采样**。

当然，采样只能实现时间上的离散化，幅度上仍然是连续的。为了将幅度也离散化，我们还需要**量化**。但这都是后话了。

![](https://ref.xht03.online/202504131945032.png)

在某些离散时间点上提取连续时间信号值的过程称为**采样**。

## 采样的数学模型

在时域上，采样后的信号 $x_p(t)$ 是原信号 $x(t)$ 与采样函数 $p(t)$ 的乘积。

$$
x_p(t) = x(t) \cdot p(t)
$$

对其做傅里叶变换，得到频域上的采样信号 $X_p(j\omega)$ 。

$$
X_p(j\omega) = \frac{1}{2\pi} X(j\omega) * P(j\omega)
$$

其中 $P(j\omega)$ 是采样函数的傅里叶变换。

![](https://ref.xht03.online/202504131957260.png)

---

## 自然采样

在实际中，采样函数 $p(t)$ 通常是一个矩形脉冲函数。我们称这种采样方式为**自然采样**。

$$
p(t) = \sum_{n=-\infty}^{+\infty} R_{\tau}\left(\frac{t - nT}{\tau}\right)
$$

其中 $R_{\tau}(t)$ 是一个宽度为 $\tau$ 高度为 1 的矩形脉冲函数。

则采样后的信号为：

$$
\begin{aligned}
x_p(t) &= x(t) \cdot p(t)\\
&= x(t) \left( \frac{\tau}{T_s} \sum_{k=-\infty}^{\infty} Sa(\frac{k\omega_s\tau}{2}) e^{jk\omega_s t} \right)\\
&= \frac{\tau}{T_s} \sum_{k=-\infty}^{\infty} Sa(\frac{k\omega_s\tau}{2}) \cdot x(t) e^{jk\omega_s t}\\
\end{aligned}
$$

其中 $T_s$ 是采样周期，$\omega_s = \frac{2\pi}{T_s}$ 是采样频率。

由于：傅里叶变换具有线性和频移性质

$$
\begin{aligned}
X_p(j\omega) &= \mathcal{F}(x_p(t))\\
&= \mathcal{F}\left( \frac{\tau}{T_s} \sum_{k=-\infty}^{\infty} Sa(\frac{k\omega_s\tau}{2}) \cdot x(t) e^{jk\omega_s t} \right)\\
&= \frac{\tau}{T_s} \sum_{k=-\infty}^{\infty} Sa(\frac{k\omega_s\tau}{2}) \cdot \mathcal{F}(x(t) e^{jk\omega_s t})\\
&= \frac{\tau}{T_s} \sum_{k=-\infty}^{\infty} Sa(\frac{k\omega_s\tau}{2}) \cdot X(\omega - k\omega_s)\\
\end{aligned}
$$



显然，自然采样下的幅度谱与原信号的幅度谱不是完全一致的，由于 $Sa$ 函数的存在，频谱幅度衰减了。

---

## 理想采样

如果 $p(t)$ 能每个采样点都准确地取到 $x(t)$ 的值，那么 $p(t)$ 就是一个个冲激函数的叠加。

$$
p(t) = \sum_{n=-\infty}^{+\infty} \delta(t - nT)
$$

其中 $T$ 是采样周期。那么：

$$
x_p(t) = \sum_{n=-\infty}^{+\infty} x(nT) \delta(t - nT)
$$

类似的，我们不难得到 $x_p(t)$ 的频谱：

$$
X_p(j\omega) = \frac{1}{T} \sum_{k=-\infty}^{+\infty} X(\omega - k\omega_s)
$$

所以，**采样后的频谱是原信号频谱的周期延拓**。

![](https://ref.xht03.online/202504132051111.png)

---

## 时域采样定理

在理想采样下，采样后的频谱是原信号频谱的周期延拓。但是只要原信号的频谱延拓之后不重叠，那么我们就可以通过滤波器将采样后的频谱恢复为原信号的频谱。

由于实信号的频谱是对称的，所以我们设原信号的最大频率为 $\omega_M$ ，采样频率为 $\omega_s$ 。由上图，为了不重叠，显然需要 $\omega_s \geq 2\omega_M$ 。也就是**采样频率至少是原信号带宽的两倍**。

这就是著名的**采样定理**。

Nyquist 采样定理表述如下：

当 $\omega_M \leq \frac{\omega_s}{2}$ 时，实信号 $x(t)$ 频谱采样之后不会发生重叠，可以用理想低通滤波器取出该信号。

---

需要注意的是，采样定理并不是充要条件。也就是说，$\omega_s < 2\omega_M$ 时，也可能不会发生重叠。

比如，对于**实值带通信号** $x(t)$ ，假设：

- 带宽 $B = f_2 - f_1$

- 中心频率 $f_c = \frac{f_1 + f_2}{2}$

如果 $f_c > \frac{B}{2}$ ，且 $f_2$ 是带宽 $B$ 的整数倍，则当采样频率 $f_s = 2B$ 时，采样后的频谱不会发生混叠。

![](https://ref.xht03.online/202504211012569.png)

所以，采样定理的实质是：原信号在采样后，周期延拓之后，选取合适的采样频率，使得频谱不会发生重叠。

---

## 频域采样

连续时间信号的傅里叶变换（FT）或离散时间信号的DTFT（离散时间傅里叶变换）的频谱是连续的，但是计算机只能处理离散数据。因此，我们需要对频谱进行采样。频域采样的过程称为**频域采样**。

设连续信号 $f(t) \xrightarrow{FT} F(\omega)$ ，其频谱 $F(\omega)$ 是连续的。

若我们知道完整的频谱 $F(\omega)$ ，则毫无疑问地，我们可以复原出原信号 $F(\omega) \xrightarrow{IFT} f(t)$ 。

但是，计算机只能处理有限个离散值，因此我们已知的频谱一定是采样后的离散值 $ F_1(\omega) = F(\omega) \cdot \delta_{\omega_1}(\omega)$ 。

其中，

- $\omega_1$ 是采样频率

- 理想采样信号： 
  $$
  \delta_{\omega_1}(\omega) = \sum_{n=-\infty}^{+\infty} \delta(\omega - n\omega_1) \xrightarrow{IFT} \frac{1}{\omega_1} \delta_{T_1}(t)
  $$

  其中 $\delta_T(t)$ 是时域中以 $T_1 = \frac{2\pi}{\omega_1}$ 为周期的冲激串：

  $$
  \delta_{T_1}(t) = \sum_{k=-\infty}^{+\infty} \delta(t - kT_1)
  $$

因此，我们有：

$$
F_1(\omega) = F(\omega) \cdot \delta_{\omega_1}(\omega) \xrightarrow{IFT} f_1(t) = f(t) * \delta_T(t)
$$

那么，

$$
\begin{aligned}
f_1(t) &= f(t) * \frac{1}{\omega_1} \sum_{k=-\infty}^{+\infty} \delta(t - kT_1)\\
&= \frac{1}{\omega_1} \sum_{k=-\infty}^{+\infty} f(t - kT_1)
\end{aligned}
$$

其中 $T_1 = \frac{2\pi}{\omega_1}$ 是采样周期。

也就是：采样后的信号 $f_1(t)$ 是原信号 $f(t)$ 的周期延拓。

- 频域采样，时域上周期延拓。

- 时域采样，频域上周期延拓。

---

## 频域采样定理

我们假设 $f(t)$ 是一个时限信号。也就是：$|t| > T_0$ 时 $f(t)=0$ 。

与时域上的采样定理类似，其本质也是为了避免信号混叠。所以，

$$
2 T_0 \leq T_1 = \frac{2\pi}{\omega_1}
$$

也就是：

$$
\omega_1 \leq \frac{\pi}{T_0}
$$

这就是频域采样定理。

频域采样定理表明：频域采样间隔不能超过 $\pi/T_0$ 。