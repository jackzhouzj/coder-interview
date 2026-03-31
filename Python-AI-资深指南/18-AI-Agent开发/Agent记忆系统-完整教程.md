# Agent 记忆系统完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

记忆系统是 AI Agent 从"无状态工具"进化为"有状态智能体"的关键。本教程深入讲解 Agent 记忆的分类、实现方案和最佳实践，涵盖短期记忆、长期记忆、情景记忆、语义记忆等核心概念。

### 版本信息
- **重要程度**：⭐⭐⭐⭐⭐（必学）
- **难度等级**：⭐⭐⭐⭐（较难）
- **预计学习时间**：15-20 小时

### 学习目标
- 理解 Agent 记忆系统的分类和设计原则
- 掌握短期记忆（对话上下文）的实现
- 掌握长期记忆（向量存储）的实现
- 学会使用 Mem0 等记忆框架
- 能够设计生产级 Agent 记忆架构

---

## 1. 记忆系统分类

### 1.1 四种记忆类型

```
Agent 记忆系统
├── 短期记忆（Short-term / Working Memory）
│   ├── 对话历史（最近 N 轮）
│   ├── 当前任务上下文
│   └── 实现：内存 / Redis
├── 长期记忆（Long-term Memory）
│   ├── 用户偏好和画像
│   ├── 历史交互摘要
│   └── 实现：向量数据库 / 知识图谱
├── 情景记忆（Episodic Memory）
│   ├── 具体事件和经历
│   ├── 时间线索引
│   └── 实现：时序数据库 + 向量检索
└── 语义记忆（Semantic Memory）
    ├── 通用知识和事实
    ├── 概念关系
    └── 实现：知识图谱 / RAG
```

### 1.2 记忆生命周期

```python
from enum import Enum
from dataclasses import dataclass, field
from datetime import datetime
from typing import Optional

class MemoryType(Enum):
    SHORT_TERM = "short_term"
    LONG_TERM = "long_term"
    EPISODIC = "episodic"
    SEMANTIC = "semantic"

@dataclass
class MemoryItem:
    """记忆条目"""
    content: str
    memory_type: MemoryType
    created_at: datetime = field(default_factory=datetime.now)
    last_accessed: datetime = field(default_factory=datetime.now)
    access_count: int = 0
    importance: float = 0.5  # 0-1
    metadata: dict = field(default_factory=dict)
    embedding: Optional[list] = None
    
    def access(self):
        """访问记忆，更新元数据"""
        self.last_accessed = datetime.now()
        self.access_count += 1
    
    def decay_score(self) -> float:
        """计算记忆衰减分数（模拟遗忘曲线）"""
        hours_since_access = (
            datetime.now() - self.last_accessed
        ).total_seconds() / 3600
        
        # 艾宾浩斯遗忘曲线简化版
        decay = 1.0 / (1.0 + hours_since_access * 0.1)
        return self.importance * decay * (1 + self.access_count * 0.1)
```

---

## 2. 短期记忆实现

### 2.1 滑动窗口记忆

```python
from collections import deque
from typing import List, Dict

class SlidingWindowMemory:
    """滑动窗口短期记忆"""
    
    def __init__(self, max_messages: int = 20):
        self.max_messages = max_messages
        self.messages: deque = deque(maxlen=max_messages)
    
    def add(self, role: str, content: str):
        self.messages.append({
            "role": role,
            "content": content,
            "timestamp": datetime.now().isoformat()
        })
    
    def get_context(self) -> List[Dict]:
        """获取对话上下文"""
        return list(self.messages)
    
    def get_formatted(self) -> str:
        """格式化为字符串"""
        return "\n".join(
            f"{m['role']}: {m['content']}" 
            for m in self.messages
        )

# 使用
memory = SlidingWindowMemory(max_messages=10)
memory.add("user", "什么是Python？")
memory.add("assistant", "Python是一种高级编程语言...")
memory.add("user", "它有什么特点？")
```

### 2.2 Token 感知记忆

```python
import tiktoken

class TokenAwareMemory:
    """Token 感知的短期记忆，确保不超过上下文窗口"""
    
    def __init__(self, max_tokens: int = 4000, model: str = "gpt-4o"):
        self.max_tokens = max_tokens
        self.encoder = tiktoken.encoding_for_model(model)
        self.messages: List[Dict] = []
    
    def _count_tokens(self, text: str) -> int:
        return len(self.encoder.encode(text))
    
    def _total_tokens(self) -> int:
        return sum(
            self._count_tokens(m["content"]) 
            for m in self.messages
        )
    
    def add(self, role: str, content: str):
        self.messages.append({"role": role, "content": content})
        
        # 超出限制时，移除最早的消息
        while self._total_tokens() > self.max_tokens and len(self.messages) > 2:
            self.messages.pop(0)
    
    def get_context(self) -> List[Dict]:
        return self.messages.copy()
```

### 2.3 摘要压缩记忆

```python
from openai import OpenAI

class SummaryMemory:
    """摘要压缩记忆：旧对话压缩为摘要，保留近期对话"""
    
    def __init__(self, recent_count: int = 6, max_summary_tokens: int = 500):
        self.recent_count = recent_count
        self.max_summary_tokens = max_summary_tokens
        self.messages: List[Dict] = []
        self.summary: str = ""
        self.client = OpenAI()
    
    def add(self, role: str, content: str):
        self.messages.append({"role": role, "content": content})
        
        # 当消息过多时，压缩旧消息为摘要
        if len(self.messages) > self.recent_count * 2:
            self._compress()
    
    def _compress(self):
        """将旧消息压缩为摘要"""
        old_messages = self.messages[:-self.recent_count]
        
        conversation = "\n".join(
            f"{m['role']}: {m['content']}" for m in old_messages
        )
        
        response = self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"请将以下对话压缩为简洁的摘要，保留关键信息：\n\n{conversation}"
            }],
            max_tokens=self.max_summary_tokens
        )
        
        self.summary = response.choices[0].message.content
        self.messages = self.messages[-self.recent_count:]
    
    def get_context(self) -> List[Dict]:
        """获取完整上下文（摘要 + 近期消息）"""
        context = []
        if self.summary:
            context.append({
                "role": "system",
                "content": f"之前的对话摘要：{self.summary}"
            })
        context.extend(self.messages)
        return context
```

---

## 3. 长期记忆实现

### 3.1 向量存储长期记忆

```python
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings
from langchain.schema import Document
from datetime import datetime
from typing import List

class VectorLongTermMemory:
    """基于向量存储的长期记忆"""
    
    def __init__(self):
        self.embeddings = OpenAIEmbeddings()
        self.vectorstore = FAISS.from_texts(
            ["初始化"], self.embeddings
        )
    
    def store(self, content: str, metadata: dict = None):
        """存储记忆"""
        meta = {
            "timestamp": datetime.now().isoformat(),
            "type": "long_term",
            **(metadata or {})
        }
        doc = Document(page_content=content, metadata=meta)
        self.vectorstore.add_documents([doc])
    
    def recall(self, query: str, top_k: int = 5) -> List[str]:
        """检索相关记忆"""
        docs = self.vectorstore.similarity_search(query, k=top_k)
        return [doc.page_content for doc in docs]
    
    def recall_with_score(self, query: str, top_k: int = 5, 
                          threshold: float = 0.7):
        """带相似度分数的检索"""
        results = self.vectorstore.similarity_search_with_score(
            query, k=top_k
        )
        return [
            {"content": doc.page_content, "score": score}
            for doc, score in results
            if score >= threshold
        ]
    
    def save(self, path: str):
        self.vectorstore.save_local(path)
    
    def load(self, path: str):
        self.vectorstore = FAISS.load_local(path, self.embeddings)

# 使用
memory = VectorLongTermMemory()
memory.store("用户喜欢Python和机器学习", {"user_id": "u001"})
memory.store("用户是后端开发工程师", {"user_id": "u001"})

# 检索
results = memory.recall("用户的技术背景是什么？")
print(results)
```

### 3.2 使用 Mem0 框架

```python
"""
Mem0 是专为 AI Agent 设计的记忆层框架
支持用户级、会话级、Agent级记忆管理
"""
from mem0 import Memory

# 🔥 初始化 Mem0
config = {
    "llm": {
        "provider": "openai",
        "config": {"model": "gpt-4o-mini"}
    },
    "embedder": {
        "provider": "openai",
        "config": {"model": "text-embedding-3-small"}
    },
    "vector_store": {
        "provider": "qdrant",
        "config": {
            "host": "localhost",
            "port": 6333,
            "collection_name": "agent_memory"
        }
    }
}

m = Memory.from_config(config)

# 🔥 添加记忆（自动提取关键信息）
m.add(
    "我是一名Python开发者，主要做AI方向，喜欢用PyTorch",
    user_id="user-001"
)

m.add(
    "我最近在学习RAG和Agent开发",
    user_id="user-001"
)

# 🔥 搜索记忆
results = m.search("用户的技术栈", user_id="user-001")
for r in results:
    print(f"记忆: {r['memory']}")
    print(f"相关度: {r['score']}")

# 🔥 获取所有记忆
all_memories = m.get_all(user_id="user-001")
for mem in all_memories:
    print(f"- {mem['memory']}")

# 🔥 更新记忆（Mem0 自动处理冲突和合并）
m.add(
    "我现在转向用LangGraph做Agent开发了",
    user_id="user-001"
)
```

### 3.3 在 Agent 中集成记忆

```python
from agents import Agent, Runner, function_tool
from mem0 import Memory

# 初始化记忆
mem = Memory.from_config(config)

@function_tool
def remember(info: str, user_id: str = "default") -> str:
    """将重要信息存入长期记忆
    
    Args:
        info: 要记住的信息
        user_id: 用户ID
    """
    mem.add(info, user_id=user_id)
    return f"已记住: {info}"

@function_tool
def recall(query: str, user_id: str = "default") -> str:
    """从长期记忆中检索相关信息
    
    Args:
        query: 检索查询
        user_id: 用户ID
    """
    results = mem.search(query, user_id=user_id)
    if not results:
        return "没有找到相关记忆"
    return "\n".join(f"- {r['memory']}" for r in results[:5])

# 🔥 带记忆的 Agent
memory_agent = Agent(
    name="记忆助手",
    instructions="""你是一个有记忆能力的助手。
    - 对话中发现重要信息时，使用 remember 工具存储
    - 回答问题前，先用 recall 工具检索相关记忆
    - 结合记忆和当前对话给出个性化回答""",
    tools=[remember, recall]
)
```

---

## 4. 情景记忆与语义记忆

### 4.1 情景记忆（事件记录）

```python
from dataclasses import dataclass
from datetime import datetime
from typing import List, Optional

@dataclass
class Episode:
    """情景记忆条目"""
    event: str
    context: str
    outcome: str
    timestamp: datetime
    importance: float
    tags: List[str]

class EpisodicMemory:
    """情景记忆：记录具体事件和经历"""
    
    def __init__(self):
        self.episodes: List[Episode] = []
        self.embeddings = OpenAIEmbeddings()
    
    def record(self, event: str, context: str, 
               outcome: str, importance: float = 0.5,
               tags: List[str] = None):
        """记录一个事件"""
        episode = Episode(
            event=event,
            context=context,
            outcome=outcome,
            timestamp=datetime.now(),
            importance=importance,
            tags=tags or []
        )
        self.episodes.append(episode)
    
    def recall_similar(self, query: str, top_k: int = 3) -> List[Episode]:
        """检索相似的历史事件"""
        # 基于向量相似度检索
        query_embedding = self.embeddings.embed_query(query)
        
        scored = []
        for ep in self.episodes:
            ep_text = f"{ep.event} {ep.context} {ep.outcome}"
            ep_embedding = self.embeddings.embed_query(ep_text)
            score = self._cosine_similarity(query_embedding, ep_embedding)
            scored.append((ep, score))
        
        scored.sort(key=lambda x: x[1], reverse=True)
        return [ep for ep, _ in scored[:top_k]]
    
    def recall_by_time(self, start: datetime, 
                       end: datetime) -> List[Episode]:
        """按时间范围检索"""
        return [
            ep for ep in self.episodes
            if start <= ep.timestamp <= end
        ]
    
    @staticmethod
    def _cosine_similarity(a, b):
        import numpy as np
        return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# 使用
episodic = EpisodicMemory()
episodic.record(
    event="用户询问了Python异步编程",
    context="用户正在学习FastAPI开发",
    outcome="推荐了asyncio教程，用户表示满意",
    importance=0.8,
    tags=["python", "async", "learning"]
)
```

### 4.2 语义记忆（知识图谱）

```python
from typing import Dict, Set, Tuple

class SemanticMemory:
    """语义记忆：基于知识图谱的概念关系存储"""
    
    def __init__(self):
        self.entities: Dict[str, dict] = {}
        self.relations: List[Tuple[str, str, str]] = []  # (主体, 关系, 客体)
    
    def add_entity(self, name: str, entity_type: str, 
                   properties: dict = None):
        """添加实体"""
        self.entities[name] = {
            "type": entity_type,
            "properties": properties or {}
        }
    
    def add_relation(self, subject: str, relation: str, obj: str):
        """添加关系"""
        self.relations.append((subject, relation, obj))
    
    def query_relations(self, entity: str) -> List[Tuple]:
        """查询实体的所有关系"""
        results = []
        for s, r, o in self.relations:
            if s == entity:
                results.append((r, o))
            elif o == entity:
                results.append((f"被{r}", s))
        return results
    
    def get_context(self, entity: str, depth: int = 2) -> str:
        """获取实体的上下文信息（多跳）"""
        visited = set()
        context = []
        
        def traverse(e, d):
            if d == 0 or e in visited:
                return
            visited.add(e)
            
            for r, target in self.query_relations(e):
                context.append(f"{e} {r} {target}")
                traverse(target, d - 1)
        
        traverse(entity, depth)
        return "\n".join(context)

# 使用
semantic = SemanticMemory()
semantic.add_entity("用户A", "person", {"role": "开发者"})
semantic.add_entity("Python", "language")
semantic.add_entity("RAG", "technology")

semantic.add_relation("用户A", "擅长", "Python")
semantic.add_relation("用户A", "正在学习", "RAG")
semantic.add_relation("RAG", "依赖", "向量数据库")

print(semantic.get_context("用户A"))
```

---

## 5. 生产级记忆架构

### 5.1 统一记忆管理器

```python
class UnifiedMemoryManager:
    """统一记忆管理器：整合四种记忆类型"""
    
    def __init__(self, user_id: str):
        self.user_id = user_id
        self.short_term = SummaryMemory()
        self.long_term = VectorLongTermMemory()
        self.episodic = EpisodicMemory()
        self.semantic = SemanticMemory()
    
    def add_message(self, role: str, content: str):
        """添加对话消息"""
        self.short_term.add(role, content)
    
    def store_fact(self, fact: str, metadata: dict = None):
        """存储长期事实"""
        self.long_term.store(fact, metadata)
    
    def build_context(self, query: str) -> str:
        """构建完整的记忆上下文"""
        parts = []
        
        # 1. 长期记忆
        long_term_results = self.long_term.recall(query, top_k=3)
        if long_term_results:
            parts.append("【用户画像】\n" + "\n".join(
                f"- {r}" for r in long_term_results
            ))
        
        # 2. 情景记忆
        similar_episodes = self.episodic.recall_similar(query, top_k=2)
        if similar_episodes:
            parts.append("【相关历史】\n" + "\n".join(
                f"- {ep.event} → {ep.outcome}" 
                for ep in similar_episodes
            ))
        
        # 3. 短期记忆（对话上下文）
        short_context = self.short_term.get_formatted()
        if short_context:
            parts.append("【当前对话】\n" + short_context)
        
        return "\n\n".join(parts)
```

---

## 6. 总结

### 核心要点
1. Agent 记忆分为短期、长期、情景、语义四种类型
2. 短期记忆用滑动窗口或摘要压缩，长期记忆用向量存储
3. Mem0 等框架提供了开箱即用的记忆管理能力
4. 生产环境需要统一的记忆管理器整合多种记忆类型

### 学习路径
1. 实现基础的滑动窗口短期记忆
2. 学习向量存储长期记忆
3. 使用 Mem0 框架简化记忆管理
4. 理解情景记忆和语义记忆
5. 设计统一的记忆架构

---

## 🔗 相关资源

- [Mem0 官方文档](https://docs.mem0.ai/)
- [LangChain Memory](https://python.langchain.com/docs/modules/memory/)
- [MemGPT / Letta](https://github.com/cpacker/MemGPT)

---

**@author erik.zhou**
