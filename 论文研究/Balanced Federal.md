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

### DFL方法
**UBAR**——Byzantine-resilient decentralized stochastic gradient descent
- **Distance-based Filtering**，首先计算所有邻居的距离（欧氏距离），选出对应的最近邻
 
- **Loss-based Filtering**，只有当邻居的表现**优于或等于**自己时，才认为它是善意的。
![[Pasted image 20250909200506.png]]
**聚合公式**
![[Pasted image 20250909200542.png]]
这个的缺点特别明显**稀疏性**让它的**统计假设**失效，**异构性**让它的**相似性假设**和**有益性假设**双双失效。

**LEARN**——Collaborative learning in the jungle
 - 进行多次梯度交换与聚合，客户端计算基于自己本地数据的梯度，将梯度发送给邻居，收到邻居梯度后，使用鲁棒的聚合规则来聚合，得到共识梯度，以此来更新本地模型
 ![[Pasted image 20250909201310.png]]
 - 每个客户段根据共识阶段结束后的模型发给邻居，收到邻居的模型后，再去进行新一轮的聚合，获得最终的模型，
 方法问题也很明显，需要网络是完全连通图，违背了稀疏连接设定。

**SCCLIP**——Byzantine-robust decentralized learning via self-centered clipping
- 核心在于Clipping，以自我为中心进行建材，获得了邻居的模型后，先生成一个安全版的模型，
![[Pasted image 20250909201655.png]]
![[Pasted image 20250909202131.png]]
- 最终进行聚合
![[Pasted image 20250909201847.png]]
问题在于SCCLIP只看大小不看方向，所以对于幅度正常但方向错误的攻击无能为力
但我感觉没问题啊，考虑到用分布式无中心且节点稀疏的情况下进行联邦学习的实际场景，当我们采取这种方式去训练的时，大概率是会用在数据异构性较强的业务里，那么SCCLIP能够很好的解决异构性的问题啊
 



