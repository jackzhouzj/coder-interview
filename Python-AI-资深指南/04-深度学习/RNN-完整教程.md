# RNN循环神经网络完整教程

> 掌握循环神经网络，处理序列数据
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: PyTorch 2.0+  
**学习难度**: ⭐⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐  
**预计学习时长**: 25-35小时

## 🎯 学习目标

- [ ] 理解RNN的基本原理和结构
- [ ] 掌握LSTM和GRU架构
- [ ] 能够处理序列数据
- [ ] 掌握时间序列预测
- [ ] 理解序列到序列模型
- [ ] 能够实现文本生成
- [ ] 理解注意力机制在RNN中的应用
- [ ] 掌握RNN的优化技巧

## 📖 目录

1. [RNN基础](#1-rnn基础)
2. [RNN实现](#2-rnn实现)
3. [LSTM](#3-lstm)
4. [GRU](#4-gru)
5. [双向RNN](#5-双向rnn)
6. [序列到序列](#6-序列到序列)
7. [时间序列预测](#7-时间序列预测)
8. [文本生成](#8-文本生成)
9. [注意力机制](#9-注意力机制)
10. [最佳实践](#10-最佳实践)

---

## 1. RNN基础

### 1.1 为什么需要RNN

```python
"""
RNN vs 前馈神经网络
@author erik.zhou
"""
import torch
import torch.nn as nn

print("""
序列数据的特点：

1. 时间依赖性：
   - 当前输出依赖历史信息
   - 例如：预测下一个词需要看前面的词

2. 变长输入：
   - 序列长度不固定
   - 例如：不同长度的句子

3. 参数共享：
   - 相同的模式在不同位置出现
   - 例如：语法规则在句子各处适用

前馈网络的局限：
- 固定输入大小
- 无法处理时序信息
- 参数量随序列长度增长

RNN的优势：
- 处理变长序列
- 共享参数
- 记忆历史信息
- 适合序列建模

应用场景：
- 自然语言处理：机器翻译、文本生成
- 语音识别：语音转文字
- 时间序列：股票预测、天气预报
- 视频分析：动作识别
""")
```

### 1.2 RNN结构

```python
"""
RNN基本结构
@author erik.zhou
"""
import torch
import torch.nn as nn

class SimpleRNN(nn.Module):
    """简单RNN实现"""
    
    def __init__(self, input_size, hidden_size, output_size):
        """
        初始化
        
        Args:
            input_size: 输入维度
            hidden_size: 隐藏层维度
            output_size: 输出维度
        """
        super(SimpleRNN, self).__init__()
        
        self.hidden_size = hidden_size
        
        # 输入到隐藏层
        self.i2h = nn.Linear(input_size + hidden_size, hidden_size)
        # 隐藏层到输出
        self.i2o = nn.Linear(input_size + hidden_size, output_size)
        # 激活函数
        self.tanh = nn.Tanh()
        self.softmax = nn.LogSoftmax(dim=1)
    
    def forward(self, x, hidden):
        """
        前向传播
        
        Args:
            x: 输入 (batch, input_size)
            hidden: 隐藏状态 (batch, hidden_size)
        
        Returns:
            output: 输出 (batch, output_size)
            hidden: 新的隐藏状态 (batch, hidden_size)
        """
        # 拼接输入和隐藏状态
        combined = torch.cat((x, hidden), 1)
        
        # 计算新的隐藏状态
        hidden = self.tanh(self.i2h(combined))
        
        # 计算输出
        output = self.i2o(combined)
        output = self.softmax(output)
        
        return output, hidden
    
    def init_hidden(self, batch_size):
        """初始化隐藏状态"""
        return torch.zeros(batch_size, self.hidden_size)

# 测试
input_size = 10
hidden_size = 20
output_size = 5
batch_size = 3

rnn = SimpleRNN(input_size, hidden_size, output_size)

# 初始化隐藏状态
hidden = rnn.init_hidden(batch_size)

# 输入序列
x = torch.randn(batch_size, input_size)

# 前向传播
output, hidden = rnn(x, hidden)

print(f"输入形状: {x.shape}")
print(f"隐藏状态形状: {hidden.shape}")
print(f"输出形状: {output.shape}")

print("""
RNN的计算过程：

h_t = tanh(W_hh * h_{t-1} + W_xh * x_t + b_h)
y_t = W_hy * h_t + b_y

其中：
- h_t: 当前时刻的隐藏状态
- h_{t-1}: 上一时刻的隐藏状态
- x_t: 当前时刻的输入
- y_t: 当前时刻的输出
- W_hh, W_xh, W_hy: 权重矩阵
- b_h, b_y: 偏置
""")
```

### 1.3 RNN展开图

```python
"""
RNN时间展开
@author erik.zhou
"""

print("""
RNN时间展开示意图：

折叠形式（循环）：
    ┌─────┐
    │     │
x_t→│ RNN │→y_t
    │     │
    └──↑──┘
       │
      h_t

展开形式（时间序列）：
       h_0      h_1      h_2      h_3
        ↓        ↓        ↓        ↓
    ┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐
x_0→│ RNN │→│ RNN │→│ RNN │→│ RNN │
    └──↓──┘  └──↓──┘  └──↓──┘  └──↓──┘
      y_0      y_1      y_2      y_3

关键点：
1. 参数共享：所有时间步使用相同的权重
2. 信息传递：隐藏状态h在时间步之间传递
3. 序列处理：可以处理任意长度的序列

RNN的类型：
1. One-to-One: 标准神经网络
2. One-to-Many: 图像描述生成
3. Many-to-One: 情感分类
4. Many-to-Many (同步): 视频分类
5. Many-to-Many (异步): 机器翻译
""")
```

---

## 2. RNN实现

### 2.1 使用PyTorch的RNN

```python
"""
使用PyTorch内置RNN
@author erik.zhou
"""
import torch
import torch.nn as nn

class RNNModel(nn.Module):
    """使用PyTorch RNN的模型"""
    
    def __init__(self, input_size, hidden_size, num_layers, output_size):
        super(RNNModel, self).__init__()
        
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        # RNN层
        self.rnn = nn.RNN(
            input_size=input_size,
            hidden_size=hidden_size,
            num_layers=num_layers,
            batch_first=True  # 输入形状为(batch, seq, feature)
        )
        
        # 全连接层
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        """
        前向传播
        
        Args:
            x: (batch, seq_len, input_size)
        
        Returns:
            output: (batch, seq_len, output_size)
        """
        # 初始化隐藏状态
        h0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        
        # RNN前向传播
        out, _ = self.rnn(x, h0)  # out: (batch, seq_len, hidden_size)
        
        # 通过全连接层
        out = self.fc(out)  # (batch, seq_len, output_size)
        
        return out

# 创建模型
input_size = 10
hidden_size = 20
num_layers = 2
output_size = 5
batch_size = 3
seq_len = 7

model = RNNModel(input_size, hidden_size, num_layers, output_size)

# 测试
x = torch.randn(batch_size, seq_len, input_size)
output = model(x)

print(f"输入形状: {x.shape}")
print(f"输出形状: {output.shape}")

# 参数量
total_params = sum(p.numel() for p in model.parameters())
print(f"总参数量: {total_params:,}")
```

### 2.2 序列分类示例

```python
"""
使用RNN进行序列分类
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.optim as optim

class SequenceClassifier(nn.Module):
    """序列分类器"""
    
    def __init__(self, vocab_size, embedding_dim, hidden_size, num_classes):
        super(SequenceClassifier, self).__init__()
        
        # Embedding层
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        
        # RNN层
        self.rnn = nn.RNN(embedding_dim, hidden_size, batch_first=True)
        
        # 分类层
        self.fc = nn.Linear(hidden_size, num_classes)
    
    def forward(self, x):
        """
        前向传播
        
        Args:
            x: (batch, seq_len) - 词索引序列
        
        Returns:
            output: (batch, num_classes) - 分类结果
        """
        # Embedding
        embedded = self.embedding(x)  # (batch, seq_len, embedding_dim)
        
        # RNN
        _, hidden = self.rnn(embedded)  # hidden: (1, batch, hidden_size)
        
        # 使用最后的隐藏状态进行分类
        output = self.fc(hidden.squeeze(0))  # (batch, num_classes)
        
        return output

# 创建模型
vocab_size = 1000
embedding_dim = 50
hidden_size = 128
num_classes = 2

model = SequenceClassifier(vocab_size, embedding_dim, hidden_size, num_classes)

# 训练示例
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# 模拟数据
batch_size = 32
seq_len = 20
x = torch.randint(0, vocab_size, (batch_size, seq_len))
y = torch.randint(0, num_classes, (batch_size,))

# 训练一步
model.train()
optimizer.zero_grad()
output = model(x)
loss = criterion(output, y)
loss.backward()
optimizer.step()

print(f"输入形状: {x.shape}")
print(f"输出形状: {output.shape}")
print(f"损失: {loss.item():.4f}")
```

---

## 3. LSTM

### 3.1 LSTM原理

```python
"""
LSTM（Long Short-Term Memory）
@author erik.zhou
"""
import torch
import torch.nn as nn

print("""
LSTM的动机：

RNN的问题：
1. 梯度消失：长序列训练困难
2. 梯度爆炸：梯度不稳定
3. 长期依赖：难以记住早期信息

LSTM的解决方案：
1. 细胞状态（Cell State）：长期记忆
2. 门控机制：控制信息流动
   - 遗忘门（Forget Gate）：决定丢弃什么信息
   - 输入门（Input Gate）：决定存储什么新信息
   - 输出门（Output Gate）：决定输出什么信息

LSTM的计算过程：

1. 遗忘门：f_t = σ(W_f · [h_{t-1}, x_t] + b_f)
2. 输入门：i_t = σ(W_i · [h_{t-1}, x_t] + b_i)
3. 候选值：C̃_t = tanh(W_C · [h_{t-1}, x_t] + b_C)
4. 更新细胞状态：C_t = f_t * C_{t-1} + i_t * C̃_t
5. 输出门：o_t = σ(W_o · [h_{t-1}, x_t] + b_o)
6. 隐藏状态：h_t = o_t * tanh(C_t)

其中：
- σ: Sigmoid函数（输出0-1）
- *: 逐元素乘法
- [h_{t-1}, x_t]: 拼接
""")
```

### 3.2 LSTM实现

```python
"""
LSTM实现
@author erik.zhou
"""
import torch
import torch.nn as nn

class LSTMModel(nn.Module):
    """LSTM模型"""
    
    def __init__(self, input_size, hidden_size, num_layers, output_size, dropout=0.2):
        super(LSTMModel, self).__init__()
        
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        # LSTM层
        self.lstm = nn.LSTM(
            input_size=input_size,
            hidden_size=hidden_size,
            num_layers=num_layers,
            batch_first=True,
            dropout=dropout if num_layers > 1 else 0
        )
        
        # 全连接层
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        """
        前向传播
        
        Args:
            x: (batch, seq_len, input_size)
        
        Returns:
            output: (batch, seq_len, output_size)
        """
        # 初始化隐藏状态和细胞状态
        h0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        c0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        
        # LSTM前向传播
        out, (hn, cn) = self.lstm(x, (h0, c0))
        
        # 通过全连接层
        out = self.fc(out)
        
        return out

# 创建模型
input_size = 10
hidden_size = 50
num_layers = 2
output_size = 5

model = LSTMModel(input_size, hidden_size, num_layers, output_size)

# 测试
batch_size = 4
seq_len = 10
x = torch.randn(batch_size, seq_len, input_size)
output = model(x)

print(f"输入形状: {x.shape}")
print(f"输出形状: {output.shape}")

total_params = sum(p.numel() for p in model.parameters())
print(f"总参数量: {total_params:,}")

print("""
LSTM vs RNN：

优势：
1. 解决梯度消失问题
2. 能够学习长期依赖
3. 训练更稳定

劣势：
1. 参数量更大（4倍于RNN）
2. 计算更慢
3. 更容易过拟合

使用建议：
- 长序列：使用LSTM
- 短序列：RNN可能足够
- 资源受限：考虑GRU
""")
```

---

## 4. GRU

### 4.1 GRU原理

```python
"""
GRU（Gated Recurrent Unit）
@author erik.zhou
"""
import torch
import torch.nn as nn

print("""
GRU的特点：

GRU vs LSTM：
1. 更简单：只有2个门（更新门、重置门）
2. 参数更少：约为LSTM的75%
3. 训练更快：计算量更小
4. 效果相当：在很多任务上与LSTM相当

GRU的计算过程：

1. 重置门：r_t = σ(W_r · [h_{t-1}, x_t])
2. 更新门：z_t = σ(W_z · [h_{t-1}, x_t])
3. 候选隐藏状态：h̃_t = tanh(W · [r_t * h_{t-1}, x_t])
4. 最终隐藏状态：h_t = (1 - z_t) * h_{t-1} + z_t * h̃_t

门的作用：
- 重置门：控制保留多少过去信息
- 更新门：控制更新多少新信息

GRU vs LSTM：
- GRU没有单独的细胞状态
- GRU的更新门同时控制遗忘和输入
- GRU更简洁，LSTM更灵活
""")
```

### 4.2 GRU实现

```python
"""
GRU实现
@author erik.zhou
"""
import torch
import torch.nn as nn

class GRUModel(nn.Module):
    """GRU模型"""
    
    def __init__(self, input_size, hidden_size, num_layers, output_size, dropout=0.2):
        super(GRUModel, self).__init__()
        
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        # GRU层
        self.gru = nn.GRU(
            input_size=input_size,
            hidden_size=hidden_size,
            num_layers=num_layers,
            batch_first=True,
            dropout=dropout if num_layers > 1 else 0
        )
        
        # 全连接层
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        """前向传播"""
        # 初始化隐藏状态
        h0 = torch.zeros(self.num_layers, x.size(0), self.hidden_size).to(x.device)
        
        # GRU前向传播
        out, _ = self.gru(x, h0)
        
        # 通过全连接层
        out = self.fc(out)
        
        return out

# 比较RNN、LSTM、GRU
models = {
    'RNN': nn.RNN(10, 50, 2, batch_first=True),
    'LSTM': nn.LSTM(10, 50, 2, batch_first=True),
    'GRU': nn.GRU(10, 50, 2, batch_first=True)
}

print("参数量对比:")
for name, model in models.items():
    params = sum(p.numel() for p in model.parameters())
    print(f"{name}: {params:,}")

print("""
选择建议：

1. RNN：
   - 短序列
   - 简单任务
   - 资源极度受限

2. LSTM：
   - 长序列
   - 复杂任务
   - 需要精确控制信息流

3. GRU：
   - 中等长度序列
   - 平衡性能和效率
   - 数据量较小时（参数少，不易过拟合）
""")
```


---

## 5. 双向RNN

### 5.1 双向RNN原理

```python
"""
双向RNN（Bidirectional RNN）
@author erik.zhou
"""
import torch
import torch.nn as nn

print("""
双向RNN的动机：

单向RNN的局限：
- 只能看到过去的信息
- 无法利用未来的上下文
- 例如：预测"我爱___"，看不到后面的词

双向RNN的优势：
- 同时利用过去和未来的信息
- 更好地理解上下文
- 适合非实时任务

应用场景：
- 文本分类：需要完整句子信息
- 命名实体识别：需要前后文
- 机器翻译：编码器部分
- 语音识别：离线识别

不适用场景：
- 实时生成任务
- 在线预测
- 流式处理
""")
```

### 5.2 双向RNN实现

```python
"""
双向RNN实现
@author erik.zhou
"""
import torch
import torch.nn as nn

class BiRNNModel(nn.Module):
    """双向RNN模型"""
    
    def __init__(self, input_size, hidden_size, num_layers, output_size):
        super(BiRNNModel, self).__init__()
        
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        # 双向RNN
        self.rnn = nn.RNN(
            input_size=input_size,
            hidden_size=hidden_size,
            num_layers=num_layers,
            batch_first=True,
            bidirectional=True  # 关键参数
        )
        
        # 全连接层（注意：hidden_size * 2）
        self.fc = nn.Linear(hidden_size * 2, output_size)
    
    def forward(self, x):
        """
        前向传播
        
        Args:
            x: (batch, seq_len, input_size)
        
        Returns:
            output: (batch, seq_len, output_size)
        """
        # 初始化隐藏状态（注意：num_layers * 2）
        h0 = torch.zeros(self.num_layers * 2, x.size(0), self.hidden_size).to(x.device)
        
        # 双向RNN前向传播
        out, _ = self.rnn(x, h0)  # out: (batch, seq_len, hidden_size * 2)
        
        # 全连接层
        out = self.fc(out)
        
        return out

# 测试
input_size = 10
hidden_size = 20
num_layers = 2
output_size = 5

model = BiRNNModel(input_size, hidden_size, num_layers, output_size)

batch_size = 3
seq_len = 7
x = torch.randn(batch_size, seq_len, input_size)
output = model(x)

print(f"输入形状: {x.shape}")
print(f"输出形状: {output.shape}")

total_params = sum(p.numel() for p in model.parameters())
print(f"总参数量: {total_params:,}")
```

### 5.3 双向LSTM应用

```python
"""
双向LSTM用于文本分类
@author erik.zhou
"""
import torch
import torch.nn as nn

class BiLSTMClassifier(nn.Module):
    """双向LSTM文本分类器"""
    
    def __init__(self, vocab_size, embedding_dim, hidden_size, num_classes):
        super(BiLSTMClassifier, self).__init__()
        
        # Embedding层
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        
        # 双向LSTM
        self.lstm = nn.LSTM(
            embedding_dim,
            hidden_size,
            batch_first=True,
            bidirectional=True
        )
        
        # 分类层
        self.fc = nn.Linear(hidden_size * 2, num_classes)
    
    def forward(self, x):
        """
        前向传播
        
        Args:
            x: (batch, seq_len) - 词索引
        
        Returns:
            output: (batch, num_classes)
        """
        # Embedding
        embedded = self.embedding(x)  # (batch, seq_len, embedding_dim)
        
        # 双向LSTM
        lstm_out, (hidden, cell) = self.lstm(embedded)
        
        # 拼接前向和后向的最后隐藏状态
        # hidden: (2, batch, hidden_size)
        forward_hidden = hidden[0]  # (batch, hidden_size)
        backward_hidden = hidden[1]  # (batch, hidden_size)
        combined = torch.cat([forward_hidden, backward_hidden], dim=1)
        
        # 分类
        output = self.fc(combined)
        
        return output

# 创建模型
vocab_size = 5000
embedding_dim = 100
hidden_size = 128
num_classes = 2

model = BiLSTMClassifier(vocab_size, embedding_dim, hidden_size, num_classes)

# 测试
batch_size = 16
seq_len = 30
x = torch.randint(0, vocab_size, (batch_size, seq_len))
output = model(x)

print(f"输入形状: {x.shape}")
print(f"输出形状: {output.shape}")

print("""
双向RNN的注意事项：

1. 参数量：
   - 双向RNN参数量是单向的2倍
   - 计算量也增加约2倍

2. 隐藏状态：
   - 前向和后向各有一个隐藏状态
   - 通常拼接使用：[h_forward; h_backward]

3. 使用场景：
   - 适合：文本分类、NER、情感分析
   - 不适合：文本生成、实时预测

4. 性能优化：
   - 使用pack_padded_sequence处理变长序列
   - 考虑使用单向RNN + 注意力机制
""")
```

---

## 6. 序列到序列

### 6.1 Seq2Seq架构

```python
"""
Seq2Seq（Sequence to Sequence）模型
@author erik.zhou
"""
import torch
import torch.nn as nn

class Encoder(nn.Module):
    """编码器"""
    
    def __init__(self, input_size, embedding_dim, hidden_size, num_layers):
        super(Encoder, self).__init__()
        
        self.embedding = nn.Embedding(input_size, embedding_dim)
        self.lstm = nn.LSTM(embedding_dim, hidden_size, num_layers, batch_first=True)
    
    def forward(self, x):
        """
        编码输入序列
        
        Args:
            x: (batch, seq_len)
        
        Returns:
            outputs: (batch, seq_len, hidden_size)
            hidden: (num_layers, batch, hidden_size)
            cell: (num_layers, batch, hidden_size)
        """
        embedded = self.embedding(x)
        outputs, (hidden, cell) = self.lstm(embedded)
        return outputs, hidden, cell

class Decoder(nn.Module):
    """解码器"""
    
    def __init__(self, output_size, embedding_dim, hidden_size, num_layers):
        super(Decoder, self).__init__()
        
        self.embedding = nn.Embedding(output_size, embedding_dim)
        self.lstm = nn.LSTM(embedding_dim, hidden_size, num_layers, batch_first=True)
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x, hidden, cell):
        """
        解码一个时间步
        
        Args:
            x: (batch, 1) - 当前输入
            hidden: (num_layers, batch, hidden_size)
            cell: (num_layers, batch, hidden_size)
        
        Returns:
            output: (batch, output_size)
            hidden: 新的隐藏状态
            cell: 新的细胞状态
        """
        embedded = self.embedding(x)  # (batch, 1, embedding_dim)
        output, (hidden, cell) = self.lstm(embedded, (hidden, cell))
        prediction = self.fc(output.squeeze(1))  # (batch, output_size)
        return prediction, hidden, cell
```

### 6.2 Seq2Seq完整实现

```python
"""
完整的Seq2Seq模型
@author erik.zhou
"""
import torch
import torch.nn as nn
import random

class Seq2Seq(nn.Module):
    """Seq2Seq模型"""
    
    def __init__(self, encoder, decoder, device):
        super(Seq2Seq, self).__init__()
        
        self.encoder = encoder
        self.decoder = decoder
        self.device = device
    
    def forward(self, src, trg, teacher_forcing_ratio=0.5):
        """
        前向传播
        
        Args:
            src: 源序列 (batch, src_len)
            trg: 目标序列 (batch, trg_len)
            teacher_forcing_ratio: 教师强制比例
        
        Returns:
            outputs: (batch, trg_len, output_size)
        """
        batch_size = src.size(0)
        trg_len = trg.size(1)
        trg_vocab_size = self.decoder.fc.out_features
        
        # 存储输出
        outputs = torch.zeros(batch_size, trg_len, trg_vocab_size).to(self.device)
        
        # 编码
        _, hidden, cell = self.encoder(src)
        
        # 解码器的第一个输入（<SOS>标记）
        input = trg[:, 0].unsqueeze(1)
        
        # 逐步解码
        for t in range(1, trg_len):
            output, hidden, cell = self.decoder(input, hidden, cell)
            outputs[:, t, :] = output
            
            # 教师强制：使用真实标签还是预测结果
            teacher_force = random.random() < teacher_forcing_ratio
            top1 = output.argmax(1).unsqueeze(1)
            input = trg[:, t].unsqueeze(1) if teacher_force else top1
        
        return outputs

# 创建模型
input_vocab_size = 5000
output_vocab_size = 5000
embedding_dim = 256
hidden_size = 512
num_layers = 2
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

encoder = Encoder(input_vocab_size, embedding_dim, hidden_size, num_layers)
decoder = Decoder(output_vocab_size, embedding_dim, hidden_size, num_layers)
model = Seq2Seq(encoder, decoder, device).to(device)

# 测试
batch_size = 4
src_len = 10
trg_len = 12

src = torch.randint(0, input_vocab_size, (batch_size, src_len)).to(device)
trg = torch.randint(0, output_vocab_size, (batch_size, trg_len)).to(device)

outputs = model(src, trg)
print(f"源序列形状: {src.shape}")
print(f"目标序列形状: {trg.shape}")
print(f"输出形状: {outputs.shape}")

print("""
Seq2Seq的关键概念：

1. 编码器-解码器架构：
   - 编码器：将输入序列编码为固定长度的向量
   - 解码器：根据编码向量生成输出序列

2. 教师强制（Teacher Forcing）：
   - 训练时使用真实标签作为下一步输入
   - 加速训练，但可能导致曝光偏差

3. 应用场景：
   - 机器翻译
   - 文本摘要
   - 对话系统
   - 代码生成

4. 局限性：
   - 信息瓶颈：固定长度向量
   - 长序列性能下降
   - 解决方案：注意力机制
""")
```

---

## 7. 时间序列预测

### 7.1 时间序列数据准备

```python
"""
时间序列数据准备
@author erik.zhou
"""
import torch
import numpy as np
from torch.utils.data import Dataset, DataLoader

class TimeSeriesDataset(Dataset):
    """时间序列数据集"""
    
    def __init__(self, data, seq_len, pred_len):
        """
        初始化
        
        Args:
            data: 时间序列数据 (n_samples,)
            seq_len: 输入序列长度
            pred_len: 预测长度
        """
        self.data = data
        self.seq_len = seq_len
        self.pred_len = pred_len
    
    def __len__(self):
        return len(self.data) - self.seq_len - self.pred_len + 1
    
    def __getitem__(self, idx):
        """
        获取一个样本
        
        Returns:
            x: 输入序列 (seq_len,)
            y: 目标序列 (pred_len,)
        """
        x = self.data[idx:idx + self.seq_len]
        y = self.data[idx + self.seq_len:idx + self.seq_len + self.pred_len]
        return torch.FloatTensor(x), torch.FloatTensor(y)

# 生成示例数据（正弦波 + 噪声）
np.random.seed(42)
t = np.linspace(0, 100, 1000)
data = np.sin(t) + 0.1 * np.random.randn(1000)

# 数据归一化
data_mean = data.mean()
data_std = data.std()
data_normalized = (data - data_mean) / data_std

# 创建数据集
seq_len = 50
pred_len = 10
dataset = TimeSeriesDataset(data_normalized, seq_len, pred_len)
dataloader = DataLoader(dataset, batch_size=32, shuffle=True)

# 查看一个批次
x_batch, y_batch = next(iter(dataloader))
print(f"输入批次形状: {x_batch.shape}")
print(f"目标批次形状: {y_batch.shape}")
```

### 7.2 LSTM时间序列预测模型

```python
"""
LSTM时间序列预测
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.optim as optim

class LSTMPredictor(nn.Module):
    """LSTM时间序列预测器"""
    
    def __init__(self, input_size=1, hidden_size=64, num_layers=2, output_size=1):
        super(LSTMPredictor, self).__init__()
        
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        # LSTM层
        self.lstm = nn.LSTM(
            input_size,
            hidden_size,
            num_layers,
            batch_first=True,
            dropout=0.2
        )
        
        # 全连接层
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        """
        前向传播
        
        Args:
            x: (batch, seq_len, input_size)
        
        Returns:
            output: (batch, output_size)
        """
        # LSTM
        lstm_out, _ = self.lstm(x)
        
        # 使用最后一个时间步的输出
        last_output = lstm_out[:, -1, :]
        
        # 预测
        output = self.fc(last_output)
        
        return output

# 训练函数
def train_model(model, dataloader, num_epochs=50):
    """训练模型"""
    criterion = nn.MSELoss()
    optimizer = optim.Adam(model.parameters(), lr=0.001)
    
    model.train()
    for epoch in range(num_epochs):
        total_loss = 0
        for x_batch, y_batch in dataloader:
            # 调整形状
            x_batch = x_batch.unsqueeze(-1)  # (batch, seq_len, 1)
            y_batch = y_batch[:, 0].unsqueeze(-1)  # (batch, 1) - 预测下一个值
            
            # 前向传播
            optimizer.zero_grad()
            predictions = model(x_batch)
            loss = criterion(predictions, y_batch)
            
            # 反向传播
            loss.backward()
            optimizer.step()
            
            total_loss += loss.item()
        
        if (epoch + 1) % 10 == 0:
            avg_loss = total_loss / len(dataloader)
            print(f"Epoch [{epoch+1}/{num_epochs}], Loss: {avg_loss:.4f}")

# 创建和训练模型
model = LSTMPredictor(input_size=1, hidden_size=64, num_layers=2, output_size=1)
train_model(model, dataloader, num_epochs=50)

print("""
时间序列预测的关键点：

1. 数据预处理：
   - 归一化/标准化
   - 滑动窗口切分
   - 处理缺失值

2. 模型选择：
   - 单变量：LSTM、GRU
   - 多变量：多输入LSTM
   - 长期预测：Transformer

3. 评估指标：
   - MAE（平均绝对误差）
   - RMSE（均方根误差）
   - MAPE（平均绝对百分比误差）

4. 常见问题：
   - 过拟合：使用Dropout、正则化
   - 梯度消失：使用LSTM/GRU
   - 长期依赖：增加层数、使用注意力
""")
```

---

## 8. 文本生成

### 8.1 字符级文本生成

```python
"""
字符级RNN文本生成
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.nn.functional as F

class CharRNN(nn.Module):
    """字符级RNN"""
    
    def __init__(self, vocab_size, embedding_dim, hidden_size, num_layers):
        super(CharRNN, self).__init__()
        
        self.hidden_size = hidden_size
        self.num_layers = num_layers
        
        # Embedding层
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        
        # LSTM层
        self.lstm = nn.LSTM(
            embedding_dim,
            hidden_size,
            num_layers,
            batch_first=True,
            dropout=0.3
        )
        
        # 输出层
        self.fc = nn.Linear(hidden_size, vocab_size)
    
    def forward(self, x, hidden=None):
        """
        前向传播
        
        Args:
            x: (batch, seq_len)
            hidden: 隐藏状态（可选）
        
        Returns:
            output: (batch, seq_len, vocab_size)
            hidden: 新的隐藏状态
        """
        # Embedding
        embedded = self.embedding(x)
        
        # LSTM
        if hidden is None:
            lstm_out, hidden = self.lstm(embedded)
        else:
            lstm_out, hidden = self.lstm(embedded, hidden)
        
        # 输出
        output = self.fc(lstm_out)
        
        return output, hidden
    
    def generate(self, start_char, char_to_idx, idx_to_char, length=100, temperature=1.0):
        """
        生成文本
        
        Args:
            start_char: 起始字符
            char_to_idx: 字符到索引的映射
            idx_to_char: 索引到字符的映射
            length: 生成长度
            temperature: 温度参数（控制随机性）
        
        Returns:
            generated_text: 生成的文本
        """
        self.eval()
        with torch.no_grad():
            # 初始化
            current_char = start_char
            generated_text = current_char
            hidden = None
            
            # 逐字符生成
            for _ in range(length):
                # 转换为索引
                x = torch.LongTensor([[char_to_idx[current_char]]])
                
                # 前向传播
                output, hidden = self(x, hidden)
                
                # 应用温度
                output = output.squeeze(0).squeeze(0) / temperature
                probs = F.softmax(output, dim=0)
                
                # 采样下一个字符
                next_idx = torch.multinomial(probs, 1).item()
                next_char = idx_to_char[next_idx]
                
                generated_text += next_char
                current_char = next_char
            
            return generated_text

# 示例：准备数据
text = "hello world! this is a simple example for character-level text generation."
chars = sorted(list(set(text)))
char_to_idx = {ch: i for i, ch in enumerate(chars)}
idx_to_char = {i: ch for i, ch in enumerate(chars)}

vocab_size = len(chars)
print(f"词汇表大小: {vocab_size}")
print(f"字符集: {chars}")
```

### 8.2 词级文本生成

```python
"""
词级RNN文本生成
@author erik.zhou
"""
import torch
import torch.nn as nn

class WordRNN(nn.Module):
    """词级RNN"""
    
    def __init__(self, vocab_size, embedding_dim, hidden_size, num_layers):
        super(WordRNN, self).__init__()
        
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        self.lstm = nn.LSTM(embedding_dim, hidden_size, num_layers, 
                           batch_first=True, dropout=0.3)
        self.fc = nn.Linear(hidden_size, vocab_size)
    
    def forward(self, x, hidden=None):
        embedded = self.embedding(x)
        if hidden is None:
            output, hidden = self.lstm(embedded)
        else:
            output, hidden = self.lstm(embedded, hidden)
        output = self.fc(output)
        return output, hidden

print("""
文本生成的关键技术：

1. 采样策略：
   - 贪心采样：选择概率最高的词
   - 随机采样：按概率分布采样
   - Top-k采样：从概率最高的k个词中采样
   - Top-p（Nucleus）采样：累积概率达到p时采样

2. 温度参数（Temperature）：
   - T < 1：更确定性，输出更保守
   - T = 1：标准采样
   - T > 1：更随机性，输出更多样

3. 训练技巧：
   - 使用预训练词向量
   - 梯度裁剪防止梯度爆炸
   - 学习率调度
   - Dropout防止过拟合

4. 评估指标：
   - 困惑度（Perplexity）
   - BLEU分数
   - 人工评估
""")

# 温度采样示例
def sample_with_temperature(logits, temperature=1.0):
    """
    使用温度参数采样
    
    Args:
        logits: 模型输出 (vocab_size,)
        temperature: 温度参数
    
    Returns:
        sampled_idx: 采样的索引
    """
    logits = logits / temperature
    probs = torch.softmax(logits, dim=0)
    sampled_idx = torch.multinomial(probs, 1).item()
    return sampled_idx

# Top-k采样示例
def top_k_sampling(logits, k=10):
    """
    Top-k采样
    
    Args:
        logits: 模型输出 (vocab_size,)
        k: 保留的top-k个候选
    
    Returns:
        sampled_idx: 采样的索引
    """
    top_k_logits, top_k_indices = torch.topk(logits, k)
    probs = torch.softmax(top_k_logits, dim=0)
    sampled_idx = top_k_indices[torch.multinomial(probs, 1)].item()
    return sampled_idx

# Top-p采样示例
def top_p_sampling(logits, p=0.9):
    """
    Top-p（Nucleus）采样
    
    Args:
        logits: 模型输出 (vocab_size,)
        p: 累积概率阈值
    
    Returns:
        sampled_idx: 采样的索引
    """
    probs = torch.softmax(logits, dim=0)
    sorted_probs, sorted_indices = torch.sort(probs, descending=True)
    cumsum_probs = torch.cumsum(sorted_probs, dim=0)
    
    # 找到累积概率超过p的位置
    mask = cumsum_probs <= p
    mask[0] = True  # 至少保留一个
    
    filtered_probs = sorted_probs[mask]
    filtered_indices = sorted_indices[mask]
    
    # 重新归一化并采样
    filtered_probs = filtered_probs / filtered_probs.sum()
    sampled_idx = filtered_indices[torch.multinomial(filtered_probs, 1)].item()
    return sampled_idx
```

---

## 9. 注意力机制

### 9.1 注意力机制在RNN中的应用

```python
"""
RNN + 注意力机制
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.nn.functional as F

class Attention(nn.Module):
    """注意力层"""
    
    def __init__(self, hidden_size):
        super(Attention, self).__init__()
        self.hidden_size = hidden_size
        
        # 注意力权重计算
        self.attn = nn.Linear(hidden_size * 2, hidden_size)
        self.v = nn.Linear(hidden_size, 1, bias=False)
    
    def forward(self, hidden, encoder_outputs):
        """
        计算注意力
        
        Args:
            hidden: 解码器隐藏状态 (batch, hidden_size)
            encoder_outputs: 编码器输出 (batch, seq_len, hidden_size)
        
        Returns:
            context: 上下文向量 (batch, hidden_size)
            attention_weights: 注意力权重 (batch, seq_len)
        """
        seq_len = encoder_outputs.size(1)
        
        # 重复hidden以匹配encoder_outputs的形状
        hidden = hidden.unsqueeze(1).repeat(1, seq_len, 1)
        
        # 计算注意力分数
        energy = torch.tanh(self.attn(torch.cat([hidden, encoder_outputs], dim=2)))
        attention = self.v(energy).squeeze(2)
        
        # Softmax归一化
        attention_weights = F.softmax(attention, dim=1)
        
        # 计算上下文向量
        context = torch.bmm(attention_weights.unsqueeze(1), encoder_outputs)
        context = context.squeeze(1)
        
        return context, attention_weights

class AttentionDecoder(nn.Module):
    """带注意力的解码器"""
    
    def __init__(self, output_size, embedding_dim, hidden_size, num_layers):
        super(AttentionDecoder, self).__init__()
        
        self.embedding = nn.Embedding(output_size, embedding_dim)
        self.attention = Attention(hidden_size)
        self.lstm = nn.LSTM(embedding_dim + hidden_size, hidden_size, 
                           num_layers, batch_first=True)
        self.fc = nn.Linear(hidden_size, output_size)
    
    def forward(self, x, hidden, cell, encoder_outputs):
        """
        前向传播
        
        Args:
            x: 当前输入 (batch, 1)
            hidden: 隐藏状态
            cell: 细胞状态
            encoder_outputs: 编码器输出 (batch, seq_len, hidden_size)
        
        Returns:
            output: 预测 (batch, output_size)
            hidden: 新的隐藏状态
            cell: 新的细胞状态
            attention_weights: 注意力权重
        """
        # Embedding
        embedded = self.embedding(x)  # (batch, 1, embedding_dim)
        
        # 计算注意力
        context, attention_weights = self.attention(hidden[-1], encoder_outputs)
        
        # 拼接embedding和context
        lstm_input = torch.cat([embedded, context.unsqueeze(1)], dim=2)
        
        # LSTM
        output, (hidden, cell) = self.lstm(lstm_input, (hidden, cell))
        
        # 预测
        prediction = self.fc(output.squeeze(1))
        
        return prediction, hidden, cell, attention_weights

print("""
注意力机制的优势：

1. 解决信息瓶颈：
   - 不再依赖固定长度的上下文向量
   - 可以直接访问编码器的所有隐藏状态

2. 对齐能力：
   - 自动学习输入输出的对应关系
   - 可视化注意力权重

3. 性能提升：
   - 长序列性能显著提升
   - 机器翻译质量改善

4. 应用场景：
   - Seq2Seq任务
   - 机器翻译
   - 文本摘要
   - 图像描述生成
""")
```

### 9.2 注意力可视化

```python
"""
注意力权重可视化
@author erik.zhou
"""
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

def visualize_attention(attention_weights, source_tokens, target_tokens):
    """
    可视化注意力权重
    
    Args:
        attention_weights: (target_len, source_len)
        source_tokens: 源序列词列表
        target_tokens: 目标序列词列表
    """
    plt.figure(figsize=(12, 8))
    
    sns.heatmap(attention_weights,
                xticklabels=source_tokens,
                yticklabels=target_tokens,
                cmap='YlOrRd',
                cbar_kws={'label': '注意力权重'})
    
    plt.xlabel('源序列（Source）')
    plt.ylabel('目标序列（Target）')
    plt.title('注意力权重热力图')
    plt.tight_layout()
    plt.show()

# 示例
source_tokens = ['I', 'love', 'deep', 'learning']
target_tokens = ['我', '爱', '深度', '学习']

# 模拟注意力权重
attention_weights = np.array([
    [0.8, 0.1, 0.05, 0.05],
    [0.1, 0.7, 0.1, 0.1],
    [0.05, 0.1, 0.7, 0.15],
    [0.05, 0.1, 0.15, 0.7]
])

visualize_attention(attention_weights, source_tokens, target_tokens)

print("""
注意力权重解读：

1. 对角线高权重：
   - 表示词对词的对齐
   - 例如："I" -> "我"

2. 非对角线权重：
   - 表示一对多或多对一的关系
   - 例如："deep learning" -> "深度学习"

3. 权重分布：
   - 集中：模型确定对齐关系
   - 分散：模型不确定，需要多个词的信息
""")
```

---

## 10. 最佳实践

### 10.1 RNN训练技巧

```python
"""
RNN训练最佳实践
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.optim as optim

class RNNTrainer:
    """RNN训练器"""
    
    def __init__(self, model, device='cpu'):
        self.model = model.to(device)
        self.device = device
    
    def train_epoch(self, dataloader, criterion, optimizer, clip_grad=1.0):
        """
        训练一个epoch
        
        Args:
            dataloader: 数据加载器
            criterion: 损失函数
            optimizer: 优化器
            clip_grad: 梯度裁剪阈值
        
        Returns:
            avg_loss: 平均损失
        """
        self.model.train()
        total_loss = 0
        
        for batch_idx, (x, y) in enumerate(dataloader):
            x, y = x.to(self.device), y.to(self.device)
            
            # 前向传播
            optimizer.zero_grad()
            output = self.model(x)
            loss = criterion(output, y)
            
            # 反向传播
            loss.backward()
            
            # 梯度裁剪（重要！）
            torch.nn.utils.clip_grad_norm_(self.model.parameters(), clip_grad)
            
            optimizer.step()
            
            total_loss += loss.item()
        
        return total_loss / len(dataloader)
    
    def evaluate(self, dataloader, criterion):
        """评估模型"""
        self.model.eval()
        total_loss = 0
        
        with torch.no_grad():
            for x, y in dataloader:
                x, y = x.to(self.device), y.to(self.device)
                output = self.model(x)
                loss = criterion(output, y)
                total_loss += loss.item()
        
        return total_loss / len(dataloader)

print("""
RNN训练最佳实践：

1. 梯度裁剪（Gradient Clipping）：
   - 防止梯度爆炸
   - 通常设置为1.0或5.0
   - 使用torch.nn.utils.clip_grad_norm_

2. 学习率调度：
   - 使用学习率预热（Warmup）
   - 余弦退火（Cosine Annealing）
   - ReduceLROnPlateau

3. 正则化：
   - Dropout（0.2-0.5）
   - 权重衰减（Weight Decay）
   - 层归一化（Layer Normalization）

4. 优化器选择：
   - Adam：通用选择
   - AdamW：带权重衰减的Adam
   - SGD + Momentum：需要仔细调参

5. 批次大小：
   - 较小批次（16-64）通常效果更好
   - 使用梯度累积处理大批次

6. 序列长度：
   - 使用pack_padded_sequence处理变长序列
   - 截断过长序列
   - 使用分层softmax处理大词汇表
""")
```

### 10.2 常见问题和解决方案

```python
"""
RNN常见问题和解决方案
@author erik.zhou
"""

print("""
常见问题和解决方案：

1. 梯度消失/爆炸：
   问题：长序列训练困难
   解决：
   - 使用LSTM/GRU
   - 梯度裁剪
   - 残差连接
   - 层归一化

2. 过拟合：
   问题：训练集表现好，验证集差
   解决：
   - 增加Dropout
   - 减少模型复杂度
   - 数据增强
   - 早停（Early Stopping）
   - 正则化

3. 训练速度慢：
   问题：RNN无法并行
   解决：
   - 使用CuDNN优化的LSTM
   - 减少序列长度
   - 使用更大的批次
   - 考虑使用Transformer

4. 长期依赖问题：
   问题：难以学习长距离关系
   解决：
   - 使用LSTM/GRU
   - 增加层数
   - 使用注意力机制
   - 使用Transformer

5. 内存不足：
   问题：长序列占用大量内存
   解决：
   - 减少批次大小
   - 截断序列
   - 使用梯度检查点
   - 混合精度训练

6. 生成质量差：
   问题：生成文本重复或不连贯
   解决：
   - 使用更大的模型
   - 增加训练数据
   - 调整采样策略
   - 使用预训练模型
""")
```

### 10.3 性能优化

```python
"""
RNN性能优化技巧
@author erik.zhou
"""
import torch
import torch.nn as nn
from torch.nn.utils.rnn import pack_padded_sequence, pad_packed_sequence

class OptimizedLSTM(nn.Module):
    """优化的LSTM模型"""
    
    def __init__(self, vocab_size, embedding_dim, hidden_size, num_layers):
        super(OptimizedLSTM, self).__init__()
        
        self.embedding = nn.Embedding(vocab_size, embedding_dim)
        
        # 使用CuDNN优化的LSTM
        self.lstm = nn.LSTM(
            embedding_dim,
            hidden_size,
            num_layers,
            batch_first=True,
            dropout=0.3,
            bidirectional=False
        )
        
        self.fc = nn.Linear(hidden_size, vocab_size)
    
    def forward(self, x, lengths):
        """
        前向传播（处理变长序列）
        
        Args:
            x: (batch, max_len)
            lengths: 每个序列的实际长度
        
        Returns:
            output: (batch, max_len, vocab_size)
        """
        # Embedding
        embedded = self.embedding(x)
        
        # Pack序列
        packed = pack_padded_sequence(embedded, lengths, 
                                     batch_first=True, 
                                     enforce_sorted=False)
        
        # LSTM
        packed_output, _ = self.lstm(packed)
        
        # Unpack序列
        output, _ = pad_packed_sequence(packed_output, batch_first=True)
        
        # 输出层
        output = self.fc(output)
        
        return output

print("""
性能优化技巧：

1. 使用pack_padded_sequence：
   - 跳过padding位置的计算
   - 显著提升训练速度
   - 减少内存使用

2. 混合精度训练：
   - 使用torch.cuda.amp
   - 减少内存占用
   - 加速训练（需要支持的GPU）

3. 数据加载优化：
   - 使用多进程DataLoader
   - 预取数据（prefetch）
   - 固定内存（pin_memory）

4. 模型并行：
   - 层间并行
   - 数据并行
   - 使用DistributedDataParallel

5. 编译优化：
   - 使用torch.jit.script
   - ONNX导出
   - TensorRT推理加速
""")
```

---

## 📝 学习检查清单

- [ ] 理解RNN的基本原理和结构
- [ ] 掌握LSTM和GRU架构
- [ ] 能够处理序列数据
- [ ] 掌握时间序列预测
- [ ] 理解序列到序列模型
- [ ] 能够实现文本生成
- [ ] 理解注意力机制在RNN中的应用
- [ ] 掌握RNN的优化技巧
- [ ] 了解双向RNN的应用场景
- [ ] 能够解决RNN训练中的常见问题

## 🔗 相关资源

- [Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)
- [The Unreasonable Effectiveness of RNN](http://karpathy.github.io/2015/05/21/rnn-effectiveness/)
- [PyTorch RNN Tutorial](https://pytorch.org/tutorials/intermediate/char_rnn_classification_tutorial.html)
- [Sequence Models - Coursera](https://www.coursera.org/learn/nlp-sequence-models)

---

**@author erik.zhou**  
**最后更新**: 2026-02-13
