# Function Calling 与工具使用完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

Function Calling（函数调用/工具使用）是 AI Agent 的核心能力基石。它让 LLM 从"只能生成文本"进化为"能够执行动作"的智能体。本教程深入讲解 Function Calling 的原理、各平台实现、生产级最佳实践和常见陷阱。

### 版本信息
- **重要程度**：⭐⭐⭐⭐⭐（必学，Agent 开发基础）
- **难度等级**：⭐⭐⭐（中等）
- **预计学习时间**：15-20 小时

### 学习目标
- 理解 Function Calling 的工作原理
- 掌握 OpenAI / Anthropic / Google 各平台的实现
- 学会设计高质量的工具 Schema
- 掌握并行调用、流式调用、错误恢复等生产模式
- 理解结构化输出（Structured Output）与 Function Calling 的关系

### 前置知识
- Python 基础
- OpenAI API 基础
- JSON Schema 基础

---

## 1. Function Calling 原理

### 1.1 工作流程

```
Function Calling 完整流程：

1. 定义工具 Schema（函数名、参数、描述）
2. 用户发送消息 + 工具定义 → LLM
3. LLM 判断是否需要调用工具
   ├── 不需要 → 直接返回文本回答
   └── 需要 → 返回工具调用请求（函数名 + 参数 JSON）
4. 应用层执行函数，获取结果
5. 将函数结果返回给 LLM
6. LLM 基于函数结果生成最终回答
7. （可选）LLM 可能请求调用更多工具 → 循环
```

### 1.2 核心概念

```python
"""
Function Calling 三要素：

1. Tool Definition（工具定义）
   - 函数名称和描述
   - 参数的 JSON Schema
   - LLM 根据描述决定何时调用

2. Tool Call（工具调用）
   - LLM 生成的调用请求
   - 包含函数名和参数值
   - 可能同时调用多个工具（并行）

3. Tool Result（工具结果）
   - 函数执行的返回值
   - 反馈给 LLM 用于生成最终回答
"""
```

---

## 2. OpenAI Function Calling

### 2.1 基础用法

```python
from openai import OpenAI
import json

client = OpenAI()

# 🔥 步骤 1：定义工具
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取指定城市的当前天气信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "城市名称，如'北京'、'上海'"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "温度单位"
                    }
                },
                "required": ["city"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "search_products",
            "description": "搜索商品信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {
                        "type": "string",
                        "description": "搜索关键词"
                    },
                    "category": {
                        "type": "string",
                        "enum": ["electronics", "clothing", "food"],
                        "description": "商品类别"
                    },
                    "max_price": {
                        "type": "number",
                        "description": "最高价格"
                    }
                },
                "required": ["query"]
            }
        }
    }
]

# 🔥 步骤 2：发送请求
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "北京今天天气怎么样？"}
    ],
    tools=tools,
    tool_choice="auto"  # auto / none / required / 指定函数
)

message = response.choices[0].message

# 🔥 步骤 3：检查是否有工具调用
if message.tool_calls:
    for tool_call in message.tool_calls:
        func_name = tool_call.function.name
        func_args = json.loads(tool_call.function.arguments)
        
        print(f"调用函数: {func_name}")
        print(f"参数: {func_args}")
        
        # 步骤 4：执行函数
        if func_name == "get_weather":
            result = get_weather(**func_args)
        elif func_name == "search_products":
            result = search_products(**func_args)
        
        # 步骤 5：将结果返回给 LLM
        messages = [
            {"role": "user", "content": "北京今天天气怎么样？"},
            message,  # 包含 tool_calls 的 assistant 消息
            {
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": json.dumps(result, ensure_ascii=False)
            }
        ]
        
        # 步骤 6：获取最终回答
        final_response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools
        )
        
        print(final_response.choices[0].message.content)
else:
    print(message.content)
```

### 2.2 并行工具调用

```python
# 🔥 LLM 可以同时调用多个工具
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "北京和上海今天天气怎么样？"}
    ],
    tools=tools,
    parallel_tool_calls=True  # 允许并行调用（默认 True）
)

message = response.choices[0].message

# 可能返回多个 tool_calls
if message.tool_calls:
    print(f"并行调用 {len(message.tool_calls)} 个工具")
    
    # 并行执行所有工具
    tool_results = []
    for tool_call in message.tool_calls:
        func_name = tool_call.function.name
        func_args = json.loads(tool_call.function.arguments)
        result = execute_function(func_name, func_args)
        
        tool_results.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": json.dumps(result, ensure_ascii=False)
        })
    
    # 一次性返回所有结果
    messages = [
        {"role": "user", "content": "北京和上海今天天气怎么样？"},
        message,
        *tool_results
    ]
    
    final = client.chat.completions.create(
        model="gpt-4o", messages=messages, tools=tools
    )
    print(final.choices[0].message.content)
```

### 2.3 结构化输出（Structured Output）

```python
from pydantic import BaseModel
from openai import OpenAI

client = OpenAI()

# 🔥 使用 Pydantic 定义输出结构
class MovieReview(BaseModel):
    title: str
    rating: float
    pros: list[str]
    cons: list[str]
    summary: str

# 使用 response_format 获取结构化输出
response = client.beta.chat.completions.parse(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "评价电影《星际穿越》"}
    ],
    response_format=MovieReview
)

review = response.choices[0].message.parsed
print(f"电影: {review.title}")
print(f"评分: {review.rating}")
print(f"优点: {review.pros}")
```

---

## 3. Anthropic Tool Use

### 3.1 Claude 工具使用

```python
import anthropic
import json

client = anthropic.Anthropic()

# 🔥 Anthropic 的工具定义格式
tools = [
    {
        "name": "get_weather",
        "description": "获取指定城市的天气信息",
        "input_schema": {
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

# 发送请求
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=tools,
    messages=[
        {"role": "user", "content": "北京天气如何？"}
    ]
)

# 处理工具调用
for block in response.content:
    if block.type == "tool_use":
        print(f"工具: {block.name}")
        print(f"参数: {block.input}")
        print(f"ID: {block.id}")
        
        # 执行工具并返回结果
        result = get_weather(**block.input)
        
        # 继续对话
        final = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            tools=tools,
            messages=[
                {"role": "user", "content": "北京天气如何？"},
                {"role": "assistant", "content": response.content},
                {
                    "role": "user",
                    "content": [{
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": json.dumps(result)
                    }]
                }
            ]
        )
        
        print(final.content[0].text)
```

---

## 4. 完整的工具调用循环

### 4.1 Agent Loop 实现

```python
from openai import OpenAI
import json
from typing import Callable

client = OpenAI()

class ToolAgent:
    """带工具调用循环的 Agent"""
    
    def __init__(self, model: str = "gpt-4o", max_iterations: int = 10):
        self.model = model
        self.max_iterations = max_iterations
        self.tools_registry: dict[str, Callable] = {}
        self.tools_schema: list[dict] = []
    
    def register_tool(self, func: Callable, schema: dict):
        """注册工具"""
        self.tools_registry[schema["function"]["name"]] = func
        self.tools_schema.append(schema)
    
    def _execute_tool(self, name: str, args: dict) -> str:
        """执行工具"""
        func = self.tools_registry.get(name)
        if not func:
            return json.dumps({"error": f"未知工具: {name}"})
        
        try:
            result = func(**args)
            return json.dumps(result, ensure_ascii=False, default=str)
        except Exception as e:
            return json.dumps({"error": f"{type(e).__name__}: {str(e)}"})
    
    def run(self, user_message: str, system_prompt: str = "") -> str:
        """🔥 完整的工具调用循环"""
        messages = []
        if system_prompt:
            messages.append({"role": "system", "content": system_prompt})
        messages.append({"role": "user", "content": user_message})
        
        for i in range(self.max_iterations):
            response = client.chat.completions.create(
                model=self.model,
                messages=messages,
                tools=self.tools_schema if self.tools_schema else None
            )
            
            message = response.choices[0].message
            messages.append(message)
            
            # 没有工具调用，返回最终结果
            if not message.tool_calls:
                return message.content
            
            # 执行所有工具调用
            for tool_call in message.tool_calls:
                func_name = tool_call.function.name
                func_args = json.loads(tool_call.function.arguments)
                
                print(f"  [迭代{i+1}] 调用: {func_name}({func_args})")
                
                result = self._execute_tool(func_name, func_args)
                
                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": result
                })
        
        return "达到最大迭代次数，未能完成任务"

# 使用示例
agent = ToolAgent()

# 注册工具
def get_weather(city: str) -> dict:
    return {"city": city, "temp": "25°C", "condition": "晴"}

def calculate(expression: str) -> dict:
    try:
        return {"result": eval(expression)}
    except Exception as e:
        return {"error": str(e)}

agent.register_tool(get_weather, {
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "获取城市天气",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "城市名"}
            },
            "required": ["city"]
        }
    }
})

agent.register_tool(calculate, {
    "type": "function",
    "function": {
        "name": "calculate",
        "description": "数学计算",
        "parameters": {
            "type": "object",
            "properties": {
                "expression": {"type": "string", "description": "数学表达式"}
            },
            "required": ["expression"]
        }
    }
})

result = agent.run("北京天气怎么样？另外帮我算一下 123 * 456")
print(result)
```

---

## 5. 工具 Schema 设计最佳实践

### 5.1 好的 Schema 设计

```python
# ✅ 好的设计：描述清晰、参数约束明确
good_tool = {
    "type": "function",
    "function": {
        "name": "search_orders",
        "description": "根据条件搜索用户订单。支持按订单号、状态、日期范围筛选。返回匹配的订单列表。",
        "parameters": {
            "type": "object",
            "properties": {
                "order_id": {
                    "type": "string",
                    "description": "订单号，格式如 ORD-20240101-001"
                },
                "status": {
                    "type": "string",
                    "enum": ["pending", "shipped", "delivered", "cancelled"],
                    "description": "订单状态筛选"
                },
                "start_date": {
                    "type": "string",
                    "description": "开始日期，格式 YYYY-MM-DD"
                },
                "end_date": {
                    "type": "string",
                    "description": "结束日期，格式 YYYY-MM-DD"
                },
                "limit": {
                    "type": "integer",
                    "description": "返回数量上限，默认10，最大100",
                    "default": 10
                }
            },
            "required": []  # 所有参数可选，灵活组合
        }
    }
}

# ❌ 不好的设计：描述模糊、缺少约束
bad_tool = {
    "type": "function",
    "function": {
        "name": "search",
        "description": "搜索",  # 太模糊
        "parameters": {
            "type": "object",
            "properties": {
                "q": {"type": "string"},  # 参数名不清晰，无描述
                "n": {"type": "integer"}
            },
            "required": ["q"]
        }
    }
}
```

### 5.2 设计原则

```python
"""
工具 Schema 设计 7 原则：

1. 函数名用动词+名词：get_weather, search_orders, create_ticket
2. description 写清楚：做什么、什么时候用、返回什么
3. 参数描述要具体：包含格式、范围、示例
4. 用 enum 约束有限选项：status, category, priority
5. 合理设置 required：必填参数最少化
6. 单一职责：一个工具做一件事，不要大而全
7. 返回值要结构化：方便 LLM 理解和使用
"""
```

---

## 6. 生产级模式

### 6.1 错误恢复

```python
import time
from openai import OpenAI, RateLimitError, APIError

def robust_tool_call(client, messages, tools, max_retries=3):
    """带重试和错误恢复的工具调用"""
    for attempt in range(max_retries):
        try:
            response = client.chat.completions.create(
                model="gpt-4o",
                messages=messages,
                tools=tools,
                timeout=30
            )
            
            message = response.choices[0].message
            
            # 验证工具调用参数
            if message.tool_calls:
                for tc in message.tool_calls:
                    try:
                        json.loads(tc.function.arguments)
                    except json.JSONDecodeError:
                        # 参数 JSON 解析失败，要求重新生成
                        messages.append({
                            "role": "assistant",
                            "content": "工具调用参数格式错误，请重试。"
                        })
                        continue
            
            return response
            
        except RateLimitError:
            wait = 2 ** attempt
            print(f"速率限制，等待 {wait}s 后重试...")
            time.sleep(wait)
        except APIError as e:
            print(f"API 错误: {e}，重试 {attempt+1}/{max_retries}")
            time.sleep(1)
    
    raise Exception("工具调用失败，已达最大重试次数")
```

### 6.2 工具调用日志和监控

```python
import logging
import time
from functools import wraps

logger = logging.getLogger("tool_calls")

def log_tool_call(func):
    """工具调用日志装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        func_name = func.__name__
        
        logger.info(f"工具调用开始: {func_name}, 参数: {kwargs}")
        
        try:
            result = func(*args, **kwargs)
            duration = time.time() - start
            logger.info(f"工具调用成功: {func_name}, 耗时: {duration:.2f}s")
            return result
        except Exception as e:
            duration = time.time() - start
            logger.error(f"工具调用失败: {func_name}, 错误: {e}, 耗时: {duration:.2f}s")
            return {"error": str(e)}
    
    return wrapper

@log_tool_call
def get_weather(city: str) -> dict:
    """获取天气"""
    return {"city": city, "temp": "25°C"}
```

### 6.3 安全防护

```python
# 🔥 工具调用安全检查
ALLOWED_TOOLS = {"get_weather", "search_products", "calculate"}
DANGEROUS_TOOLS = {"execute_sql", "delete_file", "run_command"}

def safe_execute_tool(tool_name: str, args: dict, user_role: str = "user"):
    """安全的工具执行"""
    # 1. 白名单检查
    if tool_name not in ALLOWED_TOOLS:
        raise PermissionError(f"工具 {tool_name} 不在允许列表中")
    
    # 2. 危险工具需要管理员权限
    if tool_name in DANGEROUS_TOOLS and user_role != "admin":
        raise PermissionError(f"工具 {tool_name} 需要管理员权限")
    
    # 3. 参数清洗
    sanitized_args = sanitize_args(args)
    
    # 4. 执行
    return execute_function(tool_name, sanitized_args)

def sanitize_args(args: dict) -> dict:
    """参数清洗，防止注入"""
    sanitized = {}
    for key, value in args.items():
        if isinstance(value, str):
            # 移除潜在的注入字符
            value = value.replace(";", "").replace("--", "")
        sanitized[key] = value
    return sanitized
```

---

## 7. 流式工具调用

```python
from openai import OpenAI

client = OpenAI()

# 🔥 流式 Function Calling
stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "北京天气如何？"}],
    tools=tools,
    stream=True
)

# 收集流式工具调用
tool_calls_buffer = {}
content_buffer = ""

for chunk in stream:
    delta = chunk.choices[0].delta
    
    if delta.content:
        content_buffer += delta.content
        print(delta.content, end="", flush=True)
    
    if delta.tool_calls:
        for tc in delta.tool_calls:
            idx = tc.index
            if idx not in tool_calls_buffer:
                tool_calls_buffer[idx] = {
                    "id": tc.id or "",
                    "name": "",
                    "arguments": ""
                }
            if tc.id:
                tool_calls_buffer[idx]["id"] = tc.id
            if tc.function:
                if tc.function.name:
                    tool_calls_buffer[idx]["name"] = tc.function.name
                if tc.function.arguments:
                    tool_calls_buffer[idx]["arguments"] += tc.function.arguments

# 处理收集到的工具调用
for idx, tc in tool_calls_buffer.items():
    print(f"\n工具调用: {tc['name']}({tc['arguments']})")
```

---

## 8. 总结

### 核心要点
1. Function Calling 是 Agent 的基础能力，让 LLM 能执行动作
2. 工具 Schema 设计质量直接影响调用准确率
3. 生产环境必须有错误恢复、日志监控、安全防护
4. 并行调用和流式调用提升用户体验
5. 结构化输出是 Function Calling 的延伸

### 学习路径
1. 掌握 OpenAI Function Calling 基础
2. 实现完整的工具调用循环（Agent Loop）
3. 学习 Schema 设计最佳实践
4. 添加错误恢复和安全防护
5. 进阶：流式调用、并行调用、多模型适配

---

## 🔗 相关资源

- [OpenAI Function Calling 文档](https://platform.openai.com/docs/guides/function-calling)
- [Anthropic Tool Use 文档](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- [Google Gemini Function Calling](https://ai.google.dev/gemini-api/docs/function-calling)

---

**@author erik.zhou**
