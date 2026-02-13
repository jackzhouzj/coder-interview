# OpenAI API 完整教程

> @author erik.zhou
> 
> 更新日期：2026-02-13

## 📋 课程概述

OpenAI API 提供了访问 GPT 系列模型的接口，包括 GPT-4、GPT-3.5-Turbo 等。本教程将全面介绍如何使用 OpenAI API 开发应用。

### 学习目标
- 掌握 OpenAI API 的基本使用
- 理解不同模型的特点和选择
- 学会处理流式响应和错误
- 掌握 Function Calling 和高级功能

---

## 1. 快速开始

### 1.1 安装和配置

```python
# 安装 OpenAI SDK
pip install openai

# 设置 API Key
import os
os.environ["OPENAI_API_KEY"] = "your-api-key"

# 或者在代码中设置
from openai import OpenAI

client = OpenAI(api_key="your-api-key")
```

### 1.2 第一个 API 调用

```python
from openai import OpenAI

client = OpenAI()

# 创建聊天完成
response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Python?"}
    ]
)

print(response.choices[0].message.content)
```

---

## 2. 模型选择

### 2.1 可用模型

```python
# GPT-4 系列（最强大）
models = {
    "gpt-4": "最强大，适合复杂任务",
    "gpt-4-turbo": "更快更便宜的 GPT-4",
    "gpt-4o": "多模态模型，支持图像",
}

# GPT-3.5 系列（性价比高）
models_35 = {
    "gpt-3.5-turbo": "快速且经济",
    "gpt-3.5-turbo-16k": "更长的上下文窗口",
}

# 使用示例
response = client.chat.completions.create(
    model="gpt-4-turbo",
    messages=[{"role": "user", "content": "Explain quantum computing"}]
)
```

### 2.2 模型参数

```python
response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": "Write a poem"}],
    
    # 温度：控制随机性（0-2）
    temperature=0.7,
    
    # Top P：核采样（0-1）
    top_p=1.0,
    
    # 最大 Token 数
    max_tokens=500,
    
    # 频率惩罚（-2.0 到 2.0）
    frequency_penalty=0.0,
    
    # 存在惩罚（-2.0 到 2.0）
    presence_penalty=0.0,
    
    # 停止序列
    stop=["\n\n", "END"],
    
    # 返回多个结果
    n=1,
)
```

---

## 3. 聊天完成 API

### 3.1 基础对话

```python
def chat(messages, model="gpt-3.5-turbo"):
    """简单的聊天函数"""
    response = client.chat.completions.create(
        model=model,
        messages=messages
    )
    return response.choices[0].message.content

# 使用
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Hello!"}
]

reply = chat(messages)
print(reply)

# 继续对话
messages.append({"role": "assistant", "content": reply})
messages.append({"role": "user", "content": "Tell me a joke"})

reply = chat(messages)
print(reply)
```

### 3.2 流式响应

```python
def stream_chat(messages, model="gpt-3.5-turbo"):
    """流式聊天"""
    stream = client.chat.completions.create(
        model=model,
        messages=messages,
        stream=True
    )
    
    for chunk in stream:
        if chunk.choices[0].delta.content is not None:
            print(chunk.choices[0].delta.content, end="", flush=True)
    print()

# 使用
messages = [
    {"role": "user", "content": "Write a short story"}
]

stream_chat(messages)
```

### 3.3 异步调用

```python
import asyncio
from openai import AsyncOpenAI

async_client = AsyncOpenAI()

async def async_chat(messages):
    """异步聊天"""
    response = await async_client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=messages
    )
    return response.choices[0].message.content

# 使用
async def main():
    messages = [
        {"role": "user", "content": "What is AI?"}
    ]
    reply = await async_chat(messages)
    print(reply)

asyncio.run(main())
```

---

## 4. Function Calling

### 4.1 定义函数

```python
# 定义函数工具
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get the current weather for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "The city and state, e.g. San Francisco, CA"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "The temperature unit"
                    }
                },
                "required": ["location"]
            }
        }
    }
]

# 实现函数
def get_weather(location, unit="celsius"):
    """获取天气（模拟）"""
    return {
        "location": location,
        "temperature": 22,
        "unit": unit,
        "condition": "sunny"
    }
```

### 4.2 使用 Function Calling

```python
import json

def chat_with_functions(messages, tools):
    """带函数调用的聊天"""
    # 第一次调用
    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=messages,
        tools=tools,
        tool_choice="auto"
    )
    
    response_message = response.choices[0].message
    tool_calls = response_message.tool_calls
    
    # 如果模型要调用函数
    if tool_calls:
        # 执行函数调用
        messages.append(response_message)
        
        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_args = json.loads(tool_call.function.arguments)
            
            # 调用实际函数
            if function_name == "get_weather":
                function_response = get_weather(**function_args)
            
            # 添加函数响应
            messages.append({
                "tool_call_id": tool_call.id,
                "role": "tool",
                "name": function_name,
                "content": json.dumps(function_response)
            })
        
        # 第二次调用，获取最终响应
        second_response = client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=messages
        )
        
        return second_response.choices[0].message.content
    
    return response_message.content

# 使用
messages = [
    {"role": "user", "content": "What's the weather in San Francisco?"}
]

reply = chat_with_functions(messages, tools)
print(reply)
```

---

## 5. Embeddings

### 5.1 生成 Embeddings

```python
def get_embedding(text, model="text-embedding-3-small"):
    """获取文本的 Embedding"""
    response = client.embeddings.create(
        model=model,
        input=text
    )
    return response.data[0].embedding

# 使用
text = "LangChain is a framework for developing applications"
embedding = get_embedding(text)
print(f"Embedding dimension: {len(embedding)}")
```

### 5.2 批量生成

```python
def get_embeddings_batch(texts, model="text-embedding-3-small"):
    """批量获取 Embeddings"""
    response = client.embeddings.create(
        model=model,
        input=texts
    )
    return [data.embedding for data in response.data]

# 使用
texts = [
    "First document",
    "Second document",
    "Third document"
]

embeddings = get_embeddings_batch(texts)
print(f"Generated {len(embeddings)} embeddings")
```

### 5.3 相似度计算

```python
import numpy as np

def cosine_similarity(a, b):
    """计算余弦相似度"""
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# 使用
text1 = "Python is a programming language"
text2 = "Java is a programming language"
text3 = "I love pizza"

emb1 = get_embedding(text1)
emb2 = get_embedding(text2)
emb3 = get_embedding(text3)

print(f"Similarity(1,2): {cosine_similarity(emb1, emb2):.3f}")
print(f"Similarity(1,3): {cosine_similarity(emb1, emb3):.3f}")
```

---

## 6. 图像处理（GPT-4 Vision）

### 6.1 分析图像

```python
def analyze_image(image_url, prompt="What's in this image?"):
    """分析图像"""
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": prompt},
                    {
                        "type": "image_url",
                        "image_url": {"url": image_url}
                    }
                ]
            }
        ],
        max_tokens=300
    )
    return response.choices[0].message.content

# 使用
image_url = "https://example.com/image.jpg"
description = analyze_image(image_url)
print(description)
```

### 6.2 本地图像

```python
import base64

def encode_image(image_path):
    """编码本地图像"""
    with open(image_path, "rb") as image_file:
        return base64.b64encode(image_file.read()).decode('utf-8')

def analyze_local_image(image_path, prompt="Describe this image"):
    """分析本地图像"""
    base64_image = encode_image(image_path)
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": prompt},
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": f"data:image/jpeg;base64,{base64_image}"
                        }
                    }
                ]
            }
        ]
    )
    return response.choices[0].message.content

# 使用
description = analyze_local_image("./image.jpg")
print(description)
```

---

## 7. 错误处理和重试

### 7.1 错误处理

```python
from openai import OpenAIError, RateLimitError, APIError
import time

def safe_chat(messages, max_retries=3):
    """带错误处理的聊天"""
    for attempt in range(max_retries):
        try:
            response = client.chat.completions.create(
                model="gpt-3.5-turbo",
                messages=messages
            )
            return response.choices[0].message.content
        
        except RateLimitError:
            print(f"Rate limit exceeded. Retrying in {2 ** attempt} seconds...")
            time.sleep(2 ** attempt)
        
        except APIError as e:
            print(f"API error: {e}")
            if attempt == max_retries - 1:
                raise
            time.sleep(1)
        
        except OpenAIError as e:
            print(f"OpenAI error: {e}")
            raise
    
    raise Exception("Max retries exceeded")

# 使用
messages = [{"role": "user", "content": "Hello"}]
reply = safe_chat(messages)
```

### 7.2 指数退避重试

```python
import time
from functools import wraps

def retry_with_exponential_backoff(
    max_retries=5,
    initial_delay=1,
    exponential_base=2,
    jitter=True
):
    """指数退避重试装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            num_retries = 0
            delay = initial_delay
            
            while True:
                try:
                    return func(*args, **kwargs)
                
                except RateLimitError:
                    num_retries += 1
                    
                    if num_retries > max_retries:
                        raise
                    
                    # 添加抖动
                    if jitter:
                        delay *= (1 + 0.1 * (2 * np.random.random() - 1))
                    
                    time.sleep(delay)
                    delay *= exponential_base
        
        return wrapper
    return decorator

@retry_with_exponential_backoff()
def chat_with_retry(messages):
    """带重试的聊天"""
    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=messages
    )
    return response.choices[0].message.content
```

---

## 8. Token 管理

### 8.1 计算 Token

```python
import tiktoken

def count_tokens(text, model="gpt-3.5-turbo"):
    """计算文本的 Token 数量"""
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

# 使用
text = "Hello, how are you?"
tokens = count_tokens(text)
print(f"Tokens: {tokens}")
```

### 8.2 Token 限制管理

```python
def truncate_text(text, max_tokens=4000, model="gpt-3.5-turbo"):
    """截断文本到指定 Token 数"""
    encoding = tiktoken.encoding_for_model(model)
    tokens = encoding.encode(text)
    
    if len(tokens) <= max_tokens:
        return text
    
    truncated_tokens = tokens[:max_tokens]
    return encoding.decode(truncated_tokens)

# 使用
long_text = "..." * 10000
truncated = truncate_text(long_text, max_tokens=1000)
```

### 8.3 成本估算

```python
def estimate_cost(prompt_tokens, completion_tokens, model="gpt-3.5-turbo"):
    """估算 API 调用成本"""
    # 价格（每 1000 tokens，美元）
    prices = {
        "gpt-3.5-turbo": {"input": 0.0005, "output": 0.0015},
        "gpt-4": {"input": 0.03, "output": 0.06},
        "gpt-4-turbo": {"input": 0.01, "output": 0.03},
    }
    
    price = prices.get(model, prices["gpt-3.5-turbo"])
    
    input_cost = (prompt_tokens / 1000) * price["input"]
    output_cost = (completion_tokens / 1000) * price["output"]
    
    return input_cost + output_cost

# 使用
response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": "Hello"}]
)

usage = response.usage
cost = estimate_cost(
    usage.prompt_tokens,
    usage.completion_tokens,
    model="gpt-3.5-turbo"
)
print(f"Cost: ${cost:.6f}")
```

---

## 9. 最佳实践

### 9.1 缓存响应

```python
from functools import lru_cache
import hashlib
import json

class ChatCache:
    """聊天缓存"""
    
    def __init__(self):
        self.cache = {}
    
    def _get_key(self, messages, model):
        """生成缓存键"""
        content = json.dumps(messages) + model
        return hashlib.md5(content.encode()).hexdigest()
    
    def get(self, messages, model):
        """获取缓存"""
        key = self._get_key(messages, model)
        return self.cache.get(key)
    
    def set(self, messages, model, response):
        """设置缓存"""
        key = self._get_key(messages, model)
        self.cache[key] = response
    
    def chat(self, messages, model="gpt-3.5-turbo"):
        """带缓存的聊天"""
        # 检查缓存
        cached = self.get(messages, model)
        if cached:
            return cached
        
        # 调用 API
        response = client.chat.completions.create(
            model=model,
            messages=messages
        )
        
        result = response.choices[0].message.content
        
        # 保存到缓存
        self.set(messages, model, result)
        
        return result

# 使用
cache = ChatCache()
messages = [{"role": "user", "content": "What is AI?"}]

# 第一次调用（API）
reply1 = cache.chat(messages)

# 第二次调用（缓存）
reply2 = cache.chat(messages)
```

### 9.2 批量处理

```python
import asyncio
from openai import AsyncOpenAI

async_client = AsyncOpenAI()

async def process_batch(messages_list):
    """批量处理多个请求"""
    tasks = []
    
    for messages in messages_list:
        task = async_client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=messages
        )
        tasks.append(task)
    
    responses = await asyncio.gather(*tasks)
    
    return [r.choices[0].message.content for r in responses]

# 使用
async def main():
    messages_list = [
        [{"role": "user", "content": "What is AI?"}],
        [{"role": "user", "content": "What is ML?"}],
        [{"role": "user", "content": "What is DL?"}],
    ]
    
    results = await process_batch(messages_list)
    for result in results:
        print(result)
        print("---")

asyncio.run(main())
```

---

## 10. 总结

### 核心要点
1. 选择合适的模型平衡性能和成本
2. 使用流式响应提升用户体验
3. Function Calling 扩展模型能力
4. 做好错误处理和重试机制
5. 管理 Token 使用控制成本

### 学习建议
1. 从简单的聊天开始实践
2. 理解不同模型的特点
3. 学习 Function Calling 的使用
4. 关注成本优化

### 下一步
- 学习 LangChain 集成 OpenAI
- 探索 Fine-tuning
- 实践复杂的应用场景
- 学习多模态应用开发

---

**@author erik.zhou**
