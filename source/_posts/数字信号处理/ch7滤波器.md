---
title: DSP 7 Digital Filters
date: 2025-07-08 11:05:37
tags:
- note
categories:
- Digital Signal Processing
---

## 数字滤波器

滤波（Filtering）的基本目的，是**改变信号中不同频率分量之间的相对比例**，从而保留我们需要的频率成分，并抑制或去除不需要的频率成分。




对于一个离散时间 LTI 系统，设输入为 $x(n)$，单位脉冲响应为 $h(n)$，输出为 $y(n)$，则：

$$
y(n)=x(n)*h(n)
$$

对两边进行 DTFT：

$$
Y(e^{j\omega})=X(e^{j\omega})H(e^{j\omega})
$$

因此，从频域来看，滤波器实际上就是利用 $H(e^{j\omega})$ 对输入信号的不同频率分量进行不同程度的改变：

$$
|Y(e^{j\omega})| = |X(e^{j\omega})||H(e^{j\omega})|
$$

而输出信号的相位为：

$$
\angle Y(e^{j\omega})
=
\angle X(e^{j\omega})
+
\angle H(e^{j\omega})
$$

所以，一个滤波器不仅会影响不同频率分量的**幅度**，还可能改变它们的**相位**。

---

根据允许哪些频率分量通过，可以将滤波器分为：

* **低通滤波器（Low-pass Filter）**：保留低频分量，抑制高频分量。
* **高通滤波器（High-pass Filter）**：保留高频分量，抑制低频分量。
* **带通滤波器（Band-pass Filter）**：只允许某一频带通过。
* **带阻滤波器（Band-stop Filter）**：抑制某一频带。
* **全通滤波器（All-pass Filter）**：所有频率分量的幅度均保持不变，只改变相位。

例如理想低通滤波器的幅频响应为：

$$
|H_d(e^{j\omega})|=
\begin{cases}
1,&|\omega|\leq\omega_c\\
0,&\omega_c<|\omega|\leq\pi
\end{cases}
$$

其中 $\omega_c$ 为截止频率。

然而，实际系统无法实现从 $1$ 突然跳变为 $0$ 的理想频率响应。因此，实际滤波器通常划分为三个区域：

* **通带（Passband）**：希望信号基本不受衰减地通过；
* **阻带（Stopband）**：希望信号受到充分抑制；
* **过渡带（Transition Band）**：频率响应从通带逐渐下降到阻带。

![](https://ref.xht03.online/202608211230025.png)

对于低通滤波器，通常用 $\omega_p$ 表示通带截止频率，用 $\omega_s$ 表示阻带截止频率，因此过渡带宽度为：

$$
B_t=\omega_s-\omega_p
$$

实际滤波器的通带也并不会严格等于 $1$，阻带也不会严格等于 $0$，而是允许一定程度的波动。若幅度已经归一化，则可以用 $\delta_1$ 和 $\delta_2$ 分别描述通带和阻带允许的逼近误差：

$$
1-\delta_1
\leq
|H(e^{j\omega})|
\leq
1+\delta_1,
\qquad |\omega|\leq\omega_p
$$

$$
|H(e^{j\omega})|
\leq
\delta_2,
\qquad
\omega_s\leq|\omega|\leq\pi
$$

$\delta_1$ 越小，通带越平坦；$\delta_2$ 越小，阻带抑制能力越强。工程中也经常用 dB 表示通带最大衰减 $\alpha_p$ 和阻带最小衰减 $\alpha_s$（只是换单位而已）。一般来说，$\alpha_p$ 越小，表示通带越平坦；$\alpha_s$ 越大，表示阻带抑制能力越强。

---

## FIR 与 IIR 滤波器

对于离散时间 LTI 系统，其输出由输入 $x(n)$ 与单位脉冲响应 $h(n)$ 的卷积得到：

$$
y(n)
=x(n)*h(n)
=\sum_{k=-\infty}^{+\infty}
h(k)x(n-k)
$$

根据单位脉冲响应 $h(n)$ 是否具有有限长度，可以将数字滤波器分为 FIR 和 IIR 两类。

---

### FIR 滤波器

FIR（Finite Impulse Response）称为**有限脉冲响应滤波器**。如果一个因果滤波器的单位脉冲响应只有有限个非零值，例如一个长度为 $N$ 的 FIR 滤波器：

$$
h(n)=0,
\quad
n<0\ \text{或}\ n>N-1
$$

那么原来的无限卷积中，实际上只有 $0\leq k\leq N-1$ 的项非零，因此可以直接写成：

$$
y(n)
=
\sum_{k=0}^{N-1}
h(k)x(n-k)
$$

因此，FIR 滤波器只需要有限个当前和过去的输入样本就能够计算当前输出。其系统函数为：

$$
\begin{aligned}
H(z)
&=
\mathcal Z\{h(n)\}\\
&=
\sum_{n=0}^{N-1}
h(n)z^{-n}
\end{aligned}
$$

所以 FIR 的系统函数是一个关于 $z^{-1}$ 的有限多项式。FIR 通常采用**非递归结构**实现，即计算当前输出时不需要过去的输出 $y(n-k)$ 。但需要注意的是：“没有反馈”不是 FIR 的定义，而是 FIR 最自然、最常见的实现方式。

---

> 离散时间 LTI 系统稳定（BIBO）的充要条件是：
> 
> $$
> \sum_{n=-\infty}^{+\infty}|h(n)|<\infty
> $$

**证明：**

BIBO 的意思是：只要输入有界，输出就一定有界。也就是如果存在有限常数 $B_x$ 使得 $|x(n)|\le B_x$ 对所有 n 成立，那么也必须存在有限常数 $B_y$ 使 $|y(n)|\le B_y$ 。

(=>) 充分性：根据假设和输入有界得到，

$$
\sum_{n=-\infty}^{+\infty}|h(n)|<\infty \quad \text{且} \quad |x(n)|\le B_x
$$

所以，

$$
\begin{aligned}
y(n)
&=
\left|\sum_{k=-\infty}^{\infty}
h(k)x(n-k)\right| \\
&\le
\sum_{k=-\infty}^{\infty}
|h(k)||x(n-k)| \\
&\le
B_x
\sum_{k=-\infty}^{\infty}
|h(k)|
\end{aligned}
$$

令 $B_y=B_x\sum_{k=-\infty}^{\infty}|h(k)|$ ，则 $B_y$ 是一个有限常数。

(<=) 必要性：欲证明: $\text{系统稳定} \to h(n)\text{绝对可和}$ 。考虑反证法，假设：

$$
\sum_{n=-\infty}^{+\infty}|h(n)|=\infty
$$

欲证明系统一定不稳定，只需要找到一个有界输入，使输出无界即可。若 $h(n)$ 是实数，我们构造：

$$
x(-k)=
\begin{cases}
1, & h(k) \ge 0 \\
-1, & h(k) < 0
\end{cases}
$$
显然这个输入 $x(n)$ 有界，且

$$
\begin{aligned}
y(0)
&=
\sum_{k=-\infty}^{\infty}
h(k)x(-k) \\
&=
\sum_{k=-\infty}^{\infty}
|h(k)| \\
&= \infty
\end{aligned} 
$$

综上，LTI 系统稳定当且仅当单位脉冲响应 $h(n)$ 绝对可和。$\square$

---

对于长度为 $N$ 的 FIR 滤波器：

$$
\sum_{n=-\infty}^{+\infty}|h(n)|
=
\sum_{n=0}^{N-1}|h(n)|
$$

这里只包含有限个有限数之和，因此必然有：

$$
\sum_{n=0}^{N-1}|h(n)|<\infty
$$

所以，**有限系数的 FIR LTI 滤波器一定是 BIBO 稳定的**。此外，FIR 滤波器还可以通过适当设计 $h(n)$ 的对称性，比较容易地获得严格的线性相位。

---

### IIR 滤波器

IIR（Infinite Impulse Response）称为**无限脉冲响应滤波器**。如果单位脉冲响应 $h(n)$ 有无限多个非零值，则该系统称为 IIR 滤波器。

例如 $h(n)=\left(\frac{1}{2}\right)^n u(n)$ ，其单位脉冲响应为：

$$
h(0)=1\quad
h(1)=\frac12 \quad
h(2)=\frac14 \quad
h(3)=\frac18 \quad \cdots
$$

虽然这些值越来越小，但是永远不会在某一个有限的 $n$ 之后全部严格变成 $0$，因此这是一个 IIR 系统。

根据卷积：

$$
y(n)
=
\sum_{k=0}^{+\infty}
h(k)x(n-k)
$$

代入 $h(k)=\left(\frac12\right)^k$ 得到：

$$
y(n)
=
x(n)
+\frac12x(n-1)
+\frac14x(n-2)
+\frac18x(n-3)
+\cdots
$$

可见 IIR 在数学上仍然完全可以写成“输入与 $h(n)$ 的卷积”，并不要求表达式中一定出现过去的输出 $y(n-k)$。但是这里出现了一个实际问题：如果按照这个表达式直接实现，就需要保存并计算无限多个过去的输入样本，这是无法真正实现的。

对于许多 IIR 系统，可以**利用递归关系将这个无限求和转换成只包含有限项的差分方程**。仍以上面的例子为例：

$$
y(n)
=
x(n)
+\frac12x(n-1)
+\frac14x(n-2)
+\frac18x(n-3)
+\cdots
$$

将 $n$ 替换为 $n-1$：

$$
y(n-1)
=
x(n-1)
+\frac12x(n-2)
+\frac14x(n-3)
+\cdots
$$

可以发现，原式中除 $x(n)$ 之外的所有项，正好等于 $\frac12y(n-1)$，所以

$$
y(n)=x(n)+\frac12y(n-1)
$$

这样，原本需要无限多个历史输入才能计算的无限卷积，就被压缩成了 $x(n)$ 和 $y(n-1)$ 两个量。因此，IIR 通常**利用过去的输出进行反馈，从而用有限计算实现无限长的冲激响应**。这就是为什么 IIR 滤波器通常采用**递归结构**。

但同样需要注意：**“存在反馈”并不是 IIR 的严格定义。IIR 的定义仍然是单位脉冲响应 $h(n)$ 无限长，反馈只是实现 IIR 滤波器最常见的方法。**

|          | FIR                     | IIR                       |
| -------- | ----------------------- | ------------------------- |
| 全称        | Finite Impulse Response       | Infinite Impulse Response      |
| **根本定义** | $h(n)$ 有限长                 | $h(n)$ 无限长                   |
| 卷积形式     | 有限项                        | 无限项                          |
| 典型实现     | 非递归、无反馈                 | 递归、有反馈                     |
| 当前输出通常依赖 | 当前及有限个过去输入         | 当前/过去输入以及有限个过去输出     |
| 系统函数     | 通常为 $z^{-1}$ 的有限多项式    | 通常为有理函数，具有非平凡分母      |
| 稳定性       | 有限系数时天然 BIBO 稳定        | 需要根据极点和 ROC 判断           |
| 线性相位     | 容易实现严格线性相位             | 一般难以实现严格线性相位           |
| 实现相同幅频指标 | 通常需要较高阶数             | 通常可以使用较低阶数               |

---

### FIR 与 IIR 的统一表示

常见的 LTI 数字滤波器，一般地（无论 FIR 还是 IIR），可以使用**线性常系数差分方程**描述：

$$
y(n)
=
\sum_{k=1}^{N}a_k y(n-k)
+
\sum_{k=0}^{M}b_k x(n-k)
$$

其中，$x(n)$ 为输入，$y(n)$ 为输出。第一项表示过去的输出对当前输出的影响，第二项表示当前及过去的输入对当前输出的影响。于是对原差分方程逐项进行 Z 变换：

$$
Y(z)
=
\sum_{k=1}^{N}a_kz^{-k}Y(z)
+
\sum_{k=0}^{M}b_kz^{-k}X(z)
$$

于是得到：

$$
Y(z)
\left(
1-\sum_{k=1}^{N}a_kz^{-k}
\right)
=
X(z)
\sum_{k=0}^{M}b_kz^{-k}
$$

又因为系统函数定义为：

$$
H(z)=\frac{Y(z)}{X(z)}
$$

从而得到：

$$
H(z)
=
\frac{
\displaystyle\sum_{k=0}^{M}b_kz^{-k}
}{
\displaystyle1-\sum_{k=1}^{N}a_kz^{-k}
}
$$

这个一般差分方程也把 FIR 与 IIR 统一了起来。

- 当所有 $a_k=0$ 时，$H(z)$ 退化为关于 $z^{-1}$ 的多项式，对应 FIR；
- 只要存在非零的 $a_k$，$H(z)$ 就是分母非平凡的有理函数，对应 IIR，而 $\sum_{k=1}^{N}a_k y(n-k)$ 正是前面所说的“反馈”项。

因此，Z 变换在离散系统分析中的一个重要作用，就是将时域中的延迟 $x(n-k)$ 转化为 Z 域中的乘法 $z^{-k}X(z)$ 从而**把原本包含多个延迟项的差分方程转化为关于 $X(z)$ 和 $Y(z)$ 的代数方程**。

---

## 线性相位

滤波器对信号的影响不仅体现在幅度上，也体现在相位上。设滤波器频率响应为：

$$
H(e^{j\omega})
=
|H(e^{j\omega})|e^{j\theta(\omega)}
$$

其中 $\theta(\omega)$ 为系统的相频特性。

一个复杂信号通常可以看作多个不同频率分量的叠加。例如：

$$
x(n)
=
\cos(\omega_1 n)
+
\cos(\omega_2 n)
$$

信号最终呈现出的波形不仅取决于两个频率分量各自的幅度，也取决于它们之间的**相位关系**。假设信号通过滤波器后，各频率分量的幅度没有发生变化，但产生了不同的相移：

$$
y(n)
=
\cos[\omega_1n+\theta(\omega_1)]
+
\cos[\omega_2n+\theta(\omega_2)]
$$

如果 $\theta(\omega_1)$ 和 $\theta(\omega_2)$ 使两个频率分量之间原有的相位差发生了改变，那么它们在各个时刻的相长、相消关系也会随之变化，最终重新叠加得到的波形就可能与原信号不同。这称为**相位失真**。

因此，要保持原信号的波形，关键并不只是某一个频率分量发生了多少相移，而是不同频率分量之间的**相对相位关系**不能被破坏。为了描述相位随频率变化的快慢，定义群时延（Group Delay）：

$$
\tau_g(\omega)
=
-\frac{d\theta(\omega)}{d\omega}
$$

它反映了相邻频率分量之间相位差随频率的变化关系。如果要求所有频率分量具有相同的群时延 $\tau$，那么：

$$
\tau_g(\omega)
=
-\frac{d\theta(\omega)}{d\omega}
=
\tau
$$

积分得到：

$$
\theta(\omega)
=
-\tau\omega+\theta_0
$$

因此，当相频特性 $\theta(\omega)$ 是频率 $\omega$ 的线性函数时，群时延为常数，各频率分量之间的相对相位关系能够保持一致，从而避免波形产生相位失真。这就是**线性相位（Linear Phase）**。

---

> 对于一个长度为 $N$ 的实序列 FIR 滤波器，要获得严格线性相位，当且仅当其单位脉冲响应 $h(n)$ 关于 $\frac{N-1}{2}$ 对称或者反对称。

**证明**

设 $h(n)$ 为实序列，记对称中心 $\alpha=\dfrac{N-1}{2}$。由频率响应定义

$$
\begin{aligned}
H(e^{j\omega})
&=\sum_{n=0}^{N-1}h(n)e^{-j\omega n}\\
&=e^{-j\alpha\omega}\sum_{n=0}^{N-1}h(n)\,e^{j\omega(\alpha-n)}\\
\end{aligned}
$$

再按实部和虚部展开：

$$
H(e^{j\omega})e^{j\alpha\omega}
=\sum_{n}h(n)\cos[\omega(\alpha-n)]
\;+\;j\sum_{n}h(n)\sin[\omega(\alpha-n)]
$$

**(⇒) 对称 ⟹ 线性相位。**

**偶对称**：

若 $h(n)=h(N-1-n)$，把 $n$ 与 $N-n-1$ 配对，注意到 $\alpha-(N-1-n)=n-\alpha$，虚部中 $\sin[\omega(\alpha-n)]$ 的项两两抵消，故

$$
H(e^{j\omega})e^{j\alpha\omega}
=\underbrace{\sum_{n}h(n)\cos[\omega(\alpha-n)]}_{A(\omega)\ \text{为实数}}
$$

即 $H(e^{j\omega})=A(\omega)e^{-j\alpha\omega}$。因此 $\theta(\omega)=-\alpha\omega=-\dfrac{N-1}{2}\omega$，群时延 $\tau=\dfrac{N-1}{2}$。

**奇对称**

若 $h(n)=-h(N-1-n)$：此时配对的余弦项两两抵消、正弦项翻倍，得

$$
H(e^{j\omega})e^{j\alpha\omega}=jB(\omega)\quad(B\ \text{为实数})
$$

即 $H(e^{j\omega})=B(\omega)e^{-j\alpha\omega+j\pi/2}$。因此 $\theta(\omega)=-\dfrac{N-1}{2}\omega+\dfrac{\pi}{2}$，群时延仍为 $\tau=\dfrac{N-1}{2}$。

**(⇐) 线性相位 ⟹ 对称（或反对称）。**

设 $\theta(\omega)=-\alpha\omega+\beta$，即 $H(e^{j\omega})=A(\omega)e^{j(-\alpha\omega+\beta)}$，其中 $A(\omega)$ 为实函数。注意到

$$
H(e^{j\omega})=\sum_{n}h(n)e^{-j\omega n}=A(\omega)e^{j(-\alpha\omega+\beta)}
$$

则

$$
\begin{aligned}
\sum_{n}h(n)e^{-j\omega n} \cdot e^{j\alpha\omega}&=A(\omega)e^{j\beta}\\
\sum_{n}h(n)e^{j\omega(\alpha-n)}&=A(\omega)e^{j\beta}
\end{aligned}
$$

因为 $A(\omega)$ 为实数、$e^{j\beta}$ 为常相位因子，所以左式对所有 $\omega$ 都具有恒定相位 $\beta$。我们记

$$
F(w)=\sum_{n}h(n)e^{j\omega(\alpha-n)}=H(e^{j\omega}) \cdot e^{j\omega\alpha}
$$

于是由 $h(n)$ 为实序列，则 $H(e^{j\omega})$ 共轭对称。又由 $e^{j\omega\alpha}$ 显然也关于 $\omega$ 共轭对称。那么

$$
F(-\omega)=\overline{F(\omega)}
$$

由于 $F(w)$ 恒定相位为 $\beta$，$\overline{F(\omega)}$ 相位也恒定为 $-\beta$，代入上式得到：

$$
F(-\omega)=e^{-2j\beta}F(\omega)
$$

把 $F$ 的定义代入两端：

$$
\sum_{n}h(n)e^{j\omega(n-\alpha)}=\sum_{n}h(n)e^{-2j\beta}e^{j\omega(\alpha-n)}
$$

二者对任意 $\omega$ 恒等，逐项比较 $e^{j\omega k}$ 的系数（左边取 $k=n-\alpha$，右边取 $k=\alpha-n$）得到：

$$
h(\alpha+k)=e^{-2j\beta}\,h(\alpha-k)
$$

由于 $h(\alpha+k)$ 和 $h(\alpha-k)$ 都是实数，因此 $e^{-2j\beta}$ 也必须是实数；而它的模又恒为 $1$，实数且模为 $1$ 的复数只有 $\pm1$ 两种可能。于是：

- 若 $e^{-2j\beta}=1$（即 $\beta=0,\pi$），则 $h(\alpha+k)=h(\alpha-k)$，即 $h(n)$ **关于 $\alpha$ 偶对称**；
- 若 $e^{-2j\beta}=-1$（即 $\beta=\pm\frac{\pi}{2}$），则 $h(\alpha+k)=-h(\alpha-k)$，即 **关于 $\alpha$ 奇对称**。

由于 $h(n)$ 只在 $0\le n\le N-1$ 内非零，这一对称中心只能是 $\alpha=\dfrac{N-1}{2}$。$\quad\square$

---

上述证明的结果可以汇总成下面的表。由于 $h(n)$ 可以是偶对称或奇对称，而序列长度 $N$ 又可以是奇数或偶数，因此一共有四种情况，每种对应一种相位与群时延：

| 类型 | 对称性            | $N$ | 相位 $\theta(\omega)$ | 群时延 $\tau$ | 必然满足的性质      |
| -- | ---------------- | --- | ---------------- | ---------- | ------------------------- |
| 1  | $h(n)=h(N-1-n)$  | 奇数  | $-\frac{N-1}{2}\omega$ | $\frac{N-1}{2}$ |                 |
| 2  | $h(n)=h(N-1-n)$  | 偶数  | $-\frac{N-1}{2}\omega$ | $\frac{N-1}{2}$ | $H(e^{j\pi})=0$ |
| 3  | $h(n)=-h(N-1-n)$ | 奇数  | $-\frac{N-1}{2}\omega+\frac{\pi}{2}$ | $\frac{N-1}{2}$ | $H(e^{j0})=H(e^{j\pi})=0$ |
| 4  | $h(n)=-h(N-1-n)$ | 偶数  | $-\frac{N-1}{2}\omega+\frac{\pi}{2}$ | $\frac{N-1}{2}$ | $H(e^{j0})=0$             |

> 表里的相位常数 $\beta$ 只在 $\mathrm{mod}\,\pi$ 意义下有定义。因为 $H(e^{j\omega})=A(\omega)e^{j(\beta-\alpha\omega)}$ 中 $A(\omega)$ 为实数、可正可负，把 $-1$ 吸收进 $A(\omega)$ 就等价于给相位加 $\pi$，所以 $\beta=0$ 与 $\beta=\pi$ 同属一类（偶对称），$\beta=+\dfrac{\pi}{2}$ 与 $\beta=-\dfrac{\pi}{2}$ 也同属一类（奇对称）。正负号只是记号选择，不影响幅频特性与群时延。这一约定的根源是 $h(n)$ 为实序列时频谱的共轭对称性（$|H|$ 为偶函数、相位为奇函数），它使我们只需在 $[0,\pi]$ 上讨论问题。

例如，如果希望设计一个在 $\omega=\pi$ 处具有非零响应的线性相位高通滤波器，就不能使用偶长度的偶对称 FIR，因此通常选择**奇数长度的偶对称 FIR**。

---

## FIR 滤波器的设计

设计数字滤波器时，我们通常已经知道希望获得的理想频率响应 $H_d(e^{j\omega})$，然后寻找一个实际可以实现的系统 $H(e^{j\omega})$，使其尽可能逼近 $H_d(e^{j\omega})$。

FIR 滤波器常见的设计方法有：

1. **窗函数法**：在时域中进行逼近；
2. **频率采样法**：在频域中进行逼近；
3. **最优化设计法**：例如等波纹逼近。

这里主要讨论**窗函数设计法**。

---

## 窗函数

假设希望得到一个理想频率响应 $H_d(e^{j\omega})$。首先进行 DTFT 反变换：

$$
h_d(n) =
\frac{1}{2\pi}
\int_{-\pi}^{\pi}
H_d(e^{j\omega})e^{j\omega n}
d\omega
$$

即可得到理想滤波器的单位脉冲响应 $h_d(n)$。问题在于，理想滤波器的频率响应通常存在突变，因此其单位脉冲响应 $h_d(n)$ 往往是一个**无限长序列**。而一个实际可以实现的 FIR 滤波器必须满足：

$$
h(n)=0,\qquad n<0\ \text{或}\ n>N-1
$$

也就是说，$h(n)$ 必须是有限长、因果序列。因此，我们需要使用一个有限长序列 $h(n)$ 去逼近无限长的 $h_d(n)$。最直接的方法就是：**从理想单位脉冲响应 $h_d(n)$ 中截取有限的一段。**这种“截取”可以看作给 $h_d(n)$ 乘上一个有限长度的窗函数 $w(n)$：

$$
h(n)=h_d(n)w(n)
$$

这就是**窗函数设计法**的核心思想。

---

我们以一个例子展示窗函数的设计过程。考虑一个截止频率为 $\omega_c$ 的理想线性相位低通滤波器：

$$
H_d(e^{j\omega}) =
\begin{cases}
e^{-j\omega\alpha}, & |\omega|\leq\omega_c\\
0, & \omega_c<|\omega|\leq\pi
\end{cases}
$$

其中 $e^{-j\omega\alpha}$ 对应一个 $\alpha$ 个采样点的延迟。

![](https://ref.xht03.online/202609012113735.png)

对其进行 DTFT 反变换：

$$
\begin{aligned}
h_d(n) &=
\frac{1}{2\pi}
\int_{-\omega_c}^{\omega_c}
e^{-j\omega\alpha}e^{j\omega n}
d\omega\\
&=
\frac{1}{2\pi}
\int_{-\omega_c}^{\omega_c}
e^{j\omega(n-\alpha)}
d\omega
\end{aligned}
$$

积分后得到：

$$
h_d(n)=
\begin{cases}
\dfrac{\sin[\omega_c(n-\alpha)]}
{\pi(n-\alpha)}, & n\neq\alpha\\[8pt]
\dfrac{\omega_c}{\pi}, & n=\alpha
\end{cases}
$$

可见，$h_d(n)$ 是一个以 $\alpha$ 为中心对称的无限长序列。为了保证得到线性相位 FIR 滤波器，应令 $\alpha=\frac{N-1}{2}$。这样截取之后得到的 $h(n)$ 仍然关于 $(N-1)/2$ 对称。

![](https://ref.xht03.online/202609012111069.png)

---

我们再考虑如何设计窗函数。最简单的窗函数是矩形窗：

$$
R_N(n) =
\begin{cases}
1,&0\leq n\leq N-1\\
0,&\text{其他}
\end{cases}
$$

如果我们使用矩形窗，也就是直接将 $h_d(n)$ 在 $0\sim N-1$ 之外的部分全部截去。

$$
h(n) = h_d(n)R_N(n)
$$

那么，矩形窗的频率响应为：

$$
\begin{aligned}
W_R(e^{j\omega})&=\sum_{n=-\infty}^{\infty}R_N(n)e^{-j\omega n}
=\sum_{n=0}^{N-1}1\cdot e^{-j\omega n}\\
&=\frac{1-e^{-j\omega N}}{1-e^{-j\omega}}\\
&=\frac{e^{-j\omega N/2}\big(e^{j\omega N/2}-e^{-j\omega N/2}\big)}{e^{-j\omega/2}\big(e^{j\omega/2}-e^{-j\omega/2}\big)}\\
&=\underbrace{e^{-j\omega\frac{N-1}{2}}}_{\text{线性相位(延迟)}}\cdot\frac{\sin(N\omega/2)}{\sin(\omega/2)}
\end{aligned}
$$

其幅度函数的第一个零点位于 $\omega=\pm\frac{2\pi}{N}$，因此主瓣的零点到零点宽度约为 $\frac{4\pi}{N}$，随着 $N$ 增大，主瓣会越来越窄。

![](https://ref.xht03.online/202609012124423.png)

---

由于

$$
h(n)=h_d(n)w(n)
$$

而 DTFT 中，**时域相乘对应频域卷积**。所以

$$
H(e^{j\omega}) =
\frac{1}{2\pi}
H_d(e^{j\omega})
*
W(e^{j\omega})
$$

也就是说，实际得到的滤波器频率响应并不是理想频率响应 $H_d(e^{j\omega})$，而是理想频率响应 $H_d(e^{j\omega})$ 与窗函数频谱 $W(e^{j\omega})$ 的卷积。因此，窗函数的频谱直接决定了最终滤波器的性能。

其中有两个非常重要的关系：

$$
\begin{aligned}
\text{窗函数主瓣宽度}
&\longleftrightarrow
\text{滤波器过渡带宽度}\\
\text{窗函数旁瓣高度}
&\longleftrightarrow
\text{滤波器通带、阻带波纹}
\end{aligned}
$$

也就是

- 主瓣越窄，滤波器从通带进入阻带的变化就可以越迅速，因此过渡带越窄。
- 旁瓣越低，理想频率响应在卷积之后受到的“泄漏”越小，因此通带和阻带的波动越小，阻带衰减越大。

---

## Gibbs 现象

当我们用矩形窗把理想低通滤波器的无限长脉冲响应 $h_d(n)$ 截断成有限长 $N$ 后，得到的实际频响 $H(e^{j\omega})$ 并不能完美复现理想的矩形频响。它会在理想频响的**跳变位置**（即通带边缘 $\omega_c$）附近出现明显的**振荡纹波**：通带边缘向外"鼓"出一个尖峰（过冲），两侧还有一圈圈起伏。这种"在间断点附近出现、且不随项数增加而消失的振荡"，就是**吉布斯现象**。

它的两个关键特征：

- 最大波动约为理想跳变量的 $8.95\%$；
- 这个波动**不会随 $N$ 的增大而消失**。$N$ 越大，纹波只是被压得越贴近 $\omega_c$、振得越密，峰值高度保持不变。


![](https://ref.xht03.online/202609012141180.png)

**为什么会这样？**

理想低通滤波器的频响是一个以 $2\pi$ 为周期的周期函数：

$$
H_d(e^{j\omega})=
\begin{cases}
e^{-j\omega\alpha},&|\omega|\le\omega_c\\
0,&\omega_c<|\omega|\le\pi
\end{cases}
$$

它在 $\omega=\pm\omega_c$ 处从 $1$ 跳到 $0$，是一个**阶跃间断**。任何周期函数都能展开成傅里叶级数，而离散时间系统的频响展开式正是

$$
H(e^{j\omega})=\sum_{n=-\infty}^{\infty}h(n)\,e^{-j\omega n}
$$

这里的“级数系数”就是单位冲激响应 $h(n)$。要精确表示那个间断的矩形频响，需要**无穷多项**，这对应着 $h_d(n)$ 是一个无限长序列。把 $h_d(n)$ 用矩形窗截断成 $N$ 个采样，等价于只保留傅里叶级数的前 $N$ 项：

$$
H(e^{j\omega})=\sum_{n=0}^{N-1}h(n)\,e^{-j\omega n}
$$

所以整个 FIR 设计过程，本质上就是**用有限项傅里叶级数去逼近一个不连续的周期函数**。Gibbs 的经典结论是：有限项级数在间断点附近必然出现过冲振荡，并且**项数再多也不会消失**，只会更贴近间断点、振得更密。

---

## 常用窗函数

### 矩形窗

$$
w(n)=1,
\qquad
0\leq n\leq N-1
$$

矩形窗的主瓣最窄，但旁瓣较高，因此过渡带较窄，而阻带波纹较大。

---

### Bartlett 窗

Bartlett 窗又称**三角窗**：

$$
w(n) =
1-
\left|
\frac{2n-(N-1)}{N-1}
\right|,
\qquad
0\leq n\leq N-1
$$

与矩形窗相比，其旁瓣有所降低，但主瓣变宽。

---

### Hann 窗

Hann 窗也常称为汉宁窗或 Hanning 窗，是一种升余弦窗：

$$
w(n) =
0.5 - 0.5\cos
\left(
\frac{2\pi n}{N-1}
\right)
$$

其中：

$$
0\leq n\leq N-1
$$

Hann 窗进一步降低了旁瓣，因此能够获得更好的阻带衰减，但过渡带也更宽。

---

### Hamming 窗

Hamming 窗是改进的升余弦窗：

$$
w(n) =
0.54 - 0.46\cos
\left(
\frac{2\pi n}{N-1}
\right)
$$

其中：

$$
0\leq n\leq N-1
$$

它的旁瓣比 Hann 窗更低，因此阻带衰减能力更强。

---

不同窗函数的典型性能如下：

| 窗函数                      | 旁瓣峰值 / dB |  过渡带宽度近似值 |   过渡带宽度参考值 | 阻带最低电平 / dB |
| ------------------------ | --------: | --------: | ---------: | ----------: |
| 矩形窗                      |     $-13$ |  $4\pi/N$ | $1.8\pi/N$ |       $-21$ |
| 三角窗                      |     $-25$ |  $8\pi/N$ | $6.1\pi/N$ |       $-25$ |
| Hann 窗                   |     $-31$ |  $8\pi/N$ | $6.2\pi/N$ |       $-44$ |
| Hamming 窗                |     $-41$ |  $8\pi/N$ | $6.6\pi/N$ |       $-53$ |
| Blackman 窗               |     $-57$ | $12\pi/N$ |  $11\pi/N$ |       $-74$ |
| Kaiser 窗 $(\beta=7.865)$ |     $-57$ |         — |  $10\pi/N$ |       $-80$ |

如果将阻带衰减 $\alpha_s$ 表示为正值，那么例如 Hann 窗对应的阻带衰减约为 $44,\mathrm{dB}$，Hamming 窗约为 $53,\mathrm{dB}$。可以看出：

> 窗函数的旁瓣越低，一般阻带衰减越强；但与此同时主瓣往往越宽，从而导致过渡带变宽。

因此窗函数的选择实际上就是在**过渡带宽度**与**阻带衰减**之间进行折中。

---

## 设计窗函数

使用窗函数设计 FIR 滤波器时，可以按照如下过程：

1. 根据给出的通带、阻带指标选择合适的窗函数；

2. 根据允许的过渡带宽度确定滤波器长度 $N$；

3. 确定理想截止频率 $\omega_c$，构造理想频率响应 $H_d(e^{j\omega})$；

4. 对 $H_d(e^{j\omega})$ 进行 DTFT 反变换，得到理想单位脉冲响应 $h_d(n)$；

5. 计算窗函数 $w(n)$，得到：

   $$
   h(n)=h_d(n)w(n)
   $$

6. 计算实际频率响应 $H(e^{j\omega})$，检查是否满足设计指标；如果不满足，则重新选择窗函数或滤波器长度 $N$。

---

> 例 1
> 要求设计一个线性相位高通 FIR 滤波器 $\omega_p=\frac{\pi}{2}$，$\omega_s=\frac{\pi}{4}$，通带最大衰减为 $\alpha_p=1\text{ dB}$，阻带最小衰减为 $\alpha_s=40\text{ dB}$。

**解**：

第一步，选择窗函数并确定长度。要求阻带衰减至少为 $40\text{ dB}$。Hann 窗的典型阻带衰减约为 $44\text{ dB}$，因此可以选择 Hann 窗。允许的过渡带宽度为

$$
B_t
\leq
\omega_p-\omega_s
=
\frac{\pi}{4}
$$

Hann 窗的过渡带宽度参考值约为：

$$
B_t
=
\frac{6.2\pi}{N}
$$

因此要求：

$$
\frac{6.2\pi}{N}
\leq
\frac{\pi}{4}
$$

得到：

$$
N\geq24.8
$$

对于这种线性相位高通滤波器，为保证 $\omega=\pi$ 处不被强制为零，应使用奇数长度，因此取 $N=25$。相应的对称中心为 $\alpha=\frac{N-1}{2}=12$

---

第二步，确定理想截止频率。通常取过渡带中心作为理想截止频率

$$
\omega_c
=
\frac{\omega_p+\omega_s}{2}
$$

因此

$$
\omega_c
=
\frac{
\frac{\pi}{2}+\frac{\pi}{4}
}{2}
=
\frac{3\pi}{8}
$$

理想高通滤波器为

$$
H_d(e^{j\omega})
=
\begin{cases}
0, & |\omega|<\dfrac{3\pi}{8}\\[6pt]
e^{-j12\omega}, & \dfrac{3\pi}{8}\leq|\omega|\leq\pi
\end{cases}
$$

---

第三步，求理想单位脉冲响应。理想高通滤波器可以看作**全通滤波器减去截止频率为 $\omega_c$ 的理想低通滤波器**。延迟 $\alpha$ 个采样点的全通系统，其单位脉冲响应为 $\delta(n-\alpha)$，因此：

$$
h_d(n)
=
\delta(n-\alpha)
-
h_{LP}(n)
$$

其中：

$$
h_{LP}(n) =
\begin{cases}
\dfrac{\sin[\omega_c(n-\alpha)]}
{\pi(n-\alpha)}, & n\neq\alpha\\[8pt]
\dfrac{\omega_c}{\pi}, & n=\alpha
\end{cases}
$$

代入 $\alpha=12$ 和 $\omega_c=\frac{3\pi}{8}$ 得到：

$$
h_d(n)
=
\begin{cases}
-\dfrac{
\sin\left[\dfrac{3\pi}{8}(n-12)\right]
}
{\pi(n-12)}, & n\neq12\\[10pt]
1-\dfrac38=\dfrac58, & n=12
\end{cases}
$$

---

第四步，加窗。选择长度为 $N=25$ 的 Hann 窗：

$$
w(n) =
0.5 - 0.5\cos
\left(
\frac{2\pi n}{24}
\right),
\qquad
0\leq n\leq24
$$

最终 FIR 滤波器的单位脉冲响应为：

$$
h(n)=h_d(n)w(n)
$$

得到 $h(n)$ 后即可通过 DTFT 求出实际频率响应 $H(e^{j\omega})$，并检查其通带衰减、阻带衰减和过渡带宽度是否满足要求。

---

## 有限字长效应

在前面的理论分析中，我们默认所有数据都可以具有无限的计算精度。但实际数字系统只能使用有限数量的二进制位来表示数据，因此滤波器系数和中间运算结果都需要进行量化。于是实际系统中会出现**有限字长效应（Finite Word Length Effect）**。

例如，本来希望使用系数 $b_k$，实际计算机中只能保存其有限精度近似值 $\hat b_k=b_k+\Delta b_k$。系数发生变化之后，系统函数也会发生变化：

$$
H(z)
\longrightarrow
\hat H(z)
$$

因此其零极点位置以及频率响应都会产生误差。此外，在每一次乘法和加法运算后进行有限精度舍入，也会产生量化误差。因此，数字滤波器虽然在数学上由精确的 $H(z)$ 或 $h(n)$ 描述，但真正实现到数字硬件或计算机上时，还必须考虑有限精度带来的误差。
