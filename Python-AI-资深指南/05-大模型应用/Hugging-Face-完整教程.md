# Hugging Face 完整教程

> @author erik.zhou
> 
> 更新日期：2026-02-13

## 📋 课程概述

Hugging Face 是开源 AI 社区的核心平台，提供了 Transformers 库、模型库、数据集库等工具。本教程将全面介绍如何使用 Hugging Face 开发 AI 应用。

### 学习目标
- 掌握 Transformers 库的使用
- 学会加载和使用预训练模型
- 理解 Pipeline 快速应用
- 掌握模型微调和部署

---

## 1. 快速开始

### 1.1 安装

```python
# 安装 Transformers
pip install transformers

# 安装 PyTorch（推荐）
pip install torch

# 或安装 TensorFlow
pip install tensorflow

# 安装其他依赖
pip install datasets accelerate
```

### 1.2 第一个模型

```python
from transformers import pipeline

# 创建文本生成 Pipeline
generator = pipeline("text-generation", model="gpt2")

# 生成文本
result = generator("Hello, I am", max_length=30, num_return_sequences=1)
print(result[0]['generated_text'])
```

---

## 2. Pipeline 快速应用

### 2.1 文本分类

```python
from transformers import pipeline

# 情感分析
classifier = pipeline("sentiment-analysis")
result = classifier("I love this product!")
print(result)  # [{'label': 'POSITIVE', 'score': 0.9998}]

# 批量处理
texts = [
    "I love this!",
    "I hate this!",
    "It's okay."
]
results = classifier(texts)
for text, result in zip(texts, results):
    print(f"{text}: {result['label']} ({result['score']:.2f})")
```

### 2.2 命名实体识别

```python
# NER Pipeline
ner = pipeline("ner", grouped_entities=True)

text = "My name is John and I live in New York"
entities = ner(text)

for entity in entities:
    print(f"{entity['word']}: {entity['entity_group']} ({entity['score']:.2f})")
```

### 2.3 问答系统

```python
# QA Pipeline
qa = pipeline("question-answering")

context = """
Python is a high-level programming language.
It was created by Guido van Rossum in 1991.
"""

question = "Who created Python?"

result = qa(question=question, context=context)
print(f"Answer: {result['answer']}")
print(f"Score: {result['score']:.2f}")
```

### 2.4 文本摘要

```python
# 摘要 Pipeline
summarizer = pipeline("summarization")

article = """
Long article text here...
"""

summary = summarizer(article, max_length=130, min_length=30, do_sample=False)
print(summary[0]['summary_text'])
```

### 2.5 翻译

```python
# 翻译 Pipeline
translator = pipeline("translation_en_to_zh", model="Helsinki-NLP/opus-mt-en-zh")

text = "Hello, how are you?"
translation = translator(text)
print(translation[0]['translation_text'])
```

---

## 3. 模型加载和使用

### 3.1 加载预训练模型

```python
from transformers import AutoTokenizer, AutoModel

# 加载 Tokenizer 和模型
model_name = "bert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModel.from_pretrained(model_name)

# 使用模型
text = "Hello, world!"
inputs = tokenizer(text, return_tensors="pt")
outputs = model(**inputs)

print(f"Last hidden state shape: {outputs.last_hidden_state.shape}")
```

### 3.2 不同任务的模型

```python
from transformers import (
    AutoModelForSequenceClassification,
    AutoModelForQuestionAnswering,
    AutoModelForTokenClassification,
    AutoModelForCausalLM
)

# 文本分类
classification_model = AutoModelForSequenceClassification.from_pretrained(
    "distilbert-base-uncased-finetuned-sst-2-english"
)

# 问答
qa_model = AutoModelForQuestionAnswering.from_pretrained(
    "distilbert-base-cased-distilled-squad"
)

# Token 分类（NER）
ner_model = AutoModelForTokenClassification.from_pretrained(
    "dslim/bert-base-NER"
)

# 文本生成
generation_model = AutoModelForCausalLM.from_pretrained("gpt2")
```

### 3.3 本地保存和加载

```python
# 保存模型
model.save_pretrained("./my_model")
tokenizer.save_pretrained("./my_model")

# 加载本地模型
loaded_model = AutoModel.from_pretrained("./my_model")
loaded_tokenizer = AutoTokenizer.from_pretrained("./my_model")
```

---

## 4. Tokenizer 详解

### 4.1 基础使用

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

# 编码文本
text = "Hello, world!"
tokens = tokenizer.tokenize(text)
print(f"Tokens: {tokens}")

# 转换为 ID
token_ids = tokenizer.convert_tokens_to_ids(tokens)
print(f"Token IDs: {token_ids}")

# 一步完成
encoded = tokenizer(text)
print(f"Encoded: {encoded}")
```

### 4.2 批量处理

```python
# 批量编码
texts = [
    "First sentence.",
    "Second sentence.",
    "Third sentence."
]

# 自动填充和截断
encoded = tokenizer(
    texts,
    padding=True,  # 填充到最长序列
    truncation=True,  # 截断超长序列
    max_length=128,
    return_tensors="pt"  # 返回 PyTorch 张量
)

print(f"Input IDs shape: {encoded['input_ids'].shape}")
print(f"Attention mask shape: {encoded['attention_mask'].shape}")
```

### 4.3 特殊 Token

```python
# 查看特殊 Token
print(f"CLS token: {tokenizer.cls_token}")
print(f"SEP token: {tokenizer.sep_token}")
print(f"PAD token: {tokenizer.pad_token}")
print(f"UNK token: {tokenizer.unk_token}")

# 添加特殊 Token
text = "Hello, world!"
encoded = tokenizer(
    text,
    add_special_tokens=True,  # 添加 [CLS] 和 [SEP]
    return_tensors="pt"
)

# 解码
decoded = tokenizer.decode(encoded['input_ids'][0])
print(f"Decoded: {decoded}")
```

---

## 5. 模型推理

### 5.1 文本分类推理

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

# 加载模型
model_name = "distilbert-base-uncased-finetuned-sst-2-english"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)

# 推理
text = "I love this product!"
inputs = tokenizer(text, return_tensors="pt")

with torch.no_grad():
    outputs = model(**inputs)
    predictions = torch.nn.functional.softmax(outputs.logits, dim=-1)

# 获取结果
labels = ["NEGATIVE", "POSITIVE"]
predicted_class = torch.argmax(predictions).item()
confidence = predictions[0][predicted_class].item()

print(f"Prediction: {labels[predicted_class]}")
print(f"Confidence: {confidence:.2f}")
```

### 5.2 文本生成推理

```python
from transformers import AutoTokenizer, AutoModelForCausalLM

# 加载模型
model_name = "gpt2"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name)

# 生成文本
prompt = "Once upon a time"
inputs = tokenizer(prompt, return_tensors="pt")

# 生成
outputs = model.generate(
    inputs['input_ids'],
    max_length=50,
    num_return_sequences=1,
    temperature=0.7,
    top_k=50,
    top_p=0.95,
    do_sample=True
)

# 解码
generated_text = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(generated_text)
```

### 5.3 批量推理

```python
def batch_inference(texts, model, tokenizer, batch_size=8):
    """批量推理"""
    results = []
    
    for i in range(0, len(texts), batch_size):
        batch = texts[i:i+batch_size]
        
        # 编码
        inputs = tokenizer(
            batch,
            padding=True,
            truncation=True,
            return_tensors="pt"
        )
        
        # 推理
        with torch.no_grad():
            outputs = model(**inputs)
            predictions = torch.nn.functional.softmax(outputs.logits, dim=-1)
        
        # 收集结果
        for pred in predictions:
            results.append(pred.tolist())
    
    return results

# 使用
texts = ["Text 1", "Text 2", "Text 3"] * 10
results = batch_inference(texts, model, tokenizer)
```

---

## 6. 数据集处理

### 6.1 加载数据集

```python
from datasets import load_dataset

# 加载公开数据集
dataset = load_dataset("imdb")

print(f"Train size: {len(dataset['train'])}")
print(f"Test size: {len(dataset['test'])}")

# 查看样本
print(dataset['train'][0])
```

### 6.2 数据预处理

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

def preprocess_function(examples):
    """预处理函数"""
    return tokenizer(
        examples['text'],
        truncation=True,
        padding='max_length',
        max_length=512
    )

# 应用预处理
tokenized_dataset = dataset.map(
    preprocess_function,
    batched=True,
    remove_columns=['text']
)

print(tokenized_dataset['train'][0])
```

### 6.3 自定义数据集

```python
from datasets import Dataset

# 创建自定义数据集
data = {
    'text': [
        "I love this!",
        "I hate this!",
        "It's okay."
    ],
    'label': [1, 0, 2]
}

custom_dataset = Dataset.from_dict(data)

# 保存数据集
custom_dataset.save_to_disk("./my_dataset")

# 加载数据集
loaded_dataset = Dataset.load_from_disk("./my_dataset")
```

---

## 7. 模型微调

### 7.1 准备训练

```python
from transformers import (
    AutoTokenizer,
    AutoModelForSequenceClassification,
    TrainingArguments,
    Trainer
)
from datasets import load_dataset

# 加载数据和模型
dataset = load_dataset("imdb")
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModelForSequenceClassification.from_pretrained(
    "bert-base-uncased",
    num_labels=2
)

# 预处理
def tokenize_function(examples):
    return tokenizer(
        examples['text'],
        padding='max_length',
        truncation=True,
        max_length=512
    )

tokenized_datasets = dataset.map(tokenize_function, batched=True)

# 小数据集用于演示
small_train_dataset = tokenized_datasets["train"].shuffle(seed=42).select(range(1000))
small_eval_dataset = tokenized_datasets["test"].shuffle(seed=42).select(range(1000))
```

### 7.2 配置训练参数

```python
training_args = TrainingArguments(
    output_dir="./results",
    evaluation_strategy="epoch",
    learning_rate=2e-5,
    per_device_train_batch_size=8,
    per_device_eval_batch_size=8,
    num_train_epochs=3,
    weight_decay=0.01,
    logging_dir='./logs',
    logging_steps=10,
    save_strategy="epoch",
    load_best_model_at_end=True,
)
```

### 7.3 训练模型

```python
from transformers import Trainer
import numpy as np
from datasets import load_metric

# 定义评估指标
metric = load_metric("accuracy")

def compute_metrics(eval_pred):
    logits, labels = eval_pred
    predictions = np.argmax(logits, axis=-1)
    return metric.compute(predictions=predictions, references=labels)

# 创建 Trainer
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=small_train_dataset,
    eval_dataset=small_eval_dataset,
    compute_metrics=compute_metrics,
)

# 开始训练
trainer.train()

# 评估
eval_results = trainer.evaluate()
print(f"Evaluation results: {eval_results}")

# 保存模型
trainer.save_model("./fine_tuned_model")
```

---

## 8. 模型部署

### 8.1 使用 FastAPI 部署

```python
from fastapi import FastAPI
from pydantic import BaseModel
from transformers import pipeline

app = FastAPI()

# 加载模型
classifier = pipeline("sentiment-analysis")

class TextRequest(BaseModel):
    text: str

class PredictionResponse(BaseModel):
    label: str
    score: float

@app.post("/predict", response_model=PredictionResponse)
async def predict(request: TextRequest):
    result = classifier(request.text)[0]
    return PredictionResponse(
        label=result['label'],
        score=result['score']
    )

# 运行: uvicorn main:app --reload
```

### 8.2 使用 Gradio 创建界面

```python
import gradio as gr
from transformers import pipeline

# 加载模型
classifier = pipeline("sentiment-analysis")

def predict(text):
    """预测函数"""
    result = classifier(text)[0]
    return f"{result['label']}: {result['score']:.2f}"

# 创建界面
interface = gr.Interface(
    fn=predict,
    inputs=gr.Textbox(lines=5, placeholder="Enter text here..."),
    outputs="text",
    title="Sentiment Analysis",
    description="Analyze the sentiment of your text"
)

# 启动
interface.launch()
```

---

## 9. 高级功能

### 9.1 模型量化

```python
from transformers import AutoModelForSequenceClassification
import torch

# 加载模型
model = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased")

# 动态量化
quantized_model = torch.quantization.quantize_dynamic(
    model,
    {torch.nn.Linear},
    dtype=torch.qint8
)

# 保存量化模型
torch.save(quantized_model.state_dict(), "quantized_model.pth")
```

### 9.2 模型导出 ONNX

```python
from transformers import AutoTokenizer, AutoModel
import torch

# 加载模型
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
model = AutoModel.from_pretrained("bert-base-uncased")

# 准备输入
dummy_input = tokenizer("Hello world", return_tensors="pt")

# 导出 ONNX
torch.onnx.export(
    model,
    tuple(dummy_input.values()),
    "model.onnx",
    input_names=['input_ids', 'attention_mask'],
    output_names=['last_hidden_state'],
    dynamic_axes={
        'input_ids': {0: 'batch', 1: 'sequence'},
        'attention_mask': {0: 'batch', 1: 'sequence'},
        'last_hidden_state': {0: 'batch', 1: 'sequence'}
    }
)
```

---

## 10. 总结

### 核心要点
1. Pipeline 提供快速应用接口
2. Transformers 支持多种任务和模型
3. 数据预处理是关键步骤
4. Trainer API 简化微调流程
5. 多种部署方式可选

### 学习建议
1. 从 Pipeline 开始实践
2. 理解 Tokenizer 的工作原理
3. 学习模型微调技巧
4. 关注模型优化和部署

### 下一步
- 探索更多预训练模型
- 学习高级微调技术
- 实践模型压缩和加速
- 研究多模态模型

---

**@author erik.zhou**
