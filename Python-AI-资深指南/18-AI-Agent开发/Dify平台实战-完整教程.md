# Dify 平台实战完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

Dify 是一个开源的 LLMOps 平台（GitHub 80K+ Stars），提供可视化工作流编排、RAG 管道、Agent 框架和模型管理。它让开发者和非技术团队都能快速构建、部署和运营生产级 AI 应用。

### 版本信息
- **Dify 版本**：1.x+
- **部署方式**：Docker 自托管 / Dify Cloud
- **重要程度**：⭐⭐⭐⭐（重要）
- **难度等级**：⭐⭐（较易）
- **预计学习时间**：10-15 小时

### 学习目标
- 理解 Dify 的核心架构和功能模块
- 掌握可视化工作流编排
- 学会构建 RAG 知识库应用
- 能够创建 Agent 应用
- 掌握 API 集成和生产部署

### 前置知识
- LLM 基础概念
- Docker 基础（自托管时需要）
- REST API 基础

---

## 1. Dify 简介

### 1.1 核心功能

```
Dify 功能架构
├── 应用类型
│   ├── 聊天助手（Chatbot）
│   ├── 文本生成（Completion）
│   ├── Agent（智能体）
│   └── 工作流（Workflow）
├── RAG 引擎
│   ├── 知识库管理
│   ├── 文档导入（PDF/Word/Markdown/网页）
│   ├── 自动切分和向量化
│   └── 混合检索（向量+全文）
├── 模型管理
│   ├── OpenAI / Anthropic / Google
│   ├── 开源模型（Llama/Qwen/DeepSeek）
│   └── 本地模型（Ollama/vLLM）
├── 工作流编排
│   ├── 可视化拖拽画布
│   ├── 条件分支 / 循环
│   ├── 代码节点（Python/JavaScript）
│   └── HTTP 请求节点
└── 运营工具
    ├── 日志和追踪
    ├── 标注和反馈
    ├── 数据分析
    └── API 发布
```

### 1.2 Dify vs 纯代码开发

| 维度 | Dify | 纯代码（LangChain等） |
|------|------|----------------------|
| 开发速度 | 快（可视化拖拽） | 慢（需要编码） |
| 灵活性 | 中等 | 极高 |
| 学习门槛 | 低 | 高 |
| 适合人群 | 全栈/产品/运营 | 开发者 |
| RAG 管理 | 内置完整方案 | 需要自建 |
| 模型切换 | 界面一键切换 | 需要改代码 |
| 生产运维 | 内置监控 | 需要自建 |

---

## 2. 环境搭建

### 2.1 Docker 部署（推荐）

```bash
# 克隆仓库
git clone https://github.com/langgenius/dify.git
cd dify/docker

# 复制环境变量
cp .env.example .env

# 启动服务
docker compose up -d

# 访问
# Web 界面：http://localhost/install
# API 地址：http://localhost/v1
```

### 2.2 初始配置

```python
"""
Dify 初始配置步骤：

1. 访问 http://localhost/install
2. 创建管理员账号
3. 进入"设置" → "模型供应商"
4. 配置 LLM 提供商：
   - OpenAI：填入 API Key
   - Anthropic：填入 API Key
   - 本地模型：配置 Ollama 地址（http://host.docker.internal:11434）
5. 配置 Embedding 模型
"""
```

---

## 3. 构建聊天助手

### 3.1 创建应用

```python
"""
通过 Dify Web 界面创建聊天助手：

1. 点击"创建应用" → 选择"聊天助手"
2. 设置应用名称和描述
3. 配置系统提示词（Prompt）：

示例提示词：
---
你是一位专业的 Python 技术顾问。

规则：
1. 使用中文回答
2. 回答要专业且有深度
3. 必要时给出代码示例
4. 代码示例使用 Python 3.10+
---

4. 选择模型（如 gpt-4o）
5. 调整参数（Temperature、Max Tokens）
6. 点击"发布"
"""
```

### 3.2 通过 API 调用

```python
import requests

# 🔥 Dify API 调用
DIFY_API_URL = "http://localhost/v1"
API_KEY = "app-xxxxxxxxxxxxxxxx"  # 应用 API Key

def chat_with_dify(message: str, conversation_id: str = None) -> dict:
    """调用 Dify 聊天 API"""
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    }
    
    payload = {
        "inputs": {},
        "query": message,
        "response_mode": "blocking",  # blocking / streaming
        "user": "user-001"
    }
    
    if conversation_id:
        payload["conversation_id"] = conversation_id
    
    response = requests.post(
        f"{DIFY_API_URL}/chat-messages",
        headers=headers,
        json=payload
    )
    
    return response.json()

# 使用
result = chat_with_dify("Python 的 GIL 是什么？")
print(result["answer"])
print(f"会话ID: {result['conversation_id']}")

# 多轮对话
result2 = chat_with_dify(
    "它对多线程有什么影响？",
    conversation_id=result["conversation_id"]
)
print(result2["answer"])
```

### 3.3 流式调用

```python
import requests
import json

def chat_streaming(message: str):
    """流式调用 Dify"""
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    }
    
    payload = {
        "inputs": {},
        "query": message,
        "response_mode": "streaming",
        "user": "user-001"
    }
    
    response = requests.post(
        f"{DIFY_API_URL}/chat-messages",
        headers=headers,
        json=payload,
        stream=True
    )
    
    for line in response.iter_lines():
        if line:
            line = line.decode("utf-8")
            if line.startswith("data: "):
                data = json.loads(line[6:])
                event = data.get("event")
                
                if event == "message":
                    print(data["answer"], end="", flush=True)
                elif event == "message_end":
                    print(f"\n\n[Token 使用: {data['metadata']['usage']}]")

chat_streaming("解释 Python 装饰器")
```

---

## 4. 构建 RAG 知识库应用

### 4.1 创建知识库

```python
"""
通过 Web 界面创建知识库：

1. 进入"知识库" → "创建知识库"
2. 上传文档（支持 PDF/Word/Markdown/TXT/CSV/网页）
3. 配置切分策略：
   - 自动切分（推荐）
   - 自定义切分（设置 chunk_size 和 overlap）
4. 选择 Embedding 模型
5. 选择索引方式：
   - 高质量模式（向量 + 全文混合检索）
   - 经济模式（仅关键词检索）
6. 等待索引完成
"""

# 🔥 通过 API 创建知识库
import requests

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# 创建知识库
response = requests.post(
    f"{DIFY_API_URL}/datasets",
    headers=headers,
    json={
        "name": "Python技术文档",
        "description": "Python 相关技术文档知识库"
    }
)
dataset_id = response.json()["id"]

# 上传文档
with open("python_guide.pdf", "rb") as f:
    response = requests.post(
        f"{DIFY_API_URL}/datasets/{dataset_id}/document/create_by_file",
        headers={"Authorization": f"Bearer {API_KEY}"},
        files={"file": f},
        data={
            "indexing_technique": "high_quality",
            "process_rule": json.dumps({
                "mode": "automatic"
            })
        }
    )
```

### 4.2 在应用中使用知识库

```python
"""
将知识库关联到聊天助手：

1. 编辑应用 → "上下文" 区域
2. 添加知识库
3. 配置检索参数：
   - Top K：返回文档数量（推荐 3-5）
   - Score 阈值：相似度阈值（推荐 0.5-0.7）
   - 检索模式：混合检索（推荐）
4. 配置提示词中引用知识库：

示例提示词：
---
你是一个技术文档助手。

请基于以下参考资料回答用户问题：
{{#context#}}

规则：
1. 只基于参考资料回答，不要编造
2. 如果资料中没有相关信息，明确告知用户
3. 引用来源文档名称
---
"""
```

---

## 5. 构建工作流

### 5.1 工作流节点类型

```python
"""
Dify 工作流节点类型：

1. 开始节点（Start）
   - 定义输入变量
   - 设置变量类型

2. LLM 节点
   - 调用大语言模型
   - 配置提示词和参数

3. 知识检索节点
   - 从知识库检索文档
   - 配置检索参数

4. 代码节点（Code）
   - 执行 Python / JavaScript 代码
   - 数据处理和转换

5. 条件分支节点（IF/ELSE）
   - 基于条件路由

6. HTTP 请求节点
   - 调用外部 API

7. 模板转换节点
   - Jinja2 模板渲染

8. 变量聚合节点
   - 合并多个分支的变量

9. Agent 节点 🔥
   - 让 LLM 自主推理和使用工具
   - 支持自定义 Agent 策略

10. 结束节点（End）
    - 定义输出变量
"""
```

### 5.2 工作流示例：智能客服

```python
"""
智能客服工作流设计：

开始 → 意图分类(LLM) → 条件分支
                          ├── 技术问题 → 知识检索 → LLM回答 → 结束
                          ├── 订单查询 → HTTP请求(查订单API) → LLM格式化 → 结束
                          └── 其他 → LLM通用回答 → 结束

各节点配置：

1. 意图分类 LLM 节点：
   提示词：将用户问题分类为 technical/order/other
   输出：category 变量

2. 条件分支：
   IF category == "technical" → 技术问题分支
   ELIF category == "order" → 订单查询分支
   ELSE → 通用回答分支

3. 知识检索节点：
   知识库：技术文档库
   Top K：3

4. HTTP 请求节点：
   URL：https://api.example.com/orders
   Method：GET
   参数：从用户消息中提取订单号
"""
```

### 5.3 通过 API 运行工作流

```python
def run_workflow(inputs: dict) -> dict:
    """运行 Dify 工作流"""
    headers = {
        "Authorization": f"Bearer {API_KEY}",
        "Content-Type": "application/json"
    }
    
    response = requests.post(
        f"{DIFY_API_URL}/workflows/run",
        headers=headers,
        json={
            "inputs": inputs,
            "response_mode": "blocking",
            "user": "user-001"
        }
    )
    
    return response.json()

# 使用
result = run_workflow({
    "query": "我的订单 ORD-001 到哪了？"
})
print(result["data"]["outputs"])
```

---

## 6. 构建 Agent 应用

### 6.1 Agent 配置

```python
"""
Dify Agent 应用配置：

1. 创建应用 → 选择"Agent"
2. 配置系统提示词
3. 添加工具：
   - 内置工具：Google搜索、天气查询、计算器等
   - 自定义工具：通过 OpenAPI Schema 定义
   - 工作流作为工具：将工作流封装为工具
4. 配置 Agent 策略：
   - Function Calling（推荐）
   - ReAct
5. 设置最大迭代次数
"""
```

### 6.2 自定义工具

```python
"""
通过 OpenAPI Schema 定义自定义工具：

在 Dify 中：设置 → 工具 → 创建自定义工具

OpenAPI Schema 示例：
"""

openapi_schema = """
openapi: 3.0.0
info:
  title: 订单查询 API
  version: 1.0.0
servers:
  - url: https://api.example.com
paths:
  /orders/{order_id}:
    get:
      operationId: getOrder
      summary: 查询订单详情
      parameters:
        - name: order_id
          in: path
          required: true
          schema:
            type: string
          description: 订单号
      responses:
        '200':
          description: 订单信息
          content:
            application/json:
              schema:
                type: object
                properties:
                  order_id:
                    type: string
                  status:
                    type: string
                  total:
                    type: number
"""
```

---

## 7. 插件开发

### 7.1 Dify 插件体系

```python
"""
Dify 1.x 插件体系：

插件类型：
1. Tool 插件 — 扩展工具能力
2. Model 插件 — 接入新的模型提供商
3. Extension 插件 — 扩展平台功能

插件开发流程：
1. 安装 Dify 插件 CLI
2. 初始化插件项目
3. 实现插件逻辑
4. 打包和发布
"""

# 安装 CLI
# pip install dify-plugin-daemon

# 初始化插件
# dify plugin init my-tool-plugin
```

### 7.2 Tool 插件示例

```python
"""
自定义 Tool 插件结构：

my-tool-plugin/
├── manifest.yaml        # 插件清单
├── provider/
│   └── my_tool.yaml     # 工具提供商定义
├── tools/
│   ├── weather.yaml     # 工具定义
│   └── weather.py       # 工具实现
└── requirements.txt
"""

# tools/weather.py
from dify_plugin import Tool, ToolInvokeMessage

class WeatherTool(Tool):
    def _invoke(self, tool_parameters: dict) -> ToolInvokeMessage:
        city = tool_parameters.get("city", "")
        
        # 调用天气 API
        weather = self._get_weather(city)
        
        return self.create_text_message(
            f"{city}天气：{weather['condition']}，温度{weather['temp']}"
        )
    
    def _get_weather(self, city: str) -> dict:
        # 实际调用天气 API
        return {"condition": "晴", "temp": "25°C"}
```

---

## 8. 生产部署最佳实践

### 8.1 部署架构

```python
"""
生产环境部署建议：

1. 基础设施
   - Docker Compose（小规模）
   - Kubernetes（大规模）
   - 数据库：PostgreSQL（必须）
   - 缓存：Redis（必须）
   - 对象存储：S3/MinIO（文件上传）
   - 向量数据库：Qdrant/Weaviate（RAG）

2. 性能优化
   - 配置 Nginx 反向代理
   - 启用 Redis 缓存
   - 向量数据库独立部署
   - Worker 数量根据负载调整

3. 安全配置
   - 启用 HTTPS
   - 配置 API 限流
   - 设置 CORS 白名单
   - 定期备份数据库
"""
```

### 8.2 监控和运维

```python
"""
Dify 运维要点：

1. 日志监控
   - 应用日志：查看每次对话的完整链路
   - Token 消耗：监控成本
   - 响应时间：性能监控

2. 数据标注
   - 对 LLM 回答进行人工标注
   - 收集用户反馈
   - 持续优化提示词

3. 知识库维护
   - 定期更新文档
   - 监控检索质量
   - 优化切分策略
"""
```

---

## 9. 总结

### 核心要点
1. Dify 是构建 AI 应用的高效平台，适合快速原型和生产部署
2. 可视化工作流编排降低了 Agent 开发门槛
3. 内置 RAG 引擎省去了自建检索系统的工作
4. 插件体系支持灵活扩展
5. 适合与纯代码方案互补使用

### 适用场景
- 快速构建 AI 聊天助手和客服系统
- 企业知识库问答
- 复杂业务工作流自动化
- AI 应用原型验证
- 非技术团队的 AI 应用开发

### 学习路径
1. Docker 部署 Dify
2. 创建第一个聊天助手
3. 构建 RAG 知识库应用
4. 学习工作流编排
5. 创建 Agent 应用
6. API 集成到业务系统

---

## 🔗 相关资源

- [Dify 官方文档](https://docs.dify.ai/)
- [Dify GitHub](https://github.com/langgenius/dify)
- [Dify Cloud](https://cloud.dify.ai/)
- [Dify 插件市场](https://marketplace.dify.ai/)

---

**@author erik.zhou**
