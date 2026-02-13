# LangChain Hub 完整教程

> @author erik.zhou
> 
> 更新日期：2026-02-13

## 📋 课程概述

LangChain Hub 是一个用于发现、分享和管理 LangChain 提示词（Prompts）、Chain 和 Agent 的平台。它类似于 GitHub，但专注于 LangChain 组件的共享和复用。

### 学习目标
- 掌握 LangChain Hub 的使用
- 学会查找和使用社区提示词
- 理解如何发布自己的提示词
- 掌握提示词版本管理

### 前置知识
- Python 基础
- LangChain 基础
- Prompt Engineering 基础

---

## 1. LangChain Hub 基础

### 1.1 安装和配置

```python
# 安装 langchainhub
pip install langchainhub

# 或者使用 langchain 自带的 hub 功能
pip install langchain

# 设置 API Key（可选，用于发布提示词）
export LANGCHAIN_API_KEY="your-api-key"
```

### 1.2 基本使用

```python
from langchain import hub

# 拉取一个提示词模板
prompt = hub.pull("rlm/rag-prompt")

# 查看提示词内容
print(prompt)

# 使用提示词
from langchain.chat_models import ChatOpenAI

model = ChatOpenAI()
chain = prompt | model

# 调用
response = chain.invoke({
    "context": "LangChain is a framework for developing applications powered by language models.",
    "question": "What is LangChain?"
})
print(response)
```

### 1.3 浏览 Hub

访问 [LangChain Hub](https://smith.langchain.com/hub) 浏览可用的提示词：

- 按类别筛选（RAG、Agent、Chat 等）
- 按热度排序
- 查看使用示例
- 查看版本历史

---

## 2. 使用社区提示词

### 2.1 RAG 提示词

```python
from langchain import hub
from langchain.chat_models import ChatOpenAI
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS
from langchain.schema.output_parser import StrOutputParser
from langchain.schema.runnable import RunnablePassthrough

# 拉取 RAG 提示词
rag_prompt = hub.pull("rlm/rag-prompt")

# 创建向量存储
embeddings = OpenAIEmbeddings()
texts = [
    "LangChain is a framework for developing applications powered by language models.",
    "It provides tools for prompt management, chains, and agents.",
]
vectorstore = FAISS.from_texts(texts, embeddings)
retriever = vectorstore.as_retriever()

# 创建 RAG Chain
model = ChatOpenAI()
rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | rag_prompt
    | model
    | StrOutputParser()
)

# 使用
response = rag_chain.invoke("What is LangChain?")
print(response)
```

### 2.2 Agent 提示词

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_react_agent
from langchain.chat_models import ChatOpenAI
from langchain.tools import Tool

# 拉取 ReAct 提示词
react_prompt = hub.pull("hwchase17/react")

# 定义工具
def search_tool(query: str) -> str:
    """搜索工具"""
    return f"Search results for: {query}"

def calculator_tool(expression: str) -> str:
    """计算器工具"""
    try:
        return str(eval(expression))
    except:
        return "Invalid expression"

tools = [
    Tool(
        name="Search",
        func=search_tool,
        description="useful for searching information"
    ),
    Tool(
        name="Calculator",
        func=calculator_tool,
        description="useful for mathematical calculations"
    )
]

# 创建 Agent
model = ChatOpenAI()
agent = create_react_agent(model, tools, react_prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

# 使用
response = agent_executor.invoke({
    "input": "What is 25 * 4?"
})
print(response)
```

### 2.3 对话提示词

```python
from langchain import hub
from langchain.chat_models import ChatOpenAI
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain

# 拉取对话提示词
conversation_prompt = hub.pull("langchain-ai/conversation-prompt")

# 创建对话链
model = ChatOpenAI()
memory = ConversationBufferMemory()
conversation = ConversationChain(
    llm=model,
    memory=memory,
    prompt=conversation_prompt,
    verbose=True
)

# 使用
response1 = conversation.predict(input="Hi, my name is Alice")
print(response1)

response2 = conversation.predict(input="What's my name?")
print(response2)
```

---

## 3. 发布提示词

### 3.1 创建提示词

```python
from langchain.prompts import ChatPromptTemplate

# 创建一个自定义提示词
custom_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant that translates {input_language} to {output_language}."),
    ("human", "{text}")
])

# 测试提示词
from langchain.chat_models import ChatOpenAI

model = ChatOpenAI()
chain = custom_prompt | model

response = chain.invoke({
    "input_language": "English",
    "output_language": "Chinese",
    "text": "Hello, how are you?"
})
print(response)
```

### 3.2 推送到 Hub

```python
from langchain import hub

# 推送提示词到 Hub
hub.push(
    "your-username/translation-prompt",
    custom_prompt,
    api_key="your-api-key"
)

# 添加描述和标签
hub.push(
    "your-username/translation-prompt",
    custom_prompt,
    api_key="your-api-key",
    description="A prompt for translating text between languages",
    tags=["translation", "multilingual"]
)
```

### 3.3 版本管理

```python
from langchain import hub

# 推送新版本
hub.push(
    "your-username/translation-prompt:v2",
    updated_prompt,
    api_key="your-api-key"
)

# 拉取特定版本
prompt_v1 = hub.pull("your-username/translation-prompt:v1")
prompt_v2 = hub.pull("your-username/translation-prompt:v2")

# 拉取最新版本（默认）
prompt_latest = hub.pull("your-username/translation-prompt")
```

---

## 4. 高级用法

### 4.1 提示词组合

```python
from langchain import hub
from langchain.prompts import ChatPromptTemplate

# 拉取多个提示词
base_prompt = hub.pull("langchain-ai/base-prompt")
rag_prompt = hub.pull("rlm/rag-prompt")

# 组合提示词
combined_prompt = ChatPromptTemplate.from_messages([
    base_prompt.messages[0],  # 系统消息
    ("human", "Context: {context}"),
    ("human", "Question: {question}")
])

# 使用组合后的提示词
from langchain.chat_models import ChatOpenAI

model = ChatOpenAI()
chain = combined_prompt | model

response = chain.invoke({
    "context": "LangChain is a framework...",
    "question": "What is LangChain?"
})
```

### 4.2 提示词模板化

```python
from langchain import hub
from langchain.prompts import ChatPromptTemplate

# 创建可配置的提示词模板
def create_custom_prompt(style: str = "formal"):
    """创建自定义风格的提示词"""
    if style == "formal":
        system_message = "You are a professional assistant."
    elif style == "casual":
        system_message = "You are a friendly assistant."
    else:
        system_message = "You are a helpful assistant."
    
    return ChatPromptTemplate.from_messages([
        ("system", system_message),
        ("human", "{input}")
    ])

# 使用不同风格
formal_prompt = create_custom_prompt("formal")
casual_prompt = create_custom_prompt("casual")

# 推送到 Hub
hub.push("your-username/formal-prompt", formal_prompt)
hub.push("your-username/casual-prompt", casual_prompt)
```

### 4.3 提示词测试

```python
from langchain import hub
from langchain.chat_models import ChatOpenAI
from langchain.evaluation import load_evaluator

# 拉取提示词
prompt = hub.pull("your-username/custom-prompt")

# 创建测试用例
test_cases = [
    {"input": "Hello", "expected": "greeting"},
    {"input": "What's the weather?", "expected": "weather_query"},
    {"input": "Tell me a joke", "expected": "entertainment"},
]

# 测试提示词
model = ChatOpenAI()
chain = prompt | model

results = []
for test_case in test_cases:
    response = chain.invoke(test_case["input"])
    results.append({
        "input": test_case["input"],
        "output": response.content,
        "expected": test_case["expected"]
    })

# 评估结果
for result in results:
    print(f"Input: {result['input']}")
    print(f"Output: {result['output']}")
    print(f"Expected: {result['expected']}")
    print("---")
```

---

## 5. 实战案例

### 5.1 构建提示词库

```python
from langchain import hub
from langchain.prompts import ChatPromptTemplate
from typing import Dict, List

class PromptLibrary:
    """提示词库管理类"""
    
    def __init__(self, username: str, api_key: str):
        self.username = username
        self.api_key = api_key
        self.prompts: Dict[str, ChatPromptTemplate] = {}
    
    def add_prompt(self, name: str, prompt: ChatPromptTemplate):
        """添加提示词到库"""
        self.prompts[name] = prompt
    
    def push_all(self):
        """推送所有提示词到 Hub"""
        for name, prompt in self.prompts.items():
            hub.push(
                f"{self.username}/{name}",
                prompt,
                api_key=self.api_key
            )
            print(f"Pushed: {name}")
    
    def pull_all(self, prompt_names: List[str]):
        """从 Hub 拉取提示词"""
        for name in prompt_names:
            prompt = hub.pull(f"{self.username}/{name}")
            self.prompts[name] = prompt
            print(f"Pulled: {name}")
    
    def get_prompt(self, name: str) -> ChatPromptTemplate:
        """获取提示词"""
        return self.prompts.get(name)

# 使用示例
library = PromptLibrary("your-username", "your-api-key")

# 添加提示词
translation_prompt = ChatPromptTemplate.from_template(
    "Translate {text} to {language}"
)
library.add_prompt("translation", translation_prompt)

summarization_prompt = ChatPromptTemplate.from_template(
    "Summarize the following text: {text}"
)
library.add_prompt("summarization", summarization_prompt)

# 推送到 Hub
library.push_all()

# 从 Hub 拉取
library.pull_all(["translation", "summarization"])
```

### 5.2 提示词 A/B 测试

```python
from langchain import hub
from langchain.chat_models import ChatOpenAI
from typing import List, Dict
import random

class PromptABTest:
    """提示词 A/B 测试类"""
    
    def __init__(self, prompt_a: str, prompt_b: str):
        self.prompt_a = hub.pull(prompt_a)
        self.prompt_b = hub.pull(prompt_b)
        self.model = ChatOpenAI()
        self.results = {"a": [], "b": []}
    
    def run_test(self, inputs: List[Dict], sample_size: int = 100):
        """运行 A/B 测试"""
        for i in range(sample_size):
            # 随机选择提示词
            variant = random.choice(["a", "b"])
            prompt = self.prompt_a if variant == "a" else self.prompt_b
            
            # 随机选择输入
            input_data = random.choice(inputs)
            
            # 执行
            chain = prompt | self.model
            response = chain.invoke(input_data)
            
            # 记录结果
            self.results[variant].append({
                "input": input_data,
                "output": response.content
            })
    
    def analyze_results(self):
        """分析测试结果"""
        print(f"Variant A: {len(self.results['a'])} samples")
        print(f"Variant B: {len(self.results['b'])} samples")
        
        # 这里可以添加更复杂的分析逻辑
        # 例如：响应长度、用户满意度等

# 使用示例
ab_test = PromptABTest(
    "your-username/prompt-v1",
    "your-username/prompt-v2"
)

test_inputs = [
    {"question": "What is AI?"},
    {"question": "How does machine learning work?"},
    {"question": "Explain neural networks"},
]

ab_test.run_test(test_inputs, sample_size=50)
ab_test.analyze_results()
```

### 5.3 提示词优化工作流

```python
from langchain import hub
from langchain.chat_models import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from typing import List, Dict

class PromptOptimizer:
    """提示词优化器"""
    
    def __init__(self, base_prompt: ChatPromptTemplate):
        self.base_prompt = base_prompt
        self.model = ChatOpenAI()
        self.versions = [base_prompt]
    
    def generate_variations(self, num_variations: int = 3) -> List[ChatPromptTemplate]:
        """生成提示词变体"""
        variations = []
        
        # 使用 LLM 生成变体
        meta_prompt = ChatPromptTemplate.from_template(
            "Generate {num} variations of the following prompt:\n{prompt}\n\nVariations:"
        )
        
        chain = meta_prompt | self.model
        response = chain.invoke({
            "num": num_variations,
            "prompt": str(self.base_prompt)
        })
        
        # 解析响应并创建新的提示词
        # 这里简化处理，实际应该解析 LLM 的输出
        for i in range(num_variations):
            variation = ChatPromptTemplate.from_template(
                f"Variation {i+1}: " + str(self.base_prompt)
            )
            variations.append(variation)
        
        return variations
    
    def evaluate_prompt(self, prompt: ChatPromptTemplate, test_cases: List[Dict]) -> float:
        """评估提示词性能"""
        chain = prompt | self.model
        scores = []
        
        for test_case in test_cases:
            response = chain.invoke(test_case["input"])
            # 这里应该有实际的评估逻辑
            score = len(response.content) / 100  # 简化的评分
            scores.append(score)
        
        return sum(scores) / len(scores)
    
    def optimize(self, test_cases: List[Dict], iterations: int = 3):
        """优化提示词"""
        best_prompt = self.base_prompt
        best_score = self.evaluate_prompt(best_prompt, test_cases)
        
        for i in range(iterations):
            print(f"Iteration {i+1}")
            
            # 生成变体
            variations = self.generate_variations()
            
            # 评估每个变体
            for variation in variations:
                score = self.evaluate_prompt(variation, test_cases)
                print(f"Score: {score}")
                
                if score > best_score:
                    best_score = score
                    best_prompt = variation
                    self.versions.append(best_prompt)
        
        return best_prompt, best_score

# 使用示例
base_prompt = ChatPromptTemplate.from_template(
    "Answer the question: {question}"
)

optimizer = PromptOptimizer(base_prompt)

test_cases = [
    {"input": {"question": "What is AI?"}},
    {"input": {"question": "How does ML work?"}},
]

best_prompt, score = optimizer.optimize(test_cases, iterations=3)
print(f"Best prompt score: {score}")

# 推送最佳提示词到 Hub
hub.push("your-username/optimized-prompt", best_prompt)
```

---

## 6. 最佳实践

### 6.1 提示词命名规范

```python
# 好的命名
"username/rag-qa-prompt"
"username/translation-en-zh"
"username/code-review-agent"

# 不好的命名
"username/prompt1"
"username/test"
"username/my-prompt"
```

### 6.2 提示词文档化

```python
from langchain.prompts import ChatPromptTemplate

# 创建带有详细文档的提示词
prompt = ChatPromptTemplate.from_messages([
    ("system", """You are a translation assistant.
    
    Guidelines:
    - Maintain the original tone and style
    - Preserve formatting and structure
    - Handle technical terms appropriately
    """),
    ("human", "Translate from {source_lang} to {target_lang}: {text}")
])

# 推送时添加详细描述
hub.push(
    "username/translation-prompt",
    prompt,
    description="""
    A professional translation prompt that:
    - Maintains tone and style
    - Preserves formatting
    - Handles technical terms
    
    Usage:
    ```python
    prompt.invoke({
        "source_lang": "English",
        "target_lang": "Chinese",
        "text": "Hello world"
    })
    ```
    """,
    tags=["translation", "multilingual", "professional"]
)
```

### 6.3 版本控制策略

```python
# 语义化版本
hub.push("username/prompt:v1.0.0", prompt_v1)  # 初始版本
hub.push("username/prompt:v1.1.0", prompt_v1_1)  # 新增功能
hub.push("username/prompt:v2.0.0", prompt_v2)  # 重大更新

# 使用标签
hub.push("username/prompt:latest", prompt_latest)
hub.push("username/prompt:stable", prompt_stable)
hub.push("username/prompt:beta", prompt_beta)
```

---

## 7. 总结

### 核心要点
1. LangChain Hub 是提示词共享和管理平台
2. 支持版本控制和协作
3. 提供丰富的社区提示词
4. 适合团队协作和知识共享

### 学习建议
1. 先浏览社区提示词，学习优秀案例
2. 实践使用不同类型的提示词
3. 学习提示词优化技巧
4. 分享自己的提示词到社区

### 下一步
- 探索更多社区提示词
- 学习 Prompt Engineering 高级技巧
- 实践提示词优化和测试
- 参与社区贡献

---

**@author erik.zhou**
