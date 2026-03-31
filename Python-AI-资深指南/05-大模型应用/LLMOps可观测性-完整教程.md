# LLMOps 可观测性完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

LLM 应用的可观测性（Observability）是确保 AI 系统在生产环境中可靠运行的关键。本教程系统讲解 LLMOps 可观测性的全链路方案，涵盖 Tracing、评估、成本监控、Prompt 版本管理、告警和持续优化，并对比主流工具（Langfuse、LangSmith、Portkey、LangWatch）的选型。

### 版本信息
- **重要程度**：⭐⭐⭐⭐⭐（必学，生产必备）
- **难度等级**：⭐⭐⭐（中等）
- **预计学习时间**：15-20 小时

### 学习目标
- 理解 LLMOps 可观测性的核心维度
- 掌握 Tracing 全链路追踪的实现
- 学会构建 LLM 评估和质量监控体系
- 掌握成本分析和优化方法
- 能够搭建生产级 LLMOps 监控平台

### 前置知识
- LLM 应用开发基础
- LangChain / OpenAI API 使用经验
- 基础的监控概念

---

## 1. LLMOps 可观测性全景

### 1.1 五大核心维度

```
LLMOps 可观测性五维模型

1. Tracing（追踪）
   ├── 请求全链路追踪
   ├── LLM 调用、工具调用、检索过程
   ├── 每一步的输入/输出/耗时
   └── 父子 Span 关系

2. Quality（质量）
   ├── 回答准确性评估
   ├── 幻觉检测
   ├── 相关性评分
   └── 用户满意度

3. Cost（成本）
   ├── Token 消耗统计
   ├── 按模型/功能/用户的成本分析
   ├── 成本趋势和预测
   └── 预算告警

4. Latency（延迟）
   ├── 端到端延迟
   ├── LLM 推理延迟
   ├── 检索延迟
   └── P50/P95/P99 分位数

5. Safety（安全）
   ├── Prompt 注入检测
   ├── PII 泄露监控
   ├── 有害内容检测
   └── 合规审计
```

### 1.2 工具选型对比

| 工具 | 类型 | Tracing | 评估 | 成本 | Prompt管理 | 部署 |
|------|------|---------|------|------|-----------|------|
| **Langfuse** | 开源 | ✅ | ✅ | ✅ | ✅ | 自托管/云 |
| **LangSmith** | 商业 | ✅ | ✅ | ✅ | ✅ | 云 |
| **Portkey** | 商业 | ✅ | ✅ | ✅ | ❌ | 云 |
| **LangWatch** | 开源 | ✅ | ✅ | ✅ | ❌ | 自托管/云 |
| **Arize Phoenix** | 开源 | ✅ | ✅ | ❌ | ❌ | 自托管 |
| **OpenTelemetry** | 标准 | ✅ | ❌ | ❌ | ❌ | 自建 |

---

## 2. Tracing 全链路追踪

### 2.1 使用 Langfuse 追踪

```python
"""
Langfuse 是最成熟的开源 LLMOps 平台
支持 LangChain、OpenAI、LlamaIndex 等主流框架
"""
from langfuse import Langfuse
from langfuse.decorators import observe, langfuse_context

langfuse = Langfuse()

# 🔥 方式一：装饰器（最简单）
@observe()
def rag_pipeline(question: str) -> str:
    """RAG 管道 — 自动追踪每一步"""
    
    # 设置用户信息
    langfuse_context.update_current_trace(
        user_id="user-001",
        session_id="session-abc",
        tags=["rag", "production"]
    )
    
    # 步骤 1：检索
    docs = retrieve_documents(question)
    
    # 步骤 2：生成
    answer = generate_answer(question, docs)
    
    return answer

@observe()
def retrieve_documents(query: str) -> list:
    """检索文档 — 自动创建子 Span"""
    langfuse_context.update_current_observation(
        metadata={"retriever": "qdrant", "top_k": 5}
    )
    # 实际检索逻辑
    return ["doc1", "doc2", "doc3"]

@observe(as_type="generation")
def generate_answer(question: str, docs: list) -> str:
    """LLM 生成 — 标记为 generation 类型"""
    from openai import OpenAI
    client = OpenAI()
    
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": f"基于以下文档回答：{docs}"},
            {"role": "user", "content": question}
        ]
    )
    
    # 自动记录 Token 使用
    langfuse_context.update_current_observation(
        usage={
            "input": response.usage.prompt_tokens,
            "output": response.usage.completion_tokens
        },
        model="gpt-4o"
    )
    
    return response.choices[0].message.content
```

### 2.2 LangChain 集成追踪

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema.output_parser import StrOutputParser
from langfuse.callback import CallbackHandler

# 🔥 一行代码集成 Langfuse
langfuse_handler = CallbackHandler()

# 创建 LangChain 链
llm = ChatOpenAI(model="gpt-4o")
prompt = ChatPromptTemplate.from_template("回答问题：{question}")
chain = prompt | llm | StrOutputParser()

# 自动追踪整个链的执行
result = chain.invoke(
    {"question": "什么是 RAG？"},
    config={
        "callbacks": [langfuse_handler],
        "metadata": {"user_id": "user-001"}
    }
)
```

### 2.3 OpenTelemetry 标准追踪

```python
"""
使用 OpenTelemetry 标准实现追踪
优势：与现有 APM 系统（Datadog/Jaeger/Grafana）集成
"""
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import SimpleSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

# 配置 OpenTelemetry
provider = TracerProvider()
exporter = OTLPSpanExporter(endpoint="http://localhost:4317")
provider.add_span_processor(SimpleSpanProcessor(exporter))
trace.set_tracer_provider(provider)

tracer = trace.get_tracer("llm-app")

# 🔥 手动追踪
def rag_query(question: str) -> str:
    with tracer.start_as_current_span("rag_pipeline") as span:
        span.set_attribute("user.question", question)
        
        # 检索
        with tracer.start_as_current_span("retrieval") as retrieval_span:
            docs = retrieve(question)
            retrieval_span.set_attribute("docs.count", len(docs))
        
        # LLM 生成
        with tracer.start_as_current_span("llm_generation") as gen_span:
            answer = generate(question, docs)
            gen_span.set_attribute("model", "gpt-4o")
            gen_span.set_attribute("tokens.total", 500)
        
        span.set_attribute("answer.length", len(answer))
        return answer
```

---

## 3. 质量评估体系

### 3.1 在线评估（实时）

```python
from langfuse import Langfuse

langfuse = Langfuse()

class OnlineEvaluator:
    """在线质量评估器"""
    
    def __init__(self):
        self.langfuse = Langfuse()
    
    def score_response(self, trace_id: str, question: str,
                       answer: str, sources: list = None):
        """对每次响应进行评分"""
        
        # 1. 用户反馈评分（由前端收集）
        # langfuse.score(trace_id=trace_id, name="user-feedback", value=0.9)
        
        # 2. LLM-as-Judge 自动评分
        from openai import OpenAI
        client = OpenAI()
        
        judge_response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"""评估回答质量（0-1分）：
问题：{question}
回答：{answer}
只返回数字。"""
            }],
            temperature=0
        )
        
        try:
            score = float(judge_response.choices[0].message.content.strip())
        except ValueError:
            score = 0.5
        
        # 记录到 Langfuse
        self.langfuse.score(
            trace_id=trace_id,
            name="quality-auto",
            value=score,
            comment="LLM-as-Judge 自动评估"
        )
        
        # 3. 幻觉检测评分
        if sources:
            hallucination_score = self._check_hallucination(answer, sources)
            self.langfuse.score(
                trace_id=trace_id,
                name="faithfulness",
                value=hallucination_score
            )
        
        return score
    
    def _check_hallucination(self, answer: str, sources: list) -> float:
        """幻觉检测"""
        # 简化版：检查答案中的关键词是否出现在源文档中
        source_text = " ".join(sources).lower()
        answer_words = set(answer.lower().split())
        source_words = set(source_text.split())
        
        overlap = len(answer_words & source_words)
        return min(overlap / max(len(answer_words), 1), 1.0)
```

### 3.2 离线评估（批量）

```python
from langfuse import Langfuse

langfuse = Langfuse()

def run_offline_evaluation(dataset_name: str, pipeline_fn):
    """离线批量评估"""
    
    # 从 Langfuse 获取数据集
    dataset = langfuse.get_dataset(dataset_name)
    
    results = []
    for item in dataset.items:
        question = item.input["question"]
        expected = item.expected_output
        
        # 运行管道
        actual = pipeline_fn(question)
        
        # 评估
        from ragas.metrics import faithfulness, answer_relevancy
        # ... RAGAS 评估逻辑
        
        # 记录结果
        item.link(
            trace_id=actual.get("trace_id"),
            run_name=f"eval-{dataset_name}"
        )
        
        results.append({
            "question": question,
            "expected": expected,
            "actual": actual["answer"],
        })
    
    return results
```

---

## 4. 成本监控与优化

### 4.1 成本追踪

```python
# Token 价格表（2026 年参考价格）
TOKEN_PRICES = {
    "gpt-4o": {"input": 2.50, "output": 10.00},        # per 1M tokens
    "gpt-4o-mini": {"input": 0.15, "output": 0.60},
    "claude-sonnet-4": {"input": 3.00, "output": 15.00},
    "claude-haiku-3.5": {"input": 0.80, "output": 4.00},
    "deepseek-v3": {"input": 0.27, "output": 1.10},
}

class CostTracker:
    """成本追踪器"""
    
    def __init__(self):
        self.records = []
    
    def record(self, model: str, input_tokens: int, output_tokens: int,
               feature: str = "default", user_id: str = None):
        """记录一次调用的成本"""
        prices = TOKEN_PRICES.get(model, {"input": 5.0, "output": 15.0})
        cost = (
            input_tokens / 1_000_000 * prices["input"] +
            output_tokens / 1_000_000 * prices["output"]
        )
        
        self.records.append({
            "model": model,
            "input_tokens": input_tokens,
            "output_tokens": output_tokens,
            "cost_usd": cost,
            "feature": feature,
            "user_id": user_id,
            "timestamp": datetime.now()
        })
        
        return cost
    
    def get_daily_cost(self) -> dict:
        """获取每日成本"""
        from collections import defaultdict
        daily = defaultdict(float)
        for r in self.records:
            day = r["timestamp"].strftime("%Y-%m-%d")
            daily[day] += r["cost_usd"]
        return dict(daily)
    
    def get_cost_by_model(self) -> dict:
        """按模型统计成本"""
        from collections import defaultdict
        by_model = defaultdict(float)
        for r in self.records:
            by_model[r["model"]] += r["cost_usd"]
        return dict(by_model)
    
    def get_cost_by_feature(self) -> dict:
        """按功能统计成本"""
        from collections import defaultdict
        by_feature = defaultdict(float)
        for r in self.records:
            by_feature[r["feature"]] += r["cost_usd"]
        return dict(by_feature)
```

### 4.2 成本优化策略

```python
"""
LLM 成本优化七大策略：

1. 模型降级（Model Downgrade）
   - 简单任务用 gpt-4o-mini 代替 gpt-4o
   - 节省 90%+ 成本，质量损失 < 10%

2. 缓存（Caching）
   - 相同问题直接返回缓存结果
   - 语义缓存：相似问题也命中缓存

3. Prompt 压缩
   - 精简系统提示词
   - 压缩检索文档（只保留关键段落）

4. 批处理（Batching）
   - 合并多个请求为一次调用
   - 使用 Batch API（OpenAI 50% 折扣）

5. 流量控制
   - 按用户/功能设置 Token 预算
   - 超限降级或排队

6. 模型路由
   - 简单问题 → 小模型
   - 复杂问题 → 大模型
   - 使用分类器自动路由

7. 开源模型替代
   - 非核心功能用开源模型
   - DeepSeek / Qwen 性价比极高
"""

import hashlib
import json

class SemanticCache:
    """语义缓存"""
    
    def __init__(self, vectorstore, ttl_seconds: int = 3600):
        self.vectorstore = vectorstore
        self.cache = {}
        self.ttl = ttl_seconds
    
    def get(self, query: str, threshold: float = 0.95) -> str | None:
        """查询缓存"""
        results = self.vectorstore.similarity_search_with_score(query, k=1)
        if results and results[0][1] >= threshold:
            cache_key = results[0][0].metadata.get("cache_key")
            cached = self.cache.get(cache_key)
            if cached and (time.time() - cached["time"]) < self.ttl:
                return cached["answer"]
        return None
    
    def set(self, query: str, answer: str):
        """设置缓存"""
        cache_key = hashlib.md5(query.encode()).hexdigest()
        self.cache[cache_key] = {"answer": answer, "time": time.time()}
        self.vectorstore.add_texts(
            [query], metadatas=[{"cache_key": cache_key}]
        )
```

---

## 5. Prompt 版本管理

### 5.1 使用 Langfuse 管理 Prompt

```python
from langfuse import Langfuse

langfuse = Langfuse()

# 🔥 获取 Prompt（支持版本和标签）
prompt = langfuse.get_prompt("rag-qa-prompt", label="production")

# 编译 Prompt（填充变量）
messages = prompt.compile(
    context="检索到的文档内容...",
    question="用户的问题"
)

# 在 LLM 调用中使用
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages
)

# 🔥 Prompt 与 Trace 关联（用于分析不同版本的效果）
trace = langfuse.trace(name="rag-query")
generation = trace.generation(
    name="llm-call",
    model="gpt-4o",
    prompt=prompt,  # 关联 Prompt 版本
    input=messages,
    output=response.choices[0].message.content
)
```

### 5.2 Prompt A/B 测试

```python
import random

def ab_test_prompt(question: str, user_id: str):
    """Prompt A/B 测试"""
    # 随机分配变体
    variant = "A" if hash(user_id) % 2 == 0 else "B"
    
    # 获取对应版本的 Prompt
    prompt = langfuse.get_prompt("rag-qa-prompt", label=f"variant-{variant}")
    
    # 执行并追踪
    trace = langfuse.trace(
        name="ab-test",
        user_id=user_id,
        metadata={"variant": variant, "prompt_version": prompt.version}
    )
    
    # ... 执行 LLM 调用 ...
    
    # 在 Langfuse Dashboard 中对比两个变体的质量和成本
```

---

## 6. 告警与仪表盘

### 6.1 告警规则

```python
class LLMAlertManager:
    """LLM 告警管理器"""
    
    def __init__(self):
        self.rules = []
    
    def add_rule(self, name: str, condition, severity: str = "warning"):
        self.rules.append({"name": name, "condition": condition, "severity": severity})
    
    def check(self, metrics: dict) -> list:
        """检查告警"""
        alerts = []
        for rule in self.rules:
            if rule["condition"](metrics):
                alerts.append({
                    "name": rule["name"],
                    "severity": rule["severity"],
                    "metrics": metrics
                })
        return alerts

# 🔥 配置告警规则
alert_mgr = LLMAlertManager()

# 成本告警
alert_mgr.add_rule(
    "daily_cost_high",
    lambda m: m.get("daily_cost_usd", 0) > 100,
    severity="critical"
)

# 质量告警
alert_mgr.add_rule(
    "quality_drop",
    lambda m: m.get("avg_quality_score", 1) < 0.6,
    severity="warning"
)

# 延迟告警
alert_mgr.add_rule(
    "high_latency",
    lambda m: m.get("p95_latency_ms", 0) > 5000,
    severity="warning"
)

# 错误率告警
alert_mgr.add_rule(
    "high_error_rate",
    lambda m: m.get("error_rate", 0) > 0.05,
    severity="critical"
)

# 安全告警
alert_mgr.add_rule(
    "injection_detected",
    lambda m: m.get("injection_count", 0) > 0,
    severity="critical"
)
```

---

## 7. 生产级监控架构

```python
"""
生产级 LLMOps 监控架构

数据采集层
├── Langfuse SDK（Tracing + 评估）
├── OpenTelemetry（标准化追踪）
├── 自定义 Metrics（成本/安全）
└── 用户反馈收集

存储层
├── Langfuse（Trace + Score + Prompt）
├── PostgreSQL（业务数据）
├── ClickHouse（分析数据）
└── Redis（实时缓存）

分析层
├── Langfuse Dashboard（Trace 分析）
├── Grafana（指标可视化）
├── 自定义报表（成本/质量趋势）
└── A/B 测试分析

告警层
├── 成本超限告警
├── 质量下降告警
├── 安全事件告警
├── 延迟异常告警
└── 通知渠道（Slack/邮件/钉钉）

优化层
├── Prompt 版本管理和 A/B 测试
├── 模型路由优化
├── 缓存策略优化
└── 持续评估和改进
"""
```

---

## 8. 总结

### 核心要点
1. LLMOps 可观测性覆盖 Tracing、质量、成本、延迟、安全五个维度
2. Langfuse 是最成熟的开源方案，支持自托管和云服务
3. 成本优化的核心是模型路由 + 缓存 + Prompt 压缩
4. Prompt 版本管理和 A/B 测试是持续优化的基础
5. 生产环境必须有完善的告警体系

### 学习路径
1. 集成 Langfuse 实现基础 Tracing
2. 建立质量评估体系（LLM-as-Judge + RAGAS）
3. 搭建成本监控和优化方案
4. 实现 Prompt 版本管理
5. 配置告警规则和仪表盘

---

## 🔗 相关资源

- [Langfuse 文档](https://langfuse.com/docs)
- [LangSmith 文档](https://docs.smith.langchain.com/)
- [Portkey 文档](https://portkey.ai/docs)
- [OpenTelemetry for LLM](https://opentelemetry.io/)
- [LangWatch](https://langwatch.ai/)

---

**@author erik.zhou**
