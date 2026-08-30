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

---

## Z 变换的性质

设序列 $x(n)$ 的双边 Z 变换为：

$$
X(z)
=
\sum_{n=-\infty}^{+\infty}
x(n)z^{-n}
$$

Z 变换具有许多重要性质，其中线性、时间移位和时域卷积在离散时间系统分析中尤其重要。

### 线性

设：

$$
x_1(n)
\overset{\mathcal Z}{\leftrightarrow}
X_1(z)
$$

$$
x_2(n)
\overset{\mathcal Z}{\leftrightarrow}
X_2(z)
$$

根据 Z 变换的定义，我们有：

$$
\begin{aligned}
\mathcal Z\{ax_1(n)+bx_2(n)\}
&=
\sum_{n=-\infty}^{+\infty}
[ax_1(n)+bx_2(n)]z^{-n}\\
&=
a\sum_{n=-\infty}^{+\infty}x_1(n)z^{-n}
+
b\sum_{n=-\infty}^{+\infty}x_2(n)z^{-n}\\
&=
aX_1(z)+bX_2(z)
\end{aligned}
$$

则对于任意常数 $a,b$ ，均有：

$$
\boxed{
ax_1(n)+bx_2(n)
\overset{\mathcal Z}{\leftrightarrow}
aX_1(z)+bX_2(z)
}
$$

因此 Z 变换满足**线性**性质。其收敛域至少包含 $X_1(z)$ 和 $X_2(z)$ 收敛域的交集。若相加过程中发生零极点相消，实际收敛域可能进一步扩大。

---

### 时间移位

设：

$$
x(n)
\overset{\mathcal Z}{\leftrightarrow}
X(z)
$$

考虑将序列延迟 $k$ 个采样点 $x(n-k)$ ，其 Z 变换为：

$$
\mathcal Z\{x(n-k)\}
=
\sum_{n=-\infty}^{+\infty}
x(n-k)z^{-n}
$$

令 $m=n-k$ ，则：

$$
\begin{aligned}
\mathcal Z\{x(n-k)\}
&=
\sum_{m=-\infty}^{+\infty}
x(m)z^{-(m+k)}\\
&=
z^{-k}
\sum_{m=-\infty}^{+\infty}
x(m)z^{-m}\\
&=
z^{-k}X(z)
\end{aligned}
$$

所以：

$$
\boxed{
x(n-k)
\overset{\mathcal Z}{\leftrightarrow}
z^{-k}X(z)
}
$$

因此，**时域中的延迟对应 Z 域中乘以 $z^{-k}$**。

这一性质在使用 Z 变换求解差分方程时非常重要，因为它可以把时域中的延迟项转换为 Z 域中的代数乘法。时间移位通常不会改变 ROC 的基本边界，但由于乘上了 $z^{\pm k}$，可能影响 $z=0$ 或 $z=\infty$ 是否属于 ROC。

---

### 时域卷积

设：

$$
x_1(n)
\overset{\mathcal Z}{\leftrightarrow}
X_1(z)
$$

$$
x_2(n)
\overset{\mathcal Z}{\leftrightarrow}
X_2(z)
$$

两个序列的离散卷积定义为：

$$
y(n)
=
x_1(n)*x_2(n)
=
\sum_{k=-\infty}^{+\infty}
x_1(k)x_2(n-k)
$$

对 $y(n)$ 进行 Z 变换：

$$
\begin{aligned}
Y(z)
&=
\sum_{n=-\infty}^{+\infty}
y(n)z^{-n}\\
&=
\sum_{n=-\infty}^{+\infty}
\left[
\sum_{k=-\infty}^{+\infty}
x_1(k)x_2(n-k)
\right]z^{-n}\\
&=
\sum_{k=-\infty}^{+\infty}
x_1(k)
\sum_{n=-\infty}^{+\infty}
x_2(n-k)z^{-n}
\end{aligned}
$$

根据前面的时间移位性质：

$$
\sum_{n=-\infty}^{+\infty}
x_2(n-k)z^{-n}
=
z^{-k}X_2(z)
$$

所以：

$$
\begin{aligned}
Y(z)
&=
\sum_{k=-\infty}^{+\infty}
x_1(k)z^{-k}X_2(z)\\
&=
X_2(z)
\sum_{k=-\infty}^{+\infty}
x_1(k)z^{-k}\\
&=
X_1(z)X_2(z)
\end{aligned}
$$

因此：

$$
\boxed{
x_1(n)*x_2(n)
\overset{\mathcal Z}{\leftrightarrow}
X_1(z)X_2(z)
}
$$

即：

> **时域中的卷积，对应 Z 域中的乘法。**

其 ROC 至少包含 $X_1(z)$ 与 $X_2(z)$ 收敛域的交集；如果相乘后发生零极点相消，实际 ROC 可能更大。这一性质对于分析 线性时不变（LTI）系统尤其重要。

---

## 系统函数与频率响应

对于一个离散时间 LTI 系统，设输入为 $x(n)$、输出为 $y(n)$，输入输出满足卷积关系 $y(n)=x(n)*h(n)$，其中 $h(n)$ 是系统的单位脉冲响应。将 $h(n)$ 的 Z 变换称为系统的**系统函数**（transfer function）：

$$
H(z)=\mathcal Z\{h(n)\}=\sum_{n=-\infty}^{+\infty}h(n)z^{-n}
$$

由本章的时域卷积性质 $Z\{x(n)*h(n)\}=X(z)H(z)$，输入输出的 Z 变换满足：

$$
Y(z)=X(z)H(z)
$$

因此系统函数也可以写成输出与输入 Z 变换之比：

$$
H(z)=\frac{Y(z)}{X(z)}
$$

在本章开头我们提到，令 $z=re^{j\omega}$，$X(z)$ 就相当于序列 $x(n)r^{-n}$ 的 DTFT。当 $r=1$，也就是 $z$ 落在**单位圆**上时，Z 变换就退化为 DTFT。因此，只要单位圆位于 $H(z)$ 的收敛域（ROC）中，就能在单位圆上得到系统的频率响应：

$$
H(e^{j\omega})=H(z)\big|_{z=e^{j\omega}}
$$

也就是说，**系统函数 $H(z)$ 在单位圆上的取值，就是系统的频率响应。** 这建立了 Z 平面上的零极点分布与频率响应之间的联系。设：

$$
H(z)=K\frac{\prod_i(z-z_i)}{\prod_k(z-p_k)}
$$

那么在单位圆 $z=e^{j\omega}$ 上：

$$
|H(e^{j\omega})|=|K|\frac{\prod_i|e^{j\omega}-z_i|}{\prod_k|e^{j\omega}-p_k|}
$$

其中 $z_i$ 为零点，$p_k$ 为极点。因此 $|H(e^{j\omega})|$ 可以理解为：**单位圆上的点 $e^{j\omega}$ 到所有零点距离之积，与到所有极点距离之积的比值**。于是：

* 当单位圆上的某一点靠近**零点**时，$|H(e^{j\omega})|$ 会减小；
* 当单位圆上的某一点靠近**极点**时，$|H(e^{j\omega})|$ 会增大。

这让我们可以直接通过零极点的位置大致判断系统的频率特性。例如：

$$
H(z)=\frac{1}{1-0.9z^{-1}}=\frac{z}{z-0.9}
$$

其零点位于 $z=0$，极点位于实轴上的 $z=0.9$。当 $\omega=0$ 时，单位圆上的点为 $z=1$，距离极点 $0.9$ 很近，因此频率响应较大；而当 $\omega=\pi$ 时，$z=-1$，距离极点较远，因此频率响应较小。所以该系统具有明显的**低通特性**。

零极点还与稳定性直接相关：对于一个因果系统（$h(n)=0,\ n<0$，ROC 为某个圆外区域），当且仅当其所有极点都位于单位圆内（$|p_k|<1$）时，系统是 BIBO 稳定的。

