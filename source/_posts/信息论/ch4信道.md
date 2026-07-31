---
title: Chapter 4 Information Channels
date: 2025-04-20 11:04:10
tags:
- translation
categories:
- Information and Coding Theory
---

> The equivocation of the fiend that lies like truth. —— Macbeth

在本章中，我们考虑这样一种信息传输过程：一个信息源通过一个**不可靠的（unreliable/noisy）信道**向接收者发送消息。信道中的“噪声（noise）”可能表示：

- 机械错误；
- 人为错误；
- 来自另一个信息源的干扰。

一个很好的例子是：一个电源不足的太空探测器发送回一条消息，而这条消息必须从许多其他更强的竞争信号中被提取出来。由于噪声的存在，接收到的符号可能并不与发送出的符号相同。

本章的目标是利用熵函数的几种不同形式，衡量：

- 有多少信息被传输了；
- 在这个过程中有多少信息丢失。

然后进一步研究：这些信息量与所使用编码的平均码长之间的关系。

## 4.1 Notation and Definitions

我们将信息信道 $\Gamma$ 的输入看作一个信息源 $A$。设 $A$ 的符号集合（字母表）为有限集合 $A=\{a_1,\ldots,a_r\}$，其中各符号具有概率 $p_i=\Pr(a=a_i)$，且要求

$$
0\leq p_i\leq1 \text{ and } \sum_{i=1}^{r}p_i=1
$$

我们假设：每当一个符号 $a_i\in A$ 被发送进入信道 $\Gamma$ 时，都会有某个符号从信道 $\Gamma$ 中输出。信道 $\Gamma$ 的输出将被看作另一个信息源 $B$。设 $B$ 的符号集合（有限字母表）为 $B=\{b_1,\ldots,b_s\}$，并且各符号具有概率 $q_j=\Pr(b=b_j)$

其中：

$$
0\leq q_j\leq1 \text{ and } \sum_{j=1}^{s}q_j=1
$$

> **例 4.1**
>
> 在**二元对称信道**（Binary Symmetric Channel，简称 BSC）中，我们有 $A=B=\mathbb{Z}_2=\{0,1\}$。每个输入符号：$a=0 \text{ or } 1$，以概率 $P$ 被正确传输，并且以概率 $\bar{P}=1-P$ 被错误传输。其中 $P$ 为某个常数，并满足 $0\leq P\leq1$。如图 4.2 所示。

![](https://ref.xht03.online/202607301256513.png)

> **例 4.2**
> 
> 在**二元擦除信道**（Binary Erasure Channel，简称 BEC）中，我们有 $A=\mathbb{Z}_2=\{0,1\}$，并且 $B=\{0,1,?\}$。每个输入符号： $a=0 \text{ or } 1$，以概率 $P$ 被正确传输，并且以概率 $\bar{P}$ 被擦除（erased）。擦除由输出符号 $b=?$ 表示。如图 4.3 所示。

![](https://ref.xht03.online/202607301300885.png)

---

通常情况下，我们假设信道 $\Gamma$ 的行为完全由它的**前向概率（forward probabilities）**决定：

$$
P_{ij}
=
\Pr(b=b_j\mid a=a_i)
=
\Pr(b_j\mid a_i)
$$

因此，$P_{ij}$ 是这样的条件概率：已知对应的输入符号为 $a_i$，输出符号 $b$ 为 $b_j$ 的概率。我们假设：

- $P_{ij}$ 与时间无关；
- $P_{ij}$ 不依赖于之前发送或接收的任何符号。

那么，如果：$a=a_i$，则 $b$ 必定是所有输出符号 $b_j$ 中的某一个，因此：

$$
\sum_{j=1}^{s}P_{ij}=1  \quad \forall i=1,\ldots,r
$$

这 $rs$ 个数 $P_{ij}$ 构成了**信道矩阵（channel matrix）**：

$$
M=(P_{ij})
=
\begin{pmatrix}
P_{11}&\cdots&P_{1s}\\
\vdots&\ddots&\vdots\\
P_{r1}&\cdots&P_{rs}
\end{pmatrix}
$$

例如，如果 $\Gamma$ 是 BSC（二元对称信道）或 BEC（二元擦除信道），则分别有：

$$
M=
M=
\begin{pmatrix}
P&\bar P\\
\bar P&P
\end{pmatrix}
\text{ or }
\begin{pmatrix}
P&0&\bar P\\
0&P&\bar P
\end{pmatrix}
$$

信道矩阵 $M$ 的具体形式取决于：输入符号 $a_i$ 的排列顺序，以及输出符号 $b_j$ 的排列顺序。如果改变排列顺序，就会分别导致矩阵 $M$ 的行或列发生置换。上面的 BEC 矩阵采用输出符号顺序是：$0,1,?$，而如果采用输出符号顺序：$0,?,1$，则信道矩阵变为：

$$
M=
\begin{pmatrix}
P&\bar P&0\\
0&\bar P&P
\end{pmatrix}
$$

---

有多种方法可以将两个信道 $\Gamma$ 和 $\Gamma'$ 组合成一个新的信道。

如果 $\Gamma$ 和 $\Gamma'$ 具有不相交的输入字母表 $A,\ A'$，以及不相交的输出字母表 $B,\ B'$，那么它们的和 $\Gamma+\Gamma'$ 具有输入和输出字母表：$A\cup A'$ 和 $B\cup B'$。其信道矩阵为一个分块矩阵：

$$
\begin{pmatrix}
M&0\\
0&M'
\end{pmatrix}
$$

其中 $M$ 和 $M'$ 分别是信道 $\Gamma$ 和 $\Gamma'$ 的信道矩阵。显然，这个定义可以自然推广到任意有限个信道的和。

对于乘积 $\Gamma\times\Gamma'$，我们不需要假设：$A$ 和 $A'$ 是不相交的，且 $B$ 和 $B'$ 是不相交的。此时，输入字母表为 $A\times A'$，输出字母表为 $B\times B'$。发送端发送一个输入符号对 $(a,a')\in A\times A'$，其方式为通过信道 $\Gamma$ 和 $\Gamma'$ 同时分别发送 $a$ 和 $a'$，接收到的符号为 (b,b')\in B\times B'$。

于是，前向概率（forward probabilities）为：

$$
\Pr((b,b')\mid(a,a'))
=
\Pr(b\mid a)\cdot\Pr(b'\mid a')
$$

因此，该信道的信道矩阵为矩阵 $M$ 和 $M'$ 的 **Kronecker 积**：

$$
M\otimes M'
$$

具体地，若 $M=(P_{ij})$ 和 $M'=(P'_{kl})$，分别是大小为 $r\times s$ 和 $r'\times s'$ 的矩阵，那么 $M\otimes M'$

是一个 $rr'\times ss'$ 的矩阵，其元素为 $P_{ij}P'_{kl}$ 这些元素的排列顺序取决于 $A\times A'$ 和 $B\times B'$ 中的元素顺序。

> **例 4.3**
>
> 如果 $\Gamma$ 和 $\Gamma'$ 都是二元对称信道（binary symmetric channels），其信道矩阵分别为：
>
> $$
> M=
> \begin{pmatrix}
> P&\bar P\\
> \bar P&P
> \end{pmatrix}
> \quad
> M'=
> \begin{pmatrix}
> P'&\bar P'\\
> \bar P'&P'
> \end{pmatrix}
> $$
>
> 那么 $\Gamma+\Gamma'$ 和 $\Gamma\times\Gamma'$ 的信道矩阵分别为：
>
> $$
> \begin{pmatrix}
> P&\bar P&0&0\\
> \bar P&P&0&0\\
> 0&0&P'&\bar P'\\
> 0&0&\bar P'&P'
> \end{pmatrix}
> \quad
> \begin{pmatrix}
> PP'&\bar PP'&P\bar P'&\bar P\bar P'\\
> \bar PP'&PP'&\bar P\bar P'&P\bar P'\\
> P\bar P'&\bar P\bar P'&PP'&\bar PP'\\
> \bar P\bar P'&P\bar P'&\bar PP'&PP'
> \end{pmatrix}
> $$
>
> 对于 $\Gamma\times\Gamma'$，我们采用 $A\times A'=B\times B'=\mathbb{Z}_2^2$ 中元素的排列顺序：$(0,0),(1,0),(0,1),(1,1)$。

> **练习 4.1**
>
> 信道 $\Gamma$ 的输出被用作另一个信道 $\Gamma'$ 的输入。求由此得到的复合信道 $\Gamma\circ\Gamma'$ 的信道矩阵，并用 $\Gamma$ 和 $\Gamma'$ 的信道矩阵表示。将这一结果推广到任意多个串联信道的组合情况，这称为**信道级联（cascade of channels）**。

---

回到单个信道 $\Gamma$ 的情况。将两个等式：$\sum_i p_i=1$ 和 $\sum_jP_{ij}=1$ 相乘，得到：

$$
\sum_{i=1}^{r}\sum_{j=1}^{s}p_iP_{ij}=1
\tag{4.1}
$$

发送符号 $a_i$ 并接收到符号 $b_j$ 的概率为 $p_iP_{ij}$。如果接收到 $b_j$，那么一定有且仅有一个符号 $a_i$ 被发送。因此，我们得到信道关系：

$$
\sum_{i=1}^{r}p_iP_{ij}=q_j
\qquad
j=1,\ldots,s
\tag{4.2}
$$

如果我们将 $(p_i)$ 看作向量 $\mathbf p\in\mathbb{R}^r$，并将 $(q_j)$ 看作向量 $\mathbf q\in\mathbb{R}^s$，那么式（4.2）可以写成：

$$
\mathbf pM=\mathbf q
\tag{4.2'}
$$

如果我们对式（4.2）中的所有 $j$ 求和，然后交换求和顺序，并利用 $\sum_jq_j=1$，就可以得到式（4.1）。

---

除了前向概率 $P_{ij}$ 之外，引入**后向概率（backward probabilities）**也是有用的：

$$
Q_{ij}
=
\Pr(a=a_i\mid b=b_j)
=
\Pr(a_i\mid b_j)
$$

以及**联合概率（joint probabilities）**：

$$
R_{ij}
=
\Pr(a=a_i\text{ and }b=b_j)
=
\Pr(a_i,b_j)
$$

可以将前向概率 $P_{ij}$ 看作发送者的视角：发送者知道输入符号 $a_i$，并试图预测最终产生的输出符号 $b_j$。类似地，后向概率 $Q_{ij}$ 表示接收者的视角：接收者知道输出符号 $b_j$，并试图推测对应的输入符号 $a_i$。而联合概率（joint probabilities）$R_{ij}$ 则表示一个外部观察者的视角：该观察者试图同时推测 $a_i$ 和 $b_j$。

对于每一个 $i$ 和 $j$，都有：

$$
p_iP_{ij}
=
\Pr(a_i)\Pr(b_j\mid a_i)
=
\Pr(a_i,b_j)
=
\Pr(b_j)\Pr(a_i\mid b_j)
=
q_jQ_{ij}
$$

这些量都等于 $R_{ij}$。

因此得到 **贝叶斯公式（Bayes' Formula）**：

$$
Q_{ij}
=
\frac{p_i}{q_j}P_{ij}
\tag{4.3}
$$

将该公式与式（4.2）结合，可以得到：

$$
Q_{ij}
=
\frac{p_iP_{ij}}
{\sum_{k=1}^{r}p_kP_{kj}}
\tag{4.4}
$$

下一节中，我们将考虑这些公式的一些具体例子。

## 4.2 The Binary Symmetric Channel