### **CNN模型构建**
```python
# cnn_model.py
import torch.nn as nn
import torch.nn.functional as F

class CNN_MNIST(nn.Module):
    def __init__(self):
        super(CNN_MNIST, self).__init__()
        # 论文中的架构: Conv -> Pool -> Conv -> Pool -> FC -> Softmax
        self.conv1 = nn.Conv2d(1, 30, kernel_size=3) # 输入通道1, 输出通道30, 3x3卷积核
        self.conv2 = nn.Conv2d(30, 50, kernel_size=3) # 输入通道30, 输出通道50, 3x3卷积核
        
        # 尺寸计算:
        # 输入: 28x28
        # conv1后: (28-3+1) = 26x26
        # pool1后: 13x13
        # conv2后: (13-3+1) = 11x11
        # pool2后: 5x5 (torch.nn.MaxPool2d默认向下取整)
        # 展平后维度: 50 * 5 * 5 = 1250
        
        self.fc1 = nn.Linear(1250, 100) # 全连接层
        self.fc2 = nn.Linear(100, 10)  # 输出层, 10个类别

    def forward(self, x):
        x = F.relu(self.conv1(x))
        x = F.max_pool2d(x, 2)
        x = F.relu(self.conv2(x))
        x = F.max_pool2d(x, 2)
        x = x.view(-1, 1250) # 展平
        x = F.relu(self.fc1(x))
        x = self.fc2(x)
        return F.log_softmax(x, dim=1)
```
### 数据准备
```python
# data_loader.py
import torch
from torchvision import datasets, transforms
import numpy as np

def get_mnist_dataloaders(num_clients=20, non_iid_alpha=0.8, batch_size=32, malicious_clients_ids=None, attack_type='none'):
    """
    准备MNIST数据集并进行Non-IID划分
    """
    if malicious_clients_ids is None:
        malicious_clients_ids = []

    transform = transforms.Compose([
        transforms.ToTensor(),
        transforms.Normalize((0.1307,), (0.3081,))
    ])

    # 下载训练集和测试集
    train_dataset = datasets.MNIST('./data', train=True, download=True, transform=transform)
    test_dataset = datasets.MNIST('./data', train=False, transform=transform)

    # 按标签对训练数据进行分组
    labels = np.array(train_dataset.targets)
    label_indices = {i: np.where(labels == i)[0] for i in range(10)}

    # 使用Dirichlet分布为客户端分配数据
    client_indices = [[] for _ in range(num_clients)]
    for k in range(10):
        # 为每个标签的样本随机分配给客户端
        indices_k = label_indices[k]
        np.random.shuffle(indices_k)
        # Dirichlet分布决定每个客户端从该标签中获取多少样本
        proportions = np.random.dirichlet(np.repeat(non_iid_alpha, num_clients))
        # 按比例切分
        proportions = (np.cumsum(proportions) * len(indices_k)).astype(int)[:-1]
        client_indices = [
            all_indices + part.tolist()
            for all_indices, part in zip(client_indices, np.split(indices_k, proportions))
        ]

    # 创建每个客户端的DataLoader
    train_loaders = []
    for i in range(num_clients):
        client_data_indices = client_indices[i]
        
        # --- 实现标签翻转攻击 (LF Attack) ---
        if i in malicious_clients_ids and attack_type == 'label_flipping':
            # 复制一份数据以进行修改
            attacked_dataset = torch.utils.data.TensorDataset(
                train_dataset.data[client_data_indices].clone().float(),
                torch.tensor(train_dataset.targets)[client_data_indices].clone()
            )
            # 将标签从 c 翻转到 9-c
            attacked_dataset.tensors[1][:] = 9 - attacked_dataset.tensors[1][:]
            
            # 由于TensorDataset没有transform, 我们手动应用
            attacked_dataset.tensors[0] /= 255.0
            attacked_dataset.tensors[0] = (attacked_dataset.tensors[0] - 0.1307) / 0.3081
            
            # 添加通道维度
            attacked_dataset.tensors[0] = attacked_dataset.tensors[0].unsqueeze(1)
            
            loader = torch.utils.data.DataLoader(attacked_dataset, batch_size=batch_size, shuffle=True)
        else:
            sampler = torch.utils.data.SubsetRandomSampler(client_data_indices)
            loader = torch.utils.data.DataLoader(train_dataset, batch_size=batch_size, sampler=sampler)
        
        train_loaders.append(loader)

    # 全局测试集
    test_loader = torch.utils.data.DataLoader(test_dataset, batch_size=1000, shuffle=False)
    
    return train_loaders, test_loader
```

### 客户端定义
```python
# client.py
import torch
import torch.optim as optim
import torch.nn.functional as F
from collections import OrderedDict

# --- 辅助函数：在图像上添加后门触发器 ---
def add_backdoor_trigger(data, target_label=7):
    """在MNIST图像的右下角添加一个2x2的白色方块作为触发器"""
    triggered_data = data.clone()
    triggered_data[:, :, 26:28, 26:28] = triggered_data.max() # 设置为最大像素值 (白色)
    return triggered_data, torch.full((data.shape[0],), target_label, dtype=torch.long)


class Client:
    """良性客户端"""
    def __init__(self, client_id, model, train_loader, learning_rate=3e-4):
        self.client_id = client_id
        self.model = model
        self.train_loader = train_loader
        self.optimizer = optim.Adam(self.model.parameters(), lr=learning_rate)

    def local_train(self, **kwargs): # 使用kwargs以保持和MaliciousClient签名一致
        self.model.train()
        for epoch in range(1): # 简化为训练1个epoch
            for data, target in self.train_loader:
                self.optimizer.zero_grad()
                output = self.model(data)
                loss = F.nll_loss(output, target)
                loss.backward()
                self.optimizer.step()
        return self.model.state_dict()


class MaliciousClient(Client):
    """恶意客户端，执行各种模型投毒攻击"""
    def __init__(self, client_id, model, train_loader, learning_rate, attack_config):
        super().__init__(client_id, model, train_loader, learning_rate)
        self.attack_type = attack_config['attack_type']
        self.attack_config = attack_config

    def local_train(self, **kwargs):
        # 1. 首先，像良性客户端一样进行正常训练
        benign_state_dict = super().local_train()
        
        # 2. 根据攻击类型，对训练好的模型进行投毒
        poisoned_state_dict = OrderedDict()

        if self.attack_type == 'gaussian':
            scale = self.attack_config.get('gauss_scale', 0.1)
            for key, param in benign_state_dict.items():
                noise = torch.randn_like(param) * scale
                poisoned_state_dict[key] = param + noise
            return poisoned_state_dict

        elif self.attack_type == 'trim':
            scale = self.attack_config.get('trim_scale', -2.0)
            for key, param in benign_state_dict.items():
                poisoned_state_dict[key] = param * scale
            return poisoned_state_dict

        elif self.attack_type == 'backdoor':
            target_label = self.attack_config.get('backdoor_target_label', 7)
            self.model.train()
            for epoch in range(1):
                for data, target in self.train_loader:
                    half_batch = data.shape[0] // 2
                    if half_batch == 0: continue
                    
                    clean_data, clean_target = data[:half_batch], target[:half_batch]
                    triggered_data, triggered_target = add_backdoor_trigger(data[half_batch:], target_label)
                    
                    combined_data = torch.cat([clean_data, triggered_data], dim=0)
                    combined_target = torch.cat([clean_target, triggered_target], dim=0)
                    
                    self.optimizer.zero_grad()
                    output = self.model(combined_data)
                    loss = F.nll_loss(output, combined_target)
                    loss.backward()
                    self.optimizer.step()
            return self.model.state_dict()
        else: 
            return benign_state_dict
```

### 恶意客户端
```python
# client.py
import torch
import torch.optim as optim
import torch.nn.functional as F
from collections import OrderedDict

# --- 辅助函数：在图像上添加后门触发器 ---
def add_backdoor_trigger(data, target_label=7):
    """在MNIST图像的右下角添加一个2x2的白色方块作为触发器"""
    triggered_data = data.clone()
    # image is 28x28. Add trigger to bottom right corner.
    triggered_data[:, :, 26:28, 26:28] = triggered_data.max() # Set to max pixel value (white)
    return triggered_data, torch.full_like(data.mean(dim=(1,2,3)), target_label).long()


class Client:
    """良性客户端"""
    def __init__(self, client_id, model, train_loader, learning_rate=3e-4):
        self.client_id = client_id
        self.model = model
        self.train_loader = train_loader
        self.optimizer = optim.Adam(self.model.parameters(), lr=learning_rate)

    def local_train(self, **kwargs): # 使用kwargs以保持和MaliciousClient签名一致
        self.model.train()
        for epoch in range(1): # 简化为训练1个epoch
            for data, target in self.train_loader:
                self.optimizer.zero_grad()
                output = self.model(data)
                loss = F.nll_loss(output, target)
                loss.backward()
                self.optimizer.step()
        return self.model.state_dict()


class MaliciousClient(Client):
    """恶意客户端，执行各种模型投毒攻击"""
    def __init__(self, client_id, model, train_loader, learning_rate, attack_config):
        super().__init__(client_id, model, train_loader, learning_rate)
        self.attack_type = attack_config['attack_type']
        self.attack_config = attack_config

    def local_train(self, **kwargs):
        # 1. 首先，像良性客户端一样进行正常训练
        benign_state_dict = super().local_train()
        
        # 2. 根据攻击类型，对训练好的模型进行投毒
        poisoned_state_dict = OrderedDict()

        if self.attack_type == 'gaussian':
            scale = self.attack_config.get('gauss_scale', 0.1)
            for key, param in benign_state_dict.items():
                noise = torch.randn_like(param) * scale
                poisoned_state_dict[key] = param + noise
            return poisoned_state_dict

        elif self.attack_type == 'trim':
            # 这种攻击旨在发送一个与良性模型方向相反的模型
            scale = self.attack_config.get('trim_scale', -2.0)
            for key, param in benign_state_dict.items():
                poisoned_state_dict[key] = param * scale
            return poisoned_state_dict

        elif self.attack_type == 'backdoor':
            # 后门攻击需要特殊的训练过程
            target_label = self.attack_config.get('backdoor_target_label', 7)
            # 同时学习主任务和后门任务
            self.model.train()
            for epoch in range(1):
                for data, target in self.train_loader:
                    # 将一半数据注入后门
                    half_batch = data.shape[0] // 2
                    clean_data, clean_target = data[:half_batch], target[:half_batch]
                    triggered_data, triggered_target = add_backdoor_trigger(data[half_batch:], target_label)
                    
                    combined_data = torch.cat([clean_data, triggered_data], dim=0)
                    combined_target = torch.cat([clean_target, triggered_target], dim=0)
                    
                    self.optimizer.zero_grad()
                    output = self.model(combined_data)
                    loss = F.nll_loss(output, combined_target)
                    loss.backward()
                    self.optimizer.step()
            return self.model.state_dict()

        else: # 如果是数据投毒（如LF），则在数据加载时已处理，这里直接返回良性训练结果
            return benign_state_dict
```
### 主程序
```python
# main.py
import torch
import numpy as np
from collections import OrderedDict
import matplotlib.pyplot as plt

from cnn_model import CNN_MNIST
from data_loader import get_mnist_dataloaders
from client import Client, MaliciousClient # 导入两个类

# --- 辅助函数 ---
def get_model_norm(model_state_dict):
    norm = 0.0
    for param in model_state_dict.values():
        norm += torch.norm(param.float())**2
    return torch.sqrt(norm).item()

def get_model_dist(model1_state_dict, model2_state_dict):
    dist = 0.0
    for key in model1_state_dict:
        dist += torch.norm(model1_state_dict[key].float() - model2_state_dict[key].float())**2
    return torch.sqrt(dist).item()

# --- 评估函数 (新增后门评估) ---
def evaluate_model(model, test_loader, attack_config):
    model.eval()
    
    # 1. 评估主任务的测试错误率 (TER)
    correct = 0
    with torch.no_grad():
        for data, target in test_loader:
            output = model(data)
            pred = output.argmax(dim=1, keepdim=True)
            correct += pred.eq(target.view_as(pred)).sum().item()
    error_rate = 1.0 - (correct / len(test_loader.dataset))

    # 2. 评估后门攻击成功率 (ASR)
    attack_success_rate = 0.0
    if attack_config['attack_type'] == 'backdoor':
        target_label = attack_config.get('backdoor_target_label', 7)
        triggered_correct = 0
        total_triggered = 0
        with torch.no_grad():
            for data, target in test_loader:
                # 排除目标类别的样本，以避免误报
                non_target_mask = (target != target_label)
                if non_target_mask.sum() == 0:
                    continue
                
                non_target_data = data[non_target_mask]
                
                from client import add_backdoor_trigger # 导入触发器函数
                triggered_data, _ = add_backdoor_trigger(non_target_data, target_label)
                
                output = model(triggered_data)
                pred = output.argmax(dim=1, keepdim=True)
                
                triggered_correct += (pred.view(-1) == target_label).sum().item()
                total_triggered += len(triggered_data)
        
        attack_success_rate = triggered_correct / total_triggered if total_triggered > 0 else 0.0
        
    return error_rate, attack_success_rate

# --- 聚合算法 ---
def aggregate_fedavg(neighbor_models_states):
    avg_state_dict = OrderedDict()
    for key in neighbor_models_states[0]:
        avg_state_dict[key] = torch.stack([state[key] for state in neighbor_models_states]).mean(0)
    return avg_state_dict

def aggregate_balance(own_model_state, neighbor_models_states, gamma, kappa, lambda_t):
    accepted_models_states = []
    own_norm = get_model_norm(own_model_state)
    threshold = gamma * np.exp(-kappa * lambda_t) * own_norm
    for neighbor_state in neighbor_models_states:
        dist = get_model_dist(own_model_state, neighbor_state)
        if dist <= threshold:
            accepted_models_states.append(neighbor_state)
    if not accepted_models_states: return None
    return aggregate_fedavg(accepted_models_states)

# --- 主函数 ---
def run_experiment(config):
    print(f"--- Running: Algo={config['algorithm']}, Attack={config['attack_type']} ---")
    
    # 1. 准备数据
    malicious_ids = list(range(config['num_malicious'])) if config['attack_type'] != 'none' else []
    train_loaders, test_loader = get_mnist_dataloaders(
        num_clients=config['num_clients'],
        malicious_clients_ids=malicious_ids,
        attack_type='label_flipping' if config['attack_type'] == 'label_flipping' else 'none'
    )
    
    # 2. 创建通信图
    adj_matrix = np.zeros((config['num_clients'], config['num_clients']), dtype=int)
    for i in range(config['num_clients']):
        for j in range(1, config['num_neighbors'] // 2 + 1):
            adj_matrix[i, (i + j) % config['num_clients']] = 1
            adj_matrix[i, (i - j + config['num_clients']) % config['num_clients']] = 1

    # 3. 初始化客户端
    clients = []
    initial_model = CNN_MNIST()
    for i in range(config['num_clients']):
        client_model = CNN_MNIST()
        client_model.load_state_dict(initial_model.state_dict())
        if i in malicious_ids:
            client = MaliciousClient(i, client_model, train_loaders[i], config['learning_rate'], config)
        else:
            client = Client(i, client_model, train_loaders[i], config['learning_rate'])
        clients.append(client)
        
    # 4. 训练循环
    for t in range(config['rounds']):
        # a. 本地训练 (所有客户端先独立完成)
        local_models_states = {}
        for i in range(config['num_clients']):
            # 传递整个config，因为恶意客户端可能需要更多信息
            local_models_states[i] = clients[i].local_train(all_models_states=local_models_states, round=t, config=config)
            
        # b. 去中心化聚合
        next_round_models_states = {}
        for i in range(config['num_clients']):
            neighbor_ids = np.where(adj_matrix[i] == 1)[0]
            neighbor_models = [local_models_states[j] for j in neighbor_ids]
            
            if config['algorithm'] == 'FedAvg':
                aggregated_state = aggregate_fedavg(neighbor_models)
            elif config['algorithm'] == 'BALANCE':
                lambda_t_val = t / config['rounds']
                aggregated_state = aggregate_balance(
                    local_models_states[i], neighbor_models,
                    config['gamma'], config['kappa'], lambda_t_val
                )
            
            # c. 更新模型
            if aggregated_state:
                final_state = OrderedDict()
                alpha = config['alpha']
                for key in local_models_states[i]:
                    final_state[key] = alpha * local_models_states[i][key] + (1 - alpha) * aggregated_state[key]
                next_round_models_states[i] = final_state
            else: 
                next_round_models_states[i] = local_models_states[i]
        # 将更新后的模型加载回客户端
        for i in range(config['num_clients']):
            clients[i].model.load_state_dict(next_round_models_states[i])

        if (t + 1) % 200 == 0: # 减少打印频率
            print(f"  Round {t+1}/{config['rounds']} completed.")

    # 5. 评估
    benign_clients = [c for c in clients if c.client_id not in malicious_ids]
    error_rates = []
    attack_success_rates = []
    for client in benign_clients:
        error, asr = evaluate_model(client.model, test_loader, config)
        error_rates.append(error)
        attack_success_rates.append(asr)
        
    max_ter = max(error_rates) if error_rates else 0.0
    max_asr = max(attack_success_rates) if attack_success_rates and config['attack_type'] == 'backdoor' else 0.0
    
    print(f"Final Result: Max.TER = {max_ter:.4f}, Max.ASR = {max_asr:.4f}\n")
    return max_ter, max_asr


def visualize_results(results):
    """
    使用matplotlib将实验结果可视化为条形图
    """
    labels = list(results.keys())
    fedavg_ters = [res['FedAvg']['ter'] for res in results.values()]
    balance_ters = [res['BALANCE']['ter'] for res in results.values()]
    fedavg_asrs = [res['FedAvg']['asr'] for res in results.values()]
    balance_asrs = [res['BALANCE']['asr'] for res in results.values()]

    x = np.arange(len(labels))
    width = 0.35

    fig, ax = plt.subplots(figsize=(14, 7))
    
    # 绘制 TER 条形图
    rects1 = ax.bar(x - width/2, fedavg_ters, width, label='FedAvg (Max.TER)', color='salmon')
    rects2 = ax.bar(x + width/2, balance_ters, width, label='BALANCE (Max.TER)', color='skyblue')

    # 有后门攻击，绘制 ASR 条形图 (使用虚线边框)
    if any(asr > 0 for asr in fedavg_asrs + balance_asrs):
         ax.bar(x - width/2, fedavg_asrs, width, label='FedAvg (Max.ASR)', color='salmon', edgecolor='black', hatch='//')
         ax.bar(x + width/2, balance_asrs, width, label='BALANCE (Max.ASR)', color='skyblue', edgecolor='black', hatch='\\\\')


    ax.set_ylabel('Rate')
    ax.set_title('Comparison of DFL Methods under Various Attacks on MNIST')
    ax.set_xticks(x)
    ax.set_xticklabels(labels, rotation=45, ha="right")
    ax.legend()
    ax.grid(axis='y', linestyle='--', alpha=0.7)
    
    # 在条形图上显示数值
    def autolabel(rects):
        for rect in rects:
            height = rect.get_height()
            ax.annotate(f'{height:.2f}',
                        xy=(rect.get_x() + rect.get_width() / 2, height),
                        xytext=(0, 3),  # 3 points vertical offset
                        textcoords="offset points",
                        ha='center', va='bottom', fontsize=8)

    autolabel(rects1)
    autolabel(rects2)
    
    fig.tight_layout()
    plt.savefig("experiment_results.png")
    plt.show()


if __name__ == '__main__':
    torch.manual_seed(42)
    np.random.seed(42)

    # 论文中的超参数 (Section 6.1.5 and default values)
    PARAMS = {
        'num_clients': 20,
        'num_neighbors': 10,
        'num_malicious': 4,
        'rounds': 1000, # 减少轮次以便快速看到结果，2000轮会更准
        'learning_rate': 3e-4,
        'alpha': 0.5,
        'gamma': 0.3,
        'kappa': 1.0,
        # 攻击特定参数
        'gauss_scale': 0.5,
        'trim_scale': -2.0,
        'backdoor_target_label': 7
    }

    # 定义要运行的攻击场景
    attacks_to_run = {
        'No Attack': {'attack_type': 'none'},
        'Label Flipping': {'attack_type': 'label_flipping'},
        'Gaussian': {'attack_type': 'gaussian'},
        'Trim': {'attack_type': 'trim'},
        'Backdoor': {'attack_type': 'backdoor'}
    }
    
    # 存储所有实验结果
    all_results = OrderedDict()

    for attack_name, attack_params in attacks_to_run.items():
        all_results[attack_name] = {'FedAvg': {}, 'BALANCE': {}}
        
        # --- 运行 FedAvg ---
        fedavg_config = PARAMS.copy()
        fedavg_config.update(attack_params)
        fedavg_config['algorithm'] = 'FedAvg'
        ter, asr = run_experiment(fedavg_config)
        all_results[attack_name]['FedAvg']['ter'] = ter
        all_results[attack_name]['FedAvg']['asr'] = asr
        
        # --- 运行 BALANCE ---
        balance_config = PARAMS.copy()
        balance_config.update(attack_params)
        balance_config['algorithm'] = 'BALANCE'
        ter, asr = run_experiment(balance_config)
        all_results[attack_name]['BALANCE']['ter'] = ter
        all_results[attack_name]['BALANCE']['asr'] = asr

    # 可视化结果
    visualize_results(all_results)
```