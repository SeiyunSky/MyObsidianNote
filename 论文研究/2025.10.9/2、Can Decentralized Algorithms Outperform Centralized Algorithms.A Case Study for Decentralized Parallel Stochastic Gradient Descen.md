**算法核心目标**：
目标是获得最小全局损失函数——所有节点本地损失函数的均值
对于本地节点，需要从数据集的采样数据点、本地总期望损失函数最小
$$\min_{x \in \mathbb{R}^{d}} F(x) \quad \text{where} \quad F(x) := \frac{1}{n} \sum_{i=1}^{n} F_{i}(x)$$
$$F_i(x) := \mathbb{E}_{\zeta \sim \mathcal{D}_i} [f_i(x; \zeta)]$$
理想条件下是得到真实的总梯度——但分布式节点无法获得
$$\nabla F(x) = \frac{1}{n} \sum_{i=1}^{n} \nabla F_i(x)$$
因此求近似结果——获得本地真实梯度的无偏估计
其中$g_k$是计算得出的随机梯度
$$\mathbb{E}[g_{k,i} | x_{k,i}] = \nabla F_i(x_{k,i})$$
进行参数模型交换、聚合与模型更新
$$x_{k+\frac{1}{2}, i} = \sum_{j=1}^{n} W_{ij} x_{k,j}$$
$$x_{k+1, i} \leftarrow x_{k+\frac{1}{2}, i} - \gamma g_{k,i}$$
![[Pasted image 20251003103621.png|500]]
先交换聚合共识，再本地更新梯度 —— 强前提解决客户端偏移

**算法收敛性分析**
核心假设——
**L-利普希茨连续梯度**
$$∥∇f(x)−∇f(y)∥≤L∥x−y∥$$
  引理——L-光滑+二次上界

**方差有界**
$$\mathbb{E}{i \sim U([n])} \mathbb{E}{\xi \sim \mathcal{D}_i} ||\nabla F_i(x; \xi) - \nabla f(x)||^2$$
  总体来说，本地节点更新梯度方差与真实梯度方差有限
$$\mathbb{E}_{\xi \sim \mathcal{D}_i} ||\nabla F_i(x; \xi) - \nabla f_i(x)||^2 \leq \sigma^2, \quad \forall i, \forall x
$$
  全局梯度差异性有界，量化数据分布的异构性
$$
\mathbb{E}_{i \sim U([n])} ||\nabla f_i(x) - \nabla f(x)||^2 \leq \zeta^2, \quad \forall x$$
**谱隙**
$$ \rho := \left( \max\{ |\lambda_2(\mathbf{W})|, |\lambda_n(\mathbf{W})| \} \right)^2 $$  
$$ \rho = \beta^2 = (\max\{ |\lambda_2(\mathbf{W})|, |\lambda_n(\mathbf{W})| \})^2 $$
**从0开始** x_0 = 0

**定理一**
收敛速率
![[Pasted image 20251003112429.png|600]]
第一项是本地真实梯度均值的期望范数平方的均值
第二项是全局平均模型梯度的期望范数平方的均值

第一项是优化误差项——从初始点到最优点的距离
第二项是随机梯度方差项
第三项是聚合时网络拓扑结构的影响
第四项是数据异构性的影响

运用的引理：
$\left| \frac{\mathbf{1}_n}{n} - W^k e_i \right|^2 \leq \rho^k, \quad \forall i \in {1, 2, \dots, n}, k \in \mathbb{N}$
$\mathbb{E}|\partial f(X_j)|^2 \leq 3\mathbb{E}L^2 \sum_{h=1}^n \left| \frac{\sum_{i=1}^n x_{j,i}'}{n} - x_{j,h}' \right|^2 + 3n\zeta^2 + 3\mathbb{E}\left|\nabla f\left(\frac{X_j\mathbf{1}_n}{n}\right)\right|^2, \quad \forall j.$

首先——从整体损失函数的期望开始
#p23-7
![[Pasted image 20251003113630.png]]
并且将我们的**聚合规则**带入其中
![[Pasted image 20251003113659.png|200]]
其中有$\partial f(X_k) := [ \nabla f_1(x_{k,1}) \quad \nabla f_2(x_{k,2}) \quad \cdots \quad \nabla f_n(x_{k,n}) ] \in \mathbb{R}^{d \times n}$
得到该公式
![[Pasted image 20251003113753.png]]
【**随机梯度方差项**】
对于尾项项，**添加零项**，**并展开平方项**$||a+b||^2 = ||a||^+||b||^2+2<a,b>$
得到![[Pasted image 20251003135348.png|400]]
对于内积的因子，基于期望的迭代定律和无偏性假设，可以得到内积的第一项是0，因此可以得到
![[Pasted image 20251003135512.png|300]]
对于上面的第一个部分，**提出常数项，展开平方项**，**内积消除**
根据方差的有界性，可以得到其范围，**该常数项说明随着总结点的数量变大，梯度方差会降低**
![[Pasted image 20251003135802.png]]

**利用范数展开平方项公式展开负值内积**$2⟨a,b⟩=∥a∥^2+∥b∥^2−∥a−b∥^2$，得到
![[Pasted image 20251003140445.png|500]]
对于T1—用**Jensen 不等式**提出其中的累加项，将全局梯度表示为局部梯度累加和的均值
![[Pasted image 20251003140942.png|300]]
再通过**L-利普希茨连续梯度**得到
![[Pasted image 20251003141105.png|300]]
其中的$Q_{k,i}$可以反写为**特征矩阵相减**的形式，权重矩阵直接使用了双随机矩阵，再放入通用的聚合规则$x_{k+1, i} \leftarrow x_{k+\frac{1}{2}, i} - \gamma g_{k,i}$替换，根据X0为0的准则，经过**消项**和**引入零项进行方差偏差求解**，依照
$∥a+b∥^2≤2∥a∥^2+2∥b∥^2$，得到：
![[Pasted image 20251003141613.png|350]]
对于T2，根据**方差无偏性**和**谱隙**就能得到![[Pasted image 20251003142016.png|100]]
对于T3，经过拆分可以得到
![[Pasted image 20251003142150.png|400]]
最终可以得到
![[Pasted image 20251003142232.png|400]]


**定理四**
共识速率
![[Pasted image 20251003112519.png]]
**不展开阐述**