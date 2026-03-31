# 高级 RAG 技术完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

本教程涵盖 2025-2026 年 RAG 领域的前沿技术，包括 GraphRAG、Agentic RAG、Self-RAG、多模态 RAG 等高级范式。这些技术将基础 RAG 从"单次检索+生成"提升为具备多跳推理、自我纠错、知识图谱增强的智能系统。

### 版本信息
- **重要程度**：⭐⭐⭐⭐⭐（必学）
- **难度等级**：⭐⭐⭐⭐⭐（高级）
- **预计学习时间**：25-35 小时
- **前置知识**：RAG 基础、LangChain/LlamaIndex、向量数据库

### 学习目标
- 掌握 GraphRAG 的原理和实现
- 理解 Agentic RAG 的架构设计
- 学会 Self-RAG 自我反思机制
- 掌握高级 Chunking 策略
- 了解 RAG 评估框架（RAGAS）
- 能够设计生产级高级 RAG 系统

---

## 1. RAG 技术演进

### 1.1 RAG 发展阶段

```
RAG 技术演进路线

Naive RAG（2023）
├── 简单的"检索-生成"流水线
├── 单次检索，无反馈
└── 问题：检索质量差、无法多跳推理

Advanced RAG（2024）
├── 查询重写、混合检索、重排序
├── 文档压缩、上下文增强
└── 问题：仍是静态流水线

Modular RAG（2024-2025）
├── 模块化设计，可插拔组件
├── 自适应检索策略
└── 问题：缺乏自主决策能力

Agentic RAG（2025-2026）🔥
├── Agent 驱动的动态 RAG
├── 多轮检索、自我纠错、工具使用
├── 知识图谱增强（GraphRAG）
└── 当前最先进的 RAG 范式
```

### 1.2 高级 RAG 技术全景

| 技术 | 核心思想 | 适用场景 |
|------|----------|----------|
| **GraphRAG** | 知识图谱增强检索 | 多跳推理、关系查询 |
| **Agentic RAG** | Agent 驱动的动态检索 | 复杂问题、多步骤任务 |
| **Self-RAG** | 自我反思和纠错 | 高准确性要求 |
| **Corrective RAG** | 检索结果校正 | 减少幻觉 |
| **Adaptive RAG** | 自适应检索策略 | 混合查询类型 |
| **Multi-Modal RAG** | 多模态检索和生成 | 图文混合场景 |
| **HyDE** | 假设文档嵌入 | 提升检索召回率 |

---

## 2. GraphRAG

### 2.1 GraphRAG 原理

```python
"""
GraphRAG 核心思想：
1. 将文档构建为知识图谱（实体 + 关系）
2. 对图进行社区检测，生成层级摘要
3. 查询时结合图结构和向量检索
4. 支持全局摘要查询和局部精确查询
"""

### 2.2 使用 Microsoft GraphRAG

```bash
# 安装 Microsoft GraphRAG
pip install graphrag
```

```python
"""
Microsoft GraphRAG 使用示例
"""
import asyncio
from graphrag.index import run_pipeline
from graphrag.query.structured_search.local_search import LocalSearch
from graphrag.query.structured_search.global_search import GlobalSearch

# 步骤 1：初始化项目
# graphrag init --root ./my_graphrag

# 步骤 2：索引构建（从文档构建知识图谱）
# graphrag index --root ./my_graphrag

# 步骤 3：查询
# 局部查询（精确的实体/关系查询）
# graphrag query --root ./my_graphrag --method local "谁是CEO？"

# 全局查询（需要全局理解的摘要性问题）
# graphrag query --root ./my_graphrag --method global "这个行业的主要趋势是什么？"
```

### 2.3 使用 LlamaIndex 构建 GraphRAG

```python
"""
使用 LlamaIndex 的 PropertyGraphIndex 构建 GraphRAG
"""
from llama_index.core import SimpleDirectoryReader, PropertyGraphIndex
from llama_index.core.indices.property_graph import (
    SimpleLLMPathExtractor,
    ImplicitPathExtractor
)
from llama_index.llms.openai import OpenAI
from llama_index.embeddings.openai import OpenAIEmbedding

# 加载文档
documents = SimpleDirectoryReader("./data").load_data()

# 🔥 构建属性图索引
index = PropertyGraphIndex.from_documents(
    documents,
    llm=OpenAI(model="gpt-4o-mini"),
    embed_model=OpenAIEmbedding(),
    # 知识提取器
    kg_extractors=[
        SimpleLLMPathExtractor(
            llm=OpenAI(model="gpt-4o-mini"),
            max_paths_per_chunk=10
        ),
        ImplicitPathExtractor()  # 基于文档结构的隐式关系
    ],
    show_progress=True
)

# 🔥 查询（自动结合图结构和向量检索）
query_engine = index.as_query_engine(
    include_text=True,  # 包含原始文本
    similarity_top_k=5
)

response = query_engine.query("Python和机器学习之间有什么关系？")
print(response)

# 保存和加载
index.storage_context.persist("./graph_store")
```

### 2.4 自定义知识图谱构建

```python
"""
自定义知识图谱构建流程
"""
from openai import OpenAI
from typing import List, Tuple
import json

client = OpenAI()

def extract_entities_and_relations(text: str) -> dict:
    """使用 LLM 从文本中提取实体和关系"""
    
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{
            "role": "user",
            "content": f"""从以下文本中提取实体和关系，返回JSON格式：

文本：{text}

返回格式：
{{
    "entities": [
        {{"name": "实体名", "type": "类型", "description": "描述"}}
    ],
    "relations": [
        {{"source": "源实体", "relation": "关系", "target": "目标实体"}}
    ]
}}"""
        }],
        response_format={"type": "json_object"}
    )
    
    return json.loads(response.choices[0].message.content)

def build_knowledge_graph(documents: List[str]) -> dict:
    """从文档列表构建知识图谱"""
    all_entities = {}
    all_relations = []
    
    for doc in documents:
        result = extract_entities_and_relations(doc)
        
        for entity in result.get("entities", []):
            name = entity["name"]
            if name not in all_entities:
                all_entities[name] = entity
        
        all_relations.extend(result.get("relations", []))
    
    return {
        "entities": list(all_entities.values()),
        "relations": all_relations
    }

# 使用
docs = [
    "Python是一种编程语言，由Guido van Rossum创建。",
    "PyTorch是一个深度学习框架，基于Python开发。",
    "RAG技术结合了检索和生成，常用Python实现。"
]

kg = build_knowledge_graph(docs)
print(f"实体数: {len(kg['entities'])}")
print(f"关系数: {len(kg['relations'])}")
for r in kg["relations"]:
    print(f"  {r['source']} --[{r['relation']}]--> {r['target']}")
```

---

## 3. Agentic RAG

### 3.1 Agentic RAG 架构

```python
"""
Agentic RAG：Agent 驱动的动态 RAG

与传统 RAG 的区别：
- 传统 RAG：查询 → 检索 → 生成（单次、线性）
- Agentic RAG：Agent 自主决定何时检索、检索什么、是否需要再次检索

核心能力：
1. 查询分析和规划
2. 多轮迭代检索
3. 检索结果评估和自我纠错
4. 工具使用（搜索、计算、API调用）
5. 答案综合和引用
"""
```

### 3.2 使用 LangGraph 实现 Agentic RAG

```python
from typing import TypedDict, Annotated, Literal
from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain_core.messages import HumanMessage
import operator

# 定义状态
class AgenticRAGState(TypedDict):
    question: str
    retrieved_docs: Annotated[list, operator.add]
    generation: str
    search_queries: list[str]
    iteration: int
    is_relevant: bool
    is_sufficient: bool

# 初始化组件
llm = ChatOpenAI(model="gpt-4o")
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.load_local("./knowledge_base", embeddings)
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# 🔥 节点 1：查询分析
def analyze_query(state: AgenticRAGState) -> dict:
    """分析查询，决定检索策略"""
    response = llm.invoke(f"""分析以下问题，生成1-3个搜索查询：
    
问题：{state['question']}

返回JSON格式的查询列表。""")
    
    queries = [state["question"]]  # 至少包含原始查询
    return {"search_queries": queries, "iteration": 0}

# 🔥 节点 2：检索
def retrieve(state: AgenticRAGState) -> dict:
    """执行检索"""
    all_docs = []
    for query in state["search_queries"]:
        docs = retriever.get_relevant_documents(query)
        all_docs.extend([doc.page_content for doc in docs])
    
    # 去重
    unique_docs = list(set(all_docs))
    return {
        "retrieved_docs": unique_docs,
        "iteration": state["iteration"] + 1
    }

# 🔥 节点 3：评估检索质量
def evaluate_retrieval(state: AgenticRAGState) -> dict:
    """评估检索结果是否相关和充分"""
    docs_text = "\n".join(state["retrieved_docs"][:5])
    
    response = llm.invoke(f"""评估以下检索结果对回答问题的帮助程度：

问题：{state['question']}
检索结果：{docs_text[:2000]}

回答两个问题（yes/no）：
1. 检索结果是否与问题相关？
2. 检索结果是否足够回答问题？""")
    
    content = response.content.lower()
    is_relevant = "yes" in content.split("\n")[0] if "\n" in content else True
    is_sufficient = "yes" in content.split("\n")[-1] if "\n" in content else True
    
    return {"is_relevant": is_relevant, "is_sufficient": is_sufficient}

# 🔥 节点 4：生成答案
def generate(state: AgenticRAGState) -> dict:
    """基于检索结果生成答案"""
    docs_text = "\n---\n".join(state["retrieved_docs"][:5])
    
    response = llm.invoke(f"""基于以下参考资料回答问题。
如果资料不足以回答，请明确说明。

参考资料：
{docs_text}

问题：{state['question']}

请给出详细回答，并标注信息来源。""")
    
    return {"generation": response.content}

# 🔥 路由：决定下一步
def should_retry(state: AgenticRAGState) -> Literal["generate", "retrieve", "end"]:
    """决定是否需要重新检索"""
    if state["iteration"] >= 3:
        return "generate"  # 超过最大迭代次数，直接生成
    
    if not state.get("is_relevant", True):
        return "retrieve"  # 不相关，重新检索
    
    if not state.get("is_sufficient", True):
        return "retrieve"  # 不充分，补充检索
    
    return "generate"

# 🔥 构建 Agentic RAG 图
workflow = StateGraph(AgenticRAGState)

workflow.add_node("analyze", analyze_query)
workflow.add_node("retrieve", retrieve)
workflow.add_node("evaluate", evaluate_retrieval)
workflow.add_node("generate", generate)

workflow.add_edge(START, "analyze")
workflow.add_edge("analyze", "retrieve")
workflow.add_edge("retrieve", "evaluate")

workflow.add_conditional_edges(
    "evaluate",
    should_retry,
    {
        "generate": "generate",
        "retrieve": "retrieve",
        "end": END
    }
)
workflow.add_edge("generate", END)

app = workflow.compile()

# 运行
result = app.invoke({
    "question": "GraphRAG和传统RAG有什么区别？各自适用什么场景？",
    "retrieved_docs": [],
    "generation": "",
    "search_queries": [],
    "iteration": 0,
    "is_relevant": True,
    "is_sufficient": True
})

print(result["generation"])
```

---

## 4. Self-RAG（自我反思 RAG）

### 4.1 Self-RAG 原理

```python
"""
Self-RAG 核心机制：

1. 检索判断（Retrieve Token）
   - 判断是否需要检索：有些问题不需要外部知识

2. 相关性判断（ISREL Token）
   - 判断检索到的文档是否与问题相关

3. 支持度判断（ISSUP Token）
   - 判断生成的答案是否被检索文档支持

4. 有用性判断（ISUSE Token）
   - 判断最终答案是否有用
"""
```

### 4.2 Self-RAG 实现

```python
from typing import TypedDict, Literal
from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o")

class SelfRAGState(TypedDict):
    question: str
    documents: list[str]
    generation: str
    needs_retrieval: bool
    relevance_score: float
    support_score: float
    usefulness_score: float

# 🔥 步骤 1：判断是否需要检索
def check_need_retrieval(state: SelfRAGState) -> dict:
    response = llm.invoke(f"""判断以下问题是否需要检索外部知识来回答。
    
问题：{state['question']}

如果是常识性问题或简单计算，回答 NO。
如果需要特定领域知识或最新信息，回答 YES。
只回答 YES 或 NO。""")
    
    needs = "yes" in response.content.lower()
    return {"needs_retrieval": needs}

# 🔥 步骤 2：检索后判断相关性
def check_relevance(state: SelfRAGState) -> dict:
    if not state["documents"]:
        return {"relevance_score": 0.0}
    
    doc_text = "\n".join(state["documents"][:3])
    response = llm.invoke(f"""评估检索文档与问题的相关性（0-1分）：

问题：{state['question']}
文档：{doc_text[:1500]}

只返回一个 0-1 之间的数字。""")
    
    try:
        score = float(response.content.strip())
    except ValueError:
        score = 0.5
    
    return {"relevance_score": score}

# 🔥 步骤 3：生成后判断支持度
def check_support(state: SelfRAGState) -> dict:
    doc_text = "\n".join(state["documents"][:3])
    response = llm.invoke(f"""评估生成的答案是否被检索文档支持（0-1分）：

文档：{doc_text[:1500]}
答案：{state['generation']}

只返回一个 0-1 之间的数字。""")
    
    try:
        score = float(response.content.strip())
    except ValueError:
        score = 0.5
    
    return {"support_score": score}

# 🔥 步骤 4：判断有用性
def check_usefulness(state: SelfRAGState) -> dict:
    response = llm.invoke(f"""评估答案对问题的有用性（0-1分）：

问题：{state['question']}
答案：{state['generation']}

只返回一个 0-1 之间的数字。""")
    
    try:
        score = float(response.content.strip())
    except ValueError:
        score = 0.5
    
    return {"usefulness_score": score}

# 路由逻辑
def route_after_relevance(state: SelfRAGState) -> str:
    if state["relevance_score"] < 0.5:
        return "regenerate"  # 文档不相关，不使用检索结果直接生成
    return "generate_with_docs"

def route_after_support(state: SelfRAGState) -> str:
    if state["support_score"] < 0.5:
        return "regenerate"  # 答案不被支持，重新生成
    return "check_useful"
```

---

## 5. 高级 Chunking 策略

### 5.1 语义切分

```python
from langchain_experimental.text_splitter import SemanticChunker
from langchain_openai import OpenAIEmbeddings

# 🔥 语义切分：基于语义相似度而非固定长度
semantic_splitter = SemanticChunker(
    OpenAIEmbeddings(),
    breakpoint_threshold_type="percentile",
    breakpoint_threshold_amount=95  # 语义断点阈值
)

chunks = semantic_splitter.split_text(long_document)
for i, chunk in enumerate(chunks):
    print(f"Chunk {i}: {len(chunk)} chars")
    print(chunk[:100])
    print("---")
```

### 5.2 Agentic Chunking

```python
from openai import OpenAI

client = OpenAI()

def agentic_chunk(document: str) -> list[dict]:
    """使用 LLM 智能切分文档"""
    
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{
            "role": "user",
            "content": f"""请将以下文档切分为语义完整的段落。
每个段落应该：
1. 包含一个完整的主题或概念
2. 长度在 200-1000 字之间
3. 保持上下文完整性

文档：
{document}

返回 JSON 格式：
[
    {{"title": "段落标题", "content": "段落内容", "keywords": ["关键词"]}}
]"""
        }],
        response_format={"type": "json_object"}
    )
    
    import json
    return json.loads(response.choices[0].message.content)
```

### 5.3 父子文档切分

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.storage import InMemoryStore
from langchain.retrievers import ParentDocumentRetriever
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings

# 🔥 父文档切分器（大块，用于返回上下文）
parent_splitter = RecursiveCharacterTextSplitter(
    chunk_size=2000,
    chunk_overlap=200
)

# 🔥 子文档切分器（小块，用于精确检索）
child_splitter = RecursiveCharacterTextSplitter(
    chunk_size=400,
    chunk_overlap=50
)

# 创建父子文档检索器
vectorstore = FAISS.from_texts(["init"], OpenAIEmbeddings())
docstore = InMemoryStore()

retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=docstore,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter
)

# 添加文档
retriever.add_documents(documents)

# 检索时：用小块匹配，返回大块上下文
results = retriever.get_relevant_documents("查询内容")
```

---

## 6. RAG 评估（RAGAS）

### 6.1 RAGAS 框架

```bash
pip install ragas
```

```python
"""
RAGAS（Retrieval Augmented Generation Assessment）
RAG 系统的标准评估框架
"""
from ragas import evaluate
from ragas.metrics import (
    faithfulness,        # 忠实度：答案是否基于检索文档
    answer_relevancy,    # 答案相关性：答案是否回答了问题
    context_precision,   # 上下文精确度：检索文档是否精确
    context_recall,      # 上下文召回率：是否检索到所有相关文档
    context_relevancy,   # 上下文相关性：检索文档是否相关
    answer_correctness   # 答案正确性：答案是否正确
)
from datasets import Dataset

# 准备评估数据
eval_data = {
    "question": [
        "什么是GraphRAG？",
        "RAG和微调有什么区别？"
    ],
    "answer": [
        "GraphRAG是一种结合知识图谱的RAG技术...",
        "RAG通过检索外部知识增强生成，微调则修改模型参数..."
    ],
    "contexts": [
        ["GraphRAG利用知识图谱结构进行检索增强..."],
        ["RAG检索外部文档...", "微调需要训练数据..."]
    ],
    "ground_truth": [
        "GraphRAG是将知识图谱与RAG结合的技术...",
        "RAG实时检索外部知识，微调修改模型权重..."
    ]
}

dataset = Dataset.from_dict(eval_data)

# 🔥 运行评估
results = evaluate(
    dataset,
    metrics=[
        faithfulness,
        answer_relevancy,
        context_precision,
        context_recall,
        answer_correctness
    ]
)

print(results)
# 输出各指标分数
print(f"忠实度: {results['faithfulness']:.3f}")
print(f"答案相关性: {results['answer_relevancy']:.3f}")
print(f"上下文精确度: {results['context_precision']:.3f}")
print(f"上下文召回率: {results['context_recall']:.3f}")
print(f"答案正确性: {results['answer_correctness']:.3f}")
```

### 6.2 自定义评估流水线

```python
from openai import OpenAI
from typing import List, Dict

client = OpenAI()

class RAGEvaluator:
    """自定义 RAG 评估器"""
    
    def evaluate_faithfulness(self, answer: str, 
                              contexts: List[str]) -> float:
        """评估答案忠实度：答案是否基于检索文档"""
        ctx_text = "\n".join(contexts)
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"""评估答案中的每个声明是否都能在参考文档中找到依据。

参考文档：{ctx_text[:2000]}
答案：{answer}

返回 0-1 之间的分数（1=完全忠实，0=完全编造）。
只返回数字。"""
            }]
        )
        try:
            return float(response.choices[0].message.content.strip())
        except ValueError:
            return 0.5
    
    def evaluate_relevancy(self, question: str, answer: str) -> float:
        """评估答案相关性"""
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"""评估答案是否回答了问题。

问题：{question}
答案：{answer}

返回 0-1 之间的分数。只返回数字。"""
            }]
        )
        try:
            return float(response.choices[0].message.content.strip())
        except ValueError:
            return 0.5
    
    def evaluate_batch(self, test_cases: List[Dict]) -> Dict:
        """批量评估"""
        scores = {"faithfulness": [], "relevancy": []}
        
        for case in test_cases:
            f_score = self.evaluate_faithfulness(
                case["answer"], case["contexts"]
            )
            r_score = self.evaluate_relevancy(
                case["question"], case["answer"]
            )
            scores["faithfulness"].append(f_score)
            scores["relevancy"].append(r_score)
        
        return {
            "avg_faithfulness": sum(scores["faithfulness"]) / len(scores["faithfulness"]),
            "avg_relevancy": sum(scores["relevancy"]) / len(scores["relevancy"])
        }
```

---

## 7. 向量数据库选型

### 7.1 主流向量数据库对比

| 数据库 | 类型 | 适用场景 | 特点 |
|--------|------|----------|------|
| **FAISS** | 库 | 开发/小规模 | Meta 开源，纯内存，速度极快 |
| **Chroma** | 嵌入式 | 开发/中小规模 | 简单易用，支持持久化 |
| **Milvus** | 分布式 | 生产/大规模 | 云原生，支持十亿级向量 |
| **Qdrant** | 独立服务 | 生产/中大规模 | Rust 编写，性能优秀 |
| **Weaviate** | 独立服务 | 生产/中大规模 | 支持混合搜索，GraphQL API |
| **pgvector** | PG 扩展 | 已有 PG 的项目 | 无需额外基础设施 |
| **Pinecone** | 云服务 | 生产/免运维 | 全托管，按需付费 |

### 7.2 选型建议

```python
"""
向量数据库选型决策树：

1. 是否需要分布式？
   ├── 是 → Milvus / Weaviate
   └── 否 → 继续

2. 数据规模？
   ├── < 100万向量 → Chroma / Qdrant
   ├── 100万-10亿 → Qdrant / Milvus
   └── > 10亿 → Milvus

3. 是否已有 PostgreSQL？
   ├── 是 → pgvector（简单场景）
   └── 否 → 继续

4. 是否需要混合搜索（向量+关键词）？
   ├── 是 → Weaviate / Qdrant
   └── 否 → FAISS / Chroma

5. 是否需要免运维？
   ├── 是 → Pinecone / Zilliz Cloud
   └── 否 → 自建 Qdrant / Milvus
"""
```

---

## 8. 多模态 RAG

### 8.1 图文混合 RAG

```python
"""
多模态 RAG：支持文本和图片的检索与生成
"""
from llama_index.core import SimpleDirectoryReader, VectorStoreIndex
from llama_index.multi_modal_llms.openai import OpenAIMultiModal
from llama_index.core.schema import ImageDocument

# 加载多模态文档
documents = SimpleDirectoryReader(
    "./data",
    required_exts=[".txt", ".md", ".png", ".jpg"]
).load_data()

# 创建多模态索引
index = VectorStoreIndex.from_documents(documents)

# 使用多模态 LLM 查询
mm_llm = OpenAIMultiModal(model="gpt-4o")

query_engine = index.as_query_engine(
    multi_modal_llm=mm_llm
)

response = query_engine.query("描述图片中的架构图")
print(response)
```

---

## 9. 生产级 RAG 架构

### 9.1 完整架构设计

```python
"""
生产级 RAG 系统架构

数据层
├── 文档处理管道（ETL）
│   ├── 文档加载（PDF/Word/HTML/API）
│   ├── 智能切分（语义/父子文档）
│   ├── 元数据提取
│   └── 增量更新
├── 存储层
│   ├── 向量数据库（Qdrant/Milvus）
│   ├── 知识图谱（Neo4j）
│   └── 全文索引（Elasticsearch）

检索层
├── 混合检索（向量 + BM25 + 图谱）
├── 重排序（Cohere Rerank / Cross-Encoder）
├── 查询重写和扩展
└── 自适应检索策略

生成层
├── Prompt 模板管理
├── 上下文压缩
├── 引用追踪
└── 答案后处理

评估层
├── RAGAS 自动评估
├── 人工评估
├── A/B 测试
└── 监控告警（Langfuse）
"""
```

---

## 10. 总结

### 核心要点
1. GraphRAG 通过知识图谱增强多跳推理能力
2. Agentic RAG 让 Agent 自主决定检索策略，支持多轮迭代
3. Self-RAG 通过自我反思机制提升答案质量
4. 高级 Chunking（语义切分、父子文档）显著影响检索质量
5. RAGAS 提供标准化的 RAG 评估框架

### 学习路径
1. 掌握基础 RAG（参考 RAG应用开发-完整教程）
2. 学习高级 Chunking 策略
3. 实现 Agentic RAG（LangGraph）
4. 探索 GraphRAG
5. 建立 RAG 评估体系
6. 设计生产级 RAG 架构

### 进阶方向
- Speculative RAG（推测性 RAG）
- RAPTOR（递归抽象处理的树状检索）
- 多模态 RAG 深度应用
- RAG + Agent 融合架构

---

## 🔗 相关资源

- [Microsoft GraphRAG](https://github.com/microsoft/graphrag)
- [RAGAS 评估框架](https://docs.ragas.io/)
- [LangGraph RAG 教程](https://langchain-ai.github.io/langgraph/tutorials/rag/)
- [LlamaIndex PropertyGraph](https://docs.llamaindex.ai/en/stable/examples/property_graph/)

---

**@author erik.zhou**
