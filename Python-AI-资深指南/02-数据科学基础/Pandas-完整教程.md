# Pandas完整教程

> 掌握Pandas数据分析，高效处理结构化数据
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: Pandas 2.0+  
**学习难度**: ⭐⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐⭐  
**预计学习时长**: 20-30小时

## 🎯 学习目标

- [ ] 理解Series和DataFrame数据结构
- [ ] 掌握数据读取和写入
- [ ] 能够进行数据清洗和预处理
- [ ] 掌握数据选择和过滤
- [ ] 理解数据分组和聚合
- [ ] 能够进行数据合并和连接
- [ ] 掌握时间序列处理
- [ ] 理解数据可视化基础

## 📖 目录

1. [Pandas基础](#1-pandas基础)
2. [数据结构](#2-数据结构)
3. [数据读写](#3-数据读写)
4. [数据选择](#4-数据选择)
5. [数据清洗](#5-数据清洗)
6. [数据转换](#6-数据转换)
7. [数据分组](#7-数据分组)
8. [数据合并](#8-数据合并)
9. [时间序列](#9-时间序列)
10. [最佳实践](#10-最佳实践)

---

## 1. Pandas基础

### 1.1 安装和导入

```python
"""
Pandas安装和导入
@author erik.zhou
"""
# 安装
# pip install pandas

import pandas as pd
import numpy as np

# 查看版本
print(f"Pandas版本: {pd.__version__}")
```

### 1.2 基本配置

```python
"""
Pandas基本配置
@author erik.zhou
"""
import pandas as pd

# 设置显示选项
pd.set_option('display.max_rows', 100)  # 最大显示行数
pd.set_option('display.max_columns', 20)  # 最大显示列数
pd.set_option('display.width', 1000)  # 显示宽度
pd.set_option('display.precision', 2)  # 浮点数精度

# 查看所有选项
# pd.describe_option()

# 重置选项
# pd.reset_option('all')
```

---

## 2. 数据结构

### 2.1 Series

```python
"""
Series数据结构
@author erik.zhou
"""
import pandas as pd
import numpy as np

# 从列表创建
s1 = pd.Series([1, 2, 3, 4, 5])
print(f"Series:\n{s1}\n")

# 指定索引
s2 = pd.Series([1, 2, 3], index=['a', 'b', 'c'])
print(f"带索引的Series:\n{s2}\n")

# 从字典创建
s3 = pd.Series({'a': 1, 'b': 2, 'c': 3})
print(f"从字典创建:\n{s3}\n")

# Series属性
print(f"值: {s2.values}")
print(f"索引: {s2.index}")
print(f"数据类型: {s2.dtype}")
print(f"形状: {s2.shape}")
print(f"大小: {s2.size}")
```

### 2.2 DataFrame

```python
"""
DataFrame数据结构
@author erik.zhou
"""
import pandas as pd

# 从字典创建
data = {
    'name': ['Alice', 'Bob', 'Charlie', 'David'],
    'age': [25, 30, 35, 40],
    'city': ['Beijing', 'Shanghai', 'Guangzhou', 'Shenzhen']
}
df = pd.DataFrame(data)
print(f"DataFrame:\n{df}\n")

# 从列表创建
data_list = [
    ['Alice', 25, 'Beijing'],
    ['Bob', 30, 'Shanghai'],
    ['Charlie', 35, 'Guangzhou']
]
df2 = pd.DataFrame(data_list, columns=['name', 'age', 'city'])
print(f"从列表创建:\n{df2}\n")

# DataFrame属性
print(f"形状: {df.shape}")
print(f"列名: {df.columns.tolist()}")
print(f"索引: {df.index.tolist()}")
print(f"数据类型:\n{df.dtypes}\n")
print(f"信息:\n{df.info()}\n")
print(f"描述统计:\n{df.describe()}")
```

---

## 3. 数据读写

### 3.1 读取数据

```python
"""
读取各种格式的数据
@author erik.zhou
"""
import pandas as pd

# 读取CSV
# df_csv = pd.read_csv('data.csv', encoding='utf-8')

# 读取Excel
# df_excel = pd.read_excel('data.xlsx', sheet_name='Sheet1')

# 读取JSON
# df_json = pd.read_json('data.json')

# 读取SQL
# from sqlalchemy import create_engine
# engine = create_engine('postgresql://user:password@localhost:5432/dbname')
# df_sql = pd.read_sql('SELECT * FROM table_name', engine)

# 读取HTML表格
# df_html = pd.read_html('https://example.com/table.html')[0]

# 示例：创建示例数据
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['Beijing', 'Shanghai', 'Guangzhou']
})
print(f"示例数据:\n{df}")
```

### 3.2 写入数据

```python
"""
写入各种格式的数据
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'city': ['Beijing', 'Shanghai', 'Guangzhou']
})

# 写入CSV
df.to_csv('output.csv', index=False, encoding='utf-8')

# 写入Excel
# df.to_excel('output.xlsx', sheet_name='Sheet1', index=False)

# 写入JSON
df.to_json('output.json', orient='records', force_ascii=False)

# 写入SQL
# from sqlalchemy import create_engine
# engine = create_engine('postgresql://user:password@localhost:5432/dbname')
# df.to_sql('table_name', engine, if_exists='replace', index=False)

print("数据已写入文件")
```

---

## 4. 数据选择

### 4.1 列选择

```python
"""
列选择操作
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'David'],
    'age': [25, 30, 35, 40],
    'city': ['Beijing', 'Shanghai', 'Guangzhou', 'Shenzhen'],
    'salary': [5000, 6000, 7000, 8000]
})

# 选择单列
name_series = df['name']
print(f"单列（Series）:\n{name_series}\n")

# 选择多列
subset = df[['name', 'age']]
print(f"多列（DataFrame）:\n{subset}\n")

# 使用点号访问
ages = df.age
print(f"点号访问:\n{ages}")
```

### 4.2 行选择

```python
"""
行选择操作
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'David'],
    'age': [25, 30, 35, 40],
    'city': ['Beijing', 'Shanghai', 'Guangzhou', 'Shenzhen']
})

# 使用iloc（位置索引）
first_row = df.iloc[0]
print(f"第一行:\n{first_row}\n")

first_three = df.iloc[0:3]
print(f"前三行:\n{first_three}\n")

# 使用loc（标签索引）
row_by_label = df.loc[0]
print(f"标签索引:\n{row_by_label}\n")

# 条件选择
adults = df[df['age'] > 30]
print(f"年龄大于30:\n{adults}\n")

# 多条件选择
filtered = df[(df['age'] > 25) & (df['age'] < 40)]
print(f"年龄在25-40之间:\n{filtered}")
```

### 4.3 混合选择

```python
"""
混合选择操作
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie', 'David'],
    'age': [25, 30, 35, 40],
    'city': ['Beijing', 'Shanghai', 'Guangzhou', 'Shenzhen'],
    'salary': [5000, 6000, 7000, 8000]
})

# iloc选择特定位置
value = df.iloc[0, 1]  # 第0行第1列
print(f"特定位置的值: {value}")

# loc选择特定标签
value = df.loc[0, 'age']
print(f"特定标签的值: {value}")

# 选择行和列的子集
subset = df.loc[0:2, ['name', 'age']]
print(f"子集:\n{subset}\n")

# 使用at和iat快速访问单个值
value_at = df.at[0, 'name']
value_iat = df.iat[0, 0]
print(f"at访问: {value_at}")
print(f"iat访问: {value_iat}")
```

---

## 5. 数据清洗

### 5.1 处理缺失值

```python
"""
处理缺失值
@author erik.zhou
"""
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'A': [1, 2, np.nan, 4],
    'B': [5, np.nan, np.nan, 8],
    'C': [9, 10, 11, 12]
})
print(f"原始数据:\n{df}\n")

# 检测缺失值
print(f"缺失值检测:\n{df.isnull()}\n")
print(f"缺失值统计:\n{df.isnull().sum()}\n")

# 删除缺失值
df_dropna = df.dropna()  # 删除包含缺失值的行
print(f"删除缺失值:\n{df_dropna}\n")

df_dropna_col = df.dropna(axis=1)  # 删除包含缺失值的列
print(f"删除缺失列:\n{df_dropna_col}\n")

# 填充缺失值
df_fillna = df.fillna(0)  # 用0填充
print(f"填充0:\n{df_fillna}\n")

df_fillna_mean = df.fillna(df.mean())  # 用均值填充
print(f"填充均值:\n{df_fillna_mean}\n")

# 前向填充和后向填充
df_ffill = df.fillna(method='ffill')  # 前向填充
df_bfill = df.fillna(method='bfill')  # 后向填充
print(f"前向填充:\n{df_ffill}\n")
print(f"后向填充:\n{df_bfill}")
```

### 5.2 处理重复值

```python
"""
处理重复值
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Alice', 'Charlie', 'Bob'],
    'age': [25, 30, 25, 35, 30],
    'city': ['Beijing', 'Shanghai', 'Beijing', 'Guangzhou', 'Shanghai']
})
print(f"原始数据:\n{df}\n")

# 检测重复值
print(f"重复行:\n{df.duplicated()}\n")

# 删除重复值
df_drop_duplicates = df.drop_duplicates()
print(f"删除重复行:\n{df_drop_duplicates}\n")

# 基于特定列删除重复
df_drop_by_name = df.drop_duplicates(subset=['name'])
print(f"基于name列删除重复:\n{df_drop_by_name}\n")

# 保留最后一个重复项
df_keep_last = df.drop_duplicates(keep='last')
print(f"保留最后一个:\n{df_keep_last}")
```

### 5.3 数据类型转换

```python
"""
数据类型转换
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'A': ['1', '2', '3'],
    'B': ['4.5', '5.5', '6.5'],
    'C': ['2024-01-01', '2024-01-02', '2024-01-03']
})
print(f"原始数据类型:\n{df.dtypes}\n")

# 转换为数值类型
df['A'] = df['A'].astype(int)
df['B'] = df['B'].astype(float)
print(f"转换后:\n{df.dtypes}\n")

# 转换为日期类型
df['C'] = pd.to_datetime(df['C'])
print(f"日期转换后:\n{df.dtypes}\n")

# 转换为分类类型
df_cat = pd.DataFrame({
    'grade': ['A', 'B', 'A', 'C', 'B']
})
df_cat['grade'] = df_cat['grade'].astype('category')
print(f"分类类型:\n{df_cat.dtypes}")
```

---

## 6. 数据转换

### 6.1 应用函数

```python
"""
应用函数
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'name': ['alice', 'bob', 'charlie'],
    'age': [25, 30, 35],
    'salary': [5000, 6000, 7000]
})

# apply应用函数
df['name_upper'] = df['name'].apply(lambda x: x.upper())
print(f"应用函数:\n{df}\n")

# map映射
grade_map = {25: 'Junior', 30: 'Mid', 35: 'Senior'}
df['level'] = df['age'].map(grade_map)
print(f"映射:\n{df}\n")

# applymap应用到所有元素（DataFrame）
df_numeric = df[['age', 'salary']]
df_doubled = df_numeric.applymap(lambda x: x * 2)
print(f"applymap:\n{df_doubled}")
```

### 6.2 字符串操作

```python
"""
字符串操作
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'name': ['Alice Smith', 'Bob Jones', 'Charlie Brown'],
    'email': ['alice@example.com', 'bob@example.com', 'charlie@example.com']
})

# 字符串方法
df['first_name'] = df['name'].str.split().str[0]
df['last_name'] = df['name'].str.split().str[1]
print(f"分割字符串:\n{df}\n")

# 大小写转换
df['name_lower'] = df['name'].str.lower()
df['name_upper'] = df['name'].str.upper()
print(f"大小写转换:\n{df[['name', 'name_lower', 'name_upper']]}\n")

# 包含判断
df['has_alice'] = df['name'].str.contains('Alice')
print(f"包含判断:\n{df[['name', 'has_alice']]}\n")

# 替换
df['email_masked'] = df['email'].str.replace('@', '[at]')
print(f"替换:\n{df[['email', 'email_masked']]}")
```

### 6.3 数据重塑

```python
"""
数据重塑
@author erik.zhou
"""
import pandas as pd

# pivot - 透视表
df = pd.DataFrame({
    'date': ['2024-01-01', '2024-01-01', '2024-01-02', '2024-01-02'],
    'city': ['Beijing', 'Shanghai', 'Beijing', 'Shanghai'],
    'temperature': [5, 8, 6, 9]
})
print(f"原始数据:\n{df}\n")

df_pivot = df.pivot(index='date', columns='city', values='temperature')
print(f"透视表:\n{df_pivot}\n")

# melt - 逆透视
df_melt = df_pivot.reset_index().melt(id_vars=['date'], var_name='city', value_name='temperature')
print(f"逆透视:\n{df_melt}\n")

# stack和unstack
df_stacked = df_pivot.stack()
print(f"stack:\n{df_stacked}\n")

df_unstacked = df_stacked.unstack()
print(f"unstack:\n{df_unstacked}")
```

---

## 7. 数据分组

### 7.1 groupby基础

```python
"""
groupby基础操作
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'department': ['IT', 'IT', 'HR', 'HR', 'Sales', 'Sales'],
    'name': ['Alice', 'Bob', 'Charlie', 'David', 'Eve', 'Frank'],
    'salary': [5000, 6000, 4500, 5500, 7000, 6500]
})

# 分组聚合
grouped = df.groupby('department')['salary'].mean()
print(f"按部门平均工资:\n{grouped}\n")

# 多列聚合
agg_result = df.groupby('department').agg({
    'salary': ['mean', 'sum', 'count']
})
print(f"多列聚合:\n{agg_result}\n")

# 自定义聚合函数
def salary_range(x):
    return x.max() - x.min()

custom_agg = df.groupby('department')['salary'].agg([
    ('平均工资', 'mean'),
    ('工资范围', salary_range)
])
print(f"自定义聚合:\n{custom_agg}")
```

### 7.2 分组转换

```python
"""
分组转换操作
@author erik.zhou
"""
import pandas as pd

df = pd.DataFrame({
    'department': ['IT', 'IT', 'HR', 'HR', 'Sales', 'Sales'],
    'name': ['Alice', 'Bob', 'Charlie', 'David', 'Eve', 'Frank'],
    'salary': [5000, 6000, 4500, 5500, 7000, 6500]
})

# transform - 保持原始形状
df['dept_avg_salary'] = df.groupby('department')['salary'].transform('mean')
print(f"部门平均工资:\n{df}\n")

# 标准化
df['salary_normalized'] = df.groupby('department')['salary'].transform(
    lambda x: (x - x.mean()) / x.std()
)
print(f"标准化工资:\n{df[['name', 'department', 'salary', 'salary_normalized']]}")
```

---

## 8. 数据合并

### 8.1 concat连接

```python
"""
concat连接操作
@author erik.zhou
"""
import pandas as pd

df1 = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df2 = pd.DataFrame({'A': [5, 6], 'B': [7, 8]})

# 垂直连接
result_vertical = pd.concat([df1, df2], ignore_index=True)
print(f"垂直连接:\n{result_vertical}\n")

# 水平连接
result_horizontal = pd.concat([df1, df2], axis=1)
print(f"水平连接:\n{result_horizontal}")
```

### 8.2 merge合并

```python
"""
merge合并操作
@author erik.zhou
"""
import pandas as pd

df_employees = pd.DataFrame({
    'emp_id': [1, 2, 3, 4],
    'name': ['Alice', 'Bob', 'Charlie', 'David'],
    'dept_id': [10, 20, 10, 30]
})

df_departments = pd.DataFrame({
    'dept_id': [10, 20, 30],
    'dept_name': ['IT', 'HR', 'Sales']
})

# 内连接
inner_join = pd.merge(df_employees, df_departments, on='dept_id', how='inner')
print(f"内连接:\n{inner_join}\n")

# 左连接
left_join = pd.merge(df_employees, df_departments, on='dept_id', how='left')
print(f"左连接:\n{left_join}\n")

# 右连接
right_join = pd.merge(df_employees, df_departments, on='dept_id', how='right')
print(f"右连接:\n{right_join}\n")

# 外连接
outer_join = pd.merge(df_employees, df_departments, on='dept_id', how='outer')
print(f"外连接:\n{outer_join}")
```

### 8.3 join连接

```python
"""
join连接操作
@author erik.zhou
"""
import pandas as pd

df1 = pd.DataFrame({'A': [1, 2, 3]}, index=['a', 'b', 'c'])
df2 = pd.DataFrame({'B': [4, 5, 6]}, index=['a', 'b', 'd'])

# join（基于索引）
result = df1.join(df2, how='inner')
print(f"join结果:\n{result}")
```

---

## 9. 时间序列

### 9.1 日期时间处理

```python
"""
日期时间处理
@author erik.zhou
"""
import pandas as pd

# 创建日期范围
dates = pd.date_range('2024-01-01', periods=10, freq='D')
print(f"日期范围:\n{dates}\n")

# 创建时间序列
ts = pd.Series(range(10), index=dates)
print(f"时间序列:\n{ts}\n")

# 日期时间属性
df = pd.DataFrame({'date': dates})
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['day'] = df['date'].dt.day
df['dayofweek'] = df['date'].dt.dayofweek
print(f"日期属性:\n{df}")
```

### 9.2 时间序列操作

```python
"""
时间序列操作
@author erik.zhou
"""
import pandas as pd
import numpy as np

# 创建时间序列数据
dates = pd.date_range('2024-01-01', periods=100, freq='D')
ts = pd.Series(np.random.randn(100), index=dates)

# 重采样
monthly = ts.resample('M').mean()
print(f"月度重采样:\n{monthly.head()}\n")

# 滚动窗口
rolling_mean = ts.rolling(window=7).mean()
print(f"7天滚动均值:\n{rolling_mean.head(10)}\n")

# 时间偏移
ts_shifted = ts.shift(1)  # 向后移动1天
print(f"偏移后:\n{ts_shifted.head()}")
```

---

## 10. 最佳实践

### 10.1 性能优化

```python
"""
性能优化技巧
@author erik.zhou
"""
import pandas as pd
import numpy as np

# 1. 使用向量化操作
df = pd.DataFrame({'A': range(1000000)})

# 好的做法
df['B'] = df['A'] * 2

# 避免的做法
# df['B'] = df['A'].apply(lambda x: x * 2)

# 2. 使用category类型
df_cat = pd.DataFrame({
    'grade': ['A', 'B', 'C'] * 100000
})
df_cat['grade'] = df_cat['grade'].astype('category')
print(f"内存使用: {df_cat.memory_usage(deep=True)['grade']} bytes")

# 3. 使用query方法
df = pd.DataFrame({
    'A': range(100),
    'B': range(100, 200)
})
# 好的做法
result = df.query('A > 50 and B < 150')
# 等价于
# result = df[(df['A'] > 50) & (df['B'] < 150)]
```

### 10.2 代码规范

```python
"""
Pandas代码规范
@author erik.zhou
"""
import pandas as pd

# 1. 链式操作
df = pd.DataFrame({
    'name': ['alice', 'bob', 'charlie'],
    'age': [25, 30, 35],
    'salary': [5000, 6000, 7000]
})

# 好的做法（链式操作）
result = (df
    .assign(name_upper=lambda x: x['name'].str.upper())
    .query('age > 25')
    .sort_values('salary', ascending=False)
)
print(f"链式操作结果:\n{result}\n")

# 2. 使用copy避免SettingWithCopyWarning
df_subset = df[df['age'] > 25].copy()
df_subset['new_col'] = 1

# 3. 使用inplace参数要谨慎
# 避免使用inplace=True，除非确实需要
df_sorted = df.sort_values('age')  # 返回新DataFrame
# df.sort_values('age', inplace=True)  # 原地修改
```

---

## 📝 学习检查清单

- [ ] 理解Series和DataFrame数据结构
- [ ] 能够读取和写入各种格式的数据
- [ ] 掌握数据选择和过滤
- [ ] 能够处理缺失值和重复值
- [ ] 掌握数据转换和重塑
- [ ] 理解groupby分组操作
- [ ] 能够进行数据合并和连接
- [ ] 掌握时间序列处理
- [ ] 了解性能优化技巧

## 🔗 相关资源

- [Pandas官方文档](https://pandas.pydata.org/docs/)
- [Pandas用户指南](https://pandas.pydata.org/docs/user_guide/index.html)
- [Python for Data Analysis](https://wesmckinney.com/book/)
- [Pandas Cookbook](https://pandas.pydata.org/docs/user_guide/cookbook.html)

---

**@author erik.zhou**  
**最后更新**: 2025-02-12
