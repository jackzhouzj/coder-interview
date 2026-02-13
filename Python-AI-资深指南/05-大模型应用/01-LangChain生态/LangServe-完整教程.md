# LangServe 完整教程

> @author erik.zhou
> 
> 更新日期：2026-02-13

## 📋 课程概述

LangServe 是 LangChain 生态系统中用于将 LangChain 应用部署为 REST API 的框架。它基于 FastAPI 构建，提供了简单易用的方式来将你的 Chain 和 Runnable 对象转换为生产级的 API 服务。

### 学习目标
- 掌握 LangServe 的基本使用
- 理解如何部署 LangChain 应用
- 学会处理流式响应和批处理
- 掌握生产环境部署最佳实践

### 前置知识
- Python 基础
- LangChain 基础
- FastAPI 基础（推荐）
- REST API 概念

---

## 1. LangServe 基础

### 1.1 安装和配置

```python
# 安装 LangServe
pip install "langserve[all]"

# 或者只安装服务端
pip install "langserve[server]"

# 或者只安装客户端
pip install "langserve[client]"
```

### 1.2 第一个 LangServe 应用

```python
#!/usr/bin/env python
"""
最简单的 LangServe 应用示例
"""
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langserve import add_routes

# 创建 FastAPI 应用
app = FastAPI(
    title="LangChain Server",
    version="1.0",
    description="A simple API server using LangChain's Runnable interfaces",
)

# 创建一个简单的 Chain
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template("Tell me a joke about {topic}")
chain = prompt | model

# 添加路由
add_routes(
    app,
    chain,
    path="/joke",
)

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="localhost", port=8000)
```

### 1.3 客户端调用

```python
from langserve import RemoteRunnable

# 创建远程 Runnable
remote_chain = RemoteRunnable("http://localhost:8000/joke/")

# 同步调用
response = remote_chain.invoke({"topic": "cats"})
print(response)

# 批量调用
responses = remote_chain.batch([
    {"topic": "cats"},
    {"topic": "dogs"}
])

# 流式调用
for chunk in remote_chain.stream({"topic": "cats"}):
    print(chunk, end="", flush=True)
```

---

## 2. 高级功能

### 2.1 多个端点

```python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.schema.output_parser import StrOutputParser
from langserve import add_routes

app = FastAPI(
    title="Multi-Endpoint Server",
    version="1.0",
)

# 创建多个 Chain
model = ChatOpenAI()

# 笑话生成器
joke_chain = (
    ChatPromptTemplate.from_template("Tell me a joke about {topic}")
    | model
    | StrOutputParser()
)

# 诗歌生成器
poem_chain = (
    ChatPromptTemplate.from_template("Write a poem about {topic}")
    | model
    | StrOutputParser()
)

# 翻译器
translate_chain = (
    ChatPromptTemplate.from_template(
        "Translate the following text to {language}: {text}"
    )
    | model
    | StrOutputParser()
)

# 添加多个路由
add_routes(app, joke_chain, path="/joke")
add_routes(app, poem_chain, path="/poem")
add_routes(app, translate_chain, path="/translate")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="localhost", port=8000)
```

### 2.2 流式响应

```python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.schema.output_parser import StrOutputParser
from langserve import add_routes

app = FastAPI()

# 创建支持流式输出的 Chain
model = ChatOpenAI(streaming=True)
prompt = ChatPromptTemplate.from_template("Write a story about {topic}")
chain = prompt | model | StrOutputParser()

# 添加路由，启用流式响应
add_routes(
    app,
    chain,
    path="/story",
    enable_feedback_endpoint=True,
    enable_public_trace_link_endpoint=True,
)

# 客户端流式调用
from langserve import RemoteRunnable

remote_chain = RemoteRunnable("http://localhost:8000/story/")

# 流式输出
for chunk in remote_chain.stream({"topic": "dragons"}):
    print(chunk, end="", flush=True)
```

### 2.3 批处理

```python
from langserve import RemoteRunnable

remote_chain = RemoteRunnable("http://localhost:8000/joke/")

# 批量处理多个请求
inputs = [
    {"topic": "cats"},
    {"topic": "dogs"},
    {"topic": "birds"},
]

# 同步批处理
responses = remote_chain.batch(inputs)
for response in responses:
    print(response)

# 异步批处理
import asyncio

async def async_batch():
    responses = await remote_chain.abatch(inputs)
    for response in responses:
        print(response)

asyncio.run(async_batch())
```

---

## 3. 配置和定制

### 3.1 自定义配置

```python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langserve import add_routes
from langchain.schema.runnable import RunnableConfig

app = FastAPI()

model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
chain = prompt | model

# 添加路由时配置
add_routes(
    app,
    chain,
    path="/chat",
    # 启用配置端点
    config_keys=["configurable"],
    # 启用反馈端点
    enable_feedback_endpoint=True,
    # 启用追踪链接
    enable_public_trace_link_endpoint=True,
    # 启用 playground
    playground_type="default",
)

# 客户端使用配置
from langserve import RemoteRunnable

remote_chain = RemoteRunnable("http://localhost:8000/chat/")

# 传递配置
config = RunnableConfig(
    configurable={
        "model_name": "gpt-4",
        "temperature": 0.7,
    }
)

response = remote_chain.invoke(
    {"topic": "AI"},
    config=config
)
```

### 3.2 输入输出验证

```python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langserve import add_routes
from pydantic import BaseModel, Field
from typing import List

app = FastAPI()

# 定义输入模型
class JokeRequest(BaseModel):
    topic: str = Field(..., description="The topic of the joke")
    style: str = Field(default="funny", description="The style of the joke")

# 定义输出模型
class JokeResponse(BaseModel):
    joke: str = Field(..., description="The generated joke")
    topic: str = Field(..., description="The topic of the joke")

# 创建 Chain
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template(
    "Tell me a {style} joke about {topic}"
)

def format_output(response):
    return JokeResponse(
        joke=response.content,
        topic=response.additional_kwargs.get("topic", "")
    )

chain = prompt | model

# 添加路由时指定输入输出类型
add_routes(
    app,
    chain,
    path="/joke",
    input_type=JokeRequest,
    output_type=JokeResponse,
)
```

---

## 4. 生产环境部署

### 4.1 添加认证

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langserve import add_routes
import os

app = FastAPI()

# 配置认证
security = HTTPBearer()
API_KEY = os.getenv("API_KEY", "your-secret-key")

def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    """验证 API Key"""
    if credentials.credentials != API_KEY:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
        )
    return credentials.credentials

# 创建 Chain
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
chain = prompt | model

# 添加需要认证的路由
add_routes(
    app,
    chain,
    path="/chat",
    dependencies=[Depends(verify_token)],
)
```

### 4.2 添加限流

```python
from fastapi import FastAPI, Request
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langserve import add_routes

# 创建限流器
limiter = Limiter(key_func=get_remote_address)
app = FastAPI()
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

# 创建 Chain
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
chain = prompt | model

# 添加路由
add_routes(app, chain, path="/chat")

# 添加限流装饰器
@app.get("/limited")
@limiter.limit("5/minute")
async def limited_endpoint(request: Request):
    return {"message": "This endpoint is rate limited"}
```

### 4.3 添加日志和监控

```python
from fastapi import FastAPI, Request
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langserve import add_routes
import logging
import time

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI()

# 添加中间件记录请求
@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    
    # 记录请求
    logger.info(f"Request: {request.method} {request.url}")
    
    # 处理请求
    response = await call_next(request)
    
    # 记录响应时间
    process_time = time.time() - start_time
    logger.info(f"Response time: {process_time:.2f}s")
    
    return response

# 创建 Chain
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
chain = prompt | model

# 添加路由
add_routes(app, chain, path="/chat")
```

### 4.4 Docker 部署

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 暴露端口
EXPOSE 8000

# 启动应用
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  langserve:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - API_KEY=${API_KEY}
    restart: unless-stopped
```

---

## 5. 实战案例

### 5.1 RAG 问答系统

```python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnablePassthrough
from langserve import add_routes

app = FastAPI(
    title="RAG QA System",
    description="Question answering system using RAG",
)

# 加载向量数据库
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.load_local("./vectorstore", embeddings)
retriever = vectorstore.as_retriever()

# 创建 RAG Chain
template = """Answer the question based only on the following context:
{context}

Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)
model = ChatOpenAI()

rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 添加路由
add_routes(
    app,
    rag_chain,
    path="/qa",
    enable_feedback_endpoint=True,
)

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="localhost", port=8000)
```

### 5.2 多模型对话系统

```python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI, ChatAnthropic
from langchain.schema.output_parser import StrOutputParser
from langserve import add_routes
from typing import Literal

app = FastAPI(
    title="Multi-Model Chat System",
)

# 创建不同的模型
gpt4_model = ChatOpenAI(model="gpt-4")
gpt35_model = ChatOpenAI(model="gpt-3.5-turbo")
claude_model = ChatAnthropic(model="claude-3-opus-20240229")

# 创建提示模板
prompt = ChatPromptTemplate.from_template("Answer the question: {question}")

# 创建不同的 Chain
gpt4_chain = prompt | gpt4_model | StrOutputParser()
gpt35_chain = prompt | gpt35_model | StrOutputParser()
claude_chain = prompt | claude_model | StrOutputParser()

# 添加路由
add_routes(app, gpt4_chain, path="/gpt4")
add_routes(app, gpt35_chain, path="/gpt35")
add_routes(app, claude_chain, path="/claude")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="localhost", port=8000)
```

---

## 6. 最佳实践

### 6.1 错误处理

```python
from fastapi import FastAPI, HTTPException
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langserve import add_routes
import logging

logger = logging.getLogger(__name__)
app = FastAPI()

# 创建 Chain
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
chain = prompt | model

# 添加全局异常处理
@app.exception_handler(Exception)
async def global_exception_handler(request, exc):
    logger.error(f"Global exception: {exc}")
    return HTTPException(
        status_code=500,
        detail="Internal server error"
    )

# 添加路由
add_routes(app, chain, path="/chat")
```

### 6.2 性能优化

```python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.cache import InMemoryCache
from langchain.globals import set_llm_cache
from langserve import add_routes

# 启用缓存
set_llm_cache(InMemoryCache())

app = FastAPI()

# 创建 Chain
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
chain = prompt | model

# 添加路由
add_routes(app, chain, path="/chat")

# 配置 uvicorn 性能参数
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        app,
        host="0.0.0.0",
        port=8000,
        workers=4,  # 多进程
        limit_concurrency=100,  # 并发限制
        timeout_keep_alive=5,
    )
```

### 6.3 健康检查

```python
from fastapi import FastAPI
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langserve import add_routes

app = FastAPI()

# 健康检查端点
@app.get("/health")
async def health_check():
    return {"status": "healthy"}

# 就绪检查端点
@app.get("/ready")
async def readiness_check():
    try:
        # 检查模型是否可用
        model = ChatOpenAI()
        model.invoke("test")
        return {"status": "ready"}
    except Exception as e:
        return {"status": "not ready", "error": str(e)}

# 创建 Chain
model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
chain = prompt | model

# 添加路由
add_routes(app, chain, path="/chat")
```

---

## 7. 总结

### 核心要点
1. LangServe 基于 FastAPI，易于部署和扩展
2. 支持流式响应、批处理和异步调用
3. 提供完整的客户端 SDK
4. 适合生产环境部署

### 学习建议
1. 先掌握 FastAPI 基础
2. 理解 LangChain Runnable 接口
3. 实践流式响应和批处理
4. 学习生产环境部署最佳实践

### 下一步
- 学习 LangSmith 进行监控和调试
- 探索 LangChain Hub 获取更多模板
- 实践复杂的 RAG 应用部署
- 学习 Kubernetes 部署

---

**@author erik.zhou**
