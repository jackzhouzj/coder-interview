# A2A 协议与 Google ADK 完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

A2A（Agent-to-Agent）是 Google 于 2025 年 4 月发布的开放协议，旨在实现不同框架、不同厂商的 AI Agent 之间的标准化通信与协作。Google ADK（Agent Development Kit）是配套的 Agent 开发框架，原生支持 A2A 和 MCP 协议。

### 版本信息
- **A2A 协议版本**：1.0+
- **Google ADK 版本**：1.0+ GA（2025 年 Google I/O 发布）
- **重要程度**：⭐⭐⭐⭐（重要）
- **难度等级**：⭐⭐⭐⭐（较难）
- **预计学习时间**：15-20 小时

### 学习目标
- 理解 A2A 协议的设计理念和核心概念
- 掌握 A2A 与 MCP 的区别和互补关系
- 学会使用 Google ADK 构建 Agent
- 能够实现跨框架的 Agent 协作

### 前置知识
- Python 异步编程
- HTTP/JSON-RPC 基础
- AI Agent 开发基础
- MCP 协议基础（推荐）

---

## 1. A2A 协议核心概念

### 1.1 A2A vs MCP

```
两大协议的定位差异：

MCP（Model Context Protocol）
├── 解决：Agent ↔ 工具/数据 的连接
├── 类比：USB-C 接口（连接外设）
├── 方向：Agent 调用外部能力
└── 场景：数据库查询、文件操作、API调用

A2A（Agent-to-Agent Protocol）
├── 解决：Agent ↔ Agent 的通信协作
├── 类比：HTTP 协议（服务间通信）
├── 方向：Agent 之间互相发现和协作
└── 场景：多Agent系统、跨平台Agent协作

两者互补，不是竞争关系！
```

### 1.2 A2A 核心概念

| 概念 | 说明 |
|------|------|
| **Agent Card** | Agent 的"名片"，描述能力、端点、认证方式 |
| **Task** | Agent 间协作的基本单位 |
| **Message** | Task 中的通信消息 |
| **Part** | Message 的内容部分（文本/文件/数据） |
| **Artifact** | Task 产生的输出物 |

### 1.3 A2A 工作流程

```python
"""
A2A 协议工作流程

1. 发现（Discovery）
   Client 通过 /.well-known/agent.json 发现 Agent

2. 发送任务（Send Task）
   Client 向 Agent 发送 Task 请求

3. 处理（Processing）
   Agent 处理任务，可能产生多轮交互

4. 返回结果（Response）
   Agent 返回 Artifact（输出物）
"""

# Agent Card 示例
agent_card = {
    "name": "天气查询Agent",
    "description": "提供全球城市天气查询服务",
    "url": "https://weather-agent.example.com",
    "version": "1.0.0",
    "capabilities": {
        "streaming": True,
        "pushNotifications": False
    },
    "skills": [
        {
            "id": "get-weather",
            "name": "天气查询",
            "description": "查询指定城市的天气信息",
            "inputModes": ["text"],
            "outputModes": ["text"]
        }
    ],
    "authentication": {
        "schemes": ["bearer"]
    }
}
```

---

## 2. Google ADK 快速开始

### 2.1 安装

```bash
# 安装 Google ADK
pip install google-adk

# 安装 A2A 支持
pip install google-adk[a2a]

# 设置 API Key（支持 Gemini 或其他模型）
export GOOGLE_API_KEY="your-api-key"
# 或使用 OpenAI
export OPENAI_API_KEY="your-api-key"
```

### 2.2 创建第一个 ADK Agent

```python
"""
Google ADK 基础示例
"""
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService

# 🔥 定义 Agent
root_agent = Agent(
    name="greeting_agent",
    model="gemini-2.0-flash",  # 或 "openai:gpt-4o"
    instruction="你是一个友好的中文助手，简洁地回答问题。",
    description="通用问答助手"
)

# 创建会话服务
session_service = InMemorySessionService()

# 创建 Runner
runner = Runner(
    agent=root_agent,
    app_name="my_app",
    session_service=session_service
)

# 运行
async def main():
    session = await session_service.create_session(
        app_name="my_app",
        user_id="user-001"
    )
    
    response = await runner.run(
        user_id="user-001",
        session_id=session.id,
        new_message="你好，介绍一下你自己"
    )
    
    print(response.text)
```

### 2.3 添加工具

```python
from google.adk.agents import Agent
from google.adk.tools import FunctionTool

# 定义工具函数
def get_weather(city: str) -> dict:
    """获取城市天气
    
    Args:
        city: 城市名称
    
    Returns:
        天气信息字典
    """
    return {
        "city": city,
        "temperature": "25°C",
        "condition": "晴天"
    }

def search_knowledge(query: str, top_k: int = 3) -> str:
    """搜索知识库
    
    Args:
        query: 搜索关键词
        top_k: 返回数量
    """
    return f"搜索 '{query}' 的前 {top_k} 条结果..."

# 🔥 创建带工具的 Agent
agent = Agent(
    name="tool_agent",
    model="gemini-2.0-flash",
    instruction="你是一个智能助手，使用工具回答问题。",
    tools=[
        FunctionTool(get_weather),
        FunctionTool(search_knowledge)
    ]
)
```

---

## 3. ADK 多 Agent 协作

### 3.1 子 Agent 模式

```python
from google.adk.agents import Agent

# 专业子 Agent
weather_agent = Agent(
    name="weather_agent",
    model="gemini-2.0-flash",
    instruction="你专门负责天气查询。",
    tools=[FunctionTool(get_weather)]
)

code_agent = Agent(
    name="code_agent",
    model="gemini-2.0-flash",
    instruction="你专门负责代码相关问题。"
)

# 🔥 路由 Agent（包含子 Agent）
root_agent = Agent(
    name="router",
    model="gemini-2.0-flash",
    instruction="""你是一个路由助手。
    - 天气相关问题交给 weather_agent
    - 代码相关问题交给 code_agent
    - 其他问题自己回答""",
    sub_agents=[weather_agent, code_agent]  # 🔥 子Agent
)
```

### 3.2 ADK + MCP 集成

```python
from google.adk.agents import Agent
from google.adk.tools.mcp_tool import MCPToolset, StdioServerParameters

# 🔥 ADK 原生支持 MCP
mcp_tools = MCPToolset(
    connection_params=StdioServerParameters(
        command="python",
        args=["my_mcp_server.py"]
    )
)

agent = Agent(
    name="mcp_agent",
    model="gemini-2.0-flash",
    instruction="使用 MCP 工具完成任务。",
    tools=[mcp_tools]
)
```

### 3.3 ADK + A2A 跨平台协作

```python
from google.adk.agents import Agent
from google.adk.tools.a2a_tool import A2AToolset

# 🔥 连接远程 A2A Agent
remote_agent_tool = A2AToolset(
    agent_card_url="https://remote-agent.example.com/.well-known/agent.json"
)

# 本地 Agent 可以调用远程 A2A Agent
local_agent = Agent(
    name="orchestrator",
    model="gemini-2.0-flash",
    instruction="你是一个编排器，协调本地和远程Agent完成任务。",
    tools=[remote_agent_tool]
)
```

---

## 4. A2A Server 开发

### 4.1 实现 A2A Server

```python
"""
A2A Server 实现示例
使用 google-adk 的 A2A 服务端支持
"""
from google.adk.agents import Agent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.a2a import A2AServer

# 定义 Agent
analysis_agent = Agent(
    name="data_analyst",
    model="gemini-2.0-flash",
    instruction="你是一位数据分析师，分析用户提供的数据。",
    description="数据分析Agent，支持数据解读和趋势分析"
)

# 创建 A2A Server
session_service = InMemorySessionService()
runner = Runner(
    agent=analysis_agent,
    app_name="analyst",
    session_service=session_service
)

a2a_server = A2AServer(
    runner=runner,
    agent_card={
        "name": "数据分析Agent",
        "description": "提供数据分析和解读服务",
        "skills": [
            {
                "id": "analyze-data",
                "name": "数据分析",
                "description": "分析数据并给出洞察",
                "inputModes": ["text"],
                "outputModes": ["text"]
            }
        ]
    }
)

# 启动 A2A Server
if __name__ == "__main__":
    a2a_server.run(host="0.0.0.0", port=8080)
    # Agent Card 可通过 http://localhost:8080/.well-known/agent.json 访问
```

---

## 5. 最佳实践

### 5.1 协议选择指南

```python
"""
何时使用 MCP vs A2A：

使用 MCP 的场景：
- Agent 需要访问数据库、文件系统、API 等外部工具
- 工具是确定性的、无状态的
- 需要标准化的工具接口

使用 A2A 的场景：
- 多个 Agent 需要协作完成复杂任务
- Agent 分布在不同平台/框架上
- 需要 Agent 间的发现和通信机制
- 任务需要多轮交互和状态管理

同时使用 MCP + A2A：
- Agent 通过 MCP 连接工具
- Agent 之间通过 A2A 协作
- 这是推荐的生产架构
"""
```

### 5.2 Agent Card 设计

```python
# ✅ 好的 Agent Card：描述清晰、能力明确
good_card = {
    "name": "SQL查询Agent",
    "description": "安全地执行只读SQL查询，支持MySQL和PostgreSQL",
    "skills": [
        {
            "id": "query-sql",
            "name": "SQL查询",
            "description": "执行SELECT查询并返回结果",
            "inputModes": ["text"],
            "outputModes": ["text", "data"]
        }
    ]
}

# ❌ 不好的 Agent Card：描述模糊
bad_card = {
    "name": "助手",
    "description": "做各种事情",
    "skills": []
}
```

---

## 6. 总结

### 核心要点
1. A2A 解决 Agent 间通信，MCP 解决 Agent 与工具的连接，两者互补
2. Google ADK 原生支持 A2A + MCP，是构建多 Agent 系统的强力框架
3. Agent Card 是 A2A 的核心，定义了 Agent 的能力和接口
4. A2A 已获 150+ 组织支持，是 Agent 互操作的行业标准

### 学习路径
1. 理解 A2A 与 MCP 的区别和互补关系
2. 使用 Google ADK 构建基础 Agent
3. 实现多 Agent 协作（子Agent + A2A）
4. 集成 MCP 工具
5. 部署 A2A Server

---

## 🔗 相关资源

- [A2A 协议规范](https://github.com/google/A2A)
- [Google ADK 文档](https://google.github.io/adk-docs/)
- [Google ADK GitHub](https://github.com/google/adk-python)
- [A2A + MCP 集成指南](https://developers.googleblog.com/developers-guide-to-ai-agent-protocols/)

---

**@author erik.zhou**
