# CNN卷积神经网络完整教程

> 掌握卷积神经网络，构建计算机视觉应用
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: PyTorch 2.0+, TensorFlow 2.x  
**学习难度**: ⭐⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐⭐  
**预计学习时长**: 30-40小时

## 🎯 学习目标

- [ ] 理解卷积神经网络的基本原理
- [ ] 掌握卷积层和池化层的作用
- [ ] 理解经典CNN架构（LeNet、AlexNet、VGG、ResNet等）
- [ ] 能够实现图像分类任务
- [ ] 掌握迁移学习技术
- [ ] 理解数据增强方法
- [ ] 能够进行目标检测和图像分割
- [ ] 掌握CNN可视化技术

## 📖 目录

1. [CNN基础](#1-cnn基础)
2. [卷积层](#2-卷积层)
3. [池化层](#3-池化层)
4. [经典架构](#4-经典架构)
5. [图像分类](#5-图像分类)
6. [数据增强](#6-数据增强)
7. [迁移学习](#7-迁移学习)
8. [目标检测](#8-目标检测)
9. [图像分割](#9-图像分割)
10. [最佳实践](#10-最佳实践)

---

## 1. CNN基础

### 1.1 为什么需要CNN

```python
"""
CNN vs 全连接网络
@author erik.zhou
"""
import numpy as np
import torch
import torch.nn as nn

# 全连接网络处理图像的问题
image_size = 224 * 224 * 3  # 224x224 RGB图像
hidden_size = 1000
output_size = 10

# 全连接层参数量
fc_params = image_size * hidden_size + hidden_size * output_size
print(f"全连接网络参数量: {fc_params:,}")  # 约1.5亿参数

# CNN参数量（简化示例）
conv_params = 3 * 3 * 3 * 64 + 3 * 3 * 64 * 128  # 两个卷积层
fc_params_cnn = 7 * 7 * 128 * output_size  # 最后的全连接层
total_cnn_params = conv_params + fc_params_cnn
print(f"CNN参数量: {total_cnn_params:,}")  # 约6万参数

print(f"\n参数量减少: {(1 - total_cnn_params/fc_params) * 100:.1f}%")

print("""
CNN的优势：
1. 参数共享：卷积核在整个图像上共享
2. 局部连接：每个神经元只连接局部区域
3. 平移不变性：对图像平移具有鲁棒性
4. 层次特征：从低级特征到高级特征
""")
```

### 1.2 卷积操作原理

```python
"""
卷积操作原理演示
@author erik.zhou
"""
import numpy as np
import matplotlib.pyplot as plt

def convolve2d(image, kernel):
    """
    2D卷积操作
    
    Args:
        image: 输入图像 (H, W)
        kernel: 卷积核 (K, K)
    
    Returns:
        输出特征图
    """
    H, W = image.shape
    K = kernel.shape[0]
    
    # 输出大小
    out_H = H - K + 1
    out_W = W - K + 1
    
    output = np.zeros((out_H, out_W))
    
    for i in range(out_H):
        for j in range(out_W):
            # 提取局部区域
            region = image[i:i+K, j:j+K]
            # 逐元素相乘并求和
            output[i, j] = np.sum(region * kernel)
    
    return output

# 创建示例图像
image = np.array([
    [1, 2, 3, 4, 5],
    [6, 7, 8, 9, 10],
    [11, 12, 13, 14, 15],
    [16, 17, 18, 19, 20],
    [21, 22, 23, 24, 25]
])

# 边缘检测卷积核
edge_kernel = np.array([
    [-1, -1, -1],
    [-1,  8, -1],
    [-1, -1, -1]
])

# 执行卷积
output = convolve2d(image, edge_kernel)

# 可视化
fig, axes = plt.subplots(1, 3, figsize=(15, 5))

axes[0].imshow(image, cmap='gray')
axes[0].set_title('输入图像')
axes[0].axis('off')

axes[1].imshow(edge_kernel, cmap='gray')
axes[1].set_title('卷积核（边缘检测）')
axes[1].axis('off')

axes[2].imshow(output, cmap='gray')
axes[2].set_title('输出特征图')
axes[2].axis('off')

plt.tight_layout()
plt.show()

print(f"输入形状: {image.shape}")
print(f"卷积核形状: {edge_kernel.shape}")
print(f"输出形状: {output.shape}")
```

### 1.3 填充和步长

```python
"""
填充（Padding）和步长（Stride）
@author erik.zhou
"""
import torch
import torch.nn as nn

# 创建输入
x = torch.randn(1, 1, 5, 5)  # (batch, channels, height, width)

# 不同的填充和步长
configs = [
    ('无填充, stride=1', nn.Conv2d(1, 1, kernel_size=3, stride=1, padding=0)),
    ('padding=1, stride=1', nn.Conv2d(1, 1, kernel_size=3, stride=1, padding=1)),
    ('padding=1, stride=2', nn.Conv2d(1, 1, kernel_size=3, stride=2, padding=1)),
]

print("输入形状:", x.shape)
print()

for name, conv in configs:
    output = conv(x)
    print(f"{name}:")
    print(f"  输出形状: {output.shape}")
    print(f"  输出大小: {output.shape[2]}x{output.shape[3]}")
    print()

print("""
输出大小计算公式：
output_size = (input_size + 2*padding - kernel_size) / stride + 1

常用配置：
1. Same Padding: padding = (kernel_size - 1) / 2, stride=1
   保持输入输出大小相同
2. Valid Padding: padding = 0
   输出大小减小
3. Stride > 1: 下采样，减小特征图大小
""")
```


---

## 2. 卷积层

### 2.1 卷积层实现

```python
"""
卷积层的PyTorch实现
@author erik.zhou
"""
import torch
import torch.nn as nn

class SimpleCNN(nn.Module):
    """简单的CNN模型"""
    
    def __init__(self):
        super(SimpleCNN, self).__init__()
        
        # 卷积层
        self.conv1 = nn.Conv2d(
            in_channels=3,      # 输入通道数（RGB）
            out_channels=32,    # 输出通道数（特征图数量）
            kernel_size=3,      # 卷积核大小
            stride=1,           # 步长
            padding=1           # 填充
        )
        
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3, stride=1, padding=1)
        
        # 激活函数
        self.relu = nn.ReLU()
        
        # 池化层
        self.pool = nn.MaxPool2d(kernel_size=2, stride=2)
        
        # 全连接层
        self.fc = nn.Linear(64 * 8 * 8, 10)
    
    def forward(self, x):
        """前向传播"""
        # 第一个卷积块
        x = self.conv1(x)      # (batch, 32, 32, 32)
        x = self.relu(x)
        x = self.pool(x)       # (batch, 32, 16, 16)
        
        # 第二个卷积块
        x = self.conv2(x)      # (batch, 64, 16, 16)
        x = self.relu(x)
        x = self.pool(x)       # (batch, 64, 8, 8)
        
        # 展平
        x = x.view(x.size(0), -1)  # (batch, 64*8*8)
        
        # 全连接层
        x = self.fc(x)         # (batch, 10)
        
        return x

# 创建模型
model = SimpleCNN()

# 测试
x = torch.randn(4, 3, 32, 32)  # 4张32x32的RGB图像
output = model(x)

print(f"输入形状: {x.shape}")
print(f"输出形状: {output.shape}")

# 查看模型结构
print("\n模型结构:")
print(model)

# 计算参数量
total_params = sum(p.numel() for p in model.parameters())
print(f"\n总参数量: {total_params:,}")
```

### 2.2 不同类型的卷积

```python
"""
不同类型的卷积操作
@author erik.zhou
"""
import torch
import torch.nn as nn

# 1. 标准卷积
standard_conv = nn.Conv2d(3, 64, kernel_size=3, padding=1)

# 2. 深度可分离卷积（Depthwise Separable Convolution）
class DepthwiseSeparableConv(nn.Module):
    """深度可分离卷积"""
    
    def __init__(self, in_channels, out_channels, kernel_size=3):
        super(DepthwiseSeparableConv, self).__init__()
        
        # 深度卷积（每个通道单独卷积）
        self.depthwise = nn.Conv2d(
            in_channels, in_channels, 
            kernel_size=kernel_size,
            padding=kernel_size//2,
            groups=in_channels  # 关键：groups=in_channels
        )
        
        # 逐点卷积（1x1卷积）
        self.pointwise = nn.Conv2d(in_channels, out_channels, kernel_size=1)
    
    def forward(self, x):
        x = self.depthwise(x)
        x = self.pointwise(x)
        return x

# 3. 空洞卷积（Dilated/Atrous Convolution）
dilated_conv = nn.Conv2d(3, 64, kernel_size=3, padding=2, dilation=2)

# 4. 转置卷积（Transposed Convolution）
transposed_conv = nn.ConvTranspose2d(64, 3, kernel_size=4, stride=2, padding=1)

# 测试
x = torch.randn(1, 3, 32, 32)

print("标准卷积:")
print(f"  输入: {x.shape}")
print(f"  输出: {standard_conv(x).shape}")

print("\n深度可分离卷积:")
dsconv = DepthwiseSeparableConv(3, 64)
print(f"  输入: {x.shape}")
print(f"  输出: {dsconv(x).shape}")

print("\n空洞卷积:")
print(f"  输入: {x.shape}")
print(f"  输出: {dilated_conv(x).shape}")

print("\n转置卷积:")
x_down = torch.randn(1, 64, 16, 16)
print(f"  输入: {x_down.shape}")
print(f"  输出: {transposed_conv(x_down).shape}")

# 参数量对比
standard_params = sum(p.numel() for p in standard_conv.parameters())
dsconv_params = sum(p.numel() for p in dsconv.parameters())

print(f"\n参数量对比:")
print(f"  标准卷积: {standard_params:,}")
print(f"  深度可分离卷积: {dsconv_params:,}")
print(f"  参数减少: {(1 - dsconv_params/standard_params) * 100:.1f}%")
```

### 2.3 卷积层可视化

```python
"""
卷积层特征图可视化
@author erik.zhou
"""
import torch
import torch.nn as nn
import matplotlib.pyplot as plt
import numpy as np

def visualize_feature_maps(model, image, layer_name='conv1'):
    """
    可视化卷积层的特征图
    
    Args:
        model: CNN模型
        image: 输入图像
        layer_name: 要可视化的层名称
    """
    # 注册hook获取中间层输出
    activation = {}
    
    def get_activation(name):
        def hook(model, input, output):
            activation[name] = output.detach()
        return hook
    
    # 注册hook
    getattr(model, layer_name).register_forward_hook(get_activation(layer_name))
    
    # 前向传播
    model.eval()
    with torch.no_grad():
        _ = model(image)
    
    # 获取特征图
    feature_maps = activation[layer_name].squeeze(0)  # 移除batch维度
    
    # 可视化
    num_features = min(16, feature_maps.shape[0])  # 最多显示16个特征图
    fig, axes = plt.subplots(4, 4, figsize=(12, 12))
    axes = axes.flatten()
    
    for i in range(num_features):
        axes[i].imshow(feature_maps[i].cpu().numpy(), cmap='viridis')
        axes[i].set_title(f'特征图 {i+1}')
        axes[i].axis('off')
    
    # 隐藏多余的子图
    for i in range(num_features, 16):
        axes[i].axis('off')
    
    plt.suptitle(f'{layer_name} 特征图可视化', fontsize=16)
    plt.tight_layout()
    plt.show()

# 创建模型和输入
model = SimpleCNN()
x = torch.randn(1, 3, 32, 32)

# 可视化第一层卷积的特征图
visualize_feature_maps(model, x, 'conv1')
```

---

## 3. 池化层

### 3.1 池化层类型

```python
"""
不同类型的池化层
@author erik.zhou
"""
import torch
import torch.nn as nn
import matplotlib.pyplot as plt

# 创建测试数据
x = torch.randn(1, 1, 8, 8)

# 1. 最大池化（Max Pooling）
max_pool = nn.MaxPool2d(kernel_size=2, stride=2)

# 2. 平均池化（Average Pooling）
avg_pool = nn.AvgPool2d(kernel_size=2, stride=2)

# 3. 全局平均池化（Global Average Pooling）
global_avg_pool = nn.AdaptiveAvgPool2d((1, 1))

# 4. 全局最大池化（Global Max Pooling）
global_max_pool = nn.AdaptiveMaxPool2d((1, 1))

# 应用池化
max_pooled = max_pool(x)
avg_pooled = avg_pool(x)
global_avg_pooled = global_avg_pool(x)
global_max_pooled = global_max_pool(x)

# 可视化
fig, axes = plt.subplots(2, 3, figsize=(15, 10))

axes[0, 0].imshow(x.squeeze().numpy(), cmap='viridis')
axes[0, 0].set_title('原始特征图 (8x8)')
axes[0, 0].axis('off')

axes[0, 1].imshow(max_pooled.squeeze().numpy(), cmap='viridis')
axes[0, 1].set_title('最大池化 (4x4)')
axes[0, 1].axis('off')

axes[0, 2].imshow(avg_pooled.squeeze().numpy(), cmap='viridis')
axes[0, 2].set_title('平均池化 (4x4)')
axes[0, 2].axis('off')

axes[1, 0].text(0.5, 0.5, f'全局平均池化\n{global_avg_pooled.item():.4f}',
                ha='center', va='center', fontsize=14)
axes[1, 0].axis('off')

axes[1, 1].text(0.5, 0.5, f'全局最大池化\n{global_max_pooled.item():.4f}',
                ha='center', va='center', fontsize=14)
axes[1, 1].axis('off')

axes[1, 2].axis('off')

plt.tight_layout()
plt.show()

print(f"原始形状: {x.shape}")
print(f"最大池化后: {max_pooled.shape}")
print(f"平均池化后: {avg_pooled.shape}")
print(f"全局平均池化后: {global_avg_pooled.shape}")
print(f"全局最大池化后: {global_max_pooled.shape}")
```

### 3.2 池化层的作用

```python
"""
池化层的作用演示
@author erik.zhou
"""
import torch
import torch.nn as nn

print("""
池化层的作用：

1. 降维：减少特征图的空间尺寸
   - 减少计算量
   - 减少参数量
   - 防止过拟合

2. 平移不变性：对输入的小幅平移具有鲁棒性
   - 最大池化保留最显著的特征
   - 平均池化保留整体信息

3. 增大感受野：每次池化后，后续层的感受野增大

4. 特征选择：
   - 最大池化：选择最强的激活
   - 平均池化：综合区域信息

选择建议：
- 图像分类：最大池化（保留显著特征）
- 图像分割：平均池化或不使用池化
- 目标检测：最大池化
- 全局特征：全局平均池化（替代全连接层）
""")

# 演示感受野增大
class ReceptiveFieldDemo(nn.Module):
    """感受野演示"""
    
    def __init__(self):
        super(ReceptiveFieldDemo, self).__init__()
        self.conv1 = nn.Conv2d(1, 1, kernel_size=3, padding=1)
        self.pool1 = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(1, 1, kernel_size=3, padding=1)
        self.pool2 = nn.MaxPool2d(2, 2)
    
    def forward(self, x):
        print(f"输入: {x.shape}, 感受野: 1x1")
        
        x = self.conv1(x)
        print(f"Conv1后: {x.shape}, 感受野: 3x3")
        
        x = self.pool1(x)
        print(f"Pool1后: {x.shape}, 感受野: 6x6")
        
        x = self.conv2(x)
        print(f"Conv2后: {x.shape}, 感受野: 10x10")
        
        x = self.pool2(x)
        print(f"Pool2后: {x.shape}, 感受野: 20x20")
        
        return x

model = ReceptiveFieldDemo()
x = torch.randn(1, 1, 32, 32)
_ = model(x)
```

---

## 4. 经典架构

### 4.1 LeNet-5

```python
"""
LeNet-5实现（1998年）
@author erik.zhou
"""
import torch
import torch.nn as nn

class LeNet5(nn.Module):
    """LeNet-5网络"""
    
    def __init__(self, num_classes=10):
        super(LeNet5, self).__init__()
        
        self.features = nn.Sequential(
            # C1: 卷积层
            nn.Conv2d(1, 6, kernel_size=5, padding=2),
            nn.Tanh(),
            # S2: 池化层
            nn.AvgPool2d(kernel_size=2, stride=2),
            # C3: 卷积层
            nn.Conv2d(6, 16, kernel_size=5),
            nn.Tanh(),
            # S4: 池化层
            nn.AvgPool2d(kernel_size=2, stride=2),
        )
        
        self.classifier = nn.Sequential(
            # C5: 全连接层
            nn.Linear(16 * 5 * 5, 120),
            nn.Tanh(),
            # F6: 全连接层
            nn.Linear(120, 84),
            nn.Tanh(),
            # 输出层
            nn.Linear(84, num_classes)
        )
    
    def forward(self, x):
        x = self.features(x)
        x = x.view(x.size(0), -1)
        x = self.classifier(x)
        return x

# 创建模型
model = LeNet5()
print("LeNet-5结构:")
print(model)

# 测试
x = torch.randn(1, 1, 28, 28)
output = model(x)
print(f"\n输入: {x.shape}")
print(f"输出: {output.shape}")

# 参数量
total_params = sum(p.numel() for p in model.parameters())
print(f"总参数量: {total_params:,}")
```

### 4.2 AlexNet

```python
"""
AlexNet实现（2012年）
@author erik.zhou
"""
import torch
import torch.nn as nn

class AlexNet(nn.Module):
    """AlexNet网络"""
    
    def __init__(self, num_classes=1000):
        super(AlexNet, self).__init__()
        
        self.features = nn.Sequential(
            # Conv1
            nn.Conv2d(3, 96, kernel_size=11, stride=4, padding=2),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),
            
            # Conv2
            nn.Conv2d(96, 256, kernel_size=5, padding=2),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),
            
            # Conv3
            nn.Conv2d(256, 384, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            
            # Conv4
            nn.Conv2d(384, 384, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            
            # Conv5
            nn.Conv2d(384, 256, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),
        )
        
        self.avgpool = nn.AdaptiveAvgPool2d((6, 6))
        
        self.classifier = nn.Sequential(
            nn.Dropout(p=0.5),
            nn.Linear(256 * 6 * 6, 4096),
            nn.ReLU(inplace=True),
            nn.Dropout(p=0.5),
            nn.Linear(4096, 4096),
            nn.ReLU(inplace=True),
            nn.Linear(4096, num_classes),
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.avgpool(x)
        x = torch.flatten(x, 1)
        x = self.classifier(x)
        return x

# 创建模型
model = AlexNet(num_classes=1000)

# 测试
x = torch.randn(1, 3, 224, 224)
output = model(x)

print(f"输入: {x.shape}")
print(f"输出: {output.shape}")

total_params = sum(p.numel() for p in model.parameters())
print(f"总参数量: {total_params:,}")

print("""
AlexNet的创新点：
1. 使用ReLU激活函数（替代Tanh）
2. 使用Dropout防止过拟合
3. 使用数据增强
4. 使用GPU加速训练
5. 使用局部响应归一化（LRN）
""")
```

### 4.3 VGGNet

```python
"""
VGGNet实现（2014年）
@author erik.zhou
"""
import torch
import torch.nn as nn

class VGG16(nn.Module):
    """VGG16网络"""
    
    def __init__(self, num_classes=1000):
        super(VGG16, self).__init__()
        
        self.features = nn.Sequential(
            # Block 1
            nn.Conv2d(3, 64, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(64, 64, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=2, stride=2),
            
            # Block 2
            nn.Conv2d(64, 128, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(128, 128, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=2, stride=2),
            
            # Block 3
            nn.Conv2d(128, 256, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(256, 256, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(256, 256, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=2, stride=2),
            
            # Block 4
            nn.Conv2d(256, 512, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(512, 512, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(512, 512, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=2, stride=2),
            
            # Block 5
            nn.Conv2d(512, 512, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(512, 512, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.Conv2d(512, 512, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=2, stride=2),
        )
        
        self.avgpool = nn.AdaptiveAvgPool2d((7, 7))
        
        self.classifier = nn.Sequential(
            nn.Linear(512 * 7 * 7, 4096),
            nn.ReLU(inplace=True),
            nn.Dropout(p=0.5),
            nn.Linear(4096, 4096),
            nn.ReLU(inplace=True),
            nn.Dropout(p=0.5),
            nn.Linear(4096, num_classes),
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.avgpool(x)
        x = torch.flatten(x, 1)
        x = self.classifier(x)
        return x

# 创建模型
model = VGG16()

# 测试
x = torch.randn(1, 3, 224, 224)
output = model(x)

print(f"输入: {x.shape}")
print(f"输出: {output.shape}")

total_params = sum(p.numel() for p in model.parameters())
print(f"总参数量: {total_params:,}")

print("""
VGGNet的特点：
1. 使用小卷积核（3x3）
2. 网络更深（16-19层）
3. 结构简单规整
4. 参数量大（138M）
""")
```

### 4.4 ResNet

```python
"""
ResNet实现（2015年）
@author erik.zhou
"""
import torch
import torch.nn as nn

class ResidualBlock(nn.Module):
    """残差块"""
    
    def __init__(self, in_channels, out_channels, stride=1):
        super(ResidualBlock, self).__init__()
        
        self.conv1 = nn.Conv2d(in_channels, out_channels, 
                               kernel_size=3, stride=stride, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)
        self.relu = nn.ReLU(inplace=True)
        
        self.conv2 = nn.Conv2d(out_channels, out_channels,
                               kernel_size=3, stride=1, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)
        
        # 如果输入输出维度不同，需要调整shortcut
        self.shortcut = nn.Sequential()
        if stride != 1 or in_channels != out_channels:
            self.shortcut = nn.Sequential(
                nn.Conv2d(in_channels, out_channels, 
                         kernel_size=1, stride=stride, bias=False),
                nn.BatchNorm2d(out_channels)
            )
    
    def forward(self, x):
        identity = x
        
        out = self.conv1(x)
        out = self.bn1(out)
        out = self.relu(out)
        
        out = self.conv2(out)
        out = self.bn2(out)
        
        # 残差连接
        out += self.shortcut(identity)
        out = self.relu(out)
        
        return out

class ResNet18(nn.Module):
    """ResNet-18网络"""
    
    def __init__(self, num_classes=1000):
        super(ResNet18, self).__init__()
        
        self.conv1 = nn.Conv2d(3, 64, kernel_size=7, stride=2, padding=3, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        self.relu = nn.ReLU(inplace=True)
        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
        
        # 残差层
        self.layer1 = self._make_layer(64, 64, 2, stride=1)
        self.layer2 = self._make_layer(64, 128, 2, stride=2)
        self.layer3 = self._make_layer(128, 256, 2, stride=2)
        self.layer4 = self._make_layer(256, 512, 2, stride=2)
        
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512, num_classes)
    
    def _make_layer(self, in_channels, out_channels, num_blocks, stride):
        """构建残差层"""
        layers = []
        layers.append(ResidualBlock(in_channels, out_channels, stride))
        for _ in range(1, num_blocks):
            layers.append(ResidualBlock(out_channels, out_channels, 1))
        return nn.Sequential(*layers)
    
    def forward(self, x):
        x = self.conv1(x)
        x = self.bn1(x)
        x = self.relu(x)
        x = self.maxpool(x)
        
        x = self.layer1(x)
        x = self.layer2(x)
        x = self.layer3(x)
        x = self.layer4(x)
        
        x = self.avgpool(x)
        x = torch.flatten(x, 1)
        x = self.fc(x)
        
        return x

# 创建模型
model = ResNet18()

# 测试
x = torch.randn(1, 3, 224, 224)
output = model(x)

print(f"输入: {x.shape}")
print(f"输出: {output.shape}")

total_params = sum(p.numel() for p in model.parameters())
print(f"总参数量: {total_params:,}")

print("""
ResNet的创新点：
1. 残差连接：解决梯度消失问题
2. 可以训练非常深的网络（50/101/152层）
3. 使用批归一化
4. 使用全局平均池化替代全连接层
5. 参数量相对较少
""")
```


---

## 5. 图像分类

### 5.1 完整的图像分类流程

```python
"""
完整的图像分类训练流程
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader
import torchvision
import torchvision.transforms as transforms

# 数据预处理
transform_train = transforms.Compose([
    transforms.RandomCrop(32, padding=4),
    transforms.RandomHorizontalFlip(),
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010)),
])

transform_test = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010)),
])

# 加载CIFAR-10数据集（示例）
# trainset = torchvision.datasets.CIFAR10(root='./data', train=True, 
#                                          download=True, transform=transform_train)
# trainloader = DataLoader(trainset, batch_size=128, shuffle=True, num_workers=2)

# testset = torchvision.datasets.CIFAR10(root='./data', train=False,
#                                         download=True, transform=transform_test)
# testloader = DataLoader(testset, batch_size=100, shuffle=False, num_workers=2)

def train_model(model, trainloader, testloader, epochs=10):
    """训练模型"""
    device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
    model = model.to(device)
    
    criterion = nn.CrossEntropyLoss()
    optimizer = optim.SGD(model.parameters(), lr=0.1, momentum=0.9, weight_decay=5e-4)
    scheduler = optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max=epochs)
    
    for epoch in range(epochs):
        # 训练
        model.train()
        train_loss = 0
        correct = 0
        total = 0
        
        for batch_idx, (inputs, targets) in enumerate(trainloader):
            inputs, targets = inputs.to(device), targets.to(device)
            
            optimizer.zero_grad()
            outputs = model(inputs)
            loss = criterion(outputs, targets)
            loss.backward()
            optimizer.step()
            
            train_loss += loss.item()
            _, predicted = outputs.max(1)
            total += targets.size(0)
            correct += predicted.eq(targets).sum().item()
        
        train_acc = 100. * correct / total
        
        # 验证
        model.eval()
        test_loss = 0
        correct = 0
        total = 0
        
        with torch.no_grad():
            for inputs, targets in testloader:
                inputs, targets = inputs.to(device), targets.to(device)
                outputs = model(inputs)
                loss = criterion(outputs, targets)
                
                test_loss += loss.item()
                _, predicted = outputs.max(1)
                total += targets.size(0)
                correct += predicted.eq(targets).sum().item()
        
        test_acc = 100. * correct / total
        
        print(f'Epoch: {epoch+1}/{epochs}')
        print(f'  Train Loss: {train_loss/(batch_idx+1):.3f} | Train Acc: {train_acc:.2f}%')
        print(f'  Test Loss: {test_loss/len(testloader):.3f} | Test Acc: {test_acc:.2f}%')
        
        scheduler.step()

print("图像分类训练流程示例")
```

---

## 6. 数据增强

```python
"""
数据增强技术
@author erik.zhou
"""
import torchvision.transforms as transforms

# 常用数据增强方法
data_augmentation = transforms.Compose([
    # 几何变换
    transforms.RandomCrop(32, padding=4),
    transforms.RandomHorizontalFlip(p=0.5),
    transforms.RandomRotation(degrees=15),
    transforms.RandomAffine(degrees=0, translate=(0.1, 0.1)),
    
    # 颜色变换
    transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.1),
    transforms.RandomGrayscale(p=0.1),
    
    # 其他
    transforms.RandomErasing(p=0.5, scale=(0.02, 0.33)),
    
    # 转换为张量
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225]),
])

print("""
数据增强的作用：
1. 增加训练数据多样性
2. 提高模型泛化能力
3. 防止过拟合
4. 提高模型鲁棒性

常用方法：
- 几何变换：裁剪、翻转、旋转、缩放
- 颜色变换：亮度、对比度、饱和度调整
- 噪声注入：高斯噪声、椒盐噪声
- 混合方法：Mixup、CutMix
""")
```

---

## 7. 迁移学习

```python
"""
迁移学习实现
@author erik.zhou
"""
import torch
import torch.nn as nn
import torchvision.models as models

# 方法1：使用预训练模型作为特征提取器
def create_feature_extractor(num_classes=10):
    """创建特征提取器"""
    # 加载预训练的ResNet18
    model = models.resnet18(pretrained=True)
    
    # 冻结所有层
    for param in model.parameters():
        param.requires_grad = False
    
    # 替换最后的全连接层
    num_features = model.fc.in_features
    model.fc = nn.Linear(num_features, num_classes)
    
    return model

# 方法2：微调整个网络
def create_finetuned_model(num_classes=10):
    """创建微调模型"""
    # 加载预训练模型
    model = models.resnet18(pretrained=True)
    
    # 替换最后的全连接层
    num_features = model.fc.in_features
    model.fc = nn.Linear(num_features, num_classes)
    
    # 所有层都可训练，但使用不同的学习率
    return model

# 使用不同学习率
def get_optimizer_with_different_lr(model):
    """为不同层设置不同学习率"""
    # 预训练层使用较小学习率
    pretrained_params = []
    new_params = []
    
    for name, param in model.named_parameters():
        if 'fc' in name:
            new_params.append(param)
        else:
            pretrained_params.append(param)
    
    optimizer = torch.optim.SGD([
        {'params': pretrained_params, 'lr': 0.001},
        {'params': new_params, 'lr': 0.01}
    ], momentum=0.9)
    
    return optimizer

print("""
迁移学习策略：

1. 特征提取：
   - 冻结预训练层
   - 只训练新添加的层
   - 适用于数据量小的情况

2. 微调：
   - 解冻部分或全部预训练层
   - 使用较小学习率
   - 适用于数据量中等的情况

3. 从头训练：
   - 不使用预训练权重
   - 适用于数据量大且任务差异大的情况

选择建议：
- 数据量小 + 任务相似 → 特征提取
- 数据量中等 + 任务相似 → 微调
- 数据量大 + 任务差异大 → 从头训练
""")
```

---

## 8-10. 高级主题（简要）

```python
"""
CNN高级主题概览
@author erik.zhou
"""

print("""
8. 目标检测：
   - YOLO系列：实时目标检测
   - Faster R-CNN：两阶段检测
   - SSD：单阶段检测
   - 关键技术：锚框、NMS、IoU

9. 图像分割：
   - 语义分割：FCN、U-Net、DeepLab
   - 实例分割：Mask R-CNN
   - 全景分割：Panoptic FPN
   - 关键技术：上采样、跳跃连接

10. 最佳实践：
    - 数据预处理标准化
    - 使用预训练模型
    - 合理的数据增强
    - 学习率调度
    - 早停和模型检查点
    - 混合精度训练
    - 模型集成
""")
```

---

## 📝 学习检查清单

- [ ] 理解CNN的基本原理和优势
- [ ] 掌握卷积层和池化层的作用
- [ ] 能够实现经典CNN架构
- [ ] 掌握图像分类任务流程
- [ ] 理解数据增强的重要性
- [ ] 能够使用迁移学习
- [ ] 了解目标检测和图像分割
- [ ] 掌握CNN可视化技术
- [ ] 理解CNN的最佳实践

## 🔗 相关资源

- [CS231n: CNN for Visual Recognition](http://cs231n.stanford.edu/)
- [PyTorch官方教程](https://pytorch.org/tutorials/)
- [深度学习计算机视觉](https://www.deeplearningbook.org/)
- [Papers with Code](https://paperswithcode.com/)

---

**@author erik.zhou**  
**最后更新**: 2026-02-13
