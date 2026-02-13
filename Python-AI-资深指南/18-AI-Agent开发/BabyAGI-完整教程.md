# BabyAGI 完整教程

> @author erik.zhou
> 
> 更新日期：2026-02-13

## 📋 教程概述

BabyAGI 是一个轻量级的自主任务管理系统，由 Yohei Nakajima 创建。它使用 OpenAI 和向量数据库（如 Pinecone）来创建、优先排序和执行任务。BabyAGI 的核心思想是通过任务列表、执行代理和任务创建代理的循环来实现目标。

### 学习目标
- 理解 BabyAGI 的核心架构和工作原理
- 掌握任务优先级管理机制
- 学会使用向量数据库存储任务结果
- 能够构建自主任务执行系统
- 理解 BabyAGI 与 AutoGPT 的区别

### 前置知识
- Python 基础
- OpenAI API 使用
- 向量数据库基础（Pinecone/Chroma）
- LangChain 基础

## 1. BabyAGI 核心概念

### 1.1 架构设计

BabyAGI 采用简单而强大的三步循环：

```python
# BabyAGI 核心循环
while True:
    # 1. 从任务列表中获取第一个任务
    task = task_list.popleft()
    
    # 2. 执行任务
    result = execute_task(task)
    
    # 3. 存储结果到向量数据库
    store_result(task, result)
    
    # 4. 基于目标和结果创建新任务
    new_tasks = create_tasks(objective, result)
    
    # 5. 对任务列表重新排序
    task_list = prioritize_tasks(task_list, new_tasks)
```

### 1.2 核心组件


```python
from collections import deque
from typing import Dict, List
import openai
from chromadb import Client

class BabyAGI:
    """BabyAGI 核心类"""
    
    def __init__(self, objective: str):
        self.objective = objective
        self.task_list = deque()
        self.task_id_counter = 1
        self.vector_store = Client()
        
    def add_task(self, task: Dict):
        """添加任务到列表"""
        self.task_list.append(task)
        
    def execute_task(self, task: Dict) -> str:
        """执行单个任务"""
        prompt = f"""
        你是一个任务执行AI。
        目标：{self.objective}
        当前任务：{task['task_name']}
        
        请执行这个任务并返回结果。
        """
        
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        return response.choices[0].message.content
        
    def create_tasks(self, result: str, task_description: str) -> List[Dict]:
        """基于结果创建新任务"""
        prompt = f"""
        你是一个任务创建AI。
        目标：{self.objective}
        上一个任务：{task_description}
        任务结果：{result}
        
        基于结果，创建新的任务列表来完成目标。
        返回格式：每行一个任务名称。
        """
        
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        
        new_tasks = response.choices[0].message.content.strip().split('\n')
        return [{"task_id": self.task_id_counter + i, "task_name": task} 
                for i, task in enumerate(new_tasks) if task.strip()]
```

## 2. 环境搭建

### 2.1 安装依赖

```bash
# 安装核心依赖
pip install openai chromadb langchain python-dotenv

# 可选：使用 Pinecone
pip install pinecone-client
```

### 2.2 配置环境变量

```python
# .env 文件
OPENAI_API_KEY=your_openai_api_key
PINECONE_API_KEY=your_pinecone_api_key  # 可选
PINECONE_ENVIRONMENT=your_environment    # 可选
```

### 2.3 基础配置

```python
import os
from dotenv import load_dotenv

load_dotenv()

# OpenAI 配置
openai.api_key = os.getenv("OPENAI_API_KEY")

# 向量数据库配置
VECTOR_STORE = "chroma"  # 或 "pinecone"
```

## 3. 任务优先级管理

### 3.1 任务优先级排序

```python
class TaskPrioritizer:
    """任务优先级管理器"""
    
    def __init__(self, objective: str):
        self.objective = objective
        
    def prioritize_tasks(self, task_list: List[Dict]) -> List[Dict]:
        """对任务列表进行优先级排序"""
        task_names = [t["task_name"] for t in task_list]
        
        prompt = f"""
        你是一个任务优先级AI。
        目标：{self.objective}
        
        任务列表：
        {chr(10).join(f"{i+1}. {task}" for i, task in enumerate(task_names))}
        
        请按照对实现目标的重要性对任务重新排序。
        返回格式：每行一个任务编号（如：3, 1, 2）
        """
        
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        
        # 解析优先级顺序
        order = [int(x.strip()) - 1 for x in response.choices[0].message.content.split(',')]
        return [task_list[i] for i in order if i < len(task_list)]
```

### 3.2 动态优先级调整

```python
class DynamicPrioritizer:
    """动态优先级调整器"""
    
    def __init__(self):
        self.task_history = []
        self.success_rate = {}
        
    def adjust_priority(self, task: Dict, result: str) -> float:
        """根据执行结果调整优先级"""
        # 评估任务执行质量
        quality_score = self._evaluate_quality(result)
        
        # 更新成功率
        task_type = self._get_task_type(task["task_name"])
        if task_type not in self.success_rate:
            self.success_rate[task_type] = []
        self.success_rate[task_type].append(quality_score)
        
        # 计算新优先级
        avg_success = sum(self.success_rate[task_type]) / len(self.success_rate[task_type])
        return avg_success
        
    def _evaluate_quality(self, result: str) -> float:
        """评估结果质量（0-1）"""
        prompt = f"""
        评估以下任务结果的质量（0-1分）：
        {result}
        
        只返回数字。
        """
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        return float(response.choices[0].message.content.strip())
```

## 4. 任务生成与执行

### 4.1 智能任务生成

```python
class TaskGenerator:
    """智能任务生成器"""
    
    def __init__(self, objective: str):
        self.objective = objective
        self.completed_tasks = []
        
    def generate_tasks(self, context: Dict) -> List[Dict]:
        """基于上下文生成任务"""
        prompt = f"""
        目标：{self.objective}
        
        已完成任务：
        {self._format_completed_tasks()}
        
        当前上下文：
        - 上一个任务：{context.get('last_task', 'None')}
        - 任务结果：{context.get('result', 'None')}
        - 遇到的问题：{context.get('issues', 'None')}
        
        请生成下一步需要执行的任务列表。
        每个任务应该：
        1. 具体可执行
        2. 有明确的输出
        3. 与目标相关
        
        返回格式：每行一个任务
        """
        
        response = openai.ChatCompletion.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        
        tasks = response.choices[0].message.content.strip().split('\n')
        return [{"task_name": task.strip(), "context": context} 
                for task in tasks if task.strip()]
```

### 4.2 任务执行引擎

```python
class TaskExecutor:
    """任务执行引擎"""
    
    def __init__(self):
        self.execution_history = []
        self.tools = self._init_tools()
        
    def execute(self, task: Dict) -> Dict:
        """执行任务并返回结果"""
        try:
            # 选择合适的工具
            tool = self._select_tool(task)
            
            # 执行任务
            result = tool.run(task["task_name"])
            
            # 记录执行历史
            self.execution_history.append({
                "task": task,
                "result": result,
                "status": "success"
            })
            
            return {
                "status": "success",
                "result": result,
                "task_id": task["task_id"]
            }
            
        except Exception as e:
            return {
                "status": "failed",
                "error": str(e),
                "task_id": task["task_id"]
            }
            
    def _select_tool(self, task: Dict):
        """选择合适的工具执行任务"""
        task_name = task["task_name"].lower()
        
        if "search" in task_name:
            return self.tools["search"]
        elif "write" in task_name or "create" in task_name:
            return self.tools["write"]
        else:
            return self.tools["default"]
```

## 5. 向量存储与检索

### 5.1 使用 ChromaDB

```python
import chromadb
from chromadb.config import Settings

class VectorStore:
    """向量存储管理器"""
    
    def __init__(self, collection_name: str = "babyagi"):
        self.client = chromadb.Client(Settings(
            chroma_db_impl="duckdb+parquet",
            persist_directory="./chroma_db"
        ))
        self.collection = self.client.get_or_create_collection(
            name=collection_name
        )
        
    def store_result(self, task_id: int, task_name: str, result: str):
        """存储任务结果"""
        self.collection.add(
            documents=[result],
            metadatas=[{"task_id": task_id, "task_name": task_name}],
            ids=[f"task_{task_id}"]
        )
        
    def query_similar(self, query: str, n_results: int = 5) -> List[Dict]:
        """查询相似任务结果"""
        results = self.collection.query(
            query_texts=[query],
            n_results=n_results
        )
        
        return [{
            "task_name": results["metadatas"][0][i]["task_name"],
            "result": results["documents"][0][i],
            "distance": results["distances"][0][i]
        } for i in range(len(results["documents"][0]))]
```

### 5.2 使用 Pinecone

```python
import pinecone
from langchain.embeddings import OpenAIEmbeddings

class PineconeStore:
    """Pinecone 向量存储"""
    
    def __init__(self, index_name: str = "babyagi"):
        pinecone.init(
            api_key=os.getenv("PINECONE_API_KEY"),
            environment=os.getenv("PINECONE_ENVIRONMENT")
        )
        
        # 创建或连接索引
        if index_name not in pinecone.list_indexes():
            pinecone.create_index(index_name, dimension=1536)
        
        self.index = pinecone.Index(index_name)
        self.embeddings = OpenAIEmbeddings()
        
    def store_result(self, task_id: int, task_name: str, result: str):
        """存储任务结果到 Pinecone"""
        # 生成嵌入向量
        vector = self.embeddings.embed_query(result)
        
        # 存储到 Pinecone
        self.index.upsert([(
            f"task_{task_id}",
            vector,
            {"task_name": task_name, "result": result}
        )])
```

## 6. 完整实现示例

### 6.1 基础 BabyAGI 实现

```python
from collections import deque
import time

class SimpleBabyAGI:
    """简化版 BabyAGI 实现"""
    
    def __init__(self, objective: str, initial_task: str):
        self.objective = objective
        self.task_list = deque([{
            "task_id": 1,
            "task_name": initial_task
        }])
        self.task_id_counter = 1
        self.vector_store = VectorStore()
        
    def run(self, max_iterations: int = 10):
        """运行 BabyAGI"""
        iteration = 0
        
        while self.task_list and iteration < max_iterations:
            iteration += 1
            print(f"\n{'='*50}")
            print(f"迭代 {iteration}")
            print(f"{'='*50}")
            
            # 1. 获取任务
            task = self.task_list.popleft()
            print(f"\n执行任务 {task['task_id']}: {task['task_name']}")
            
            # 2. 执行任务
            result = self._execute_task(task)
            print(f"结果: {result[:200]}...")
            
            # 3. 存储结果
            self.vector_store.store_result(
                task["task_id"],
                task["task_name"],
                result
            )
            
            # 4. 创建新任务
            new_tasks = self._create_tasks(result, task["task_name"])
            print(f"\n创建了 {len(new_tasks)} 个新任务")
            
            # 5. 优先级排序
            self.task_list = self._prioritize_tasks(
                list(self.task_list) + new_tasks
            )
            
            print(f"\n当前任务队列: {len(self.task_list)} 个任务")
            for t in list(self.task_list)[:3]:
                print(f"  - {t['task_name']}")
            
            time.sleep(1)  # 避免 API 限流
```

### 6.2 增强版实现

```python
class EnhancedBabyAGI:
    """增强版 BabyAGI"""
    
    def __init__(self, objective: str, initial_task: str):
        self.objective = objective
        self.task_list = deque([{
            "task_id": 1,
            "task_name": initial_task,
            "priority": 1.0
        }])
        self.task_id_counter = 1
        self.vector_store = VectorStore()
        self.executor = TaskExecutor()
        self.prioritizer = DynamicPrioritizer()
        self.completed_tasks = []
        
    def run(self, max_iterations: int = 20):
        """运行增强版 BabyAGI"""
        for iteration in range(max_iterations):
            if not self.task_list:
                print("所有任务已完成！")
                break
                
            # 获取最高优先级任务
            task = self._get_highest_priority_task()
            
            # 执行任务
            result = self.executor.execute(task)
            
            if result["status"] == "success":
                # 存储结果
                self.vector_store.store_result(
                    task["task_id"],
                    task["task_name"],
                    result["result"]
                )
                
                # 调整优先级
                priority = self.prioritizer.adjust_priority(
                    task, result["result"]
                )
                
                # 创建新任务
                context = {
                    "last_task": task["task_name"],
                    "result": result["result"],
                    "priority": priority
                }
                new_tasks = self._create_contextual_tasks(context)
                
                # 添加到任务列表
                for new_task in new_tasks:
                    self.task_list.append(new_task)
                    
                self.completed_tasks.append(task)
            else:
                # 任务失败，重新加入队列
                task["retry_count"] = task.get("retry_count", 0) + 1
                if task["retry_count"] < 3:
                    self.task_list.append(task)
```

## 7. 实战案例

### 7.1 研究助手

```python
# 创建研究助手
research_agent = SimpleBabyAGI(
    objective="研究 Python 异步编程的最佳实践",
    initial_task="搜索 Python asyncio 的官方文档"
)

# 运行
research_agent.run(max_iterations=15)
```

### 7.2 内容创作助手

```python
# 创建内容创作助手
content_agent = EnhancedBabyAGI(
    objective="写一篇关于 AI Agent 的技术博客",
    initial_task="列出博客的主要章节"
)

# 运行
content_agent.run(max_iterations=20)
```

### 7.3 项目规划助手

```python
# 创建项目规划助手
project_agent = SimpleBabyAGI(
    objective="规划一个 RAG 应用的开发计划",
    initial_task="分析 RAG 应用的核心功能需求"
)

# 运行
project_agent.run(max_iterations=25)
```

## 8. 最佳实践

### 8.1 任务设计原则

```python
# ✅ 好的任务设计
good_tasks = [
    "搜索 Python asyncio 的官方文档",
    "总结 asyncio 的核心概念（不超过 500 字）",
    "编写一个使用 asyncio 的示例代码"
]

# ❌ 不好的任务设计
bad_tasks = [
    "学习 Python",  # 太宽泛
    "做一些研究",   # 不具体
    "完成项目"      # 没有明确输出
]
```

### 8.2 优化建议

```python
class OptimizedBabyAGI:
    """优化版 BabyAGI"""
    
    def __init__(self, objective: str, initial_task: str):
        self.objective = objective
        self.task_list = deque([self._create_task(initial_task)])
        self.task_id_counter = 1
        
        # 优化配置
        self.max_task_queue_size = 10  # 限制队列大小
        self.task_timeout = 60  # 任务超时时间
        self.enable_caching = True  # 启用结果缓存
        
    def _create_task(self, task_name: str, priority: float = 1.0) -> Dict:
        """创建任务（带验证）"""
        # 验证任务名称
        if len(task_name) < 5:
            raise ValueError("任务名称太短")
        if len(task_name) > 200:
            raise ValueError("任务名称太长")
            
        self.task_id_counter += 1
        return {
            "task_id": self.task_id_counter,
            "task_name": task_name,
            "priority": priority,
            "created_at": time.time()
        }
```

## 9. BabyAGI vs AutoGPT

### 9.1 核心区别

| 特性 | BabyAGI | AutoGPT |
|------|---------|---------|
| 架构 | 简单循环 | 复杂状态机 |
| 任务管理 | 优先级队列 | 目标树 |
| 记忆系统 | 向量数据库 | 文件 + 向量 |
| 工具使用 | 有限 | 丰富 |
| 适用场景 | 研究、规划 | 复杂任务执行 |

### 9.2 选择建议

```python
# 使用 BabyAGI 的场景
babyagi_scenarios = [
    "研究和信息收集",
    "内容规划和大纲",
    "任务分解和优先级排序",
    "简单的自动化流程"
]

# 使用 AutoGPT 的场景
autogpt_scenarios = [
    "需要多种工具协作",
    "长期运行的复杂任务",
    "需要文件操作和代码执行",
    "需要持久化状态管理"
]
```

## 10. 总结

### 10.1 核心要点

- BabyAGI 采用简单的任务循环架构
- 任务优先级管理是核心功能
- 向量数据库用于存储和检索任务结果
- 适合研究、规划类任务

### 10.2 学习路径

1. 理解 BabyAGI 的核心循环
2. 实现基础的任务执行系统
3. 集成向量数据库
4. 优化任务优先级算法
5. 扩展工具和功能

### 10.3 进阶方向

- 集成更多工具（搜索、代码执行等）
- 实现任务依赖关系管理
- 添加任务执行监控和可视化
- 优化任务生成策略
- 实现多 Agent 协作

## 11. 参考资源

- [BabyAGI GitHub](https://github.com/yoheinakajima/babyagi)
- [BabyAGI 论文](https://yoheinakajima.com/birth-of-babyagi/)
- [LangChain BabyAGI](https://python.langchain.com/docs/use_cases/autonomous_agents/baby_agi)
- [向量数据库对比](https://www.pinecone.io/learn/vector-database/)

---

**@author erik.zhou**
