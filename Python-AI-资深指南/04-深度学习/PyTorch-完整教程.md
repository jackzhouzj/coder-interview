# PyTorch完整教程

> 掌握PyTorch深度学习框架，构建和训练神经网络
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: PyTorch 2.0+  
**学习难度**: ⭐⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐⭐  
**预计学习时长**: 40-50小时

## 🎯 学习目标

- [ ] 理解PyTorch的核心概念和架构
- [ ] 掌握张量操作和自动微分
- [ ] 能够构建和训练神经网络
- [ ] 理解数据加载和预处理
- [ ] 掌握模型保存和加载
- [ ] 能够使用GPU加速训练
- [ ] 理解分布式训练基础
- [ ] 掌握模型部署方法

## 📖 目录

1. [PyTorch基础](#1-pytorch基础)
2. [张量操作](#2-张量操作)
3. [自动微分](#3-自动微分)
4. [神经网络模块](#4-神经网络模块)
5. [数据加载](#5-数据加载)
6. [训练流程](#6-训练流程)
7. [模型保存](#7-模型保存)
8. [GPU加速](#8-gpu加速)
9. [高级技巧](#9-高级技巧)
10. [最佳实践](#10-最佳实践)

---

## 1. PyTorch基础

### 1.1 安装和导入

```python
"""
PyTorch安装和导入
@author erik.zhou
"""
# 安装PyTorch
# CPU版本: pip install torch torchvision torchaudio
# GPU版本: pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

import torch
import torch.nn as nn
import torch.optim as optim
import numpy as np

# 查看版本
print(f"PyTorch版本: {torch.__version__}")
print(f"CUDA是否可用: {torch.cuda.is_available()}")
if torch.cuda.is_available():
    print(f"CUDA版本: {torch.version.cuda}")
    print(f"GPU设备: {torch.cuda.get_device_name(0)}")
```

### 1.2 PyTorch vs NumPy

```python
"""
PyTorch与NumPy对比
@author erik.zhou
"""
import torch
import numpy as np

# NumPy数组
np_array = np.array([[1, 2, 3], [4, 5, 6]])
print("NumPy数组:")
print(np_array)
print(f"类型: {type(np_array)}")

# PyTorch张量
torch_tensor = torch.tensor([[1, 2, 3], [4, 5, 6]])
print("\nPyTorch张量:")
print(torch_tensor)
print(f"类型: {type(torch_tensor)}")

# NumPy转PyTorch
tensor_from_numpy = torch.from_numpy(np_array)
print("\nNumPy转PyTorch:")
print(tensor_from_numpy)

# PyTorch转NumPy
numpy_from_tensor = torch_tensor.numpy()
print("\nPyTorch转NumPy:")
print(numpy_from_tensor)
```

### 1.3 张量基础

```python
"""
张量基础操作
@author erik.zhou
"""
import torch

# 创建张量
x = torch.tensor([[1, 2], [3, 4]])
print("张量:")
print(x)

# 张量属性
print(f"\n形状: {x.shape}")
print(f"数据类型: {x.dtype}")
print(f"设备: {x.device}")
print(f"维度: {x.ndim}")
print(f"元素总数: {x.numel()}")

# 张量类型转换
x_float = x.float()
x_double = x.double()
x_long = x.long()

print(f"\nfloat类型: {x_float.dtype}")
print(f"double类型: {x_double.dtype}")
print(f"long类型: {x_long.dtype}")
```

---

## 2. 张量操作

### 2.1 创建张量

```python
"""
创建张量的多种方法
@author erik.zhou
"""
import torch

# 从列表创建
tensor_from_list = torch.tensor([[1, 2], [3, 4]])
print("从列表创建:")
print(tensor_from_list)

# 全零张量
zeros = torch.zeros(3, 4)
print("\n全零张量:")
print(zeros)

# 全一张量
ones = torch.ones(2, 3)
print("\n全一张量:")
print(ones)

# 单位矩阵
eye = torch.eye(3)
print("\n单位矩阵:")
print(eye)

# 随机张量
rand = torch.rand(2, 3)  # [0, 1)均匀分布
randn = torch.randn(2, 3)  # 标准正态分布
print("\n随机张量(均匀分布):")
print(rand)
print("\n随机张量(正态分布):")
print(randn)

# 等差数列
arange = torch.arange(0, 10, 2)
print("\n等差数列:")
print(arange)

# 线性空间
linspace = torch.linspace(0, 1, 5)
print("\n线性空间:")
print(linspace)

# 与已有张量相同形状
x = torch.tensor([[1, 2], [3, 4]])
zeros_like = torch.zeros_like(x)
ones_like = torch.ones_like(x)
print("\n与x相同形状的全零张量:")
print(zeros_like)
```

### 2.2 张量索引和切片

```python
"""
张量索引和切片
@author erik.zhou
"""
import torch

x = torch.tensor([[1, 2, 3, 4],
                  [5, 6, 7, 8],
                  [9, 10, 11, 12]])

print("原始张量:")
print(x)

# 基础索引
print(f"\n第一行: {x[0]}")
print(f"第一列: {x[:, 0]}")
print(f"元素[1,2]: {x[1, 2]}")

# 切片
print(f"\n前两行: \n{x[:2]}")
print(f"前两列: \n{x[:, :2]}")
print(f"子矩阵: \n{x[1:3, 1:3]}")

# 布尔索引
mask = x > 5
print(f"\n大于5的元素: {x[mask]}")

# 高级索引
indices = torch.tensor([0, 2])
print(f"\n选择第0和第2行: \n{x[indices]}")
```

### 2.3 张量运算

```python
"""
张量运算
@author erik.zhou
"""
import torch

x = torch.tensor([[1, 2], [3, 4]], dtype=torch.float32)
y = torch.tensor([[5, 6], [7, 8]], dtype=torch.float32)

# 基础运算
print("加法:")
print(x + y)
print(torch.add(x, y))

print("\n减法:")
print(x - y)

print("\n乘法(逐元素):")
print(x * y)

print("\n除法:")
print(x / y)

print("\n幂运算:")
print(x ** 2)

# 矩阵运算
print("\n矩阵乘法:")
print(torch.mm(x, y))
print(x @ y)

# 转置
print("\n转置:")
print(x.t())
print(x.T)

# 聚合运算
print(f"\n求和: {x.sum()}")
print(f"均值: {x.mean()}")
print(f"最大值: {x.max()}")
print(f"最小值: {x.min()}")
print(f"标准差: {x.std()}")

# 按维度聚合
print(f"\n按行求和: {x.sum(dim=0)}")
print(f"按列求和: {x.sum(dim=1)}")
```

### 2.4 张量形状操作

```python
"""
张量形状操作
@author erik.zhou
"""
import torch

x = torch.arange(12)
print("原始张量:")
print(x)

# reshape
x_reshaped = x.reshape(3, 4)
print("\nreshape(3, 4):")
print(x_reshaped)

# view（要求连续内存）
x_view = x.view(2, 6)
print("\nview(2, 6):")
print(x_view)

# 转置后需要contiguous
x_t = x_reshaped.t()
x_view2 = x_t.contiguous().view(-1)
print("\n转置后展平:")
print(x_view2)

# squeeze和unsqueeze
x = torch.tensor([[[1, 2, 3]]])
print(f"\n原始形状: {x.shape}")

x_squeezed = x.squeeze()
print(f"squeeze后: {x_squeezed.shape}")

x_unsqueezed = x_squeezed.unsqueeze(0)
print(f"unsqueeze(0)后: {x_unsqueezed.shape}")

# 拼接
x1 = torch.tensor([[1, 2], [3, 4]])
x2 = torch.tensor([[5, 6], [7, 8]])

cat_dim0 = torch.cat([x1, x2], dim=0)
print("\ncat(dim=0):")
print(cat_dim0)

cat_dim1 = torch.cat([x1, x2], dim=1)
print("\ncat(dim=1):")
print(cat_dim1)

# stack
stacked = torch.stack([x1, x2], dim=0)
print(f"\nstack后形状: {stacked.shape}")
print(stacked)
```

---

## 3. 自动微分

### 3.1 自动微分基础

```python
"""
自动微分基础
@author erik.zhou
"""
import torch

# 创建需要梯度的张量
x = torch.tensor([2.0], requires_grad=True)
print(f"x: {x}")
print(f"requires_grad: {x.requires_grad}")

# 定义函数 y = x^2 + 2x + 1
y = x**2 + 2*x + 1
print(f"\ny: {y}")

# 反向传播
y.backward()

# 查看梯度 dy/dx = 2x + 2 = 6
print(f"\ndy/dx: {x.grad}")
```

### 3.2 梯度计算

```python
"""
梯度计算示例
@author erik.zhou
"""
import torch

# 多变量函数
x = torch.tensor([1.0, 2.0, 3.0], requires_grad=True)
y = torch.tensor([4.0, 5.0, 6.0], requires_grad=True)

# z = x^2 + y^2
z = (x**2 + y**2).sum()
print(f"z: {z}")

# 反向传播
z.backward()

print(f"\ndz/dx: {x.grad}")  # 2x
print(f"dz/dy: {y.grad}")  # 2y

# 梯度清零
x.grad.zero_()
y.grad.zero_()
print(f"\n清零后x.grad: {x.grad}")
```

### 3.3 梯度控制

```python
"""
梯度控制
@author erik.zhou
"""
import torch

x = torch.tensor([1.0, 2.0, 3.0], requires_grad=True)

# 禁用梯度计算
with torch.no_grad():
    y = x * 2
    print(f"y.requires_grad: {y.requires_grad}")

# detach分离
y = x * 2
z = y.detach()
print(f"\nz.requires_grad: {z.requires_grad}")

# 阻止梯度传播
x = torch.tensor([1.0, 2.0], requires_grad=True)
y = x * 2
y = y.detach()
z = y * 3

z.sum().backward()
print(f"\nx.grad: {x.grad}")  # None，因为y被detach了
```

---

## 4. 神经网络模块

### 4.1 nn.Module基础

```python
"""
nn.Module基础
@author erik.zhou
"""
import torch
import torch.nn as nn

class SimpleNet(nn.Module):
    """简单的神经网络"""
    
    def __init__(self, input_size, hidden_size, output_size):
        super(SimpleNet, self).__init__()
        self.fc1 = nn.Linear(input_size, hidden_size)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        """前向传播"""
        x = self.fc1(x)
        x = self.relu(x)
        x = self.fc2(x)
        return x

# 创建模型
model = SimpleNet(input_size=10, hidden_size=20, output_size=2)
print("模型结构:")
print(model)

# 查看参数
print("\n模型参数:")
for name, param in model.named_parameters():
    print(f"{name}: {param.shape}")

# 前向传播
x = torch.randn(5, 10)
output = model(x)
print(f"\n输入形状: {x.shape}")
print(f"输出形状: {output.shape}")
```

### 4.2 常用层

```python
"""
常用神经网络层
@author erik.zhou
"""
import torch
import torch.nn as nn

# 全连接层
linear = nn.Linear(in_features=10, out_features=5)
x = torch.randn(3, 10)
output = linear(x)
print(f"Linear输出形状: {output.shape}")

# 卷积层
conv2d = nn.Conv2d(in_channels=3, out_channels=16, kernel_size=3, padding=1)
x = torch.randn(1, 3, 32, 32)  # (batch, channels, height, width)
output = conv2d(x)
print(f"\nConv2d输出形状: {output.shape}")

# 池化层
maxpool = nn.MaxPool2d(kernel_size=2, stride=2)
output = maxpool(output)
print(f"MaxPool2d输出形状: {output.shape}")

# Dropout
dropout = nn.Dropout(p=0.5)
x = torch.randn(10, 20)
output = dropout(x)
print(f"\nDropout输出形状: {output.shape}")

# BatchNorm
batchnorm = nn.BatchNorm1d(num_features=20)
output = batchnorm(x)
print(f"BatchNorm输出形状: {output.shape}")

# 激活函数
relu = nn.ReLU()
sigmoid = nn.Sigmoid()
tanh = nn.Tanh()

x = torch.randn(5)
print(f"\n原始值: {x}")
print(f"ReLU: {relu(x)}")
print(f"Sigmoid: {sigmoid(x)}")
print(f"Tanh: {tanh(x)}")
```


### 4.3 Sequential容器

```python
"""
Sequential容器
@author erik.zhou
"""
import torch
import torch.nn as nn

# 方式1: 顺序添加
model = nn.Sequential(
    nn.Linear(10, 20),
    nn.ReLU(),
    nn.Linear(20, 10),
    nn.ReLU(),
    nn.Linear(10, 2)
)

print("Sequential模型:")
print(model)

# 方式2: 使用OrderedDict
from collections import OrderedDict

model = nn.Sequential(OrderedDict([
    ('fc1', nn.Linear(10, 20)),
    ('relu1', nn.ReLU()),
    ('fc2', nn.Linear(20, 10)),
    ('relu2', nn.ReLU()),
    ('fc3', nn.Linear(10, 2))
]))

print("\n带名称的Sequential模型:")
print(model)

# 访问子模块
print(f"\n第一层: {model[0]}")
print(f"fc1层: {model.fc1}")
```

### 4.4 自定义层

```python
"""
自定义神经网络层
@author erik.zhou
"""
import torch
import torch.nn as nn

class CustomLinear(nn.Module):
    """自定义线性层"""
    
    def __init__(self, in_features, out_features):
        super(CustomLinear, self).__init__()
        self.weight = nn.Parameter(torch.randn(out_features, in_features))
        self.bias = nn.Parameter(torch.randn(out_features))
    
    def forward(self, x):
        return torch.mm(x, self.weight.t()) + self.bias

# 使用自定义层
layer = CustomLinear(10, 5)
x = torch.randn(3, 10)
output = layer(x)

print(f"输入形状: {x.shape}")
print(f"输出形状: {output.shape}")
print(f"\n权重形状: {layer.weight.shape}")
print(f"偏置形状: {layer.bias.shape}")
```

---

## 5. 数据加载

### 5.1 Dataset和DataLoader

```python
"""
Dataset和DataLoader基础
@author erik.zhou
"""
import torch
from torch.utils.data import Dataset, DataLoader
import numpy as np

class CustomDataset(Dataset):
    """自定义数据集"""
    
    def __init__(self, data, labels):
        self.data = data
        self.labels = labels
    
    def __len__(self):
        return len(self.data)
    
    def __getitem__(self, idx):
        return self.data[idx], self.labels[idx]

# 创建数据
data = torch.randn(100, 10)
labels = torch.randint(0, 2, (100,))

# 创建数据集
dataset = CustomDataset(data, labels)
print(f"数据集大小: {len(dataset)}")

# 创建DataLoader
dataloader = DataLoader(
    dataset,
    batch_size=16,
    shuffle=True,
    num_workers=0
)

# 遍历数据
for batch_idx, (batch_data, batch_labels) in enumerate(dataloader):
    print(f"\nBatch {batch_idx}:")
    print(f"数据形状: {batch_data.shape}")
    print(f"标签形状: {batch_labels.shape}")
    if batch_idx == 2:
        break
```

### 5.2 数据增强

```python
"""
数据增强
@author erik.zhou
"""
import torch
import torchvision.transforms as transforms
from PIL import Image
import numpy as np

# 定义数据增强
transform = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.RandomHorizontalFlip(p=0.5),
    transforms.RandomRotation(degrees=15),
    transforms.ColorJitter(brightness=0.2, contrast=0.2),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                       std=[0.229, 0.224, 0.225])
])

# 创建示例图像
img_array = np.random.randint(0, 255, (256, 256, 3), dtype=np.uint8)
img = Image.fromarray(img_array)

# 应用变换
img_transformed = transform(img)
print(f"变换后形状: {img_transformed.shape}")
print(f"数据类型: {img_transformed.dtype}")
print(f"数值范围: [{img_transformed.min():.2f}, {img_transformed.max():.2f}]")
```

### 5.3 内置数据集

```python
"""
使用PyTorch内置数据集
@author erik.zhou
"""
import torch
import torchvision
import torchvision.transforms as transforms

# 定义变换
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.5,), (0.5,))
])

# 加载MNIST数据集（示例，实际需要下载）
# train_dataset = torchvision.datasets.MNIST(
#     root='./data',
#     train=True,
#     download=True,
#     transform=transform
# )

# test_dataset = torchvision.datasets.MNIST(
#     root='./data',
#     train=False,
#     download=True,
#     transform=transform
# )

# 创建DataLoader
# train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True)
# test_loader = DataLoader(test_dataset, batch_size=64, shuffle=False)

print("数据集加载完成（示例代码）")
```

---

## 6. 训练流程

### 6.1 完整训练示例

```python
"""
完整的训练流程
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

# 1. 准备数据
X_train = torch.randn(1000, 10)
y_train = torch.randint(0, 2, (1000,))
X_test = torch.randn(200, 10)
y_test = torch.randint(0, 2, (200,))

train_dataset = TensorDataset(X_train, y_train)
test_dataset = TensorDataset(X_test, y_test)

train_loader = DataLoader(train_dataset, batch_size=32, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=32, shuffle=False)

# 2. 定义模型
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.fc1 = nn.Linear(10, 20)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(20, 2)
    
    def forward(self, x):
        x = self.fc1(x)
        x = self.relu(x)
        x = self.fc2(x)
        return x

model = Net()

# 3. 定义损失函数和优化器
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# 4. 训练循环
num_epochs = 10

for epoch in range(num_epochs):
    model.train()
    train_loss = 0.0
    train_correct = 0
    
    for batch_data, batch_labels in train_loader:
        # 前向传播
        outputs = model(batch_data)
        loss = criterion(outputs, batch_labels)
        
        # 反向传播和优化
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        
        # 统计
        train_loss += loss.item()
        _, predicted = torch.max(outputs.data, 1)
        train_correct += (predicted == batch_labels).sum().item()
    
    # 验证
    model.eval()
    test_correct = 0
    
    with torch.no_grad():
        for batch_data, batch_labels in test_loader:
            outputs = model(batch_data)
            _, predicted = torch.max(outputs.data, 1)
            test_correct += (predicted == batch_labels).sum().item()
    
    # 打印结果
    train_acc = 100 * train_correct / len(train_dataset)
    test_acc = 100 * test_correct / len(test_dataset)
    
    print(f'Epoch [{epoch+1}/{num_epochs}], '
          f'Loss: {train_loss/len(train_loader):.4f}, '
          f'Train Acc: {train_acc:.2f}%, '
          f'Test Acc: {test_acc:.2f}%')
```

### 6.2 学习率调度

```python
"""
学习率调度器
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.optim as optim
from torch.optim.lr_scheduler import StepLR, ReduceLROnPlateau, CosineAnnealingLR

model = nn.Linear(10, 2)
optimizer = optim.SGD(model.parameters(), lr=0.1)

# 方式1: StepLR - 每隔step_size个epoch，学习率乘以gamma
scheduler1 = StepLR(optimizer, step_size=10, gamma=0.1)

# 方式2: ReduceLROnPlateau - 当指标不再改善时降低学习率
scheduler2 = ReduceLROnPlateau(optimizer, mode='min', factor=0.1, patience=5)

# 方式3: CosineAnnealingLR - 余弦退火
scheduler3 = CosineAnnealingLR(optimizer, T_max=50, eta_min=0.001)

# 使用示例
for epoch in range(20):
    # 训练代码...
    
    # 更新学习率
    scheduler1.step()
    
    # 或者基于验证损失
    # val_loss = ...
    # scheduler2.step(val_loss)
    
    print(f"Epoch {epoch}, LR: {optimizer.param_groups[0]['lr']:.6f}")
```

### 6.3 早停和模型检查点

```python
"""
早停和模型检查点
@author erik.zhou
"""
import torch
import torch.nn as nn
import numpy as np

class EarlyStopping:
    """早停类"""
    
    def __init__(self, patience=7, min_delta=0):
        self.patience = patience
        self.min_delta = min_delta
        self.counter = 0
        self.best_loss = None
        self.early_stop = False
    
    def __call__(self, val_loss):
        if self.best_loss is None:
            self.best_loss = val_loss
        elif val_loss > self.best_loss - self.min_delta:
            self.counter += 1
            if self.counter >= self.patience:
                self.early_stop = True
        else:
            self.best_loss = val_loss
            self.counter = 0

# 使用示例
model = nn.Linear(10, 2)
early_stopping = EarlyStopping(patience=5)

for epoch in range(100):
    # 训练代码...
    val_loss = np.random.random()  # 示例
    
    # 检查早停
    early_stopping(val_loss)
    
    if early_stopping.early_stop:
        print(f"Early stopping at epoch {epoch}")
        break
    
    # 保存最佳模型
    if val_loss == early_stopping.best_loss:
        torch.save(model.state_dict(), 'best_model.pth')
        print(f"Saved best model at epoch {epoch}")
```

---

## 7. 模型保存

### 7.1 保存和加载模型

```python
"""
模型保存和加载
@author erik.zhou
"""
import torch
import torch.nn as nn

class SimpleNet(nn.Module):
    def __init__(self):
        super(SimpleNet, self).__init__()
        self.fc = nn.Linear(10, 2)
    
    def forward(self, x):
        return self.fc(x)

model = SimpleNet()

# 方式1: 只保存参数（推荐）
torch.save(model.state_dict(), 'model_params.pth')

# 加载参数
model_new = SimpleNet()
model_new.load_state_dict(torch.load('model_params.pth'))
model_new.eval()

# 方式2: 保存整个模型
torch.save(model, 'model_complete.pth')

# 加载整个模型
model_loaded = torch.load('model_complete.pth')
model_loaded.eval()

# 方式3: 保存检查点（包含更多信息）
checkpoint = {
    'epoch': 10,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': None,  # optimizer.state_dict()
    'loss': 0.5,
}
torch.save(checkpoint, 'checkpoint.pth')

# 加载检查点
checkpoint = torch.load('checkpoint.pth')
model.load_state_dict(checkpoint['model_state_dict'])
epoch = checkpoint['epoch']
loss = checkpoint['loss']

print(f"加载检查点: epoch={epoch}, loss={loss}")
```

### 7.2 模型导出

```python
"""
模型导出为ONNX
@author erik.zhou
"""
import torch
import torch.nn as nn

class SimpleNet(nn.Module):
    def __init__(self):
        super(SimpleNet, self).__init__()
        self.fc = nn.Linear(10, 2)
    
    def forward(self, x):
        return self.fc(x)

model = SimpleNet()
model.eval()

# 创建示例输入
dummy_input = torch.randn(1, 10)

# 导出为ONNX
torch.onnx.export(
    model,
    dummy_input,
    'model.onnx',
    export_params=True,
    opset_version=11,
    do_constant_folding=True,
    input_names=['input'],
    output_names=['output'],
    dynamic_axes={
        'input': {0: 'batch_size'},
        'output': {0: 'batch_size'}
    }
)

print("模型已导出为ONNX格式")
```

---

## 8. GPU加速

### 8.1 使用GPU

```python
"""
GPU加速训练
@author erik.zhou
"""
import torch
import torch.nn as nn

# 检查GPU可用性
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"使用设备: {device}")

# 创建模型并移到GPU
model = nn.Linear(10, 2)
model = model.to(device)

# 创建数据并移到GPU
x = torch.randn(5, 10).to(device)
y = model(x)

print(f"输入设备: {x.device}")
print(f"输出设备: {y.device}")

# 将数据移回CPU
y_cpu = y.cpu()
print(f"移回CPU后: {y_cpu.device}")
```

### 8.2 多GPU训练

```python
"""
多GPU并行训练
@author erik.zhou
"""
import torch
import torch.nn as nn

# 检查GPU数量
if torch.cuda.device_count() > 1:
    print(f"使用 {torch.cuda.device_count()} 个GPU")
    
    model = nn.Linear(10, 2)
    
    # DataParallel包装
    model = nn.DataParallel(model)
    model = model.cuda()
    
    # 训练代码...
    x = torch.randn(32, 10).cuda()
    y = model(x)
    
    print(f"输出形状: {y.shape}")
else:
    print("只有一个GPU或没有GPU")
```

### 8.3 混合精度训练

```python
"""
混合精度训练（FP16）
@author erik.zhou
"""
import torch
import torch.nn as nn
from torch.cuda.amp import autocast, GradScaler

model = nn.Linear(10, 2).cuda()
optimizer = torch.optim.Adam(model.parameters())
scaler = GradScaler()

# 训练循环
for epoch in range(10):
    x = torch.randn(32, 10).cuda()
    y = torch.randint(0, 2, (32,)).cuda()
    
    optimizer.zero_grad()
    
    # 使用autocast进行混合精度
    with autocast():
        outputs = model(x)
        loss = nn.functional.cross_entropy(outputs, y)
    
    # 使用GradScaler进行梯度缩放
    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
    
    if epoch % 2 == 0:
        print(f"Epoch {epoch}, Loss: {loss.item():.4f}")
```

---

## 9. 高级技巧

### 9.1 自定义损失函数

```python
"""
自定义损失函数
@author erik.zhou
"""
import torch
import torch.nn as nn

class FocalLoss(nn.Module):
    """Focal Loss用于处理类别不平衡"""
    
    def __init__(self, alpha=1, gamma=2):
        super(FocalLoss, self).__init__()
        self.alpha = alpha
        self.gamma = gamma
    
    def forward(self, inputs, targets):
        ce_loss = nn.functional.cross_entropy(inputs, targets, reduction='none')
        pt = torch.exp(-ce_loss)
        focal_loss = self.alpha * (1 - pt) ** self.gamma * ce_loss
        return focal_loss.mean()

# 使用自定义损失
criterion = FocalLoss(alpha=1, gamma=2)
outputs = torch.randn(10, 3)
targets = torch.randint(0, 3, (10,))
loss = criterion(outputs, targets)

print(f"Focal Loss: {loss.item():.4f}")
```

### 9.2 梯度裁剪

```python
"""
梯度裁剪防止梯度爆炸
@author erik.zhou
"""
import torch
import torch.nn as nn

model = nn.Linear(10, 2)
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)

x = torch.randn(5, 10)
y = torch.randint(0, 2, (5,))

# 前向传播
outputs = model(x)
loss = nn.functional.cross_entropy(outputs, y)

# 反向传播
optimizer.zero_grad()
loss.backward()

# 梯度裁剪
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)

# 更新参数
optimizer.step()

print("梯度裁剪完成")
```

### 9.3 模型微调

```python
"""
模型微调（冻结部分层）
@author erik.zhou
"""
import torch
import torch.nn as nn

class PretrainedModel(nn.Module):
    def __init__(self):
        super(PretrainedModel, self).__init__()
        self.features = nn.Sequential(
            nn.Linear(10, 20),
            nn.ReLU(),
            nn.Linear(20, 10)
        )
        self.classifier = nn.Linear(10, 2)
    
    def forward(self, x):
        x = self.features(x)
        x = self.classifier(x)
        return x

model = PretrainedModel()

# 冻结特征提取层
for param in model.features.parameters():
    param.requires_grad = False

# 只训练分类器
optimizer = torch.optim.Adam(
    filter(lambda p: p.requires_grad, model.parameters()),
    lr=0.001
)

# 查看可训练参数
print("可训练参数:")
for name, param in model.named_parameters():
    if param.requires_grad:
        print(f"  {name}")
```

---

## 10. 最佳实践

### 10.1 代码组织

```python
"""
PyTorch项目代码组织
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader

class Config:
    """配置类"""
    batch_size = 32
    learning_rate = 0.001
    num_epochs = 10
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

class Trainer:
    """训练器类"""
    
    def __init__(self, model, train_loader, val_loader, config):
        self.model = model.to(config.device)
        self.train_loader = train_loader
        self.val_loader = val_loader
        self.config = config
        
        self.criterion = nn.CrossEntropyLoss()
        self.optimizer = optim.Adam(model.parameters(), lr=config.learning_rate)
    
    def train_epoch(self):
        """训练一个epoch"""
        self.model.train()
        total_loss = 0
        
        for batch_data, batch_labels in self.train_loader:
            batch_data = batch_data.to(self.config.device)
            batch_labels = batch_labels.to(self.config.device)
            
            self.optimizer.zero_grad()
            outputs = self.model(batch_data)
            loss = self.criterion(outputs, batch_labels)
            loss.backward()
            self.optimizer.step()
            
            total_loss += loss.item()
        
        return total_loss / len(self.train_loader)
    
    def validate(self):
        """验证"""
        self.model.eval()
        correct = 0
        total = 0
        
        with torch.no_grad():
            for batch_data, batch_labels in self.val_loader:
                batch_data = batch_data.to(self.config.device)
                batch_labels = batch_labels.to(self.config.device)
                
                outputs = self.model(batch_data)
                _, predicted = torch.max(outputs.data, 1)
                total += batch_labels.size(0)
                correct += (predicted == batch_labels).sum().item()
        
        return 100 * correct / total
    
    def train(self):
        """完整训练流程"""
        for epoch in range(self.config.num_epochs):
            train_loss = self.train_epoch()
            val_acc = self.validate()
            
            print(f'Epoch [{epoch+1}/{self.config.num_epochs}], '
                  f'Loss: {train_loss:.4f}, Val Acc: {val_acc:.2f}%')

# 使用示例
# config = Config()
# trainer = Trainer(model, train_loader, val_loader, config)
# trainer.train()
```

### 10.2 调试技巧

```python
"""
PyTorch调试技巧
@author erik.zhou
"""
import torch
import torch.nn as nn

# 1. 检查梯度
model = nn.Linear(10, 2)
x = torch.randn(5, 10, requires_grad=True)
y = model(x)
loss = y.sum()
loss.backward()

print("检查梯度:")
for name, param in model.named_parameters():
    if param.grad is not None:
        print(f"{name}: grad_norm={param.grad.norm().item():.4f}")

# 2. 检查NaN
def check_nan(tensor, name="tensor"):
    """检查张量中是否有NaN"""
    if torch.isnan(tensor).any():
        print(f"警告: {name} 包含NaN值")
        return True
    return False

x = torch.tensor([1.0, 2.0, float('nan')])
check_nan(x, "x")

# 3. 打印模型信息
def print_model_info(model):
    """打印模型信息"""
    total_params = sum(p.numel() for p in model.parameters())
    trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
    
    print(f"总参数量: {total_params:,}")
    print(f"可训练参数: {trainable_params:,}")
    print(f"模型大小: {total_params * 4 / 1024 / 1024:.2f} MB")

model = nn.Sequential(
    nn.Linear(100, 50),
    nn.ReLU(),
    nn.Linear(50, 10)
)
print_model_info(model)
```

### 10.3 性能优化

```python
"""
PyTorch性能优化
@author erik.zhou
"""
import torch
import torch.nn as nn
import time

# 1. 使用DataLoader的num_workers
# train_loader = DataLoader(dataset, batch_size=32, num_workers=4)

# 2. 固定随机种子
def set_seed(seed=42):
    """设置随机种子"""
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False

set_seed(42)

# 3. 使用inplace操作
x = torch.randn(1000, 1000)

# 非inplace
start = time.time()
y = torch.relu(x)
print(f"非inplace: {time.time() - start:.4f}秒")

# inplace
start = time.time()
x.relu_()
print(f"inplace: {time.time() - start:.4f}秒")

# 4. 避免频繁的CPU-GPU数据传输
# 不好的做法
# for i in range(100):
#     x = torch.randn(10, 10).cuda()
#     y = model(x)
#     loss = y.sum().cpu()  # 频繁传输

# 好的做法
# x = torch.randn(100, 10, 10).cuda()
# for i in range(100):
#     y = model(x[i])
#     loss = y.sum()
# loss_cpu = loss.cpu()  # 一次传输
```

---

## 📝 学习检查清单

- [ ] 理解PyTorch的核心概念
- [ ] 掌握张量操作和自动微分
- [ ] 能够构建自定义神经网络
- [ ] 理解数据加载和预处理
- [ ] 掌握完整的训练流程
- [ ] 能够保存和加载模型
- [ ] 理解GPU加速和混合精度训练
- [ ] 掌握模型微调技术
- [ ] 了解调试和性能优化技巧

## 🔗 相关资源

- [PyTorch官方文档](https://pytorch.org/docs/stable/index.html)
- [PyTorch教程](https://pytorch.org/tutorials/)
- [PyTorch示例](https://github.com/pytorch/examples)
- [动手学深度学习PyTorch版](https://zh.d2l.ai/)

---

**@author erik.zhou**  
**最后更新**: 2026-02-13
