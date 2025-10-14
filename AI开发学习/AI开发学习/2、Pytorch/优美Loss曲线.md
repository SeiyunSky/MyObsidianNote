### 安装TensorBoard库
1. 打开Anaconda Prompt。
2. 通过conda activate 切换到你的conda环境。
3. pip install tensorboard

```python
from torch.utils.tensorboard import SummaryWriter

writer = SummaryWriter(log_dir="C:/projects/lr/runs/")

#训练过程的每一步写入loss
writer.add_scalar("loss/train", loss.item(), i)
```

在Anaconda Prompt里激活环境运行命令
```bash
tensorboard --logdir=C:/projects/lr/runs/
```
