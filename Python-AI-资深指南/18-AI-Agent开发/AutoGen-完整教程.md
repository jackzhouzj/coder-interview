# Microsoft AutoGen 完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

AutoGen 是微软开源的多 Agent 对话框架，2025 年与 Semantic Kernel 合并为统一的 Agent Framework，于 2026 年 Q1 发布 1.0 GA。它支持创造性 Agent 行为和确定性工作流执行的双模式编排，是企业级多 Agent 系统的重要选择。

### 版本信息
- **AutoGen 版本**：0.4+ / 1.0 GA（2026）
- **Python 要求**：>=3.10
- **重要程度**：⭐⭐⭐⭐（重要）
- **难度等级**：⭐⭐⭐⭐（较难）
- **预计学习时间**：20-25 小时

### 学习目标
- 理解 AutoGen 的核心架构和设计理念
- 掌握 Agent 创建和多 Agent 对话
- 学会使用 AgentChat 高级 API
- 理解 Agent Runtime 和消息传递机制
- 能够构建复杂的多 Agent 工作流

### 前置知识
- Python 异步编程
- LLM API 使用经验
- 多 Agent 系统基本概念

---

## 1. AutoGen 架构

### 1.1 核心架构

```
AutoGen 架构（0.4+ / 1.0）
├── autogen-core（核心层）
│   ├── Agent Runtime（Agent 运行时）
│   ├── Message Protocol（消息协议）
│   ├── Subscription（订阅机制）
│   └── Agent Lifecycle（生命周期管理）
├── autogen-agentchat（高级 API）🔥 推荐入口
│   ├── AssistantAgent（助手 Agent）
│   ├── UserProxyAgent（用户代理）
│   ├── Teams（团队编排）
│   │   ├── RoundRobinGroupChat（轮询）
│   │   ├── SelectorGroupChat（选择器）
│   │   └── Swarm（蜂群模式）
│   └── Termination Conditions（终止条件）
├── autogen-ext（扩展）
│   ├── Model Clients（模型客户端）
│   ├── Tools（工具集成）
│   ├── Code Executors（代码执行器）
│   └── MCP Integration（MCP 集成）
└── AutoGen Studio（可视化界面）
```

### 1.2 AutoGen vs 其他框架

| 特性 | AutoGen | CrewAI | OpenAI Agents SDK |
|------|---------|--------|-------------------|
| 多 Agent 对话 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| 代码执行 | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| 企业级特性 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 易用性 | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 可视化 | ⭐⭐⭐⭐（Studio） | ⭐⭐ | ⭐⭐ |
| 模型无关 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐（OpenAI优先） |

---

## 2. 环境搭建

### 2.1 安装

```bash
# 🔥 安装 AgentChat（推荐入口）
pip install "autogen-agentchat" "autogen-ext[openai]"

# 安装 OpenAI 模型客户端
pip install "autogen-ext[openai]"

# 安装 Anthropic 模型客户端
pip install "autogen-ext[anthropic]"

# 安装代码执行器
pip install "autogen-ext[docker]"  # Docker 执行器
pip install "autogen-ext[local]"   # 本地执行器

# 安装 MCP 支持
pip install "autogen-ext[mcp]"

# 安装 AutoGen Studio（可视化）
pip install autogenstudio
```

### 2.2 配置

```python
import os
os.environ["OPENAI_API_KEY"] = "sk-xxx"
```

---

## 3. AgentChat 快速开始

### 3.1 单 Agent 对话

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

# 🔥 创建模型客户端
model_client = OpenAIChatCompletionClient(model="gpt-4o")

# 🔥 创建 Agent
agent = AssistantAgent(
    name="assistant",
    model_client=model_client,
    system_message="你是一个专业的 Python 技术顾问，使用中文回答。"
)

# 运行
async def main():
    response = await agent.run(task="解释 Python 的 GIL 机制")
    print(response.messages[-1].content)

asyncio.run(main())
```

### 3.2 带工具的 Agent

```python
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_core.tools import FunctionTool

# 定义工具函数
async def get_weather(city: str) -> str:
    """获取城市天气信息"""
    weather_data = {"北京": "晴天 25°C", "上海": "多云 22°C"}
    return weather_data.get(city, f"{city}: 暂无数据")

async def calculate(expression: str) -> str:
    """计算数学表达式"""
    try:
        result = eval(expression)
        return f"计算结果: {result}"
    except Exception as e:
        return f"计算错误: {e}"

# 🔥 创建工具
weather_tool = FunctionTool(get_weather, description="获取城市天气")
calc_tool = FunctionTool(calculate, description="数学计算")

# 创建带工具的 Agent
agent = AssistantAgent(
    name="tool_agent",
    model_client=OpenAIChatCompletionClient(model="gpt-4o"),
    tools=[weather_tool, calc_tool],
    system_message="你是一个智能助手，使用工具回答问题。"
)

async def main():
    response = await agent.run(
        task="北京天气怎么样？另外帮我算 123 * 456"
    )
    for msg in response.messages:
        print(f"[{msg.source}]: {msg.content}")

asyncio.run(main())
```

---

## 4. 多 Agent 团队

### 4.1 RoundRobin 轮询对话

```python
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination
from autogen_ext.models.openai import OpenAIChatCompletionClient

model = OpenAIChatCompletionClient(model="gpt-4o")

# 🔥 创建多个 Agent
researcher = AssistantAgent(
    name="researcher",
    model_client=model,
    system_message="""你是一位技术研究员。
    收集和分析给定主题的关键信息。
    完成后说 APPROVE 表示研究完成。"""
)

writer = AssistantAgent(
    name="writer",
    model_client=model,
    system_message="""你是一位技术写作者。
    基于研究员的信息撰写文章。
    完成后说 APPROVE 表示写作完成。"""
)

reviewer = AssistantAgent(
    name="reviewer",
    model_client=model,
    system_message="""你是一位内容审核者。
    审核文章质量，给出修改建议或确认通过。
    如果通过，说 APPROVE。"""
)

# 🔥 创建团队（轮询模式）
termination = TextMentionTermination("APPROVE")

team = RoundRobinGroupChat(
    participants=[researcher, writer, reviewer],
    termination_condition=termination,
    max_turns=10
)

async def main():
    result = await team.run(
        task="写一篇关于 GraphRAG 技术的技术博客"
    )
    for msg in result.messages:
        print(f"\n[{msg.source}]:")
        print(msg.content[:200])

asyncio.run(main())
```

### 4.2 Selector 选择器模式

```python
from autogen_agentchat.teams import SelectorGroupChat

# 🔥 选择器模式：由 LLM 决定下一个发言的 Agent
team = SelectorGroupChat(
    participants=[researcher, writer, reviewer],
    model_client=model,  # 用于选择下一个 Agent 的模型
    termination_condition=termination,
    max_turns=10,
    # 选择器提示词
    selector_prompt="""根据对话内容，选择下一个最合适的参与者。
    - researcher：需要收集信息时
    - writer：需要撰写内容时
    - reviewer：需要审核内容时"""
)

async def main():
    result = await team.run(task="分析 AI Agent 技术趋势")
    print(f"总共 {len(result.messages)} 轮对话")

asyncio.run(main())
```

### 4.3 Swarm 蜂群模式

```python
from autogen_agentchat.teams import Swarm
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.conditions import HandoffTermination

# 🔥 Swarm 模式：Agent 之间通过 Handoff 传递控制权
# 类似 OpenAI Agents SDK 的 Handoff 机制

order_agent = AssistantAgent(
    name="order_agent",
    model_client=model,
    handoffs=["tech_agent"],  # 可以交接给技术支持
    system_message="""你负责处理订单问题。
    如果遇到技术问题，使用 transfer_to_tech_agent 交接。"""
)

tech_agent = AssistantAgent(
    name="tech_agent",
    model_client=model,
    handoffs=["order_agent"],
    system_message="""你负责处理技术问题。
    如果遇到订单问题，使用 transfer_to_order_agent 交接。"""
)

termination = HandoffTermination(target="user")

swarm = Swarm(
    participants=[order_agent, tech_agent],
    termination_condition=termination
)

async def main():
    result = await swarm.run(task="我的订单出了技术问题")
    for msg in result.messages:
        print(f"[{msg.source}]: {msg.content[:100]}")

asyncio.run(main())
```

---

## 5. 代码执行

### 5.1 Docker 代码执行器

```python
from autogen_agentchat.agents import AssistantAgent, CodeExecutorAgent
from autogen_ext.code_executors.docker import DockerCommandLineCodeExecutor

# 🔥 Docker 代码执行器（安全隔离）
code_executor = DockerCommandLineCodeExecutor(
    image="python:3.11-slim",
    timeout=60,
    work_dir="/tmp/code"
)

# 代码执行 Agent
coder = AssistantAgent(
    name="coder",
    model_client=model,
    system_message="你是一个 Python 程序员，编写代码解决问题。"
)

executor = CodeExecutorAgent(
    name="executor",
    code_executor=code_executor
)

# 组成团队
team = RoundRobinGroupChat(
    participants=[coder, executor],
    max_turns=6
)

async def main():
    async with code_executor:  # 上下文管理器管理 Docker 容器
        result = await team.run(
            task="用 Python 生成斐波那契数列前 20 项并画图"
        )
        print(result.messages[-1].content)

asyncio.run(main())
```

---

## 6. MCP 集成

```python
from autogen_ext.tools.mcp import MCPToolAdapter, StdioServerParams

# 🔥 AutoGen + MCP
async def main():
    # 连接 MCP Server
    mcp_params = StdioServerParams(
        command="python",
        args=["my_mcp_server.py"]
    )
    
    # 获取 MCP 工具
    tools = await MCPToolAdapter.from_server(mcp_params)
    
    # 创建使用 MCP 工具的 Agent
    agent = AssistantAgent(
        name="mcp_agent",
        model_client=model,
        tools=tools,
        system_message="使用 MCP 工具完成任务。"
    )
    
    result = await agent.run(task="查询数据库中的用户表")
    print(result.messages[-1].content)

asyncio.run(main())
```

---

## 7. AutoGen Studio

```python
"""
AutoGen Studio 是 AutoGen 的可视化界面

安装和启动：
  pip install autogenstudio
  autogenstudio ui --port 8080

功能：
1. 可视化创建和配置 Agent
2. 拖拽式团队编排
3. 实时对话测试
4. 会话历史管理
5. 工具和模型配置

适合：
- 快速原型验证
- 非开发人员使用
- 团队协作和演示
"""
```

---

## 8. 最佳实践

### 8.1 Agent 设计原则

```python
# ✅ 好的设计：职责明确、提示词清晰
good_agent = AssistantAgent(
    name="sql_analyst",
    model_client=model,
    tools=[query_tool],
    system_message="""你是一位 SQL 数据分析师。

职责：
1. 分析用户的数据查询需求
2. 编写安全的 SELECT 查询
3. 解读查询结果

规则：
- 只使用 SELECT 语句
- 所有查询必须带 LIMIT
- 用中文解释结果"""
)

# ❌ 不好的设计
bad_agent = AssistantAgent(
    name="agent",
    model_client=model,
    system_message="帮用户做事"
)
```

### 8.2 终止条件设计

```python
from autogen_agentchat.conditions import (
    TextMentionTermination,
    MaxMessageTermination,
    TokenUsageTermination,
    TimeoutTermination
)

# 组合多个终止条件
termination = (
    TextMentionTermination("DONE")  # 关键词终止
    | MaxMessageTermination(20)      # 最大消息数
    | TokenUsageTermination(10000)   # Token 上限
    | TimeoutTermination(300)        # 超时（秒）
)
```

---

## 9. 总结

### 核心要点
1. AutoGen 是微软的企业级多 Agent 框架，支持复杂对话编排
2. AgentChat 是推荐的高级 API 入口，提供 Team 编排能力
3. 支持 RoundRobin/Selector/Swarm 三种团队模式
4. 内置代码执行器（Docker 隔离），适合数据分析场景
5. 原生支持 MCP 协议集成

### 学习路径
1. 使用 AgentChat 创建单 Agent
2. 学习多 Agent 团队编排
3. 集成工具和代码执行
4. 探索 AutoGen Studio
5. 生产部署和优化

---

## 🔗 相关资源

- [AutoGen 官方文档](https://microsoft.github.io/autogen/)
- [AutoGen GitHub](https://github.com/microsoft/autogen)
- [AutoGen Studio](https://microsoft.github.io/autogen/docs/autogen-studio/)

---

**@author erik.zhou**
