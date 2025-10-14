对于外卖送餐假设有如下数据：
$$time=2∗lights+0.01∗distance+5$$

|time|lights|distance|
|---|---|---|
|19|2|1000|
|31|3|2000|
|14|2|500|
|15|1|800|
|43|4|3000|
梯度下降代码：
```python
import torch

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
inputs = torch.tensor([[2,1000],[3,2000],[2,500],[1,800],[4,3000]])
labels = torch.tensor([[19],[31],[14],[15],[43]],dtype = torch.float,device = device)

w = torch.ones(2,1,requires_grad = True , device = device)
b = torch.onse(1,requires_grad = True,device = device)

epoch = 2000
lr = 0.001

for i in range(epoch)
  outputs = inputs @ w + b
  loss = torch.mean(torch.square(outputs - labels))
  loss.backward()
  
  with torch.no_grad()
    w -= w.grad * lr
    b -= b.grad * lr
    
w.grad.zero_()
b.grad.zero_()
```

然而如此会因为w1和w2终值差距太大，学习率影响导致训练非常慢。
因此需要所有feature的取值范围相同，对其进行归一化，以适用于统一的学习率
**对feature进行归一化**
```python
#根据比例差异进行输入数据归一化
inputs = inputs / torch.tensor([4, 3000], device=device)
```

**对特征进行标准化**
在深度学习里，更常用的是对特征进行标准化处理。
**对每个feature，减去自己的均值，再除以自己的标准差**。
这样就把这个feature转化为均值为0，标准差为1的分布了。
```python
#计算特征的均值和标准差
mean = inputs.mean(dim=0)
std = inputs.std(dim=0)
#对特征进行标准化
inputs_norm = (inputs-mean)/std
```
