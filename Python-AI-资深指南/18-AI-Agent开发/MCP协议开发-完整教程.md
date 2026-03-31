# MCP（Model Context Protocol）协议开发完整教程

> @author erik.zhou
> 
> 更新日期：2026-03-31

## 📋 教程概述

MCP（Model Context Protocol）是 Anthropic 于 2024 年 11 月发布的开源协议，2025 年底捐赠给 Linux 基金会旗下的 Agentic AI Foundation（AAIF）。MCP 被称为"AI 的 USB-C 接口"，它标准化了 AI 模型与外部工具、数据源之间的连接方式，已获得 OpenAI、Google、Microsoft、AWS 等主要厂商支持。

### 版本信息
- **协议版本**：2025-06-18（最新）
- **Python SDK**：mcp 1.x+
- **重要程度**：⭐⭐⭐⭐⭐（必学）
- **难度等级**：⭐⭐⭐⭐（较难）
- **预计学习时间**：20-25 小时

### 学习目标
- 理解 MCP 协议的架构和核心概念
- 掌握 MCP Server 的开发方法
- 学会构建 Tools、Resources、Prompts
- 理解 MCP 传输层（stdio/SSE/Streamable HTTP）
- 能够将 MCP Server 集成到各种 AI 框架中

### 前置知识
- Python 异步编程
- HTTP/JSON-RPC 基础
- AI Agent 开发基础

---

## 1. MCP 核心架构

### 1.1 协议架构

```
MCP 架构概览

Host（宿主应用）
├── MCP Client ──── stdio ────── MCP Server A（本地工具）
├── MCP Client ──── HTTP/SSE ─── MCP Server B（远程API）
└── MCP Client ──── HTTP/SSE ─── MCP Server C（数据库）

核心组件：
- Host：宿主应用（如 Claude Desktop、IDE、自定义应用）
- Client：MCP 客户端，维护与 Server 的 1:1 连接
- Server：MCP 服务端，暴露 Tools/Resources/Prompts
- Transport：传输层（stdio / SSE / Streamable HTTP）
```

### 1.2 三大能力原语

| 原语 | 说明 | 控制方 | 类比 |
|------|------|--------|------|
| **Tools** | 模型可调用的函数 | 模型控制 | POST 端点 |
| **Resources** | 模型可读取的数据 | 应用控制 | GET 端点 |
| **Prompts** | 预定义的提示词模板 | 用户控制 | API 模板 |

---

## 2. 环境搭建

### 2.1 安装

```bash
# 安装 MCP Python SDK
pip install mcp

# 或使用 uv（推荐）
uv add mcp

# 安装 CLI 工具
pip install mcp[cli]
```

### 2.2 项目结构

```
my-mcp-server/
├── src/
│   └── my_server/
│       ├── __init__.py
│       └── server.py
├── pyproject.toml
└── README.md
```

---

## 3. 开发 MCP Server

### 3.1 快速开始：使用 FastMCP

```python
"""
最简 MCP Server 示例
使用 FastMCP 高级 API（推荐）
"""
from mcp.server.fastmcp import FastMCP

# 🔥 创建 MCP Server
mcp = FastMCP("我的工具服务")

# 🔥 定义 Tool（模型可调用的函数）
@mcp.tool()
def add(a: int, b: int) -> int:
    """两数相加"""
    return a + b

@mcp.tool()
def get_weather(city: str) -> str:
    """获取指定城市的天气信息
    
    Args:
        city: 城市名称
    """
    weather_data = {
        "北京": "晴天 25°C",
        "上海": "多云 22°C",
    }
    return weather_data.get(city, f"{city}: 暂无数据")

# 🔥 定义 Resource（模型可读取的数据）
@mcp.resource("config://app")
def get_app_config() -> str:
    """获取应用配置"""
    return "App Version: 1.0.0, Environment: production"

# 🔥 定义动态 Resource
@mcp.resource("users://{user_id}/profile")
def get_user_profile(user_id: str) -> str:
    """获取用户资料"""
    return f"用户 {user_id} 的资料信息"

# 🔥 定义 Prompt（预定义提示词模板）
@mcp.prompt()
def code_review(code: str, language: str = "python") -> str:
    """代码审查提示词模板"""
    return f"""请审查以下 {language} 代码：

```{language}
{code}
```

请从以下方面进行审查：
1. 代码质量和可读性
2. 潜在的 Bug
3. 性能问题
4. 安全隐患
5. 改进建议"""

# 启动服务
if __name__ == "__main__":
    mcp.run()  # 默认使用 stdio 传输
```

### 3.2 运行和测试

```bash
# 使用 MCP Inspector 测试（推荐）
mcp dev src/my_server/server.py

# 直接运行
python src/my_server/server.py

# 安装到 Claude Desktop
mcp install src/my_server/server.py --name "我的工具"
```

### 3.3 异步工具

```python
from mcp.server.fastmcp import FastMCP
import httpx

mcp = FastMCP("异步工具服务")

@mcp.tool()
async def fetch_url(url: str) -> str:
    """抓取网页内容
    
    Args:
        url: 要抓取的 URL
    """
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        return response.text[:2000]  # 截取前 2000 字符

@mcp.tool()
async def query_database(sql: str) -> str:
    """执行数据库查询（只读）
    
    Args:
        sql: SQL 查询语句（仅支持 SELECT）
    """
    if not sql.strip().upper().startswith("SELECT"):
        return "错误：仅支持 SELECT 查询"
    
    # 实际项目中连接数据库
    return f"查询结果：{sql}"
```

---

## 4. 传输层

### 4.1 stdio 传输（本地进程）

```python
# 默认使用 stdio，适合本地工具
if __name__ == "__main__":
    mcp.run()  # 等同于 mcp.run(transport="stdio")
```

### 4.2 SSE 传输（远程服务）

```python
# 使用 SSE 传输，适合远程部署
if __name__ == "__main__":
    mcp.run(transport="sse", host="0.0.0.0", port=8080)
```

### 4.3 Streamable HTTP 传输（最新推荐）

```python
# 使用 Streamable HTTP（协议 2025-06-18 新增）
if __name__ == "__main__":
    mcp.run(transport="streamable-http", host="0.0.0.0", port=8080)
```

---

## 5. 高级功能

### 5.1 Context 上下文

```python
from mcp.server.fastmcp import FastMCP, Context

mcp = FastMCP("上下文示例")

@mcp.tool()
async def long_task(data: str, ctx: Context) -> str:
    """执行长时间任务，带进度报告"""
    # 🔥 报告进度
    await ctx.report_progress(0, 100)
    
    # 模拟处理
    for i in range(10):
        await ctx.report_progress(i * 10, 100)
    
    # 🔥 记录日志
    await ctx.info(f"处理完成: {data}")
    
    # 🔥 读取其他 Resource
    config = await ctx.read_resource("config://app")
    
    return f"处理完成: {data}"
```

### 5.2 资源模板和订阅

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("资源服务")

# 🔥 带参数的资源模板
@mcp.resource("db://{table}/schema")
def get_table_schema(table: str) -> str:
    """获取数据库表结构"""
    schemas = {
        "users": "id INT, name VARCHAR, email VARCHAR",
        "orders": "id INT, user_id INT, amount DECIMAL",
    }
    return schemas.get(table, "表不存在")

# 🔥 动态资源列表
@mcp.resource("logs://recent")
def get_recent_logs() -> str:
    """获取最近的日志"""
    return "2026-03-31 10:00:00 INFO 系统启动\n..."
```

### 5.3 图片和二进制数据

```python
from mcp.server.fastmcp import FastMCP, Image
from PIL import Image as PILImage

mcp = FastMCP("图片工具")

@mcp.tool()
def create_thumbnail(image_path: str) -> Image:
    """创建图片缩略图"""
    img = PILImage.open(image_path)
    img.thumbnail((200, 200))
    return Image(data=img.tobytes(), format="png")
```

---

## 6. 实战案例

### 6.1 数据库查询 MCP Server

```python
"""
数据库查询 MCP Server
支持 MySQL 只读查询
"""
from mcp.server.fastmcp import FastMCP
import aiomysql

mcp = FastMCP("数据库查询工具")

# 数据库连接池
pool = None

async def get_pool():
    global pool
    if pool is None:
        pool = await aiomysql.create_pool(
            host="localhost",
            port=3306,
            user="readonly_user",
            password="password",
            db="mydb",
            minsize=1,
            maxsize=5
        )
    return pool

@mcp.tool()
async def query_sql(sql: str) -> str:
    """执行 SQL 查询（仅支持 SELECT）
    
    Args:
        sql: SQL 查询语句
    """
    # 安全检查
    normalized = sql.strip().upper()
    if not normalized.startswith("SELECT"):
        return "错误：仅支持 SELECT 查询"
    
    forbidden = ["DROP", "DELETE", "UPDATE", "INSERT", "ALTER", "TRUNCATE"]
    for keyword in forbidden:
        if keyword in normalized:
            return f"错误：禁止使用 {keyword} 操作"
    
    db_pool = await get_pool()
    async with db_pool.acquire() as conn:
        async with conn.cursor(aiomysql.DictCursor) as cur:
            await cur.execute(sql)
            rows = await cur.fetchall()
            return str(rows[:100])  # 限制返回行数

@mcp.resource("db://tables")
async def list_tables() -> str:
    """列出所有数据库表"""
    db_pool = await get_pool()
    async with db_pool.acquire() as conn:
        async with conn.cursor() as cur:
            await cur.execute("SHOW TABLES")
            tables = await cur.fetchall()
            return "\n".join(t[0] for t in tables)

@mcp.resource("db://{table}/schema")
async def get_schema(table: str) -> str:
    """获取表结构"""
    db_pool = await get_pool()
    async with db_pool.acquire() as conn:
        async with conn.cursor() as cur:
            await cur.execute(f"DESCRIBE `{table}`")
            columns = await cur.fetchall()
            return "\n".join(f"{c[0]} {c[1]}" for c in columns)

if __name__ == "__main__":
    mcp.run()
```

### 6.2 知识库 RAG MCP Server

```python
"""
知识库 RAG MCP Server
提供文档检索和问答能力
"""
from mcp.server.fastmcp import FastMCP
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings

mcp = FastMCP("知识库检索")

# 初始化向量库
embeddings = OpenAIEmbeddings()
vectorstore = None

def get_vectorstore():
    global vectorstore
    if vectorstore is None:
        vectorstore = FAISS.load_local("./knowledge_base", embeddings)
    return vectorstore

@mcp.tool()
def search_knowledge(query: str, top_k: int = 5) -> str:
    """在知识库中搜索相关文档
    
    Args:
        query: 搜索查询
        top_k: 返回结果数量
    """
    vs = get_vectorstore()
    docs = vs.similarity_search(query, k=top_k)
    
    results = []
    for i, doc in enumerate(docs, 1):
        source = doc.metadata.get("source", "未知")
        results.append(f"[{i}] 来源: {source}\n{doc.page_content[:500]}")
    
    return "\n---\n".join(results)

@mcp.resource("knowledge://stats")
def get_knowledge_stats() -> str:
    """获取知识库统计信息"""
    vs = get_vectorstore()
    return f"知识库文档数量: {vs.index.ntotal}"

@mcp.prompt()
def rag_qa(question: str) -> str:
    """RAG 问答提示词模板"""
    return f"""请基于知识库中的信息回答以下问题。
如果知识库中没有相关信息，请明确说明。

问题：{question}

请先使用 search_knowledge 工具搜索相关信息，然后基于搜索结果回答。"""

if __name__ == "__main__":
    mcp.run()
```

---

## 7. 客户端集成

### 7.1 在 Claude Desktop 中使用

```json
// claude_desktop_config.json
{
  "mcpServers": {
    "my-tools": {
      "command": "python",
      "args": ["path/to/server.py"],
      "env": {
        "DATABASE_URL": "mysql://..."
      }
    },
    "remote-tools": {
      "url": "http://localhost:8080/sse"
    }
  }
}
```

### 7.2 在 OpenAI Agents SDK 中使用

```python
from agents import Agent, Runner
from agents.mcp import MCPServerStdio

async def main():
    async with MCPServerStdio(
        command="python",
        args=["my_server.py"]
    ) as server:
        agent = Agent(
            name="助手",
            instructions="使用 MCP 工具完成任务。",
            mcp_servers=[server]
        )
        result = await Runner.run(agent, "查询数据库中的用户表")
        print(result.final_output)
```

### 7.3 在 LangChain 中使用

```python
from langchain_mcp_adapters.client import MultiServerMCPClient
from langgraph.prebuilt import create_react_agent
from langchain_openai import ChatOpenAI

async def main():
    async with MultiServerMCPClient({
        "db-tools": {
            "command": "python",
            "args": ["db_server.py"]
        }
    }) as client:
        tools = client.get_tools()
        
        agent = create_react_agent(
            ChatOpenAI(model="gpt-4o"),
            tools
        )
        
        result = await agent.ainvoke({
            "messages": [{"role": "user", "content": "查询用户表结构"}]
        })
        print(result)
```

---

## 8. 最佳实践

### 8.1 安全性

```python
# ✅ 输入验证
@mcp.tool()
def safe_query(sql: str) -> str:
    """安全的数据库查询"""
    # 1. 白名单检查
    if not sql.strip().upper().startswith("SELECT"):
        raise ValueError("仅支持 SELECT")
    
    # 2. 禁止危险操作
    dangerous = ["DROP", "DELETE", "UPDATE", "INSERT", "--", ";"]
    for d in dangerous:
        if d in sql.upper():
            raise ValueError(f"禁止使用: {d}")
    
    # 3. 限制返回行数
    if "LIMIT" not in sql.upper():
        sql += " LIMIT 100"
    
    return execute_query(sql)
```

### 8.2 错误处理

```python
@mcp.tool()
async def robust_tool(param: str) -> str:
    """带完善错误处理的工具"""
    try:
        result = await do_something(param)
        return result
    except ConnectionError:
        return "错误：无法连接到服务，请稍后重试"
    except TimeoutError:
        return "错误：操作超时"
    except Exception as e:
        return f"错误：{type(e).__name__}: {str(e)}"
```

---

## 9. 总结

### 核心要点
1. MCP 是 AI 工具集成的标准协议，已获行业广泛支持
2. 三大原语：Tools（模型调用）、Resources（数据读取）、Prompts（模板）
3. FastMCP 提供了 Pythonic 的高级 API，开发体验极佳
4. 支持 stdio/SSE/Streamable HTTP 三种传输方式

### 学习路径
1. 使用 FastMCP 开发第一个 MCP Server
2. 掌握 Tools/Resources/Prompts 三大原语
3. 学习不同传输层的适用场景
4. 集成到 Claude Desktop / OpenAI Agents SDK / LangChain
5. 实战：构建生产级 MCP Server

---

## 🔗 相关资源

- [MCP 官方文档](https://modelcontextprotocol.io/)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- [MCP Servers 仓库](https://github.com/modelcontextprotocol/servers)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)

---

**@author erik.zhou**
