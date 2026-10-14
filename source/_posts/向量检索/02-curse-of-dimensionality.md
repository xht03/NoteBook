---
title: VS 02 Curse of Dimensionality
date: 2026-10-13 20:01:33
tags:
- note
categories:
- Vector Search
---

## 1 Introduction

01 篇里我们建立了各种距离，也看到它们在归一化后可以互相转化。但那里藏着一个尚未检验的前提：**这些距离在高维空间里还“有用”吗？**

向量检索面对的都是几百到几千维的向量，而维度灾难（curse of dimensionality）说的正是：**随着维度升高，空间会变得反直觉，距离会失去区分能力，所有点看起来差不多远**。如果真是这样，那“找最近邻”在高维下就没有意义了——大家一样近，选谁当最近邻都一样。

此篇要做两件事：

1. 用数学的推导把这个现象讲清楚，尤其是“距离集中”到底以什么速度发生；
2. 指出它**没有那么可怕**，因为真实数据通常有低内蕴维数，这才是近似检索仍然可行的根本原因。

---

## 2 Intuition

先建立两个直觉：

1. **体积往表面跑。** 半径 $R$ 的球，维度一高，体积全挤在贴着表面的薄壳。
2. **随机向量几乎正交。** 随便抓两个高维随机向量，它们的夹角几乎总是接近 $90^\circ$。这是前两个画面的必然结果——既然点都挤在球壳上，叉积（内积）就几乎抵消掉了。

下面我们逐一从数学上展开。

---

### 2.1 Volume Concentrates Near the Surface

$d$ 维空间里半径 $R$ 的球，体积正比于 $R^d$：

$$
V_d(R)=V_d(1)\,R^d
$$

其中 $V_d(1)=\dfrac{\pi^{d/2}}{\Gamma\!\left(\frac{d}{2}+1\right)}$ 是单位球体积，$\Gamma$ 是伽马函数，它是个不随 $R$ 变化的常数。

现在看**薄壳**：半径在 $(1-\epsilon)R$ 到 $R$ 之间的那层壳。它的体积占整个球的比例是

$$
\begin{aligned}
\frac{V_d(R)-V_d((1-\epsilon)R)}{V_d(R)}
&=\frac{V_d(1)R^d-V_d(1)\big((1-\epsilon)R\big)^d}{V_d(1)R^d} \\
&=1-\frac{(1-\epsilon)^d R^d}{R^d}\\
&=1-(1-\epsilon)^d
\end{aligned}
$$

对固定的 $\epsilon>0$，当 $d\to\infty$ 时

$$
(1-\epsilon)^d\to0
$$

**结论：任意薄壳，只要足够薄（$\epsilon$ 固定），维度一高就装下几乎全部体积。** 举个数字：取 $\epsilon=0.01$（1% 厚的壳），当 $d=1000$ 时 $(1-\epsilon)^d=0.99^{1000}\approx4\times10^{-5}$，壳几乎就是整个球。这就是"全是皮"的定量版本。

---

### 2.2 Random Vectors are Almost Orthogonal

取两个独立的随机向量 $x,y$。**假设各个方向上的坐标独立同分布，每个坐标均值为 0、方差为 $\sigma^2$**。内积

$$
\langle x,y\rangle=\sum_{i=1}^{d}x_i y_i
$$

是 $d$ 个随机项的和。我们先计算它的期望和方差。

**期望。** 由期望的线性性，且 $x_i$ 与 $y_i$ 相互独立：

$$
\begin{aligned}
\mathbb{E}\big[\langle x,y\rangle\big]
&=\mathbb{E}\Big[\sum_{i=1}^{d}x_iy_i\Big]=\sum_{i=1}^{d}\mathbb{E}[x_iy_i]\\
&=\sum_{i=1}^{d}\mathbb{E}[x_i]\,\mathbb{E}[y_i]\\
&=0\cdot0\\
&=0
\end{aligned}
$$

**方差。** 先算单个乘积项 $Z_i=x_iy_i$ 的方差，
用定义 $\operatorname{Var}[Z]=\mathbb{E}[Z^2]-\big(\mathbb{E}[Z]\big)^2$：

$$
\begin{aligned}
\operatorname{Var}[Z]
&=\mathbb{E}[Z^2]-\big(\mathbb{E}[Z]\big)^2\\
&=\mathbb{E}[x_i^2y_i^2]-\mathbb{E}[x_iy_i]^2\\
&=\mathbb{E}[x_i^2]\,\mathbb{E}[y_i^2]-0\\
&=\sigma^2\cdot\sigma^2\\
&=\sigma^4
\end{aligned}
$$

接下来要把 $d$ 个项的方差加起来。由于**各个方向上是独立同分布**：各项两两不相关。对不相关的随机变量，和的方差等于方差的和：

$$\operatorname{Var}\big[\langle x,y\rangle\big]=\operatorname{Var}\Big[\sum_{i=1}^{d}Z_i\Big]=\sum_{i=1}^{d}\operatorname{Var}[Z_i]=\sum_{i=1}^{d}\sigma^4=d\sigma^4.$$

所以**内积的标准差是 $\sqrt{d\sigma^4}=\sigma^2\sqrt d$**。

均值 0、标准差 $\sigma^2\sqrt d$，说明 $\langle x,y\rangle$ 在 0 附近做 $O(\sigma^2\sqrt d)$ 量级的涨落——它随 $d$ 以 $\sqrt d$ 增长。

**再看余弦。** 一个向量的长度平方是

$$
\|x\|_2^2=\sum_{i=1}^d x_i^2
$$

这也是 $d$ 个项的和，但每项非负、均值为 $\sigma^2$，按大数定律集中在 $d\sigma^2$ 附近，所以 $\|x\|_2\approx\sigma\sqrt d$。两条长度相乘约是 $\sigma\sqrt d\cdot\sigma\sqrt d=\sigma^2 d$。于是

$$\cos(x,y)=\frac{\langle x,y\rangle}{\|x\|_2\|y\|_2}=O\!\left(\frac{\sigma^2\sqrt d}{\sigma^2 d}\right)=O\!\left(\frac{1}{\sqrt d}\right)\xrightarrow{\ d\to\infty\ }0.$$


**任意两个随机向量几乎垂直。**

---

## 3 Distance Concentration

**距离集中**才是对向量检索有致命的影响。

考虑两个独立取自同一分布的点 $x$ 与 $q$（例如一个数据点和一个查询），**每个坐标独立同分布、均值为 0、方差为 $\sigma^2$**。我们算一算它们的距离会有多大波动。

$$\|x-q\|_2^2=\|x\|_2^2+\|q\|_2^2-2\langle x,q\rangle.$$

逐项分析右边。

第一项 $\|x\|_2^2$ 是 $d$ 个独立同分布变量的和，每个 $x_i^2$ 的期望是 $\sigma^2$，方差是个有限常数，记为 $\tau^2$。由独立性、和的方差可加，标准差是 $\tau\sqrt d$，所以相对波动是 $\tau\sqrt d/(d\sigma^2)=O(1/\sqrt d)$。

$$
\|x\|_2^2=\sum_{i=1}^{d}x_i^2=d\sigma^2\Big(1+O\big(\tfrac{1}{\sqrt d}\big)\Big)
$$

第二项 $\|q\|_2^2$ 与第一项同理，也集中在 $d\sigma^2$ 附近。

$$
\|q\|_2^2=\sum_{i=1}^{d} q_i^2=d\sigma^2\Big(1+O\big(\tfrac{1}{\sqrt d}\big)\Big)
$$

第三项 $\langle x,q\rangle=\sum_i x_i q_i$ 关于 $x$ 的均值为 0，方差是 $\sum_i q_i^2\sigma^2=\sigma^2\|q\|_2^2$，所以

$$\langle x,q\rangle=O\big(\sigma\|q\|_2\big)=O\big(\sigma\cdot\sigma\sqrt d\big)=O\big(\sigma^2\sqrt d\big)$$

现在把三项代回去。主导项是 $d\sigma^2$ 和 $\|q\|_2^2\approx d\sigma^2$，加起来 $\approx2d\sigma^2$；而第三项只有 $O(\sigma^2\sqrt d)$，比主导项小了一个 $1/\sqrt d$ 的因子：

$$
\begin{aligned}
\|x-q\|_2^2
&=2d\sigma^2\Big(1+O\big(\tfrac{1}{\sqrt d}\big)\Big)-2O\big(\sigma^2\sqrt d\big)\\
&=2d\sigma^2\Big(1+O\big(\tfrac{1}{\sqrt d}\big)-O\big(\tfrac{1}{\sqrt d}\big)\big)\\
&=2d\sigma^2\Big(1+O\big(\tfrac{1}{\sqrt d}\big)\big)\\
\end{aligned}
$$

开根号，得到距离本身：

$$\|x-q\|_2\approx\sigma\sqrt{2d}\Big(1+O\big(\tfrac{1}{\sqrt d}\big)\Big).$$

**这就是距离集中。** 任意一个点到查询的距离都集中在 $\sigma\sqrt{2d}$ 附近，相对波动只有 $O(1/\sqrt d)$。

把 $n$ 个点里最近和最远的拿出来比（注意取极值会多一个 $\sqrt{\log n}$ 因子，这是极值理论的结果）：

$$\frac{\|x_{\max}-q\|_2-\|x_{\min}-q\|_2}{\|x_{\min}-q\|_2}=O\!\left(\frac{\sqrt{\log n}}{\sqrt d}\right)\xrightarrow{\ d\to\infty\ }0.$$

最近与最远几乎一样远。维数越高，“最近邻”和“随便挑一个”就差得越少。

---

## 4 Intrinsic Dimensionality

如果到此为止，结论很悲观。但以上结论都基于一个假设：**所有向量在各个方向上是独立同分布的**。

但真实数据不是独立同分布的随机噪声，**它通常落在一个低维结构上——图像、文本的嵌入往往集中在一张低维流形附近**。这个“数据实际变化所张成的方向数”叫**内蕴维数（intrinsic dimensionality）**，记作 $d_{\text{int}}$。

但真实数据的 $d_{\text{int}}$ 不是一个能精确测量的整数，因为数据只是“大致”低维，每个方向上都有点残余变化。所以实践中测量出来的是一个实数，叫**有效维数**，记作 $d_{\text{eff}}$。简言之，内蕴维数是“真实值”，而有效维数是“测量值”。

最常见的测量有效维数的办法是：看**协方差谱**。算出数据协方差矩阵的特征值 $\lambda_1\ge\cdots\ge\lambda_d$，它们代表各个方向上的“变化量”。如果数据真在低维子空间里，就会看到几个大特征值、后面一长串接近 0。把这串谱压成一个数，我们将之视为有效维数：

$$d_{\text{eff}}\approx\frac{\big(\sum_i\lambda_i\big)^2}{\sum_i\lambda_i^2}.$$

特征值全相等时 $d_{\text{eff}}$ 等于 d，只有一个非零时 $d_{\text{eff}}$ 等于 1。

---

> 在探究一个问题时，我们往往先从简单情形入手，然后再分析背后原因。这也是为什么在第 3 节先假设“各方向上独立同分布”的原因——数据在所有方向上强度相同、彼此无结构，这是“最分散、最没有可利用规律”的情形。也正因如此，它让距离集中严重（它对应有效维数取到最大值 $d$，最坏情形）。


既然真实数据不是在各方向上独立同分布的，那么距离集中的“灾难”是否还存在呢？

进一步问：**距离集中现象的根本原因是什么呢？**

事实上，集中现象的核心是大数定律：若有 $k$ 个独立项求和，相对波动约 $1/\sqrt k$。所以“维度灾难有多严重”这个问题，本质就是“你到底在加多少个独立项”。

独立同分布时独立项个数 k 是 $d$；一般情况下它不是 $d$，而是 $d_{\text{eff}}$。下面我们从数学上展示这一点。

---

设数据协方差矩阵 $\Sigma$ 的特征值为 $\lambda_1\ge\cdots\ge\lambda_d$，对应一组标准正交特征向量 $u_1,\dots,u_d$。把任意点 $x$ 在这组基上展开：

$$
x=\sum_{j=1}^{d}z_j u_j,\qquad z_j=\langle x,u_j\rangle
$$

这里每个 $z_j$ 的方差正是 $\lambda_j$（因为 $\operatorname{Var}(z_j)=u_j^{\top}\Sigma u_j=\lambda_j$），且不同 $z_j$ 互不相关。于是

$$\|x\|_2^2=\sum_{j=1}^{d}z_j^2$$

**期望。** 由期望线性性，且不妨设 $z_j$ 均值皆为 0（平移坐标系不改变向量间的相对位置），于是

$$
\mathbb{E}\|x\|_2^2=\sum_{j=1}^{d}\mathbb{E}[z_j^2]=\sum_{j=1}^{d}\lambda_j
$$

**方差。** 各项不相关，则方差可加。单个项 $z_j^2$ 的方差是（以高斯为例，$\mathbb{E}[z_j^4]=3\lambda_j^2$）

$$\operatorname{Var}(z_j^2)=\mathbb{E}[z_j^4]-\big(\mathbb{E}[z_j^2]\big)^2=3\lambda_j^2-\lambda_j^2=2\lambda_j^2,$$

所以

$$\operatorname{Var}\big(\|x\|_2^2\big)=\sum_{j=1}^{d}2\lambda_j^2=2\sum_{j=1}^{d}\lambda_j^2.$$

于是

$$\text{相对波动}=\frac{\sqrt{2\sum_j\lambda_j^2}}{\sum_j\lambda_j}.$$

现在把 $d_{\text{eff}}=\dfrac{(\sum_j\lambda_j)^2}{\sum_j\lambda_j^2}$ 代进去

$$\text{相对波动}=\frac{\sqrt{2\sum_j\lambda_j^2}}{\sum_j\lambda_j}=\frac{\sqrt{2}\cdot\dfrac{\sum_j\lambda_j}{\sqrt{d_{\text{eff}}}}}{\sum_j\lambda_j}=\sqrt{\frac{2}{d_{\text{eff}}}}.$$

关键就是这里：**相对波动只由 $d_{\text{eff}}$ 决定，$\lambda_j$ 具体怎么分布已经消失了**。

## Afterword

**真实的图像、文本数据的嵌入集中在低维流形附近，那为什么要用高维的向量空间？**

这是个很好的问题，打个比方你就明白了：

- 一张纸内在是二维的，可你要把它揉皱了塞进空间里、还不让它自己穿透自己，就需要三维的空间给它皱的余地。
- 地图是二维的，但要把整张地图无撕裂地贴到一个球面上，就得用三维。低维结构想“弯”，必须借助一个更高维的环境。

数学上这就更清楚了。一个 $k$ 维的弯曲流形，无法无损地压进 $k$ 维欧氏空间。拿最小的例子：圆是 1 维流形，你把它映到一条直线（1 维）上，为了让它不重叠，就必须把圆“剪开”，而剪开后原本相邻的点距离就错了——所以圆至少要放进 2 维平面。

Whitney 嵌入定理说任意 $k$ 维光滑流形可以嵌进 $\mathbb{R}^{2k}$（注意是 $2k$，已经比 $k$ 高一倍）；如果还要求保持距离（等距嵌入），Nash 嵌入定理给出的维数还要高得多。**所以“用一个比内蕴维数高的空间”是数学上的要求，不是工程上的偷懒。**