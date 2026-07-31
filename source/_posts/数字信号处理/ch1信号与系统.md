---
title: DSP 1 Signals and Systems
date: 2025-06-26 21:03:24
tags:
- note
categories:
- Digital Signal Processing
---

## 信号

### 信号的定义

信号：可检测的载有信息且随时间变化的物理量或物理现象（例如电压、电流、磁场强度等）。

### 信号的表示

信号可表示：一个或多个自变量的函数。

- 有一个自变量的信号称一维信号，如语音。

- 有多个自变量的信号称多维信号，如图像。

### 信号的分类

信号可分为如下四类：

- 模拟信号（analog signal）：连续时间、连续幅度的信号。

- 数字信号（digital signal）：离散时间、离散幅度的信号。

- 连续时间信号（continuous-time signal）：时域上的自变量是连续的。

- 离散时间信号（discrete-time signal）：时域上的自变量是离散的。

| 信号类型 | 时间 | 幅度 |
| :---: | :---: | :---: |
| 模拟信号 | 连续 | 连续 |
| 数字信号 | 离散 | 离散 |
| 连续时间信号 | 连续 | 离散 |
| 离散时间信号 | 离散 | 连续 |

另一种分类方法：

- 确定信号：能够用确定的时间函数表示。

- 随机信号：不能用确定的时间函数表示。

例如我们每一次讲话，用手机麦克风录音，即使讲话内容一致，录音后你看到的信号波形总会存在差异，任何时刻的信号幅度都是不可能事先确定的，其取值是随机的，当然要注意，这种随机性是存在统计规律的。

---

### 连续时间信号

连续时间信号在任何时刻除了有限个不连续点外都有确定的函数值。

接下来，我们介绍几种常见的连续时间信号。

![连续时间信号](https://ref.xht03.online/202503241535286.png)

### 单位阶跃信号

单位阶跃信号（unit step signal）$u(t)$ ：

$$
u(t) = \left\{
\begin{aligned}
0, & \quad t < 0 \\
1, & \quad t \geq 0
\end{aligned}
\right.
$$

![](https://ref.xht03.online/202503241540148.png)

单位阶跃信号不是一个连续的函数，我们可以将阶跃函数 $u(t)$ 视为如下的函数的极限形式：

$$
u_{\Delta}(t) = \left\{
\begin{aligned}
0, & \quad t < 0 \\
\frac{t}{\Delta}, & \quad 0 \leq t < \Delta \\
1, & \quad t \geq \Delta
\end{aligned}
\right.
$$

显然当 $\Delta \to 0$ 时，$u_{\Delta}(t)$ 逐渐趋近于 $u(t)$ 。

![](https://ref.xht03.online/202503241544266.png)

我们定义：

- $u_{\Delta}(t)$ 的导数为：$\delta_{\Delta}(t) = \frac{du_{\Delta}(t)}{dt}$

- $u(t)$ 的导数为：$\delta(t) = \frac{du(t)}{dt}$

则我们有：

$$ 
\delta(t) = \delta_{\Delta}(t) \quad \text{When} \quad \Delta \to 0
$$

这里的 $\delta(t)$ 称为**单位冲激函数**（unit impulse function）。

![](https://ref.xht03.online/202503241603440.png)

### 单位冲激信号

单位冲激信号（unit impulse signal）$\delta(t)$ ：

$$
\delta(t) = \frac{du(t)}{dt} \quad \quad u(t) = \int_{-\infty}^{t} \delta(\tau) d\tau
$$

冲击函数具有如下性质：

1. 抽样性：

    $$f(t) \delta(t) = f(0) \delta(t)$$

    $$\int_{-\infty}^{\infty} f(t) \delta(t) dt = f(0)$$

2. 奇偶性：

    $$\delta(t) = \delta(-t)$$

3. 比例性：

    $$\delta(at) = \frac{1}{|a|} \delta(t)$$

4. 卷积性质：

    $$f(t)*\delta(t) = f(t)$$

---

### 离散时间信号

离散时间信号：通常按时间顺序的一组数值，也称时间序列，简称**序列（sequence）**。

特别的，一维离散时间信号（序列）是：在一系列**不连续的时刻** k 有函数值，在其它时刻并无定义。

习惯上，对于时间离散的函数，自变量用 $k$ 表示，而不是 $t$ 。

接下来，我们介绍几种常见的离散时间信号。

![离散时间信号](https://ref.xht03.online/202503241828132.png)

### 单位阶跃序列

单位阶跃序列（unit step sequence）$u(n)$ ：

$$
u(n) = \left\{
\begin{aligned}
0, & \quad n < 0 \\
1, & \quad n \geq 0
\end{aligned}
\right.
$$

![](https://ref.xht03.online/202503251014221.png)

### 单位采样序列

单位采样序列（unit impulse sequence）$\delta(n)$ ：

$$
\delta(n) = \left\{
\begin{aligned}
0, & \quad n \neq 0 \\
1, & \quad n = 0
\end{aligned}
\right.
$$

注意到，单位冲激信号**时间离散**后就是单位采样序列了。

单位采样序列可以用来表示任意序列：

$$
x(n) = \sum_{k=-\infty}^{\infty} x(k) \delta(n-k)
$$

![](https://ref.xht03.online/202503251018274.png)

连续时间下，单位阶跃信号和单位冲激信号是**求导**关系；同样的，单位阶跃序列和单位采样序列也是**离散情况下**的求导关系，也就是**差分**的关系。

$$
\delta(n) = u(n) - u(n-1)
$$

$$
u(n) = \sum_{k=0}^{\infty} \delta(n-k)
$$

这两个式子直观上也是很容易验证的。

### 矩形序列

矩形信号（rectangular signal）$R_N(n)$ ：

$$
R_N(n) = \left\{
\begin{aligned}
1, & \quad 0 \leq n \leq N-1 \\
0, & \quad \text{其他}
\end{aligned}
\right.
$$

矩形信号也可以用单位阶跃序列或单位采样序列表示：

$$
R_N(n) = u(n) - u(n-N) = \sum_{k=0}^{N-1} \delta(n-k)
$$

![](https://ref.xht03.online/202503251029180.png)

---

### 周期信号

**周期信号**是指经过一定时间重复出现的信号。

$$
f(t) = f(t + kT) \quad k = 0, \pm 1, \pm 2, \cdots
$$

**非周期信号**在时间上不具有周而复始的特性，但我们可以将其看作是周期信号的特例，**周期趋于无穷大**。

对于离散时间的序列，我们也可同样地定义**周期序列**：

如果对所有整数 $n$ ，存在最小的正整数 $N$ 使得：

$$
x(n) = x(n + N) \quad -\infty < n <\infty
$$

则称序列 $x(n)$ 为周期序列，$N$ 为序列的周期。

---

### 信号的能量和功率

对于连续时间信号 $f(t)$ ：

- 能量：

    $$
    E = \lim_{T \to \infty}\int_{-T}^{T} f(t)^2 dt
    $$
- 功率：
    
    $$
    P = \lim_{T \to \infty}\frac{1}{2T}\int_{-T}^{T} f(t)^2 dt
    $$

对于离散时间信号 $f(n)$ ：

- 能量：

    $$
    E = \sum_{n=-\infty}^{\infty} f(n)^2
    $$

- 功率：
    
    $$
    P = \lim_{N \to \infty}\frac{1}{2N}\sum_{n=-N}^{N} f(n)^2
    $$

### 能量信号和功率信号

- 能量信号：信号总能量为有限值，信号平均功率为零。

- 功率信号：信号总能量为无穷大，信号平均功率为有限值。

一个信号不可能既是功率信号，又是能量信号，但可以既非功率信号，又非能量信号。周期信号都是功率信号，但是非周期信号没有定论。

---

### 信号的波形变换

> 例 1
>
> 已知 $f(5-2t)$ 波形如图所示，求 $f(t)$ 的波形。
> 
> ![](https://ref.xht03.online/202503251112069.png)
>
> 解：
>
>

---

## 系统

系统是一个由若干互有关联的单元组成的有机整体，具有某种功能。

系统还可看作一个转换（或一种运算）：$$r（t）＝T[e(t)]$$

![](https://ref.xht03.online/202503251114200.png)

### 系统的分类

按输入输出信号的时间性质分类：

- 连续时间系统：输入、输出都是**连续时间信号**，其数学模型是微分方程。

- 离散时间系统：输入、输出都是**离散时间信号**，其数学模型是差分方程。

- 混合系统：以上两种系统组合运用。

### 系统的性质

按输入输出信号的**性质**分类：

- **线性系统**：满足齐次性和叠加性。

    $$
    T[ae_1(t)+be_2(t)] = aT[e_1(t)] + bT[e_2(t)]
    $$

- **时不变系统**：若构成系统的元件参数不随时间而变化，则称此系统为时不变系统。

    $$
    r(t) = T[e(t)] \Rightarrow r(t-t_0) = T[e(t-t_0)]
    $$

- 因果系统：系统在 $t_0$ 时刻的输出只取决于 $t \leq t_0$ 时刻的输入，而与 $t > t_0$ 时刻的输入无关。

- 稳定系统：如果系统对任意有界输入都只产生有界输出。

    $$|e(t)| < \infty \Rightarrow |r(t)| < \infty$$

- 无记忆系统：在任意时刻 $n=n_0$ 的输出仅取决于 $n_0$ 时刻的输入。

线性和时不变性是两个互不相关的概念，系统可以既是线性的又是时不变的，这种系统称为**线性时不变系统**。线性时不变系统具有良好的性质，也是信号处理中主要研究的系统。（现实中，严格的时不变系统是几乎不存在的，元件老化必然导致系统参数的变化，但是在一定时间范围内，我们可以近似认为系统是时不变的。）

此外，一般而言，任何物理可实现系统都具有因果性。

---

## 卷积

### 卷积的含义

对于一般信号 $x(t)$ ，可以将 $x(t)$ 看作是一个个冲激函数的叠加（冲激函数用于抽样信号在某一时候值）。

我们 $x(t)$ 将其分成很多个 $\Delta$ 宽度的区间，用一个阶梯信号 $x_{\Delta}(t)$ 来近似表示。当 $\Delta \to 0$ 时，$x_{\Delta}(t)$ 就变成了一个时间上连续的信号 $x(t)$ 。

引入 $\delta_{\Delta}(t)$ 函数：

$$
\delta_{\Delta}(t) = \left\{
\begin{aligned}
0, & \quad \text{otherwise} \\
\frac{1}{\Delta}, & \quad 0 < t < \Delta \\
\end{aligned}
\right.
$$

则有：

$$
\Delta \delta_{\Delta}(t) = \left\{
\begin{aligned}
0, & \quad \text{otherwise} \\
1, & \quad 0 < t < \Delta \\
\end{aligned}
\right.
$$

则图中示例的第 k 个矩形信号可以表示为：$x(k\Delta) \delta_{\Delta}(t-k\Delta)\Delta$ 。将这些矩形叠加起来，就可以得到一个近似的阶梯形信号 $x_{\Delta}(t)$ ：

$$
x_{\Delta}(t) = \sum_{k=-\infty}^{\infty} x(k\Delta) \delta_{\Delta}(t-k\Delta)\Delta
$$

当 $\Delta \to 0$ 时：$k\Delta \to \tau$ ，$\Delta \to d_{\tau}$ ，$\Sigma \to \int$ 。

因而：

$$
x(t) = \int_{-\infty}^{\infty} x(\tau) \delta(t-\tau)d\tau
$$

这也是卷积的定义。可见，**任何连续时间信号都可以分解成单位冲激函数位移加权后的线性组合**。

![](https://ref.xht03.online/202503310956412.png)

---

### 冲激响应

对于一个系统的输入信号，考虑到：任何连续时间信号都可以分解成单位冲激函数的线性组合，所以我们只要知道系统对单位冲激函数的响应，就可以得到系统对任意输入信号的响应。

所以我们引入系统的**单位冲激响应** $h(t)$ ，它是系统对单位冲激函数 $\delta(t)$ 的响应。

当输入信号为 $x(t) = \delta(t)$ 时，输出信号为 $y(t) = h(t)$ 。

因此，

$$
y(t) = \int_{-\infty}^{\infty} x(\tau) h(t-\tau) d\tau = x(t)*h(t)
$$

---

### 卷积的定义

信号处理中，卷积的严谨定义如下。

设 $f(t)$ 和 $g(t)$ 是两个连续时间信号，则它们的卷积定义为：

$$
(f*g)(t) = \int_{-\infty}^{\infty} f(\tau) g(t-\tau) d\tau
$$

其中 $f(t)$ 和 $g(t)$ 称为卷积的两个因子，$t$ 是卷积的自变量。

---

### 常见卷积的计算

1. $u(t)*u(t) = tu(t)$

    证明：

    $$
    (u*u)(t) = \int_{-\infty}^{\infty} u(\tau) u(t-\tau) d\tau
    = \int_{0}^{t} u(t-\tau) d\tau
    = \int_{0}^{t} 1 d\tau = t
    $$

2. $e^{-at}u(t)*e^{-bt}u(t) = \frac{1}{a+b} (e^{-at}-e^{-bt}) u(t), \quad a\neq b$

    证明：

    $$
    (e^{-at}u*e^{-bt}u)(t) = \int_{0}^{t} e^{-a\tau} e^{-(t-\tau)b} d\tau
    = e^{-bt}\int_{0}^{t} e^{-(a-b)\tau} d\tau
    = \frac{1}{b-a}(e^{-at}-e^{-bt})u(t)
    $$

3. $f(t) * \delta(t) = f(t)$ ，其中 $\delta(t)$ 是单位冲激函数，$f(t)$ 是任意连续时间信号。

    证明：

    $$
    (f*\delta)(t) = \int_{-\infty}^{\infty} f(\tau) \delta(t-\tau) d\tau
    = f(t)
    $$

---

### 卷积的性质

1. 交换律

    $$
    (f_1*f_2)(t) = (f_2*f_1)(t)
    $$

2. 结合律

    $$
    (f_1*f_2)*f_3 = f_1*(f_2*f_3)
    $$

3. 分配律

    $$
    f_1*(f_2+f_3) = f_1*f_2 + f_1*f_3
    $$

证明：

- 交换律是显然的。

- 结合律证明：

    $$
    \begin{aligned}
    (f_1 * f_2) * f_3 &= \int_{-\infty}^{\infty} f_1(\tau) f_2(t - \tau) \, d\tau * f_3(t) \\
    &= \int_{-\infty}^{\infty} \left( \int_{-\infty}^{\infty} f_1(\tau) f_2(\sigma - \tau) \, d\tau \right) f_3(t - \sigma) \, d\sigma \\
    &= \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} f_1(\tau)  f_2(\sigma - \tau) f_3(t - \sigma) \,  d\tau d\sigma \\
    &= \int_{-\infty}^{\infty} f_1(\tau) \left( \int_{-\infty}^{\infty} f_2(\sigma) f_3(t - \sigma - \tau) \, d\sigma \right) d\tau\\
    &= f_1 * (f_2 * f_3)(t)
    \end{aligned}
    $$

- 分配律证明：

    $$
    \begin{aligned}
    f_1 * (f_2 + f_3) &= \int_{-\infty}^{\infty} f_1(\tau) (f_2 + f_3)(t - \tau) \, d\tau \\
    &= \int_{-\infty}^{\infty} f_1(\tau) f_2(t - \tau) \, d\tau + \int_{-\infty}^{\infty} f_1(\tau) f_3(t - \tau) \, d\tau \\
    &= f_1 * f_2 + f_1 * f_3
    \end{aligned}
    $$

---

4. 卷积的时移性质

    设 $y(t) = f_1(t)*f_2(t)$ ，则：

    $$
    y(t) = f_1(t-t_0)*f_2(t+t_0)
    $$

    $$
    y(t-t_0) = f_1(t)*f_2(t-t_0) = f_1(t-t_0)*f_2(t)
    $$

---

5. 卷积的微分性质：两个函数卷积后的导数等于其中一函数之导数与另一函数之卷积。

    $$
    \frac{dy(t)}{dt} = \frac{d}{dt}[f_1(t)*f_2(t)] = \frac{df_1(t)}{dt} * f_2(t) = f_1(t) * \frac{df_2(t)}{dt}
    $$

6. 卷积的积分性质：两个函数卷积后的积分等于其中一函数之积分与另一函数之卷积。

    $$
    \int_{-\infty}^{t} y(\tau) d\tau = \int_{-\infty}^{t} [f_1(\tau)*f_2(\tau)] d\tau = f_1(\tau) * \int_{-\infty}^{t} f_2(\tau) d{\tau}
    $$

7. 由性质 5 和 6 可得：

    $$
    y(t) = f_1(t) * f_2(t) = \frac{df_1(t)}{dt} * \int_{-\infty}^{t} f_2(\tau) d\tau
    $$
    
证明：

- 微分性质证明

    $$
    \begin{aligned}
    \frac{dy(t)}{dt} &= \frac{d}{dt} \int_{-\infty}^{\infty} f_1(\tau) f_2(t - \tau) d\tau \\
    &= \int_{-\infty}^{\infty} f_1(\tau) \frac{d}{dt} f_2(t - \tau) d\tau \\
    &= \int_{-\infty}^{\infty} f_1(\tau) f_2'(t - \tau) d\tau
    \end{aligned}
    $$

- 积分性质证明

    $$
    \begin{aligned}
    \int_{-\infty}^{t} y(\tau) d\tau &= \int_{-\infty}^{t} \int_{-\infty}^{\infty} f_1(\sigma) f_2(\tau - \sigma) d\sigma d\tau \\
    &= \int_{-\infty}^{\infty} f_1(\sigma) \left( \int_{-\infty}^{t} f_2(\tau - \sigma) d\tau \right) d\sigma \\
    &= \int_{-\infty}^{\infty} f_1(\sigma) \left( \int_{-\infty}^{t - \sigma} f_2(\tau) d\tau \right) d\sigma \\
    &= \int_{-\infty}^{\infty} f_1(\sigma) F_2(t-\sigma) d\sigma \\
    &= f_1(t) * F_2(t)
    \end{aligned}
    $$

    其中 $F_2(t) = \int_{-\infty}^{t} f_2(\tau) d\tau$ 。

---

### 离散时间信号的卷积

以上对卷积的讨论都是在连续时间信号下进行的。如果不加说明，我们认为时间变量记为 $t$ 的信号是连续时间信号，记为 $n$ 的信号是离散时间信号。

离散时间信号的卷积定义为：

$$
x(n) * h(n) = \sum_{k=-\infty}^{\infty} x(k) h(n-k)
$$