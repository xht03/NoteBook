我们考虑一个离散时间 LTI 系统，已知它的冲激响应 $h(n)$。如果给定一个输入 $x(n)$，如何求输出 $y(n)$ 呢？根据第一章的内容，我们已经学会了时域的做法：输出等于输入与冲激响应的卷积。

$$
y(n)=x(n)*h(n)=\sum_{k=-\infty}^{+\infty}h(k)x(n-k)
$$

这是一个**无穷和**，且每一项都要算，相当费劲。那么有没有更省事的办法？**有，办法就是换到频域**。

<!-- 本节的全部目的，就是讲清楚：**为什么在频域，这个无穷的卷积会简化成一次乘法。** -->

---

先看一个最简单的输入，复指数 $x(n)=e^{j\omega n}$，我们试问：这个复指数经过 LTI 系统后，输出如何？

把 $x(n)=e^{j\omega n}$ 代入 $y=x*h$

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
- 后面那串 $\sum_{k=-\infty}^{+\infty}h(k)\,e^{-j\omega k}$ **和 n 一点关系都没有**，它只取决于频率 $\omega$。

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
那推广到任意信号 $x(n)$，经过 LTI 系统后，输出如何呢？

根据 DTFT 的逆变换，任意信号 $x(n)$ 可以写成复指数的"连续加权和"：

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

这就是卷积定理：**一个时域的无穷卷积 $y=x*h$，在频域只是一次乘法**。

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

