# OpenAI Agents SDK 完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

OpenAI Agents SDK 是 OpenAI 于 2025 年 3 月发布的官方 Python Agent 开发框架，旨在以极简抽象构建多 Agent 工作流。它强调轻量级、可组合、可观测，是构建生产级 Agent 系统的首选方案之一。

### 版本信息
- **SDK 版本**：1.x+
- **Python 要求**：>=3.9
- **重要程度**：⭐⭐⭐⭐⭐（必学）
- **难度等级**：⭐⭐⭐（中等）
- **预计学习时间**：15-20 小时

### 学习目标
- 理解 OpenAI Agents SDK 的核心概念和设计哲学
- 掌握 Agent、Tool、Handoff、Guardrail 四大原语
- 能够构建单 Agent 和多 Agent 协作系统
- 掌握 Tracing 可观测性和调试技巧
- 理解与 MCP 协议的集成方式

### 前置知识
- Python 异步编程（async/await）
- OpenAI API 基础
- Agent 开发基本概念
- Prompt Engineering 基础

---

## 1. SDK 简介与设计哲学

### 1.1 为什么选择 OpenAI Agents SDK

OpenAI Agents SDK 的设计遵循三个核心原则：

1. **极简抽象**：仅提供 Agent、Handoff、Guardrail、Tool 四个核心原语
2. **高度可定制**：每一层都可以替换和扩展
3. **内置可观测性**：原生 Tracing 支持，方便调试和监控

```
OpenAI Agents SDK 核心架构
├── Agent（智能体）
│   ├── instructions（系统指令）
│   ├── model（模型选择）
│   ├── tools（工具列表）
│   └── handoffs（交接目标）
├── Tool（工具）
│   ├── Function Tool（函数工具）
│   ├── Hosted Tool（托管工具：代码解释器、文件搜索、Web搜索）
│   └── MCP Tool（MCP 协议工具）
├── Handoff（交接）
│   └── Agent 间任务委托
├── Guardrail（护栏）
│   ├── Input Guardrail（输入校验）
│   └── Output Guardrail（输出校验）
└── Tracing（追踪）
    ├── Trace / Span
    └── 自定义 Span
```

### 1.2 与其他框架对比

| 特性 | OpenAI Agents SDK | LangGraph | CrewAI |
|------|-------------------|-----------|--------|
| 抽象层级 | 极简（4个原语） | 中等（状态图） | 较高（角色扮演） |
| 多 Agent | Handoff 机制 | 图节点 | Crew 团队 |
| 工具集成 | 原生 + MCP | LangChain 工具 | CrewAI Tools |
| 可观测性 | 内置 Tracing | 需集成 | 需集成 |
| 模型绑定 | OpenAI 优先 | 模型无关 | 模型无关 |
| 学习曲线 | 低 | 中 | 中 |

---

## 2. 环境搭建

### 2.1 安装

```bash
# 安装 OpenAI Agents SDK
pip install openai-agents

# 如果需要 MCP 支持
pip install "openai-agents[mcp]"

# 如果需要语音支持
pip install "openai-agents[voice]"

# 设置 API Key
export OPENAI_API_KEY="sk-xxx"
```

### 2.2 基础配置

```python
import os
from dotenv import load_dotenv

load_dotenv()

# 确保 API Key 已设置
assert os.environ.get("OPENAI_API_KEY"), "请设置 OPENAI_API_KEY"
```

---

## 3. 核心概念：Agent

### 3.1 创建基础 Agent

```python
from agents import Agent, Runner

# 🔥 创建一个简单的 Agent
agent = Agent(
    name="助手",
    instructions="你是一个友好的中文AI助手，简洁明了地回答问题。",
    model="gpt-4o"
)

# 同步运行
result = Runner.run_sync(agent, "什么是机器学习？")
print(result.final_output)

# 异步运行
import asyncio

async def main():
    result = await Runner.run(agent, "什么是深度学习？")
    print(result.final_output)

asyncio.run(main())
```

### 3.2 Agent 配置详解

```python
from agents import Agent, ModelSettings

agent = Agent(
    name="高级助手",
    instructions="""你是一位资深技术顾问。
    - 回答要专业且有深度
    - 使用中文回答
    - 必要时给出代码示例""",
    model="gpt-4o",
    # 模型参数配置
    model_settings=ModelSettings(
        temperature=0.7,
        top_p=0.9,
        max_tokens=2000
    ),
    # 工具列表
    tools=[],
    # 交接目标
    handoffs=[],
    # 输出类型（结构化输出）
    output_type=None,  # 默认为字符串
)
```

### 3.3 结构化输出

```python
from pydantic import BaseModel
from agents import Agent, Runner

# 定义输出结构
class CodeReview(BaseModel):
    score: int          # 代码评分 1-10
    issues: list[str]   # 发现的问题
    suggestions: list[str]  # 改进建议
    summary: str        # 总结

# 创建带结构化输出的 Agent
reviewer = Agent(
    name="代码审查员",
    instructions="你是一位严格的代码审查员，请审查用户提交的代码。",
    model="gpt-4o",
    output_type=CodeReview  # 🔥 指定输出类型
)

result = Runner.run_sync(reviewer, """
请审查以下 Python 代码：
```python
def calc(x,y):
    return x/y
```
""")

review = result.final_output_as(CodeReview)
print(f"评分: {review.score}")
print(f"问题: {review.issues}")
print(f"建议: {review.suggestions}")
```

---

## 4. 核心概念：Tool（工具）

### 4.1 函数工具（Function Tool）

```python
from agents import Agent, Runner, function_tool

# 🔥 使用装饰器定义工具
@function_tool
def get_weather(city: str) -> str:
    """获取指定城市的天气信息
    
    Args:
        city: 城市名称，如"北京"、"上海"
    """
    # 实际项目中调用天气 API
    weather_data = {
        "北京": "晴天，25°C",
        "上海": "多云，22°C",
        "深圳": "阵雨，28°C"
    }
    return weather_data.get(city, f"{city}：暂无数据")

@function_tool
def search_database(query: str, limit: int = 5) -> str:
    """在知识库中搜索相关信息
    
    Args:
        query: 搜索关键词
        limit: 返回结果数量上限
    """
    return f"搜索 '{query}' 的前 {limit} 条结果：..."

# 创建带工具的 Agent
agent = Agent(
    name="天气助手",
    instructions="你是一个天气查询助手，使用工具获取天气信息。",
    tools=[get_weather, search_database]
)

result = Runner.run_sync(agent, "北京和上海今天天气怎么样？")
print(result.final_output)
```

### 4.2 托管工具（Hosted Tools）

```python
from agents import Agent, Runner
from agents.tools import CodeInterpreterTool, FileSearchTool, WebSearchTool

# 🔥 代码解释器
code_agent = Agent(
    name="数据分析师",
    instructions="你是一个数据分析师，使用代码解释器分析数据。",
    tools=[CodeInterpreterTool()]
)

# 🔥 文件搜索（需要先上传文件到 Vector Store）
file_agent = Agent(
    name="文档助手",
    instructions="你是一个文档助手，从上传的文件中检索信息。",
    tools=[FileSearchTool(
        vector_store_ids=["vs_xxx"]  # Vector Store ID
    )]
)

# 🔥 Web 搜索
web_agent = Agent(
    name="搜索助手",
    instructions="你是一个搜索助手，使用网络搜索获取最新信息。",
    tools=[WebSearchTool()]
)
```

### 4.3 MCP 工具集成

```python
from agents import Agent, Runner
from agents.mcp import MCPServerStdio, MCPServerSse

# 🔥 通过 stdio 连接 MCP Server
async def main():
    async with MCPServerStdio(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/tmp"]
    ) as mcp_server:
        
        # 获取 MCP 工具列表
        tools = await mcp_server.list_tools()
        
        agent = Agent(
            name="文件管理助手",
            instructions="你是一个文件管理助手。",
            mcp_servers=[mcp_server]  # 🔥 直接传入 MCP Server
        )
        
        result = await Runner.run(agent, "列出 /tmp 目录下的文件")
        print(result.final_output)

# 通过 SSE 连接远程 MCP Server
async def remote_mcp():
    async with MCPServerSse(
        url="http://localhost:8080/sse"
    ) as mcp_server:
        agent = Agent(
            name="远程工具助手",
            instructions="使用远程工具完成任务。",
            mcp_servers=[mcp_server]
        )
        result = await Runner.run(agent, "执行远程操作")
        print(result.final_output)
```

---

## 5. 核心概念：Handoff（交接）

### 5.1 基础交接

```python
from agents import Agent, Runner

# 🔥 定义专业 Agent
chinese_agent = Agent(
    name="中文助手",
    instructions="你只用中文回答问题。",
    model="gpt-4o-mini"
)

english_agent = Agent(
    name="English Assistant",
    instructions="You only answer in English.",
    model="gpt-4o-mini"
)

# 🔥 路由 Agent（通过 handoffs 实现交接）
triage_agent = Agent(
    name="路由助手",
    instructions="""你是一个语言路由助手。
    - 如果用户说中文，交接给中文助手
    - 如果用户说英文，交接给 English Assistant""",
    handoffs=[chinese_agent, english_agent]  # 🔥 交接目标
)

# 运行
result = Runner.run_sync(triage_agent, "请介绍一下Python")
print(result.final_output)  # 中文助手回答

result = Runner.run_sync(triage_agent, "Tell me about Python")
print(result.final_output)  # English Assistant 回答
```

### 5.2 多 Agent 协作流水线

```python
from agents import Agent, Runner

# 研究员 Agent
researcher = Agent(
    name="研究员",
    instructions="""你是一位技术研究员。
    收集和整理给定主题的关键信息，完成后交接给写作者。""",
    handoffs=[]  # 稍后设置
)

# 写作者 Agent
writer = Agent(
    name="写作者",
    instructions="""你是一位技术写作者。
    基于研究员提供的信息，撰写一篇结构清晰的文章，完成后交接给审核者。""",
    handoffs=[]
)

# 审核者 Agent
reviewer = Agent(
    name="审核者",
    instructions="""你是一位内容审核者。
    审核文章的准确性和可读性，给出最终版本。"""
)

# 🔥 设置交接链
researcher.handoffs = [writer]
writer.handoffs = [reviewer]

# 运行流水线
result = Runner.run_sync(
    researcher,
    "研究 2025-2026 年 AI Agent 技术的最新进展"
)
print(result.final_output)
```

---

## 6. 核心概念：Guardrail（护栏）

### 6.1 输入护栏

```python
from agents import Agent, Runner, InputGuardrail, GuardrailFunctionOutput
from pydantic import BaseModel

# 定义检测结果
class SafetyCheck(BaseModel):
    is_safe: bool
    reason: str

# 🔥 安全检查 Agent
safety_agent = Agent(
    name="安全检查",
    instructions="判断用户输入是否安全合规，不包含恶意内容。",
    output_type=SafetyCheck
)

# 定义输入护栏
async def check_input_safety(ctx, agent, input_text):
    result = await Runner.run(safety_agent, input_text)
    check = result.final_output_as(SafetyCheck)
    return GuardrailFunctionOutput(
        output_info=check,
        tripwire_triggered=not check.is_safe  # 不安全时触发
    )

# 创建带护栏的 Agent
safe_agent = Agent(
    name="安全助手",
    instructions="你是一个安全的AI助手。",
    input_guardrails=[
        InputGuardrail(guardrail_function=check_input_safety)
    ]
)
```

### 6.2 输出护栏

```python
from agents import Agent, OutputGuardrail, GuardrailFunctionOutput
from pydantic import BaseModel

class OutputCheck(BaseModel):
    contains_pii: bool
    sanitized_output: str

# 输出检查 Agent
pii_checker = Agent(
    name="PII检查",
    instructions="检查输出中是否包含个人隐私信息（PII），如有则脱敏。",
    output_type=OutputCheck
)

async def check_output_pii(ctx, agent, output_text):
    result = await Runner.run(pii_checker, output_text)
    check = result.final_output_as(OutputCheck)
    return GuardrailFunctionOutput(
        output_info=check,
        tripwire_triggered=check.contains_pii
    )

# 带输出护栏的 Agent
agent = Agent(
    name="客服助手",
    instructions="你是客服助手，回答用户问题。",
    output_guardrails=[
        OutputGuardrail(guardrail_function=check_output_pii)
    ]
)
```

---

## 7. Tracing 可观测性

### 7.1 内置 Tracing

```python
from agents import Agent, Runner, trace

# 🔥 SDK 默认开启 Tracing，每次 Runner.run 自动创建 Trace
result = Runner.run_sync(agent, "你好")

# 自定义 Trace
with trace("my-custom-workflow"):
    result1 = Runner.run_sync(agent1, "步骤1")
    result2 = Runner.run_sync(agent2, f"步骤2: {result1.final_output}")
```

### 7.2 集成外部追踪系统

```python
from agents.tracing import set_trace_processors
from agents.tracing.processors import BackendSpanExporter

# 发送到 OpenAI 后台（默认）
# Trace 数据可在 OpenAI Dashboard 中查看

# 自定义导出器
class CustomExporter(BackendSpanExporter):
    def export(self, spans):
        for span in spans:
            print(f"Span: {span.span_id}, Duration: {span.duration_ms}ms")
        super().export(spans)

set_trace_processors([CustomExporter()])
```

---

## 8. 实战案例

### 8.1 智能客服系统

```python
from agents import Agent, Runner, function_tool

@function_tool
def query_order(order_id: str) -> str:
    """查询订单状态"""
    orders = {
        "ORD001": "已发货，预计明天到达",
        "ORD002": "处理中，预计3天内发货",
    }
    return orders.get(order_id, "订单不存在")

@function_tool
def create_ticket(issue: str, priority: str = "normal") -> str:
    """创建工单"""
    return f"工单已创建：{issue}（优先级：{priority}）"

# 订单查询 Agent
order_agent = Agent(
    name="订单助手",
    instructions="你负责处理订单相关查询。",
    tools=[query_order]
)

# 技术支持 Agent
tech_agent = Agent(
    name="技术支持",
    instructions="你负责处理技术问题，必要时创建工单。",
    tools=[create_ticket]
)

# 路由 Agent
triage = Agent(
    name="客服路由",
    instructions="""你是智能客服入口。
    - 订单相关问题 → 交接给订单助手
    - 技术问题 → 交接给技术支持
    - 其他问题直接回答""",
    handoffs=[order_agent, tech_agent]
)

# 运行
result = Runner.run_sync(triage, "我的订单 ORD001 到哪了？")
print(result.final_output)
```

### 8.2 数据分析 Agent

```python
from agents import Agent, Runner, function_tool
from pydantic import BaseModel

class AnalysisReport(BaseModel):
    title: str
    key_findings: list[str]
    recommendations: list[str]
    data_summary: str

@function_tool
def query_sales_data(period: str) -> str:
    """查询销售数据"""
    return f"{period} 销售数据：总额 500 万，同比增长 15%，TOP3 产品：A/B/C"

@function_tool
def query_user_metrics(metric: str) -> str:
    """查询用户指标"""
    metrics = {
        "DAU": "日活 10 万",
        "retention": "次日留存 45%",
        "conversion": "转化率 3.2%"
    }
    return metrics.get(metric, "指标不存在")

analyst = Agent(
    name="数据分析师",
    instructions="""你是一位资深数据分析师。
    使用工具查询数据，生成结构化的分析报告。""",
    tools=[query_sales_data, query_user_metrics],
    output_type=AnalysisReport
)

result = Runner.run_sync(analyst, "分析上个月的业务表现")
report = result.final_output_as(AnalysisReport)
print(f"标题: {report.title}")
for finding in report.key_findings:
    print(f"  - {finding}")
```

---

## 9. 最佳实践

### 9.1 Agent 设计原则

```python
# ✅ 好的设计：职责单一、指令清晰
good_agent = Agent(
    name="SQL查询助手",
    instructions="""你是一个 SQL 查询助手。
    规则：
    1. 只生成 SELECT 查询，禁止 DELETE/UPDATE/DROP
    2. 所有查询必须带 LIMIT
    3. 使用参数化查询防止注入""",
    tools=[execute_readonly_query]
)

# ❌ 不好的设计：职责模糊、指令不清
bad_agent = Agent(
    name="助手",
    instructions="帮用户做事",
    tools=[everything_tool]
)
```

### 9.2 错误处理

```python
from agents import Agent, Runner
from agents.exceptions import MaxTurnsExceeded, GuardrailTripwireTriggered

async def safe_run(agent, prompt):
    try:
        result = await Runner.run(
            agent, prompt,
            max_turns=10  # 限制最大轮次
        )
        return result.final_output
    except MaxTurnsExceeded:
        return "对话轮次超限，请简化问题"
    except GuardrailTripwireTriggered as e:
        return f"安全检查未通过：{e}"
    except Exception as e:
        return f"系统错误：{e}"
```

---

## 10. 总结

### 核心要点
1. OpenAI Agents SDK 以极简四原语（Agent/Tool/Handoff/Guardrail）构建 Agent 系统
2. Handoff 机制实现优雅的多 Agent 协作
3. 内置 Tracing 提供开箱即用的可观测性
4. 原生支持 MCP 协议，可接入丰富的外部工具生态

### 学习路径
1. 掌握单 Agent + Function Tool 基础
2. 学习 Handoff 多 Agent 协作
3. 添加 Guardrail 安全护栏
4. 集成 MCP 工具和 Tracing
5. 实战：构建生产级 Agent 系统

### 进阶方向
- 与 MCP 协议深度集成
- 结合 A2A 协议实现跨平台 Agent 通信
- 语音 Agent 开发
- 大规模 Agent 编排和监控

---

## 🔗 相关资源

- [OpenAI Agents SDK 官方文档](https://openai.github.io/openai-agents-python/)
- [OpenAI Agents SDK GitHub](https://github.com/openai/openai-agents-python)
- [OpenAI Cookbook - Agents](https://cookbook.openai.com/)

---

**@author erik.zhou**
