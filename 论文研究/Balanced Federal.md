**核心公式**：
- 模型相似性检查公式
![[Pasted image 20250909110045.png]]
- 模型聚合公式
![[Pasted image 20250909110147.png]]

### 文中提到的FL一些聚合和架构方法
**Krum** —— Peva Blanchard, El Mahdi El Mhamdi, Rachid Guerraoui, and Julien Stainer.2017. Machine learning with adversaries: Byzantine tolerant gradient descent. In NeurIPS.
![[Pasted image 20250909110546.png]]
K是与wi最近的n-f-2个模型集合，根据评分得到对应比例，得到综合距离最小的模型集合，用这些最可信的模型进行更新。
拜占庭容忍能力——n＞2f+1

**FLTrust**——Xiaoyu Cao, Minghong Fang, Jia Liu, and Neil Zhenqiang Gong. 2021. Fltrust:Byzantine-robust federated learning via trust bootstrapping. In NDSS.
![[Pasted image 20250909132154.png]]![[Pasted image 20250909132201.png]]
权重计算公式
![[Pasted image 20250909132217.png]]
需要有一个绝对可信的中心权威作为尺度，由方向确定超参数，大小关系影响信任的基数

**Geometric Median**——几何中位数，寻找到高维空间中和其他点距离之和最小的点。

#### 经验性防御
**BayBFed**——BayBFed: Bayesian Backdoor Defense for Federated Learning
通过贝叶斯，去判定提交是否符合可能的预测概率
在数据异构性较高(Non-IID)的情况下，就会导致性能较低，且存在冷启动问题。

**Every Vote Counts**——Every Vote Counts: Ranking-Based Training of Federated Learning to Resist Poisoning Attacks
![[Pasted image 20250909192031.png]]
问题是会保留拜占庭节点的影响，进而导致长期更新后偏离问题

#### 非经验防御
**CRFL**——CRFL: Certifiably Robust Federated Learning against Backdoor Attacks
通过随机化平滑，降低性能提升安全性
![[Pasted image 20250909192953.png]]
其中噪声分布是
![[Pasted image 20250909193008.png]]
加噪聚合：在将t时刻所有的模型更新聚合求平均后，添加上高斯噪音向量
![[Pasted image 20250909193101.png]]

**Coordinate-wise Median** |**Trimmed Mean**——Byzantine-Robust Distributed Learning: Towards Optimal Statistical Rates
取中位数集合，将所有维度都如此操作，进行更新
![[Pasted image 20250909194401.png]]
**裁剪均值**则通过删除极值来进行排序取均值，需要取β超参数进行判断
![[Pasted image 20250909194446.png]]
裁剪均值和Krum的区别在于 Krum将客户端提交的整个更新向量作为不可分割的整体
计算向量之间的整体距离，而对于TrimmedMean方法，将每一个维度作为单独的对象进行处理

**FLDetector**——FLDetector: Defending federated learning against model poisoning attacks via detecting malicious clients
记录单轮贡献分与时间序列特征向量
![[Pasted image 20250909195056.png]]![[Pasted image 20250909195104.png]]
训练侦探模型进行二分类
对于该框架性方法来说，核心目标并非判定收敛性，而是去确定优化目标
![[Pasted image 20250909195447.png]]

### 文中的DFL方法
**UBAR**——Byzantine-resilient decentralized stochastic gradient descent
- **Distance-based Filtering**，首先计算所有邻居的距离（欧氏距离），选出对应的最近邻
- **Loss-based Filtering**，只有当邻居的表现**优于或等于**自己时，才认为它是善意的。
![[Pasted image 20250909200506.png]]
**聚合公式**
![[Pasted image 20250909200542.png]]
算法的目标是​**​ resilient aggregation​**​（弹性聚合），即尽管存在拜占庭节点，仍能输出一个聚合结果。但它​**​无法保证​**​这个聚合结果指向全局损失函数的下降方向。它只能保证诚实节点的参数向量最终相同，但这个相同的向量可能位于一个​**​糟糕的局部最优点​**​甚至​**​鞍点​**​上，而不是真正的全局最优点。
确保了一致性和收敛，但无法保证学习性能的准确性。
需要有相对多的邻居的同时，还需要按照一定的比例去进行选择
这个的缺点特别明显**稀疏性**让它的**统计假设**失效，**异构性**让它的**相似性假设**和**有益性假设**双双失效。

**LEARN**——Collaborative learning in the jungle
 - 进行多次梯度交换与聚合，客户端计算基于自己本地数据的梯度，将梯度发送给邻居，收到邻居梯度后，使用鲁棒的聚合规则来聚合，得到共识梯度，以此来更新本地模型
 ![[Pasted image 20250909201310.png]]
 - 每个客户段根据共识阶段结束后的模型发给邻居，收到邻居的模型后，再去进行新一轮的聚合，获得最终的模型，
 - 每轮需要沟通$logt$次梯度，再进行`1`次模型交换，安全上以Trimmed Mean为例，通过多步的持续性的沟通，基于良性模型在大方向上的一致性实现安全性。
 方法问题也很明显，**需要网络是完全连通图**，违背了稀疏连接设定。

**SCCLIP**——Byzantine-robust decentralized learning via self-centered clipping
- 核心在于Clipping，以自我为中心进行建材，获得了邻居的模型后，先生成一个安全版的模型，
![[Pasted image 20250909201655.png]]
![[Pasted image 20250909202131.png]]
- 最终进行聚合
![[Pasted image 20250909201847.png]]
问题在于SCCLIP只看大小不看方向，所以对于幅度正常但方向错误的攻击无能为力
但我感觉没问题啊，考虑到用分布式无中心且节点稀疏的情况下进行联邦学习的实际场景，当我们采取这种方式去训练的时，大概率是会用在数据异构性较强的业务里，那么SCCLIP能够很好的解决异构性的问题啊——**可乐与告示牌**

### 本文攻击方法
客户端拥有高度非独立同分布的训练数据（例如，每个客户端只拥有三类数据），
客户端使用不同的鲁棒聚合规则来组合接收到的模型，
客户端从不同的初始模型开始，恶意和良性客户端之间的边有不同的比例，以及时变的通信图

## DFL
FL的目的是解决ERM问题——经验风险最小化
![[Pasted image 20250910200435.png]]
本文的网络拓扑是异构无向**无权**无自环的通信图

- **进行本地模型训练与交换** 
    $w_i^{(t+\frac{1}{2})}$是中间模型，作为训练后聚合前的中间状态
    在训练完毕后，客户端i从邻居j那里接收$w_j^{(t+\frac{1}{2})}$
- **本地模型聚合**
    获得最终在第t轮训练完毕的模型，作为下一轮训练的起点，α代表了一个置信学习度，根据α的大小来决定对于自我革新和对邻居客户端模型的更新权重。
    $$\mathbf{w}_{i}^{t+1}=\alpha\mathbf{w}_{i}^{t+\frac{1}{2}}+(1-\alpha)\mathrm{AGG}\left\{\mathbf{w}_{j}^{t+\frac{1}{2}}, j\in\mathcal{N}_{i}\right\}  \ \ \ \ \ \ \ \ \ \ \ \ \ (2)$$
    AGG是一个占位符，可以引入任何【鲁棒聚合规则】

**On the (In)security of Peer-to-Peer Decentralized Machine Learning**——$$\mathbf{w}_{i}^{t+1}=\operatorname{AGG}\left\{\mathbf{w}_{j}^{t+\frac{1}{2}}, j\in\mathcal{N}_{i}\text{ with }\mathcal{N}_{i}=\mathcal{N}_{i}\cup\{i\}\right\}$$
这篇文章的聚合方式是将自己也作为邻居的一部分，促进网络达成共识
在Krum的情况下，很可能会出现赢家通吃的问题，进而导致无法学习到邻居的参数
而如果使用Trimmed Mean，其本身的目的是为了处理一个单峰分布的数据，在高度异构的DFL网络中，多峰分布往往是更加常见的场景——忽略了对长尾节点的训练公平性。
使用几何中位数能够更合适——能够在理论上更加的提高鲁棒性，但闭式解很难找
$\mathbf{w}^* = \underset{\mathbf{w}}{\operatorname{argmin}} \sum_{i=1}^{n} \| \mathbf{w} - \mathbf{w}_i \|_2$

### 一些投毒方法
#### **数据投毒攻击**
- **Poisoning attacks against support vector machines** 针对支持向量机的投毒攻击
  ![[Pasted image 20250910205614.png]]
  双层优化包含了内层优化和外层优化，对于$X_c^*$，其核心在于掌握内层优化的训练目标，因为$x_c$影响的$θ^*$，而$L_{attack}$又依赖于前者。

- **Towards poisoning of deep learning algorithms with back-gradient optimization** 反向梯度优化进行深度算法攻击
  核心就是通过负样本来影响模型训练

- **Data poisoning attacks against federated learning systems** 针对联邦学习的数据投毒攻击
  核心思想也就是教学本地模型坏数据，造成模型学习样本污染

#### 模型投毒攻击
- **How to backdoor federated learning** 后门攻击联邦学习
  进行模型替换，在绝大部分输入上表现完全正常
- **A little is enough: Circumventing defenses for distributed learning**
  ![[Pasted image 20250910210716.png]]
  通过友好客户端分布情况，构造恶意更新数值，小范围内优化
- **Manipulating the Byzantine: Optimizing Model Poisoning Attacks and Defenses for Federated Learning**
  Adpat攻击
- **Dba: Distributed backdoor attacks against federated learning**
  利用分布式训练的特点，分布化植入触发器。


### **Balance Alorithm**
$$\left\| w_{i}^{t + \frac{1}{2}} - w_{j}^{t + \frac{1}{2}} \right\| \leq \gamma \cdot \exp(-\kappa \cdot \lambda(t)) \cdot \left\| w_{i}^{t + \frac{1}{2}} \right\| \quad \ \ \ \ \ \ \ \ (3)$$
首先，模型之间的绝对差异需要小于
`原客户端模型*容忍度*每轮次的动态衰减因子`
确保在学习后期的高度一致性
其中的主要超参数是  
$k$ 决定函数下降速率，用来控制容忍度收紧速度 根据网络内数据差异化情况去选择大小
γ 基础容忍度 设定在最宽松的情况下所能容忍的相对最大差异
$λ(t)$ 作为根据时间来缩紧的参数，往往设置为当前轮次同总轮次的比例关系

$$w_{i}^{t+1} = \alpha w_{i}^{t + \frac{1}{2}} + (1 - \alpha) \frac{1}{|S_{i}^{t}|} \sum_{j \in S_{i}^{t}} w_{j}^{t + \frac{1}{2}}$$
最后将通过Balance审查的邻居进行聚合

#### 基本假设
- 强凸性——$$F(w_1) + \langle \nabla F(w_1), w_2 - w_1 \rangle + \frac{\mu}{2} \| w_2 - w_1 \|^2 \leq F(w_2)$$
- L-SMOOTH——$$\|\nabla F(w_1) - \nabla F(w_2)\| \leq L \|w_1 - w_2\|$$
  其推论包含了![[Pasted image 20250911171430.png]]
  [# L-smooth函数相关推导：梯度Lipschitz连续与二次上界的等价性证明](obsidian://open?vault=MyObsidianNote-main&file=%E8%AE%BA%E6%96%87%E7%A0%94%E7%A9%B6%2FL-smooth%E5%87%BD%E6%95%B0%E7%9B%B8%E5%85%B3%E6%8E%A8%E5%AF%BC%EF%BC%9A%E6%A2%AF%E5%BA%A6Lipschitz%E8%BF%9E%E7%BB%AD%E4%B8%8E%E4%BA%8C%E6%AC%A1%E4%B8%8A%E7%95%8C%E7%9A%84%E7%AD%89%E4%BB%B7%E6%80%A7%E8%AF%81%E6%98%8E)

- Unbiased & Bounded Variance——
  训练方向均值最终等于实际梯度变化结果$\mathbb{E}[g(\mathbf{w}_i)] = \nabla F(\mathbf{w}_i)$   
  两者之间的差异有一个阈值$\mathbb{E}[\| g(\mathbf{w}_i) - \nabla F(\mathbf{w}_i) \|^2] \leq \sigma^2$
- 有界性——∀i∈B, we have ∥wi​∥≤ψ, and ∥∇F(wi​)∥≤ρ
- 不存在孤岛好人

### 两个基本定理
**定理一**——强凸
 左边——误差均值
 右边——指数衰减下的历史遗留的初始误差+随机梯度带来的抖动+拜占庭容忍误差
  $$\mathbb{E}\left[F\left(\mathbf{w}_{i}^{T}\right)-F\left(\mathbf{w}^{\star}\right)\right] \leq (1-\mu\eta)^{T}\left[F\left(\mathbf{w}_{i}^{0}\right)-F\left(\mathbf{w}^{\star}\right)\right] + \frac{2L\eta\delta^{2}}{\mu} + \frac{2\gamma\rho\psi(1-\alpha)}{\mu\eta}$$
其中$γ\leq\frac{\mu}{L·(1-a)}$  学习率范围也要在推算中保证学习过程的无偏性——基于和$\mu,\eta$的范围
  ![[Pasted image 20250911162643.png]]
  
  
  
  
  

