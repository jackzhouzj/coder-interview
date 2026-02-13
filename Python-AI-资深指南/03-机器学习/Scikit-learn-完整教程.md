# Scikit-learn完整教程

> 掌握Scikit-learn机器学习框架，构建高效ML应用
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: Scikit-learn 1.3+  
**学习难度**: ⭐⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐⭐  
**预计学习时长**: 30-40小时

## 🎯 学习目标

- [ ] 理解Scikit-learn的核心概念和API设计
- [ ] 掌握监督学习算法（分类和回归）
- [ ] 掌握无监督学习算法（聚类和降维）
- [ ] 能够进行数据预处理和特征工程
- [ ] 理解模型评估和选择方法
- [ ] 掌握管道和网格搜索
- [ ] 能够进行模型持久化

## 📖 目录

1. [Scikit-learn基础](#1-scikit-learn基础)
2. [数据预处理](#2-数据预处理)
3. [监督学习-分类](#3-监督学习-分类)
4. [监督学习-回归](#4-监督学习-回归)
5. [无监督学习](#5-无监督学习)
6. [模型评估](#6-模型评估)
7. [模型选择与调优](#7-模型选择与调优)
8. [管道Pipeline](#8-管道pipeline)
9. [模型持久化](#9-模型持久化)
10. [最佳实践](#10-最佳实践)

---

## 1. Scikit-learn基础

### 1.1 安装和导入

```python
"""
Scikit-learn安装和导入
@author erik.zhou
"""
# 安装
# pip install scikit-learn

import sklearn
from sklearn import datasets
import numpy as np
import pandas as pd

# 查看版本
print(f"Scikit-learn版本: {sklearn.__version__}")
```

### 1.2 基本工作流程

```python
"""
Scikit-learn基本工作流程
@author erik.zhou
"""
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

# 1. 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 2. 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 3. 数据预处理
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 4. 训练模型
model = LogisticRegression(random_state=42)
model.fit(X_train_scaled, y_train)

# 5. 预测
y_pred = model.predict(X_test_scaled)

# 6. 评估
accuracy = accuracy_score(y_test, y_pred)
print(f"准确率: {accuracy:.4f}")
```

---

## 2. 数据预处理

### 2.1 标准化和归一化

```python
"""
数据标准化和归一化
@author erik.zhou
"""
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler
import numpy as np

# 示例数据
X = np.array([[1, 2], [3, 4], [5, 6], [7, 8]])

# StandardScaler - 标准化（均值0，标准差1）
scaler_std = StandardScaler()
X_std = scaler_std.fit_transform(X)
print(f"标准化后:\n{X_std}\n")

# MinMaxScaler - 归一化到[0,1]
scaler_minmax = MinMaxScaler()
X_minmax = scaler_minmax.fit_transform(X)
print(f"归一化后:\n{X_minmax}\n")

# RobustScaler - 对异常值鲁棒的缩放
scaler_robust = RobustScaler()
X_robust = scaler_robust.fit_transform(X)
print(f"鲁棒缩放后:\n{X_robust}")
```

### 2.2 编码分类变量

```python
"""
编码分类变量
@author erik.zhou
"""
from sklearn.preprocessing import LabelEncoder, OneHotEncoder
import numpy as np

# LabelEncoder - 标签编码
le = LabelEncoder()
labels = ['cat', 'dog', 'cat', 'bird', 'dog']
labels_encoded = le.fit_transform(labels)
print(f"标签编码: {labels_encoded}")
print(f"类别: {le.classes_}\n")

# OneHotEncoder - 独热编码
ohe = OneHotEncoder(sparse_output=False)
categories = np.array([['cat'], ['dog'], ['cat'], ['bird'], ['dog']])
categories_encoded = ohe.fit_transform(categories)
print(f"独热编码:\n{categories_encoded}")
print(f"类别: {ohe.categories_}")
```

### 2.3 处理缺失值

```python
"""
处理缺失值
@author erik.zhou
"""
from sklearn.impute import SimpleImputer
import numpy as np

# 示例数据（包含缺失值）
X = np.array([[1, 2], [np.nan, 3], [7, 6], [4, np.nan]])

# 用均值填充
imputer_mean = SimpleImputer(strategy='mean')
X_mean = imputer_mean.fit_transform(X)
print(f"均值填充:\n{X_mean}\n")

# 用中位数填充
imputer_median = SimpleImputer(strategy='median')
X_median = imputer_median.fit_transform(X)
print(f"中位数填充:\n{X_median}\n")

# 用最频繁值填充
imputer_most_frequent = SimpleImputer(strategy='most_frequent')
X_frequent = imputer_most_frequent.fit_transform(X)
print(f"最频繁值填充:\n{X_frequent}")
```

---

## 3. 监督学习-分类

### 3.1 逻辑回归

```python
"""
逻辑回归分类
@author erik.zhou
"""
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import classification_report, confusion_matrix

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 划分数据集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 训练模型
lr = LogisticRegression(max_iter=200, random_state=42)
lr.fit(X_train, y_train)

# 预测
y_pred = lr.predict(X_test)

# 评估
print("混淆矩阵:")
print(confusion_matrix(y_test, y_pred))
print("\n分类报告:")
print(classification_report(y_test, y_pred, target_names=iris.target_names))
```

### 3.2 决策树

```python
"""
决策树分类
@author erik.zhou
"""
from sklearn.tree import DecisionTreeClassifier
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 划分数据集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 训练模型
dt = DecisionTreeClassifier(max_depth=3, random_state=42)
dt.fit(X_train, y_train)

# 预测和评估
y_pred = dt.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print(f"准确率: {accuracy:.4f}")

# 特征重要性
feature_importance = dt.feature_importances_
for i, importance in enumerate(feature_importance):
    print(f"特征{i}: {importance:.4f}")
```

### 3.3 随机森林

```python
"""
随机森林分类
@author erik.zhou
"""
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 划分数据集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 训练模型
rf = RandomForestClassifier(n_estimators=100, max_depth=3, random_state=42)
rf.fit(X_train, y_train)

# 预测和评估
y_pred = rf.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print(f"准确率: {accuracy:.4f}")

# 特征重要性
feature_importance = rf.feature_importances_
for i, importance in enumerate(feature_importance):
    print(f"特征{i}: {importance:.4f}")
```

### 3.4 支持向量机

```python
"""
支持向量机分类
@author erik.zhou
"""
from sklearn.svm import SVC
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import accuracy_score

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 划分数据集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 标准化（SVM对特征缩放敏感）
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 训练模型
svm = SVC(kernel='rbf', C=1.0, gamma='scale', random_state=42)
svm.fit(X_train_scaled, y_train)

# 预测和评估
y_pred = svm.predict(X_test_scaled)
accuracy = accuracy_score(y_test, y_pred)
print(f"准确率: {accuracy:.4f}")
```

---

## 4. 监督学习-回归

### 4.1 线性回归

```python
"""
线性回归
@author erik.zhou
"""
from sklearn.linear_model import LinearRegression
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

# 生成回归数据
X, y = make_regression(n_samples=100, n_features=1, noise=10, random_state=42)

# 划分数据集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 训练模型
lr = LinearRegression()
lr.fit(X_train, y_train)

# 预测
y_pred = lr.predict(X_test)

# 评估
mse = mean_squared_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)
print(f"均方误差: {mse:.4f}")
print(f"R²分数: {r2:.4f}")
print(f"系数: {lr.coef_}")
print(f"截距: {lr.intercept_:.4f}")
```

### 4.2 岭回归和Lasso回归

```python
"""
岭回归和Lasso回归
@author erik.zhou
"""
from sklearn.linear_model import Ridge, Lasso
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error

# 生成数据
X, y = make_regression(n_samples=100, n_features=10, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 岭回归（L2正则化）
ridge = Ridge(alpha=1.0)
ridge.fit(X_train, y_train)
y_pred_ridge = ridge.predict(X_test)
mse_ridge = mean_squared_error(y_test, y_pred_ridge)
print(f"岭回归MSE: {mse_ridge:.4f}")

# Lasso回归（L1正则化）
lasso = Lasso(alpha=1.0)
lasso.fit(X_train, y_train)
y_pred_lasso = lasso.predict(X_test)
mse_lasso = mean_squared_error(y_test, y_pred_lasso)
print(f"Lasso回归MSE: {mse_lasso:.4f}")

# Lasso特征选择
print(f"\nLasso非零系数数量: {np.sum(lasso.coef_ != 0)}")
```

---

## 5. 无监督学习

### 5.1 K-Means聚类

```python
"""
K-Means聚类
@author erik.zhou
"""
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs
import matplotlib.pyplot as plt

# 生成聚类数据
X, y_true = make_blobs(n_samples=300, centers=4, random_state=42)

# K-Means聚类
kmeans = KMeans(n_clusters=4, random_state=42)
y_pred = kmeans.fit_predict(X)

# 聚类中心
centers = kmeans.cluster_centers_
print(f"聚类中心:\n{centers}")

# 惯性（样本到最近聚类中心的距离平方和）
print(f"惯性: {kmeans.inertia_:.4f}")
```

### 5.2 层次聚类

```python
"""
层次聚类
@author erik.zhou
"""
from sklearn.cluster import AgglomerativeClustering
from sklearn.datasets import make_blobs

# 生成数据
X, y_true = make_blobs(n_samples=100, centers=3, random_state=42)

# 层次聚类
agg = AgglomerativeClustering(n_clusters=3, linkage='ward')
y_pred = agg.fit_predict(X)

print(f"聚类标签: {y_pred[:10]}")
```

### 5.3 PCA降维

```python
"""
PCA主成分分析
@author erik.zhou
"""
from sklearn.decomposition import PCA
from sklearn.datasets import load_iris
import numpy as np

# 加载数据
iris = load_iris()
X = iris.data

# PCA降维
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X)

# 解释方差比
print(f"解释方差比: {pca.explained_variance_ratio_}")
print(f"累计解释方差比: {np.cumsum(pca.explained_variance_ratio_)}")

# 主成分
print(f"\n主成分:\n{pca.components_}")
```

---

## 6. 模型评估

### 6.1 分类评估指标

```python
"""
分类评估指标
@author erik.zhou
"""
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, 
    f1_score, roc_auc_score, confusion_matrix
)
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier

# 加载数据
cancer = load_breast_cancer()
X, y = cancer.data, cancer.target

# 划分数据集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 训练模型
rf = RandomForestClassifier(random_state=42)
rf.fit(X_train, y_train)

# 预测
y_pred = rf.predict(X_test)
y_pred_proba = rf.predict_proba(X_test)[:, 1]

# 评估指标
print(f"准确率: {accuracy_score(y_test, y_pred):.4f}")
print(f"精确率: {precision_score(y_test, y_pred):.4f}")
print(f"召回率: {recall_score(y_test, y_pred):.4f}")
print(f"F1分数: {f1_score(y_test, y_pred):.4f}")
print(f"AUC: {roc_auc_score(y_test, y_pred_proba):.4f}")
print(f"\n混淆矩阵:\n{confusion_matrix(y_test, y_pred)}")
```

### 6.2 回归评估指标

```python
"""
回归评估指标
@author erik.zhou
"""
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error, 
    r2_score, mean_absolute_percentage_error
)
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
import numpy as np

# 生成数据
X, y = make_regression(n_samples=100, n_features=5, noise=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 训练模型
lr = LinearRegression()
lr.fit(X_train, y_train)
y_pred = lr.predict(X_test)

# 评估指标
print(f"均方误差(MSE): {mean_squared_error(y_test, y_pred):.4f}")
print(f"均方根误差(RMSE): {np.sqrt(mean_squared_error(y_test, y_pred)):.4f}")
print(f"平均绝对误差(MAE): {mean_absolute_error(y_test, y_pred):.4f}")
print(f"R²分数: {r2_score(y_test, y_pred):.4f}")
print(f"平均绝对百分比误差(MAPE): {mean_absolute_percentage_error(y_test, y_pred):.4f}")
```

---

## 7. 模型选择与调优

### 7.1 交叉验证

```python
"""
交叉验证
@author erik.zhou
"""
from sklearn.model_selection import cross_val_score, KFold
from sklearn.datasets import load_iris
from sklearn.ensemble import RandomForestClassifier

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 模型
rf = RandomForestClassifier(random_state=42)

# K折交叉验证
scores = cross_val_score(rf, X, y, cv=5, scoring='accuracy')
print(f"交叉验证分数: {scores}")
print(f"平均分数: {scores.mean():.4f}")
print(f"标准差: {scores.std():.4f}")

# 自定义K折
kfold = KFold(n_splits=5, shuffle=True, random_state=42)
scores_custom = cross_val_score(rf, X, y, cv=kfold, scoring='accuracy')
print(f"\n自定义K折分数: {scores_custom}")
```

### 7.2 网格搜索

```python
"""
网格搜索调参
@author erik.zhou
"""
from sklearn.model_selection import GridSearchCV
from sklearn.datasets import load_iris
from sklearn.svm import SVC

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 参数网格
param_grid = {
    'C': [0.1, 1, 10],
    'gamma': ['scale', 'auto', 0.001, 0.01],
    'kernel': ['rbf', 'linear']
}

# 网格搜索
svm = SVC(random_state=42)
grid_search = GridSearchCV(
    svm, param_grid, cv=5, scoring='accuracy', n_jobs=-1
)
grid_search.fit(X, y)

# 最佳参数
print(f"最佳参数: {grid_search.best_params_}")
print(f"最佳分数: {grid_search.best_score_:.4f}")

# 最佳模型
best_model = grid_search.best_estimator_
```

### 7.3 随机搜索

```python
"""
随机搜索调参
@author erik.zhou
"""
from sklearn.model_selection import RandomizedSearchCV
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris
from scipy.stats import randint

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 参数分布
param_dist = {
    'n_estimators': randint(10, 200),
    'max_depth': randint(1, 20),
    'min_samples_split': randint(2, 20),
    'min_samples_leaf': randint(1, 10)
}

# 随机搜索
rf = RandomForestClassifier(random_state=42)
random_search = RandomizedSearchCV(
    rf, param_dist, n_iter=20, cv=5, 
    scoring='accuracy', random_state=42, n_jobs=-1
)
random_search.fit(X, y)

# 最佳参数
print(f"最佳参数: {random_search.best_params_}")
print(f"最佳分数: {random_search.best_score_:.4f}")
```

---

## 8. 管道Pipeline

### 8.1 基础管道

```python
"""
基础管道
@author erik.zhou
"""
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 创建管道
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', LogisticRegression(random_state=42))
])

# 训练
pipeline.fit(X_train, y_train)

# 预测
score = pipeline.score(X_test, y_test)
print(f"准确率: {score:.4f}")
```

### 8.2 管道与网格搜索

```python
"""
管道与网格搜索结合
@author erik.zhou
"""
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.svm import SVC
from sklearn.model_selection import GridSearchCV
from sklearn.datasets import load_iris

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target

# 创建管道
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(random_state=42))
])

# 参数网格（注意使用步骤名称前缀）
param_grid = {
    'svm__C': [0.1, 1, 10],
    'svm__gamma': ['scale', 'auto'],
    'svm__kernel': ['rbf', 'linear']
}

# 网格搜索
grid_search = GridSearchCV(pipeline, param_grid, cv=5, n_jobs=-1)
grid_search.fit(X, y)

print(f"最佳参数: {grid_search.best_params_}")
print(f"最佳分数: {grid_search.best_score_:.4f}")
```

---

## 9. 模型持久化

### 9.1 使用joblib保存和加载

```python
"""
模型持久化
@author erik.zhou
"""
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris
import joblib

# 训练模型
iris = load_iris()
X, y = iris.data, iris.target
rf = RandomForestClassifier(random_state=42)
rf.fit(X, y)

# 保存模型
joblib.dump(rf, 'random_forest_model.pkl')
print("模型已保存")

# 加载模型
loaded_model = joblib.load('random_forest_model.pkl')
print("模型已加载")

# 使用加载的模型预测
predictions = loaded_model.predict(X[:5])
print(f"预测结果: {predictions}")
```

---

## 10. 最佳实践

### 10.1 完整的机器学习流程

```python
"""
完整的机器学习流程示例
@author erik.zhou
"""
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, confusion_matrix
from sklearn.pipeline import Pipeline
import joblib

# 1. 加载数据
cancer = load_breast_cancer()
X, y = cancer.data, cancer.target

# 2. 划分数据集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# 3. 创建管道
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', RandomForestClassifier(random_state=42))
])

# 4. 参数调优
param_grid = {
    'classifier__n_estimators': [50, 100, 200],
    'classifier__max_depth': [None, 10, 20],
    'classifier__min_samples_split': [2, 5, 10]
}

grid_search = GridSearchCV(
    pipeline, param_grid, cv=5, 
    scoring='f1', n_jobs=-1, verbose=1
)

# 5. 训练
grid_search.fit(X_train, y_train)

# 6. 评估
best_model = grid_search.best_estimator_
y_pred = best_model.predict(X_test)

print(f"最佳参数: {grid_search.best_params_}")
print(f"\n分类报告:")
print(classification_report(y_test, y_pred, target_names=cancer.target_names))
print(f"\n混淆矩阵:")
print(confusion_matrix(y_test, y_pred))

# 7. 保存模型
joblib.dump(best_model, 'best_cancer_model.pkl')
print("\n模型已保存")
```

---

## 📝 学习检查清单

- [ ] 理解Scikit-learn的API设计理念
- [ ] 掌握数据预处理方法
- [ ] 能够使用各种分类算法
- [ ] 能够使用各种回归算法
- [ ] 掌握聚类和降维技术
- [ ] 理解模型评估指标
- [ ] 能够进行交叉验证和参数调优
- [ ] 掌握Pipeline的使用
- [ ] 能够保存和加载模型

## 🔗 相关资源

- [Scikit-learn官方文档](https://scikit-learn.org/stable/)
- [Scikit-learn用户指南](https://scikit-learn.org/stable/user_guide.html)
- [Scikit-learn示例](https://scikit-learn.org/stable/auto_examples/index.html)
- [Hands-On Machine Learning](https://www.oreilly.com/library/view/hands-on-machine-learning/9781492032632/)

---

**@author erik.zhou**  
**最后更新**: 2025-02-12
