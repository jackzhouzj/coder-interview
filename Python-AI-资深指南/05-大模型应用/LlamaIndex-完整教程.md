# LlamaIndex 完整教程

> @author erik.zhou
> 
> 更新日期：2026-02-13

## 📋 课程概述

LlamaIndex（原名 GPT Index）是一个专注于数据索引和检索的框架，用于构建 LLM 应用。它提供了强大的数据连接器、索引结构和查询引擎。

### 学习目标
- 掌握 LlamaIndex 的核心概念
- 学会构建和查询索引
- 理解不同的索引类型
- 掌握与 LangChain 的集成

---

## 1. 快速开始

### 1.1 安装

```python
# 安装 LlamaIndex
pip install llama-index

# 安装 OpenAI（默认 LLM）
pip install openai

# 设置 API Key
import os
os.environ["OPENAI_API_KEY"] = "your-api-key"
```

### 1.2 第一个索引

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader

# 加载文档
documents = SimpleDirectoryReader("./data").load_data()

# 创建索引
index = VectorStoreIndex.from_documents(documents)

# 查询
query_engine = index.as_query_engine()
response = query_engine.query("What is LlamaIndex?")
print(response)
```

---

## 2. 核心概念

### 2.1 文档和节点

```python
from llama_index.core import Document
from llama_index.core.node_parser import SimpleNodeParser

# 创建文档
doc1 = Document(text="LlamaIndex is a data framework for LLM applications.")
doc2 = Document(text="It provides tools for data ingestion and indexing.")

documents = [doc1, doc2]

# 解析为节点
parser = SimpleNodeParser.from_defaults()
nodes = parser.get_nodes_from_documents(documents)

print(f"Number of nodes: {len(nodes)}")
for node in nodes:
    print(node.text[:100])
```

### 2.2 索引类型

```python
from llama_index.core import (
    VectorStoreIndex,
    ListIndex,
    TreeIndex,
    KeywordTableIndex
)

# 向量索引（最常用）
vector_index = VectorStoreIndex.from_documents(documents)

# 列表索引
list_index = ListIndex.from_documents(documents)

# 树索引
tree_index = TreeIndex.from_documents(documents)

# 关键词表索引
keyword_index = KeywordTableIndex.from_documents(documents)
```

### 2.3 查询引擎

```python
# 创建查询引擎
query_engine = index.as_query_engine(
    similarity_top_k=3,  # 返回前3个最相关的节点
    response_mode="compact"  # 响应模式
)

# 查询
response = query_engine.query("What is LlamaIndex?")
print(response)

# 获取源节点
print("\nSource nodes:")
for node in response.source_nodes:
    print(f"Score: {node.score:.3f}")
    print(f"Text: {node.text[:100]}...")
```

---

## 3. 数据加载

### 3.1 文件加载器

```python
from llama_index.core import SimpleDirectoryReader

# 加载目录
documents = SimpleDirectoryReader("./data").load_data()

# 加载特定文件类型
documents = SimpleDirectoryReader(
    "./data",
    required_exts=[".txt", ".md"]
).load_data()

# 递归加载
documents = SimpleDirectoryReader(
    "./data",
    recursive=True
).load_data()
```

### 3.2 不同数据源

```python
from llama_index.readers.file import (
    PDFReader,
    DocxReader,
    MarkdownReader
)

# PDF 加载器
pdf_reader = PDFReader()
pdf_docs = pdf_reader.load_data("document.pdf")

# Word 加载器
docx_reader = DocxReader()
docx_docs = docx_reader.load_data("document.docx")

# Markdown 加载器
md_reader = MarkdownReader()
md_docs = md_reader.load_data("document.md")
```

### 3.3 网页加载

```python
from llama_index.readers.web import SimpleWebPageReader

# 加载网页
urls = [
    "https://example.com/page1",
    "https://example.com/page2"
]

web_reader = SimpleWebPageReader()
web_docs = web_reader.load_data(urls)
```

---

## 4. 文档处理

### 4.1 文本分割

```python
from llama_index.core.node_parser import (
    SimpleNodeParser,
    SentenceSplitter
)

# 简单分割器
simple_parser = SimpleNodeParser.from_defaults(
    chunk_size=1024,
    chunk_overlap=20
)
nodes = simple_parser.get_nodes_from_documents(documents)

# 句子分割器
sentence_splitter = SentenceSplitter(
    chunk_size=1024,
    chunk_overlap=20
)
nodes = sentence_splitter.get_nodes_from_documents(documents)
```

### 4.2 元数据提取

```python
from llama_index.core import Document

# 创建带元数据的文档
doc = Document(
    text="LlamaIndex is a data framework.",
    metadata={
        "source": "documentation",
        "category": "tutorial",
        "date": "2024-01-01"
    }
)

# 使用元数据过滤
from llama_index.core.vector_stores import MetadataFilters, ExactMatchFilter

filters = MetadataFilters(
    filters=[
        ExactMatchFilter(key="category", value="tutorial")
    ]
)

query_engine = index.as_query_engine(filters=filters)
response = query_engine.query("What is LlamaIndex?")
```

---

## 5. 索引构建

### 5.1 向量索引

```python
from llama_index.core import VectorStoreIndex
from llama_index.embeddings.openai import OpenAIEmbedding

# 使用自定义 Embedding
embed_model = OpenAIEmbedding(model="text-embedding-3-small")

index = VectorStoreIndex.from_documents(
    documents,
    embed_model=embed_model
)

# 保存索引
index.storage_context.persist(persist_dir="./storage")

# 加载索引
from llama_index.core import StorageContext, load_index_from_storage

storage_context = StorageContext.from_defaults(persist_dir="./storage")
loaded_index = load_index_from_storage(storage_context)
```

### 5.2 使用向量数据库

```python
from llama_index.vector_stores.chroma import ChromaVectorStore
from llama_index.core import VectorStoreIndex, StorageContext
import chromadb

# 创建 Chroma 客户端
chroma_client = chromadb.PersistentClient(path="./chroma_db")
chroma_collection = chroma_client.create_collection("my_collection")

# 创建向量存储
vector_store = ChromaVectorStore(chroma_collection=chroma_collection)
storage_context = StorageContext.from_defaults(vector_store=vector_store)

# 创建索引
index = VectorStoreIndex.from_documents(
    documents,
    storage_context=storage_context
)
```

### 5.3 增量更新

```python
# 添加新文档
new_doc = Document(text="New content to add")
index.insert(new_doc)

# 删除文档
index.delete_ref_doc("doc_id")

# 更新文档
index.update_ref_doc("doc_id", new_doc)

# 刷新索引
index.refresh_ref_docs(documents)
```

---

## 6. 查询和检索

### 6.1 基础查询

```python
# 创建查询引擎
query_engine = index.as_query_engine()

# 查询
response = query_engine.query("What is LlamaIndex?")
print(response)

# 流式查询
streaming_response = query_engine.query("What is LlamaIndex?")
streaming_response.print_response_stream()
```

### 6.2 高级查询

```python
from llama_index.core import QueryBundle
from llama_index.core.retrievers import VectorIndexRetriever

# 创建检索器
retriever = VectorIndexRetriever(
    index=index,
    similarity_top_k=5
)

# 检索节点
query_bundle = QueryBundle(query_str="What is LlamaIndex?")
nodes = retriever.retrieve(query_bundle)

for node in nodes:
    print(f"Score: {node.score:.3f}")
    print(f"Text: {node.text[:100]}...")
```

### 6.3 混合检索

```python
from llama_index.core.retrievers import (
    VectorIndexRetriever,
    KeywordTableSimpleRetriever
)
from llama_index.core.query_engine import RetrieverQueryEngine

# 向量检索器
vector_retriever = VectorIndexRetriever(
    index=vector_index,
    similarity_top_k=3
)

# 关键词检索器
keyword_retriever = KeywordTableSimpleRetriever(
    index=keyword_index
)

# 组合检索器
from llama_index.core.retrievers import BaseRetriever

class HybridRetriever(BaseRetriever):
    def __init__(self, vector_retriever, keyword_retriever):
        self.vector_retriever = vector_retriever
        self.keyword_retriever = keyword_retriever
    
    def _retrieve(self, query_bundle):
        vector_nodes = self.vector_retriever.retrieve(query_bundle)
        keyword_nodes = self.keyword_retriever.retrieve(query_bundle)
        
        # 合并结果
        all_nodes = vector_nodes + keyword_nodes
        
        # 去重
        unique_nodes = {}
        for node in all_nodes:
            if node.node_id not in unique_nodes:
                unique_nodes[node.node_id] = node
        
        return list(unique_nodes.values())

hybrid_retriever = HybridRetriever(vector_retriever, keyword_retriever)
query_engine = RetrieverQueryEngine(retriever=hybrid_retriever)
```

---

## 7. 响应合成

### 7.1 响应模式

```python
# Compact 模式（默认）
query_engine = index.as_query_engine(response_mode="compact")

# Tree Summarize 模式
query_engine = index.as_query_engine(response_mode="tree_summarize")

# Simple Summarize 模式
query_engine = index.as_query_engine(response_mode="simple_summarize")

# No Text 模式（仅返回源节点）
query_engine = index.as_query_engine(response_mode="no_text")
```

### 7.2 自定义提示词

```python
from llama_index.core import PromptTemplate

# 自定义 QA 提示词
qa_prompt_tmpl = PromptTemplate(
    "Context information is below.\n"
    "---------------------\n"
    "{context_str}\n"
    "---------------------\n"
    "Given the context information and not prior knowledge, "
    "answer the question: {query_str}\n"
)

query_engine = index.as_query_engine(
    text_qa_template=qa_prompt_tmpl
)
```

---

## 8. 与 LangChain 集成

### 8.1 作为 LangChain 检索器

```python
from llama_index.core import VectorStoreIndex
from langchain.chains import RetrievalQA
from langchain.chat_models import ChatOpenAI

# 创建 LlamaIndex 索引
index = VectorStoreIndex.from_documents(documents)

# 转换为 LangChain 检索器
retriever = index.as_retriever()

# 创建 LangChain QA Chain
llm = ChatOpenAI()
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever
)

# 查询
response = qa_chain.run("What is LlamaIndex?")
print(response)
```

### 8.2 使用 LangChain LLM

```python
from llama_index.llms.langchain import LangChainLLM
from langchain.chat_models import ChatOpenAI

# 创建 LangChain LLM
langchain_llm = ChatOpenAI(model="gpt-3.5-turbo")

# 包装为 LlamaIndex LLM
llm = LangChainLLM(llm=langchain_llm)

# 在 LlamaIndex 中使用
from llama_index.core import Settings

Settings.llm = llm

index = VectorStoreIndex.from_documents(documents)
query_engine = index.as_query_engine()
```

---

## 9. 高级功能

### 9.1 子问题查询

```python
from llama_index.core.query_engine import SubQuestionQueryEngine
from llama_index.core.tools import QueryEngineTool

# 创建多个索引
index1 = VectorStoreIndex.from_documents(docs1)
index2 = VectorStoreIndex.from_documents(docs2)

# 创建查询工具
query_tool1 = QueryEngineTool.from_defaults(
    query_engine=index1.as_query_engine(),
    description="Information about topic 1"
)

query_tool2 = QueryEngineTool.from_defaults(
    query_engine=index2.as_query_engine(),
    description="Information about topic 2"
)

# 创建子问题查询引擎
sub_question_engine = SubQuestionQueryEngine.from_defaults(
    query_engine_tools=[query_tool1, query_tool2]
)

# 查询
response = sub_question_engine.query(
    "Compare topic 1 and topic 2"
)
```

### 9.2 路由查询

```python
from llama_index.core.query_engine import RouterQueryEngine
from llama_index.core.selectors import LLMSingleSelector

# 创建路由器
router_query_engine = RouterQueryEngine(
    selector=LLMSingleSelector.from_defaults(),
    query_engine_tools=[query_tool1, query_tool2]
)

# 查询（自动路由到合适的索引）
response = router_query_engine.query("Question about topic 1")
```

### 9.3 对话式查询

```python
from llama_index.core.memory import ChatMemoryBuffer

# 创建对话引擎
memory = ChatMemoryBuffer.from_defaults(token_limit=3000)

chat_engine = index.as_chat_engine(
    chat_mode="condense_plus_context",
    memory=memory,
    verbose=True
)

# 多轮对话
response1 = chat_engine.chat("What is LlamaIndex?")
print(response1)

response2 = chat_engine.chat("What are its main features?")
print(response2)

# 重置对话
chat_engine.reset()
```

---

## 10. 实战案例

### 10.1 文档问答系统

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.core.node_parser import SimpleNodeParser

# 加载文档
documents = SimpleDirectoryReader("./docs").load_data()

# 解析节点
parser = SimpleNodeParser.from_defaults(
    chunk_size=512,
    chunk_overlap=50
)
nodes = parser.get_nodes_from_documents(documents)

# 创建索引
index = VectorStoreIndex(nodes)

# 创建查询引擎
query_engine = index.as_query_engine(
    similarity_top_k=3,
    response_mode="compact"
)

# 查询
def ask_question(question):
    response = query_engine.query(question)
    print(f"Question: {question}")
    print(f"Answer: {response}")
    print("\nSources:")
    for node in response.source_nodes:
        print(f"- {node.metadata.get('file_name', 'Unknown')}")
    print()

# 使用
ask_question("What is the main topic?")
ask_question("How does it work?")
```

### 10.2 多文档比较

```python
from llama_index.core import VectorStoreIndex
from llama_index.core.query_engine import SubQuestionQueryEngine
from llama_index.core.tools import QueryEngineTool

# 为每个文档创建索引
doc1_index = VectorStoreIndex.from_documents([doc1])
doc2_index = VectorStoreIndex.from_documents([doc2])

# 创建查询工具
tool1 = QueryEngineTool.from_defaults(
    query_engine=doc1_index.as_query_engine(),
    description="Document 1 information"
)

tool2 = QueryEngineTool.from_defaults(
    query_engine=doc2_index.as_query_engine(),
    description="Document 2 information"
)

# 创建比较引擎
compare_engine = SubQuestionQueryEngine.from_defaults(
    query_engine_tools=[tool1, tool2]
)

# 比较查询
response = compare_engine.query(
    "What are the differences between the two documents?"
)
print(response)
```

---

## 11. 总结

### 核心要点
1. LlamaIndex 专注于数据索引和检索
2. 支持多种索引类型和数据源
3. 提供灵活的查询和检索机制
4. 与 LangChain 无缝集成

### 学习建议
1. 从简单的向量索引开始
2. 理解不同索引类型的适用场景
3. 学习查询优化技巧
4. 实践复杂的检索场景

### 下一步
- 探索高级索引结构
- 学习性能优化技巧
- 实践大规模数据索引
- 研究多模态索引

---

**@author erik.zhou**
