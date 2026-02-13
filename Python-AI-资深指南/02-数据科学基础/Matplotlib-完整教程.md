# Matplotlib完整教程

> 掌握Matplotlib数据可视化，创建专业的统计图表
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: Matplotlib 3.7+  
**学习难度**: ⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐⭐  
**预计学习时长**: 15-25小时

## 🎯 学习目标

- [ ] 理解Matplotlib的核心概念和架构
- [ ] 掌握基础图表类型的绘制
- [ ] 能够自定义图表样式和布局
- [ ] 掌握子图和多图布局
- [ ] 理解坐标轴和标签设置
- [ ] 能够创建交互式图表
- [ ] 掌握图表保存和导出
- [ ] 理解最佳实践和性能优化

## 📖 目录

1. [Matplotlib基础](#1-matplotlib基础)
2. [基础图表](#2-基础图表)
3. [图表样式](#3-图表样式)
4. [子图布局](#4-子图布局)
5. [坐标轴设置](#5-坐标轴设置)
6. [文本和注释](#6-文本和注释)
7. [高级图表](#7-高级图表)
8. [3D图表](#8-3d图表)
9. [动画和交互](#9-动画和交互)
10. [最佳实践](#10-最佳实践)

---

## 1. Matplotlib基础

### 1.1 安装和导入

```python
"""
Matplotlib安装和导入
@author erik.zhou
"""
# 安装
# pip install matplotlib

import matplotlib.pyplot as plt
import numpy as np

# 查看版本
print(f"Matplotlib版本: {plt.matplotlib.__version__}")

# 设置中文字体支持
plt.rcParams['font.sans-serif'] = ['SimHei']  # 用来正常显示中文标签
plt.rcParams['axes.unicode_minus'] = False  # 用来正常显示负号
```

### 1.2 基本绘图流程

```python
"""
基本绘图流程
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 1. 准备数据
x = np.linspace(0, 10, 100)
y = np.sin(x)

# 2. 创建图形和坐标轴
fig, ax = plt.subplots()

# 3. 绘制图表
ax.plot(x, y)

# 4. 设置标签和标题
ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('正弦函数')

# 5. 显示图表
plt.show()
```


### 1.3 两种绘图接口

```python
"""
两种绘图接口对比
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

# 方式1: pyplot接口（类似MATLAB）
plt.figure()
plt.plot(x, y)
plt.xlabel('X轴')
plt.ylabel('Y轴')
plt.title('pyplot接口')
plt.show()

# 方式2: 面向对象接口（推荐）
fig, ax = plt.subplots()
ax.plot(x, y)
ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('面向对象接口')
plt.show()
```

---

## 2. 基础图表

### 2.1 折线图

```python
"""
折线图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y1 = np.sin(x)
y2 = np.cos(x)

fig, ax = plt.subplots(figsize=(10, 6))

# 绘制多条线
ax.plot(x, y1, label='sin(x)', color='blue', linewidth=2, linestyle='-')
ax.plot(x, y2, label='cos(x)', color='red', linewidth=2, linestyle='--')

# 设置标签和标题
ax.set_xlabel('X轴', fontsize=12)
ax.set_ylabel('Y轴', fontsize=12)
ax.set_title('三角函数图', fontsize=14, fontweight='bold')

# 添加图例
ax.legend(loc='upper right')

# 添加网格
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 2.2 散点图

```python
"""
散点图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 生成随机数据
np.random.seed(42)
x = np.random.randn(100)
y = np.random.randn(100)
colors = np.random.rand(100)
sizes = 1000 * np.random.rand(100)

fig, ax = plt.subplots(figsize=(10, 6))

# 绘制散点图
scatter = ax.scatter(x, y, c=colors, s=sizes, alpha=0.5, cmap='viridis')

# 添加颜色条
plt.colorbar(scatter, ax=ax, label='颜色值')

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('散点图示例')
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 2.3 柱状图

```python
"""
柱状图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 数据
categories = ['A', 'B', 'C', 'D', 'E']
values1 = [23, 45, 56, 78, 32]
values2 = [34, 55, 43, 67, 45]

x = np.arange(len(categories))
width = 0.35

fig, ax = plt.subplots(figsize=(10, 6))

# 绘制柱状图
bars1 = ax.bar(x - width/2, values1, width, label='组1', color='skyblue')
bars2 = ax.bar(x + width/2, values2, width, label='组2', color='lightcoral')

# 在柱子上添加数值标签
for bars in [bars1, bars2]:
    for bar in bars:
        height = bar.get_height()
        ax.text(bar.get_x() + bar.get_width()/2., height,
                f'{height:.0f}',
                ha='center', va='bottom', fontsize=10)

ax.set_xlabel('类别')
ax.set_ylabel('数值')
ax.set_title('分组柱状图')
ax.set_xticks(x)
ax.set_xticklabels(categories)
ax.legend()
ax.grid(True, alpha=0.3, axis='y')

plt.tight_layout()
plt.show()
```

### 2.4 直方图

```python
"""
直方图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 生成正态分布数据
np.random.seed(42)
data1 = np.random.normal(100, 15, 1000)
data2 = np.random.normal(120, 20, 1000)

fig, ax = plt.subplots(figsize=(10, 6))

# 绘制直方图
ax.hist(data1, bins=30, alpha=0.5, label='数据1', color='blue', edgecolor='black')
ax.hist(data2, bins=30, alpha=0.5, label='数据2', color='red', edgecolor='black')

ax.set_xlabel('数值')
ax.set_ylabel('频数')
ax.set_title('直方图示例')
ax.legend()
ax.grid(True, alpha=0.3, axis='y')

plt.tight_layout()
plt.show()
```

### 2.5 饼图

```python
"""
饼图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt

# 数据
labels = ['Python', 'Java', 'JavaScript', 'C++', '其他']
sizes = [35, 25, 20, 15, 5]
colors = ['#ff9999', '#66b3ff', '#99ff99', '#ffcc99', '#ff99cc']
explode = (0.1, 0, 0, 0, 0)  # 突出显示第一块

fig, ax = plt.subplots(figsize=(10, 8))

# 绘制饼图
wedges, texts, autotexts = ax.pie(sizes, explode=explode, labels=labels,
                                    colors=colors, autopct='%1.1f%%',
                                    shadow=True, startangle=90)

# 美化文本
for text in texts:
    text.set_fontsize(12)
for autotext in autotexts:
    autotext.set_color('white')
    autotext.set_fontweight('bold')
    autotext.set_fontsize(10)

ax.set_title('编程语言使用占比', fontsize=14, fontweight='bold')
ax.axis('equal')  # 保持圆形

plt.tight_layout()
plt.show()
```

### 2.6 箱线图

```python
"""
箱线图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 生成数据
np.random.seed(42)
data = [np.random.normal(0, std, 100) for std in range(1, 5)]

fig, ax = plt.subplots(figsize=(10, 6))

# 绘制箱线图
bp = ax.boxplot(data, labels=['组1', '组2', '组3', '组4'],
                patch_artist=True, notch=True)

# 自定义颜色
colors = ['lightblue', 'lightgreen', 'lightcoral', 'lightyellow']
for patch, color in zip(bp['boxes'], colors):
    patch.set_facecolor(color)

ax.set_xlabel('组别')
ax.set_ylabel('数值')
ax.set_title('箱线图示例')
ax.grid(True, alpha=0.3, axis='y')

plt.tight_layout()
plt.show()
```

---

## 3. 图表样式

### 3.1 线条样式

```python
"""
线条样式设置
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)

fig, ax = plt.subplots(figsize=(12, 6))

# 不同线条样式
line_styles = ['-', '--', '-.', ':']
line_labels = ['实线', '虚线', '点划线', '点线']

for i, (style, label) in enumerate(zip(line_styles, line_labels)):
    ax.plot(x, np.sin(x + i), linestyle=style, linewidth=2, label=label)

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('线条样式示例')
ax.legend()
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 3.2 颜色设置

```python
"""
颜色设置
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)

fig, ax = plt.subplots(figsize=(12, 6))

# 不同颜色表示方式
ax.plot(x, np.sin(x), color='r', label='单字符: r')
ax.plot(x, np.sin(x + 1), color='blue', label='颜色名: blue')
ax.plot(x, np.sin(x + 2), color='#FF5733', label='十六进制: #FF5733')
ax.plot(x, np.sin(x + 3), color=(0.2, 0.4, 0.6), label='RGB元组: (0.2, 0.4, 0.6)')

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('颜色设置示例')
ax.legend()
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 3.3 标记样式

```python
"""
标记样式设置
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 20)

fig, ax = plt.subplots(figsize=(12, 6))

# 不同标记样式
markers = ['o', 's', '^', 'D', '*', 'p']
marker_labels = ['圆形', '方形', '三角形', '菱形', '星形', '五边形']

for i, (marker, label) in enumerate(zip(markers, marker_labels)):
    ax.plot(x, np.sin(x + i*0.5), marker=marker, markersize=8,
            linestyle='-', linewidth=1, label=label)

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('标记样式示例')
ax.legend(ncol=3)
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 3.4 样式主题

```python
"""
样式主题设置
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 查看可用样式
print("可用样式:", plt.style.available)

x = np.linspace(0, 10, 100)
y = np.sin(x)

# 使用不同样式
styles = ['default', 'seaborn-v0_8', 'ggplot', 'bmh']

fig, axes = plt.subplots(2, 2, figsize=(14, 10))
axes = axes.flatten()

for ax, style in zip(axes, styles):
    with plt.style.context(style):
        ax.plot(x, y, linewidth=2)
        ax.set_title(f'样式: {style}')
        ax.set_xlabel('X轴')
        ax.set_ylabel('Y轴')
        ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

---

## 4. 子图布局

### 4.1 基础子图

```python
"""
基础子图布局
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)

# 创建2x2子图
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# 绘制不同图表
axes[0, 0].plot(x, np.sin(x))
axes[0, 0].set_title('sin(x)')

axes[0, 1].plot(x, np.cos(x), 'r')
axes[0, 1].set_title('cos(x)')

axes[1, 0].plot(x, np.tan(x))
axes[1, 0].set_title('tan(x)')
axes[1, 0].set_ylim(-5, 5)

axes[1, 1].plot(x, np.exp(-x/5))
axes[1, 1].set_title('exp(-x/5)')

# 统一设置
for ax in axes.flatten():
    ax.grid(True, alpha=0.3)
    ax.set_xlabel('X轴')
    ax.set_ylabel('Y轴')

plt.tight_layout()
plt.show()
```

### 4.2 不规则子图

```python
"""
不规则子图布局
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

fig = plt.figure(figsize=(12, 8))

# 使用subplot2grid创建不规则布局
ax1 = plt.subplot2grid((3, 3), (0, 0), colspan=3)
ax2 = plt.subplot2grid((3, 3), (1, 0), colspan=2)
ax3 = plt.subplot2grid((3, 3), (1, 2), rowspan=2)
ax4 = plt.subplot2grid((3, 3), (2, 0))
ax5 = plt.subplot2grid((3, 3), (2, 1))

x = np.linspace(0, 10, 100)

# 绘制不同图表
ax1.plot(x, np.sin(x))
ax1.set_title('大图')

ax2.plot(x, np.cos(x))
ax2.set_title('中图1')

ax3.plot(x, np.tan(x))
ax3.set_title('竖图')
ax3.set_ylim(-5, 5)

ax4.plot(x, x**2)
ax4.set_title('小图1')

ax5.plot(x, np.sqrt(x))
ax5.set_title('小图2')

plt.tight_layout()
plt.show()
```

### 4.3 嵌套子图

```python
"""
嵌套子图
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

fig = plt.figure(figsize=(12, 8))

# 主图
ax_main = fig.add_subplot(111)
x = np.linspace(0, 10, 100)
ax_main.plot(x, np.sin(x), 'b-', linewidth=2)
ax_main.set_xlabel('X轴')
ax_main.set_ylabel('Y轴')
ax_main.set_title('主图')
ax_main.grid(True, alpha=0.3)

# 嵌入小图
ax_inset = fig.add_axes([0.6, 0.6, 0.25, 0.25])  # [left, bottom, width, height]
ax_inset.plot(x, np.sin(x), 'r-')
ax_inset.set_title('局部放大', fontsize=10)
ax_inset.set_xlim(4, 6)
ax_inset.set_ylim(-1, 1)
ax_inset.grid(True, alpha=0.3)

plt.show()
```

---

## 5. 坐标轴设置

### 5.1 坐标轴范围和刻度

```python
"""
坐标轴范围和刻度设置
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# 默认设置
axes[0].plot(x, y)
axes[0].set_title('默认设置')

# 自定义范围
axes[1].plot(x, y)
axes[1].set_xlim(2, 8)
axes[1].set_ylim(-0.5, 0.5)
axes[1].set_title('自定义范围')

# 自定义刻度
axes[2].plot(x, y)
axes[2].set_xticks([0, 2, 4, 6, 8, 10])
axes[2].set_xticklabels(['零', '二', '四', '六', '八', '十'])
axes[2].set_yticks([-1, -0.5, 0, 0.5, 1])
axes[2].set_title('自定义刻度')

for ax in axes:
    ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 5.2 对数坐标轴

```python
"""
对数坐标轴
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0.1, 10, 100)
y = np.exp(x)

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# 线性坐标
axes[0].plot(x, y)
axes[0].set_title('线性坐标')
axes[0].grid(True, alpha=0.3)

# 对数Y轴
axes[1].semilogy(x, y)
axes[1].set_title('对数Y轴')
axes[1].grid(True, alpha=0.3)

# 双对数坐标
axes[2].loglog(x, y)
axes[2].set_title('双对数坐标')
axes[2].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 5.3 双Y轴

```python
"""
双Y轴图表
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y1 = np.sin(x)
y2 = np.exp(x/5)

fig, ax1 = plt.subplots(figsize=(10, 6))

# 第一个Y轴
color = 'tab:blue'
ax1.set_xlabel('X轴')
ax1.set_ylabel('sin(x)', color=color)
ax1.plot(x, y1, color=color, linewidth=2)
ax1.tick_params(axis='y', labelcolor=color)
ax1.grid(True, alpha=0.3)

# 第二个Y轴
ax2 = ax1.twinx()
color = 'tab:red'
ax2.set_ylabel('exp(x/5)', color=color)
ax2.plot(x, y2, color=color, linewidth=2)
ax2.tick_params(axis='y', labelcolor=color)

plt.title('双Y轴示例')
plt.tight_layout()
plt.show()
```

---

## 6. 文本和注释

### 6.1 添加文本

```python
"""
添加文本
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(x, y, linewidth=2)

# 添加文本
ax.text(5, 0.5, '这是文本', fontsize=14, color='red',
        ha='center', va='center',
        bbox=dict(boxstyle='round', facecolor='wheat', alpha=0.5))

# 添加数学公式
ax.text(2, -0.5, r'$y = \sin(x)$', fontsize=16)

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('文本示例')
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 6.2 添加注释

```python
"""
添加注释
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(x, y, linewidth=2)

# 标注最大值
max_idx = np.argmax(y)
ax.annotate('最大值',
            xy=(x[max_idx], y[max_idx]),
            xytext=(x[max_idx] + 1, y[max_idx] + 0.3),
            arrowprops=dict(arrowstyle='->', color='red', lw=2),
            fontsize=12, color='red')

# 标注最小值
min_idx = np.argmin(y)
ax.annotate('最小值',
            xy=(x[min_idx], y[min_idx]),
            xytext=(x[min_idx] + 1, y[min_idx] - 0.3),
            arrowprops=dict(arrowstyle='->', color='blue', lw=2),
            fontsize=12, color='blue')

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('注释示例')
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```


---

## 7. 高级图表

### 7.1 热力图

```python
"""
热力图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 生成数据
data = np.random.rand(10, 10)

fig, ax = plt.subplots(figsize=(10, 8))

# 绘制热力图
im = ax.imshow(data, cmap='YlOrRd', aspect='auto')

# 添加颜色条
cbar = plt.colorbar(im, ax=ax)
cbar.set_label('数值', rotation=270, labelpad=20)

# 设置刻度
ax.set_xticks(np.arange(10))
ax.set_yticks(np.arange(10))
ax.set_xticklabels([f'列{i}' for i in range(10)])
ax.set_yticklabels([f'行{i}' for i in range(10)])

# 在每个单元格中显示数值
for i in range(10):
    for j in range(10):
        text = ax.text(j, i, f'{data[i, j]:.2f}',
                      ha="center", va="center", color="black", fontsize=8)

ax.set_title('热力图示例')
plt.tight_layout()
plt.show()
```

### 7.2 等高线图

```python
"""
等高线图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 生成网格数据
x = np.linspace(-3, 3, 100)
y = np.linspace(-3, 3, 100)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 等高线图
contour = axes[0].contour(X, Y, Z, levels=10, cmap='viridis')
axes[0].clabel(contour, inline=True, fontsize=8)
axes[0].set_title('等高线图')
axes[0].set_xlabel('X轴')
axes[0].set_ylabel('Y轴')

# 填充等高线图
contourf = axes[1].contourf(X, Y, Z, levels=20, cmap='viridis')
plt.colorbar(contourf, ax=axes[1])
axes[1].set_title('填充等高线图')
axes[1].set_xlabel('X轴')
axes[1].set_ylabel('Y轴')

plt.tight_layout()
plt.show()
```

### 7.3 极坐标图

```python
"""
极坐标图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 生成数据
theta = np.linspace(0, 2*np.pi, 100)
r = 1 + np.sin(3*theta)

fig = plt.figure(figsize=(10, 10))
ax = fig.add_subplot(111, projection='polar')

# 绘制极坐标图
ax.plot(theta, r, linewidth=2)
ax.fill(theta, r, alpha=0.3)

ax.set_title('极坐标图示例', pad=20)
ax.grid(True)

plt.tight_layout()
plt.show()
```

### 7.4 误差条图

```python
"""
误差条图绘制
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 数据
x = np.arange(0, 10, 1)
y = np.sin(x)
yerr = 0.1 + 0.2 * np.random.rand(len(x))

fig, ax = plt.subplots(figsize=(10, 6))

# 绘制误差条图
ax.errorbar(x, y, yerr=yerr, fmt='o-', linewidth=2, markersize=8,
            capsize=5, capthick=2, label='数据')

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('误差条图示例')
ax.legend()
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### 7.5 填充区域图

```python
"""
填充区域图
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y1 = np.sin(x)
y2 = np.sin(x) + 0.5

fig, ax = plt.subplots(figsize=(10, 6))

# 绘制线条
ax.plot(x, y1, 'b-', linewidth=2, label='下界')
ax.plot(x, y2, 'r-', linewidth=2, label='上界')

# 填充区域
ax.fill_between(x, y1, y2, alpha=0.3, color='green', label='填充区域')

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('填充区域图示例')
ax.legend()
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

---

## 8. 3D图表

### 8.1 3D折线图

```python
"""
3D折线图
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')

# 生成数据
t = np.linspace(0, 10, 100)
x = np.sin(t)
y = np.cos(t)
z = t

# 绘制3D折线
ax.plot(x, y, z, linewidth=2, label='螺旋线')

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_zlabel('Z轴')
ax.set_title('3D折线图')
ax.legend()

plt.tight_layout()
plt.show()
```

### 8.2 3D散点图

```python
"""
3D散点图
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(10, 8))
ax = fig.add_subplot(111, projection='3d')

# 生成随机数据
np.random.seed(42)
n = 100
x = np.random.rand(n)
y = np.random.rand(n)
z = np.random.rand(n)
colors = np.random.rand(n)
sizes = 1000 * np.random.rand(n)

# 绘制3D散点图
scatter = ax.scatter(x, y, z, c=colors, s=sizes, alpha=0.6, cmap='viridis')

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_zlabel('Z轴')
ax.set_title('3D散点图')

plt.colorbar(scatter, ax=ax, shrink=0.5)
plt.tight_layout()
plt.show()
```

### 8.3 3D曲面图

```python
"""
3D曲面图
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(12, 8))
ax = fig.add_subplot(111, projection='3d')

# 生成网格数据
x = np.linspace(-5, 5, 50)
y = np.linspace(-5, 5, 50)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

# 绘制3D曲面
surf = ax.plot_surface(X, Y, Z, cmap='viridis', alpha=0.8)

ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_zlabel('Z轴')
ax.set_title('3D曲面图')

plt.colorbar(surf, ax=ax, shrink=0.5)
plt.tight_layout()
plt.show()
```

---

## 9. 动画和交互

### 9.1 简单动画

```python
"""
简单动画示例
@author erik.zhou
"""
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

fig, ax = plt.subplots(figsize=(10, 6))

x = np.linspace(0, 2*np.pi, 100)
line, = ax.plot(x, np.sin(x))

ax.set_xlim(0, 2*np.pi)
ax.set_ylim(-1.5, 1.5)
ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('正弦波动画')
ax.grid(True, alpha=0.3)

def animate(frame):
    """动画更新函数"""
    line.set_ydata(np.sin(x + frame/10))
    return line,

# 创建动画
anim = animation.FuncAnimation(fig, animate, frames=100,
                              interval=50, blit=True)

# 保存动画（需要安装ffmpeg）
# anim.save('sine_wave.gif', writer='pillow', fps=20)

plt.show()
```

### 9.2 交互式图表

```python
"""
交互式图表
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(10, 6))

x = np.linspace(0, 10, 100)
y = np.sin(x)

line, = ax.plot(x, y, linewidth=2)
ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('交互式图表（点击图表）')
ax.grid(True, alpha=0.3)

# 添加点击事件
def onclick(event):
    """点击事件处理"""
    if event.inaxes == ax:
        print(f'点击位置: x={event.xdata:.2f}, y={event.ydata:.2f}')
        ax.plot(event.xdata, event.ydata, 'ro', markersize=10)
        fig.canvas.draw()

fig.canvas.mpl_connect('button_press_event', onclick)

plt.show()
```

---

## 10. 最佳实践

### 10.1 图表保存

```python
"""
图表保存
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)

fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(x, y, linewidth=2)
ax.set_xlabel('X轴')
ax.set_ylabel('Y轴')
ax.set_title('图表保存示例')
ax.grid(True, alpha=0.3)

# 保存为不同格式
plt.savefig('figure.png', dpi=300, bbox_inches='tight')  # PNG格式
plt.savefig('figure.pdf', bbox_inches='tight')  # PDF格式
plt.savefig('figure.svg', bbox_inches='tight')  # SVG格式

print("图表已保存")
plt.show()
```

### 10.2 性能优化

```python
"""
性能优化技巧
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np
import time

# 大数据量绘图优化
n = 1000000
x = np.random.randn(n)
y = np.random.randn(n)

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# 方法1: 直接绘制（慢）
start = time.time()
axes[0].scatter(x[:10000], y[:10000], alpha=0.5, s=1)
axes[0].set_title(f'直接绘制 ({time.time()-start:.2f}秒)')

# 方法2: 使用rasterized（快）
start = time.time()
axes[1].scatter(x[:10000], y[:10000], alpha=0.5, s=1, rasterized=True)
axes[1].set_title(f'使用rasterized ({time.time()-start:.2f}秒)')

plt.tight_layout()
plt.show()
```

### 10.3 代码规范

```python
"""
Matplotlib代码规范
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 1. 使用面向对象接口（推荐）
fig, ax = plt.subplots(figsize=(10, 6))

x = np.linspace(0, 10, 100)
y = np.sin(x)

# 2. 设置合适的图表大小
ax.plot(x, y, linewidth=2, label='sin(x)')

# 3. 添加清晰的标签和标题
ax.set_xlabel('时间 (秒)', fontsize=12)
ax.set_ylabel('振幅', fontsize=12)
ax.set_title('正弦波形图', fontsize=14, fontweight='bold')

# 4. 添加图例
ax.legend(loc='best', fontsize=10)

# 5. 添加网格
ax.grid(True, alpha=0.3, linestyle='--')

# 6. 使用tight_layout避免标签被裁剪
plt.tight_layout()

# 7. 保存高质量图片
plt.savefig('sine_wave.png', dpi=300, bbox_inches='tight')

plt.show()
```

### 10.4 常见陷阱

```python
"""
Matplotlib常见陷阱
@author erik.zhou
"""
import matplotlib.pyplot as plt
import numpy as np

# 陷阱1: 忘记调用plt.show()
fig, ax = plt.subplots()
ax.plot([1, 2, 3], [1, 2, 3])
# plt.show()  # 必须调用才能显示

# 陷阱2: 混用pyplot和面向对象接口
# 不推荐
plt.figure()
plt.plot([1, 2, 3])
# 推荐
fig, ax = plt.subplots()
ax.plot([1, 2, 3])

# 陷阱3: 不使用tight_layout导致标签被裁剪
fig, ax = plt.subplots()
ax.plot([1, 2, 3])
ax.set_xlabel('这是一个很长的X轴标签')
plt.tight_layout()  # 避免标签被裁剪

# 陷阱4: 在循环中创建过多图形
# 不推荐
for i in range(10):
    plt.figure()  # 会创建10个图形
    plt.plot([1, 2, 3])

# 推荐
fig, axes = plt.subplots(2, 5, figsize=(15, 6))
for i, ax in enumerate(axes.flatten()):
    ax.plot([1, 2, 3])

plt.show()
```

---

## 📝 学习检查清单

- [ ] 理解Matplotlib的基本架构
- [ ] 能够绘制各种基础图表
- [ ] 掌握图表样式自定义
- [ ] 能够创建复杂的子图布局
- [ ] 掌握坐标轴和刻度设置
- [ ] 能够添加文本和注释
- [ ] 掌握高级图表类型
- [ ] 了解3D图表绘制
- [ ] 能够创建动画和交互式图表
- [ ] 理解性能优化技巧

## 🔗 相关资源

- [Matplotlib官方文档](https://matplotlib.org/stable/contents.html)
- [Matplotlib教程](https://matplotlib.org/stable/tutorials/index.html)
- [Matplotlib图表库](https://matplotlib.org/stable/gallery/index.html)
- [Python数据可视化](https://www.datacamp.com/tutorial/matplotlib-tutorial-python)

---

**@author erik.zhou**  
**最后更新**: 2026-02-13
