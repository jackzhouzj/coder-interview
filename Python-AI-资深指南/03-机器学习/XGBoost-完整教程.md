# XGBoost完整教程

## 📋 教程概述

XGBoost（eXtreme Gradient Boosting）是一个高效的梯度提升框架，在Kaggle竞赛和工业界广泛应用。本教程系统讲解XGBoost的原理、使用方法和调优技巧。

## 🎯 学习目标

- 理解XGBoost的核心原理和优势
- 掌握XGBoost的基本使用方法
- 学会XGBoost的参数调优
- 了解XGBoost的高级特性
- 能够在实际项目中应用XGBoost

## 📚 目录

1. [XGBoost简介](#1-xgboost简介)
2. [基础使用](#2-基础使用)
3. [参数详解](#3-参数详解)
4. [模型训练与预测](#4-模型训练与预测)
5. [特征重要性](#5-特征重要性)
6. [模型调优](#6-模型调优)
7. [高级特性](#7-高级特性)
8. [实战案例](#8-实战案例)

---

## 1. XGBoost简介

### 1.1 什么是XGBoost

XGBoost是一个优化的分布式梯度提升库，具有以下特点：
- 高效的并行计算
- 正则化防止过拟合
- 支持自定义损失函数
- 内置交叉验证
- 缺失值自动处理

### 1.2 XGBoost的优势

```python
"""
XGBoost vs 传统GBDT对比
@author erik.zhou
"""
import xgboost as xgb
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
import time

# 加载数据
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# 1. 传统GBDT
start_time = time.time()
gb = GradientBoostingClassifier(n_estimators=100, random_state=42)
gb.fit(X_train, y_train)
gb_time = time.time() - start_time
gb_score = gb.score(X_test, y_test)

print(f"传统GBDT - 训练时间: {gb_time:.2f}秒, 准确率: {gb_score:.4f}")

# 2. XGBoost
start_time = time.time()
xgb_model = xgb.XGBClassifier(n_estimators=100, random_state=42)
xgb_model.fit(X_train, y_train)
xgb_time = time.time() - start_time
xgb_score = xgb_model.score(X_test, y_test)

print(f"XGBoost - 训练时间: {xgb_time:.2f}秒, 准确率: {xgb_score:.4f}")
print(f"速度提升: {gb_time / xgb_time:.2f}倍")
```

---

## 2. 基础使用

### 2.1 分类任务

```python
"""
XGBoost分类任务
@author erik.zhou
"""
import xgboost as xgb
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

# 加载数据
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# 方法1：使用Scikit-learn API
model = xgb.XGBClassifier(
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
dtrain = xgb.DMatrix(X_train, label=y_train)
dtest = xgb.DMatrix(X_test, label=y_test)

params = {
    'objective': 'binary:logistic',
    'max_depth': 5,
    'learning_rate': 0.1,
    'eval_metric': 'logloss'
}

# 训练模型
bst = xgb.train(
    params,
    dtrain,
    num_boost_round=100,
    evals=[(dtrain, 'train'), (dtest, 'test')],
    early_stopping_rounds=10,
    verbose_eval=False
)

# 预测
y_pred_proba = bst.predict(dtest)
y_pred = (y_pred_proba > 0.5).astype(int)
print(f"\n原生API准确率: {accuracy_score(y_test, y_pred):.4f}")
```

### 2.2 回归任务

```python
"""
XGBoost回归任务
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
model = xgb.XGBRegressor(
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
XGBoost多分类任务
@author erik.zhou
"""
from sklearn.datasets import load_iris

# 加载数据
data = load_iris()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# 多分类模型
model = xgb.XGBClassifier(
    n_estimators=100,
    max_depth=5,
    learning_rate=0.1,
    objective='multi:softmax',
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

### 3.1 通用参数

```python
"""
XGBoost通用参数
@author erik.zhou
"""

# 通用参数
params_general = {
    'booster': 'gbtree',  # 提升器类型：gbtree, gblinear, dart
    'verbosity': 1,  # 日志级别：0(silent), 1(warning), 2(info), 3(debug)
    'nthread': -1,  # 并行线程数，-1表示使用所有CPU核心
    'disable_default_eval_metric': False,  # 是否禁用默认评估指标
}

# 示例
model = xgb.XGBClassifier(
    booster='gbtree',
    verbosity=1,
    n_jobs=-1,
    random_state=42
)
```

### 3.2 树参数

```python
"""
XGBoost树参数
@author erik.zhou
"""

# 树参数
params_tree = {
    'max_depth': 6,  # 树的最大深度，默认6
    'min_child_weight': 1,  # 子节点最小权重和，默认1
    'gamma': 0,  # 节点分裂所需的最小损失减少，默认0
    'subsample': 1.0,  # 训练样本采样比例，默认1.0
    'colsample_bytree': 1.0,  # 每棵树列采样比例，默认1.0
    'colsample_bylevel': 1.0,  # 每层列采样比例，默认1.0
    'colsample_bynode': 1.0,  # 每个节点列采样比例，默认1.0
    'lambda': 1,  # L2正则化项，默认1
    'alpha': 0,  # L1正则化项，默认0
    'tree_method': 'auto',  # 树构建算法：auto, exact, approx, hist
    'max_leaves': 0,  # 最大叶子节点数，0表示无限制
}

# 示例：防止过拟合的参数设置
model = xgb.XGBClassifier(
    max_depth=5,  # 限制树深度
    min_child_weight=3,  # 增加最小子节点权重
    gamma=0.1,  # 增加分裂阈值
    subsample=0.8,  # 行采样
    colsample_bytree=0.8,  # 列采样
    reg_lambda=1.0,  # L2正则化
    reg_alpha=0.1,  # L1正则化
    random_state=42
)
```

### 3.3 学习任务参数

```python
"""
XGBoost学习任务参数
@author erik.zhou
"""

# 学习任务参数
params_learning = {
    'objective': 'binary:logistic',  # 学习目标
    'eval_metric': 'logloss',  # 评估指标
    'seed': 42,  # 随机种子
}

# 常用objective
objectives = {
    # 回归
    'reg:squarederror': '平方损失回归',
    'reg:squaredlogerror': '平方对数损失回归',
    'reg:logistic': '逻辑回归',
    'reg:pseudohubererror': 'Pseudo-Huber损失',
    
    # 分类
    'binary:logistic': '二分类逻辑回归',
    'binary:hinge': '二分类hinge损失',
    'multi:softmax': '多分类softmax',
    'multi:softprob': '多分类softmax概率',
    
    # 排序
    'rank:pairwise': '成对排序',
    'rank:ndcg': 'NDCG排序',
    'rank:map': 'MAP排序',
}

# 常用eval_metric
eval_metrics = {
    # 回归
    'rmse': '均方根误差',
    'rmsle': '均方根对数误差',
    'mae': '平均绝对误差',
    'mape': '平均绝对百分比误差',
    
    # 分类
    'logloss': '对数损失',
    'error': '分类错误率',
    'auc': 'AUC',
    'aucpr': 'PR曲线下面积',
    'merror': '多分类错误率',
    'mlogloss': '多分类对数损失',
}
```

---

## 4. 模型训练与预测

### 4.1 早停机制

```python
"""
XGBoost早停机制
@author erik.zhou
"""
import xgboost as xgb

# 准备数据
dtrain = xgb.DMatrix(X_train, label=y_train)
dtest = xgb.DMatrix(X_test, label=y_test)

params = {
    'objective': 'binary:logistic',
    'max_depth': 5,
    'learning_rate': 0.1,
    'eval_metric': 'logloss'
}

# 使用早停
evals = [(dtrain, 'train'), (dtest, 'test')]
model = xgb.train(
    params,
    dtrain,
    num_boost_round=1000,
    evals=evals,
    early_stopping_rounds=50,  # 50轮无提升则停止
    verbose_eval=10  # 每10轮输出一次
)

print(f"最佳迭代轮数: {model.best_iteration}")
print(f"最佳分数: {model.best_score}")
```

### 4.2 交叉验证

```python
"""
XGBoost交叉验证
@author erik.zhou
"""

# 准备数据
dtrain = xgb.DMatrix(X_train, label=y_train)

params = {
    'objective': 'binary:logistic',
    'max_depth': 5,
    'learning_rate': 0.1,
    'eval_metric': 'logloss'
}

# 交叉验证
cv_results = xgb.cv(
    params,
    dtrain,
    num_boost_round=1000,
    nfold=5,
    metrics='logloss',
    early_stopping_rounds=50,
    seed=42,
    verbose_eval=10
)

print(f"\n最佳迭代轮数: {len(cv_results)}")
print(f"最佳测试分数: {cv_results['test-logloss-mean'].min():.4f}")
print(f"最佳训练分数: {cv_results['train-logloss-mean'].min():.4f}")
```

### 4.3 增量训练

```python
"""
XGBoost增量训练
@author erik.zhou
"""

# 第一阶段训练
model = xgb.XGBClassifier(n_estimators=50, random_state=42)
model.fit(X_train, y_train)

# 第二阶段增量训练
model.n_estimators = 100  # 增加到100棵树
model.fit(X_train, y_train, xgb_model=model.get_booster())

print(f"增量训练后准确率: {model.score(X_test, y_test):.4f}")
```

---

## 5. 特征重要性

### 5.1 特征重要性分析

```python
"""
XGBoost特征重要性分析
@author erik.zhou
"""
import matplotlib.pyplot as plt
import pandas as pd

# 训练模型
model = xgb.XGBClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# 1. 获取特征重要性（默认：weight）
importance_weight = model.feature_importances_

# 2. 获取不同类型的特征重要性
importance_gain = model.get_booster().get_score(importance_type='gain')
importance_cover = model.get_booster().get_score(importance_type='cover')

# 创建DataFrame
feature_importance = pd.DataFrame({
    'feature': data.feature_names,
    'weight': importance_weight,
}).sort_values('weight', ascending=False)

print("特征重要性Top 10:")
print(feature_importance.head(10))

# 3. 可视化特征重要性
xgb.plot_importance(model, max_num_features=10, importance_type='weight')
plt.title('特征重要性（Weight）')
plt.show()

xgb.plot_importance(model, max_num_features=10, importance_type='gain')
plt.title('特征重要性（Gain）')
plt.show()
```

### 5.2 SHAP值分析

```python
"""
使用SHAP分析XGBoost特征重要性
@author erik.zhou
"""
import shap

# 训练模型
model = xgb.XGBClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)

# 创建SHAP解释器
explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)

# 1. 特征重要性汇总图
shap.summary_plot(shap_values, X_test, feature_names=data.feature_names)

# 2. 单个样本解释
shap.force_plot(
    explainer.expected_value,
    shap_values[0],
    X_test[0],
    feature_names=data.feature_names
)

# 3. 依赖图
shap.dependence_plot(0, shap_values, X_test, feature_names=data.feature_names)
```

---

## 6. 模型调优

### 6.1 网格搜索调优

```python
"""
XGBoost网格搜索调优
@author erik.zhou
"""
from sklearn.model_selection import GridSearchCV

# 定义参数网格
param_grid = {
    'max_depth': [3, 5, 7],
    'learning_rate': [0.01, 0.1, 0.3],
    'n_estimators': [50, 100, 200],
    'min_child_weight': [1, 3, 5],
    'gamma': [0, 0.1, 0.2],
    'subsample': [0.8, 0.9, 1.0],
    'colsample_bytree': [0.8, 0.9, 1.0]
}

# 创建网格搜索
grid_search = GridSearchCV(
    estimator=xgb.XGBClassifier(random_state=42),
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
XGBoost分步调优策略
@author erik.zhou
"""
from sklearn.model_selection import GridSearchCV

# 步骤1：调整树的数量和学习率
param_grid_1 = {
    'n_estimators': [50, 100, 150, 200],
    'learning_rate': [0.01, 0.05, 0.1, 0.2]
}

grid_1 = GridSearchCV(
    xgb.XGBClassifier(random_state=42),
    param_grid_1,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)
grid_1.fit(X_train, y_train)
best_params_1 = grid_1.best_params_

print(f"步骤1最佳参数: {best_params_1}")

# 步骤2：调整树的深度和子节点权重
param_grid_2 = {
    'max_depth': [3, 5, 7, 9],
    'min_child_weight': [1, 3, 5, 7]
}

grid_2 = GridSearchCV(
    xgb.XGBClassifier(**best_params_1, random_state=42),
    param_grid_2,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)
grid_2.fit(X_train, y_train)
best_params_2 = {**best_params_1, **grid_2.best_params_}

print(f"步骤2最佳参数: {best_params_2}")

# 步骤3：调整gamma
param_grid_3 = {
    'gamma': [0, 0.05, 0.1, 0.2, 0.3]
}

grid_3 = GridSearchCV(
    xgb.XGBClassifier(**best_params_2, random_state=42),
    param_grid_3,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)
grid_3.fit(X_train, y_train)
best_params_3 = {**best_params_2, **grid_3.best_params_}

print(f"步骤3最佳参数: {best_params_3}")

# 步骤4：调整采样参数
param_grid_4 = {
    'subsample': [0.6, 0.7, 0.8, 0.9, 1.0],
    'colsample_bytree': [0.6, 0.7, 0.8, 0.9, 1.0]
}

grid_4 = GridSearchCV(
    xgb.XGBClassifier(**best_params_3, random_state=42),
    param_grid_4,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)
grid_4.fit(X_train, y_train)
best_params_4 = {**best_params_3, **grid_4.best_params_}

print(f"步骤4最佳参数: {best_params_4}")

# 步骤5：调整正则化参数
param_grid_5 = {
    'reg_alpha': [0, 0.01, 0.1, 1],
    'reg_lambda': [0.1, 1, 10, 100]
}

grid_5 = GridSearchCV(
    xgb.XGBClassifier(**best_params_4, random_state=42),
    param_grid_5,
    cv=5,
    scoring='accuracy',
    n_jobs=-1
)
grid_5.fit(X_train, y_train)
best_params_final = {**best_params_4, **grid_5.best_params_}

print(f"\n最终最佳参数: {best_params_final}")

# 使用最佳参数训练最终模型
final_model = xgb.XGBClassifier(**best_params_final, random_state=42)
final_model.fit(X_train, y_train)
print(f"最终测试集分数: {final_model.score(X_test, y_test):.4f}")
```

---

## 7. 高级特性

### 7.1 自定义损失函数

```python
"""
XGBoost自定义损失函数
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

# 使用自定义损失函数
dtrain = xgb.DMatrix(X_train, label=y_train)
dtest = xgb.DMatrix(X_test, label=y_test)

params = {
    'max_depth': 5,
    'learning_rate': 0.1,
}

model = xgb.train(
    params,
    dtrain,
    num_boost_round=100,
    obj=custom_loss,
    evals=[(dtest, 'test')]
)
```

### 7.2 处理不平衡数据

```python
"""
XGBoost处理不平衡数据
@author erik.zhou
"""

# 方法1：调整scale_pos_weight参数
# scale_pos_weight = 负样本数 / 正样本数
neg_count = np.sum(y_train == 0)
pos_count = np.sum(y_train == 1)
scale_pos_weight = neg_count / pos_count

model = xgb.XGBClassifier(
    scale_pos_weight=scale_pos_weight,
    random_state=42
)
model.fit(X_train, y_train)

# 方法2：使用样本权重
sample_weights = np.where(y_train == 1, 2.0, 1.0)
model = xgb.XGBClassifier(random_state=42)
model.fit(X_train, y_train, sample_weight=sample_weights)

# 方法3：使用SMOTE过采样
from imblearn.over_sampling import SMOTE

smote = SMOTE(random_state=42)
X_train_resampled, y_train_resampled = smote.fit_resample(X_train, y_train)

model = xgb.XGBClassifier(random_state=42)
model.fit(X_train_resampled, y_train_resampled)
```

### 7.3 缺失值处理

```python
"""
XGBoost缺失值处理
@author erik.zhou
"""
import numpy as np

# XGBoost自动处理缺失值
X_train_missing = X_train.copy()
X_test_missing = X_test.copy()

# 随机设置一些值为NaN
mask = np.random.random(X_train_missing.shape) < 0.1
X_train_missing[mask] = np.nan

mask = np.random.random(X_test_missing.shape) < 0.1
X_test_missing[mask] = np.nan

# XGBoost会自动学习缺失值的最佳分裂方向
model = xgb.XGBClassifier(random_state=42)
model.fit(X_train_missing, y_train)

print(f"含缺失值数据准确率: {model.score(X_test_missing, y_test):.4f}")
```

---

## 8. 实战案例

### 8.1 完整的XGBoost项目流程

```python
"""
XGBoost完整项目实战
@author erik.zhou
"""
import xgboost as xgb
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import classification_report, confusion_matrix
import pandas as pd

class XGBoostPipeline:
    """XGBoost完整流水线"""
    
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
            
            self.model = xgb.XGBClassifier(**self.params, random_state=42)
            self.model.fit(
                X_train_split, y_train_split,
                eval_set=[(X_val, y_val)],
                early_stopping_rounds=50,
                verbose=False
            )
        else:
            self.model = xgb.XGBClassifier(**self.params, random_state=42)
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

# 使用示例
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# 定义参数
params = {
    'n_estimators': 100,
    'max_depth': 5,
    'learning_rate': 0.1,
    'subsample': 0.8,
    'colsample_bytree': 0.8
}

# 创建流水线
pipeline = XGBoostPipeline(params)

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
```

---

## 📚 相关资源

- [XGBoost官方文档](https://xgboost.readthedocs.io/)
- [XGBoost参数调优指南](https://xgboost.readthedocs.io/en/latest/parameter.html)
- [XGBoost论文](https://arxiv.org/abs/1603.02754)
- [Kaggle XGBoost教程](https://www.kaggle.com/learn/intro-to-machine-learning)

---

**@author erik.zhou**
