# LightGBM完整教程

## 📋 教程概述

LightGBM（Light Gradient Boosting Machine）是微软开发的高效梯度提升框架，以速度快、内存占用低著称。本教程系统讲解LightGBM的原理、使用方法和优化技巧。

## 🎯 学习目标

- 理解LightGBM的核心原理和优势
- 掌握LightGBM的基本使用方法
- 学会LightGBM的参数调优
- 了解LightGBM与XGBoost的区别
- 能够在实际项目中应用LightGBM

## 📚 目录

1. [LightGBM简介](#1-lightgbm简介)
2. [基础使用](#2-基础使用)
3. [参数详解](#3-参数详解)
4. [模型训练与预测](#4-模型训练与预测)
5. [特征重要性](#5-特征重要性)
6. [模型调优](#6-模型调优)
7. [高级特性](#7-高级特性)
8. [实战案例](#8-实战案例)

---

## 1. LightGBM简介

### 1.1 什么是LightGBM

LightGBM是一个基于决策树的梯度提升框架，具有以下特点：
- 更快的训练速度
- 更低的内存消耗
- 更好的准确率
- 支持并行和GPU学习
- 能够处理大规模数据

### 1.2 LightGBM vs XGBoost

```python
"""
LightGBM vs XGBoost性能对比
@author erik.zhou
"""
import lightgbm as lgb
import xgboost as xgb
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
import time

# 加载数据
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# 1. XGBoost
start_time = time.time()
xgb_model = xgb.XGBClassifier(n_estimators=100, random_state=42)
xgb_model.fit(X_train, y_train)
xgb_time = time.time() - start_time
xgb_score = xgb_model.score(X_test, y_test)

print(f"XGBoost - 训练时间: {xgb_time:.2f}秒, 准确率: {xgb_score:.4f}")

# 2. LightGBM
start_time = time.time()
lgb_model = lgb.LGBMClassifier(n_estimators=100, random_state=42)
lgb_model.fit(X_train, y_train)
lgb_time = time.time() - start_time
lgb_score = lgb_model.score(X_test, y_test)

print(f"LightGBM - 训练时间: {lgb_time:.2f}秒, 准确率: {lgb_score:.4f}")
print(f"速度提升: {xgb_time / lgb_time:.2f}倍")
```

### 1.3 LightGBM的核心技术

- Histogram-based算法：将连续特征离散化为直方图
- Leaf-wise生长策略：按叶子节点增益最大化生长
- GOSS（Gradient-based One-Side Sampling）：基于梯度的单边采样
- EFB（Exclusive Feature Bundling）：互斥特征捆绑

---

## 2. 基础使用

### 2.1 分类任务

```python
"""
LightGBM分类任务
@author erik.zhou
"""
import lightgbm as lgb
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

# 加载数据
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# 方法1：使用Scikit-learn API
model = lgb.LGBMClassifier(
    n_estimators=100,
    max_depth=5,
    learning_rate=0.1,
    random_state=42
)
model.fit(X_train, y_train)

# 预测
y_pred = model.predict(X_test)
print(f"准确率: {accuracy_score(y_test, y_pred):.4f}")
print("\n分类报告:")
print(classification_report(y_test, y_pred))

# 方法2：使用原生API
train_data = lgb.Dataset(X_train, label=y_train)
test_data = lgb.Dataset(X_test, label=y_test, reference=train_data)

params = {
    'objective': 'binary',
    'metric': 'binary_logloss',
    'max_depth': 5,
    'learning_rate': 0.1,
    'verbose': -1
}

# 训练模型
bst = lgb.train(
    params,
    train_data,
    num_boost_round=100,
    valid_sets=[train_data, test_data],
    valid_names=['train', 'test'],
    callbacks=[lgb.early_stopping(stopping_rounds=10)]
)

# 预测
y_pred_proba = bst.predict(X_test)
y_pred = (y_pred_proba > 0.5).astype(int)
print(f"\n原生API准确率: {accuracy_score(y_test, y_pred):.4f}")
```

### 2.2 回归任务

```python
"""
LightGBM回归任务
@author erik.zhou
"""
from sklearn.datasets import fetch_california_housing
from sklearn.metrics import mean_squared_error, r2_score
import numpy as np

# 加载数据
data = fetch_california_housing()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# 使用Scikit-learn API
model = lgb.LGBMRegressor(
    n_estimators=100,
    max_depth=5,
    learning_rate=0.1,
    random_state=42
)
model.fit(X_train, y_train)

# 预测
y_pred = model.predict(X_test)

# 评估
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
r2 = r2_score(y_test, y_pred)

print(f"均方误差 (MSE): {mse:.4f}")
print(f"均方根误差 (RMSE): {rmse:.4f}")
print(f"R²分数: {r2:.4f}")
```

### 2.3 多分类任务

```python
"""
LightGBM多分类任务
@author erik.zhou
"""
from sklearn.datasets import load_iris

# 加载数据
data = load_iris()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# 多分类模型
model = lgb.LGBMClassifier(
    n_estimators=100,
    max_depth=5,
    learning_rate=0.1,
    objective='multiclass',
    num_class=3,
    random_state=42
)
model.fit(X_train, y_train)

# 预测
y_pred = model.predict(X_test)
print(f"准确率: {accuracy_score(y_test, y_pred):.4f}")

# 预测概率
y_pred_proba = model.predict_proba(X_test)
print(f"\n预测概率示例:\n{y_pred_proba[:5]}")
```

---

## 3. 参数详解

### 3.1 核心参数

```python
"""
LightGBM核心参数
@author erik.zhou
"""

# 核心参数
params_core = {
    'boosting_type': 'gbdt',  # 提升类型：gbdt, rf, dart, goss
    'objective': 'binary',  # 学习目标
    'metric': 'binary_logloss',  # 评估指标
    'num_leaves': 31,  # 叶子节点数，默认31
    'max_depth': -1,  # 树的最大深度，-1表示无限制
    'learning_rate': 0.1,  # 学习率
    'n_estimators': 100,  # 树的数量
}

# 示例
model = lgb.LGBMClassifier(
    boosting_type='gbdt',
    objective='binary',
    num_leaves=31,
    max_depth=-1,
    learning_rate=0.1,
    n_estimators=100,
    random_state=42
)
```

### 3.2 学习控制参数

```python
"""
LightGBM学习控制参数
@author erik.zhou
"""

# 学习控制参数
params_learning = {
    'max_depth': -1,  # 树的最大深度
    'num_leaves': 31,  # 叶子节点数
    'min_data_in_leaf': 20,  # 叶子节点最小样本数
    'min_sum_hessian_in_leaf': 1e-3,  # 叶子节点最小Hessian和
    'bagging_fraction': 1.0,  # 数据采样比例
    'bagging_freq': 0,  # 采样频率
    'feature_fraction': 1.0,  # 特征采样比例
    'lambda_l1': 0.0,  # L1正则化
    'lambda_l2': 0.0,  # L2正则化
    'min_gain_to_split': 0.0,  # 分裂最小增益
}

# 示例：防止过拟合的参数设置
model = lgb.LGBMClassifier(
    num_leaves=20,  # 减少叶子节点数
    min_data_in_leaf=30,  # 增加叶子节点最小样本数
    bagging_fraction=0.8,  # 数据采样
    bagging_freq=5,  # 每5轮采样一次
    feature_fraction=0.8,  # 特征采样
    lambda_l1=0.1,  # L1正则化
    lambda_l2=0.1,  # L2正则化
    random_state=42
)
```

### 3.3 IO参数

```python
"""
LightGBM IO参数
@author erik.zhou
"""

# IO参数
params_io = {
    'max_bin': 255,  # 最大bin数
    'min_data_in_bin': 3,  # bin中最小样本数
    'categorical_feature': 'auto',  # 类别特征
    'ignore_column': '',  # 忽略的列
    'verbose': -1,  # 日志级别：<0(Fatal), =0(Error), =1(Info), >1(Debug)
}
```

---

## 4. 模型训练与预测

### 4.1 早停机制

```python
"""
LightGBM早停机制
@author erik.zhou
"""
import lightgbm as lgb

# 准备数据
train_data = lgb.Dataset(X_train, label=y_train)
test_data = lgb.Dataset(X_test, label=y_test, reference=train_data)

params = {
    'objective': 'binary',
    'metric': 'binary_logloss',
    'max_depth': 5,
    'learning_rate': 0.1,
    'verbose': -1
}

# 使用早停
callbacks = [
    lgb.early_stopping(stopping_rounds=50),
    lgb.log_evaluation(period=10)
]

model = lgb.train(
    params,
    train_data,
    num_boost_round=1000,
    valid_sets=[train_data, test_data],
    valid_names=['train', 'test'],
    callbacks=callbacks
)

print(f"最佳迭代轮数: {model.best_iteration}")
print(f"最佳分数: {model.best_score}")
```

### 4.2 交叉验证

```python
"""
LightGBM交叉验证
@author erik.zhou
"""

# 准备数据
train_data = lgb.Dataset(X_train, label=y_train)

params = {
    'objective': 'binary',
    'metric': 'binary_logloss',
    'max_depth': 5,
    'learning_rate': 0.1,
    'verbose': -1
}

# 交叉验证
cv_results = lgb.cv(
    params,
    train_data,
    num_boost_round=1000,
    nfold=5,
    callbacks=[
        lgb.early_stopping(stopping_rounds=50),
        lgb.log_evaluation(period=10)
    ],
    return_cvbooster=True
)

print(f"\n最佳迭代轮数: {len(cv_results['valid binary_logloss-mean'])}")
print(f"最佳测试分数: {min(cv_results['valid binary_logloss-mean']):.4f}")
```

### 4.3 增量训练

```python
"""
LightGBM增量训练
@author erik.zhou
"""

# 第一阶段训练
model = lgb.LGBMClassifier(n_estimators=50, random_state=42)
model.fit(X_train, y_train)

# 第二阶段增量训练
model.n_estimators = 100  # 增加到100棵树
model.fit(X_train, y_train, init_model=model.booster_)

print(f"增量训练后准确率: {model.score(X_test, y_test):.4f}")
```

---

## 5. 特征重要性

### 5.1 特征重要性分析

```python
"""
LightGBM特征重要性分析
@author erik.zhou
"""
import matplotlib.pyplot as plt
import pandas as pd

# 训练模型
model = lgb.LGBMClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# 1. 获取特征重要性（默认：split）
importance_split = model.feature_importances_

# 2. 获取不同类型的特征重要性
importance_gain = model.booster_.feature_importance(importance_type='gain')

# 创建DataFrame
feature_importance = pd.DataFrame({
    'feature': data.feature_names,
    'split': importance_split,
    'gain': importance_gain
}).sort_values('gain', ascending=False)

print("特征重要性Top 10:")
print(feature_importance.head(10))

# 3. 可视化特征重要性
lgb.plot_importance(model, max_num_features=10, importance_type='split')
plt.title('特征重要性（Split）')
plt.show()

lgb.plot_importance(model, max_num_features=10, importance_type='gain')
plt.title('特征重要性（Gain）')
plt.show()
```

---

## 6. 模型调优

### 6.1 网格搜索调优

```python
"""
LightGBM网格搜索调优
@author erik.zhou
"""
from sklearn.model_selection import GridSearchCV

# 定义参数网格
param_grid = {
    'num_leaves': [15, 31, 63],
    'max_depth': [3, 5, 7, -1],
    'learning_rate': [0.01, 0.1, 0.3],
    'n_estimators': [50, 100, 200],
    'min_child_samples': [10, 20, 30],
    'subsample': [0.8, 0.9, 1.0],
    'colsample_bytree': [0.8, 0.9, 1.0]
}

# 创建网格搜索
grid_search = GridSearchCV(
    estimator=lgb.LGBMClassifier(random_state=42),
    param_grid=param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,
    verbose=1
)

# 执行搜索
grid_search.fit(X_train, y_train)

print(f"最佳参数: {grid_search.best_params_}")
print(f"最佳交叉验证分数: {grid_search.best_score_:.4f}")
print(f"测试集分数: {grid_search.score(X_test, y_test):.4f}")
```

### 6.2 分步调优策略

```python
"""
LightGBM分步调优策略
@author erik.zhou
"""
from sklearn.model_selection import GridSearchCV

# 步骤1：调整num_leaves和max_depth
param_grid_1 = {
    'num_leaves': [15, 31, 63, 127],
    'max_depth': [3, 5, 7, 9, -1]
}

grid_1 = GridSearchCV(
    lgb.LGBMClassifier(random_state=42),
    param_grid_1,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)
grid_1.fit(X_train, y_train)
best_params_1 = grid_1.best_params_

print(f"步骤1最佳参数: {best_params_1}")

# 步骤2：调整min_child_samples和min_child_weight
param_grid_2 = {
    'min_child_samples': [10, 20, 30, 40],
    'min_child_weight': [0.001, 0.01, 0.1]
}

grid_2 = GridSearchCV(
    lgb.LGBMClassifier(**best_params_1, random_state=42),
    param_grid_2,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)
grid_2.fit(X_train, y_train)
best_params_2 = {**best_params_1, **grid_2.best_params_}

print(f"步骤2最佳参数: {best_params_2}")

# 步骤3：调整采样参数
param_grid_3 = {
    'subsample': [0.6, 0.7, 0.8, 0.9, 1.0],
    'colsample_bytree': [0.6, 0.7, 0.8, 0.9, 1.0],
    'bagging_freq': [0, 1, 5, 10]
}

grid_3 = GridSearchCV(
    lgb.LGBMClassifier(**best_params_2, random_state=42),
    param_grid_3,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)
grid_3.fit(X_train, y_train)
best_params_3 = {**best_params_2, **grid_3.best_params_}

print(f"步骤3最佳参数: {best_params_3}")

# 步骤4：调整正则化参数
param_grid_4 = {
    'reg_alpha': [0, 0.01, 0.1, 1],
    'reg_lambda': [0, 0.01, 0.1, 1]
}

grid_4 = GridSearchCV(
    lgb.LGBMClassifier(**best_params_3, random_state=42),
    param_grid_4,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)
grid_4.fit(X_train, y_train)
best_params_final = {**best_params_3, **grid_4.best_params_}

print(f"\n最终最佳参数: {best_params_final}")

# 使用最佳参数训练最终模型
final_model = lgb.LGBMClassifier(**best_params_final, random_state=42)
final_model.fit(X_train, y_train)
print(f"最终测试集分数: {final_model.score(X_test, y_test):.4f}")
```

---

## 7. 高级特性

### 7.1 类别特征处理

```python
"""
LightGBM类别特征处理
@author erik.zhou
"""
import pandas as pd
import numpy as np

# 创建包含类别特征的数据
df = pd.DataFrame({
    'num_feature1': np.random.randn(1000),
    'num_feature2': np.random.randn(1000),
    'cat_feature1': np.random.choice(['A', 'B', 'C'], 1000),
    'cat_feature2': np.random.choice(['X', 'Y', 'Z'], 1000),
    'target': np.random.randint(0, 2, 1000)
})

# 将类别特征转换为category类型
df['cat_feature1'] = df['cat_feature1'].astype('category')
df['cat_feature2'] = df['cat_feature2'].astype('category')

X = df.drop('target', axis=1)
y = df['target']

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# LightGBM自动处理类别特征
model = lgb.LGBMClassifier(random_state=42)
model.fit(
    X_train, y_train,
    categorical_feature=['cat_feature1', 'cat_feature2']
)

print(f"准确率: {model.score(X_test, y_test):.4f}")
```

### 7.2 处理不平衡数据

```python
"""
LightGBM处理不平衡数据
@author erik.zhou
"""

# 方法1：调整scale_pos_weight参数
neg_count = np.sum(y_train == 0)
pos_count = np.sum(y_train == 1)
scale_pos_weight = neg_count / pos_count

model = lgb.LGBMClassifier(
    scale_pos_weight=scale_pos_weight,
    random_state=42
)
model.fit(X_train, y_train)

# 方法2：使用样本权重
sample_weights = np.where(y_train == 1, 2.0, 1.0)
model = lgb.LGBMClassifier(random_state=42)
model.fit(X_train, y_train, sample_weight=sample_weights)

# 方法3：使用is_unbalance参数
model = lgb.LGBMClassifier(is_unbalance=True, random_state=42)
model.fit(X_train, y_train)
```

### 7.3 自定义损失函数

```python
"""
LightGBM自定义损失函数
@author erik.zhou
"""
import numpy as np

def custom_loss(y_true, y_pred):
    """
    自定义损失函数
    
    Args:
        y_true: 真实标签
        y_pred: 预测值
    
    Returns:
        grad: 梯度
        hess: 二阶导数
    """
    # 示例：自定义平方损失
    grad = 2 * (y_pred - y_true)
    hess = 2 * np.ones_like(y_true)
    return grad, hess

def custom_eval(y_true, y_pred):
    """
    自定义评估函数
    
    Args:
        y_true: 真实标签
        y_pred: 预测值
    
    Returns:
        name: 指标名称
        value: 指标值
        is_higher_better: 是否越大越好
    """
    # 示例：自定义准确率
    accuracy = np.mean((y_pred > 0.5) == y_true)
    return 'custom_accuracy', accuracy, True

# 使用自定义损失函数
train_data = lgb.Dataset(X_train, label=y_train)
test_data = lgb.Dataset(X_test, label=y_test, reference=train_data)

params = {
    'max_depth': 5,
    'learning_rate': 0.1,
    'verbose': -1
}

model = lgb.train(
    params,
    train_data,
    num_boost_round=100,
    valid_sets=[test_data],
    fobj=custom_loss,
    feval=custom_eval
)
```

### 7.4 GPU加速

```python
"""
LightGBM GPU加速
@author erik.zhou
"""

# 使用GPU训练（需要安装GPU版本的LightGBM）
model = lgb.LGBMClassifier(
    device='gpu',
    gpu_platform_id=0,
    gpu_device_id=0,
    random_state=42
)

# 或者在参数中指定
params = {
    'objective': 'binary',
    'metric': 'binary_logloss',
    'device': 'gpu',
    'gpu_platform_id': 0,
    'gpu_device_id': 0
}

train_data = lgb.Dataset(X_train, label=y_train)
model = lgb.train(params, train_data, num_boost_round=100)
```

---

## 8. 实战案例

### 8.1 完整的LightGBM项目流程

```python
"""
LightGBM完整项目实战
@author erik.zhou
"""
import lightgbm as lgb
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import classification_report, confusion_matrix
import pandas as pd

class LightGBMPipeline:
    """LightGBM完整流水线"""
    
    def __init__(self, params=None):
        self.params = params or {}
        self.model = None
        self.scaler = StandardScaler()
    
    def preprocess(self, X_train, X_test):
        """数据预处理"""
        X_train_scaled = self.scaler.fit_transform(X_train)
        X_test_scaled = self.scaler.transform(X_test)
        return X_train_scaled, X_test_scaled
    
    def train(self, X_train, y_train, use_early_stopping=True):
        """训练模型"""
        if use_early_stopping:
            X_train_split, X_val, y_train_split, y_val = train_test_split(
                X_train, y_train, test_size=0.2, random_state=42
            )
            
            self.model = lgb.LGBMClassifier(**self.params, random_state=42)
            self.model.fit(
                X_train_split, y_train_split,
                eval_set=[(X_val, y_val)],
                callbacks=[lgb.early_stopping(stopping_rounds=50)]
            )
        else:
            self.model = lgb.LGBMClassifier(**self.params, random_state=42)
            self.model.fit(X_train, y_train)
        
        return self.model
    
    def evaluate(self, X_test, y_test):
        """评估模型"""
        y_pred = self.model.predict(X_test)
        
        print("分类报告:")
        print(classification_report(y_test, y_pred))
        
        print("\n混淆矩阵:")
        print(confusion_matrix(y_test, y_pred))
        
        score = self.model.score(X_test, y_test)
        print(f"\n测试集准确率: {score:.4f}")
        
        return y_pred
    
    def get_feature_importance(self, feature_names):
        """获取特征重要性"""
        importance = pd.DataFrame({
            'feature': feature_names,
            'importance': self.model.feature_importances_
        }).sort_values('importance', ascending=False)
        
        return importance
    
    def save_model(self, filename):
        """保存模型"""
        self.model.booster_.save_model(filename)
        print(f"模型已保存到: {filename}")
    
    def load_model(self, filename):
        """加载模型"""
        self.model = lgb.Booster(model_file=filename)
        print(f"模型已从 {filename} 加载")

# 使用示例
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# 定义参数
params = {
    'n_estimators': 100,
    'num_leaves': 31,
    'max_depth': 5,
    'learning_rate': 0.1,
    'subsample': 0.8,
    'colsample_bytree': 0.8
}

# 创建流水线
pipeline = LightGBMPipeline(params)

# 数据预处理
X_train_scaled, X_test_scaled = pipeline.preprocess(X_train, X_test)

# 训练模型
pipeline.train(X_train_scaled, y_train, use_early_stopping=True)

# 评估模型
y_pred = pipeline.evaluate(X_test_scaled, y_test)

# 特征重要性
importance = pipeline.get_feature_importance(data.feature_names)
print("\n特征重要性Top 10:")
print(importance.head(10))

# 保存模型
pipeline.save_model('lightgbm_model.txt')
```

### 8.2 LightGBM vs XGBoost完整对比

```python
"""
LightGBM vs XGBoost完整对比
@author erik.zhou
"""
import lightgbm as lgb
import xgboost as xgb
from sklearn.model_selection import cross_val_score
import time

# 准备数据
data = load_breast_cancer()
X, y = data.data, data.target

# 定义相同的参数
params_lgb = {
    'n_estimators': 100,
    'max_depth': 5,
    'learning_rate': 0.1,
    'num_leaves': 31,
    'random_state': 42
}

params_xgb = {
    'n_estimators': 100,
    'max_depth': 5,
    'learning_rate': 0.1,
    'random_state': 42
}

# 1. 训练速度对比
print("=" * 50)
print("训练速度对比")
print("=" * 50)

start_time = time.time()
lgb_model = lgb.LGBMClassifier(**params_lgb)
lgb_model.fit(X, y)
lgb_time = time.time() - start_time

start_time = time.time()
xgb_model = xgb.XGBClassifier(**params_xgb)
xgb_model.fit(X, y)
xgb_time = time.time() - start_time

print(f"LightGBM训练时间: {lgb_time:.2f}秒")
print(f"XGBoost训练时间: {xgb_time:.2f}秒")
print(f"LightGBM速度提升: {xgb_time / lgb_time:.2f}倍")

# 2. 准确率对比
print("\n" + "=" * 50)
print("准确率对比（5折交叉验证）")
print("=" * 50)

lgb_scores = cross_val_score(lgb_model, X, y, cv=5, scoring='accuracy')
xgb_scores = cross_val_score(xgb_model, X, y, cv=5, scoring='accuracy')

print(f"LightGBM准确率: {lgb_scores.mean():.4f} (+/- {lgb_scores.std() * 2:.4f})")
print(f"XGBoost准确率: {xgb_scores.mean():.4f} (+/- {xgb_scores.std() * 2:.4f})")

# 3. 内存占用对比
import sys

lgb_size = sys.getsizeof(lgb_model)
xgb_size = sys.getsizeof(xgb_model)

print("\n" + "=" * 50)
print("内存占用对比")
print("=" * 50)
print(f"LightGBM模型大小: {lgb_size / 1024:.2f} KB")
print(f"XGBoost模型大小: {xgb_size / 1024:.2f} KB")
```

---

## 📚 相关资源

- [LightGBM官方文档](https://lightgbm.readthedocs.io/)
- [LightGBM参数调优指南](https://lightgbm.readthedocs.io/en/latest/Parameters-Tuning.html)
- [LightGBM论文](https://papers.nips.cc/paper/6907-lightgbm-a-highly-efficient-gradient-boosting-decision-tree)
- [LightGBM vs XGBoost对比](https://lightgbm.readthedocs.io/en/latest/Experiments.html)

---

**@author erik.zhou**
