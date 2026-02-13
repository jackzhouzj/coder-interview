# Transformer完整教程

> 掌握Transformer架构，理解现代AI的核心技术
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: PyTorch 2.0+  
**学习难度**: ⭐⭐⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐⭐  
**预计学习时长**: 30-40小时

## 🎯 学习目标

- [ ] 理解Transformer的核心架构
- [ ] 掌握Self-Attention机制
- [ ] 理解Multi-Head Attention
- [ ] 掌握位置编码原理
- [ ] 能够实现完整的Transformer
- [ ] 理解BERT和GPT架构
- [ ] 掌握Transformer的应用场景
- [ ] 了解Transformer的优化技术

## 📖 目录

1. [Transformer概述](#1-transformer概述)
2. [注意力机制](#2-注意力机制)
3. [Self-Attention](#3-self-attention)
4. [Multi-Head Attention](#4-multi-head-attention)
5. [位置编码](#5-位置编码)
6. [Transformer架构](#6-transformer架构)
7. [BERT](#7-bert)
8. [GPT](#8-gpt)
9. [应用场景](#9-应用场景)
10. [最佳实践](#10-最佳实践)

---

## 1. Transformer概述

### 1.1 为什么需要Transformer

```python
"""
Transformer vs RNN/CNN
@author erik.zhou
"""
import torch
import torch.nn as nn

print("""
传统序列模型的问题：

RNN/LSTM的局限：
1. 顺序计算，无法并行
2. 长距离依赖问题
3. 梯度消失/爆炸
4. 训练速度慢

CNN的局限：
1. 感受野有限
2. 难以捕获长距离依赖
3. 需要多层堆叠

Transformer的优势：
1. 完全并行计算
2. 直接建模长距离依赖
3. 训练速度快
4. 效果更好
5. 可扩展性强

应用领域：
- NLP：机器翻译、文本生成、问答系统
- CV：图像分类、目标检测、图像生成
- 多模态：图文匹配、视频理解
- 其他：蛋白质结构预测、音乐生成
""")
```

### 1.2 Transformer架构图

```python
"""
Transformer整体架构
@author erik.zhou
"""

architecture = """
Transformer架构（Encoder-Decoder）：

输入序列
    ↓
[Embedding + Positional Encoding]
    ↓
┌─────────────────────────────────┐
│         Encoder (Nx)            │
│  ┌───────────────────────────┐  │
│  │  Multi-Head Attention     │  │
│  └───────────────────────────┘  │
│            ↓                     │
│  ┌───────────────────────────┐  │
│  │  Add & Norm               │  │
│  └───────────────────────────┘  │
│            ↓                     │
│  ┌───────────────────────────┐  │
│  │  Feed Forward             │  │
│  └───────────────────────────┘  │
│            ↓                     │
│  ┌───────────────────────────┐  │
│  │  Add & Norm               │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│         Decoder (Nx)            │
│  ┌───────────────────────────┐  │
│  │  Masked Multi-Head Attn   │  │
│  └───────────────────────────┘  │
│            ↓                     │
│  ┌───────────────────────────┐  │
│  │  Add & Norm               │  │
│  └───────────────────────────┘  │
│            ↓                     │
│  ┌───────────────────────────┐  │
│  │  Cross Attention          │  │
│  └───────────────────────────┘  │
│            ↓                     │
│  ┌───────────────────────────┐  │
│  │  Add & Norm               │  │
│  └───────────────────────────┘  │
│            ↓                     │
│  ┌───────────────────────────┐  │
│  │  Feed Forward             │  │
│  └───────────────────────────┘  │
│            ↓                     │
│  ┌───────────────────────────┐  │
│  │  Add & Norm               │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
    ↓
[Linear + Softmax]
    ↓
输出序列

核心组件：
1. Multi-Head Attention：多头注意力机制
2. Feed Forward：前馈神经网络
3. Add & Norm：残差连接和层归一化
4. Positional Encoding：位置编码
"""

print(architecture)
```

---

## 2. 注意力机制

### 2.1 注意力机制基础

```python
"""
注意力机制原理
@author erik.zhou
"""
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

def scaled_dot_product_attention(Q, K, V, mask=None):
    """
    缩放点积注意力
    
    Args:
        Q: Query矩阵 (batch, seq_len, d_k)
        K: Key矩阵 (batch, seq_len, d_k)
        V: Value矩阵 (batch, seq_len, d_v)
        mask: 掩码 (batch, seq_len, seq_len)
    
    Returns:
        output: 注意力输出 (batch, seq_len, d_v)
        attention_weights: 注意力权重 (batch, seq_len, seq_len)
    """
    d_k = Q.size(-1)
    
    # 计算注意力分数: Q @ K^T / sqrt(d_k)
    scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(d_k)
    
    # 应用掩码（可选）
    if mask is not None:
        scores = scores.masked_fill(mask == 0, -1e9)
    
    # Softmax归一化
    attention_weights = F.softmax(scores, dim=-1)
    
    # 加权求和
    output = torch.matmul(attention_weights, V)
    
    return output, attention_weights

# 示例
batch_size = 2
seq_len = 4
d_k = 8
d_v = 8

Q = torch.randn(batch_size, seq_len, d_k)
K = torch.randn(batch_size, seq_len, d_k)
V = torch.randn(batch_size, seq_len, d_v)

output, attention_weights = scaled_dot_product_attention(Q, K, V)

print(f"Query形状: {Q.shape}")
print(f"Key形状: {K.shape}")
print(f"Value形状: {V.shape}")
print(f"输出形状: {output.shape}")
print(f"注意力权重形状: {attention_weights.shape}")

print("\n注意力权重示例:")
print(attention_weights[0])

print("""
注意力机制的直观理解：

1. Query (Q): 查询向量，"我想要什么信息"
2. Key (K): 键向量，"我有什么信息"
3. Value (V): 值向量，"具体的信息内容"

计算过程：
1. 计算相似度: Q @ K^T（点积）
2. 缩放: 除以sqrt(d_k)（防止梯度消失）
3. Softmax: 归一化为概率分布
4. 加权求和: 注意力权重 @ V

为什么要缩放？
- 点积结果可能很大
- 导致softmax梯度很小
- 除以sqrt(d_k)可以稳定梯度
""")
```

### 2.2 注意力可视化

```python
"""
注意力权重可视化
@author erik.zhou
"""
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

def visualize_attention(attention_weights, tokens):
    """
    可视化注意力权重
    
    Args:
        attention_weights: 注意力权重矩阵
        tokens: 词元列表
    """
    plt.figure(figsize=(10, 8))
    
    sns.heatmap(attention_weights, 
                xticklabels=tokens,
                yticklabels=tokens,
                cmap='YlOrRd',
                annot=True,
                fmt='.2f',
                cbar_kws={'label': '注意力权重'})
    
    plt.xlabel('Key (被关注的词)')
    plt.ylabel('Query (查询的词)')
    plt.title('注意力权重热力图')
    plt.tight_layout()
    plt.show()

# 示例
tokens = ['我', '爱', '学习', 'AI']
attention_weights = np.array([
    [0.7, 0.1, 0.1, 0.1],
    [0.2, 0.6, 0.1, 0.1],
    [0.1, 0.3, 0.5, 0.1],
    [0.1, 0.1, 0.2, 0.6]
])

visualize_attention(attention_weights, tokens)
```

---

## 3. Self-Attention

### 3.1 Self-Attention实现

```python
"""
Self-Attention实现
@author erik.zhou
"""
import torch
import torch.nn as nn
import math

class SelfAttention(nn.Module):
    """Self-Attention层"""
    
    def __init__(self, d_model):
        """
        初始化
        
        Args:
            d_model: 模型维度
        """
        super(SelfAttention, self).__init__()
        
        self.d_model = d_model
        
        # Q, K, V的线性变换
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        
        # 输出线性变换
        self.W_o = nn.Linear(d_model, d_model)
    
    def forward(self, x, mask=None):
        """
        前向传播
        
        Args:
            x: 输入 (batch, seq_len, d_model)
            mask: 掩码 (batch, seq_len, seq_len)
        
        Returns:
            output: 输出 (batch, seq_len, d_model)
        """
        # 线性变换得到Q, K, V
        Q = self.W_q(x)  # (batch, seq_len, d_model)
        K = self.W_k(x)  # (batch, seq_len, d_model)
        V = self.W_v(x)  # (batch, seq_len, d_model)
        
        # 计算注意力分数
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_model)
        
        # 应用掩码
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
        
        # Softmax
        attention_weights = torch.softmax(scores, dim=-1)
        
        # 加权求和
        context = torch.matmul(attention_weights, V)
        
        # 输出线性变换
        output = self.W_o(context)
        
        return output, attention_weights

# 测试
batch_size = 2
seq_len = 5
d_model = 64

x = torch.randn(batch_size, seq_len, d_model)
self_attn = SelfAttention(d_model)

output, attention_weights = self_attn(x)

print(f"输入形状: {x.shape}")
print(f"输出形状: {output.shape}")
print(f"注意力权重形状: {attention_weights.shape}")

print("""
Self-Attention的特点：

1. Q, K, V都来自同一个输入
2. 每个位置都关注所有位置
3. 可以捕获长距离依赖
4. 完全并行计算

与传统注意力的区别：
- 传统注意力：Q来自decoder，K和V来自encoder
- Self-Attention：Q、K、V都来自同一个序列
""")
```

### 3.2 Masked Self-Attention

```python
"""
Masked Self-Attention（用于Decoder）
@author erik.zhou
"""
import torch
import torch.nn as nn

def create_causal_mask(seq_len):
    """
    创建因果掩码（下三角矩阵）
    
    Args:
        seq_len: 序列长度
    
    Returns:
        mask: 因果掩码 (seq_len, seq_len)
    """
    mask = torch.tril(torch.ones(seq_len, seq_len))
    return mask

# 示例
seq_len = 5
mask = create_causal_mask(seq_len)

print("因果掩码（1表示可见，0表示不可见）:")
print(mask)

print("""
Masked Self-Attention的作用：

在Decoder中，生成第i个词时：
- 只能看到前i-1个词
- 不能看到第i个及之后的词
- 防止信息泄露

掩码矩阵：
- 下三角为1（可见）
- 上三角为0（不可见）

示例：生成"我爱学习"
- 生成"我"：看不到任何词
- 生成"爱"：只能看到"我"
- 生成"学"：只能看到"我爱"
- 生成"习"：只能看到"我爱学"
""")
```


---

## 4. Multi-Head Attention

```python
"""
Multi-Head Attention实现
@author erik.zhou
"""
import torch
import torch.nn as nn
import math

class MultiHeadAttention(nn.Module):
    """多头注意力机制"""
    
    def __init__(self, d_model, num_heads):
        """
        初始化
        
        Args:
            d_model: 模型维度
            num_heads: 注意力头数
        """
        super(MultiHeadAttention, self).__init__()
        
        assert d_model % num_heads == 0, "d_model必须能被num_heads整除"
        
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        
        # Q, K, V的线性变换
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        
        # 输出线性变换
        self.W_o = nn.Linear(d_model, d_model)
    
    def split_heads(self, x):
        """
        分割成多个头
        
        Args:
            x: (batch, seq_len, d_model)
        
        Returns:
            (batch, num_heads, seq_len, d_k)
        """
        batch_size, seq_len, d_model = x.size()
        return x.view(batch_size, seq_len, self.num_heads, self.d_k).transpose(1, 2)
    
    def forward(self, Q, K, V, mask=None):
        """
        前向传播
        
        Args:
            Q, K, V: (batch, seq_len, d_model)
            mask: (batch, 1, seq_len, seq_len)
        
        Returns:
            output: (batch, seq_len, d_model)
        """
        batch_size = Q.size(0)
        
        # 线性变换
        Q = self.W_q(Q)
        K = self.W_k(K)
        V = self.W_v(V)
        
        # 分割成多个头
        Q = self.split_heads(Q)  # (batch, num_heads, seq_len, d_k)
        K = self.split_heads(K)
        V = self.split_heads(V)
        
        # 计算注意力
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
        
        attention_weights = torch.softmax(scores, dim=-1)
        context = torch.matmul(attention_weights, V)
        
        # 合并多个头
        context = context.transpose(1, 2).contiguous().view(batch_size, -1, self.d_model)
        
        # 输出线性变换
        output = self.W_o(context)
        
        return output, attention_weights

# 测试
d_model = 512
num_heads = 8
batch_size = 2
seq_len = 10

Q = K = V = torch.randn(batch_size, seq_len, d_model)
mha = MultiHeadAttention(d_model, num_heads)

output, attention_weights = mha(Q, K, V)

print(f"输入形状: {Q.shape}")
print(f"输出形状: {output.shape}")
print(f"注意力权重形状: {attention_weights.shape}")

print(f"""
Multi-Head Attention的优势：

1. 多个表示子空间：
   - 每个头关注不同的特征
   - 类似CNN的多个卷积核
   
2. 增强表达能力：
   - 捕获不同类型的依赖关系
   - 语法、语义、位置等

3. 参数效率：
   - 总参数量不变
   - d_model = num_heads * d_k

示例配置：
- d_model = 512
- num_heads = 8
- d_k = d_v = 64
""")
```

---

## 5. 位置编码

```python
"""
位置编码实现
@author erik.zhou
"""
import torch
import torch.nn as nn
import math

class PositionalEncoding(nn.Module):
    """位置编码"""
    
    def __init__(self, d_model, max_len=5000):
        """
        初始化
        
        Args:
            d_model: 模型维度
            max_len: 最大序列长度
        """
        super(PositionalEncoding, self).__init__()
        
        # 创建位置编码矩阵
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
        
        # PE(pos, 2i) = sin(pos / 10000^(2i/d_model))
        pe[:, 0::2] = torch.sin(position * div_term)
        # PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
        pe[:, 1::2] = torch.cos(position * div_term)
        
        pe = pe.unsqueeze(0)  # (1, max_len, d_model)
        self.register_buffer('pe', pe)
    
    def forward(self, x):
        """
        添加位置编码
        
        Args:
            x: (batch, seq_len, d_model)
        
        Returns:
            x + pe: (batch, seq_len, d_model)
        """
        x = x + self.pe[:, :x.size(1), :]
        return x

# 可视化位置编码
import matplotlib.pyplot as plt

d_model = 128
max_len = 100

pe_layer = PositionalEncoding(d_model, max_len)
pe = pe_layer.pe.squeeze(0).numpy()

plt.figure(figsize=(15, 5))
plt.imshow(pe.T, cmap='RdBu', aspect='auto')
plt.xlabel('位置')
plt.ylabel('维度')
plt.title('位置编码可视化')
plt.colorbar(label='编码值')
plt.tight_layout()
plt.show()

print("""
位置编码的作用：

1. 为什么需要？
   - Transformer没有循环结构
   - 无法感知词的顺序
   - 需要显式编码位置信息

2. 正弦位置编码的优势：
   - 可以处理任意长度序列
   - 相对位置关系可学习
   - 周期性模式

3. 其他位置编码方法：
   - 可学习位置编码
   - 相对位置编码
   - 旋转位置编码（RoPE）
""")
```

---

## 6. Transformer架构

```python
"""
完整的Transformer实现
@author erik.zhou
"""
import torch
import torch.nn as nn

class TransformerBlock(nn.Module):
    """Transformer块（Encoder层）"""
    
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        super(TransformerBlock, self).__init__()
        
        # Multi-Head Attention
        self.attention = MultiHeadAttention(d_model, num_heads)
        self.norm1 = nn.LayerNorm(d_model)
        self.dropout1 = nn.Dropout(dropout)
        
        # Feed Forward
        self.ff = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.ReLU(),
            nn.Dropout(dropout),
            nn.Linear(d_ff, d_model)
        )
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout2 = nn.Dropout(dropout)
    
    def forward(self, x, mask=None):
        # Multi-Head Attention + 残差连接 + LayerNorm
        attn_output, _ = self.attention(x, x, x, mask)
        x = self.norm1(x + self.dropout1(attn_output))
        
        # Feed Forward + 残差连接 + LayerNorm
        ff_output = self.ff(x)
        x = self.norm2(x + self.dropout2(ff_output))
        
        return x

class Transformer(nn.Module):
    """完整的Transformer模型"""
    
    def __init__(self, vocab_size, d_model=512, num_heads=8, 
                 num_layers=6, d_ff=2048, max_len=5000, dropout=0.1):
        super(Transformer, self).__init__()
        
        # Embedding
        self.embedding = nn.Embedding(vocab_size, d_model)
        self.pos_encoding = PositionalEncoding(d_model, max_len)
        
        # Encoder层
        self.encoder_layers = nn.ModuleList([
            TransformerBlock(d_model, num_heads, d_ff, dropout)
            for _ in range(num_layers)
        ])
        
        # 输出层
        self.fc_out = nn.Linear(d_model, vocab_size)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x, mask=None):
        # Embedding + 位置编码
        x = self.embedding(x) * math.sqrt(self.embedding.embedding_dim)
        x = self.pos_encoding(x)
        x = self.dropout(x)
        
        # Encoder层
        for layer in self.encoder_layers:
            x = layer(x, mask)
        
        # 输出
        output = self.fc_out(x)
        
        return output

# 创建模型
vocab_size = 10000
model = Transformer(vocab_size)

# 测试
batch_size = 2
seq_len = 20
x = torch.randint(0, vocab_size, (batch_size, seq_len))

output = model(x)
print(f"输入形状: {x.shape}")
print(f"输出形状: {output.shape}")

total_params = sum(p.numel() for p in model.parameters())
print(f"总参数量: {total_params:,}")
```

---

## 7-10. 高级主题（概览）

```python
"""
Transformer高级主题
@author erik.zhou
"""

print("""
7. BERT（Bidirectional Encoder Representations from Transformers）：
   - 双向Encoder
   - 预训练任务：MLM + NSP
   - 应用：文本分类、NER、问答

8. GPT（Generative Pre-trained Transformer）：
   - 单向Decoder
   - 自回归生成
   - 应用：文本生成、对话系统

9. 应用场景：
   - NLP：翻译、摘要、问答
   - CV：ViT、DETR
   - 多模态：CLIP、DALL-E
   - 其他：AlphaFold、MusicGen

10. 最佳实践：
    - 预训练 + 微调
    - 学习率预热
    - 梯度裁剪
    - 混合精度训练
    - 模型并行
""")
```

---

## 📝 学习检查清单

- [ ] 理解Transformer的核心架构
- [ ] 掌握Self-Attention机制
- [ ] 理解Multi-Head Attention
- [ ] 掌握位置编码原理
- [ ] 能够实现完整的Transformer
- [ ] 理解BERT和GPT的区别
- [ ] 了解Transformer的应用场景
- [ ] 掌握Transformer的优化技术

## 🔗 相关资源

- [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
- [The Illustrated Transformer](http://jalammar.github.io/illustrated-transformer/)
- [Hugging Face Transformers](https://huggingface.co/docs/transformers/)
- [CS224N: NLP with Deep Learning](http://web.stanford.edu/class/cs224n/)

---

**@author erik.zhou**  
**最后更新**: 2026-02-13
