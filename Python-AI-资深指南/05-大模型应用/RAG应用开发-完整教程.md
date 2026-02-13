# RAG 应用开发完整教程

> @author erik.zhou
> 
> 更新日期：2026-02-13

## 📋 课程概述

RAG（Retrieval-Augmented Generation，检索增强生成）是一种结合信息检索和文本生成的技术，通过从外部知识库检索相关信息来增强大语言模型的回答能力。本教程将全面介绍 RAG 应用的开发。

### 学习目标
- 理解 RAG 的核心原理和架构
- 掌握文档处理和向量化技术
- 学会构建高效的检索系统
- 掌握 RAG 应用的优化技巧

### 前置知识
- Python 基础
- LangChain 基础
- 向量数据库基础
- Embedding 概念

---

## 1. RAG 基础原理

### 1.1 什么是 RAG

RAG 的核心思想是：
1. 将用户问题转换为向量
2. 从知识库中检索相关文档
3. 将检索到的文档作为上下文
4. 让 LLM 基于上下文生成回答

```
用户问题 → 向量化 → 检索相关文档 → 构建提示词 → LLM 生成回答
```

### 1.2 RAG vs 微调

| 特性 | RAG | 微调 |
|------|-----|------|
| 知识更新 | 实时更新 | 需要重新训练 |
| 成本 | 低 | 高 |
| 可解释性 | 高（可追溯来源） | 低 |
| 准确性 | 依赖检索质量 | 依赖训练数据 |
| 适用场景 | 知识密集型任务 | 特定领域任务 |

### 1.3 RAG 架构

```python
from langchain.document_loaders import TextLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnablePassthrough

# 1. 加载文档
loader = TextLoader("knowledge_base.txt")
documents = loader.load()

# 2. 文档切分
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = text_splitter.split_documents(documents)

# 3. 向量化和存储
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(splits, embeddings)

# 4. 创建检索器
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

# 5. 创建提示词模板
template = """Answer the question based only on the following context:
{context}

Question: {question}

Answer:"""
prompt = ChatPromptTemplate.from_template(template)

# 6. 创建 RAG Chain
model = ChatOpenAI()
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 7. 使用
response = rag_chain.invoke("What is LangChain?")
print(response)
```

---

## 2. 文档处理

### 2.1 文档加载

```python
from langchain.document_loaders import (
    TextLoader,
    PDFLoader,
    UnstructuredMarkdownLoader,
    CSVLoader,
    JSONLoader,
    WebBaseLoader
)

# 加载文本文件
text_loader = TextLoader("document.txt")
text_docs = text_loader.load()

# 加载 PDF
pdf_loader = PDFLoader("document.pdf")
pdf_docs = pdf_loader.load()

# 加载 Markdown
md_loader = UnstructuredMarkdownLoader("document.md")
md_docs = md_loader.load()

# 加载 CSV
csv_loader = CSVLoader("data.csv")
csv_docs = csv_loader.load()

# 加载网页
web_loader = WebBaseLoader("https://example.com")
web_docs = web_loader.load()

# 批量加载目录
from langchain.document_loaders import DirectoryLoader

loader = DirectoryLoader(
    "./documents",
    glob="**/*.txt",
    loader_cls=TextLoader
)
docs = loader.load()
```

### 2.2 文档切分策略

```python
from langchain.text_splitter import (
    RecursiveCharacterTextSplitter,
    CharacterTextSplitter,
    TokenTextSplitter,
    MarkdownHeaderTextSplitter
)

# 递归字符切分（推荐）
recursive_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    length_function=len,
    separators=["\n\n", "\n", " ", ""]
)
splits = recursive_splitter.split_documents(documents)

# 按字符切分
char_splitter = CharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200,
    separator="\n"
)

# 按 Token 切分
token_splitter = TokenTextSplitter(
    chunk_size=500,
    chunk_overlap=50
)

# Markdown 按标题切分
md_splitter = MarkdownHeaderTextSplitter(
    headers_to_split_on=[
        ("#", "Header 1"),
        ("##", "Header 2"),
        ("###", "Header 3"),
    ]
)
```

### 2.3 文档元数据

```python
from langchain.schema import Document

# 创建带元数据的文档
doc = Document(
    page_content="LangChain is a framework...",
    metadata={
        "source": "documentation.md",
        "page": 1,
        "category": "tutorial",
        "date": "2024-01-01"
    }
)

# 在切分时保留元数据
splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
splits = splitter.split_documents([doc])

# 每个切片都会保留原始元数据
for split in splits:
    print(split.metadata)
```

---

## 3. 向量化和存储

### 3.1 Embedding 模型选择

```python
from langchain.embeddings import (
    OpenAIEmbeddings,
    HuggingFaceEmbeddings,
    CohereEmbeddings
)

# OpenAI Embeddings（推荐）
openai_embeddings = OpenAIEmbeddings(
    model="text-embedding-3-small"
)

# Hugging Face Embeddings（开源）
hf_embeddings = HuggingFaceEmbeddings(
    model_name="sentence-transformers/all-MiniLM-L6-v2"
)

# Cohere Embeddings
cohere_embeddings = CohereEmbeddings(
    model="embed-english-v3.0"
)

# 测试 Embedding
text = "LangChain is a framework"
vector = openai_embeddings.embed_query(text)
print(f"Vector dimension: {len(vector)}")
```

### 3.2 向量数据库

```python
from langchain.vectorstores import (
    FAISS,
    Chroma,
    Pinecone,
    Weaviate,
    Qdrant
)

# FAISS（本地，适合开发）
faiss_store = FAISS.from_documents(
    documents=splits,
    embedding=embeddings
)

# 保存和加载
faiss_store.save_local("faiss_index")
loaded_store = FAISS.load_local("faiss_index", embeddings)

# Chroma（本地，持久化）
chroma_store = Chroma.from_documents(
    documents=splits,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# Pinecone（云端，生产环境）
import pinecone

pinecone.init(
    api_key="your-api-key",
    environment="your-environment"
)

pinecone_store = Pinecone.from_documents(
    documents=splits,
    embedding=embeddings,
    index_name="langchain-index"
)

# Qdrant（开源，可自托管）
from qdrant_client import QdrantClient

client = QdrantClient(host="localhost", port=6333)
qdrant_store = Qdrant.from_documents(
    documents=splits,
    embedding=embeddings,
    collection_name="langchain",
    client=client
)
```

### 3.3 增量更新

```python
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings

# 创建初始向量库
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(initial_docs, embeddings)

# 添加新文档
new_docs = [
    Document(page_content="New content 1"),
    Document(page_content="New content 2")
]
vectorstore.add_documents(new_docs)

# 删除文档（需要文档 ID）
vectorstore.delete(["doc_id_1", "doc_id_2"])

# 更新文档（先删除再添加）
def update_document(vectorstore, doc_id, new_doc):
    vectorstore.delete([doc_id])
    vectorstore.add_documents([new_doc])
```

---

## 4. 检索策略

### 4.1 基础检索

```python
from langchain.vectorstores import FAISS

# 创建检索器
retriever = vectorstore.as_retriever()

# 相似度检索（默认）
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 3}
)

# MMR 检索（最大边际相关性）
retriever = vectorstore.as_retriever(
    search_type="mmr",
    search_kwargs={
        "k": 3,
        "fetch_k": 10,
        "lambda_mult": 0.5
    }
)

# 相似度阈值检索
retriever = vectorstore.as_retriever(
    search_type="similarity_score_threshold",
    search_kwargs={
        "score_threshold": 0.8,
        "k": 3
    }
)

# 使用检索器
docs = retriever.get_relevant_documents("What is LangChain?")
for doc in docs:
    print(doc.page_content)
    print(doc.metadata)
    print("---")
```

### 4.2 混合检索

```python
from langchain.retrievers import (
    EnsembleRetriever,
    BM25Retriever
)

# 向量检索器
vector_retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

# BM25 检索器（关键词检索）
bm25_retriever = BM25Retriever.from_documents(documents)
bm25_retriever.k = 3

# 混合检索器
ensemble_retriever = EnsembleRetriever(
    retrievers=[vector_retriever, bm25_retriever],
    weights=[0.5, 0.5]
)

# 使用混合检索
docs = ensemble_retriever.get_relevant_documents("What is LangChain?")
```

### 4.3 上下文压缩

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor
from langchain.chat_models import ChatOpenAI

# 创建基础检索器
base_retriever = vectorstore.as_retriever(search_kwargs={"k": 5})

# 创建压缩器
llm = ChatOpenAI()
compressor = LLMChainExtractor.from_llm(llm)

# 创建压缩检索器
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=base_retriever
)

# 使用压缩检索
compressed_docs = compression_retriever.get_relevant_documents(
    "What is LangChain?"
)
```

### 4.4 多查询检索

```python
from langchain.retrievers.multi_query import MultiQueryRetriever
from langchain.chat_models import ChatOpenAI

# 创建多查询检索器
llm = ChatOpenAI()
multi_query_retriever = MultiQueryRetriever.from_llm(
    retriever=vectorstore.as_retriever(),
    llm=llm
)

# 自动生成多个查询变体并检索
docs = multi_query_retriever.get_relevant_documents(
    "What is LangChain?"
)
```

---

## 5. RAG Chain 构建

### 5.1 基础 RAG Chain

```python
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnablePassthrough

# 创建提示词模板
template = """Answer the question based on the following context:

Context: {context}

Question: {question}

Answer:"""

prompt = ChatPromptTemplate.from_template(template)

# 创建 RAG Chain
model = ChatOpenAI()
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 使用
response = rag_chain.invoke("What is LangChain?")
```

### 5.2 带引用的 RAG

```python
from langchain.schema.runnable import RunnableParallel

# 创建带引用的 RAG Chain
rag_chain_with_source = RunnableParallel(
    {
        "context": retriever,
        "question": RunnablePassthrough()
    }
).assign(
    answer=lambda x: (
        prompt
        | model
        | StrOutputParser()
    ).invoke({
        "context": x["context"],
        "question": x["question"]
    })
)

# 使用
result = rag_chain_with_source.invoke("What is LangChain?")
print("Answer:", result["answer"])
print("\nSources:")
for doc in result["context"]:
    print(f"- {doc.metadata.get('source', 'Unknown')}")
```

### 5.3 对话式 RAG

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationalRetrievalChain

# 创建对话式 RAG
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

conversational_rag = ConversationalRetrievalChain.from_llm(
    llm=model,
    retriever=retriever,
    memory=memory,
    verbose=True
)

# 多轮对话
response1 = conversational_rag({"question": "What is LangChain?"})
print(response1["answer"])

response2 = conversational_rag({"question": "What are its main features?"})
print(response2["answer"])
```

---

## 6. RAG 优化

### 6.1 查询优化

```python
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI

# 查询重写
query_rewrite_template = """Given the following question, rewrite it to be more specific and clear:

Original question: {question}

Rewritten question:"""

query_rewriter = (
    ChatPromptTemplate.from_template(query_rewrite_template)
    | ChatOpenAI()
    | StrOutputParser()
)

# 使用查询重写
original_query = "Tell me about it"
rewritten_query = query_rewriter.invoke({"question": original_query})
print(f"Rewritten: {rewritten_query}")

# 在 RAG 中使用
rag_chain_with_rewrite = (
    {"question": query_rewriter}
    | {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)
```

### 6.2 重排序

```python
from langchain.retrievers.document_compressors import CohereRerank
from langchain.retrievers import ContextualCompressionRetriever

# 创建重排序器
reranker = CohereRerank(
    model="rerank-english-v2.0",
    top_n=3
)

# 创建带重排序的检索器
reranking_retriever = ContextualCompressionRetriever(
    base_compressor=reranker,
    base_retriever=vectorstore.as_retriever(search_kwargs={"k": 10})
)

# 使用
docs = reranking_retriever.get_relevant_documents("What is LangChain?")
```

### 6.3 答案后处理

```python
from langchain.schema.runnable import RunnableLambda

def post_process_answer(answer: str) -> str:
    """后处理答案"""
    # 移除多余空格
    answer = " ".join(answer.split())
    
    # 添加引用格式
    if "[" not in answer:
        answer += "\n\n[来源：知识库]"
    
    return answer

# 在 RAG Chain 中添加后处理
rag_chain_with_postprocess = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
    | RunnableLambda(post_process_answer)
)
```

---

## 7. 评估和监控

### 7.1 RAG 评估指标

```python
from langchain.evaluation import load_evaluator

# 准备测试数据
test_cases = [
    {
        "question": "What is LangChain?",
        "ground_truth": "LangChain is a framework for developing applications powered by language models."
    },
    # 更多测试用例...
]

# 评估答案质量
qa_evaluator = load_evaluator("qa")

for test_case in test_cases:
    prediction = rag_chain.invoke(test_case["question"])
    
    result = qa_evaluator.evaluate_strings(
        prediction=prediction,
        reference=test_case["ground_truth"],
        input=test_case["question"]
    )
    
    print(f"Question: {test_case['question']}")
    print(f"Score: {result['score']}")
    print("---")
```

### 7.2 检索质量评估

```python
def evaluate_retrieval(retriever, test_cases):
    """评估检索质量"""
    results = []
    
    for test_case in test_cases:
        # 检索文档
        docs = retriever.get_relevant_documents(test_case["question"])
        
        # 计算相关文档数量
        relevant_docs = [
            doc for doc in docs
            if any(keyword in doc.page_content.lower()
                   for keyword in test_case["keywords"])
        ]
        
        # 计算精确率
        precision = len(relevant_docs) / len(docs) if docs else 0
        
        results.append({
            "question": test_case["question"],
            "precision": precision,
            "retrieved_docs": len(docs),
            "relevant_docs": len(relevant_docs)
        })
    
    return results

# 使用
test_cases = [
    {
        "question": "What is LangChain?",
        "keywords": ["langchain", "framework", "language model"]
    },
]

results = evaluate_retrieval(retriever, test_cases)
for result in results:
    print(f"Question: {result['question']}")
    print(f"Precision: {result['precision']:.2f}")
```

---

## 8. 生产环境部署

### 8.1 完整的 RAG 应用

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnablePassthrough
import logging

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# 创建 FastAPI 应用
app = FastAPI(title="RAG QA System")

# 请求模型
class QuestionRequest(BaseModel):
    question: str
    top_k: int = 3

# 响应模型
class AnswerResponse(BaseModel):
    answer: str
    sources: list[dict]

# 初始化 RAG 组件
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.load_local("./vectorstore", embeddings)
retriever = vectorstore.as_retriever()

model = ChatOpenAI()
prompt = ChatPromptTemplate.from_template(
    "Answer based on context:\n{context}\n\nQuestion: {question}"
)

rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

@app.post("/qa", response_model=AnswerResponse)
async def answer_question(request: QuestionRequest):
    """回答问题"""
    try:
        # 检索文档
        docs = retriever.get_relevant_documents(request.question)
        
        # 生成答案
        answer = rag_chain.invoke(request.question)
        
        # 构建响应
        sources = [
            {
                "content": doc.page_content[:200],
                "metadata": doc.metadata
            }
            for doc in docs[:request.top_k]
        ]
        
        logger.info(f"Question: {request.question}")
        logger.info(f"Answer generated successfully")
        
        return AnswerResponse(answer=answer, sources=sources)
    
    except Exception as e:
        logger.error(f"Error: {str(e)}")
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health_check():
    return {"status": "healthy"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## 9. 总结

### 核心要点
1. RAG 结合检索和生成，适合知识密集型任务
2. 文档切分和向量化是关键步骤
3. 检索策略直接影响答案质量
4. 需要持续评估和优化

### 学习建议
1. 从简单的 RAG 开始实践
2. 理解不同检索策略的适用场景
3. 学习评估和优化方法
4. 关注生产环境的性能和成本

### 下一步
- 学习高级检索技术（GraphRAG、HyDE）
- 探索多模态 RAG
- 实践大规模 RAG 系统
- 学习 RAG 安全和隐私保护

---

**@author erik.zhou**
