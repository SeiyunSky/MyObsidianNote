**定理1**（非正式表述）。如果f(x)是平滑、凸的，且对于$(\alpha<1 / 2)$，α比例的机器是拜占庭式的，那么ByzantineSGD会在T次迭代中找到一个点x，其中$(f(x)-f(x^{*}) ≤\varepsilon)$满足z$$(T=\tilde{O}(\frac{1}{\varepsilon}+\frac{1}{\varepsilon^{2} m}+\frac{\alpha^{2}}{\varepsilon^{2}})) $$  如果f(x)满足σ-强凸，则有
$$(T=\tilde{O}(\frac{1}{\sigma}+\frac{1}{\sigma \varepsilon m}+\frac{\alpha^{2}}{\sigma \varepsilon})) $$

**定理2**（非正式表述）。在存在α比例的拜占庭机器的情况下，如果某个算法通过每台机器从D中获取T个样本，找到一个满足$(f(x)-f(x^{*}) ≤\varepsilon)$的点x，那么 $$T=\Omega\left(\frac{1}{\varepsilon^{2} m}+\frac{\alpha^{2}}{\varepsilon^{2}}\right) \quad$$
如果 f(x)是$\sigma$-strongly强凸
$$T=\Omega\left(\frac{1}{\sigma \varepsilon m}+\frac{\alpha^{2}}{\sigma \varepsilon}\right)$$
### 2 预备知识
在本文中，使用 $\|\cdot\|$ 表示欧几里得范数，并定义 $[n] \stackrel{\text{def}}{=} \{1,2,\ldots,n\}$。回顾一下强凸性、光滑性和利普希茨连续性的定义。
**定义 2.1.** 对于一个可微函数 $f\colon \mathbb{R}^{d} \rightarrow \mathbb{R}$：

*   $f$ 是 $\sigma$-强凸的，如果 $\forall x,y \in \mathbb{R}^{d}$，满足 $f(y) \geq f(x) + \langle \nabla f(x), y-x \rangle + \frac{\sigma}{2} \|x-y\|^{2}$。

*   $f$ 是 $L$-利普希茨光滑的（简称 $L$-光滑），如果 $\forall x,y \in \mathbb{R}^{d}$，有 $\|\nabla f(x) - \nabla f(y)\| \leq L \|x-y\|$。

*   $f$ 是 $G$-利普希茨连续的，如果 $\forall x \in \mathbb{R}^{d}$，有 $\|\nabla f(x)\| \leq G$。

### 2.1 拜占庭凸随机优化
令 $m$ 表示工作机器的数量，并假设其中至多有 $\alpha$ 比例的机器是拜占庭式的（$\alpha \in [0, \frac{1}{2})$）。用 $\mathcal{G} \subseteq [m]$ 表示好机器（即非拜占庭机器）的集合，算法事先并不知道 $\mathcal{G}$。
令 $\mathcal{D}$ 是定义在（不一定凸的）函数 $f_s: \mathbb{R}^{d} \rightarrow \mathbb{R}$ 上的一个分布。我们的目标是近似最小化以下目标函数：

$$\min_{x \in \mathbb{R}^{d}} \left\{ f(x) \stackrel{\text{def}}{=} \mathbb{E}_{s \sim \mathcal{D}} [f_s(x)] \right\}, \qquad (2.1)$$

其中假设 $f$ 是凸函数。在每次迭代 $k = 1, 2, \ldots, T$ 中，算法指定一个点 $x_k$ 并查询 $m$ 台机器。每台机器 $i \in [m]$ 返回一个向量 $\nabla_{k,i} \in \mathbb{R}^{d}$，满足以下假设：

**假设 2.2.** 对于每次迭代 $k \in [T]$ 和每一个好机器 $i \in \mathcal{G}$，有：
**无偏性：机器 `i`返回的梯度，必须等于在某个随机样本 `s`上计算出的梯度。
*   $\nabla_{k,i} = \nabla f_s(x_k)$，其中样本 $s$ 随机取自分布 $\mathcal{D}$

**有界性：机器返回的梯度（基于随机样本 `s`的梯度）与真实的“全数据”梯度之间的差距，不能超过一个界限 `V`。
*   $\|\nabla_{k,i} - \nabla f(x_k)\| \leq \mathcal{V}$。

**备注 2.3.** 对于每个 $k \in [T]$ 和 $i \notin \mathcal{G}$（即非好机器），向量 $\nabla_{k,i}$ 可以是对抗性选择的，并且可能依赖于历史梯度 $\{\nabla_{k', i}\}_{k' < k, i \in [m]}$。特别地，拜占庭机器甚至可以在单次迭代中合谋。

