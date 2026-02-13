# AutoGPT 完整教程

> @author erik.zhou

## 📋 目录

1. [AutoGPT 概述](#autogpt-概述)
2. [核心概念](#核心概念)
3. [环境搭建](#环境搭建)
4. [基础使用](#基础使用)
5. [高级功能](#高级功能)
6. [实战案例](#实战案例)

---

## AutoGPT 概述

### 什么是 AutoGPT

AutoGPT 是一个实验性的开源应用，展示了 GPT-4 的自主能力。它可以自主地完成复杂任务，无需人工干预。

### 核心特性

| 特性 | 说明 |
|------|------|
| 自主执行 | 自动分解任务并执行 |
| 长期记忆 | 使用向量数据库存储记忆 |
| 工具使用 | 可以使用各种工具和API |
| 自我反思 | 评估执行结果并调整策略 |

### AutoGPT vs 传统 Agent

```python
"""
传统 Agent：
- 需要明确的指令
- 单次交互
- 有限的上下文

AutoGPT：
- 自主目标分解
- 持续执行直到完成
- 长期记忆管理
- 自我评估和改进
"""
```

---

## 核心概念

### Agent 架构

```python
from typing import List, Dict, Optional
from dataclasses import dataclass
from datetime import datetime

@dataclass
class Task:
    """任务"""
    id: str
    description: str
    status: str = "pending"  # pending, in_progress, completed, failed
    priority: int = 0
    created_at: datetime = None
    
    def __post_init__(self):
        if self.created_at is None:
            self.created_at = datetime.now()

@dataclass
class Memory:
    """记忆"""
    content: str
    timestamp: datetime
    importance: float = 0.5
    
    def __post_init__(self):
        if not hasattr(self, 'timestamp'):
            self.timestamp = datetime.now()

class AutoGPTAgent:
    """AutoGPT Agent 基础架构"""
    
    def __init__(self, name: str, role: str, goal: str):
        self.name = name
        self.role = role
        self.goal = goal
        self.tasks: List[Task] = []
        self.memories: List[Memory] = []
        self.completed_tasks: List[Task] = []
    
    def add_task(self, description: str, priority: int = 0):
        """添加任务"""
        task = Task(
            id=f"TASK_{len(self.tasks) + 1}",
            description=description,
            priority=priority
        )
        self.tasks.append(task)
        print(f"添加任务: {task.description}")
    
    def get_next_task(self) -> Optional[Task]:
        """获取下一个任务（按优先级）"""
        if not self.tasks:
            return None
        
        # 按优先级排序
        self.tasks.sort(key=lambda t: t.priority, reverse=True)
        return self.tasks[0]
    
    def complete_task(self, task: Task, result: str):
        """完成任务"""
        task.status = "completed"
        self.tasks.remove(task)
        self.completed_tasks.append(task)
        
        # 存储记忆
        memory = Memory(
            content=f"完成任务: {task.description}. 结果: {result}",
            timestamp=datetime.now(),
            importance=0.8
        )
        self.memories.append(memory)
        
        print(f"任务完成: {task.description}")
    
    def reflect(self):
        """自我反思"""
        print(f"\n=== {self.name} 反思 ===")
        print(f"目标: {self.goal}")
        print(f"已完成任务: {len(self.completed_tasks)}")
        print(f"待完成任务: {len(self.tasks)}")
        print(f"记忆数量: {len(self.memories)}")

# 使用示例
print("=== AutoGPT Agent 架构 ===")
agent = AutoGPTAgent(
    name="研究助手",
    role="AI研究员",
    goal="研究最新的AI技术趋势"
)

agent.add_task("搜索最新的AI论文", priority=10)
agent.add_task("总结关键发现", priority=8)
agent.add_task("生成研究报告", priority=5)

# 执行任务
task = agent.get_next_task()
if task:
    agent.complete_task(task, "找到10篇相关论文")

agent.reflect()
```

### 目标分解

```python
class GoalDecomposer:
    """目标分解器"""
    
    def __init__(self):
        self.decomposition_history = []
    
    def decompose(self, goal: str) -> List[str]:
        """将目标分解为子任务"""
        print(f"\n分解目标: {goal}")
        
        # 简化的分解逻辑（实际应使用LLM）
        if "研究" in goal:
            subtasks = [
                "1. 确定研究主题和范围",
                "2. 搜索相关资料和论文",
                "3. 阅读和分析资料",
                "4. 总结关键发现",
                "5. 撰写研究报告"
            ]
        elif "开发" in goal:
            subtasks = [
                "1. 需求分析",
                "2. 技术选型",
                "3. 架构设计",
                "4. 编码实现",
                "5. 测试和部署"
            ]
        else:
            subtasks = [
                "1. 分析目标",
                "2. 制定计划",
                "3. 执行计划",
                "4. 评估结果"
            ]
        
        self.decomposition_history.append({
            'goal': goal,
            'subtasks': subtasks,
            'timestamp': datetime.now()
        })
        
        for task in subtasks:
            print(f"  - {task}")
        
        return subtasks

# 使用示例
print("\n=== 目标分解 ===")
decomposer = GoalDecomposer()
subtasks = decomposer.decompose("研究大语言模型的最新进展")
```

---

## 环境搭建

### 安装依赖

```bash
# 安装核心依赖
pip install openai
pip install langchain
pip install chromadb  # 向量数据库
pip install tiktoken  # Token计数

# 可选依赖
pip install playwright  # 网页浏览
pip install beautifulsoup4  # 网页解析
```

### 配置文件

```python
import os
from dataclasses import dataclass

@dataclass
class AutoGPTConfig:
    """AutoGPT 配置"""
    
    # OpenAI 配置
    openai_api_key: str = os.getenv("OPENAI_API_KEY", "")
    model: str = "gpt-4"
    temperature: float = 0.7
    max_tokens: int = 2000
    
    # 记忆配置
    memory_backend: str = "chromadb"  # local, chromadb, pinecone
    memory_index: str = "autogpt-memory"
    
    # Agent 配置
    max_iterations: int = 10
    max_tasks: int = 20
    
    # 工具配置
    enable_web_search: bool = True
    enable_file_operations: bool = True
    enable_code_execution: bool = False  # 安全考虑
    
    def validate(self):
        """验证配置"""
        if not self.openai_api_key:
            raise ValueError("OPENAI_API_KEY 未设置")
        
        if self.max_iterations < 1:
            raise ValueError("max_iterations 必须大于0")
        
        print("配置验证通过")

# 使用示例
config = AutoGPTConfig()
config.validate()
```

---

## 基础使用

### 简单的 AutoGPT 实现

```python
from typing import List, Callable
import time

class SimpleAutoGPT:
    """简化的 AutoGPT 实现"""
    
    def __init__(self, goal: str, max_iterations: int = 5):
        self.goal = goal
        self.max_iterations = max_iterations
        self.tasks: List[str] = []
        self.completed_tasks: List[str] = []
        self.iteration = 0
    
    def think(self) -> str:
        """思考下一步行动"""
        print(f"\n[思考] 迭代 {self.iteration + 1}/{self.max_iterations}")
        print(f"目标: {self.goal}")
        print(f"已完成: {len(self.completed_tasks)} 个任务")
        
        # 简化的思考逻辑
        if not self.tasks:
            return "decompose_goal"
        elif self.tasks:
            return "execute_task"
        else:
            return "complete"
    
    def decompose_goal(self):
        """分解目标"""
        print("\n[行动] 分解目标为子任务")
        
        # 简化的分解（实际应使用LLM）
        self.tasks = [
            "收集信息",
            "分析数据",
            "生成报告"
        ]
        
        for i, task in enumerate(self.tasks, 1):
            print(f"  {i}. {task}")
    
    def execute_task(self):
        """执行任务"""
        if not self.tasks:
            return
        
        task = self.tasks.pop(0)
        print(f"\n[行动] 执行任务: {task}")
        
        # 模拟任务执行
        time.sleep(0.5)
        result = f"{task} - 完成"
        
        self.completed_tasks.append(result)
        print(f"[结果] {result}")
    
    def run(self):
        """运行 Agent"""
        print(f"=== AutoGPT 开始执行 ===")
        print(f"目标: {self.goal}\n")
        
        while self.iteration < self.max_iterations:
            action = self.think()
            
            if action == "decompose_goal":
                self.decompose_goal()
            elif action == "execute_task":
                self.execute_task()
            elif action == "complete":
                print("\n[完成] 所有任务已完成")
                break
            
            self.iteration += 1
            time.sleep(0.3)
        
        self.summary()
    
    def summary(self):
        """总结"""
        print(f"\n=== 执行总结 ===")
        print(f"目标: {self.goal}")
        print(f"迭代次数: {self.iteration}")
        print(f"完成任务数: {len(self.completed_tasks)}")
        print("\n已完成的任务:")
        for i, task in enumerate(self.completed_tasks, 1):
            print(f"  {i}. {task}")

# 使用示例
print("\n=== 简单 AutoGPT 示例 ===")
agent = SimpleAutoGPT(
    goal="分析Python在AI领域的应用",
    max_iterations=5
)
agent.run()
```

---

## 高级功能

### 长期记忆管理

```python
from typing import List, Dict
import numpy as np

class MemoryManager:
    """记忆管理器"""
    
    def __init__(self, max_memories: int = 100):
        self.max_memories = max_memories
        self.short_term: List[Memory] = []
        self.long_term: List[Memory] = []
    
    def add_memory(self, content: str, importance: float = 0.5):
        """添加记忆"""
        memory = Memory(
            content=content,
            timestamp=datetime.now(),
            importance=importance
        )
        
        self.short_term.append(memory)
        
        # 如果短期记忆过多，转移到长期记忆
        if len(self.short_term) > 10:
            self._consolidate_memories()
    
    def _consolidate_memories(self):
        """整合记忆（短期 -> 长期）"""
        # 按重要性排序
        self.short_term.sort(key=lambda m: m.importance, reverse=True)
        
        # 保留重要的记忆
        important_memories = self.short_term[:5]
        self.long_term.extend(important_memories)
        
        # 清理短期记忆
        self.short_term = self.short_term[5:]
        
        # 限制长期记忆数量
        if len(self.long_term) > self.max_memories:
            self.long_term = self.long_term[-self.max_memories:]
        
        print(f"记忆整合: 短期={len(self.short_term)}, 长期={len(self.long_term)}")
    
    def recall(self, query: str, top_k: int = 5) -> List[Memory]:
        """回忆相关记忆"""
        # 简化的相似度计算（实际应使用向量相似度）
        all_memories = self.short_term + self.long_term
        
        # 按重要性和时间排序
        all_memories.sort(
            key=lambda m: (m.importance, m.timestamp),
            reverse=True
        )
        
        return all_memories[:top_k]
    
    def get_summary(self) -> str:
        """获取记忆摘要"""
        return f"短期记忆: {len(self.short_term)}, 长期记忆: {len(self.long_term)}"

# 使用示例
print("\n=== 记忆管理 ===")
memory_mgr = MemoryManager()

# 添加记忆
memory_mgr.add_memory("学习了AutoGPT的基本概念", importance=0.8)
memory_mgr.add_memory("实现了简单的Agent", importance=0.9)
memory_mgr.add_memory("遇到了配置问题", importance=0.6)

print(memory_mgr.get_summary())

# 回忆
memories = memory_mgr.recall("AutoGPT")
print(f"\n回忆到 {len(memories)} 条相关记忆")
```

### 工具使用

```python
from abc import ABC, abstractmethod
from typing import Any, Dict

class Tool(ABC):
    """工具基类"""
    
    @property
    @abstractmethod
    def name(self) -> str:
        """工具名称"""
        pass
    
    @property
    @abstractmethod
    def description(self) -> str:
        """工具描述"""
        pass
    
    @abstractmethod
    def execute(self, **kwargs) -> Any:
        """执行工具"""
        pass

class WebSearchTool(Tool):
    """网页搜索工具"""
    
    @property
    def name(self) -> str:
        return "web_search"
    
    @property
    def description(self) -> str:
        return "搜索网页内容，返回相关结果"
    
    def execute(self, query: str) -> Dict:
        """执行搜索"""
        print(f"[搜索] {query}")
        # 模拟搜索结果
        return {
            "query": query,
            "results": [
                {"title": "结果1", "url": "https://example.com/1"},
                {"title": "结果2", "url": "https://example.com/2"}
            ]
        }

class FileWriteTool(Tool):
    """文件写入工具"""
    
    @property
    def name(self) -> str:
        return "write_file"
    
    @property
    def description(self) -> str:
        return "将内容写入文件"
    
    def execute(self, filename: str, content: str) -> Dict:
        """写入文件"""
        print(f"[写入文件] {filename}")
        # 实际应该真正写入文件
        return {
            "filename": filename,
            "status": "success",
            "bytes_written": len(content)
        }

class ToolRegistry:
    """工具注册表"""
    
    def __init__(self):
        self.tools: Dict[str, Tool] = {}
    
    def register(self, tool: Tool):
        """注册工具"""
        self.tools[tool.name] = tool
        print(f"注册工具: {tool.name}")
    
    def get_tool(self, name: str) -> Tool:
        """获取工具"""
        return self.tools.get(name)
    
    def list_tools(self) -> str:
        """列出所有工具"""
        tools_desc = []
        for name, tool in self.tools.items():
            tools_desc.append(f"- {name}: {tool.description}")
        return "\n".join(tools_desc)

# 使用示例
print("\n=== 工具使用 ===")
registry = ToolRegistry()
registry.register(WebSearchTool())
registry.register(FileWriteTool())

print("\n可用工具:")
print(registry.list_tools())

# 使用工具
search_tool = registry.get_tool("web_search")
result = search_tool.execute(query="AutoGPT教程")
print(f"\n搜索结果: {len(result['results'])} 条")
```

---

## 实战案例

### 案例 1：自动化研究助手

```python
class ResearchAgent:
    """研究助手 Agent"""
    
    def __init__(self, topic: str):
        self.topic = topic
        self.research_data = []
        self.summary = ""
        
        # 初始化工具
        self.tools = ToolRegistry()
        self.tools.register(WebSearchTool())
        self.tools.register(FileWriteTool())
    
    def research(self):
        """执行研究"""
        print(f"=== 开始研究: {self.topic} ===\n")
        
        # 步骤1: 搜索资料
        print("步骤1: 搜索相关资料")
        search_tool = self.tools.get_tool("web_search")
        results = search_tool.execute(query=self.topic)
        self.research_data.extend(results['results'])
        print(f"找到 {len(results['results'])} 条结果\n")
        
        # 步骤2: 分析数据
        print("步骤2: 分析数据")
        self._analyze_data()
        
        # 步骤3: 生成报告
        print("\n步骤3: 生成研究报告")
        self._generate_report()
        
        # 步骤4: 保存报告
        print("\n步骤4: 保存报告")
        self._save_report()
    
    def _analyze_data(self):
        """分析数据"""
        print("正在分析收集的数据...")
        # 简化的分析逻辑
        self.summary = f"关于'{self.topic}'的研究发现了{len(self.research_data)}个相关资源"
    
    def _generate_report(self):
        """生成报告"""
        report = f"""
# {self.topic} 研究报告

## 摘要
{self.summary}

## 数据来源
"""
        for i, data in enumerate(self.research_data, 1):
            report += f"{i}. {data['title']} - {data['url']}\n"
        
        self.report = report
        print("报告已生成")
    
    def _save_report(self):
        """保存报告"""
        write_tool = self.tools.get_tool("write_file")
        result = write_tool.execute(
            filename=f"{self.topic}_report.md",
            content=self.report
        )
        print(f"报告已保存: {result['filename']}")

# 使用示例
print("\n=== 研究助手案例 ===")
agent = ResearchAgent(topic="大语言模型应用")
agent.research()
```

### 案例 2：自动化代码审查

```python
class CodeReviewAgent:
    """代码审查 Agent"""
    
    def __init__(self):
        self.issues = []
        self.suggestions = []
    
    def review_code(self, code: str, language: str = "python"):
        """审查代码"""
        print(f"=== 代码审查开始 ===")
        print(f"语言: {language}\n")
        
        # 检查项
        self._check_naming(code)
        self._check_complexity(code)
        self._check_documentation(code)
        
        # 生成报告
        self._generate_report()
    
    def _check_naming(self, code: str):
        """检查命名规范"""
        print("检查1: 命名规范")
        
        # 简化的检查逻辑
        if "def " in code:
            if any(c.isupper() for c in code.split("def ")[1].split("(")[0]):
                self.issues.append({
                    "type": "naming",
                    "severity": "warning",
                    "message": "函数名应使用小写和下划线"
                })
                print("  ⚠️  发现命名问题")
            else:
                print("  ✅ 命名规范正常")
    
    def _check_complexity(self, code: str):
        """检查复杂度"""
        print("\n检查2: 代码复杂度")
        
        # 简化的复杂度检查
        lines = code.split('\n')
        if len(lines) > 50:
            self.issues.append({
                "type": "complexity",
                "severity": "info",
                "message": "函数过长，建议拆分"
            })
            print("  ℹ️  代码较长，建议拆分")
        else:
            print("  ✅ 复杂度适中")
    
    def _check_documentation(self, code: str):
        """检查文档"""
        print("\n检查3: 文档注释")
        
        if '"""' not in code and "'''" not in code:
            self.issues.append({
                "type": "documentation",
                "severity": "warning",
                "message": "缺少文档字符串"
            })
            print("  ⚠️  缺少文档注释")
        else:
            print("  ✅ 文档完整")
    
    def _generate_report(self):
        """生成报告"""
        print("\n=== 审查报告 ===")
        
        if not self.issues:
            print("✅ 未发现问题，代码质量良好！")
        else:
            print(f"发现 {len(self.issues)} 个问题:\n")
            for i, issue in enumerate(self.issues, 1):
                severity_icon = {
                    "error": "❌",
                    "warning": "⚠️",
                    "info": "ℹ️"
                }
                icon = severity_icon.get(issue['severity'], "•")
                print(f"{i}. {icon} [{issue['type']}] {issue['message']}")

# 使用示例
print("\n=== 代码审查案例 ===")
sample_code = """
def CalculateSum(numbers):
    total = 0
    for num in numbers:
        total += num
    return total
"""

reviewer = CodeReviewAgent()
reviewer.review_code(sample_code)
```

---

## 总结

### AutoGPT 核心要点

| 要点 | 说明 |
|------|------|
| 自主性 | 能够自主分解和执行任务 |
| 记忆管理 | 短期和长期记忆的有效管理 |
| 工具使用 | 灵活使用各种工具完成任务 |
| 自我反思 | 评估执行结果并调整策略 |

### 最佳实践

1. **明确目标**
   - 设定清晰、可衡量的目标
   - 避免过于宽泛的任务

2. **合理限制**
   - 设置最大迭代次数
   - 限制工具使用范围
   - 控制成本（API调用）

3. **安全考虑**
   - 谨慎启用代码执行
   - 限制文件操作权限
   - 监控Agent行为

4. **性能优化**
   - 使用向量数据库存储记忆
   - 缓存常用结果
   - 批量处理任务

### 常见问题

**Q: AutoGPT 会无限循环吗？**
A: 需要设置最大迭代次数和超时时间来防止无限循环。

**Q: 如何控制成本？**
A: 限制API调用次数、使用更便宜的模型、缓存结果。

**Q: AutoGPT 适合什么场景？**
A: 适合需要多步骤、自主决策的复杂任务，如研究、数据分析、内容生成等。

### 学习路径

1. **基础阶段**（1-2周）
   - 理解Agent概念
   - 掌握基本架构
   - 实现简单Agent

2. **进阶阶段**（2-3周）
   - 记忆管理
   - 工具集成
   - 目标分解

3. **实战阶段**（3-4周）
   - 构建实际应用
   - 性能优化
   - 安全加固

### 推荐资源

- [AutoGPT GitHub](https://github.com/Significant-Gravitas/AutoGPT)
- [LangChain Agent文档](https://python.langchain.com/docs/modules/agents/)
- [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling)

---

## 下一步学习

完成本教程后，建议继续学习：

1. [BabyAGI完整教程](BabyAGI-完整教程.md) - 更简化的Agent架构
2. [Agent协作框架完整教程](Agent协作框架-完整教程.md) - 多Agent协作
3. [自主Agent设计完整教程](自主Agent设计-完整教程.md) - 高级Agent设计

---

**记住：AutoGPT 是强大的工具，但需要合理使用和监控！** 🤖

@author erik.zhou
