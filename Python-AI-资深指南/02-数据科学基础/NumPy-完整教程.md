# NumPy完整教程

> 掌握NumPy数组计算，构建高性能数据处理应用
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: NumPy 1.24+  
**学习难度**: ⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐⭐  
**预计学习时长**: 15-20小时

## 🎯 学习目标

- [ ] 理解NumPy数组的核心概念
- [ ] 掌握数组创建和操作
- [ ] 理解广播机制
- [ ] 掌握数组索引和切片
- [ ] 能够进行数学和统计运算
- [ ] 掌握线性代数操作
- [ ] 理解性能优化技巧

## 📖 目录

1. [NumPy基础](#1-numpy基础)
2. [数组创建](#2-数组创建)
3. [数组操作](#3-数组操作)
4. [索引和切片](#4-索引和切片)
5. [数学运算](#5-数学运算)
6. [统计函数](#6-统计函数)
7. [线性代数](#7-线性代数)
8. [广播机制](#8-广播机制)
9. [性能优化](#9-性能优化)
10. [最佳实践](#10-最佳实践)

---

## 1. NumPy基础

### 1.1 安装和导入

```python
"""
NumPy安装和导入
@author erik.zhou
"""
# 安装
# pip install numpy

import numpy as np

# 查看版本
print(f"NumPy版本: {np.__version__}")
```

### 1.2 ndarray对象

```python
"""
ndarray对象基础
@author erik.zhou
"""
import numpy as np

# 创建数组
arr = np.array([1, 2, 3, 4, 5])
print(f"数组: {arr}")
print(f"类型: {type(arr)}")
print(f"形状: {arr.shape}")
print(f"维度: {arr.ndim}")
print(f"数据类型: {arr.dtype}")
print(f"元素个数: {arr.size}")

# 多维数组
arr_2d = np.array([[1, 2, 3], [4, 5, 6]])
print(f"\n二维数组:\n{arr_2d}")
print(f"形状: {arr_2d.shape}")
print(f"维度: {arr_2d.ndim}")
```

---

## 2. 数组创建

### 2.1 从列表创建

```python
"""
从列表创建数组
@author erik.zhou
"""
import numpy as np

# 一维数组
arr_1d = np.array([1, 2, 3, 4, 5])
print(f"一维数组: {arr_1d}")

# 二维数组
arr_2d = np.array([[1, 2, 3], [4, 5, 6]])
print(f"二维数组:\n{arr_2d}")

# 三维数组
arr_3d = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])
print(f"三维数组:\n{arr_3d}")

# 指定数据类型
arr_float = np.array([1, 2, 3], dtype=np.float64)
print(f"浮点数组: {arr_float}")
```

### 2.2 使用内置函数创建

```python
"""
使用内置函数创建数组
@author erik.zhou
"""
import numpy as np

# zeros - 全零数组
zeros = np.zeros((3, 4))
print(f"全零数组:\n{zeros}")

# ones - 全一数组
ones = np.ones((2, 3))
print(f"全一数组:\n{ones}")

# full - 指定值数组
full = np.full((2, 2), 7)
print(f"指定值数组:\n{full}")

# eye - 单位矩阵
eye = np.eye(3)
print(f"单位矩阵:\n{eye}")

# arange - 等差数列
arange = np.arange(0, 10, 2)
print(f"等差数列: {arange}")

# linspace - 线性空间
linspace = np.linspace(0, 1, 5)
print(f"线性空间: {linspace}")

# random - 随机数组
random = np.random.rand(3, 3)
print(f"随机数组:\n{random}")
```

---

## 3. 数组操作

### 3.1 形状操作

```python
"""
数组形状操作
@author erik.zhou
"""
import numpy as np

arr = np.arange(12)
print(f"原始数组: {arr}")

# reshape - 改变形状
reshaped = arr.reshape(3, 4)
print(f"reshape后:\n{reshaped}")

# resize - 原地改变形状
arr_copy = arr.copy()
arr_copy.resize(3, 4)
print(f"resize后:\n{arr_copy}")

# flatten - 展平为一维
flattened = reshaped.flatten()
print(f"flatten后: {flattened}")

# ravel - 展平（返回视图）
raveled = reshaped.ravel()
print(f"ravel后: {raveled}")

# transpose - 转置
transposed = reshaped.T
print(f"转置后:\n{transposed}")
```

### 3.2 数组拼接

```python
"""
数组拼接操作
@author erik.zhou
"""
import numpy as np

arr1 = np.array([[1, 2], [3, 4]])
arr2 = np.array([[5, 6], [7, 8]])

# concatenate - 沿指定轴拼接
concat_0 = np.concatenate([arr1, arr2], axis=0)
print(f"axis=0拼接:\n{concat_0}")

concat_1 = np.concatenate([arr1, arr2], axis=1)
print(f"axis=1拼接:\n{concat_1}")

# vstack - 垂直堆叠
vstacked = np.vstack([arr1, arr2])
print(f"垂直堆叠:\n{vstacked}")

# hstack - 水平堆叠
hstacked = np.hstack([arr1, arr2])
print(f"水平堆叠:\n{hstacked}")

# stack - 沿新轴堆叠
stacked = np.stack([arr1, arr2], axis=0)
print(f"新轴堆叠:\n{stacked}")
print(f"形状: {stacked.shape}")
```

### 3.3 数组分割

```python
"""
数组分割操作
@author erik.zhou
"""
import numpy as np

arr = np.arange(12).reshape(3, 4)
print(f"原始数组:\n{arr}")

# split - 等分分割
split_result = np.split(arr, 3, axis=0)
print(f"split结果: {len(split_result)}个数组")
for i, sub_arr in enumerate(split_result):
    print(f"子数组{i}:\n{sub_arr}")

# array_split - 不等分分割
array_split_result = np.array_split(arr, 2, axis=1)
print(f"array_split结果:")
for i, sub_arr in enumerate(array_split_result):
    print(f"子数组{i}:\n{sub_arr}")

# vsplit - 垂直分割
vsplit_result = np.vsplit(arr, 3)
print(f"vsplit结果: {len(vsplit_result)}个数组")

# hsplit - 水平分割
hsplit_result = np.hsplit(arr, 2)
print(f"hsplit结果: {len(hsplit_result)}个数组")
```

---

## 4. 索引和切片

### 4.1 基础索引

```python
"""
基础索引操作
@author erik.zhou
"""
import numpy as np

# 一维数组索引
arr_1d = np.array([10, 20, 30, 40, 50])
print(f"第一个元素: {arr_1d[0]}")
print(f"最后一个元素: {arr_1d[-1]}")
print(f"切片[1:4]: {arr_1d[1:4]}")
print(f"步长切片[::2]: {arr_1d[::2]}")

# 二维数组索引
arr_2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print(f"\n二维数组:\n{arr_2d}")
print(f"元素[0, 0]: {arr_2d[0, 0]}")
print(f"元素[1, 2]: {arr_2d[1, 2]}")
print(f"第一行: {arr_2d[0, :]}")
print(f"第二列: {arr_2d[:, 1]}")
print(f"子矩阵:\n{arr_2d[0:2, 1:3]}")
```

### 4.2 布尔索引

```python
"""
布尔索引操作
@author erik.zhou
"""
import numpy as np

arr = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# 条件筛选
mask = arr > 5
print(f"大于5的元素: {arr[mask]}")

# 多条件筛选
mask = (arr > 3) & (arr < 8)
print(f"3到8之间的元素: {arr[mask]}")

# 二维数组布尔索引
arr_2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
mask_2d = arr_2d > 5
print(f"大于5的元素: {arr_2d[mask_2d]}")

# 修改满足条件的元素
arr_copy = arr.copy()
arr_copy[arr_copy > 5] = 0
print(f"修改后: {arr_copy}")
```

### 4.3 花式索引

```python
"""
花式索引操作
@author erik.zhou
"""
import numpy as np

arr = np.array([10, 20, 30, 40, 50])

# 整数数组索引
indices = [0, 2, 4]
print(f"索引[0,2,4]的元素: {arr[indices]}")

# 二维数组花式索引
arr_2d = np.arange(12).reshape(3, 4)
print(f"原始数组:\n{arr_2d}")

# 选择特定行
rows = [0, 2]
print(f"选择行[0,2]:\n{arr_2d[rows]}")

# 选择特定元素
rows = [0, 1, 2]
cols = [1, 2, 3]
print(f"选择元素: {arr_2d[rows, cols]}")

# 组合索引
print(f"组合索引:\n{arr_2d[[0, 2]][:, [1, 3]]}")
```

---

## 5. 数学运算

### 5.1 基础运算

```python
"""
基础数学运算
@author erik.zhou
"""
import numpy as np

arr1 = np.array([1, 2, 3, 4])
arr2 = np.array([5, 6, 7, 8])

# 算术运算
print(f"加法: {arr1 + arr2}")
print(f"减法: {arr1 - arr2}")
print(f"乘法: {arr1 * arr2}")
print(f"除法: {arr1 / arr2}")
print(f"幂运算: {arr1 ** 2}")
print(f"取余: {arr2 % arr1}")

# 标量运算
print(f"数组+10: {arr1 + 10}")
print(f"数组*2: {arr1 * 2}")

# 比较运算
print(f"arr1 > 2: {arr1 > 2}")
print(f"arr1 == arr2: {arr1 == arr2}")
```

### 5.2 通用函数

```python
"""
通用函数（ufunc）
@author erik.zhou
"""
import numpy as np

arr = np.array([1, 4, 9, 16, 25])

# 数学函数
print(f"平方根: {np.sqrt(arr)}")
print(f"指数: {np.exp(np.array([1, 2, 3]))}")
print(f"对数: {np.log(arr)}")
print(f"绝对值: {np.abs(np.array([-1, -2, 3]))}")

# 三角函数
angles = np.array([0, np.pi/2, np.pi])
print(f"sin: {np.sin(angles)}")
print(f"cos: {np.cos(angles)}")
print(f"tan: {np.tan(angles)}")

# 取整函数
arr_float = np.array([1.2, 2.5, 3.7, 4.9])
print(f"向下取整: {np.floor(arr_float)}")
print(f"向上取整: {np.ceil(arr_float)}")
print(f"四舍五入: {np.round(arr_float)}")
```

### 5.3 聚合函数

```python
"""
聚合函数
@author erik.zhou
"""
import numpy as np

arr = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])

# 基础聚合
print(f"求和: {np.sum(arr)}")
print(f"均值: {np.mean(arr)}")
print(f"最大值: {np.max(arr)}")
print(f"最小值: {np.min(arr)}")
print(f"标准差: {np.std(arr)}")
print(f"方差: {np.var(arr)}")

# 沿轴聚合
print(f"按行求和: {np.sum(arr, axis=0)}")
print(f"按列求和: {np.sum(arr, axis=1)}")

# 累积函数
arr_1d = np.array([1, 2, 3, 4, 5])
print(f"累积和: {np.cumsum(arr_1d)}")
print(f"累积积: {np.cumprod(arr_1d)}")
```

---

## 6. 统计函数

### 6.1 描述性统计

```python
"""
描述性统计
@author erik.zhou
"""
import numpy as np

data = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])

# 中心趋势
print(f"均值: {np.mean(data)}")
print(f"中位数: {np.median(data)}")

# 离散程度
print(f"标准差: {np.std(data)}")
print(f"方差: {np.var(data)}")
print(f"极差: {np.ptp(data)}")

# 分位数
print(f"25%分位数: {np.percentile(data, 25)}")
print(f"50%分位数: {np.percentile(data, 50)}")
print(f"75%分位数: {np.percentile(data, 75)}")

# 相关性
data1 = np.array([1, 2, 3, 4, 5])
data2 = np.array([2, 4, 6, 8, 10])
correlation = np.corrcoef(data1, data2)
print(f"相关系数矩阵:\n{correlation}")
```

### 6.2 随机数生成

```python
"""
随机数生成
@author erik.zhou
"""
import numpy as np

# 设置随机种子
np.random.seed(42)

# 均匀分布
uniform = np.random.rand(3, 3)
print(f"均匀分布[0,1):\n{uniform}")

# 标准正态分布
normal = np.random.randn(3, 3)
print(f"标准正态分布:\n{normal}")

# 指定范围的随机整数
randint = np.random.randint(0, 10, size=(3, 3))
print(f"随机整数[0,10):\n{randint}")

# 正态分布
normal_custom = np.random.normal(loc=0, scale=1, size=1000)
print(f"正态分布均值: {np.mean(normal_custom):.2f}")
print(f"正态分布标准差: {np.std(normal_custom):.2f}")

# 随机选择
arr = np.array([1, 2, 3, 4, 5])
choice = np.random.choice(arr, size=3, replace=False)
print(f"随机选择: {choice}")

# 随机打乱
arr_copy = arr.copy()
np.random.shuffle(arr_copy)
print(f"随机打乱: {arr_copy}")
```

---

## 7. 线性代数

### 7.1 矩阵运算

```python
"""
矩阵运算
@author erik.zhou
"""
import numpy as np

A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

# 矩阵乘法
print(f"矩阵乘法:\n{np.dot(A, B)}")
print(f"矩阵乘法（@运算符）:\n{A @ B}")

# 矩阵转置
print(f"转置:\n{A.T}")

# 矩阵的迹
print(f"迹: {np.trace(A)}")

# 矩阵行列式
print(f"行列式: {np.linalg.det(A)}")

# 矩阵的逆
print(f"逆矩阵:\n{np.linalg.inv(A)}")

# 矩阵的秩
print(f"秩: {np.linalg.matrix_rank(A)}")
```

### 7.2 特征值和特征向量

```python
"""
特征值和特征向量
@author erik.zhou
"""
import numpy as np

A = np.array([[1, 2], [2, 1]])

# 计算特征值和特征向量
eigenvalues, eigenvectors = np.linalg.eig(A)
print(f"特征值: {eigenvalues}")
print(f"特征向量:\n{eigenvectors}")

# 验证
for i in range(len(eigenvalues)):
    lambda_i = eigenvalues[i]
    v_i = eigenvectors[:, i]
    print(f"\n验证特征值{i+1}:")
    print(f"A @ v = {A @ v_i}")
    print(f"λ * v = {lambda_i * v_i}")
```

### 7.3 线性方程组

```python
"""
线性方程组求解
@author erik.zhou
"""
import numpy as np

# 求解 Ax = b
A = np.array([[3, 1], [1, 2]])
b = np.array([9, 8])

# 使用solve求解
x = np.linalg.solve(A, b)
print(f"解: {x}")

# 验证
print(f"验证 Ax = {A @ x}")
print(f"b = {b}")

# 最小二乘解（超定方程组）
A_over = np.array([[1, 1], [1, 2], [1, 3]])
b_over = np.array([2, 3, 4])
x_lstsq, residuals, rank, s = np.linalg.lstsq(A_over, b_over, rcond=None)
print(f"最小二乘解: {x_lstsq}")
```

---

## 8. 广播机制

### 8.1 广播规则

```python
"""
广播机制示例
@author erik.zhou
"""
import numpy as np

# 标量与数组
arr = np.array([1, 2, 3, 4])
result = arr + 10
print(f"标量广播: {result}")

# 一维与二维
arr_1d = np.array([1, 2, 3])
arr_2d = np.array([[1], [2], [3]])
result = arr_1d + arr_2d
print(f"广播结果:\n{result}")

# 不同形状的数组
arr1 = np.array([[1, 2, 3]])  # (1, 3)
arr2 = np.array([[1], [2], [3]])  # (3, 1)
result = arr1 + arr2  # (3, 3)
print(f"广播结果:\n{result}")
```

### 8.2 广播应用

```python
"""
广播应用示例
@author erik.zhou
"""
import numpy as np

# 标准化数据
data = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
mean = np.mean(data, axis=0)
std = np.std(data, axis=0)
normalized = (data - mean) / std
print(f"标准化数据:\n{normalized}")

# 距离计算
points = np.array([[1, 2], [3, 4], [5, 6]])
center = np.array([3, 3])
distances = np.sqrt(np.sum((points - center) ** 2, axis=1))
print(f"到中心的距离: {distances}")
```

---

## 9. 性能优化

### 9.1 向量化操作

```python
"""
向量化操作示例
@author erik.zhou
"""
import numpy as np
import time

# 非向量化
def non_vectorized(n):
    """非向量化计算"""
    result = []
    for i in range(n):
        result.append(i ** 2)
    return result

# 向量化
def vectorized(n):
    """向量化计算"""
    arr = np.arange(n)
    return arr ** 2

# 性能对比
n = 1000000

start = time.time()
result1 = non_vectorized(n)
time1 = time.time() - start

start = time.time()
result2 = vectorized(n)
time2 = time.time() - start

print(f"非向量化耗时: {time1:.4f}秒")
print(f"向量化耗时: {time2:.4f}秒")
print(f"加速比: {time1/time2:.2f}x")
```

### 9.2 内存优化

```python
"""
内存优化示例
@author erik.zhou
"""
import numpy as np

# 使用视图而非副本
arr = np.arange(1000000)
view = arr[::2]  # 视图，不占用额外内存
copy = arr[::2].copy()  # 副本，占用额外内存

print(f"原数组是否拥有数据: {arr.flags['OWNDATA']}")
print(f"视图是否拥有数据: {view.flags['OWNDATA']}")
print(f"副本是否拥有数据: {copy.flags['OWNDATA']}")

# 原地操作
arr = np.arange(10)
arr += 10  # 原地操作
print(f"原地操作后: {arr}")

# 指定数据类型节省内存
arr_int64 = np.array([1, 2, 3], dtype=np.int64)
arr_int8 = np.array([1, 2, 3], dtype=np.int8)
print(f"int64内存: {arr_int64.nbytes}字节")
print(f"int8内存: {arr_int8.nbytes}字节")
```

---

## 10. 最佳实践

### 10.1 代码规范

```python
"""
NumPy代码规范
@author erik.zhou
"""
import numpy as np

# 1. 使用向量化操作
arr = np.arange(100)
# 好的做法
result = arr ** 2
# 避免的做法
# result = [x ** 2 for x in arr]

# 2. 预分配数组
n = 1000
# 好的做法
arr = np.zeros(n)
for i in range(n):
    arr[i] = i ** 2
# 避免的做法
# arr = []
# for i in range(n):
#     arr.append(i ** 2)

# 3. 使用合适的数据类型
# 好的做法
arr_int = np.array([1, 2, 3], dtype=np.int32)
# 避免的做法（默认int64可能浪费内存）
# arr = np.array([1, 2, 3])

# 4. 避免不必要的副本
arr = np.arange(100)
# 好的做法（使用视图）
view = arr[::2]
# 避免的做法（创建副本）
# copy = arr[::2].copy()
```

### 10.2 常见陷阱

```python
"""
NumPy常见陷阱
@author erik.zhou
"""
import numpy as np

# 陷阱1: 数组赋值是引用
arr1 = np.array([1, 2, 3])
arr2 = arr1  # 引用，不是副本
arr2[0] = 999
print(f"arr1: {arr1}")  # arr1也被修改了

# 正确做法：使用copy()
arr1 = np.array([1, 2, 3])
arr2 = arr1.copy()
arr2[0] = 999
print(f"arr1: {arr1}")  # arr1不受影响

# 陷阱2: 浮点数比较
a = 0.1 + 0.2
b = 0.3
print(f"a == b: {a == b}")  # False
# 正确做法：使用isclose()
print(f"isclose: {np.isclose(a, b)}")  # True

# 陷阱3: 整数除法
arr = np.array([1, 2, 3])
result = arr / 2
print(f"除法结果类型: {result.dtype}")  # float64

# 陷阱4: 广播陷阱
arr1 = np.array([[1, 2, 3]])  # (1, 3)
arr2 = np.array([[1], [2]])  # (2, 1)
try:
    result = arr1 + arr2  # 可以广播
    print(f"广播结果形状: {result.shape}")
except ValueError as e:
    print(f"广播错误: {e}")
```

---

## 📝 学习检查清单

- [ ] 理解ndarray的核心概念
- [ ] 能够创建各种类型的数组
- [ ] 掌握数组的形状操作
- [ ] 能够使用索引和切片
- [ ] 掌握数学和统计运算
- [ ] 理解线性代数操作
- [ ] 掌握广播机制
- [ ] 能够进行性能优化
- [ ] 了解常见陷阱和最佳实践

## 🔗 相关资源

- [NumPy官方文档](https://numpy.org/doc/stable/)
- [NumPy用户指南](https://numpy.org/doc/stable/user/index.html)
- [NumPy教程](https://numpy.org/doc/stable/user/absolute_beginners.html)
- [From Python to NumPy](https://www.labri.fr/perso/nrougier/from-python-to-numpy/)

---

**@author erik.zhou**  
**最后更新**: 2025-02-12
