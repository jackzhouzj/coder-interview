# 自主 Agent 设计完整教程

> @author erik.zhou
> 
> 更新日期：2026-02-13

## 📋 教程概述

自主 Agent 是能够在最少人工干预下独立完成复杂任务的智能系统。本教程将深入讲解自主 Agent 的架构设计、决策机制、学习能力、安全性考虑等核心内容。

### 学习目标
- 理解自主 Agent 的核心特征
- 掌握 Agent 架构设计原则
- 学会实现决策和规划机制
- 能够设计 Agent 的学习系统
- 理解并实现安全控制机制

### 前置知识
- Python 高级编程
- 机器学习基础
- LangChain/LangGraph
- 强化学习概念
- 系统设计经验

## 1. 自主 Agent 核心特征

### 1.1 自主性定义

```python
from enum import Enum
from dataclasses import dataclass
from typing import List, Dict, Optional

class AutonomyLevel(Enum):
    """自主性级别"""
    MANUAL = 0          # 完全手动
    ASSISTED = 1        # 辅助决策
    SEMI_AUTO = 2       # 半自动
    CONDITIONAL = 3     # 条件自主
    HIGH_AUTO = 4       # 高度自主
    FULL_AUTO = 5       # 完全自主

@dataclass
class AutonomousCapability:
    """自主能力定义"""
    perception: bool = True      # 感知能力
    reasoning: bool = True       # 推理能力
    planning: bool = True        # 规划能力
    learning: bool = True        # 学习能力
    adaptation: bool = True      # 适应能力
    self_monitoring: bool = True # 自我监控
```

### 1.2 Agent 生命周期

```python
from abc import ABC, abstractmethod
import asyncio

class AutonomousAgent(ABC):
    """自主 Agent 基类"""
    
    def __init__(self, config: Dict):
        self.config = config
        self.state = "initialized"
        self.knowledge_base = {}
        self.experience_buffer = []
        self.goals = []
        
    async def lifecycle(self):
        """Agent 生命周期"""
        await self.initialize()
        
        while self.state != "terminated":
            # 1. 感知环境
            observations = await self.perceive()
            
            # 2. 更新内部状态
            await self.update_state(observations)
            
            # 3. 决策
            decision = await self.decide()
            
            # 4. 执行
            result = await self.execute(decision)
            
            # 5. 学习
            await self.learn(observations, decision, result)
            
            # 6. 自我监控
            await self.self_monitor()
            
    @abstractmethod
    async def perceive(self) -> Dict:
        """感知环境"""
        pass
        
    @abstractmethod
    async def decide(self) -> Dict:
        """做出决策"""
        pass
```

## 2. Agent 架构设计

### 2.1 分层架构


```python
class LayeredArchitecture:
    """分层架构"""
    
    def __init__(self):
        # 反应层：快速响应
        self.reactive_layer = ReactiveLayer()
        
        # 执行层：任务执行
        self.executive_layer = ExecutiveLayer()
        
        # 规划层：长期规划
        self.planning_layer = PlanningLayer()
        
    async def process(self, input_data: Dict) -> Dict:
        """处理输入"""
        # 1. 反应层快速处理
        if self.reactive_layer.should_handle(input_data):
            return await self.reactive_layer.handle(input_data)
            
        # 2. 执行层处理
        if self.executive_layer.should_handle(input_data):
            return await self.executive_layer.handle(input_data)
            
        # 3. 规划层处理
        return await self.planning_layer.handle(input_data)

class ReactiveLayer:
    """反应层（快速响应）"""
    
    def __init__(self):
        self.rules = []
        self.response_time_limit = 0.1  # 100ms
        
    def should_handle(self, input_data: Dict) -> bool:
        """判断是否应该在反应层处理"""
        return input_data.get("urgency") == "high"
        
    async def handle(self, input_data: Dict) -> Dict:
        """快速处理"""
        for rule in self.rules:
            if rule.matches(input_data):
                return await rule.execute()
        return {"status": "no_match"}

class ExecutiveLayer:
    """执行层（任务执行）"""
    
    def __init__(self):
        self.current_plan = None
        self.execution_context = {}
        
    async def handle(self, input_data: Dict) -> Dict:
        """执行任务"""
        if not self.current_plan:
            self.current_plan = await self._create_plan(input_data)
            
        return await self._execute_plan()

class PlanningLayer:
    """规划层（长期规划）"""
    
    def __init__(self):
        self.goals = []
        self.strategies = []
        
    async def handle(self, input_data: Dict) -> Dict:
        """制定计划"""
        # 1. 分析目标
        goals = await self._analyze_goals(input_data)
        
        # 2. 生成策略
        strategy = await self._generate_strategy(goals)
        
        # 3. 创建计划
        plan = await self._create_plan(strategy)
        
        return {"plan": plan}
```

### 2.2 BDI 架构（信念-愿望-意图）

```python
from typing import Set

class BDIAgent:
    """BDI Agent 实现"""
    
    def __init__(self):
        self.beliefs = {}      # 信念（对世界的认知）
        self.desires = set()   # 愿望（想要达成的目标）
        self.intentions = []   # 意图（承诺要执行的计划）
        
    async def update_beliefs(self, observations: Dict):
        """更新信念"""
        for key, value in observations.items():
            self.beliefs[key] = value
            
        # 移除过时的信念
        self._prune_beliefs()
        
    async def generate_desires(self) -> Set[str]:
        """生成愿望"""
        desires = set()
        
        # 基于信念生成愿望
        if self.beliefs.get("task_pending"):
            desires.add("complete_task")
            
        if self.beliefs.get("knowledge_gap"):
            desires.add("learn_new_skill")
            
        return desires
        
    async def select_intentions(self, desires: Set[str]) -> List[Dict]:
        """选择意图"""
        intentions = []
        
        for desire in desires:
            # 评估可行性
            if await self._is_feasible(desire):
                # 生成计划
                plan = await self._plan_for_desire(desire)
                intentions.append({
                    "desire": desire,
                    "plan": plan,
                    "priority": self._calculate_priority(desire)
                })
                
        # 按优先级排序
        intentions.sort(key=lambda x: x["priority"], reverse=True)
        return intentions
        
    async def execute_intentions(self):
        """执行意图"""
        for intention in self.intentions:
            if await self._still_valid(intention):
                await self._execute_plan(intention["plan"])
            else:
                # 重新规划
                await self._replan(intention)
```

### 2.3 认知架构

```python
class CognitiveArchitecture:
    """认知架构"""
    
    def __init__(self):
        # 工作记忆（短期）
        self.working_memory = WorkingMemory(capacity=7)
        
        # 长期记忆
        self.long_term_memory = LongTermMemory()
        
        # 注意力机制
        self.attention = AttentionMechanism()
        
        # 推理引擎
        self.reasoning_engine = ReasoningEngine()
        
    async def process_information(self, input_data: Dict) -> Dict:
        """处理信息"""
        # 1. 注意力选择
        focused_data = await self.attention.focus(input_data)
        
        # 2. 加载到工作记忆
        self.working_memory.add(focused_data)
        
        # 3. 从长期记忆检索相关信息
        relevant_memories = await self.long_term_memory.retrieve(
            focused_data
        )
        
        # 4. 推理
        result = await self.reasoning_engine.reason(
            self.working_memory.get_all(),
            relevant_memories
        )
        
        # 5. 存储到长期记忆
        await self.long_term_memory.store(result)
        
        return result

class WorkingMemory:
    """工作记忆"""
    
    def __init__(self, capacity: int = 7):
        self.capacity = capacity
        self.items = []
        
    def add(self, item: Dict):
        """添加项目"""
        self.items.append(item)
        if len(self.items) > self.capacity:
            # 移除最旧的项目
            self.items.pop(0)
```

## 3. 决策机制

### 3.1 基于规则的决策

```python
class RuleBasedDecision:
    """基于规则的决策"""
    
    def __init__(self):
        self.rules = []
        
    def add_rule(self, condition, action, priority: int = 1):
        """添加规则"""
        self.rules.append({
            "condition": condition,
            "action": action,
            "priority": priority
        })
        
    async def decide(self, context: Dict) -> Optional[Dict]:
        """做出决策"""
        # 按优先级排序
        sorted_rules = sorted(self.rules, 
                            key=lambda r: r["priority"], 
                            reverse=True)
        
        for rule in sorted_rules:
            if await rule["condition"](context):
                return await rule["action"](context)
                
        return None

# 示例：任务优先级规则
decision_maker = RuleBasedDecision()

decision_maker.add_rule(
    condition=lambda ctx: ctx.get("urgency") == "critical",
    action=lambda ctx: {"action": "immediate_response"},
    priority=10
)

decision_maker.add_rule(
    condition=lambda ctx: ctx.get("complexity") == "high",
    action=lambda ctx: {"action": "detailed_analysis"},
    priority=5
)
```

### 3.2 基于效用的决策

```python
class UtilityBasedDecision:
    """基于效用的决策"""
    
    def __init__(self):
        self.utility_functions = {}
        
    def add_utility_function(self, action: str, func):
        """添加效用函数"""
        self.utility_functions[action] = func
        
    async def decide(self, context: Dict, 
                    available_actions: List[str]) -> Dict:
        """选择效用最大的行动"""
        utilities = {}
        
        for action in available_actions:
            if action in self.utility_functions:
                utility = await self.utility_functions[action](context)
                utilities[action] = utility
                
        # 选择效用最大的行动
        best_action = max(utilities.items(), key=lambda x: x[1])
        
        return {
            "action": best_action[0],
            "expected_utility": best_action[1],
            "alternatives": utilities
        }

# 示例：定义效用函数
decision_maker = UtilityBasedDecision()

decision_maker.add_utility_function(
    "research",
    lambda ctx: ctx.get("knowledge_gain", 0) * 0.8 - ctx.get("time_cost", 0) * 0.2
)

decision_maker.add_utility_function(
    "execute",
    lambda ctx: ctx.get("task_completion", 0) * 0.9 - ctx.get("risk", 0) * 0.1
)
```

### 3.3 基于学习的决策

```python
import numpy as np

class ReinforcementLearningDecision:
    """基于强化学习的决策"""
    
    def __init__(self, state_size: int, action_size: int):
        self.state_size = state_size
        self.action_size = action_size
        self.q_table = np.zeros((state_size, action_size))
        self.learning_rate = 0.1
        self.discount_factor = 0.95
        self.epsilon = 0.1  # 探索率
        
    async def decide(self, state: int) -> int:
        """选择行动（ε-贪婪策略）"""
        if np.random.random() < self.epsilon:
            # 探索：随机选择
            return np.random.randint(self.action_size)
        else:
            # 利用：选择最优行动
            return np.argmax(self.q_table[state])
            
    async def learn(self, state: int, action: int, 
                   reward: float, next_state: int):
        """更新 Q 值"""
        current_q = self.q_table[state, action]
        max_next_q = np.max(self.q_table[next_state])
        
        # Q-learning 更新
        new_q = current_q + self.learning_rate * (
            reward + self.discount_factor * max_next_q - current_q
        )
        
        self.q_table[state, action] = new_q
```

## 4. 规划与推理

### 4.1 分层任务网络（HTN）

```python
class HTNPlanner:
    """分层任务网络规划器"""
    
    def __init__(self):
        self.methods = {}  # 任务分解方法
        self.operators = {}  # 原子操作
        
    def add_method(self, task: str, method):
        """添加任务分解方法"""
        if task not in self.methods:
            self.methods[task] = []
        self.methods[task].append(method)
        
    def add_operator(self, name: str, operator):
        """添加原子操作"""
        self.operators[name] = operator
        
    async def plan(self, task: str, state: Dict) -> List[Dict]:
        """生成计划"""
        return await self._decompose(task, state, [])
        
    async def _decompose(self, task: str, state: Dict, 
                        plan: List[Dict]) -> List[Dict]:
        """递归分解任务"""
        # 如果是原子操作
        if task in self.operators:
            plan.append({"operator": task, "state": state})
            return plan
            
        # 尝试所有分解方法
        for method in self.methods.get(task, []):
            if await method.precondition(state):
                subtasks = await method.decompose(state)
                
                # 递归分解子任务
                for subtask in subtasks:
                    plan = await self._decompose(subtask, state, plan)
                    
                return plan
                
        raise ValueError(f"No method found for task: {task}")

# 示例：定义任务分解
planner = HTNPlanner()

class WriteReportMethod:
    """写报告的分解方法"""
    
    async def precondition(self, state: Dict) -> bool:
        return state.get("data_collected", False)
        
    async def decompose(self, state: Dict) -> List[str]:
        return [
            "create_outline",
            "write_introduction",
            "write_body",
            "write_conclusion",
            "review_report"
        ]

planner.add_method("write_report", WriteReportMethod())
```

### 4.2 STRIPS 规划

```python
from dataclasses import dataclass
from typing import Set

@dataclass
class Action:
    """STRIPS 动作"""
    name: str
    preconditions: Set[str]
    add_effects: Set[str]
    delete_effects: Set[str]
    
class STRIPSPlanner:
    """STRIPS 规划器"""
    
    def __init__(self):
        self.actions = []
        
    def add_action(self, action: Action):
        """添加动作"""
        self.actions.append(action)
        
    async def plan(self, initial_state: Set[str], 
                  goal: Set[str]) -> List[Action]:
        """生成计划"""
        return await self._search(initial_state, goal, [])
        
    async def _search(self, state: Set[str], goal: Set[str], 
                     plan: List[Action]) -> Optional[List[Action]]:
        """搜索计划"""
        # 目标已达成
        if goal.issubset(state):
            return plan
            
        # 尝试所有可行的动作
        for action in self.actions:
            if action.preconditions.issubset(state):
                # 应用动作
                new_state = (state - action.delete_effects) | action.add_effects
                new_plan = plan + [action]
                
                # 递归搜索
                result = await self._search(new_state, goal, new_plan)
                if result:
                    return result
                    
        return None
```

## 5. 学习能力

### 5.1 经验学习

```python
class ExperienceLearning:
    """经验学习"""
    
    def __init__(self):
        self.experience_buffer = []
        self.patterns = {}
        
    async def record_experience(self, experience: Dict):
        """记录经验"""
        self.experience_buffer.append({
            **experience,
            "timestamp": datetime.now()
        })
        
        # 定期分析经验
        if len(self.experience_buffer) >= 100:
            await self._analyze_experiences()
            
    async def _analyze_experiences(self):
        """分析经验，提取模式"""
        # 聚类相似经验
        clusters = await self._cluster_experiences()
        
        # 提取每个聚类的模式
        for cluster in clusters:
            pattern = await self._extract_pattern(cluster)
            self.patterns[pattern["id"]] = pattern
            
    async def apply_learning(self, context: Dict) -> Optional[Dict]:
        """应用学到的模式"""
        # 找到最匹配的模式
        best_match = None
        best_score = 0
        
        for pattern in self.patterns.values():
            score = await self._match_score(context, pattern)
            if score > best_score:
                best_score = score
                best_match = pattern
                
        if best_score > 0.7:  # 阈值
            return best_match["action"]
            
        return None
```

### 5.2 元学习

```python
class MetaLearning:
    """元学习（学会学习）"""
    
    def __init__(self):
        self.learning_strategies = []
        self.strategy_performance = {}
        
    async def learn_task(self, task: Dict) -> Dict:
        """学习新任务"""
        # 1. 选择学习策略
        strategy = await self._select_strategy(task)
        
        # 2. 应用策略学习
        result = await strategy.learn(task)
        
        # 3. 评估策略效果
        performance = await self._evaluate_performance(result)
        
        # 4. 更新策略选择
        await self._update_strategy_selection(strategy, performance)
        
        return result
        
    async def _select_strategy(self, task: Dict):
        """选择最适合的学习策略"""
        task_type = task.get("type")
        
        # 基于历史表现选择
        if task_type in self.strategy_performance:
            best_strategy = max(
                self.strategy_performance[task_type].items(),
                key=lambda x: x[1]
            )[0]
            return best_strategy
            
        # 默认策略
        return self.learning_strategies[0]
```

### 5.3 迁移学习

```python
class TransferLearning:
    """迁移学习"""
    
    def __init__(self):
        self.source_knowledge = {}
        self.adaptation_rules = []
        
    async def transfer_knowledge(self, source_domain: str, 
                                target_domain: str, 
                                target_task: Dict) -> Dict:
        """迁移知识"""
        # 1. 提取源领域知识
        source_knowledge = self.source_knowledge.get(source_domain, {})
        
        # 2. 识别可迁移的知识
        transferable = await self._identify_transferable(
            source_knowledge,
            target_task
        )
        
        # 3. 适应目标领域
        adapted_knowledge = await self._adapt_knowledge(
            transferable,
            target_domain
        )
        
        # 4. 应用到目标任务
        result = await self._apply_knowledge(
            adapted_knowledge,
            target_task
        )
        
        return result
```

## 6. 安全性设计

### 6.1 行为约束

```python
class BehaviorConstraints:
    """行为约束"""
    
    def __init__(self):
        self.constraints = []
        self.violation_log = []
        
    def add_constraint(self, constraint):
        """添加约束"""
        self.constraints.append(constraint)
        
    async def validate_action(self, action: Dict) -> bool:
        """验证行动是否符合约束"""
        for constraint in self.constraints:
            if not await constraint.check(action):
                self.violation_log.append({
                    "action": action,
                    "constraint": constraint.name,
                    "timestamp": datetime.now()
                })
                return False
                
        return True

# 示例：定义约束
class ResourceLimitConstraint:
    """资源限制约束"""
    
    def __init__(self, max_cost: float):
        self.name = "resource_limit"
        self.max_cost = max_cost
        
    async def check(self, action: Dict) -> bool:
        return action.get("cost", 0) <= self.max_cost

class SafetyConstraint:
    """安全约束"""
    
    def __init__(self, forbidden_actions: Set[str]):
        self.name = "safety"
        self.forbidden_actions = forbidden_actions
        
    async def check(self, action: Dict) -> bool:
        return action.get("type") not in self.forbidden_actions
```

### 6.2 人类监督

```python
class HumanOversight:
    """人类监督"""
    
    def __init__(self):
        self.approval_required_actions = set()
        self.pending_approvals = []
        
    def require_approval(self, action_type: str):
        """设置需要批准的行动类型"""
        self.approval_required_actions.add(action_type)
        
    async def execute_with_oversight(self, action: Dict) -> Dict:
        """在监督下执行行动"""
        if action["type"] in self.approval_required_actions:
            # 请求人类批准
            approved = await self._request_approval(action)
            
            if not approved:
                return {
                    "status": "rejected",
                    "reason": "human_disapproval"
                }
                
        # 执行行动
        result = await self._execute(action)
        
        # 记录执行
        await self._log_execution(action, result)
        
        return result
        
    async def _request_approval(self, action: Dict) -> bool:
        """请求人类批准"""
        approval_request = {
            "action": action,
            "timestamp": datetime.now(),
            "status": "pending"
        }
        
        self.pending_approvals.append(approval_request)
        
        # 等待批准（实际实现需要UI交互）
        # 这里简化为自动批准
        return True
```

### 6.3 可解释性

```python
class ExplainableAgent:
    """可解释 Agent"""
    
    def __init__(self):
        self.decision_log = []
        
    async def make_decision(self, context: Dict) -> Dict:
        """做出可解释的决策"""
        # 1. 收集决策因素
        factors = await self._collect_factors(context)
        
        # 2. 评估每个因素
        evaluations = await self._evaluate_factors(factors)
        
        # 3. 做出决策
        decision = await self._decide(evaluations)
        
        # 4. 生成解释
        explanation = await self._generate_explanation(
            factors,
            evaluations,
            decision
        )
        
        # 5. 记录决策过程
        self.decision_log.append({
            "context": context,
            "factors": factors,
            "evaluations": evaluations,
            "decision": decision,
            "explanation": explanation,
            "timestamp": datetime.now()
        })
        
        return {
            "decision": decision,
            "explanation": explanation
        }
        
    async def _generate_explanation(self, factors: Dict, 
                                   evaluations: Dict, 
                                   decision: Dict) -> str:
        """生成决策解释"""
        explanation = f"决策：{decision['action']}\n\n"
        explanation += "主要考虑因素：\n"
        
        # 按重要性排序因素
        sorted_factors = sorted(
            evaluations.items(),
            key=lambda x: x[1]["importance"],
            reverse=True
        )
        
        for factor, eval_result in sorted_factors[:3]:
            explanation += f"- {factor}: {eval_result['reason']}\n"
            
        return explanation
```

## 7. 完整实现示例

### 7.1 研究助手 Agent

```python
class ResearchAssistantAgent(AutonomousAgent):
    """研究助手 Agent"""
    
    def __init__(self, config: Dict):
        super().__init__(config)
        self.architecture = LayeredArchitecture()
        self.decision_maker = UtilityBasedDecision()
        self.planner = HTNPlanner()
        self.learning = ExperienceLearning()
        self.constraints = BehaviorConstraints()
        
        # 添加约束
        self.constraints.add_constraint(
            ResourceLimitConstraint(max_cost=100)
        )
        
    async def perceive(self) -> Dict:
        """感知研究环境"""
        return {
            "available_papers": await self._search_papers(),
            "current_knowledge": self.knowledge_base,
            "research_gaps": await self._identify_gaps()
        }
        
    async def decide(self) -> Dict:
        """决策下一步行动"""
        context = {
            "knowledge_gap": len(self.knowledge_base) < 10,
            "time_cost": 5,
            "knowledge_gain": 8
        }
        
        decision = await self.decision_maker.decide(
            context,
            ["research", "synthesize", "write"]
        )
        
        # 验证约束
        if not await self.constraints.validate_action(decision):
            return {"action": "wait"}
            
        return decision
        
    async def execute(self, decision: Dict) -> Dict:
        """执行决策"""
        action = decision["action"]
        
        if action == "research":
            return await self._conduct_research()
        elif action == "synthesize":
            return await self._synthesize_knowledge()
        elif action == "write":
            return await self._write_report()
            
        return {"status": "unknown_action"}
        
    async def learn(self, observations: Dict, 
                   decision: Dict, result: Dict):
        """从经验中学习"""
        experience = {
            "observations": observations,
            "decision": decision,
            "result": result,
            "success": result.get("status") == "success"
        }
        
        await self.learning.record_experience(experience)
```

## 8. 最佳实践

### 8.1 设计原则

```python
# ✅ 好的设计
class WellDesignedAgent:
    """设计良好的 Agent"""
    
    def __init__(self):
        # 1. 清晰的职责边界
        self.perception_module = PerceptionModule()
        self.decision_module = DecisionModule()
        self.execution_module = ExecutionModule()
        
        # 2. 可配置的行为
        self.config = self._load_config()
        
        # 3. 完善的监控
        self.monitor = AgentMonitor()
        
        # 4. 安全约束
        self.safety = SafetyModule()
        
    async def run(self):
        """运行 Agent"""
        while True:
            # 监控状态
            await self.monitor.check_health()
            
            # 安全检查
            if not await self.safety.is_safe():
                await self.safety.enter_safe_mode()
                continue
                
            # 正常运行
            await self._execute_cycle()
```

### 8.2 常见陷阱

```python
# ❌ 避免的设计
class PoorlyDesignedAgent:
    """设计不良的 Agent"""
    
    async def run(self):
        # 1. 没有错误处理
        result = await self.execute()  # 可能崩溃
        
        # 2. 无限循环没有退出条件
        while True:
            await self.do_something()
            
        # 3. 没有资源限制
        self.memory.append(data)  # 可能内存溢出
        
        # 4. 没有安全检查
        await self.execute_action(user_input)  # 危险
```

## 9. 总结

### 9.1 核心要点

- 自主 Agent 需要感知、决策、执行、学习的完整循环
- 架构设计要考虑分层和模块化
- 决策机制可以基于规则、效用或学习
- 安全性是自主 Agent 的首要考虑

### 9.2 学习路径

1. 理解自主性的本质和级别
2. 掌握基础架构设计（分层、BDI、认知）
3. 实现决策和规划机制
4. 添加学习能力
5. 完善安全控制

### 9.3 进阶方向

- 实现更复杂的推理能力
- 集成深度学习模型
- 实现多模态感知
- 优化长期规划能力
- 提升可解释性

## 10. 参考资源

- [Autonomous Agents](https://www.autonomousagents.org/)
- [BDI Architecture](https://en.wikipedia.org/wiki/Belief%E2%80%93desire%E2%80%93intention_software_model)
- [Reinforcement Learning](https://www.deepmind.com/learning-resources/reinforcement-learning-lecture-series-2021)
- [AI Safety](https://www.safe.ai/)

---

**@author erik.zhou**
