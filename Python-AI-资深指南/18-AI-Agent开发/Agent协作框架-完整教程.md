# Agent 协作框架完整教程

> @author erik.zhou
> 
> 更新日期：2026-02-13

## 📋 教程概述

Agent 协作框架是构建多 Agent 系统的核心技术，它定义了 Agent 之间如何通信、协作和协调。本教程将深入讲解 Agent 协作的各种模式、通信协议、冲突解决机制以及性能优化策略。

### 学习目标
- 理解多 Agent 系统的核心概念
- 掌握 Agent 通信协议设计
- 学会实现各种协作模式
- 能够处理 Agent 间的冲突
- 掌握协作系统的性能优化

### 前置知识
- Python 面向对象编程
- 异步编程基础
- 消息队列概念
- LangChain/LangGraph 基础

## 1. 多 Agent 系统基础

### 1.1 核心概念

```python
from typing import List, Dict, Optional
from enum import Enum
from dataclasses import dataclass
from datetime import datetime

class AgentRole(Enum):
    """Agent 角色定义"""
    COORDINATOR = "coordinator"  # 协调者
    EXECUTOR = "executor"        # 执行者
    ANALYZER = "analyzer"        # 分析者
    REVIEWER = "reviewer"        # 审查者
    SPECIALIST = "specialist"    # 专家

@dataclass
class AgentCapability:
    """Agent 能力定义"""
    name: str
    description: str
    input_types: List[str]
    output_types: List[str]
    max_concurrent_tasks: int = 1
    
@dataclass
class AgentProfile:
    """Agent 配置文件"""
    agent_id: str
    name: str
    role: AgentRole
    capabilities: List[AgentCapability]
    priority: int = 1
    max_retries: int = 3
```

### 1.2 基础 Agent 类

```python
import asyncio
from abc import ABC, abstractmethod

class BaseAgent(ABC):
    """基础 Agent 类"""
    
    def __init__(self, profile: AgentProfile):
        self.profile = profile
        self.status = "idle"
        self.current_task = None
        self.task_history = []
        
    @abstractmethod
    async def process(self, task: Dict) -> Dict:
        """处理任务（子类实现）"""
        pass
        
    async def execute(self, task: Dict) -> Dict:
        """执行任务（带状态管理）"""
        self.status = "busy"
        self.current_task = task
        
        try:
            result = await self.process(task)
            self.task_history.append({
                "task": task,
                "result": result,
                "status": "success",
                "timestamp": datetime.now()
            })
            return result
        except Exception as e:
            self.task_history.append({
                "task": task,
                "error": str(e),
                "status": "failed",
                "timestamp": datetime.now()
            })
            raise
        finally:
            self.status = "idle"
            self.current_task = None
```

## 2. Agent 通信协议

### 2.1 消息格式定义


```python
from enum import Enum
from uuid import uuid4

class MessageType(Enum):
    """消息类型"""
    REQUEST = "request"          # 请求
    RESPONSE = "response"        # 响应
    BROADCAST = "broadcast"      # 广播
    NOTIFICATION = "notification"  # 通知
    ERROR = "error"              # 错误

@dataclass
class Message:
    """Agent 消息"""
    message_id: str
    sender_id: str
    receiver_id: Optional[str]  # None 表示广播
    message_type: MessageType
    content: Dict
    timestamp: datetime
    priority: int = 1
    requires_response: bool = False
    
    @classmethod
    def create_request(cls, sender_id: str, receiver_id: str, content: Dict):
        """创建请求消息"""
        return cls(
            message_id=str(uuid4()),
            sender_id=sender_id,
            receiver_id=receiver_id,
            message_type=MessageType.REQUEST,
            content=content,
            timestamp=datetime.now(),
            requires_response=True
        )
```

### 2.2 消息总线实现

```python
import asyncio
from collections import defaultdict

class MessageBus:
    """消息总线"""
    
    def __init__(self):
        self.subscribers = defaultdict(list)
        self.message_queue = asyncio.Queue()
        self.message_history = []
        
    def subscribe(self, agent_id: str, message_types: List[MessageType]):
        """订阅消息类型"""
        for msg_type in message_types:
            self.subscribers[msg_type].append(agent_id)
            
    async def publish(self, message: Message):
        """发布消息"""
        self.message_history.append(message)
        
        # 点对点消息
        if message.receiver_id:
            await self.message_queue.put((message.receiver_id, message))
        # 广播消息
        else:
            subscribers = self.subscribers.get(message.message_type, [])
            for agent_id in subscribers:
                if agent_id != message.sender_id:
                    await self.message_queue.put((agent_id, message))
                    
    async def receive(self, agent_id: str, timeout: float = None) -> Optional[Message]:
        """接收消息"""
        try:
            receiver, message = await asyncio.wait_for(
                self.message_queue.get(),
                timeout=timeout
            )
            if receiver == agent_id:
                return message
        except asyncio.TimeoutError:
            return None
```

### 2.3 通信协议实现

```python
class CommunicationProtocol:
    """通信协议"""
    
    def __init__(self, message_bus: MessageBus):
        self.message_bus = message_bus
        self.pending_requests = {}
        
    async def send_request(self, sender_id: str, receiver_id: str, 
                          content: Dict, timeout: float = 30) -> Dict:
        """发送请求并等待响应"""
        message = Message.create_request(sender_id, receiver_id, content)
        
        # 记录待响应请求
        response_future = asyncio.Future()
        self.pending_requests[message.message_id] = response_future
        
        # 发送请求
        await self.message_bus.publish(message)
        
        # 等待响应
        try:
            response = await asyncio.wait_for(response_future, timeout=timeout)
            return response
        except asyncio.TimeoutError:
            raise TimeoutError(f"Request {message.message_id} timed out")
        finally:
            del self.pending_requests[message.message_id]
            
    async def send_response(self, request_message: Message, content: Dict):
        """发送响应"""
        response = Message(
            message_id=str(uuid4()),
            sender_id=request_message.receiver_id,
            receiver_id=request_message.sender_id,
            message_type=MessageType.RESPONSE,
            content={
                "request_id": request_message.message_id,
                "data": content
            },
            timestamp=datetime.now()
        )
        
        await self.message_bus.publish(response)
        
        # 完成对应的 Future
        if request_message.message_id in self.pending_requests:
            self.pending_requests[request_message.message_id].set_result(content)
```

## 3. 协作模式

### 3.1 主从模式（Master-Worker）

```python
class MasterAgent(BaseAgent):
    """主 Agent"""
    
    def __init__(self, profile: AgentProfile, workers: List[BaseAgent]):
        super().__init__(profile)
        self.workers = workers
        self.protocol = None
        
    async def process(self, task: Dict) -> Dict:
        """分解任务并分配给 Worker"""
        # 1. 分解任务
        subtasks = await self._decompose_task(task)
        
        # 2. 分配给 Worker
        results = await asyncio.gather(*[
            self._assign_to_worker(subtask)
            for subtask in subtasks
        ])
        
        # 3. 合并结果
        final_result = await self._merge_results(results)
        return final_result
        
    async def _assign_to_worker(self, subtask: Dict) -> Dict:
        """分配子任务给 Worker"""
        # 选择最合适的 Worker
        worker = self._select_worker(subtask)
        
        # 发送任务
        result = await self.protocol.send_request(
            sender_id=self.profile.agent_id,
            receiver_id=worker.profile.agent_id,
            content={"task": subtask}
        )
        return result

class WorkerAgent(BaseAgent):
    """工作 Agent"""
    
    async def process(self, task: Dict) -> Dict:
        """执行具体任务"""
        # 实现具体的任务逻辑
        result = await self._execute_task(task)
        return result
```

### 3.2 管道模式（Pipeline）

```python
class PipelineCoordinator:
    """管道协调器"""
    
    def __init__(self, agents: List[BaseAgent]):
        self.agents = agents
        self.protocol = None
        
    async def execute_pipeline(self, initial_input: Dict) -> Dict:
        """执行管道流程"""
        current_data = initial_input
        
        for agent in self.agents:
            # 传递给下一个 Agent
            result = await self.protocol.send_request(
                sender_id="coordinator",
                receiver_id=agent.profile.agent_id,
                content={"data": current_data}
            )
            current_data = result
            
        return current_data

# 示例：文档处理管道
class DocumentPipeline:
    """文档处理管道"""
    
    def __init__(self):
        self.agents = [
            ExtractorAgent(),   # 提取内容
            AnalyzerAgent(),    # 分析内容
            SummarizerAgent(),  # 生成摘要
            ReviewerAgent()     # 审查结果
        ]
        self.coordinator = PipelineCoordinator(self.agents)
        
    async def process_document(self, document: str) -> Dict:
        """处理文档"""
        return await self.coordinator.execute_pipeline({
            "document": document
        })
```

### 3.3 协商模式（Negotiation）

```python
class NegotiationAgent(BaseAgent):
    """协商 Agent"""
    
    def __init__(self, profile: AgentProfile):
        super().__init__(profile)
        self.proposals = []
        self.agreements = []
        
    async def propose(self, proposal: Dict) -> Dict:
        """提出提案"""
        self.proposals.append(proposal)
        
        # 广播提案
        await self.protocol.message_bus.publish(Message(
            message_id=str(uuid4()),
            sender_id=self.profile.agent_id,
            receiver_id=None,  # 广播
            message_type=MessageType.BROADCAST,
            content={"proposal": proposal},
            timestamp=datetime.now()
        ))
        
    async def evaluate_proposal(self, proposal: Dict) -> float:
        """评估提案（返回 0-1 的接受度）"""
        # 实现评估逻辑
        score = await self._calculate_score(proposal)
        return score
        
    async def negotiate(self, other_agents: List['NegotiationAgent'], 
                       max_rounds: int = 10) -> Optional[Dict]:
        """协商达成一致"""
        for round_num in range(max_rounds):
            # 收集所有提案
            proposals = await self._collect_proposals(other_agents)
            
            # 评估提案
            scores = await asyncio.gather(*[
                agent.evaluate_proposal(proposal)
                for agent in other_agents
                for proposal in proposals
            ])
            
            # 检查是否达成一致
            best_proposal = max(zip(proposals, scores), key=lambda x: x[1])
            if best_proposal[1] > 0.8:  # 阈值
                return best_proposal[0]
                
            # 修改提案
            await self._refine_proposals(proposals, scores)
            
        return None  # 未达成一致
```

### 3.4 竞争模式（Competition）

```python
class CompetitionCoordinator:
    """竞争协调器"""
    
    def __init__(self, agents: List[BaseAgent]):
        self.agents = agents
        self.protocol = None
        
    async def compete(self, task: Dict) -> Dict:
        """让多个 Agent 竞争完成任务"""
        # 同时发送任务给所有 Agent
        tasks = [
            self.protocol.send_request(
                sender_id="coordinator",
                receiver_id=agent.profile.agent_id,
                content={"task": task},
                timeout=30
            )
            for agent in self.agents
        ]
        
        # 等待第一个完成的结果
        done, pending = await asyncio.wait(
            tasks,
            return_when=asyncio.FIRST_COMPLETED
        )
        
        # 取消其他任务
        for task in pending:
            task.cancel()
            
        # 返回最快的结果
        result = done.pop().result()
        return result
```

## 4. 冲突解决机制

### 4.1 资源冲突

```python
class ResourceManager:
    """资源管理器"""
    
    def __init__(self):
        self.resources = {}
        self.locks = {}
        
    async def acquire(self, agent_id: str, resource_id: str, 
                     timeout: float = 10) -> bool:
        """获取资源"""
        if resource_id not in self.locks:
            self.locks[resource_id] = asyncio.Lock()
            
        try:
            await asyncio.wait_for(
                self.locks[resource_id].acquire(),
                timeout=timeout
            )
            self.resources[resource_id] = agent_id
            return True
        except asyncio.TimeoutError:
            return False
            
    def release(self, agent_id: str, resource_id: str):
        """释放资源"""
        if self.resources.get(resource_id) == agent_id:
            self.locks[resource_id].release()
            del self.resources[resource_id]
```

### 4.2 决策冲突

```python
class ConflictResolver:
    """冲突解决器"""
    
    def __init__(self):
        self.resolution_strategies = {
            "voting": self._voting_strategy,
            "priority": self._priority_strategy,
            "consensus": self._consensus_strategy
        }
        
    async def resolve(self, decisions: List[Dict], 
                     strategy: str = "voting") -> Dict:
        """解决决策冲突"""
        resolver = self.resolution_strategies.get(strategy)
        if not resolver:
            raise ValueError(f"Unknown strategy: {strategy}")
            
        return await resolver(decisions)
        
    async def _voting_strategy(self, decisions: List[Dict]) -> Dict:
        """投票策略"""
        vote_counts = defaultdict(int)
        for decision in decisions:
            key = str(decision)
            vote_counts[key] += 1
            
        winner = max(vote_counts.items(), key=lambda x: x[1])
        return eval(winner[0])  # 转回字典
        
    async def _priority_strategy(self, decisions: List[Dict]) -> Dict:
        """优先级策略"""
        return max(decisions, key=lambda d: d.get("priority", 0))
        
    async def _consensus_strategy(self, decisions: List[Dict]) -> Dict:
        """共识策略"""
        # 找到所有 Agent 都同意的部分
        common_keys = set(decisions[0].keys())
        for decision in decisions[1:]:
            common_keys &= set(decision.keys())
            
        consensus = {}
        for key in common_keys:
            values = [d[key] for d in decisions]
            if len(set(values)) == 1:  # 所有值相同
                consensus[key] = values[0]
                
        return consensus
```

### 4.3 死锁检测与解决

```python
class DeadlockDetector:
    """死锁检测器"""
    
    def __init__(self):
        self.wait_graph = defaultdict(set)
        
    def add_wait(self, agent_id: str, waiting_for: str):
        """添加等待关系"""
        self.wait_graph[agent_id].add(waiting_for)
        
    def remove_wait(self, agent_id: str, waiting_for: str):
        """移除等待关系"""
        self.wait_graph[agent_id].discard(waiting_for)
        
    def detect_deadlock(self) -> Optional[List[str]]:
        """检测死锁（返回死锁环）"""
        visited = set()
        rec_stack = set()
        
        def dfs(node: str, path: List[str]) -> Optional[List[str]]:
            visited.add(node)
            rec_stack.add(node)
            path.append(node)
            
            for neighbor in self.wait_graph[node]:
                if neighbor not in visited:
                    result = dfs(neighbor, path.copy())
                    if result:
                        return result
                elif neighbor in rec_stack:
                    # 找到环
                    cycle_start = path.index(neighbor)
                    return path[cycle_start:]
                    
            rec_stack.remove(node)
            return None
            
        for node in self.wait_graph:
            if node not in visited:
                cycle = dfs(node, [])
                if cycle:
                    return cycle
                    
        return None
        
    async def resolve_deadlock(self, cycle: List[str]):
        """解决死锁"""
        # 选择优先级最低的 Agent 回滚
        victim = min(cycle, key=lambda a: self._get_priority(a))
        
        # 回滚该 Agent 的操作
        await self._rollback_agent(victim)
```

## 5. 协作系统实现

### 5.1 完整的多 Agent 系统

```python
class MultiAgentSystem:
    """多 Agent 系统"""
    
    def __init__(self):
        self.agents = {}
        self.message_bus = MessageBus()
        self.protocol = CommunicationProtocol(self.message_bus)
        self.resource_manager = ResourceManager()
        self.conflict_resolver = ConflictResolver()
        self.deadlock_detector = DeadlockDetector()
        
    def register_agent(self, agent: BaseAgent):
        """注册 Agent"""
        self.agents[agent.profile.agent_id] = agent
        agent.protocol = self.protocol
        
    async def start(self):
        """启动系统"""
        # 启动所有 Agent
        tasks = [
            self._run_agent(agent)
            for agent in self.agents.values()
        ]
        await asyncio.gather(*tasks)
        
    async def _run_agent(self, agent: BaseAgent):
        """运行单个 Agent"""
        while True:
            # 接收消息
            message = await self.message_bus.receive(
                agent.profile.agent_id,
                timeout=1.0
            )
            
            if message:
                # 处理消息
                if message.message_type == MessageType.REQUEST:
                    result = await agent.execute(message.content)
                    await self.protocol.send_response(message, result)
```

### 5.2 实战示例：代码审查系统

```python
class CodeReviewSystem:
    """代码审查系统"""
    
    def __init__(self):
        self.system = MultiAgentSystem()
        
        # 创建 Agent
        self.analyzer = self._create_analyzer()
        self.security_checker = self._create_security_checker()
        self.style_checker = self._create_style_checker()
        self.reviewer = self._create_reviewer()
        
        # 注册 Agent
        for agent in [self.analyzer, self.security_checker, 
                     self.style_checker, self.reviewer]:
            self.system.register_agent(agent)
            
    async def review_code(self, code: str) -> Dict:
        """审查代码"""
        # 1. 并行分析
        analysis_tasks = [
            self.system.protocol.send_request(
                "coordinator",
                self.analyzer.profile.agent_id,
                {"code": code}
            ),
            self.system.protocol.send_request(
                "coordinator",
                self.security_checker.profile.agent_id,
                {"code": code}
            ),
            self.system.protocol.send_request(
                "coordinator",
                self.style_checker.profile.agent_id,
                {"code": code}
            )
        ]
        
        results = await asyncio.gather(*analysis_tasks)
        
        # 2. 综合审查
        review = await self.system.protocol.send_request(
            "coordinator",
            self.reviewer.profile.agent_id,
            {"analyses": results}
        )
        
        return review
```

## 6. 性能优化

### 6.1 消息队列优化

```python
class OptimizedMessageBus(MessageBus):
    """优化的消息总线"""
    
    def __init__(self, max_queue_size: int = 1000):
        super().__init__()
        self.message_queue = asyncio.Queue(maxsize=max_queue_size)
        self.message_cache = {}
        self.enable_batching = True
        self.batch_size = 10
        
    async def publish_batch(self, messages: List[Message]):
        """批量发布消息"""
        for message in messages:
            await self.publish(message)
```

### 6.2 负载均衡

```python
class LoadBalancer:
    """负载均衡器"""
    
    def __init__(self, agents: List[BaseAgent]):
        self.agents = agents
        self.agent_loads = {agent.profile.agent_id: 0 
                           for agent in agents}
        
    def select_agent(self, task: Dict) -> BaseAgent:
        """选择负载最低的 Agent"""
        # 过滤可用的 Agent
        available = [a for a in self.agents if a.status == "idle"]
        
        if not available:
            # 选择负载最低的
            agent_id = min(self.agent_loads.items(), key=lambda x: x[1])[0]
            return next(a for a in self.agents 
                       if a.profile.agent_id == agent_id)
        
        # 选择负载最低的可用 Agent
        return min(available, 
                  key=lambda a: self.agent_loads[a.profile.agent_id])
        
    def update_load(self, agent_id: str, delta: int):
        """更新负载"""
        self.agent_loads[agent_id] += delta
```

## 7. 监控与调试

### 7.1 系统监控

```python
class SystemMonitor:
    """系统监控器"""
    
    def __init__(self, system: MultiAgentSystem):
        self.system = system
        self.metrics = {
            "messages_sent": 0,
            "messages_received": 0,
            "tasks_completed": 0,
            "tasks_failed": 0,
            "average_response_time": 0
        }
        
    async def collect_metrics(self) -> Dict:
        """收集系统指标"""
        return {
            "agents": {
                agent_id: {
                    "status": agent.status,
                    "tasks_completed": len(agent.task_history),
                    "current_task": agent.current_task
                }
                for agent_id, agent in self.system.agents.items()
            },
            "message_bus": {
                "queue_size": self.system.message_bus.message_queue.qsize(),
                "total_messages": len(self.system.message_bus.message_history)
            },
            "metrics": self.metrics
        }
```

## 8. 总结

### 8.1 核心要点

- 多 Agent 系统需要清晰的通信协议
- 选择合适的协作模式很重要
- 冲突解决机制是系统稳定性的关键
- 性能优化需要从多个维度考虑

### 8.2 最佳实践

- 使用消息总线解耦 Agent
- 实现超时和重试机制
- 监控系统状态和性能
- 设计清晰的 Agent 职责边界

### 8.3 进阶方向

- 实现动态 Agent 注册和发现
- 添加 Agent 学习和适应能力
- 实现分布式多 Agent 系统
- 优化大规模 Agent 协作

## 9. 参考资源

- [Multi-Agent Systems](https://www.multiagent.com/)
- [LangGraph Multi-Agent](https://python.langchain.com/docs/langgraph)
- [CrewAI Documentation](https://docs.crewai.com/)
- [Agent Communication Languages](https://www.fipa.org/repository/aclspecs.html)

---

**@author erik.zhou**
