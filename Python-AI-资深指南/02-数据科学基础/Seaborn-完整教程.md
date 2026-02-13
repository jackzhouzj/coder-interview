# Seaborn完整教程

> 掌握Seaborn统计可视化，创建优雅的数据分析图表
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: Seaborn 0.12+  
**学习难度**: ⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐  
**预计学习时长**: 10-15小时

## 🎯 学习目标

- [ ] 理解Seaborn与Matplotlib的关系
- [ ] 掌握Seaborn的基础图表类型
- [ ] 能够进行分布可视化
- [ ] 掌握分类数据可视化
- [ ] 理解关系图表的使用
- [ ] 能够创建矩阵图和热力图
- [ ] 掌握主题和样式设置
- [ ] 理解FacetGrid多图布局

## 📖 目录

1. [Seaborn基础](#1-seaborn基础)
2. [分布图](#2-分布图)
3. [分类图](#3-分类图)
4. [关系图](#4-关系图)
5. [矩阵图](#5-矩阵图)
6. [回归图](#6-回归图)
7. [多图布局](#7-多图布局)
8. [主题样式](#8-主题样式)
9. [调色板](#9-调色板)
10. [最佳实践](#10-最佳实践)

---

## 1. Seaborn基础

### 1.1 安装和导入

```python
"""
Seaborn安装和导入
@author erik.zhou
"""
# 安装
# pip install seaborn

import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

# 查看版本
print(f"Seaborn版本: {sns.__version__}")

# 设置中文字体
plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

# 设置Seaborn样式
sns.set_theme()
```

### 1.2 内置数据集

```python
"""
Seaborn内置数据集
@author erik.zhou
"""
import seaborn as sns

# 查看可用数据集
print("可用数据集:", sns.get_dataset_names())

# 加载示例数据集
tips = sns.load_dataset('tips')
print("\ntips数据集前5行:")
print(tips.head())

iris = sns.load_dataset('iris')
print("\niris数据集前5行:")
print(iris.head())

titanic = sns.load_dataset('titanic')
print("\ntitanic数据集前5行:")
print(titanic.head())
```

### 1.3 基本绘图流程

```python
"""
基本绘图流程
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

# 加载数据
tips = sns.load_dataset('tips')

# 创建图表
plt.figure(figsize=(10, 6))
sns.scatterplot(data=tips, x='total_bill', y='tip', hue='time')

plt.title('账单总额与小费关系')
plt.xlabel('账单总额')
plt.ylabel('小费')
plt.tight_layout()
plt.show()
```

---

## 2. 分布图

### 2.1 直方图和KDE

```python
"""
直方图和核密度估计
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# 直方图
sns.histplot(data=tips, x='total_bill', ax=axes[0, 0])
axes[0, 0].set_title('直方图')

# 直方图 + KDE
sns.histplot(data=tips, x='total_bill', kde=True, ax=axes[0, 1])
axes[0, 1].set_title('直方图 + KDE')

# KDE图
sns.kdeplot(data=tips, x='total_bill', ax=axes[1, 0])
axes[1, 0].set_title('KDE图')

# 多组KDE对比
sns.kdeplot(data=tips, x='total_bill', hue='time', ax=axes[1, 1])
axes[1, 1].set_title('分组KDE图')

plt.tight_layout()
plt.show()
```

### 2.2 分布图（distplot）

```python
"""
分布图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 单变量分布
sns.displot(data=tips, x='total_bill', kde=True, ax=axes[0])
axes[0].set_title('单变量分布')

# 双变量分布
plt.figure(figsize=(10, 8))
sns.displot(data=tips, x='total_bill', y='tip', kind='kde')
plt.title('双变量分布')

plt.tight_layout()
plt.show()
```

### 2.3 箱线图和小提琴图

```python
"""
箱线图和小提琴图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(16, 6))

# 箱线图
sns.boxplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('箱线图')

# 小提琴图
sns.violinplot(data=tips, x='day', y='total_bill', ax=axes[1])
axes[1].set_title('小提琴图')

# 分组小提琴图
sns.violinplot(data=tips, x='day', y='total_bill', hue='sex', split=True, ax=axes[2])
axes[2].set_title('分组小提琴图')

plt.tight_layout()
plt.show()
```

### 2.4 ECDF图

```python
"""
经验累积分布函数图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

plt.figure(figsize=(10, 6))
sns.ecdfplot(data=tips, x='total_bill', hue='time')

plt.title('ECDF图')
plt.xlabel('账单总额')
plt.ylabel('累积概率')
plt.tight_layout()
plt.show()
```

---

## 3. 分类图

### 3.1 条形图

```python
"""
条形图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(16, 6))

# 基础条形图
sns.barplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('条形图')

# 分组条形图
sns.barplot(data=tips, x='day', y='total_bill', hue='sex', ax=axes[1])
axes[1].set_title('分组条形图')

# 计数条形图
sns.countplot(data=tips, x='day', hue='sex', ax=axes[2])
axes[2].set_title('计数条形图')

plt.tight_layout()
plt.show()
```

### 3.2 点图

```python
"""
点图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 点图
sns.pointplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('点图')

# 分组点图
sns.pointplot(data=tips, x='day', y='total_bill', hue='sex', ax=axes[1])
axes[1].set_title('分组点图')

plt.tight_layout()
plt.show()
```

### 3.3 分类散点图

```python
"""
分类散点图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(16, 6))

# 散点图
sns.stripplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('散点图')

# 抖动散点图
sns.stripplot(data=tips, x='day', y='total_bill', jitter=True, ax=axes[1])
axes[1].set_title('抖动散点图')

# 蜂群图
sns.swarmplot(data=tips, x='day', y='total_bill', ax=axes[2])
axes[2].set_title('蜂群图')

plt.tight_layout()
plt.show()
```

### 3.4 组合图表

```python
"""
组合分类图表
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 小提琴图 + 散点图
sns.violinplot(data=tips, x='day', y='total_bill', ax=axes[0])
sns.stripplot(data=tips, x='day', y='total_bill', color='black', alpha=0.3, ax=axes[0])
axes[0].set_title('小提琴图 + 散点图')

# 箱线图 + 蜂群图
sns.boxplot(data=tips, x='day', y='total_bill', ax=axes[1])
sns.swarmplot(data=tips, x='day', y='total_bill', color='black', alpha=0.5, ax=axes[1])
axes[1].set_title('箱线图 + 蜂群图')

plt.tight_layout()
plt.show()
```

---

## 4. 关系图

### 4.1 散点图

```python
"""
关系散点图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(16, 6))

# 基础散点图
sns.scatterplot(data=tips, x='total_bill', y='tip', ax=axes[0])
axes[0].set_title('基础散点图')

# 分组散点图
sns.scatterplot(data=tips, x='total_bill', y='tip', hue='time', ax=axes[1])
axes[1].set_title('分组散点图')

# 多维散点图
sns.scatterplot(data=tips, x='total_bill', y='tip', 
                hue='time', size='size', style='sex', ax=axes[2])
axes[2].set_title('多维散点图')

plt.tight_layout()
plt.show()
```

### 4.2 折线图

```python
"""
关系折线图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np

# 创建时间序列数据
dates = pd.date_range('2024-01-01', periods=100)
data = pd.DataFrame({
    'date': dates,
    'value1': np.cumsum(np.random.randn(100)),
    'value2': np.cumsum(np.random.randn(100))
})

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 单线图
sns.lineplot(data=data, x='date', y='value1', ax=axes[0])
axes[0].set_title('单线图')
axes[0].tick_params(axis='x', rotation=45)

# 多线图
data_melted = data.melt(id_vars='date', var_name='类型', value_name='值')
sns.lineplot(data=data_melted, x='date', y='值', hue='类型', ax=axes[1])
axes[1].set_title('多线图')
axes[1].tick_params(axis='x', rotation=45)

plt.tight_layout()
plt.show()
```

### 4.3 关系图（relplot）

```python
"""
关系图网格
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# 使用relplot创建多面板图
g = sns.relplot(data=tips, x='total_bill', y='tip',
                col='time', hue='sex', style='smoker',
                height=5, aspect=1.2)

g.set_axis_labels('账单总额', '小费')
g.set_titles('{col_name}')
plt.tight_layout()
plt.show()
```

---

## 5. 矩阵图

### 5.1 热力图

```python
"""
热力图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# 生成相关系数矩阵
tips = sns.load_dataset('tips')
corr = tips[['total_bill', 'tip', 'size']].corr()

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 基础热力图
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm', ax=axes[0])
axes[0].set_title('相关系数热力图')

# 自定义热力图
sns.heatmap(corr, annot=True, fmt='.2f', cmap='RdYlGn',
            linewidths=0.5, square=True, cbar_kws={'shrink': 0.8}, ax=axes[1])
axes[1].set_title('自定义热力图')

plt.tight_layout()
plt.show()
```

### 5.2 聚类热力图

```python
"""
聚类热力图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

# 加载数据
iris = sns.load_dataset('iris')
iris_data = iris.drop('species', axis=1)

plt.figure(figsize=(10, 8))
sns.clustermap(iris_data, cmap='viridis', standard_scale=1,
               figsize=(10, 8), dendrogram_ratio=0.15)

plt.title('聚类热力图')
plt.tight_layout()
plt.show()
```


### 5.3 配对图

```python
"""
配对图（散点图矩阵）
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

iris = sns.load_dataset('iris')

# 创建配对图
g = sns.pairplot(iris, hue='species', diag_kind='kde',
                 height=2.5, aspect=1.2)

g.fig.suptitle('鸢尾花数据配对图', y=1.02)
plt.tight_layout()
plt.show()
```

---

## 6. 回归图

### 6.1 回归散点图

```python
"""
回归散点图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(16, 6))

# 线性回归
sns.regplot(data=tips, x='total_bill', y='tip', ax=axes[0])
axes[0].set_title('线性回归')

# 多项式回归
sns.regplot(data=tips, x='total_bill', y='tip', order=2, ax=axes[1])
axes[1].set_title('多项式回归（2阶）')

# Lowess回归
sns.regplot(data=tips, x='total_bill', y='tip', lowess=True, ax=axes[2])
axes[2].set_title('Lowess回归')

plt.tight_layout()
plt.show()
```

### 6.2 残差图

```python
"""
残差图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 残差图
sns.residplot(data=tips, x='total_bill', y='tip', ax=axes[0])
axes[0].set_title('残差图')

# 分组残差图
sns.residplot(data=tips, x='total_bill', y='tip', lowess=True, ax=axes[1])
axes[1].set_title('Lowess残差图')

plt.tight_layout()
plt.show()
```

### 6.3 联合分布图

```python
"""
联合分布图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# 创建联合分布图
g = sns.jointplot(data=tips, x='total_bill', y='tip',
                  kind='reg', height=8)

g.fig.suptitle('账单与小费联合分布', y=1.02)
plt.tight_layout()
plt.show()
```

### 6.4 不同类型的联合图

```python
"""
不同类型的联合图
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# 创建多种联合图
kinds = ['scatter', 'kde', 'hex', 'reg']

fig = plt.figure(figsize=(16, 12))

for i, kind in enumerate(kinds, 1):
    plt.subplot(2, 2, i)
    g = sns.jointplot(data=tips, x='total_bill', y='tip',
                      kind=kind, height=5)
    plt.suptitle(f'联合图类型: {kind}', y=0.98)

plt.tight_layout()
plt.show()
```

---

## 7. 多图布局

### 7.1 FacetGrid基础

```python
"""
FacetGrid基础
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# 创建FacetGrid
g = sns.FacetGrid(tips, col='time', row='sex', height=4, aspect=1.2)
g.map(sns.scatterplot, 'total_bill', 'tip')

g.set_axis_labels('账单总额', '小费')
g.set_titles('{row_name} - {col_name}')
g.add_legend()

plt.tight_layout()
plt.show()
```

### 7.2 FacetGrid高级用法

```python
"""
FacetGrid高级用法
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# 创建复杂的FacetGrid
g = sns.FacetGrid(tips, col='day', hue='sex', height=4, aspect=1.2)
g.map(sns.scatterplot, 'total_bill', 'tip', alpha=0.7)
g.add_legend()

# 自定义每个子图
for ax in g.axes.flat:
    ax.grid(True, alpha=0.3)

g.set_axis_labels('账单总额', '小费')
g.set_titles('{col_name}')

plt.tight_layout()
plt.show()
```

### 7.3 catplot和relplot

```python
"""
catplot和relplot多图布局
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# catplot
g1 = sns.catplot(data=tips, x='day', y='total_bill',
                 col='time', kind='box', height=5, aspect=1.2)
g1.set_axis_labels('星期', '账单总额')
g1.set_titles('{col_name}')

# relplot
g2 = sns.relplot(data=tips, x='total_bill', y='tip',
                 col='day', hue='sex', style='time',
                 height=4, aspect=1.2)
g2.set_axis_labels('账单总额', '小费')
g2.set_titles('{col_name}')

plt.tight_layout()
plt.show()
```

---

## 8. 主题样式

### 8.1 内置主题

```python
"""
Seaborn内置主题
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# 可用主题
themes = ['darkgrid', 'whitegrid', 'dark', 'white', 'ticks']

fig, axes = plt.subplots(2, 3, figsize=(16, 10))
axes = axes.flatten()

x = np.linspace(0, 10, 100)
y = np.sin(x)

for i, theme in enumerate(themes):
    sns.set_theme(style=theme)
    axes[i].plot(x, y, linewidth=2)
    axes[i].set_title(f'主题: {theme}')
    axes[i].set_xlabel('X轴')
    axes[i].set_ylabel('Y轴')

# 隐藏多余的子图
axes[-1].axis('off')

plt.tight_layout()
plt.show()

# 重置为默认主题
sns.set_theme()
```

### 8.2 上下文设置

```python
"""
上下文设置（字体大小）
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

contexts = ['paper', 'notebook', 'talk', 'poster']

fig, axes = plt.subplots(2, 2, figsize=(14, 10))
axes = axes.flatten()

x = np.linspace(0, 10, 100)
y = np.sin(x)

for i, context in enumerate(contexts):
    sns.set_context(context)
    axes[i].plot(x, y, linewidth=2)
    axes[i].set_title(f'上下文: {context}')
    axes[i].set_xlabel('X轴')
    axes[i].set_ylabel('Y轴')
    axes[i].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()

# 重置
sns.set_context('notebook')
```

### 8.3 自定义样式

```python
"""
自定义样式
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# 自定义参数
custom_params = {
    'axes.facecolor': '#f0f0f0',
    'axes.edgecolor': 'black',
    'axes.linewidth': 2,
    'grid.color': 'white',
    'grid.linewidth': 1.5,
    'font.size': 12
}

with sns.axes_style('darkgrid', rc=custom_params):
    plt.figure(figsize=(10, 6))
    sns.scatterplot(data=tips, x='total_bill', y='tip', hue='time')
    plt.title('自定义样式示例')
    plt.tight_layout()
    plt.show()
```

---

## 9. 调色板

### 9.1 分类调色板

```python
"""
分类调色板
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

# 显示不同的分类调色板
palettes = ['deep', 'muted', 'pastel', 'bright', 'dark', 'colorblind']

fig, axes = plt.subplots(3, 2, figsize=(14, 12))
axes = axes.flatten()

tips = sns.load_dataset('tips')

for i, palette in enumerate(palettes):
    sns.barplot(data=tips, x='day', y='total_bill', hue='sex',
                palette=palette, ax=axes[i])
    axes[i].set_title(f'调色板: {palette}')
    axes[i].legend(loc='upper right')

plt.tight_layout()
plt.show()
```

### 9.2 连续调色板

```python
"""
连续调色板
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# 生成数据
data = np.random.rand(10, 10)

# 不同的连续调色板
palettes = ['viridis', 'plasma', 'inferno', 'magma', 'cividis', 'rocket']

fig, axes = plt.subplots(2, 3, figsize=(16, 10))
axes = axes.flatten()

for i, palette in enumerate(palettes):
    sns.heatmap(data, cmap=palette, annot=True, fmt='.2f',
                cbar_kws={'shrink': 0.8}, ax=axes[i])
    axes[i].set_title(f'调色板: {palette}')

plt.tight_layout()
plt.show()
```

### 9.3 自定义调色板

```python
"""
自定义调色板
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(16, 6))

# 使用颜色列表
colors1 = ['#FF6B6B', '#4ECDC4', '#45B7D1']
sns.barplot(data=tips, x='day', y='total_bill', palette=colors1, ax=axes[0])
axes[0].set_title('自定义颜色列表')

# 使用color_palette
colors2 = sns.color_palette('husl', 4)
sns.barplot(data=tips, x='day', y='total_bill', palette=colors2, ax=axes[1])
axes[1].set_title('HUSL调色板')

# 使用渐变色
colors3 = sns.light_palette('seagreen', n_colors=4)
sns.barplot(data=tips, x='day', y='total_bill', palette=colors3, ax=axes[2])
axes[2].set_title('渐变调色板')

plt.tight_layout()
plt.show()
```

### 9.4 调色板可视化

```python
"""
调色板可视化
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

# 显示调色板
palettes = {
    '分类': ['deep', 'muted', 'pastel'],
    '连续': ['viridis', 'rocket', 'mako'],
    '发散': ['vlag', 'icefire', 'coolwarm']
}

fig, axes = plt.subplots(3, 3, figsize=(14, 10))

for i, (category, palette_list) in enumerate(palettes.items()):
    for j, palette in enumerate(palette_list):
        colors = sns.color_palette(palette, 10)
        sns.palplot(colors, ax=axes[i, j])
        axes[i, j].set_title(f'{category}: {palette}')
        axes[i, j].axis('off')

plt.tight_layout()
plt.show()
```

---

## 10. 最佳实践

### 10.1 数据准备

```python
"""
数据准备最佳实践
@author erik.zhou
"""
import seaborn as sns
import pandas as pd
import numpy as np

# 1. 使用长格式数据（推荐）
# 宽格式
wide_data = pd.DataFrame({
    'A': [1, 2, 3],
    'B': [4, 5, 6],
    'C': [7, 8, 9]
})

# 转换为长格式
long_data = wide_data.melt(var_name='类别', value_name='数值')
print("长格式数据:")
print(long_data)

# 2. 确保数据类型正确
tips = sns.load_dataset('tips')
print("\n数据类型:")
print(tips.dtypes)

# 3. 处理缺失值
print(f"\n缺失值: {tips.isnull().sum().sum()}")
```

### 10.2 图表优化

```python
"""
图表优化技巧
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# 创建优化的图表
fig, ax = plt.subplots(figsize=(10, 6))

# 1. 使用合适的图表类型
sns.scatterplot(data=tips, x='total_bill', y='tip', 
                hue='time', size='size', style='sex',
                palette='Set2', alpha=0.7, ax=ax)

# 2. 添加清晰的标签
ax.set_xlabel('账单总额 ($)', fontsize=12)
ax.set_ylabel('小费 ($)', fontsize=12)
ax.set_title('账单总额与小费关系分析', fontsize=14, fontweight='bold')

# 3. 优化图例
ax.legend(title='图例', bbox_to_anchor=(1.05, 1), loc='upper left')

# 4. 添加网格
ax.grid(True, alpha=0.3, linestyle='--')

# 5. 使用tight_layout
plt.tight_layout()
plt.show()
```

### 10.3 性能优化

```python
"""
性能优化
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import time

# 生成大数据集
n = 100000
data = pd.DataFrame({
    'x': np.random.randn(n),
    'y': np.random.randn(n),
    'category': np.random.choice(['A', 'B', 'C'], n)
})

# 方法1: 直接绘制（慢）
start = time.time()
fig, ax = plt.subplots(figsize=(10, 6))
sns.scatterplot(data=data.sample(10000), x='x', y='y', hue='category', ax=ax)
time1 = time.time() - start
ax.set_title(f'采样绘制 ({time1:.2f}秒)')
plt.close()

# 方法2: 使用rasterized（快）
start = time.time()
fig, ax = plt.subplots(figsize=(10, 6))
sns.scatterplot(data=data.sample(10000), x='x', y='y', 
                hue='category', rasterized=True, ax=ax)
time2 = time.time() - start
ax.set_title(f'rasterized绘制 ({time2:.2f}秒)')

print(f"性能提升: {time1/time2:.2f}x")
plt.tight_layout()
plt.show()
```

### 10.4 代码规范

```python
"""
Seaborn代码规范
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt

# 1. 设置全局样式
sns.set_theme(style='whitegrid', context='notebook')

# 2. 加载数据
tips = sns.load_dataset('tips')

# 3. 创建图表
fig, ax = plt.subplots(figsize=(10, 6))

# 4. 使用data参数（推荐）
sns.scatterplot(data=tips, x='total_bill', y='tip', 
                hue='time', style='sex', ax=ax)

# 5. 设置标签和标题
ax.set_xlabel('账单总额', fontsize=12)
ax.set_ylabel('小费', fontsize=12)
ax.set_title('账单与小费关系', fontsize=14, fontweight='bold')

# 6. 优化图例
ax.legend(title='图例', loc='best')

# 7. 使用tight_layout
plt.tight_layout()

# 8. 保存图表
plt.savefig('seaborn_plot.png', dpi=300, bbox_inches='tight')

plt.show()
```

### 10.5 常见陷阱

```python
"""
Seaborn常见陷阱
@author erik.zhou
"""
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

tips = sns.load_dataset('tips')

# 陷阱1: 数据格式不正确
# 错误：使用宽格式数据
# 正确：转换为长格式

# 陷阱2: 忘记指定ax参数
# 错误：在子图中不指定ax
fig, axes = plt.subplots(1, 2, figsize=(14, 6))
# 正确：
sns.scatterplot(data=tips, x='total_bill', y='tip', ax=axes[0])
sns.boxplot(data=tips, x='day', y='total_bill', ax=axes[1])

# 陷阱3: 颜色映射不一致
# 确保使用相同的hue_order
hue_order = ['Lunch', 'Dinner']
sns.scatterplot(data=tips, x='total_bill', y='tip', 
                hue='time', hue_order=hue_order, ax=axes[0])

# 陷阱4: 图例重复
# 使用legend=False避免重复图例
axes[0].legend_.remove()  # 移除重复图例

plt.tight_layout()
plt.show()
```

---

## 📝 学习检查清单

- [ ] 理解Seaborn与Matplotlib的关系
- [ ] 能够绘制各种分布图
- [ ] 掌握分类数据可视化
- [ ] 能够创建关系图表
- [ ] 掌握矩阵图和热力图
- [ ] 理解回归图的使用
- [ ] 能够使用FacetGrid创建多图布局
- [ ] 掌握主题和样式设置
- [ ] 理解调色板的使用
- [ ] 了解性能优化技巧

## 🔗 相关资源

- [Seaborn官方文档](https://seaborn.pydata.org/)
- [Seaborn教程](https://seaborn.pydata.org/tutorial.html)
- [Seaborn图表库](https://seaborn.pydata.org/examples/index.html)
- [Seaborn API参考](https://seaborn.pydata.org/api.html)

---

**@author erik.zhou**  
**最后更新**: 2026-02-13
