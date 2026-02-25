# Langfuse 完整教程

## 目录
- [1. Langfuse 简介](#1-langfuse-简介)
- [2. 环境准备](#2-环境准备)
- [3. 快速开始](#3-快速开始)
- [4. 核心概念](#4-核心概念)
- [5. Trace 追踪](#5-trace-追踪)
- [6. 与 LangChain 集成](#6-与-langchain-集成)
- [7. 与 OpenAI 集成](#7-与-openai-集成)
- [8. Prompt 管理](#8-prompt-管理)
- [9. 评估与分析](#9-评估与分析)
- [10. 最佳实践](#10-最佳实践)

---

## 1. Langfuse 简介

### 1.1 什么是 Langfuse

Langfuse 是一个开源的 LLM 工程平台，提供以下核心功能：

- **可观测性**：追踪和调试 LLM 应用的每一次调用
- **Prompt 管理**：版本化管理和 A/B 测试 Prompt
- **评估**：自动化评估 LLM 输出质量
- **分析**：成本、延迟、质量等指标的可视化分析
- **数据集管理**：管理测试数据集和评估结果

### 1.2 核心特性

```python
"""
Langfuse 核心特性

@author erik.zhou
"""

# 1. 完整的调用链追踪
# 2. 多种集成方式（LangChain、OpenAI、LlamaIndex 等）
# 3. 自托管或云服务
# 4. 实时监控和告警
# 5. 用户反馈收集
# 6. 成本分析
# 7. Prompt 版本管理
```

### 1.3 应用场景

- LLM 应用开发和调试
- 生产环境监控
- Prompt 工程和优化
- 成本控制和分析
- 质量评估和改进
- 用户行为分析

---

## 2. 环境准备

### 2.1 安装依赖

```bash
# 安装 Langfuse SDK
pip install langfuse

# 安装 LangChain 集成（可选）
pip install langfuse langchain langchain-openai

# 安装 OpenAI SDK（可选）
pip install openai
```

### 2.2 获取 API 密钥

```python
"""
获取 Langfuse API 密钥

@author erik.zhou
"""

# 方式 1：使用 Langfuse Cloud（推荐新手）
# 1. 访问 https://cloud.langfuse.com
# 2. 注册账号
# 3. 创建项目
# 4. 获取 Public Key 和 Secret Key

# 方式 2：自托管 Langfuse
# 1. 使用 Docker 部署
# 2. 配置数据库
# 3. 创建项目和 API 密钥
```

### 2.3 环境变量配置

```python
"""
配置环境变量

@author erik.zhou
"""
import os

# Langfuse 配置
os.environ["LANGFUSE_PUBLIC_KEY"] = "pk-lf-xxx"
os.environ["LANGFUSE_SECRET_KEY"] = "sk-lf-xxx"
os.environ["LANGFUSE_HOST"] = "https://cloud.langfuse.com"  # 或自托管地址

# OpenAI 配置（如果使用）
os.environ["OPENAI_API_KEY"] = "sk-xxx"
```

---

## 3. 快速开始

### 3.1 基础示例

```python
"""
Langfuse 基础使用示例

@author erik.zhou
"""
from langfuse import Langfuse

# 初始化 Langfuse 客户端
langfuse = Langfuse(
    public_key="pk-lf-xxx",
    secret_key="sk-lf-xxx",
    host="https://cloud.langfuse.com"
)

# 创建一个 Trace
trace = langfuse.trace(
    name="chat-completion",
    user_id="user-123",
    metadata={"environment": "production"}
)

# 添加 Generation（LLM 调用）
generation = trace.generation(
    name="openai-chat",
    model="gpt-4",
    input=[{"role": "user", "content": "你好，介绍一下 Python"}],
    output="Python 是一种高级编程语言...",
    metadata={"temperature": 0.7}
)

# 记录使用情况
generation.end(
    usage={
        "prompt_tokens": 10,
        "completion_tokens": 50,
        "total_tokens": 60
    }
)

# 刷新数据到 Langfuse
langfuse.flush()

print("Trace 已发送到 Langfuse")
```

### 3.2 装饰器方式

```python
"""
使用装饰器追踪函数

@author erik.zhou
"""
from langfuse.decorators import observe, langfuse_context

@observe()
def generate_story(topic: str) -> str:
    """
    生成故事
    
    Args:
        topic: 故事主题
        
    Returns:
        生成的故事
    """
    # 模拟 LLM 调用
    story = f"这是一个关于{topic}的故事..."
    
    # 添加元数据
    langfuse_context.update_current_observation(
        metadata={"topic": topic}
    )
    
    return story

@observe()
def main():
    """主函数"""
    # 设置用户 ID
    langfuse_context.update_current_trace(
        user_id="user-456",
        tags=["story-generation"]
    )
    
    # 调用函数
    story = generate_story("人工智能")
    print(story)

if __name__ == "__main__":
    main()
```

---

## 4. 核心概念

### 4.1 Trace（追踪）

```python
"""
Trace 是最顶层的概念，代表一次完整的用户交互

@author erik.zhou
"""
from langfuse import Langfuse

langfuse = Langfuse()

# 创建 Trace
trace = langfuse.trace(
    name="user-query",
    user_id="user-789",
    session_id="session-abc",
    metadata={
        "user_agent": "Mozilla/5.0",
        "ip_address": "192.168.1.1"
    },
    tags=["production", "chat"]
)

print(f"Trace ID: {trace.id}")
```

### 4.2 Span（跨度）

```python
"""
Span 表示 Trace 中的一个操作步骤

@author erik.zhou
"""

# 在 Trace 中创建 Span
span = trace.span(
    name="retrieve-context",
    input={"query": "什么是机器学习？"},
    metadata={"retriever": "vector-db"}
)

# 模拟检索操作
retrieved_docs = ["文档1", "文档2", "文档3"]

# 结束 Span
span.end(
    output={"documents": retrieved_docs},
    metadata={"num_docs": len(retrieved_docs)}
)
```

### 4.3 Generation（生成）

```python
"""
Generation 表示 LLM 的一次生成调用

@author erik.zhou
"""

# 创建 Generation
generation = trace.generation(
    name="llm-response",
    model="gpt-4",
    model_parameters={
        "temperature": 0.7,
        "max_tokens": 500
    },
    input=[
        {"role": "system", "content": "你是一个AI助手"},
        {"role": "user", "content": "解释机器学习"}
    ]
)

# 模拟 LLM 响应
response = "机器学习是人工智能的一个分支..."

# 结束 Generation
generation.end(
    output=response,
    usage={
        "prompt_tokens": 20,
        "completion_tokens": 100,
        "total_tokens": 120
    },
    metadata={"finish_reason": "stop"}
)
```

### 4.4 Event（事件）

```python
"""
Event 用于记录特定事件

@author erik.zhou
"""

# 记录事件
event = trace.event(
    name="user-feedback",
    input={"rating": 5, "comment": "很有帮助"},
    metadata={"timestamp": "2024-01-01T12:00:00Z"}
)
```

---

## 5. Trace 追踪

### 5.1 完整的 RAG 应用追踪

```python
"""
完整的 RAG 应用追踪示例

@author erik.zhou
"""
from langfuse import Langfuse
from typing import List, Dict
import time

langfuse = Langfuse()

def rag_pipeline(query: str, user_id: str) -> str:
    """
    RAG 管道
    
    Args:
        query: 用户查询
        user_id: 用户ID
        
    Returns:
        生成的回答
    """
    # 创建 Trace
    trace = langfuse.trace(
        name="rag-query",
        user_id=user_id,
        input={"query": query},
        metadata={"pipeline": "rag-v1"}
    )
    
    # 步骤 1：查询重写
    rewrite_span = trace.span(
        name="query-rewrite",
        input={"original_query": query}
    )
    
    rewritten_query = f"优化后的查询: {query}"
    rewrite_span.end(output={"rewritten_query": rewritten_query})
    
    # 步骤 2：向量检索
    retrieval_span = trace.span(
        name="vector-retrieval",
        input={"query": rewritten_query},
        metadata={"index": "knowledge-base"}
    )
    
    # 模拟检索
    time.sleep(0.1)
    documents = [
        {"id": "doc1", "content": "相关文档1", "score": 0.95},
        {"id": "doc2", "content": "相关文档2", "score": 0.88}
    ]
    
    retrieval_span.end(
        output={"documents": documents},
        metadata={"num_results": len(documents)}
    )
    
    # 步骤 3：LLM 生成
    generation = trace.generation(
        name="answer-generation",
        model="gpt-4",
        model_parameters={"temperature": 0.7},
        input=[
            {"role": "system", "content": "基于以下文档回答问题"},
            {"role": "user", "content": f"文档: {documents}\n问题: {query}"}
        ]
    )
    
    # 模拟 LLM 调用
    time.sleep(0.5)
    answer = f"基于检索到的文档，{query}的答案是..."
    
    generation.end(
        output=answer,
        usage={
            "prompt_tokens": 150,
            "completion_tokens": 80,
            "total_tokens": 230
        }
    )
    
    # 结束 Trace
    trace.update(output={"answer": answer})
    
    # 刷新到 Langfuse
    langfuse.flush()
    
    return answer

# 使用示例
if __name__ == "__main__":
    result = rag_pipeline("什么是深度学习？", "user-001")
    print(result)
```

### 5.2 异步追踪

```python
"""
异步追踪示例

@author erik.zhou
"""
import asyncio
from langfuse import Langfuse

langfuse = Langfuse()

async def async_llm_call(prompt: str) -> str:
    """
    异步 LLM 调用
    
    Args:
        prompt: 提示词
        
    Returns:
        LLM 响应
    """
    # 创建 Trace
    trace = langfuse.trace(
        name="async-llm-call",
        input={"prompt": prompt}
    )
    
    # 创建 Generation
    generation = trace.generation(
        name="async-generation",
        model="gpt-3.5-turbo",
        input=[{"role": "user", "content": prompt}]
    )
    
    # 模拟异步调用
    await asyncio.sleep(1)
    response = f"异步响应: {prompt}"
    
    # 结束 Generation
    generation.end(
        output=response,
        usage={"total_tokens": 50}
    )
    
    # 刷新
    langfuse.flush()
    
    return response

# 运行异步函数
async def main():
    """主函数"""
    result = await async_llm_call("你好")
    print(result)

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 6. 与 LangChain 集成

### 6.1 CallbackHandler 集成

```python
"""
LangChain CallbackHandler 集成

@author erik.zhou
"""
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema.output_parser import StrOutputParser
from langfuse.callback import CallbackHandler

# 初始化 Langfuse CallbackHandler
langfuse_handler = CallbackHandler(
    public_key="pk-lf-xxx",
    secret_key="sk-lf-xxx",
    host="https://cloud.langfuse.com"
)

# 创建 LangChain 组件
llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0.7)

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个专业的 Python 教师"),
    ("user", "{question}")
])

chain = prompt | llm | StrOutputParser()

# 使用 Langfuse 追踪
result = chain.invoke(
    {"question": "什么是装饰器？"},
    config={"callbacks": [langfuse_handler]}
)

print(result)
```

### 6.2 LCEL 链追踪

```python
"""
LangChain Expression Language (LCEL) 链追踪

@author erik.zhou
"""
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain.prompts import ChatPromptTemplate
from langchain.schema.runnable import RunnablePassthrough
from langchain.vectorstores import FAISS
from langchain.schema.output_parser import StrOutputParser
from langfuse.callback import CallbackHandler

# 初始化组件
langfuse_handler = CallbackHandler()

embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_texts(
    ["Python 是一种编程语言", "机器学习是 AI 的分支"],
    embedding=embeddings
)
retriever = vectorstore.as_retriever()

llm = ChatOpenAI(model="gpt-3.5-turbo")

# 创建 RAG 链
template = """基于以下上下文回答问题:
{context}

问题: {question}
"""

prompt = ChatPromptTemplate.from_template(template)

rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

# 执行并追踪
result = rag_chain.invoke(
    "什么是 Python？",
    config={"callbacks": [langfuse_handler]}
)

print(result)
```

### 6.3 Agent 追踪

```python
"""
LangChain Agent 追踪

@author erik.zhou
"""
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain.tools import Tool
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder
from langfuse.callback import CallbackHandler

# 定义工具
def search_tool(query: str) -> str:
    """搜索工具"""
    return f"搜索结果: {query}"

def calculator_tool(expression: str) -> str:
    """计算器工具"""
    try:
        result = eval(expression)
        return f"计算结果: {result}"
    except Exception as e:
        return f"计算错误: {str(e)}"

tools = [
    Tool(
        name="Search",
        func=search_tool,
        description="用于搜索信息"
    ),
    Tool(
        name="Calculator",
        func=calculator_tool,
        description="用于数学计算"
    )
]

# 创建 Agent
llm = ChatOpenAI(model="gpt-4", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个有用的助手"),
    ("user", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad")
])

agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# 使用 Langfuse 追踪
langfuse_handler = CallbackHandler()

result = agent_executor.invoke(
    {"input": "计算 25 * 4 的结果"},
    config={"callbacks": [langfuse_handler]}
)

print(result)
```

---

## 7. 与 OpenAI 集成

### 7.1 OpenAI SDK 集成

```python
"""
OpenAI SDK 集成

@author erik.zhou
"""
from openai import OpenAI
from langfuse.openai import openai as langfuse_openai

# 使用 Langfuse 包装的 OpenAI 客户端
client = langfuse_openai.OpenAI(
    api_key="sk-xxx"
)

# 自动追踪 OpenAI 调用
response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "system", "content": "你是一个AI助手"},
        {"role": "user", "content": "介绍一下 Langfuse"}
    ],
    temperature=0.7,
    max_tokens=500,
    # Langfuse 特定参数
    name="chat-completion",
    metadata={"user_id": "user-123"},
    trace_id="custom-trace-id"  # 可选：自定义 trace ID
)

print(response.choices[0].message.content)
```

### 7.2 流式响应追踪

```python
"""
流式响应追踪

@author erik.zhou
"""
from openai import OpenAI
from langfuse.openai import openai as langfuse_openai

client = langfuse_openai.OpenAI()

# 流式调用
stream = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": "写一首关于 AI 的诗"}
    ],
    stream=True,
    name="streaming-chat",
    metadata={"type": "poetry"}
)

# 处理流式响应
full_response = ""
for chunk in stream:
    if chunk.choices[0].delta.content:
        content = chunk.choices[0].delta.content
        full_response += content
        print(content, end="", flush=True)

print(f"\n\n完整响应: {full_response}")
```

### 7.3 函数调用追踪

```python
"""
OpenAI 函数调用追踪

@author erik.zhou
"""
import json
from openai import OpenAI
from langfuse.openai import openai as langfuse_openai

client = langfuse_openai.OpenAI()

# 定义函数
functions = [
    {
        "name": "get_weather",
        "description": "获取指定城市的天气",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "城市名称"
                }
            },
            "required": ["city"]
        }
    }
]

# 调用 OpenAI
response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[
        {"role": "user", "content": "北京今天天气怎么样？"}
    ],
    functions=functions,
    function_call="auto",
    name="function-calling",
    metadata={"feature": "weather-query"}
)

# 处理函数调用
message = response.choices[0].message

if message.function_call:
    function_name = message.function_call.name
    function_args = json.loads(message.function_call.arguments)
    
    print(f"调用函数: {function_name}")
    print(f"参数: {function_args}")
    
    # 模拟函数执行
    if function_name == "get_weather":
        weather_result = {
            "city": function_args["city"],
            "temperature": "25°C",
            "condition": "晴天"
        }
        
        # 将结果返回给 LLM
        second_response = client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "user", "content": "北京今天天气怎么样？"},
                message,
                {
                    "role": "function",
                    "name": function_name,
                    "content": json.dumps(weather_result)
                }
            ],
            name="function-result",
            trace_id=response.id  # 关联到同一个 trace
        )
        
        print(f"\n最终回答: {second_response.choices[0].message.content}")
```

---

## 8. Prompt 管理

### 8.1 创建和使用 Prompt

```python
"""
Prompt 管理

@author erik.zhou
"""
from langfuse import Langfuse

langfuse = Langfuse()

# 方式 1：在 Langfuse UI 中创建 Prompt
# 1. 登录 Langfuse
# 2. 进入 Prompts 页面
# 3. 创建新 Prompt
# 4. 设置名称、版本、内容

# 方式 2：通过 API 获取 Prompt
prompt = langfuse.get_prompt("chat-assistant")

print(f"Prompt 名称: {prompt.name}")
print(f"Prompt 版本: {prompt.version}")
print(f"Prompt 内容: {prompt.prompt}")

# 使用 Prompt
messages = prompt.compile(
    user_name="张三",
    question="什么是机器学习？"
)

print(f"编译后的消息: {messages}")
```

### 8.2 Prompt 版本管理

```python
"""
Prompt 版本管理

@author erik.zhou
"""
from langfuse import Langfuse

langfuse = Langfuse()

# 获取特定版本的 Prompt
prompt_v1 = langfuse.get_prompt("chat-assistant", version=1)
prompt_v2 = langfuse.get_prompt("chat-assistant", version=2)

# 获取最新版本（生产环境）
prompt_latest = langfuse.get_prompt("chat-assistant", label="production")

print(f"V1 内容: {prompt_v1.prompt}")
print(f"V2 内容: {prompt_v2.prompt}")
print(f"生产版本: {prompt_latest.prompt}")
```

### 8.3 Prompt A/B 测试

```python
"""
Prompt A/B 测试

@author erik.zhou
"""
import random
from langfuse import Langfuse
from openai import OpenAI

langfuse = Langfuse()
openai_client = OpenAI()

def ab_test_prompts(user_query: str, user_id: str):
    """
    A/B 测试不同的 Prompt
    
    Args:
        user_query: 用户查询
        user_id: 用户ID
    """
    # 随机选择 Prompt 版本
    variant = random.choice(["A", "B"])
    
    if variant == "A":
        prompt = langfuse.get_prompt("chat-assistant", label="variant-a")
    else:
        prompt = langfuse.get_prompt("chat-assistant", label="variant-b")
    
    # 创建 Trace
    trace = langfuse.trace(
        name="ab-test",
        user_id=user_id,
        metadata={
            "variant": variant,
            "prompt_version": prompt.version
        }
    )
    
    # 编译 Prompt
    messages = prompt.compile(question=user_query)
    
    # 调用 LLM
    generation = trace.generation(
        name="llm-call",
        model="gpt-3.5-turbo",
        input=messages,
        prompt=prompt  # 关联 Prompt
    )
    
    response = openai_client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=messages
    )
    
    answer = response.choices[0].message.content
    
    generation.end(
        output=answer,
        usage={
            "prompt_tokens": response.usage.prompt_tokens,
            "completion_tokens": response.usage.completion_tokens,
            "total_tokens": response.usage.total_tokens
        }
    )
    
    langfuse.flush()
    
    return answer, variant

# 测试
if __name__ == "__main__":
    result, variant = ab_test_prompts("什么是深度学习？", "user-001")
    print(f"使用变体: {variant}")
    print(f"回答: {result}")
```


---

## 9. 评估与分析

### 9.1 用户反馈收集

```python
"""
收集用户反馈

@author erik.zhou
"""
from langfuse import Langfuse

langfuse = Langfuse()

def collect_user_feedback(trace_id: str, score: float, comment: str = None):
    """
    收集用户反馈
    
    Args:
        trace_id: Trace ID
        score: 评分（0-1）
        comment: 评论
    """
    langfuse.score(
        trace_id=trace_id,
        name="user-feedback",
        value=score,
        comment=comment
    )
    
    langfuse.flush()
    
    print(f"反馈已记录: Trace {trace_id}, 评分 {score}")

# 使用示例
if __name__ == "__main__":
    # 假设这是之前生成的 trace_id
    collect_user_feedback(
        trace_id="trace-abc-123",
        score=0.9,
        comment="回答很有帮助"
    )
```

### 9.2 自动化评估

```python
"""
自动化评估

@author erik.zhou
"""
from langfuse import Langfuse
from openai import OpenAI

langfuse = Langfuse()
openai_client = OpenAI()

def evaluate_response(trace_id: str, question: str, answer: str):
    """
    使用 LLM 评估响应质量
    
    Args:
        trace_id: Trace ID
        question: 问题
        answer: 回答
    """
    # 使用 LLM 作为评判者
    evaluation_prompt = f"""
    评估以下回答的质量（0-1分）：
    
    问题: {question}
    回答: {answer}
    
    评估标准:
    - 准确性
    - 完整性
    - 清晰度
    
    只返回一个 0-1 之间的分数。
    """
    
    response = openai_client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "user", "content": evaluation_prompt}
        ],
        temperature=0
    )
    
    try:
        score = float(response.choices[0].message.content.strip())
        
        # 记录评估结果
        langfuse.score(
            trace_id=trace_id,
            name="llm-evaluation",
            value=score,
            comment="自动评估"
        )
        
        langfuse.flush()
        
        return score
    except ValueError:
        print("评估失败：无法解析分数")
        return None

# 使用示例
if __name__ == "__main__":
    score = evaluate_response(
        trace_id="trace-xyz-456",
        question="什么是机器学习？",
        answer="机器学习是人工智能的一个分支..."
    )
    print(f"评估分数: {score}")
```

### 9.3 数据集管理

```python
"""
数据集管理

@author erik.zhou
"""
from langfuse import Langfuse

langfuse = Langfuse()

# 创建数据集
dataset = langfuse.create_dataset(
    name="qa-evaluation-set",
    description="问答系统评估数据集"
)

# 添加数据项
dataset.create_item(
    input={"question": "什么是 Python？"},
    expected_output="Python 是一种高级编程语言..."
)

dataset.create_item(
    input={"question": "什么是机器学习？"},
    expected_output="机器学习是人工智能的一个分支..."
)

# 获取数据集
dataset = langfuse.get_dataset("qa-evaluation-set")

# 遍历数据集
for item in dataset.items:
    print(f"问题: {item.input}")
    print(f"期望输出: {item.expected_output}")
    print("---")

langfuse.flush()
```

### 9.4 批量评估

```python
"""
批量评估

@author erik.zhou
"""
from langfuse import Langfuse
from openai import OpenAI
from typing import Dict, Any

langfuse = Langfuse()
openai_client = OpenAI()

def run_evaluation(dataset_name: str) -> Dict[str, Any]:
    """
    在数据集上运行评估
    
    Args:
        dataset_name: 数据集名称
        
    Returns:
        评估结果
    """
    # 获取数据集
    dataset = langfuse.get_dataset(dataset_name)
    
    results = []
    
    for item in dataset.items:
        question = item.input["question"]
        expected_output = item.expected_output
        
        # 创建 Trace
        trace = langfuse.trace(
            name="evaluation-run",
            metadata={
                "dataset": dataset_name,
                "item_id": item.id
            }
        )
        
        # 生成回答
        generation = trace.generation(
            name="answer-generation",
            model="gpt-3.5-turbo",
            input=[{"role": "user", "content": question}]
        )
        
        response = openai_client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "user", "content": question}]
        )
        
        actual_output = response.choices[0].message.content
        
        generation.end(
            output=actual_output,
            usage={
                "total_tokens": response.usage.total_tokens
            }
        )
        
        # 计算相似度（简化版）
        similarity = calculate_similarity(expected_output, actual_output)
        
        # 记录评分
        langfuse.score(
            trace_id=trace.id,
            name="similarity",
            value=similarity
        )
        
        results.append({
            "question": question,
            "expected": expected_output,
            "actual": actual_output,
            "similarity": similarity
        })
    
    langfuse.flush()
    
    # 计算平均分
    avg_score = sum(r["similarity"] for r in results) / len(results)
    
    return {
        "dataset": dataset_name,
        "num_items": len(results),
        "average_score": avg_score,
        "results": results
    }

def calculate_similarity(text1: str, text2: str) -> float:
    """
    计算文本相似度（简化版）
    
    Args:
        text1: 文本1
        text2: 文本2
        
    Returns:
        相似度分数
    """
    # 这里使用简单的方法，实际应用中可以使用更复杂的算法
    words1 = set(text1.lower().split())
    words2 = set(text2.lower().split())
    
    intersection = words1.intersection(words2)
    union = words1.union(words2)
    
    if len(union) == 0:
        return 0.0
    
    return len(intersection) / len(union)

# 使用示例
if __name__ == "__main__":
    results = run_evaluation("qa-evaluation-set")
    print(f"平均分数: {results['average_score']:.2f}")
```

---

## 10. 最佳实践

### 10.1 错误处理

```python
"""
错误处理最佳实践

@author erik.zhou
"""
from langfuse import Langfuse
from openai import OpenAI
import traceback

langfuse = Langfuse()
openai_client = OpenAI()

def safe_llm_call(prompt: str, user_id: str) -> str:
    """
    安全的 LLM 调用，包含错误处理
    
    Args:
        prompt: 提示词
        user_id: 用户ID
        
    Returns:
        LLM 响应
    """
    trace = None
    generation = None
    
    try:
        # 创建 Trace
        trace = langfuse.trace(
            name="safe-llm-call",
            user_id=user_id,
            input={"prompt": prompt}
        )
        
        # 创建 Generation
        generation = trace.generation(
            name="openai-call",
            model="gpt-3.5-turbo",
            input=[{"role": "user", "content": prompt}]
        )
        
        # 调用 OpenAI
        response = openai_client.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[{"role": "user", "content": prompt}],
            timeout=30
        )
        
        answer = response.choices[0].message.content
        
        # 记录成功
        generation.end(
            output=answer,
            usage={
                "total_tokens": response.usage.total_tokens
            },
            status_message="success"
        )
        
        return answer
        
    except Exception as e:
        error_message = str(e)
        error_traceback = traceback.format_exc()
        
        # 记录错误
        if generation:
            generation.end(
                status_message=f"error: {error_message}",
                metadata={
                    "error_type": type(e).__name__,
                    "traceback": error_traceback
                }
            )
        
        if trace:
            trace.update(
                output={"error": error_message},
                metadata={"status": "failed"}
            )
        
        # 记录错误事件
        if trace:
            trace.event(
                name="error",
                input={"error": error_message},
                metadata={"traceback": error_traceback}
            )
        
        raise
    
    finally:
        # 确保数据被发送
        langfuse.flush()

# 使用示例
if __name__ == "__main__":
    try:
        result = safe_llm_call("你好", "user-001")
        print(result)
    except Exception as e:
        print(f"调用失败: {e}")
```

### 10.2 性能优化

```python
"""
性能优化最佳实践

@author erik.zhou
"""
from langfuse import Langfuse
from langfuse.decorators import observe
import time

# 使用单例模式
_langfuse_instance = None

def get_langfuse() -> Langfuse:
    """
    获取 Langfuse 单例
    
    Returns:
        Langfuse 实例
    """
    global _langfuse_instance
    if _langfuse_instance is None:
        _langfuse_instance = Langfuse(
            # 启用批量发送
            flush_at=10,  # 累积 10 条记录后发送
            flush_interval=1.0  # 或每 1 秒发送一次
        )
    return _langfuse_instance

@observe()
def optimized_function(data: str) -> str:
    """
    优化的函数
    
    Args:
        data: 输入数据
        
    Returns:
        处理结果
    """
    # 使用装饰器自动追踪，减少手动代码
    time.sleep(0.1)
    return f"处理结果: {data}"

# 批量操作
def batch_process(items: list):
    """
    批量处理
    
    Args:
        items: 待处理项目列表
    """
    langfuse = get_langfuse()
    
    # 创建一个父 Trace
    parent_trace = langfuse.trace(
        name="batch-process",
        metadata={"batch_size": len(items)}
    )
    
    for i, item in enumerate(items):
        # 为每个项目创建子 Span
        span = parent_trace.span(
            name=f"process-item-{i}",
            input={"item": item}
        )
        
        result = optimized_function(item)
        
        span.end(output={"result": result})
    
    # 批量刷新
    langfuse.flush()

# 使用示例
if __name__ == "__main__":
    items = ["item1", "item2", "item3"]
    batch_process(items)
```

### 10.3 生产环境配置

```python
"""
生产环境配置最佳实践

@author erik.zhou
"""
import os
from langfuse import Langfuse
from typing import Optional

class LangfuseConfig:
    """Langfuse 配置类"""
    
    def __init__(self):
        """初始化配置"""
        self.public_key = os.getenv("LANGFUSE_PUBLIC_KEY")
        self.secret_key = os.getenv("LANGFUSE_SECRET_KEY")
        self.host = os.getenv("LANGFUSE_HOST", "https://cloud.langfuse.com")
        self.enabled = os.getenv("LANGFUSE_ENABLED", "true").lower() == "true"
        self.environment = os.getenv("ENVIRONMENT", "development")
    
    def validate(self) -> bool:
        """
        验证配置
        
        Returns:
            配置是否有效
        """
        if not self.enabled:
            return True
        
        if not self.public_key or not self.secret_key:
            print("警告: Langfuse 密钥未配置")
            return False
        
        return True

class LangfuseManager:
    """Langfuse 管理器"""
    
    _instance: Optional[Langfuse] = None
    _config: Optional[LangfuseConfig] = None
    
    @classmethod
    def initialize(cls, config: Optional[LangfuseConfig] = None):
        """
        初始化 Langfuse
        
        Args:
            config: 配置对象
        """
        if config is None:
            config = LangfuseConfig()
        
        cls._config = config
        
        if not config.validate():
            print("Langfuse 配置无效，追踪功能已禁用")
            return
        
        if config.enabled:
            cls._instance = Langfuse(
                public_key=config.public_key,
                secret_key=config.secret_key,
                host=config.host,
                release=config.environment,
                flush_at=20,
                flush_interval=2.0
            )
            print(f"Langfuse 已初始化 (环境: {config.environment})")
        else:
            print("Langfuse 追踪已禁用")
    
    @classmethod
    def get_instance(cls) -> Optional[Langfuse]:
        """
        获取 Langfuse 实例
        
        Returns:
            Langfuse 实例或 None
        """
        if cls._instance is None and cls._config and cls._config.enabled:
            cls.initialize()
        
        return cls._instance
    
    @classmethod
    def is_enabled(cls) -> bool:
        """
        检查是否启用
        
        Returns:
            是否启用
        """
        return cls._instance is not None

# 使用示例
if __name__ == "__main__":
    # 初始化
    LangfuseManager.initialize()
    
    # 使用
    if LangfuseManager.is_enabled():
        langfuse = LangfuseManager.get_instance()
        trace = langfuse.trace(name="production-trace")
        print("Trace 已创建")
    else:
        print("Langfuse 未启用，跳过追踪")
```

### 10.4 隐私和安全

```python
"""
隐私和安全最佳实践

@author erik.zhou
"""
from langfuse import Langfuse
import re
from typing import Any, Dict

langfuse = Langfuse()

def sanitize_data(data: Any) -> Any:
    """
    清理敏感数据
    
    Args:
        data: 原始数据
        
    Returns:
        清理后的数据
    """
    if isinstance(data, str):
        # 移除邮箱
        data = re.sub(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', '[EMAIL]', data)
        
        # 移除电话号码
        data = re.sub(r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b', '[PHONE]', data)
        
        # 移除身份证号
        data = re.sub(r'\b\d{17}[\dXx]\b', '[ID_CARD]', data)
        
    elif isinstance(data, dict):
        return {k: sanitize_data(v) for k, v in data.items()}
    
    elif isinstance(data, list):
        return [sanitize_data(item) for item in data]
    
    return data

def safe_trace(name: str, user_id: str, input_data: Dict, output_data: Dict):
    """
    安全的追踪，自动清理敏感数据
    
    Args:
        name: Trace 名称
        user_id: 用户ID（已脱敏）
        input_data: 输入数据
        output_data: 输出数据
    """
    # 清理数据
    sanitized_input = sanitize_data(input_data)
    sanitized_output = sanitize_data(output_data)
    
    # 创建 Trace
    trace = langfuse.trace(
        name=name,
        user_id=user_id,  # 使用脱敏的用户ID
        input=sanitized_input,
        output=sanitized_output,
        metadata={
            "privacy_level": "sanitized"
        }
    )
    
    langfuse.flush()
    
    return trace

# 使用示例
if __name__ == "__main__":
    input_data = {
        "query": "我的邮箱是 user@example.com，电话是 123-456-7890"
    }
    
    output_data = {
        "response": "已收到您的信息"
    }
    
    safe_trace(
        name="privacy-safe-trace",
        user_id="user-hash-abc123",  # 使用哈希后的用户ID
        input_data=input_data,
        output_data=output_data
    )
    
    print("安全追踪已完成")
```

### 10.5 监控和告警

```python
"""
监控和告警最佳实践

@author erik.zhou
"""
from langfuse import Langfuse
from datetime import datetime, timedelta
from typing import Dict, List

langfuse = Langfuse()

class PerformanceMonitor:
    """性能监控器"""
    
    def __init__(self):
        """初始化监控器"""
        self.metrics = {
            "total_calls": 0,
            "failed_calls": 0,
            "total_tokens": 0,
            "total_cost": 0.0,
            "avg_latency": 0.0
        }
    
    def record_call(
        self,
        success: bool,
        tokens: int,
        cost: float,
        latency: float
    ):
        """
        记录调用
        
        Args:
            success: 是否成功
            tokens: Token 数量
            cost: 成本
            latency: 延迟（秒）
        """
        self.metrics["total_calls"] += 1
        
        if not success:
            self.metrics["failed_calls"] += 1
        
        self.metrics["total_tokens"] += tokens
        self.metrics["total_cost"] += cost
        
        # 更新平均延迟
        n = self.metrics["total_calls"]
        self.metrics["avg_latency"] = (
            (self.metrics["avg_latency"] * (n - 1) + latency) / n
        )
    
    def check_alerts(self) -> List[str]:
        """
        检查告警
        
        Returns:
            告警列表
        """
        alerts = []
        
        # 检查失败率
        if self.metrics["total_calls"] > 0:
            failure_rate = self.metrics["failed_calls"] / self.metrics["total_calls"]
            if failure_rate > 0.1:  # 失败率超过 10%
                alerts.append(f"高失败率: {failure_rate:.2%}")
        
        # 检查平均延迟
        if self.metrics["avg_latency"] > 5.0:  # 平均延迟超过 5 秒
            alerts.append(f"高延迟: {self.metrics['avg_latency']:.2f}秒")
        
        # 检查成本
        if self.metrics["total_cost"] > 100.0:  # 成本超过 $100
            alerts.append(f"高成本: ${self.metrics['total_cost']:.2f}")
        
        return alerts
    
    def get_report(self) -> Dict:
        """
        获取报告
        
        Returns:
            监控报告
        """
        alerts = self.check_alerts()
        
        return {
            "timestamp": datetime.now().isoformat(),
            "metrics": self.metrics,
            "alerts": alerts,
            "status": "warning" if alerts else "healthy"
        }

# 使用示例
if __name__ == "__main__":
    monitor = PerformanceMonitor()
    
    # 模拟记录
    monitor.record_call(success=True, tokens=100, cost=0.002, latency=1.5)
    monitor.record_call(success=True, tokens=150, cost=0.003, latency=2.0)
    monitor.record_call(success=False, tokens=0, cost=0.0, latency=0.0)
    
    # 获取报告
    report = monitor.get_report()
    print(f"监控报告: {report}")
    
    # 检查告警
    if report["alerts"]:
        print(f"告警: {', '.join(report['alerts'])}")
```

---

## 11. 总结

Langfuse 是一个强大的 LLM 应用可观测性平台，本教程涵盖了：

1. **基础使用**：Trace、Span、Generation、Event 等核心概念
2. **集成方式**：LangChain、OpenAI SDK 等多种集成方式
3. **Prompt 管理**：版本管理、A/B 测试
4. **评估分析**：用户反馈、自动化评估、数据集管理
5. **最佳实践**：错误处理、性能优化、隐私安全、监控告警

通过 Langfuse，你可以：
- 完整追踪 LLM 应用的每次调用
- 优化 Prompt 和模型参数
- 监控成本和性能
- 收集用户反馈并持续改进
- 确保生产环境的稳定性

建议在开发阶段就集成 Langfuse，这样可以更好地理解和优化你的 LLM 应用。
