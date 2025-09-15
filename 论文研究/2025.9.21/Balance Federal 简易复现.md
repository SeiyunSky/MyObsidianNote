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

### 主程序
```python
# main.py
import torch
import numpy as np
from collections import OrderedDict

from cnn_model import CNN_MNIST
from data_loader import get_mnist_dataloaders
from client import Client

# --- 辅助函数 ---
def get_model_norm(model_state_dict):
    """计算模型参数的L2范数"""
    norm = 0.0
    for param in model_state_dict.values():
        norm += torch.norm(param.float())**2
    return torch.sqrt(norm).item()

def get_model_dist(model1_state_dict, model2_state_dict):
    """计算两个模型参数之间的L2距离"""
    dist = 0.0
    for key in model1_state_dict:
        dist += torch.norm(model1_state_dict[key].float() - model2_state_dict[key].float())**2
    return torch.sqrt(dist).item()

def evaluate_model(model, test_loader):
    """评估模型性能"""
    model.eval()
    test_loss = 0
    correct = 0
    with torch.no_grad():
        for data, target in test_loader:
            output = model(data)
            test_loss += F.nll_loss(output, target, reduction='sum').item()
            pred = output.argmax(dim=1, keepdim=True)
            correct += pred.eq(target.view_as(pred)).sum().item()
    
    test_loss /= len(test_loader.dataset)
    accuracy = 100. * correct / len(test_loader.dataset)
    error_rate = 1.0 - (correct / len(test_loader.dataset))
    return error_rate, accuracy


# --- 聚合算法 ---
def aggregate_fedavg(neighbor_models_states):
    """FedAvg聚合"""
    avg_state_dict = OrderedDict()
    for key in neighbor_models_states[0]:
        avg_state_dict[key] = torch.stack([state[key] for state in neighbor_models_states]).mean(0)
    return avg_state_dict

def aggregate_balance(own_model_state, neighbor_models_states, gamma, kappa, lambda_t):
    """BALANCE 聚合 (公式3)"""
    accepted_models_states = []
    own_norm = get_model_norm(own_model_state)
    
    # 根据公式(3)筛选模型
    threshold = gamma * np.exp(-kappa * lambda_t) * own_norm
    
    for neighbor_state in neighbor_models_states:
        dist = get_model_dist(own_model_state, neighbor_state)
        if dist <= threshold:
            accepted_models_states.append(neighbor_state)
            
    if not accepted_models_states:
        # 如果没有模型通过筛选, 返回一个空的dict
        return None
    
    # 对通过筛选的模型进行平均
    return aggregate_fedavg(accepted_models_states)


# --- 主函数 ---
def run_experiment(config):
    print(f"--- Running Experiment: Algorithm={config['algorithm']}, Attack={config['attack_type']} ---")
    
    # 1. 准备数据
    malicious_ids = list(range(config['num_malicious'])) if config['attack_type'] != 'none' else []
    train_loaders, test_loader = get_mnist_dataloaders(
        num_clients=config['num_clients'],
        malicious_clients_ids=malicious_ids,
        attack_type=config['attack_type']
    )
    
    # 2. 创建通信图 (Regular Graph, 每个节点连接10个邻居)
    adj_matrix = np.zeros((config['num_clients'], config['num_clients']), dtype=int)
    for i in range(config['num_clients']):
        # 为了简单, 我们连接接下来的10个邻居(循环)
        for j in range(1, config['num_neighbors'] // 2 + 1):
            adj_matrix[i, (i + j) % config['num_clients']] = 1
            adj_matrix[i, (i - j + config['num_clients']) % config['num_clients']] = 1

    # 3. 初始化客户端
    clients = []
    initial_model = CNN_MNIST()
    for i in range(config['num_clients']):
        client_model = CNN_MNIST()
        client_model.load_state_dict(initial_model.state_dict()) # 保证初始模型一致
        client = Client(i, client_model, train_loaders[i], learning_rate=config['learning_rate'])
        clients.append(client)
        
    # 4. 训练循环
    for t in range(config['rounds']):
        # a. 本地训练
        local_models_states = {}
        for i in range(config['num_clients']):
            local_models_states[i] = clients[i].local_train()
            
        # b. 去中心化聚合
        next_round_models_states = {}
        for i in range(config['num_clients']):
            # 找到邻居
            neighbor_ids = np.where(adj_matrix[i] == 1)[0]
            neighbor_models = [local_models_states[j] for j in neighbor_ids]
            
            # --- 选择聚合算法 ---
            if config['algorithm'] == 'FedAvg':
                aggregated_state = aggregate_fedavg(neighbor_models)
            elif config['algorithm'] == 'BALANCE':
                lambda_t_val = t / config['rounds'] # lambda(t) = t/T from paper section 5
                aggregated_state = aggregate_balance(
                    local_models_states[i],
                    neighbor_models,
                    config['gamma'],
                    config['kappa'],
                    lambda_t_val
                )
            else:
                raise ValueError("Unknown algorithm")

            # c. 更新模型 (公式4)
            if aggregated_state:
                final_state = OrderedDict()
                alpha = config['alpha']
                for key in local_models_states[i]:
                    final_state[key] = alpha * local_models_states[i][key] + (1 - alpha) * aggregated_state[key]
                next_round_models_states[i] = final_state
            else: # 如果BALANCE没有接受任何模型, 则不更新
                next_round_models_states[i] = local_models_states[i]
        
        # 将更新后的模型加载回客户端
        for i in range(config['num_clients']):
            clients[i].model.load_state_dict(next_round_models_states[i])

        if (t + 1) % 50 == 0:
            print(f"Round {t+1}/{config['rounds']} completed.")

    # 5. 评估
    benign_clients = [c for c in clients if c.client_id not in malicious_ids]
    error_rates = []
    for client in benign_clients:
        error, _ = evaluate_model(client.model, test_loader)
        error_rates.append(error)
        
    max_ter = max(error_rates)
    print(f"Final Result: Max.TER = {max_ter:.4f}\n")
    return max_ter


if __name__ == '__main__':
    torch.manual_seed(42)
    np.random.seed(42)

    # 论文中的超参数 (Section 6.1.5 and default values)
    PARAMS = {
        'num_clients': 20,
        'num_neighbors': 10, # regular-(20, 10) graph
        'num_malicious': 4,  # 20% of 20 clients
        'rounds': 2000,
        'learning_rate': 3e-4,
        # BALANCE & Formula (4) params
        'alpha': 0.5,
        'gamma': 0.3,
        'kappa': 1.0,
    }

    # 实验1: FedAvg, 无攻击
    fedavg_no_attack_params = PARAMS.copy()
    fedavg_no_attack_params.update({'algorithm': 'FedAvg', 'attack_type': 'none'})
    run_experiment(fedavg_no_attack_params)

    # 实验2: FedAvg, 标签翻转攻击
    fedavg_lf_attack_params = PARAMS.copy()
    fedavg_lf_attack_params.update({'algorithm': 'FedAvg', 'attack_type': 'label_flipping'})
    run_experiment(fedavg_lf_attack_params)

    # 实验3: BALANCE, 无攻击
    balance_no_attack_params = PARAMS.copy()
    balance_no_attack_params.update({'algorithm': 'BALANCE', 'attack_type': 'none'})
    run_experiment(balance_no_attack_params)

    # 实验4: BALANCE, 标签翻转攻击
    balance_lf_attack_params = PARAMS.copy()
    balance_lf_attack_params.update({'algorithm': 'BALANCE', 'attack_type': 'label_flipping'})
    run_experiment(balance_lf_attack_params)

```
