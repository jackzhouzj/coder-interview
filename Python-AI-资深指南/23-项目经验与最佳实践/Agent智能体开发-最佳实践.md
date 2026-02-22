# Agent 智能体开发最佳实践

> 基于 LangChain、LangGraph、AutoGPT 等框架的 Agent 开发实战经验
> 
> @author erik.zhou

## 📋 目录

- [1. Agent 架构设计最佳实践](#1-agent-架构设计最佳实践)
- [2. 工具调用与函数设计](#2-工具调用与函数设计)
- [3. 记忆管理策略](#3-记忆管理策略)
- [4. 多 Agent 协作模式](#4-多-agent-协作模式)
- [5. 错误处理与容错机制](#5-错误处理与容错机制)
- [6. 性能优化实战](#6-性能优化实战)
- [7. 安全与权限控制](#7-安全与权限控制)
- [8. 实战案例集](#8-实战案例集)

---

## 1. Agent 架构设计最佳实践

### 1.1 单 Agent 架构模式

#### ❌ 反模式：所有功能塞进一个 Agent

```python
# 不推荐：功能耦合严重，难以维护
class MonolithicAgent:
    def __init__(self):
        self.llm = ChatOpenAI(temperature=0)
        self.tools = [
            search_tool, calculator_tool, database_tool,
            email_tool, file_tool, api_tool, ...  # 工具过多
        ]
    
    def run(self, query: str):
        # 一个 Agent 处理所有任务
        return self.agent_executor.invoke({"input": query})
```

**问题**：
- 工具过多导致 LLM 选择困难
- 上下文窗口浪费严重
- 难以针对性优化
- 错误难以定位

#### ✅ 最佳实践：职责单一的 Agent

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder
from typing import List, Dict, Any

class ResearchAgent:
    """专注于信息检索和研究的 Agent"""
    
    def __init__(self, llm: ChatOpenAI, tools: List):
        self.llm = llm
        self.tools = tools  # 只包含搜索、爬虫等工具
        
        # 专门的 Prompt 设计
        self.prompt = ChatPromptTemplate.from_messages([
            ("system", """你是一个专业的研究助手。
            职责：
            1. 使用搜索工具查找信息
            2. 验证信息的可靠性
            3. 整理和总结研究结果
            
            限制：
            - 只使用提供的搜索和爬虫工具
            - 不要尝试执行代码或修改文件
            - 引用信息时必须标注来源
            """),
            MessagesPlaceholder(variable_name="chat_history", optional=True),
            ("human", "{input}"),
            MessagesPlaceholder(variable_name="agent_scratchpad")
        ])
        
        self.agent = create_openai_functions_agent(
            llm=self.llm,
            tools=self.tools,
            prompt=self.prompt
        )
        
        self.executor = AgentExecutor(
            agent=self.agent,
            tools=self.tools,
            verbose=True,
            max_iterations=5,  # 限制迭代次数
            handle_parsing_errors=True
        )
    
    def research(self, query: str, context: Dict[str, Any] = None) -> Dict[str, Any]:
        """执行研究任务"""
        try:
            result = self.executor.invoke({
                "input": query,
                "chat_history": context.get("history", []) if context else []
            })
            
            return {
                "success": True,
                "result": result["output"],
                "steps": result.get("intermediate_steps", []),
                "metadata": {
                    "iterations": len(result.get("intermediate_steps", [])),
                    "tools_used": self._extract_tools_used(result)
                }
            }
        except Exception as e:
            return {
                "success": False,
                "error": str(e),
                "fallback": "请尝试更具体的查询"
            }
    
    def _extract_tools_used(self, result: Dict) -> List[str]:
        """提取使用的工具列表"""
        tools = set()
        for step in result.get("intermediate_steps", []):
            if hasattr(step[0], 'tool'):
                tools.add(step[0].tool)
        return list(tools)


class CodeAgent:
    """专注于代码生成和执行的 Agent"""
    
    def __init__(self, llm: ChatOpenAI, tools: List):
        self.llm = llm
        self.tools = tools  # 只包含代码执行、文件操作工具
        
        self.prompt = ChatPromptTemplate.from_messages([
            ("system", """你是一个专业的代码助手。
            职责：
            1. 生成高质量的代码
            2. 执行代码并处理结果
            3. 进行代码审查和优化建议
            
            安全规范：
            - 执行代码前必须检查安全性
            - 不执行危险操作（删除文件、网络请求等）
            - 代码必须有异常处理
            """),
            MessagesPlaceholder(variable_name="chat_history", optional=True),
            ("human", "{input}"),
            MessagesPlaceholder(variable_name="agent_scratchpad")
        ])
        
        self.agent = create_openai_functions_agent(
            llm=self.llm,
            tools=self.tools,
            prompt=self.prompt
        )
        
        self.executor = AgentExecutor(
            agent=self.agent,
            tools=self.tools,
            verbose=True,
            max_iterations=3,
            handle_parsing_errors=True
        )
    
    def generate_and_execute(self, task: str) -> Dict[str, Any]:
        """生成并执行代码"""
        result = self.executor.invoke({"input": task})
        return {
            "code": self._extract_code(result),
            "output": result["output"],
            "success": True
        }
    
    def _extract_code(self, result: Dict) -> str:
        """从结果中提取代码"""
        # 实现代码提取逻辑
        pass
```

**优势**：
- 每个 Agent 职责清晰
- 工具集精简，提高准确率
- 便于独立测试和优化
- 错误隔离，不影响其他功能

### 1.2 Agent 配置管理

#### ✅ 使用配置文件管理 Agent 参数

```python
from dataclasses import dataclass
from typing import Optional, List, Dict
import yaml
from pathlib import Path

@dataclass
class AgentConfig:
    """Agent 配置类"""
    name: str
    model: str = "gpt-4"
    temperature: float = 0.0
    max_tokens: Optional[int] = None
    max_iterations: int = 5
    timeout: int = 300  # 秒
    tools: List[str] = None
    system_prompt: str = ""
    memory_type: str = "buffer"  # buffer, summary, vector
    memory_k: int = 10
    enable_streaming: bool = False
    retry_config: Dict = None
    
    def __post_init__(self):
        if self.tools is None:
            self.tools = []
        if self.retry_config is None:
            self.retry_config = {
                "max_retries": 3,
                "backoff_factor": 2,
                "retry_on": ["rate_limit", "timeout"]
            }


class AgentConfigManager:
    """Agent 配置管理器"""
    
    def __init__(self, config_path: str = "config/agents.yaml"):
        self.config_path = Path(config_path)
        self.configs: Dict[str, AgentConfig] = {}
        self._load_configs()
    
    def _load_configs(self):
        """加载配置文件"""
        if not self.config_path.exists():
            raise FileNotFoundError(f"配置文件不存在: {self.config_path}")
        
        with open(self.config_path, 'r', encoding='utf-8') as f:
            data = yaml.safe_load(f)
        
        for agent_name, config_dict in data.get('agents', {}).items():
            self.configs[agent_name] = AgentConfig(
                name=agent_name,
                **config_dict
            )
    
    def get_config(self, agent_name: str) -> AgentConfig:
        """获取指定 Agent 的配置"""
        if agent_name not in self.configs:
            raise ValueError(f"Agent 配置不存在: {agent_name}")
        return self.configs[agent_name]
    
    def update_config(self, agent_name: str, **kwargs):
        """动态更新配置"""
        if agent_name in self.configs:
            config = self.configs[agent_name]
            for key, value in kwargs.items():
                if hasattr(config, key):
                    setattr(config, key, value)
    
    def save_configs(self):
        """保存配置到文件"""
        data = {
            'agents': {
                name: {
                    k: v for k, v in config.__dict__.items()
                    if k != 'name'
                }
                for name, config in self.configs.items()
            }
        }
        
        with open(self.config_path, 'w', encoding='utf-8') as f:
            yaml.dump(data, f, allow_unicode=True, default_flow_style=False)


# 配置文件示例: config/agents.yaml
"""
agents:
  research_agent:
    model: gpt-4
    temperature: 0.0
    max_iterations: 5
    timeout: 300
    tools:
      - search
      - web_scraper
      - wikipedia
    system_prompt: |
      你是一个专业的研究助手...
    memory_type: vector
    memory_k: 20
    enable_streaming: true
    retry_config:
      max_retries: 3
      backoff_factor: 2
      retry_on:
        - rate_limit
        - timeout
  
  code_agent:
    model: gpt-4
    temperature: 0.2
    max_iterations: 3
    timeout: 180
    tools:
      - python_repl
      - file_manager
    system_prompt: |
      你是一个专业的代码助手...
    memory_type: buffer
    memory_k: 10
"""

# 使用示例
config_manager = AgentConfigManager("config/agents.yaml")
research_config = config_manager.get_config("research_agent")

# 根据配置创建 Agent
llm = ChatOpenAI(
    model=research_config.model,
    temperature=research_config.temperature,
    max_tokens=research_config.max_tokens
)
```



---

## 2. 工具调用与函数设计

### 2.1 工具设计原则

#### ✅ 最佳实践：清晰的工具定义

```python
from langchain.tools import BaseTool
from pydantic import BaseModel, Field
from typing import Optional, Type
import logging

logger = logging.getLogger(__name__)


class SearchInput(BaseModel):
    """搜索工具的输入模型"""
    query: str = Field(description="搜索查询关键词，应该简洁明确")
    num_results: int = Field(
        default=5,
        description="返回结果数量，默认5条",
        ge=1,
        le=10
    )
    language: str = Field(
        default="zh",
        description="搜索语言，zh表示中文，en表示英文"
    )


class SearchTool(BaseTool):
    """专业的搜索工具"""
    
    name: str = "web_search"
    description: str = """
    使用此工具在互联网上搜索信息。
    
    适用场景：
    - 需要最新的信息或数据
    - 查找特定主题的资料
    - 验证事实和数据
    
    不适用场景：
    - 已知的常识性问题
    - 需要深度分析的问题（应该先搜索再分析）
    
    使用建议：
    - 使用具体的关键词，避免模糊查询
    - 如果第一次搜索结果不理想，尝试换个关键词
    """
    
    args_schema: Type[BaseModel] = SearchInput
    return_direct: bool = False  # 不直接返回，让 Agent 处理结果
    
    def _run(self, query: str, num_results: int = 5, language: str = "zh") -> str:
        """执行搜索"""
        try:
            logger.info(f"执行搜索: query={query}, num_results={num_results}")
            
            # 实际搜索逻辑
            results = self._perform_search(query, num_results, language)
            
            # 格式化结果
            formatted_results = self._format_results(results)
            
            logger.info(f"搜索完成，返回 {len(results)} 条结果")
            return formatted_results
            
        except Exception as e:
            logger.error(f"搜索失败: {str(e)}")
            return f"搜索失败: {str(e)}。请尝试使用不同的关键词。"
    
    async def _arun(self, query: str, num_results: int = 5, language: str = "zh") -> str:
        """异步执行搜索"""
        # 实现异步搜索逻辑
        pass
    
    def _perform_search(self, query: str, num_results: int, language: str) -> list:
        """实际的搜索实现"""
        # 这里接入真实的搜索 API
        pass
    
    def _format_results(self, results: list) -> str:
        """格式化搜索结果"""
        if not results:
            return "未找到相关结果"
        
        formatted = "搜索结果：\n\n"
        for i, result in enumerate(results, 1):
            formatted += f"{i}. {result['title']}\n"
            formatted += f"   来源: {result['url']}\n"
            formatted += f"   摘要: {result['snippet']}\n\n"
        
        return formatted


class CalculatorInput(BaseModel):
    """计算器工具的输入模型"""
    expression: str = Field(
        description="要计算的数学表达式，例如: '2 + 2', '10 * 5 + 3'"
    )


class CalculatorTool(BaseTool):
    """安全的计算器工具"""
    
    name: str = "calculator"
    description: str = """
    用于执行数学计算。
    
    支持的操作：
    - 基本运算: +, -, *, /
    - 幂运算: **
    - 括号: ()
    
    示例：
    - "2 + 2" -> 4
    - "(10 + 5) * 2" -> 30
    - "2 ** 3" -> 8
    
    注意：只能计算数学表达式，不能执行代码。
    """
    
    args_schema: Type[BaseModel] = CalculatorInput
    
    def _run(self, expression: str) -> str:
        """执行计算"""
        try:
            # 安全检查：只允许数学表达式
            if not self._is_safe_expression(expression):
                return "错误：表达式包含不安全的内容"
            
            # 执行计算
            result = eval(expression, {"__builtins__": {}}, {})
            
            logger.info(f"计算: {expression} = {result}")
            return f"计算结果: {result}"
            
        except ZeroDivisionError:
            return "错误：除数不能为零"
        except SyntaxError:
            return "错误：表达式语法错误"
        except Exception as e:
            return f"计算失败: {str(e)}"
    
    def _is_safe_expression(self, expression: str) -> bool:
        """检查表达式是否安全"""
        # 只允许数字、运算符和括号
        allowed_chars = set("0123456789+-*/().** ")
        return all(c in allowed_chars for c in expression)


class DatabaseQueryInput(BaseModel):
    """数据库查询工具的输入模型"""
    query: str = Field(description="SQL 查询语句，只支持 SELECT")
    database: str = Field(
        default="default",
        description="数据库名称"
    )


class DatabaseQueryTool(BaseTool):
    """安全的数据库查询工具"""
    
    name: str = "database_query"
    description: str = """
    执行数据库查询（只读）。
    
    限制：
    - 只支持 SELECT 查询
    - 不支持 UPDATE, DELETE, DROP 等写操作
    - 查询结果最多返回 100 条
    
    使用建议：
    - 先了解表结构再查询
    - 使用 LIMIT 限制结果数量
    """
    
    args_schema: Type[BaseModel] = DatabaseQueryInput
    
    def _run(self, query: str, database: str = "default") -> str:
        """执行数据库查询"""
        try:
            # 安全检查
            if not self._is_safe_query(query):
                return "错误：只支持 SELECT 查询"
            
            # 执行查询
            results = self._execute_query(query, database)
            
            # 格式化结果
            return self._format_query_results(results)
            
        except Exception as e:
            logger.error(f"数据库查询失败: {str(e)}")
            return f"查询失败: {str(e)}"
    
    def _is_safe_query(self, query: str) -> bool:
        """检查查询是否安全"""
        query_upper = query.upper().strip()
        
        # 只允许 SELECT
        if not query_upper.startswith("SELECT"):
            return False
        
        # 禁止的关键词
        forbidden_keywords = [
            "UPDATE", "DELETE", "DROP", "INSERT",
            "ALTER", "CREATE", "TRUNCATE", "EXEC"
        ]
        
        return not any(keyword in query_upper for keyword in forbidden_keywords)
    
    def _execute_query(self, query: str, database: str) -> list:
        """执行查询"""
        # 实现数据库查询逻辑
        pass
    
    def _format_query_results(self, results: list) -> str:
        """格式化查询结果"""
        if not results:
            return "查询结果为空"
        
        # 限制返回数量
        results = results[:100]
        
        # 格式化为表格
        formatted = f"查询返回 {len(results)} 条结果：\n\n"
        # 实现表格格式化
        return formatted
```

### 2.2 工具组合与链式调用

#### ✅ 最佳实践：工具编排

```python
from typing import List, Dict, Any, Callable
from langchain.tools import Tool
import asyncio

class ToolOrchestrator:
    """工具编排器"""
    
    def __init__(self, tools: List[BaseTool]):
        self.tools = {tool.name: tool for tool in tools}
        self.execution_history: List[Dict] = []
    
    def create_workflow(self, steps: List[Dict[str, Any]]) -> Callable:
        """创建工具执行工作流"""
        
        def workflow(input_data: Dict[str, Any]) -> Dict[str, Any]:
            context = {"input": input_data, "results": {}}
            
            for step in steps:
                tool_name = step["tool"]
                params = step.get("params", {})
                
                # 支持参数引用前面步骤的结果
                resolved_params = self._resolve_params(params, context)
                
                # 执行工具
                result = self._execute_tool(tool_name, resolved_params)
                
                # 保存结果
                step_name = step.get("name", tool_name)
                context["results"][step_name] = result
                
                # 记录执行历史
                self.execution_history.append({
                    "step": step_name,
                    "tool": tool_name,
                    "params": resolved_params,
                    "result": result
                })
                
                # 检查是否需要提前终止
                if step.get("break_on_error") and not result.get("success"):
                    break
            
            return context["results"]
        
        return workflow
    
    def _resolve_params(self, params: Dict, context: Dict) -> Dict:
        """解析参数，支持引用前面步骤的结果"""
        resolved = {}
        for key, value in params.items():
            if isinstance(value, str) and value.startswith("$"):
                # 引用格式: $step_name.field
                ref_path = value[1:].split(".")
                resolved[key] = self._get_nested_value(
                    context["results"],
                    ref_path
                )
            else:
                resolved[key] = value
        return resolved
    
    def _get_nested_value(self, data: Dict, path: List[str]) -> Any:
        """获取嵌套字典的值"""
        current = data
        for key in path:
            current = current.get(key)
            if current is None:
                break
        return current
    
    def _execute_tool(self, tool_name: str, params: Dict) -> Dict[str, Any]:
        """执行工具"""
        if tool_name not in self.tools:
            return {"success": False, "error": f"工具不存在: {tool_name}"}
        
        tool = self.tools[tool_name]
        try:
            result = tool.run(**params)
            return {"success": True, "data": result}
        except Exception as e:
            return {"success": False, "error": str(e)}


# 使用示例：创建研究工作流
orchestrator = ToolOrchestrator([
    SearchTool(),
    WebScraperTool(),
    SummarizerTool()
])

research_workflow = orchestrator.create_workflow([
    {
        "name": "search",
        "tool": "web_search",
        "params": {
            "query": "$input.topic",
            "num_results": 5
        }
    },
    {
        "name": "scrape",
        "tool": "web_scraper",
        "params": {
            "urls": "$search.data.urls"
        },
        "break_on_error": False
    },
    {
        "name": "summarize",
        "tool": "summarizer",
        "params": {
            "text": "$scrape.data.content",
            "max_length": 500
        }
    }
])

# 执行工作流
result = research_workflow({
    "topic": "Python Agent 开发最佳实践"
})
```

### 2.3 工具错误处理

#### ✅ 最佳实践：优雅的错误处理

```python
from functools import wraps
from typing import Callable, Any
import time
from tenacity import (
    retry,
    stop_after_attempt,
    wait_exponential,
    retry_if_exception_type
)

class ToolError(Exception):
    """工具执行错误基类"""
    pass

class ToolTimeoutError(ToolError):
    """工具执行超时"""
    pass

class ToolRateLimitError(ToolError):
    """工具速率限制"""
    pass

class ToolValidationError(ToolError):
    """工具参数验证错误"""
    pass


def with_retry(
    max_attempts: int = 3,
    backoff_factor: float = 2.0,
    retry_on: tuple = (ToolTimeoutError, ToolRateLimitError)
):
    """工具重试装饰器"""
    
    def decorator(func: Callable) -> Callable:
        @retry(
            stop=stop_after_attempt(max_attempts),
            wait=wait_exponential(multiplier=backoff_factor),
            retry=retry_if_exception_type(retry_on),
            reraise=True
        )
        @wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            return func(*args, **kwargs)
        
        return wrapper
    
    return decorator


def with_timeout(seconds: int):
    """工具超时装饰器"""
    
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            import signal
            
            def timeout_handler(signum, frame):
                raise ToolTimeoutError(f"工具执行超时（{seconds}秒）")
            
            # 设置超时信号
            signal.signal(signal.SIGALRM, timeout_handler)
            signal.alarm(seconds)
            
            try:
                result = func(*args, **kwargs)
                signal.alarm(0)  # 取消超时
                return result
            except ToolTimeoutError:
                raise
            finally:
                signal.alarm(0)
        
        return wrapper
    
    return decorator


def with_fallback(fallback_func: Callable):
    """工具降级装饰器"""
    
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            try:
                return func(*args, **kwargs)
            except Exception as e:
                logger.warning(f"工具执行失败，使用降级方案: {str(e)}")
                return fallback_func(*args, **kwargs)
        
        return wrapper
    
    return decorator


class RobustSearchTool(BaseTool):
    """带完善错误处理的搜索工具"""
    
    name: str = "robust_search"
    description: str = "可靠的搜索工具，支持重试和降级"
    
    @with_retry(max_attempts=3, backoff_factor=2.0)
    @with_timeout(seconds=30)
    def _run(self, query: str, num_results: int = 5) -> str:
        """执行搜索"""
        try:
            # 参数验证
            self._validate_params(query, num_results)
            
            # 执行搜索
            results = self._perform_search(query, num_results)
            
            # 结果验证
            if not results:
                return self._handle_empty_results(query)
            
            return self._format_results(results)
            
        except ToolValidationError as e:
            return f"参数错误: {str(e)}"
        except ToolRateLimitError as e:
            logger.warning(f"触发速率限制: {str(e)}")
            raise  # 重试
        except ToolTimeoutError as e:
            logger.error(f"搜索超时: {str(e)}")
            return self._get_cached_results(query)  # 使用缓存
        except Exception as e:
            logger.error(f"搜索失败: {str(e)}")
            return f"搜索服务暂时不可用，请稍后重试"
    
    def _validate_params(self, query: str, num_results: int):
        """验证参数"""
        if not query or not query.strip():
            raise ToolValidationError("查询关键词不能为空")
        
        if num_results < 1 or num_results > 10:
            raise ToolValidationError("结果数量必须在 1-10 之间")
    
    def _handle_empty_results(self, query: str) -> str:
        """处理空结果"""
        suggestions = self._get_search_suggestions(query)
        return f"未找到相关结果。建议尝试：{', '.join(suggestions)}"
    
    def _get_cached_results(self, query: str) -> str:
        """获取缓存结果"""
        # 实现缓存逻辑
        pass
    
    def _get_search_suggestions(self, query: str) -> List[str]:
        """获取搜索建议"""
        # 实现搜索建议逻辑
        pass
```



---

## 3. 记忆管理策略

### 3.1 记忆类型选择

#### ✅ 最佳实践：根据场景选择记忆类型

```python
from langchain.memory import (
    ConversationBufferMemory,
    ConversationSummaryMemory,
    ConversationBufferWindowMemory,
    ConversationTokenBufferMemory,
    VectorStoreRetrieverMemory
)
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain.vectorstores import Chroma
from typing import Dict, Any, List
import json

class MemoryManager:
    """统一的记忆管理器"""
    
    def __init__(self, memory_type: str = "buffer", **kwargs):
        self.memory_type = memory_type
        self.memory = self._create_memory(memory_type, **kwargs)
    
    def _create_memory(self, memory_type: str, **kwargs):
        """根据类型创建记忆"""
        
        if memory_type == "buffer":
            # 适用场景：短对话，需要完整上下文
            return ConversationBufferMemory(
                memory_key="chat_history",
                return_messages=True,
                output_key="output"
            )
        
        elif memory_type == "window":
            # 适用场景：中等长度对话，只需要最近几轮
            return ConversationBufferWindowMemory(
                memory_key="chat_history",
                k=kwargs.get("window_size", 5),  # 保留最近5轮对话
                return_messages=True,
                output_key="output"
            )
        
        elif memory_type == "token":
            # 适用场景：需要精确控制 token 数量
            return ConversationTokenBufferMemory(
                llm=ChatOpenAI(temperature=0),
                memory_key="chat_history",
                max_token_limit=kwargs.get("max_tokens", 2000),
                return_messages=True,
                output_key="output"
            )
        
        elif memory_type == "summary":
            # 适用场景：长对话，需要压缩历史
            return ConversationSummaryMemory(
                llm=ChatOpenAI(temperature=0),
                memory_key="chat_history",
                return_messages=True,
                output_key="output"
            )
        
        elif memory_type == "vector":
            # 适用场景：需要语义检索历史对话
            vectorstore = Chroma(
                collection_name="conversation_history",
                embedding_function=OpenAIEmbeddings()
            )
            
            return VectorStoreRetrieverMemory(
                retriever=vectorstore.as_retriever(
                    search_kwargs={"k": kwargs.get("k", 5)}
                ),
                memory_key="chat_history",
                return_messages=True
            )
        
        else:
            raise ValueError(f"不支持的记忆类型: {memory_type}")
    
    def save_context(self, inputs: Dict[str, Any], outputs: Dict[str, Any]):
        """保存对话上下文"""
        self.memory.save_context(inputs, outputs)
    
    def load_memory_variables(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        """加载记忆变量"""
        return self.memory.load_memory_variables(inputs)
    
    def clear(self):
        """清空记忆"""
        self.memory.clear()


class HybridMemory:
    """混合记忆策略：结合多种记忆类型"""
    
    def __init__(
        self,
        short_term_window: int = 5,
        long_term_k: int = 3,
        summary_interval: int = 10
    ):
        # 短期记忆：最近几轮对话
        self.short_term = ConversationBufferWindowMemory(
            memory_key="recent_history",
            k=short_term_window,
            return_messages=True
        )
        
        # 长期记忆：向量检索
        vectorstore = Chroma(
            collection_name="long_term_memory",
            embedding_function=OpenAIEmbeddings()
        )
        self.long_term = VectorStoreRetrieverMemory(
            retriever=vectorstore.as_retriever(search_kwargs={"k": long_term_k}),
            memory_key="relevant_history",
            return_messages=True
        )
        
        # 摘要记忆：定期总结
        self.summary = ConversationSummaryMemory(
            llm=ChatOpenAI(temperature=0),
            memory_key="conversation_summary",
            return_messages=False
        )
        
        self.summary_interval = summary_interval
        self.turn_count = 0
    
    def save_context(self, inputs: Dict[str, Any], outputs: Dict[str, Any]):
        """保存到所有记忆"""
        # 保存到短期记忆
        self.short_term.save_context(inputs, outputs)
        
        # 保存到长期记忆
        self.long_term.save_context(inputs, outputs)
        
        # 定期更新摘要
        self.turn_count += 1
        if self.turn_count % self.summary_interval == 0:
            self.summary.save_context(inputs, outputs)
    
    def load_memory_variables(self, inputs: Dict[str, Any]) -> Dict[str, Any]:
        """加载所有记忆"""
        memory_vars = {}
        
        # 加载短期记忆
        memory_vars.update(self.short_term.load_memory_variables(inputs))
        
        # 加载相关的长期记忆
        memory_vars.update(self.long_term.load_memory_variables(inputs))
        
        # 加载摘要
        memory_vars.update(self.summary.load_memory_variables(inputs))
        
        return memory_vars
    
    def clear(self):
        """清空所有记忆"""
        self.short_term.clear()
        self.long_term.clear()
        self.summary.clear()
        self.turn_count = 0


# 使用示例
class ConversationalAgent:
    """带记忆的对话 Agent"""
    
    def __init__(self, memory_type: str = "hybrid"):
        self.llm = ChatOpenAI(temperature=0)
        
        # 根据场景选择记忆类型
        if memory_type == "hybrid":
            self.memory = HybridMemory(
                short_term_window=5,
                long_term_k=3,
                summary_interval=10
            )
        else:
            self.memory = MemoryManager(memory_type)
        
        self.agent = self._create_agent()
    
    def _create_agent(self):
        """创建 Agent"""
        # Agent 创建逻辑
        pass
    
    def chat(self, user_input: str) -> str:
        """对话"""
        # 加载记忆
        memory_vars = self.memory.load_memory_variables({"input": user_input})
        
        # 执行 Agent
        response = self.agent.run(
            input=user_input,
            **memory_vars
        )
        
        # 保存对话
        self.memory.save_context(
            {"input": user_input},
            {"output": response}
        )
        
        return response
```

### 3.2 记忆持久化

#### ✅ 最佳实践：记忆的保存与恢复

```python
from datetime import datetime
import pickle
from pathlib import Path
from typing import Optional
import redis
import json

class MemoryPersistence:
    """记忆持久化管理"""
    
    def __init__(self, storage_type: str = "file", **kwargs):
        self.storage_type = storage_type
        self.storage = self._create_storage(storage_type, **kwargs)
    
    def _create_storage(self, storage_type: str, **kwargs):
        """创建存储后端"""
        if storage_type == "file":
            return FileStorage(kwargs.get("base_path", "./memory"))
        elif storage_type == "redis":
            return RedisStorage(
                host=kwargs.get("host", "localhost"),
                port=kwargs.get("port", 6379),
                db=kwargs.get("db", 0)
            )
        else:
            raise ValueError(f"不支持的存储类型: {storage_type}")
    
    def save_memory(self, session_id: str, memory: Any):
        """保存记忆"""
        self.storage.save(session_id, memory)
    
    def load_memory(self, session_id: str) -> Optional[Any]:
        """加载记忆"""
        return self.storage.load(session_id)
    
    def delete_memory(self, session_id: str):
        """删除记忆"""
        self.storage.delete(session_id)
    
    def list_sessions(self) -> List[str]:
        """列出所有会话"""
        return self.storage.list_sessions()


class FileStorage:
    """文件存储"""
    
    def __init__(self, base_path: str):
        self.base_path = Path(base_path)
        self.base_path.mkdir(parents=True, exist_ok=True)
    
    def save(self, session_id: str, memory: Any):
        """保存到文件"""
        file_path = self.base_path / f"{session_id}.pkl"
        
        data = {
            "session_id": session_id,
            "memory": memory,
            "timestamp": datetime.now().isoformat(),
            "version": "1.0"
        }
        
        with open(file_path, 'wb') as f:
            pickle.dump(data, f)
    
    def load(self, session_id: str) -> Optional[Any]:
        """从文件加载"""
        file_path = self.base_path / f"{session_id}.pkl"
        
        if not file_path.exists():
            return None
        
        with open(file_path, 'rb') as f:
            data = pickle.load(f)
        
        return data["memory"]
    
    def delete(self, session_id: str):
        """删除文件"""
        file_path = self.base_path / f"{session_id}.pkl"
        if file_path.exists():
            file_path.unlink()
    
    def list_sessions(self) -> List[str]:
        """列出所有会话"""
        return [
            f.stem for f in self.base_path.glob("*.pkl")
        ]


class RedisStorage:
    """Redis 存储"""
    
    def __init__(self, host: str, port: int, db: int):
        self.redis_client = redis.Redis(
            host=host,
            port=port,
            db=db,
            decode_responses=False
        )
        self.key_prefix = "agent_memory:"
    
    def save(self, session_id: str, memory: Any):
        """保存到 Redis"""
        key = f"{self.key_prefix}{session_id}"
        
        data = {
            "session_id": session_id,
            "memory": memory,
            "timestamp": datetime.now().isoformat()
        }
        
        # 序列化并保存
        serialized = pickle.dumps(data)
        self.redis_client.set(key, serialized)
        
        # 设置过期时间（7天）
        self.redis_client.expire(key, 7 * 24 * 3600)
    
    def load(self, session_id: str) -> Optional[Any]:
        """从 Redis 加载"""
        key = f"{self.key_prefix}{session_id}"
        
        serialized = self.redis_client.get(key)
        if not serialized:
            return None
        
        data = pickle.loads(serialized)
        return data["memory"]
    
    def delete(self, session_id: str):
        """从 Redis 删除"""
        key = f"{self.key_prefix}{session_id}"
        self.redis_client.delete(key)
    
    def list_sessions(self) -> List[str]:
        """列出所有会话"""
        pattern = f"{self.key_prefix}*"
        keys = self.redis_client.keys(pattern)
        return [
            key.decode().replace(self.key_prefix, "")
            for key in keys
        ]


class StatefulAgent:
    """带状态持久化的 Agent"""
    
    def __init__(
        self,
        session_id: str,
        storage_type: str = "redis",
        **storage_kwargs
    ):
        self.session_id = session_id
        self.persistence = MemoryPersistence(storage_type, **storage_kwargs)
        
        # 尝试恢复记忆
        saved_memory = self.persistence.load_memory(session_id)
        if saved_memory:
            self.memory = saved_memory
            logger.info(f"恢复会话 {session_id} 的记忆")
        else:
            self.memory = HybridMemory()
            logger.info(f"创建新会话 {session_id}")
        
        self.llm = ChatOpenAI(temperature=0)
        self.agent = self._create_agent()
    
    def _create_agent(self):
        """创建 Agent"""
        # Agent 创建逻辑
        pass
    
    def chat(self, user_input: str) -> str:
        """对话"""
        # 执行对话
        response = self._execute_chat(user_input)
        
        # 保存记忆
        self.persistence.save_memory(self.session_id, self.memory)
        
        return response
    
    def _execute_chat(self, user_input: str) -> str:
        """执行对话逻辑"""
        memory_vars = self.memory.load_memory_variables({"input": user_input})
        
        response = self.agent.run(
            input=user_input,
            **memory_vars
        )
        
        self.memory.save_context(
            {"input": user_input},
            {"output": response}
        )
        
        return response
    
    def clear_history(self):
        """清空历史"""
        self.memory.clear()
        self.persistence.delete_memory(self.session_id)
    
    def export_history(self) -> Dict[str, Any]:
        """导出对话历史"""
        return {
            "session_id": self.session_id,
            "history": self.memory.load_memory_variables({}),
            "exported_at": datetime.now().isoformat()
        }


# 使用示例
# 创建或恢复会话
agent = StatefulAgent(
    session_id="user_123",
    storage_type="redis",
    host="localhost",
    port=6379
)

# 对话
response = agent.chat("你好，我想了解 Python Agent 开发")
print(response)

# 导出历史
history = agent.export_history()
```



---

## 4. 多 Agent 协作模式

### 4.1 层级协作模式

#### ✅ 最佳实践：Manager-Worker 模式

```python
from typing import List, Dict, Any, Optional
from langchain.agents import AgentExecutor
from langchain_openai import ChatOpenAI
from enum import Enum
import asyncio

class AgentRole(Enum):
    """Agent 角色"""
    MANAGER = "manager"
    RESEARCHER = "researcher"
    CODER = "coder"
    REVIEWER = "reviewer"
    WRITER = "writer"


class Task:
    """任务定义"""
    
    def __init__(
        self,
        task_id: str,
        description: str,
        assigned_to: AgentRole,
        dependencies: List[str] = None,
        priority: int = 0
    ):
        self.task_id = task_id
        self.description = description
        self.assigned_to = assigned_to
        self.dependencies = dependencies or []
        self.priority = priority
        self.status = "pending"  # pending, running, completed, failed
        self.result: Optional[Any] = None
        self.error: Optional[str] = None


class ManagerAgent:
    """管理者 Agent：负责任务分解和协调"""
    
    def __init__(self, worker_agents: Dict[AgentRole, Any]):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.worker_agents = worker_agents
        self.task_queue: List[Task] = []
        self.completed_tasks: Dict[str, Task] = {}
    
    def plan_tasks(self, user_request: str) -> List[Task]:
        """将用户请求分解为子任务"""
        
        planning_prompt = f"""
        用户请求：{user_request}
        
        请将这个请求分解为具体的子任务，每个任务应该：
        1. 有明确的目标和产出
        2. 分配给合适的专业 Agent
        3. 标注任务依赖关系
        
        可用的 Agent 角色：
        - researcher: 信息检索和研究
        - coder: 代码编写和执行
        - reviewer: 代码审查和质量检查
        - writer: 文档编写和总结
        
        请以 JSON 格式返回任务列表。
        """
        
        response = self.llm.invoke(planning_prompt)
        tasks_data = self._parse_tasks(response.content)
        
        tasks = []
        for i, task_data in enumerate(tasks_data):
            task = Task(
                task_id=f"task_{i+1}",
                description=task_data["description"],
                assigned_to=AgentRole(task_data["agent"]),
                dependencies=task_data.get("dependencies", []),
                priority=task_data.get("priority", 0)
            )
            tasks.append(task)
        
        return tasks
    
    def execute_plan(self, tasks: List[Task]) -> Dict[str, Any]:
        """执行任务计划"""
        self.task_queue = sorted(tasks, key=lambda t: t.priority, reverse=True)
        
        while self.task_queue:
            # 找到可以执行的任务（依赖已完成）
            executable_tasks = self._get_executable_tasks()
            
            if not executable_tasks:
                # 检查是否有循环依赖
                if self.task_queue:
                    raise RuntimeError("检测到循环依赖或无法执行的任务")
                break
            
            # 并行执行任务
            results = asyncio.run(self._execute_tasks_parallel(executable_tasks))
            
            # 处理结果
            for task, result in zip(executable_tasks, results):
                if result["success"]:
                    task.status = "completed"
                    task.result = result["data"]
                    self.completed_tasks[task.task_id] = task
                else:
                    task.status = "failed"
                    task.error = result["error"]
                    # 决定是否重试或终止
                    if not self._should_retry(task):
                        raise RuntimeError(f"任务失败: {task.task_id}")
                
                self.task_queue.remove(task)
        
        # 汇总结果
        return self._summarize_results()
    
    def _get_executable_tasks(self) -> List[Task]:
        """获取可执行的任务"""
        executable = []
        for task in self.task_queue:
            if task.status == "pending":
                # 检查依赖是否都已完成
                deps_completed = all(
                    dep_id in self.completed_tasks
                    for dep_id in task.dependencies
                )
                if deps_completed:
                    executable.append(task)
        return executable
    
    async def _execute_tasks_parallel(self, tasks: List[Task]) -> List[Dict[str, Any]]:
        """并行执行多个任务"""
        async def execute_single_task(task: Task):
            agent = self.worker_agents[task.assigned_to]
            
            # 准备上下文（包含依赖任务的结果）
            context = {
                dep_id: self.completed_tasks[dep_id].result
                for dep_id in task.dependencies
            }
            
            try:
                result = await agent.execute_async(task.description, context)
                return {"success": True, "data": result}
            except Exception as e:
                return {"success": False, "error": str(e)}
        
        return await asyncio.gather(*[
            execute_single_task(task) for task in tasks
        ])
    
    def _should_retry(self, task: Task) -> bool:
        """判断是否应该重试失败的任务"""
        # 实现重试逻辑
        return False
    
    def _summarize_results(self) -> Dict[str, Any]:
        """汇总所有任务结果"""
        summary_prompt = f"""
        以下是完成的所有任务及其结果：
        
        {self._format_completed_tasks()}
        
        请提供一个综合性的总结。
        """
        
        summary = self.llm.invoke(summary_prompt)
        
        return {
            "summary": summary.content,
            "tasks": [
                {
                    "task_id": task.task_id,
                    "description": task.description,
                    "status": task.status,
                    "result": task.result
                }
                for task in self.completed_tasks.values()
            ]
        }
    
    def _format_completed_tasks(self) -> str:
        """格式化已完成的任务"""
        formatted = []
        for task in self.completed_tasks.values():
            formatted.append(
                f"任务 {task.task_id}: {task.description}\n"
                f"结果: {task.result}\n"
            )
        return "\n".join(formatted)
    
    def _parse_tasks(self, response: str) -> List[Dict]:
        """解析 LLM 返回的任务列表"""
        # 实现 JSON 解析逻辑
        import json
        try:
            return json.loads(response)
        except:
            # 处理解析错误
            return []


class WorkerAgent:
    """工作者 Agent 基类"""
    
    def __init__(self, role: AgentRole, tools: List):
        self.role = role
        self.llm = ChatOpenAI(temperature=0)
        self.tools = tools
        self.agent = self._create_agent()
    
    def _create_agent(self):
        """创建 Agent"""
        # 实现 Agent 创建逻辑
        pass
    
    async def execute_async(self, task: str, context: Dict[str, Any]) -> Any:
        """异步执行任务"""
        # 准备输入
        input_data = {
            "task": task,
            "context": context
        }
        
        # 执行
        result = await self.agent.ainvoke(input_data)
        return result["output"]


# 使用示例：构建多 Agent 系统
class MultiAgentSystem:
    """多 Agent 协作系统"""
    
    def __init__(self):
        # 创建各个专业 Agent
        self.agents = {
            AgentRole.RESEARCHER: WorkerAgent(
                role=AgentRole.RESEARCHER,
                tools=[SearchTool(), WebScraperTool()]
            ),
            AgentRole.CODER: WorkerAgent(
                role=AgentRole.CODER,
                tools=[PythonREPLTool(), FileManagerTool()]
            ),
            AgentRole.REVIEWER: WorkerAgent(
                role=AgentRole.REVIEWER,
                tools=[CodeAnalysisTool(), LinterTool()]
            ),
            AgentRole.WRITER: WorkerAgent(
                role=AgentRole.WRITER,
                tools=[DocumentGeneratorTool()]
            )
        }
        
        # 创建管理者 Agent
        self.manager = ManagerAgent(self.agents)
    
    def execute(self, user_request: str) -> Dict[str, Any]:
        """执行用户请求"""
        # 1. 任务规划
        tasks = self.manager.plan_tasks(user_request)
        
        # 2. 执行任务
        result = self.manager.execute_plan(tasks)
        
        return result


# 使用示例
system = MultiAgentSystem()
result = system.execute(
    "研究 Python 异步编程的最佳实践，编写示例代码，并生成技术文档"
)
print(result["summary"])
```

### 4.2 对等协作模式

#### ✅ 最佳实践：Peer-to-Peer 协作

```python
from typing import List, Dict, Any, Callable
from dataclasses import dataclass
from queue import Queue
import threading

@dataclass
class Message:
    """Agent 间的消息"""
    sender: str
    receiver: str
    content: Any
    message_type: str  # request, response, broadcast
    metadata: Dict[str, Any] = None


class MessageBus:
    """消息总线：Agent 间通信"""
    
    def __init__(self):
        self.subscribers: Dict[str, List[Callable]] = {}
        self.message_queue = Queue()
        self.running = False
    
    def subscribe(self, agent_id: str, callback: Callable):
        """订阅消息"""
        if agent_id not in self.subscribers:
            self.subscribers[agent_id] = []
        self.subscribers[agent_id].append(callback)
    
    def publish(self, message: Message):
        """发布消息"""
        self.message_queue.put(message)
    
    def start(self):
        """启动消息处理"""
        self.running = True
        threading.Thread(target=self._process_messages, daemon=True).start()
    
    def stop(self):
        """停止消息处理"""
        self.running = False
    
    def _process_messages(self):
        """处理消息队列"""
        while self.running:
            try:
                message = self.message_queue.get(timeout=1)
                self._deliver_message(message)
            except:
                continue
    
    def _deliver_message(self, message: Message):
        """投递消息"""
        if message.message_type == "broadcast":
            # 广播给所有订阅者
            for callbacks in self.subscribers.values():
                for callback in callbacks:
                    callback(message)
        else:
            # 投递给特定接收者
            if message.receiver in self.subscribers:
                for callback in self.subscribers[message.receiver]:
                    callback(message)


class CollaborativeAgent:
    """协作型 Agent"""
    
    def __init__(
        self,
        agent_id: str,
        role: str,
        message_bus: MessageBus,
        tools: List
    ):
        self.agent_id = agent_id
        self.role = role
        self.message_bus = message_bus
        self.tools = tools
        self.llm = ChatOpenAI(temperature=0)
        
        # 订阅消息
        self.message_bus.subscribe(self.agent_id, self.handle_message)
        
        # 消息处理器
        self.message_handlers = {
            "request": self.handle_request,
            "response": self.handle_response,
            "broadcast": self.handle_broadcast
        }
    
    def handle_message(self, message: Message):
        """处理接收到的消息"""
        handler = self.message_handlers.get(message.message_type)
        if handler:
            handler(message)
    
    def handle_request(self, message: Message):
        """处理请求消息"""
        request = message.content
        
        # 执行任务
        result = self.execute_task(request)
        
        # 发送响应
        response = Message(
            sender=self.agent_id,
            receiver=message.sender,
            content=result,
            message_type="response",
            metadata={"request_id": message.metadata.get("request_id")}
        )
        self.message_bus.publish(response)
    
    def handle_response(self, message: Message):
        """处理响应消息"""
        # 处理其他 Agent 的响应
        pass
    
    def handle_broadcast(self, message: Message):
        """处理广播消息"""
        # 处理广播消息
        pass
    
    def execute_task(self, task: str) -> Any:
        """执行任务"""
        # 使用 LLM 和工具执行任务
        prompt = f"""
        你是一个 {self.role} Agent。
        任务：{task}
        
        如果需要其他 Agent 的帮助，可以发送请求消息。
        """
        
        result = self.llm.invoke(prompt)
        return result.content
    
    def request_help(self, target_agent: str, request: str) -> Any:
        """请求其他 Agent 帮助"""
        import uuid
        request_id = str(uuid.uuid4())
        
        message = Message(
            sender=self.agent_id,
            receiver=target_agent,
            content=request,
            message_type="request",
            metadata={"request_id": request_id}
        )
        
        self.message_bus.publish(message)
        
        # 等待响应（简化版，实际应该用异步）
        # 返回响应结果
        pass
    
    def broadcast(self, content: Any):
        """广播消息"""
        message = Message(
            sender=self.agent_id,
            receiver="all",
            content=content,
            message_type="broadcast"
        )
        self.message_bus.publish(message)


# 使用示例：创建协作系统
class PeerToPeerSystem:
    """对等协作系统"""
    
    def __init__(self):
        self.message_bus = MessageBus()
        self.message_bus.start()
        
        # 创建多个协作 Agent
        self.agents = {
            "researcher": CollaborativeAgent(
                agent_id="researcher",
                role="研究员",
                message_bus=self.message_bus,
                tools=[SearchTool()]
            ),
            "analyst": CollaborativeAgent(
                agent_id="analyst",
                role="分析师",
                message_bus=self.message_bus,
                tools=[DataAnalysisTool()]
            ),
            "writer": CollaborativeAgent(
                agent_id="writer",
                role="作家",
                message_bus=self.message_bus,
                tools=[DocumentTool()]
            )
        }
    
    def execute_collaborative_task(self, task: str):
        """执行协作任务"""
        # 广播任务给所有 Agent
        broadcast_message = Message(
            sender="system",
            receiver="all",
            content={"task": task, "action": "start"},
            message_type="broadcast"
        )
        self.message_bus.publish(broadcast_message)
    
    def shutdown(self):
        """关闭系统"""
        self.message_bus.stop()


# 使用示例
system = PeerToPeerSystem()
system.execute_collaborative_task("分析 2024 年 AI 发展趋势并撰写报告")
```

### 4.3 竞争协作模式

#### ✅ 最佳实践：多 Agent 投票机制

```python
from typing import List, Dict, Any, Tuple
from collections import Counter
import numpy as np

class VotingSystem:
    """多 Agent 投票系统"""
    
    def __init__(self, agents: List[Any], voting_method: str = "majority"):
        self.agents = agents
        self.voting_method = voting_method
    
    def get_consensus(self, task: str) -> Dict[str, Any]:
        """获取多个 Agent 的共识"""
        
        # 1. 收集所有 Agent 的答案
        responses = []
        for agent in self.agents:
            response = agent.execute(task)
            responses.append(response)
        
        # 2. 根据投票方法决定最终答案
        if self.voting_method == "majority":
            final_answer = self._majority_vote(responses)
        elif self.voting_method == "weighted":
            final_answer = self._weighted_vote(responses)
        elif self.voting_method == "consensus":
            final_answer = self._build_consensus(responses)
        else:
            raise ValueError(f"不支持的投票方法: {self.voting_method}")
        
        return {
            "final_answer": final_answer,
            "all_responses": responses,
            "confidence": self._calculate_confidence(responses, final_answer)
        }
    
    def _majority_vote(self, responses: List[str]) -> str:
        """多数投票"""
        counter = Counter(responses)
        most_common = counter.most_common(1)[0]
        return most_common[0]
    
    def _weighted_vote(self, responses: List[Dict[str, Any]]) -> str:
        """加权投票"""
        weighted_scores = {}
        
        for response in responses:
            answer = response["answer"]
            weight = response.get("confidence", 1.0)
            
            if answer not in weighted_scores:
                weighted_scores[answer] = 0
            weighted_scores[answer] += weight
        
        # 返回得分最高的答案
        best_answer = max(weighted_scores.items(), key=lambda x: x[1])
        return best_answer[0]
    
    def _build_consensus(self, responses: List[str]) -> str:
        """构建共识答案"""
        # 使用 LLM 综合多个答案
        llm = ChatOpenAI(temperature=0)
        
        prompt = f"""
        以下是多个 Agent 对同一问题的回答：
        
        {self._format_responses(responses)}
        
        请综合这些回答，给出一个最准确、最全面的答案。
        如果回答之间有冲突，请指出并给出你的判断。
        """
        
        consensus = llm.invoke(prompt)
        return consensus.content
    
    def _calculate_confidence(self, responses: List, final_answer: Any) -> float:
        """计算置信度"""
        if not responses:
            return 0.0
        
        # 计算与最终答案一致的比例
        matches = sum(1 for r in responses if r == final_answer)
        return matches / len(responses)
    
    def _format_responses(self, responses: List[str]) -> str:
        """格式化响应"""
        formatted = []
        for i, response in enumerate(responses, 1):
            formatted.append(f"Agent {i}: {response}")
        return "\n\n".join(formatted)


# 使用示例
class EnsembleAgentSystem:
    """集成 Agent 系统"""
    
    def __init__(self, num_agents: int = 3):
        # 创建多个独立的 Agent
        self.agents = [
            self._create_agent(f"agent_{i}")
            for i in range(num_agents)
        ]
        
        self.voting_system = VotingSystem(
            agents=self.agents,
            voting_method="consensus"
        )
    
    def _create_agent(self, agent_id: str):
        """创建单个 Agent"""
        return ResearchAgent(
            llm=ChatOpenAI(temperature=0.7),  # 不同温度增加多样性
            tools=[SearchTool(), CalculatorTool()]
        )
    
    def query(self, question: str) -> Dict[str, Any]:
        """查询并获取共识答案"""
        result = self.voting_system.get_consensus(question)
        
        return {
            "answer": result["final_answer"],
            "confidence": result["confidence"],
            "num_agents": len(self.agents),
            "all_responses": result["all_responses"]
        }


# 使用示例
ensemble = EnsembleAgentSystem(num_agents=5)
result = ensemble.query("Python 3.12 的主要新特性有哪些？")
print(f"答案: {result['answer']}")
print(f"置信度: {result['confidence']:.2%}")
```



---

## 5. 错误处理与容错机制

### 5.1 优雅降级策略

#### ✅ 最佳实践：多层降级方案

```python
from typing import Any, Callable, List, Optional
from functools import wraps
import logging

logger = logging.getLogger(__name__)


class FallbackChain:
    """降级链：按优先级尝试多个方案"""
    
    def __init__(self, strategies: List[Callable]):
        self.strategies = strategies
    
    def execute(self, *args, **kwargs) -> Any:
        """按顺序执行策略，直到成功"""
        errors = []
        
        for i, strategy in enumerate(self.strategies):
            try:
                logger.info(f"尝试策略 {i+1}/{len(self.strategies)}")
                result = strategy(*args, **kwargs)
                logger.info(f"策略 {i+1} 执行成功")
                return result
            except Exception as e:
                logger.warning(f"策略 {i+1} 失败: {str(e)}")
                errors.append(str(e))
                continue
        
        # 所有策略都失败
        raise RuntimeError(
            f"所有降级策略都失败了。错误: {'; '.join(errors)}"
        )


class ResilientAgent:
    """具有容错能力的 Agent"""
    
    def __init__(self):
        self.primary_llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.fallback_llm = ChatOpenAI(model="gpt-3.5-turbo", temperature=0)
        self.cache = {}
    
    def query_with_fallback(self, prompt: str) -> str:
        """带降级的查询"""
        
        # 定义降级策略
        fallback_chain = FallbackChain([
            lambda: self._query_primary(prompt),      # 策略1: 主模型
            lambda: self._query_fallback(prompt),     # 策略2: 备用模型
            lambda: self._query_cache(prompt),        # 策略3: 缓存
            lambda: self._query_simple_rules(prompt)  # 策略4: 规则引擎
        ])
        
        try:
            return fallback_chain.execute()
        except RuntimeError as e:
            # 最终降级：返回友好的错误消息
            logger.error(f"所有策略失败: {str(e)}")
            return "抱歉，服务暂时不可用，请稍后重试。"
    
    def _query_primary(self, prompt: str) -> str:
        """主模型查询"""
        response = self.primary_llm.invoke(prompt)
        return response.content
    
    def _query_fallback(self, prompt: str) -> str:
        """备用模型查询"""
        response = self.fallback_llm.invoke(prompt)
        return response.content
    
    def _query_cache(self, prompt: str) -> str:
        """从缓存获取"""
        if prompt in self.cache:
            logger.info("从缓存返回结果")
            return self.cache[prompt]
        raise ValueError("缓存未命中")
    
    def _query_simple_rules(self, prompt: str) -> str:
        """基于规则的简单回答"""
        # 实现简单的规则引擎
        if "你好" in prompt or "hello" in prompt.lower():
            return "你好！我是 AI 助手，有什么可以帮你的吗？"
        
        raise ValueError("无法使用规则引擎处理")


class CircuitBreaker:
    """熔断器：防止级联失败"""
    
    def __init__(
        self,
        failure_threshold: int = 5,
        timeout: int = 60,
        expected_exception: type = Exception
    ):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.expected_exception = expected_exception
        
        self.failure_count = 0
        self.last_failure_time = None
        self.state = "closed"  # closed, open, half_open
    
    def call(self, func: Callable, *args, **kwargs) -> Any:
        """通过熔断器调用函数"""
        
        if self.state == "open":
            if self._should_attempt_reset():
                self.state = "half_open"
            else:
                raise Exception("熔断器开启，服务不可用")
        
        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except self.expected_exception as e:
            self._on_failure()
            raise e
    
    def _on_success(self):
        """成功时重置计数"""
        self.failure_count = 0
        if self.state == "half_open":
            self.state = "closed"
            logger.info("熔断器关闭")
    
    def _on_failure(self):
        """失败时增加计数"""
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self.failure_count >= self.failure_threshold:
            self.state = "open"
            logger.warning(f"熔断器开启，失败次数: {self.failure_count}")
    
    def _should_attempt_reset(self) -> bool:
        """判断是否应该尝试重置"""
        return (
            self.last_failure_time is not None and
            time.time() - self.last_failure_time >= self.timeout
        )


# 使用示例
class ProductionAgent:
    """生产环境 Agent"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.circuit_breaker = CircuitBreaker(
            failure_threshold=5,
            timeout=60
        )
        self.fallback_responses = {
            "greeting": "你好！我是 AI 助手。",
            "error": "抱歉，我遇到了一些问题，请稍后重试。"
        }
    
    def chat(self, user_input: str) -> str:
        """带熔断器的对话"""
        try:
            # 通过熔断器调用 LLM
            response = self.circuit_breaker.call(
                self._call_llm,
                user_input
            )
            return response
        except Exception as e:
            logger.error(f"对话失败: {str(e)}")
            # 返回降级响应
            return self._get_fallback_response(user_input)
    
    def _call_llm(self, user_input: str) -> str:
        """调用 LLM"""
        response = self.llm.invoke(user_input)
        return response.content
    
    def _get_fallback_response(self, user_input: str) -> str:
        """获取降级响应"""
        # 简单的关键词匹配
        if any(word in user_input for word in ["你好", "hello", "hi"]):
            return self.fallback_responses["greeting"]
        return self.fallback_responses["error"]
```

### 5.2 重试机制

#### ✅ 最佳实践：智能重试

```python
import time
import random
from typing import Callable, Any, Type
from functools import wraps

class RetryStrategy:
    """重试策略基类"""
    
    def get_wait_time(self, attempt: int) -> float:
        """获取等待时间"""
        raise NotImplementedError


class ExponentialBackoff(RetryStrategy):
    """指数退避"""
    
    def __init__(self, base: float = 1.0, max_wait: float = 60.0):
        self.base = base
        self.max_wait = max_wait
    
    def get_wait_time(self, attempt: int) -> float:
        wait = min(self.base * (2 ** attempt), self.max_wait)
        # 添加随机抖动，避免惊群效应
        jitter = random.uniform(0, wait * 0.1)
        return wait + jitter


class LinearBackoff(RetryStrategy):
    """线性退避"""
    
    def __init__(self, increment: float = 1.0, max_wait: float = 30.0):
        self.increment = increment
        self.max_wait = max_wait
    
    def get_wait_time(self, attempt: int) -> float:
        return min(self.increment * attempt, self.max_wait)


def retry_with_strategy(
    max_attempts: int = 3,
    strategy: RetryStrategy = None,
    retry_on: tuple = (Exception,),
    on_retry: Callable = None
):
    """带策略的重试装饰器"""
    
    if strategy is None:
        strategy = ExponentialBackoff()
    
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs) -> Any:
            last_exception = None
            
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except retry_on as e:
                    last_exception = e
                    
                    if attempt < max_attempts - 1:
                        wait_time = strategy.get_wait_time(attempt)
                        logger.warning(
                            f"第 {attempt + 1} 次尝试失败: {str(e)}. "
                            f"等待 {wait_time:.2f} 秒后重试..."
                        )
                        
                        if on_retry:
                            on_retry(attempt, e)
                        
                        time.sleep(wait_time)
                    else:
                        logger.error(f"所有 {max_attempts} 次尝试都失败了")
            
            raise last_exception
        
        return wrapper
    
    return decorator


class SmartRetryAgent:
    """智能重试 Agent"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.retry_count = 0
    
    @retry_with_strategy(
        max_attempts=3,
        strategy=ExponentialBackoff(base=2.0, max_wait=30.0),
        retry_on=(RateLimitError, TimeoutError),
        on_retry=lambda attempt, error: logger.info(f"重试回调: {error}")
    )
    def query(self, prompt: str) -> str:
        """带重试的查询"""
        try:
            response = self.llm.invoke(prompt)
            return response.content
        except Exception as e:
            # 记录错误
            self.retry_count += 1
            logger.error(f"查询失败 (第 {self.retry_count} 次): {str(e)}")
            raise
    
    def query_with_adaptive_retry(self, prompt: str) -> str:
        """自适应重试：根据错误类型调整策略"""
        
        for attempt in range(3):
            try:
                return self.llm.invoke(prompt).content
            except RateLimitError as e:
                # 速率限制：使用指数退避
                wait_time = 2 ** attempt
                logger.warning(f"速率限制，等待 {wait_time} 秒")
                time.sleep(wait_time)
            except TimeoutError as e:
                # 超时：立即重试
                logger.warning("超时，立即重试")
                continue
            except ValidationError as e:
                # 验证错误：不重试
                logger.error(f"验证错误: {str(e)}")
                raise
        
        raise RuntimeError("所有重试都失败了")
```



---

## 6. 性能优化实战

### 6.1 并发执行优化

#### ✅ 最佳实践：异步并发处理

```python
import asyncio
from typing import List, Dict, Any, Coroutine
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import time

class AsyncAgent:
    """异步 Agent：提升并发性能"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.executor = ThreadPoolExecutor(max_workers=5)
    
    async def process_batch(self, queries: List[str]) -> List[str]:
        """批量处理查询"""
        # 创建异步任务
        tasks = [self.process_single(query) for query in queries]
        
        # 并发执行
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # 处理结果
        processed_results = []
        for i, result in enumerate(results):
            if isinstance(result, Exception):
                logger.error(f"查询 {i} 失败: {str(result)}")
                processed_results.append(f"处理失败: {str(result)}")
            else:
                processed_results.append(result)
        
        return processed_results
    
    async def process_single(self, query: str) -> str:
        """处理单个查询"""
        # 使用 asyncio 运行同步 LLM 调用
        loop = asyncio.get_event_loop()
        result = await loop.run_in_executor(
            self.executor,
            self._sync_process,
            query
        )
        return result
    
    def _sync_process(self, query: str) -> str:
        """同步处理逻辑"""
        response = self.llm.invoke(query)
        return response.content


class ParallelToolExecutor:
    """并行工具执行器"""
    
    def __init__(self, max_workers: int = 5):
        self.max_workers = max_workers
    
    async def execute_tools_parallel(
        self,
        tool_calls: List[Dict[str, Any]]
    ) -> List[Any]:
        """并行执行多个工具调用"""
        
        async def execute_single_tool(tool_call: Dict[str, Any]):
            tool_name = tool_call["tool"]
            params = tool_call["params"]
            
            # 模拟工具执行
            await asyncio.sleep(0.1)  # 模拟 I/O 操作
            return f"Tool {tool_name} result"
        
        # 创建任务
        tasks = [execute_single_tool(call) for call in tool_calls]
        
        # 并发执行，限制并发数
        semaphore = asyncio.Semaphore(self.max_workers)
        
        async def bounded_execute(task):
            async with semaphore:
                return await task
        
        results = await asyncio.gather(*[
            bounded_execute(task) for task in tasks
        ])
        
        return results


# 使用示例：批量处理
async def batch_processing_example():
    """批量处理示例"""
    agent = AsyncAgent()
    
    queries = [
        "什么是 Python?",
        "什么是 JavaScript?",
        "什么是 Go?",
        "什么是 Rust?",
        "什么是 Java?"
    ]
    
    start_time = time.time()
    results = await agent.process_batch(queries)
    end_time = time.time()
    
    print(f"处理 {len(queries)} 个查询耗时: {end_time - start_time:.2f} 秒")
    for i, result in enumerate(results):
        print(f"查询 {i+1}: {result[:100]}...")


# 运行示例
# asyncio.run(batch_processing_example())
```

### 6.2 缓存策略

#### ✅ 最佳实践：多层缓存

```python
from functools import lru_cache
from typing import Optional, Any
import hashlib
import json
import redis
from datetime import timedelta

class CacheManager:
    """多层缓存管理器"""
    
    def __init__(
        self,
        redis_host: str = "localhost",
        redis_port: int = 6379,
        enable_memory_cache: bool = True,
        enable_redis_cache: bool = True
    ):
        self.enable_memory_cache = enable_memory_cache
        self.enable_redis_cache = enable_redis_cache
        
        # Redis 缓存
        if enable_redis_cache:
            self.redis_client = redis.Redis(
                host=redis_host,
                port=redis_port,
                decode_responses=True
            )
        
        # 内存缓存（LRU）
        if enable_memory_cache:
            self._memory_cache = {}
            self._cache_size = 1000
    
    def get(self, key: str) -> Optional[Any]:
        """获取缓存"""
        # 1. 尝试从内存缓存获取
        if self.enable_memory_cache:
            if key in self._memory_cache:
                logger.debug(f"内存缓存命中: {key}")
                return self._memory_cache[key]
        
        # 2. 尝试从 Redis 获取
        if self.enable_redis_cache:
            value = self.redis_client.get(key)
            if value:
                logger.debug(f"Redis 缓存命中: {key}")
                # 反序列化
                data = json.loads(value)
                
                # 回填到内存缓存
                if self.enable_memory_cache:
                    self._set_memory_cache(key, data)
                
                return data
        
        return None
    
    def set(
        self,
        key: str,
        value: Any,
        ttl: Optional[int] = None
    ):
        """设置缓存"""
        # 1. 设置内存缓存
        if self.enable_memory_cache:
            self._set_memory_cache(key, value)
        
        # 2. 设置 Redis 缓存
        if self.enable_redis_cache:
            serialized = json.dumps(value, ensure_ascii=False)
            if ttl:
                self.redis_client.setex(key, ttl, serialized)
            else:
                self.redis_client.set(key, serialized)
    
    def _set_memory_cache(self, key: str, value: Any):
        """设置内存缓存（LRU）"""
        if len(self._memory_cache) >= self._cache_size:
            # 删除最旧的项
            oldest_key = next(iter(self._memory_cache))
            del self._memory_cache[oldest_key]
        
        self._memory_cache[key] = value
    
    def invalidate(self, key: str):
        """失效缓存"""
        if self.enable_memory_cache and key in self._memory_cache:
            del self._memory_cache[key]
        
        if self.enable_redis_cache:
            self.redis_client.delete(key)
    
    def clear_all(self):
        """清空所有缓存"""
        if self.enable_memory_cache:
            self._memory_cache.clear()
        
        if self.enable_redis_cache:
            # 注意：这会清空整个 Redis 数据库
            # 生产环境应该使用 key pattern 删除
            pass


class CachedAgent:
    """带缓存的 Agent"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.cache = CacheManager(
            enable_memory_cache=True,
            enable_redis_cache=True
        )
        self.cache_ttl = 3600  # 1小时
    
    def query(self, prompt: str, use_cache: bool = True) -> str:
        """带缓存的查询"""
        if not use_cache:
            return self._execute_query(prompt)
        
        # 生成缓存键
        cache_key = self._generate_cache_key(prompt)
        
        # 尝试从缓存获取
        cached_result = self.cache.get(cache_key)
        if cached_result:
            logger.info("使用缓存结果")
            return cached_result
        
        # 执行查询
        result = self._execute_query(prompt)
        
        # 保存到缓存
        self.cache.set(cache_key, result, ttl=self.cache_ttl)
        
        return result
    
    def _execute_query(self, prompt: str) -> str:
        """执行实际查询"""
        response = self.llm.invoke(prompt)
        return response.content
    
    def _generate_cache_key(self, prompt: str) -> str:
        """生成缓存键"""
        # 使用 prompt 的 hash 作为键
        prompt_hash = hashlib.md5(prompt.encode()).hexdigest()
        return f"agent:query:{prompt_hash}"
    
    def invalidate_cache(self, prompt: str):
        """失效特定查询的缓存"""
        cache_key = self._generate_cache_key(prompt)
        self.cache.invalidate(cache_key)


class SemanticCache:
    """语义缓存：基于相似度的缓存"""
    
    def __init__(self, similarity_threshold: float = 0.9):
        self.embeddings = OpenAIEmbeddings()
        self.vectorstore = Chroma(
            collection_name="semantic_cache",
            embedding_function=self.embeddings
        )
        self.similarity_threshold = similarity_threshold
    
    def get(self, query: str) -> Optional[str]:
        """获取语义相似的缓存结果"""
        # 搜索相似查询
        results = self.vectorstore.similarity_search_with_score(
            query,
            k=1
        )
        
        if results:
            doc, score = results[0]
            # 相似度足够高
            if score >= self.similarity_threshold:
                logger.info(f"语义缓存命中，相似度: {score:.2f}")
                return doc.metadata.get("result")
        
        return None
    
    def set(self, query: str, result: str):
        """保存查询和结果"""
        self.vectorstore.add_texts(
            texts=[query],
            metadatas=[{"result": result}]
        )


# 使用示例
agent = CachedAgent()

# 第一次查询（无缓存）
result1 = agent.query("Python 的主要特点是什么？")

# 第二次相同查询（使用缓存）
result2 = agent.query("Python 的主要特点是什么？")

# 失效缓存
agent.invalidate_cache("Python 的主要特点是什么？")
```

### 6.3 Prompt 优化

#### ✅ 最佳实践：高效 Prompt 设计

```python
from typing import Dict, Any, List
from string import Template

class PromptOptimizer:
    """Prompt 优化器"""
    
    def __init__(self):
        self.templates = {}
        self._load_templates()
    
    def _load_templates(self):
        """加载 Prompt 模板"""
        self.templates = {
            "research": Template("""
你是一个专业的研究助手。

任务：$task

要求：
1. 使用提供的工具进行信息检索
2. 验证信息的准确性
3. 提供信息来源

可用工具：$tools

请开始执行任务。
"""),
            
            "code_generation": Template("""
你是一个专业的代码生成助手。

需求：$requirement

技术栈：$tech_stack

要求：
1. 代码必须符合 PEP 8 规范
2. 添加必要的注释和文档字符串
3. 包含错误处理
4. 提供使用示例

请生成代码。
"""),
            
            "summarization": Template("""
请总结以下内容：

$content

要求：
- 长度：$max_length 字以内
- 保留关键信息
- 使用简洁的语言

总结：
""")
        }
    
    def optimize_prompt(
        self,
        template_name: str,
        variables: Dict[str, Any],
        compression: bool = False
    ) -> str:
        """优化 Prompt"""
        
        # 1. 使用模板
        if template_name not in self.templates:
            raise ValueError(f"模板不存在: {template_name}")
        
        template = self.templates[template_name]
        prompt = template.safe_substitute(**variables)
        
        # 2. 压缩（如果需要）
        if compression:
            prompt = self._compress_prompt(prompt)
        
        # 3. 验证长度
        token_count = self._estimate_tokens(prompt)
        if token_count > 4000:
            logger.warning(f"Prompt 过长: {token_count} tokens")
            prompt = self._truncate_prompt(prompt, max_tokens=4000)
        
        return prompt
    
    def _compress_prompt(self, prompt: str) -> str:
        """压缩 Prompt"""
        # 移除多余的空白
        lines = [line.strip() for line in prompt.split('\n')]
        lines = [line for line in lines if line]
        return '\n'.join(lines)
    
    def _estimate_tokens(self, text: str) -> int:
        """估算 token 数量"""
        # 简单估算：1 token ≈ 4 字符
        return len(text) // 4
    
    def _truncate_prompt(self, prompt: str, max_tokens: int) -> str:
        """截断 Prompt"""
        max_chars = max_tokens * 4
        if len(prompt) <= max_chars:
            return prompt
        
        # 保留开头和结尾，截断中间
        head_size = max_chars // 2
        tail_size = max_chars - head_size
        
        return (
            prompt[:head_size] +
            "\n...[内容已截断]...\n" +
            prompt[-tail_size:]
        )


class FewShotPromptBuilder:
    """Few-Shot Prompt 构建器"""
    
    def __init__(self):
        self.examples = []
    
    def add_example(self, input_text: str, output_text: str):
        """添加示例"""
        self.examples.append({
            "input": input_text,
            "output": output_text
        })
    
    def build_prompt(
        self,
        task: str,
        num_examples: int = 3
    ) -> str:
        """构建 Few-Shot Prompt"""
        
        # 选择最相关的示例
        selected_examples = self._select_examples(task, num_examples)
        
        # 构建 Prompt
        prompt_parts = ["以下是一些示例：\n"]
        
        for i, example in enumerate(selected_examples, 1):
            prompt_parts.append(f"示例 {i}:")
            prompt_parts.append(f"输入: {example['input']}")
            prompt_parts.append(f"输出: {example['output']}\n")
        
        prompt_parts.append(f"现在请处理以下任务：\n{task}")
        
        return "\n".join(prompt_parts)
    
    def _select_examples(
        self,
        task: str,
        num_examples: int
    ) -> List[Dict[str, str]]:
        """选择最相关的示例"""
        # 简化版：随机选择
        # 实际应该基于相似度选择
        import random
        return random.sample(
            self.examples,
            min(num_examples, len(self.examples))
        )


# 使用示例
optimizer = PromptOptimizer()

# 优化研究任务的 Prompt
research_prompt = optimizer.optimize_prompt(
    template_name="research",
    variables={
        "task": "研究 Python 3.12 的新特性",
        "tools": "search, web_scraper"
    },
    compression=True
)

print(research_prompt)
```



---

## 7. 安全与权限控制

### 7.1 输入验证与清洗

#### ✅ 最佳实践：严格的输入验证

```python
from pydantic import BaseModel, Field, validator
from typing import Optional, List
import re

class UserInput(BaseModel):
    """用户输入验证模型"""
    
    query: str = Field(..., min_length=1, max_length=1000)
    context: Optional[str] = Field(None, max_length=5000)
    tools: Optional[List[str]] = Field(default=None)
    
    @validator('query')
    def validate_query(cls, v):
        """验证查询内容"""
        # 检查是否包含危险字符
        dangerous_patterns = [
            r'<script',
            r'javascript:',
            r'eval\(',
            r'exec\(',
            r'__import__'
        ]
        
        for pattern in dangerous_patterns:
            if re.search(pattern, v, re.IGNORECASE):
                raise ValueError(f"查询包含危险内容: {pattern}")
        
        return v.strip()
    
    @validator('tools')
    def validate_tools(cls, v):
        """验证工具列表"""
        if v is None:
            return v
        
        allowed_tools = {'search', 'calculator', 'weather', 'translator'}
        
        for tool in v:
            if tool not in allowed_tools:
                raise ValueError(f"不允许的工具: {tool}")
        
        return v


class SecureAgent:
    """安全的 Agent"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.allowed_tools = {'search', 'calculator'}
    
    def process(self, user_input: dict) -> str:
        """处理用户输入"""
        try:
            # 验证输入
            validated_input = UserInput(**user_input)
            
            # 清洗输入
            clean_query = self._sanitize_input(validated_input.query)
            
            # 执行查询
            result = self._execute_query(clean_query)
            
            # 清洗输出
            clean_result = self._sanitize_output(result)
            
            return clean_result
            
        except ValueError as e:
            logger.warning(f"输入验证失败: {str(e)}")
            return "输入不合法，请检查后重试"
    
    def _sanitize_input(self, text: str) -> str:
        """清洗输入"""
        # 移除 HTML 标签
        text = re.sub(r'<[^>]+>', '', text)
        
        # 移除特殊字符
        text = re.sub(r'[^\w\s\u4e00-\u9fff.,!?;:()（）。，！？；：]', '', text)
        
        return text.strip()
    
    def _sanitize_output(self, text: str) -> str:
        """清洗输出"""
        # 移除可能的敏感信息
        # 例如：API密钥、密码等
        patterns = [
            (r'sk-[a-zA-Z0-9]{48}', '[API_KEY]'),  # OpenAI API key
            (r'\b\d{16}\b', '[CARD_NUMBER]'),      # 信用卡号
            (r'\b\d{3}-\d{2}-\d{4}\b', '[SSN]')    # 社保号
        ]
        
        for pattern, replacement in patterns:
            text = re.sub(pattern, replacement, text)
        
        return text
    
    def _execute_query(self, query: str) -> str:
        """执行查询"""
        response = self.llm.invoke(query)
        return response.content
```

### 7.2 权限控制

#### ✅ 最佳实践：基于角色的访问控制（RBAC）

```python
from enum import Enum
from typing import Set, Dict, Callable
from functools import wraps

class Permission(Enum):
    """权限枚举"""
    READ = "read"
    WRITE = "write"
    EXECUTE = "execute"
    ADMIN = "admin"


class Role(Enum):
    """角色枚举"""
    GUEST = "guest"
    USER = "user"
    DEVELOPER = "developer"
    ADMIN = "admin"


class RBACManager:
    """基于角色的访问控制管理器"""
    
    def __init__(self):
        # 角色权限映射
        self.role_permissions: Dict[Role, Set[Permission]] = {
            Role.GUEST: {Permission.READ},
            Role.USER: {Permission.READ, Permission.WRITE},
            Role.DEVELOPER: {Permission.READ, Permission.WRITE, Permission.EXECUTE},
            Role.ADMIN: {Permission.READ, Permission.WRITE, Permission.EXECUTE, Permission.ADMIN}
        }
        
        # 工具权限要求
        self.tool_permissions: Dict[str, Set[Permission]] = {
            "search": {Permission.READ},
            "calculator": {Permission.READ},
            "file_write": {Permission.WRITE},
            "code_execute": {Permission.EXECUTE},
            "system_config": {Permission.ADMIN}
        }
    
    def has_permission(self, role: Role, permission: Permission) -> bool:
        """检查角色是否有权限"""
        return permission in self.role_permissions.get(role, set())
    
    def can_use_tool(self, role: Role, tool_name: str) -> bool:
        """检查角色是否可以使用工具"""
        required_permissions = self.tool_permissions.get(tool_name, set())
        user_permissions = self.role_permissions.get(role, set())
        
        return required_permissions.issubset(user_permissions)
    
    def get_allowed_tools(self, role: Role) -> List[str]:
        """获取角色允许使用的工具列表"""
        allowed_tools = []
        for tool_name in self.tool_permissions.keys():
            if self.can_use_tool(role, tool_name):
                allowed_tools.append(tool_name)
        return allowed_tools


def require_permission(permission: Permission):
    """权限检查装饰器"""
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(self, *args, **kwargs):
            if not hasattr(self, 'user_role'):
                raise PermissionError("未设置用户角色")
            
            rbac = RBACManager()
            if not rbac.has_permission(self.user_role, permission):
                raise PermissionError(
                    f"权限不足：需要 {permission.value} 权限"
                )
            
            return func(self, *args, **kwargs)
        return wrapper
    return decorator


class SecureAgentWithRBAC:
    """带权限控制的安全 Agent"""
    
    def __init__(self, user_role: Role):
        self.user_role = user_role
        self.rbac = RBACManager()
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        
        # 根据角色过滤可用工具
        self.available_tools = self._get_available_tools()
    
    def _get_available_tools(self) -> List:
        """获取用户可用的工具"""
        allowed_tool_names = self.rbac.get_allowed_tools(self.user_role)
        
        all_tools = {
            "search": SearchTool(),
            "calculator": CalculatorTool(),
            "file_write": FileWriteTool(),
            "code_execute": CodeExecuteTool(),
            "system_config": SystemConfigTool()
        }
        
        return [
            all_tools[name] for name in allowed_tool_names
            if name in all_tools
        ]
    
    @require_permission(Permission.READ)
    def query(self, prompt: str) -> str:
        """查询（需要读权限）"""
        response = self.llm.invoke(prompt)
        return response.content
    
    @require_permission(Permission.EXECUTE)
    def execute_code(self, code: str) -> str:
        """执行代码（需要执行权限）"""
        # 执行代码逻辑
        return "代码执行完成"
    
    @require_permission(Permission.ADMIN)
    def modify_config(self, config: dict) -> str:
        """修改配置（需要管理员权限）"""
        # 修改配置逻辑
        return "配置已更新"
    
    def use_tool(self, tool_name: str, **kwargs) -> Any:
        """使用工具（带权限检查）"""
        if not self.rbac.can_use_tool(self.user_role, tool_name):
            raise PermissionError(
                f"权限不足：无法使用工具 {tool_name}"
            )
        
        # 执行工具
        tool = next(
            (t for t in self.available_tools if t.name == tool_name),
            None
        )
        
        if tool is None:
            raise ValueError(f"工具不存在: {tool_name}")
        
        return tool.run(**kwargs)


# 使用示例
# 普通用户
user_agent = SecureAgentWithRBAC(user_role=Role.USER)
result = user_agent.query("什么是 Python?")  # 成功

try:
    user_agent.execute_code("print('hello')")  # 失败：权限不足
except PermissionError as e:
    print(f"错误: {e}")

# 开发者
dev_agent = SecureAgentWithRBAC(user_role=Role.DEVELOPER)
result = dev_agent.execute_code("print('hello')")  # 成功
```



---

## 8. 实战案例集

### 案例 1：智能客服 Agent

#### 场景描述
构建一个智能客服系统，能够回答常见问题、查询订单状态、处理退款申请。

#### 实现方案

```python
from langchain.agents import AgentExecutor, create_openai_functions_agent
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain.tools import Tool
from typing import Dict, Any

class CustomerServiceAgent:
    """智能客服 Agent"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.memory = HybridMemory()
        
        # 创建工具
        self.tools = [
            Tool(
                name="query_order",
                func=self.query_order,
                description="查询订单状态。输入：订单号"
            ),
            Tool(
                name="query_faq",
                func=self.query_faq,
                description="查询常见问题答案。输入：问题关键词"
            ),
            Tool(
                name="create_refund",
                func=self.create_refund,
                description="创建退款申请。输入：订单号和退款原因"
            )
        ]
        
        # 创建 Agent
        self.agent = self._create_agent()
    
    def _create_agent(self):
        """创建客服 Agent"""
        prompt = ChatPromptTemplate.from_messages([
            ("system", """你是一个专业的客服助手。

职责：
1. 友好、耐心地回答用户问题
2. 使用工具查询订单信息
3. 帮助用户解决问题
4. 必要时创建退款申请

规范：
- 始终保持礼貌和专业
- 如果不确定，告诉用户会转接人工客服
- 保护用户隐私，不泄露敏感信息
"""),
            MessagesPlaceholder(variable_name="chat_history", optional=True),
            ("human", "{input}"),
            MessagesPlaceholder(variable_name="agent_scratchpad")
        ])
        
        agent = create_openai_functions_agent(
            llm=self.llm,
            tools=self.tools,
            prompt=prompt
        )
        
        return AgentExecutor(
            agent=agent,
            tools=self.tools,
            verbose=True,
            max_iterations=5,
            handle_parsing_errors=True
        )
    
    def query_order(self, order_id: str) -> str:
        """查询订单状态"""
        # 模拟数据库查询
        orders = {
            "ORD001": {
                "status": "已发货",
                "tracking": "SF1234567890",
                "estimated_delivery": "2024-03-15"
            },
            "ORD002": {
                "status": "处理中",
                "tracking": None,
                "estimated_delivery": "2024-03-18"
            }
        }
        
        order = orders.get(order_id)
        if not order:
            return f"未找到订单 {order_id}"
        
        result = f"订单 {order_id} 状态：{order['status']}"
        if order['tracking']:
            result += f"\n物流单号：{order['tracking']}"
        result += f"\n预计送达：{order['estimated_delivery']}"
        
        return result
    
    def query_faq(self, keyword: str) -> str:
        """查询常见问题"""
        faqs = {
            "退货": "退货政策：商品签收后7天内可申请退货，商品需保持原包装完好。",
            "发票": "发票开具：订单完成后可在订单详情页申请电子发票。",
            "配送": "配送时效：一般3-5个工作日送达，偏远地区可能需要7-10天。"
        }
        
        for key, answer in faqs.items():
            if key in keyword:
                return answer
        
        return "未找到相关问题，建议联系人工客服。"
    
    def create_refund(self, order_id: str, reason: str) -> str:
        """创建退款申请"""
        # 模拟创建退款
        refund_id = f"REF{order_id}"
        
        return f"退款申请已创建，退款单号：{refund_id}。预计3-5个工作日处理完成。"
    
    def chat(self, user_input: str) -> str:
        """处理用户输入"""
        # 加载历史
        memory_vars = self.memory.load_memory_variables({"input": user_input})
        
        # 执行
        response = self.agent.invoke({
            "input": user_input,
            **memory_vars
        })
        
        # 保存历史
        self.memory.save_context(
            {"input": user_input},
            {"output": response["output"]}
        )
        
        return response["output"]


# 使用示例
cs_agent = CustomerServiceAgent()

# 对话1：查询订单
response1 = cs_agent.chat("我想查询订单 ORD001 的状态")
print(f"客服: {response1}\n")

# 对话2：询问退货政策
response2 = cs_agent.chat("请问退货政策是什么？")
print(f"客服: {response2}\n")

# 对话3：申请退款
response3 = cs_agent.chat("我要退货，订单号是 ORD001，商品有质量问题")
print(f"客服: {response3}\n")
```

### 案例 2：代码审查 Agent

#### 场景描述
自动审查代码，检查代码规范、潜在bug、性能问题，并提供改进建议。

#### 实现方案

```python
from typing import List, Dict, Any
import ast
import re

class CodeReviewAgent:
    """代码审查 Agent"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
        self.rules = self._load_review_rules()
    
    def _load_review_rules(self) -> Dict[str, Any]:
        """加载审查规则"""
        return {
            "naming": {
                "description": "命名规范检查",
                "patterns": {
                    "class": r"^[A-Z][a-zA-Z0-9]*$",
                    "function": r"^[a-z_][a-z0-9_]*$",
                    "constant": r"^[A-Z_][A-Z0-9_]*$"
                }
            },
            "complexity": {
                "description": "复杂度检查",
                "max_lines": 50,
                "max_params": 5,
                "max_nesting": 4
            },
            "security": {
                "description": "安全检查",
                "dangerous_functions": ["eval", "exec", "__import__"]
            }
        }
    
    def review_code(self, code: str, language: str = "python") -> Dict[str, Any]:
        """审查代码"""
        issues = []
        
        if language == "python":
            # 1. 语法检查
            syntax_issues = self._check_syntax(code)
            issues.extend(syntax_issues)
            
            # 2. 规范检查
            style_issues = self._check_style(code)
            issues.extend(style_issues)
            
            # 3. 复杂度检查
            complexity_issues = self._check_complexity(code)
            issues.extend(complexity_issues)
            
            # 4. 安全检查
            security_issues = self._check_security(code)
            issues.extend(security_issues)
            
            # 5. AI 深度分析
            ai_suggestions = self._ai_analysis(code)
            
            return {
                "issues": issues,
                "suggestions": ai_suggestions,
                "score": self._calculate_score(issues),
                "summary": self._generate_summary(issues, ai_suggestions)
            }
    
    def _check_syntax(self, code: str) -> List[Dict[str, Any]]:
        """检查语法"""
        issues = []
        try:
            ast.parse(code)
        except SyntaxError as e:
            issues.append({
                "type": "syntax",
                "severity": "error",
                "line": e.lineno,
                "message": f"语法错误: {e.msg}"
            })
        return issues
    
    def _check_style(self, code: str) -> List[Dict[str, Any]]:
        """检查代码风格"""
        issues = []
        
        # 检查命名规范
        tree = ast.parse(code)
        
        for node in ast.walk(tree):
            if isinstance(node, ast.ClassDef):
                if not re.match(self.rules["naming"]["patterns"]["class"], node.name):
                    issues.append({
                        "type": "naming",
                        "severity": "warning",
                        "line": node.lineno,
                        "message": f"类名 '{node.name}' 不符合命名规范（应使用 PascalCase）"
                    })
            
            elif isinstance(node, ast.FunctionDef):
                if not re.match(self.rules["naming"]["patterns"]["function"], node.name):
                    issues.append({
                        "type": "naming",
                        "severity": "warning",
                        "line": node.lineno,
                        "message": f"函数名 '{node.name}' 不符合命名规范（应使用 snake_case）"
                    })
        
        return issues
    
    def _check_complexity(self, code: str) -> List[Dict[str, Any]]:
        """检查复杂度"""
        issues = []
        tree = ast.parse(code)
        
        for node in ast.walk(tree):
            if isinstance(node, ast.FunctionDef):
                # 检查函数长度
                func_lines = node.end_lineno - node.lineno + 1
                if func_lines > self.rules["complexity"]["max_lines"]:
                    issues.append({
                        "type": "complexity",
                        "severity": "warning",
                        "line": node.lineno,
                        "message": f"函数 '{node.name}' 过长（{func_lines} 行），建议拆分"
                    })
                
                # 检查参数数量
                num_params = len(node.args.args)
                if num_params > self.rules["complexity"]["max_params"]:
                    issues.append({
                        "type": "complexity",
                        "severity": "warning",
                        "line": node.lineno,
                        "message": f"函数 '{node.name}' 参数过多（{num_params} 个），建议重构"
                    })
        
        return issues
    
    def _check_security(self, code: str) -> List[Dict[str, Any]]:
        """检查安全问题"""
        issues = []
        
        for func in self.rules["security"]["dangerous_functions"]:
            if func in code:
                issues.append({
                    "type": "security",
                    "severity": "error",
                    "message": f"使用了危险函数 '{func}'，存在安全风险"
                })
        
        return issues
    
    def _ai_analysis(self, code: str) -> List[str]:
        """AI 深度分析"""
        prompt = f"""
请审查以下 Python 代码，提供改进建议：

```python
{code}
```

请从以下方面分析：
1. 代码可读性
2. 性能优化
3. 最佳实践
4. 潜在bug

每条建议请简洁明了。
"""
        
        response = self.llm.invoke(prompt)
        
        # 解析建议
        suggestions = response.content.split('\n')
        suggestions = [s.strip() for s in suggestions if s.strip()]
        
        return suggestions
    
    def _calculate_score(self, issues: List[Dict[str, Any]]) -> int:
        """计算代码质量分数"""
        score = 100
        
        for issue in issues:
            if issue["severity"] == "error":
                score -= 10
            elif issue["severity"] == "warning":
                score -= 5
        
        return max(0, score)
    
    def _generate_summary(
        self,
        issues: List[Dict[str, Any]],
        suggestions: List[str]
    ) -> str:
        """生成审查总结"""
        error_count = sum(1 for i in issues if i["severity"] == "error")
        warning_count = sum(1 for i in issues if i["severity"] == "warning")
        
        summary = f"代码审查完成：\n"
        summary += f"- 错误: {error_count} 个\n"
        summary += f"- 警告: {warning_count} 个\n"
        summary += f"- AI 建议: {len(suggestions)} 条\n"
        
        return summary


# 使用示例
reviewer = CodeReviewAgent()

code_to_review = """
def calculateTotal(items):
    total = 0
    for item in items:
        if item['price'] > 0:
            if item['quantity'] > 0:
                if item['discount'] > 0:
                    total += item['price'] * item['quantity'] * (1 - item['discount'])
                else:
                    total += item['price'] * item['quantity']
    return total

class userService:
    def __init__(self):
        pass
"""

result = reviewer.review_code(code_to_review)

print(result["summary"])
print(f"\n代码质量分数: {result['score']}/100\n")

print("发现的问题:")
for issue in result["issues"]:
    print(f"- [{issue['severity'].upper()}] {issue['message']}")

print("\nAI 改进建议:")
for suggestion in result["suggestions"]:
    print(f"- {suggestion}")
```

### 案例 3：数据分析 Agent

#### 场景描述
自动分析数据集，生成统计报告、可视化图表和洞察建议。

#### 实现方案

```python
import pandas as pd
import matplotlib.pyplot as plt
from typing import Dict, Any, List

class DataAnalysisAgent:
    """数据分析 Agent"""
    
    def __init__(self):
        self.llm = ChatOpenAI(model="gpt-4", temperature=0)
    
    def analyze_dataset(self, df: pd.DataFrame) -> Dict[str, Any]:
        """分析数据集"""
        
        # 1. 基础统计
        basic_stats = self._get_basic_stats(df)
        
        # 2. 数据质量检查
        quality_report = self._check_data_quality(df)
        
        # 3. 相关性分析
        correlations = self._analyze_correlations(df)
        
        # 4. 异常检测
        anomalies = self._detect_anomalies(df)
        
        # 5. AI 洞察
        insights = self._generate_insights(df, basic_stats, correlations)
        
        # 6. 生成可视化
        visualizations = self._create_visualizations(df)
        
        return {
            "basic_stats": basic_stats,
            "quality_report": quality_report,
            "correlations": correlations,
            "anomalies": anomalies,
            "insights": insights,
            "visualizations": visualizations
        }
    
    def _get_basic_stats(self, df: pd.DataFrame) -> Dict[str, Any]:
        """获取基础统计信息"""
        return {
            "shape": df.shape,
            "columns": list(df.columns),
            "dtypes": df.dtypes.to_dict(),
            "summary": df.describe().to_dict(),
            "missing_values": df.isnull().sum().to_dict()
        }
    
    def _check_data_quality(self, df: pd.DataFrame) -> Dict[str, Any]:
        """检查数据质量"""
        issues = []
        
        # 检查缺失值
        missing = df.isnull().sum()
        for col, count in missing.items():
            if count > 0:
                percentage = (count / len(df)) * 100
                issues.append({
                    "type": "missing_values",
                    "column": col,
                    "count": int(count),
                    "percentage": round(percentage, 2)
                })
        
        # 检查重复行
        duplicates = df.duplicated().sum()
        if duplicates > 0:
            issues.append({
                "type": "duplicates",
                "count": int(duplicates)
            })
        
        return {
            "issues": issues,
            "quality_score": self._calculate_quality_score(issues, len(df))
        }
    
    def _analyze_correlations(self, df: pd.DataFrame) -> Dict[str, Any]:
        """分析相关性"""
        numeric_cols = df.select_dtypes(include=['number']).columns
        
        if len(numeric_cols) < 2:
            return {"message": "数值列不足，无法进行相关性分析"}
        
        corr_matrix = df[numeric_cols].corr()
        
        # 找出强相关的列对
        strong_correlations = []
        for i in range(len(corr_matrix.columns)):
            for j in range(i+1, len(corr_matrix.columns)):
                corr_value = corr_matrix.iloc[i, j]
                if abs(corr_value) > 0.7:
                    strong_correlations.append({
                        "col1": corr_matrix.columns[i],
                        "col2": corr_matrix.columns[j],
                        "correlation": round(corr_value, 3)
                    })
        
        return {
            "matrix": corr_matrix.to_dict(),
            "strong_correlations": strong_correlations
        }
    
    def _detect_anomalies(self, df: pd.DataFrame) -> List[Dict[str, Any]]:
        """检测异常值"""
        anomalies = []
        numeric_cols = df.select_dtypes(include=['number']).columns
        
        for col in numeric_cols:
            Q1 = df[col].quantile(0.25)
            Q3 = df[col].quantile(0.75)
            IQR = Q3 - Q1
            
            lower_bound = Q1 - 1.5 * IQR
            upper_bound = Q3 + 1.5 * IQR
            
            outliers = df[(df[col] < lower_bound) | (df[col] > upper_bound)]
            
            if len(outliers) > 0:
                anomalies.append({
                    "column": col,
                    "count": len(outliers),
                    "percentage": round((len(outliers) / len(df)) * 100, 2)
                })
        
        return anomalies
    
    def _generate_insights(
        self,
        df: pd.DataFrame,
        basic_stats: Dict,
        correlations: Dict
    ) -> List[str]:
        """生成数据洞察"""
        
        # 准备数据摘要
        summary = f"""
数据集概况：
- 行数: {basic_stats['shape'][0]}
- 列数: {basic_stats['shape'][1]}
- 缺失值: {sum(basic_stats['missing_values'].values())}

强相关性：
{correlations.get('strong_correlations', [])}

请基于以上信息，提供3-5条数据洞察和建议。
"""
        
        response = self.llm.invoke(summary)
        insights = response.content.split('\n')
        insights = [s.strip() for s in insights if s.strip()]
        
        return insights
    
    def _create_visualizations(self, df: pd.DataFrame) -> List[str]:
        """创建可视化"""
        viz_paths = []
        
        # 数值列分布图
        numeric_cols = df.select_dtypes(include=['number']).columns
        if len(numeric_cols) > 0:
            fig, axes = plt.subplots(1, min(3, len(numeric_cols)), figsize=(15, 5))
            if len(numeric_cols) == 1:
                axes = [axes]
            
            for i, col in enumerate(list(numeric_cols)[:3]):
                df[col].hist(ax=axes[i], bins=30)
                axes[i].set_title(f'{col} 分布')
            
            path = 'distribution.png'
            plt.savefig(path)
            plt.close()
            viz_paths.append(path)
        
        return viz_paths
    
    def _calculate_quality_score(self, issues: List, total_rows: int) -> int:
        """计算数据质量分数"""
        score = 100
        
        for issue in issues:
            if issue["type"] == "missing_values":
                score -= min(issue["percentage"], 20)
            elif issue["type"] == "duplicates":
                dup_percentage = (issue["count"] / total_rows) * 100
                score -= min(dup_percentage, 10)
        
        return max(0, int(score))


# 使用示例
analyst = DataAnalysisAgent()

# 创建示例数据
df = pd.DataFrame({
    'age': [25, 30, 35, 40, 45, 50, 55, 60],
    'income': [30000, 45000, 55000, 65000, 75000, 85000, 95000, 105000],
    'score': [70, 75, 80, 85, 90, 95, 88, 92]
})

# 分析数据
result = analyst.analyze_dataset(df)

print("数据质量分数:", result["quality_report"]["quality_score"])
print("\n数据洞察:")
for insight in result["insights"]:
    print(f"- {insight}")
```

---

## 总结

Agent 智能体开发是一个复杂但充满潜力的领域。通过遵循本文档中的最佳实践，你可以构建出高效、可靠、安全的 Agent 系统。

关键要点：
1. 职责单一的 Agent 设计
2. 完善的工具定义和错误处理
3. 合理的记忆管理策略
4. 灵活的多 Agent 协作模式
5. 严格的安全和权限控制
6. 持续的性能优化

记住：Agent 开发是一个迭代过程，需要不断测试、优化和改进。

---

@author erik.zhou
