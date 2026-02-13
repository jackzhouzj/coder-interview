# Keras完整教程

> 掌握Keras高级API，快速构建深度学习模型
> 
> @author erik.zhou

## 📚 教程概述

**版本信息**: Keras 3.0+ (tf.keras)  
**学习难度**: ⭐⭐⭐  
**重要程度**: ⭐⭐⭐⭐  
**预计学习时长**: 20-30小时

## 🎯 学习目标

- [ ] 理解Keras的设计理念
- [ ] 掌握Sequential和Functional API
- [ ] 能够使用预训练模型
- [ ] 掌握回调函数机制
- [ ] 理解自定义组件
- [ ] 掌握模型微调技术
- [ ] 能够进行迁移学习
- [ ] 了解Keras生态系统

## 📖 目录

1. [Keras简介](#1-keras简介)
2. [模型构建](#2-模型构建)
3. [层和激活函数](#3-层和激活函数)
4. [模型编译和训练](#4-模型编译和训练)
5. [回调函数](#5-回调函数)
6. [预训练模型](#6-预训练模型)
7. [迁移学习](#7-迁移学习)
8. [模型微调](#8-模型微调)
9. [自定义组件](#9-自定义组件)
10. [最佳实践](#10-最佳实践)

---

## 1. Keras简介

### 1.1 Keras概述

```python
"""
Keras简介
@author erik.zhou
"""
import tensorflow as tf
from tensorflow import keras

print(f"Keras版本: {keras.__version__}")
print(f"TensorFlow版本: {tf.__version__}")

print("""
Keras的特点：

1. 用户友好：
   - 简洁的API设计
   - 一致的接口
   - 清晰的错误信息
   - 易于学习和使用

2. 模块化：
   - 层、模型、优化器独立
   - 可组合的构建块
   - 易于扩展

3. 多后端支持：
   - TensorFlow（主要）
   - JAX
   - PyTorch（Keras 3.0+）

4. 生产就绪：
   - 工业级性能
   - 大规模部署
   - 完整的生态系统

Keras vs 纯TensorFlow：
- Keras：高级API，快速原型
- TensorFlow：底层控制，灵活性

使用建议：
- 快速开发：使用Keras
- 研究创新：结合使用
- 生产部署：都可以
""")
```

### 1.2 第一个Keras模型

```python
"""
Hello Keras
@author erik.zhou
"""
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np

# 准备数据
(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()
x_train = x_train.reshape(-1, 784).astype('float32') / 255
x_test = x_test.reshape(-1, 784).astype('float32') / 255

# 构建模型
model = keras.Sequential([
    layers.Dense(128, activation='relu', input_shape=(784,)),
    layers.Dropout(0.2),
    layers.Dense(10, activation='softmax')
])

# 编译模型
model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# 训练模型
history = model.fit(
    x_train, y_train,
    batch_size=128,
    epochs=5,
    validation_split=0.2,
    verbose=1
)

# 评估模型
test_loss, test_acc = model.evaluate(x_test, y_test, verbose=0)
print(f'\n测试准确率: {test_acc:.4f}')

# 预测
predictions = model.predict(x_test[:5])
print(f'\n预测结果: {np.argmax(predictions, axis=1)}')
print(f'真实标签: {y_test[:5]}')
```

---

## 2. 模型构建

### 2.1 Sequential模型

```python
"""
Sequential模型详解
@author erik.zhou
"""
from tensorflow import keras
from tensorflow.keras import layers

# 方式1：逐层添加
model = keras.Sequential()
model.add(layers.Dense(64, activation='relu'))
model.add(layers.Dense(10, activation='softmax'))

# 方式2：列表初始化
model = keras.Sequential([
    layers.Dense(64, activation='relu'),
    layers.Dense(10, activation='softmax')
])

# 方式3：使用name参数
model = keras.Sequential([
    layers.Dense(64, activation='relu', name='hidden'),
    layers.Dense(10, activation='softmax', name='output')
], name='my_model')

# 查看模型结构
model.build((None, 20))  # 指定输入形状
model.summary()

print("""
Sequential模型特点：
- 简单直观
- 线性堆叠
- 适合大多数场景

限制：
- 单输入单输出
- 无法共享层
- 无法有分支
""")
```

### 2.2 Functional API

```python
"""
Functional API详解
@author erik.zhou
"""
from tensorflow import keras
from tensorflow.keras import layers

# 基本用法
inputs = keras.Input(shape=(784,))
x = layers.Dense(64, activation='relu')(inputs)
x = layers.Dropout(0.5)(x)
outputs = layers.Dense(10, activation='softmax')(x)

model = keras.Model(inputs=inputs, outputs=outputs)

# 多输入示例
text_input = keras.Input(shape=(None,), dtype='int32', name='text')
image_input = keras.Input(shape=(224, 224, 3), name='image')

# 文本分支
text_features = layers.Embedding(10000, 128)(text_input)
text_features = layers.LSTM(64)(text_features)

# 图像分支
image_features = layers.Conv2D(32, 3, activation='relu')(image_input)
image_features = layers.GlobalAveragePooling2D()(image_features)

# 合并
combined = layers.concatenate([text_features, image_features])
output = layers.Dense(1, activation='sigmoid')(combined)

model = keras.Model(inputs=[text_input, image_input], outputs=output)

model.summary()

# 多输出示例
inputs = keras.Input(shape=(100,))
x = layers.Dense(64, activation='relu')(x)

# 两个输出头
output1 = layers.Dense(1, activation='sigmoid', name='binary_output')(x)
output2 = layers.Dense(5, activation='softmax', name='multi_output')(x)

model = keras.Model(inputs=inputs, outputs=[output1, output2])

print("""
Functional API优势：
- 支持复杂拓扑
- 多输入多输出
- 共享层
- 残差连接
- 更灵活
""")
```

### 2.3 Model子类化

```python
"""
Model子类化
@author erik.zhou
"""
from tensorflow import keras
from tensorflow.keras import layers

class MyModel(keras.Model):
    """自定义模型"""
    
    def __init__(self, num_classes=10):
        super(MyModel, self).__init__()
        self.dense1 = layers.Dense(64, activation='relu')
        self.dropout = layers.Dropout(0.5)
        self.dense2 = layers.Dense(num_classes, activation='softmax')
    
    def call(self, inputs, training=False):
        """前向传播"""
        x = self.dense1(inputs)
        if training:
            x = self.dropout(x, training=training)
        return self.dense2(x)
    
    def get_config(self):
        """序列化配置"""
        return {'num_classes': self.dense2.units}

# 使用模型
model = MyModel(num_classes=10)
model.build((None, 784))
model.summary()

print("""
Model子类化特点：
- 最大灵活性
- 命令式编程
- 动态计算图
- 适合研究

缺点：
- 不能序列化
- 调试较困难
- 不能用plot_model
""")
```

---

## 3. 层和激活函数

### 3.1 常用层

```python
"""
Keras常用层
@author erik.zhou
"""
from tensorflow.keras import layers

print("""
核心层：

1. Dense（全连接层）
2. Dropout（随机失活）
3. BatchNormalization（批归一化）
4. LayerNormalization（层归一化）
5. Activation（激活函数）

卷积层：

6. Conv1D/Conv2D/Conv3D
7. SeparableConv2D（深度可分离卷积）
8. DepthwiseConv2D（深度卷积）
9. Conv2DTranspose（转置卷积）

池化层：

10. MaxPooling2D/AveragePooling2D
11. GlobalMaxPooling2D/GlobalAveragePooling2D

循环层：

12. SimpleRNN
13. LSTM
14. GRU
15. Bidirectional（双向包装器）

注意力层：

16. MultiHeadAttention
17. Attention
18. AdditiveAttention

其他：

19. Embedding（嵌入层）
20. Flatten/Reshape
21. Concatenate/Add/Multiply
22. Lambda（自定义操作）
""")

# 示例：构建ResNet块
def residual_block(x, filters):
    """残差块"""
    shortcut = x
    
    x = layers.Conv2D(filters, 3, padding='same')(x)
    x = layers.BatchNormalization()(x)
    x = layers.Activation('relu')(x)
    
    x = layers.Conv2D(filters, 3, padding='same')(x)
    x = layers.BatchNormalization()(x)
    
    x = layers.Add()([x, shortcut])
    x = layers.Activation('relu')(x)
    
    return x
```

### 3.2 激活函数

```python
"""
激活函数
@author erik.zhou
"""
from tensorflow.keras import layers, activations
import tensorflow as tf

print("""
常用激活函数：

1. ReLU：
   - 最常用
   - 解决梯度消失
   - 计算高效

2. LeakyReLU：
   - 解决ReLU死神经元
   - 负值有小梯度

3. ELU：
   - 负值平滑
   - 均值接近0

4. Sigmoid：
   - 输出0-1
   - 二分类输出层

5. Tanh：
   - 输出-1到1
   - 零中心化

6. Softmax：
   - 多分类输出层
   - 概率分布

7. Swish/SiLU：
   - 自门控
   - 性能优于ReLU

8. GELU：
   - Transformer常用
   - 平滑非线性
""")

# 使用激活函数
model = keras.Sequential([
    layers.Dense(64),
    layers.Activation('relu'),  # 方式1
    layers.Dense(64, activation='relu'),  # 方式2
    layers.Dense(64, activation=tf.nn.relu),  # 方式3
    layers.Dense(10, activation='softmax')
])
```

---

## 4. 模型编译和训练

### 4.1 编译模型

```python
"""
模型编译
@author erik.zhou
"""
from tensorflow import keras

model = keras.Sequential([
    keras.layers.Dense(64, activation='relu', input_shape=(10,)),
    keras.layers.Dense(1)
])

# 基本编译
model.compile(
    optimizer='adam',
    loss='mse',
    metrics=['mae']
)

# 自定义优化器
model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=0.001),
    loss='mse',
    metrics=['mae']
)

# 多个指标
model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=[
        'accuracy',
        keras.metrics.TopKCategoricalAccuracy(k=5),
        keras.metrics.Precision(),
        keras.metrics.Recall()
    ]
)

# 多输出模型
model.compile(
    optimizer='adam',
    loss={
        'output1': 'binary_crossentropy',
        'output2': 'categorical_crossentropy'
    },
    loss_weights={
        'output1': 1.0,
        'output2': 0.5
    },
    metrics={
        'output1': ['accuracy'],
        'output2': ['accuracy']
    }
)

print("""
常用损失函数：

分类：
- binary_crossentropy（二分类）
- categorical_crossentropy（多分类，one-hot）
- sparse_categorical_crossentropy（多分类，整数标签）

回归：
- mse（均方误差）
- mae（平均绝对误差）
- huber（Huber损失）

常用优化器：
- SGD（随机梯度下降）
- Adam（自适应学习率）
- RMSprop
- AdamW（带权重衰减）
""")
```

### 4.2 训练模型

```python
"""
模型训练
@author erik.zhou
"""
from tensorflow import keras
import numpy as np

# 准备数据
x_train = np.random.randn(1000, 10)
y_train = np.random.randint(0, 2, 1000)

# 训练
history = model.fit(
    x_train, y_train,
    batch_size=32,
    epochs=10,
    validation_split=0.2,
    verbose=1
)

# 使用验证集
x_val = np.random.randn(200, 10)
y_val = np.random.randint(0, 2, 200)

history = model.fit(
    x_train, y_train,
    batch_size=32,
    epochs=10,
    validation_data=(x_val, y_val)
)

# 使用生成器
def data_generator(batch_size=32):
    while True:
        x = np.random.randn(batch_size, 10)
        y = np.random.randint(0, 2, batch_size)
        yield x, y

history = model.fit(
    data_generator(),
    steps_per_epoch=100,
    epochs=10
)

# 评估
test_loss, test_acc = model.evaluate(x_val, y_val)
print(f'测试准确率: {test_acc:.4f}')

# 预测
predictions = model.predict(x_val[:5])
print(f'预测结果: {predictions}')
```

---

## 5. 回调函数

### 5.1 内置回调

```python
"""
Keras回调函数
@author erik.zhou
"""
from tensorflow import keras

# ModelCheckpoint：保存最佳模型
checkpoint = keras.callbacks.ModelCheckpoint(
    filepath='best_model.h5',
    monitor='val_loss',
    save_best_only=True,
    save_weights_only=False,
    verbose=1
)

# EarlyStopping：早停
early_stop = keras.callbacks.EarlyStopping(
    monitor='val_loss',
    patience=5,
    restore_best_weights=True,
    verbose=1
)

# ReduceLROnPlateau：学习率衰减
reduce_lr = keras.callbacks.ReduceLROnPlateau(
    monitor='val_loss',
    factor=0.5,
    patience=3,
    min_lr=1e-7,
    verbose=1
)

# TensorBoard：可视化
tensorboard = keras.callbacks.TensorBoard(
    log_dir='./logs',
    histogram_freq=1,
    write_graph=True,
    write_images=True
)

# CSVLogger：记录日志
csv_logger = keras.callbacks.CSVLogger(
    'training.log',
    separator=',',
    append=False
)

# 使用回调
history = model.fit(
    x_train, y_train,
    epochs=100,
    validation_data=(x_val, y_val),
    callbacks=[checkpoint, early_stop, reduce_lr, tensorboard, csv_logger]
)
```

### 5.2 自定义回调

```python
"""
自定义回调函数
@author erik.zhou
"""
from tensorflow import keras

class CustomCallback(keras.callbacks.Callback):
    """自定义回调"""
    
    def on_train_begin(self, logs=None):
        """训练开始时调用"""
        print("开始训练...")
    
    def on_epoch_end(self, epoch, logs=None):
        """每个epoch结束时调用"""
        print(f"\nEpoch {epoch + 1} 结束")
        print(f"训练损失: {logs['loss']:.4f}")
        print(f"验证损失: {logs.get('val_loss', 0):.4f}")
    
    def on_train_end(self, logs=None):
        """训练结束时调用"""
        print("训练完成!")

# 学习率调度回调
class LearningRateScheduler(keras.callbacks.Callback):
    """自定义学习率调度"""
    
    def __init__(self, schedule):
        super(LearningRateScheduler, self).__init__()
        self.schedule = schedule
    
    def on_epoch_begin(self, epoch, logs=None):
        lr = self.schedule(epoch)
        keras.backend.set_value(self.model.optimizer.lr, lr)
        print(f'\nEpoch {epoch + 1}: 学习率 = {lr}')

# 使用自定义回调
def lr_schedule(epoch):
    """学习率调度函数"""
    if epoch < 10:
        return 0.001
    elif epoch < 20:
        return 0.0001
    else:
        return 0.00001

custom_callback = CustomCallback()
lr_scheduler = LearningRateScheduler(lr_schedule)

history = model.fit(
    x_train, y_train,
    epochs=30,
    callbacks=[custom_callback, lr_scheduler]
)
```

---

## 6. 预训练模型

### 6.1 使用预训练模型

```python
"""
Keras预训练模型
@author erik.zhou
"""
from tensorflow.keras.applications import (
    VGG16, VGG19,
    ResNet50, ResNet101,
    InceptionV3, InceptionResNetV2,
    MobileNet, MobileNetV2,
    DenseNet121, DenseNet201,
    EfficientNetB0, EfficientNetB7
)

# 加载预训练模型
base_model = ResNet50(
    weights='imagenet',  # 使用ImageNet预训练权重
    include_top=False,  # 不包含顶层分类器
    input_shape=(224, 224, 3)
)

# 查看模型结构
base_model.summary()

# 冻结预训练层
base_model.trainable = False

# 添加自定义分类器
from tensorflow.keras import layers

inputs = keras.Input(shape=(224, 224, 3))
x = base_model(inputs, training=False)
x = layers.GlobalAveragePooling2D()(x)
x = layers.Dense(256, activation='relu')(x)
x = layers.Dropout(0.5)(x)
outputs = layers.Dense(10, activation='softmax')(x)

model = keras.Model(inputs, outputs)

model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

print("""
Keras预训练模型：

图像分类：
- VGG16/VGG19
- ResNet50/ResNet101/ResNet152
- InceptionV3/InceptionResNetV2
- MobileNet/MobileNetV2/MobileNetV3
- EfficientNet系列
- DenseNet系列

使用场景：
- 迁移学习
- 特征提取
- 微调
- 知识蒸馏
""")
```

---

## 7. 迁移学习

### 7.1 特征提取

```python
"""
迁移学习 - 特征提取
@author erik.zhou
"""
from tensorflow.keras.applications import MobileNetV2
from tensorflow.keras import layers

# 加载预训练模型作为特征提取器
base_model = MobileNetV2(
    weights='imagenet',
    include_top=False,
    input_shape=(224, 224, 3)
)

# 冻结所有层
base_model.trainable = False

# 构建新模型
model = keras.Sequential([
    base_model,
    layers.GlobalAveragePooling2D(),
    layers.Dense(128, activation='relu'),
    layers.Dropout(0.5),
    layers.Dense(num_classes, activation='softmax')
])

model.compile(
    optimizer=keras.optimizers.Adam(1e-3),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# 训练
history = model.fit(
    train_dataset,
    epochs=10,
    validation_data=val_dataset
)

print("""
特征提取策略：
- 冻结预训练层
- 只训练新添加的层
- 快速训练
- 适合小数据集
""")
```

---

## 8. 模型微调

### 8.1 微调预训练模型

```python
"""
模型微调（Fine-tuning）
@author erik.zhou
"""
from tensorflow.keras.applications import ResNet50

# 第一阶段：特征提取
base_model = ResNet50(weights='imagenet', include_top=False)
base_model.trainable = False

model = keras.Sequential([
    base_model,
    layers.GlobalAveragePooling2D(),
    layers.Dense(256, activation='relu'),
    layers.Dense(num_classes, activation='softmax')
])

model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# 训练顶层
model.fit(train_dataset, epochs=10, validation_data=val_dataset)

# 第二阶段：微调
# 解冻部分层
base_model.trainable = True

# 冻结前面的层，只微调后面的层
for layer in base_model.layers[:-30]:
    layer.trainable = False

# 重新编译（使用更小的学习率）
model.compile(
    optimizer=keras.optimizers.Adam(1e-5),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# 继续训练
history_fine = model.fit(
    train_dataset,
    epochs=10,
    validation_data=val_dataset
)

print("""
微调策略：

1. 两阶段训练：
   - 阶段1：冻结预训练层，训练新层
   - 阶段2：解冻部分层，微调

2. 学习率设置：
   - 特征提取：正常学习率（1e-3）
   - 微调：小学习率（1e-5）

3. 解冻策略：
   - 从后向前逐层解冻
   - 或只解冻最后几层

4. 注意事项：
   - BatchNormalization层保持冻结
   - 避免过拟合
   - 监控验证集性能
""")
```

---

## 9. 自定义组件

### 9.1 自定义层

```python
"""
自定义层
@author erik.zhou
"""
from tensorflow import keras
import tensorflow as tf

class MyDense(keras.layers.Layer):
    """自定义全连接层"""
    
    def __init__(self, units, **kwargs):
        super(MyDense, self).__init__(**kwargs)
        self.units = units
    
    def build(self, input_shape):
        """创建权重"""
        self.w = self.add_weight(
            shape=(input_shape[-1], self.units),
            initializer='glorot_uniform',
            trainable=True,
            name='kernel'
        )
        self.b = self.add_weight(
            shape=(self.units,),
            initializer='zeros',
            trainable=True,
            name='bias'
        )
    
    def call(self, inputs):
        """前向传播"""
        return tf.matmul(inputs, self.w) + self.b
    
    def get_config(self):
        """序列化配置"""
        config = super(MyDense, self).get_config()
        config.update({'units': self.units})
        return config

# 使用自定义层
layer = MyDense(10)
model = keras.Sequential([
    MyDense(64),
    keras.layers.Activation('relu'),
    MyDense(10),
    keras.layers.Activation('softmax')
])
```

### 9.2 自定义损失和指标

```python
"""
自定义损失函数和指标
@author erik.zhou
"""
import tensorflow as tf
from tensorflow import keras

# 自定义损失函数
def custom_loss(y_true, y_pred):
    """自定义损失"""
    mse = tf.reduce_mean(tf.square(y_true - y_pred))
    mae = tf.reduce_mean(tf.abs(y_true - y_pred))
    return mse + 0.5 * mae

# 自定义指标
class F1Score(keras.metrics.Metric):
    """F1分数指标"""
    
    def __init__(self, name='f1_score', **kwargs):
        super(F1Score, self).__init__(name=name, **kwargs)
        self.precision = keras.metrics.Precision()
        self.recall = keras.metrics.Recall()
    
    def update_state(self, y_true, y_pred, sample_weight=None):
        self.precision.update_state(y_true, y_pred, sample_weight)
        self.recall.update_state(y_true, y_pred, sample_weight)
    
    def result(self):
        p = self.precision.result()
        r = self.recall.result()
        return 2 * ((p * r) / (p + r + keras.backend.epsilon()))
    
    def reset_state(self):
        self.precision.reset_state()
        self.recall.reset_state()

# 使用自定义组件
model.compile(
    optimizer='adam',
    loss=custom_loss,
    metrics=[F1Score()]
)
```

---

## 10. 最佳实践

```python
"""
Keras最佳实践
@author erik.zhou
"""

print("""
Keras最佳实践：

1. 模型设计：
   - 简单任务用Sequential
   - 复杂任务用Functional API
   - 研究用Model子类化
   - 使用预训练模型

2. 数据处理：
   - 数据归一化
   - 数据增强
   - 使用tf.data管道
   - 批处理和预取

3. 训练技巧：
   - 使用回调函数
   - 早停防止过拟合
   - 学习率调度
   - 批归一化

4. 正则化：
   - Dropout（0.2-0.5）
   - L1/L2正则化
   - 数据增强
   - 早停

5. 调试：
   - 从小模型开始
   - 检查数据形状
   - 可视化训练曲线
   - 使用TensorBoard

6. 性能优化：
   - 混合精度训练
   - 分布式训练
   - 模型量化
   - 剪枝压缩

7. 部署：
   - SavedModel格式
   - TensorFlow Lite
   - TensorFlow.js
   - ONNX导出

常见陷阱：
- 忘记归一化数据
- 学习率过大或过小
- 过拟合
- 数据泄露
- 不平衡数据集
""")
```

---

## 📝 学习检查清单

- [ ] 理解Keras的设计理念
- [ ] 掌握Sequential和Functional API
- [ ] 能够使用预训练模型
- [ ] 掌握回调函数机制
- [ ] 理解自定义组件
- [ ] 掌握模型微调技术
- [ ] 能够进行迁移学习
- [ ] 了解Keras生态系统
- [ ] 掌握模型保存和加载
- [ ] 能够优化模型性能

## 🔗 相关资源

- [Keras官方文档](https://keras.io/)
- [Keras代码示例](https://keras.io/examples/)
- [TensorFlow教程](https://www.tensorflow.org/tutorials)
- [Keras Applications](https://keras.io/api/applications/)
- [Keras GitHub](https://github.com/keras-team/keras)

---

**@author erik.zhou**  
**最后更新**: 2026-02-13
