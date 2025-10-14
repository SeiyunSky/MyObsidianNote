### 生成训练数据
```python
import torch

# 确保CUDA可用
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# 生成数据
inputs = torch.rand(100, 3) # 随机生成shape为(100,3)的tensor，里边每个元素的值都是0-1之间
weights = torch.tensor([[1.1], [2.2], [3.3]]) #预设的权重
bias = torch.tensor(4.4) #预设的bias
targets = inputs @ weights + bias + 0.1*torch.randn(100, 1) #增加一些误差，模拟真实情况
```
### 初始化线性回归的参数
```python
# 初始化参数时直接放在CUDA上，并启用梯度追踪
w = torch.rand((3, 1), requires_grad=True, device=device)
b = torch.rand((1,), requires_grad=True, device=device)
```
### 进行训练
```python
# 将数据移至相同设备
inputs = inputs.to(device)
targets = targets.to(device)

#设置超参数
epoch = 10000
lr = 0.003

for i in range(epoch):
    outputs = inputs @ w + b
    loss = torch.mean(torch.square(outputs - targets))
    print("loss:", loss.item())

    loss.backward()

    with torch.no_grad(): #下边的计算不需要跟踪梯度
        w -= lr * w.grad
        b -= lr * b.grad

    # 清零梯度
    w.grad.zero_()
    b.grad.zero_()

print("训练后的权重 w:", w)
print("训练后的偏置 b:", b)
```
