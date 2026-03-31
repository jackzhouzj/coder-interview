# AI Agent开发

## 📋 模块概述

本模块涵盖 AI Agent 开发的核心技术，从基础框架到前沿协议，包括 OpenAI Agents SDK、MCP 协议、A2A 协议、CrewAI 等框架，以及 Agent 记忆系统、协作架构和自主 Agent 设计。这是 2025-2026 年 AI 领域最热门的方向。

## 📚 学习内容

### 🔥 新增：前沿 Agent 技术（2026）

### 1. OpenAI Agents SDK 🔥 新增
- 四大核心原语：Agent / Tool / Handoff / Guardrail
- 多 Agent 协作与交接机制
- 内置 Tracing 可观测性
- MCP 工具集成

### 2. MCP 协议开发 🔥 新增
- MCP 协议架构与三大原语（Tools/Resources/Prompts）
- FastMCP 高级 API 开发
- 传输层（stdio/SSE/Streamable HTTP）
- 与各框架集成（Claude/OpenAI/LangChain）

### 3. A2A 协议与 Google ADK 🔥 新增
- A2A（Agent-to-Agent）协议规范
- Google ADK Agent 开发
- 跨平台 Agent 协作
- MCP + A2A 融合架构

### 4. Agent 记忆系统 🔥 新增
- 短期记忆（滑动窗口/Token感知/摘要压缩）
- 长期记忆（向量存储/Mem0框架）
- 情景记忆与语义记忆
- 生产级记忆架构设计

### 经典 Agent 框架

### 5. CrewAI
- 多Agent协作与角色扮演
- 任务分配与团队协作模式

### 6. AutoGPT / BabyAGI
- 自主任务执行与目标分解
- 任务优先级管理

### 7. Agent协作框架
- Agent通信协议与协作模式
- 冲突解决与性能优化

### 8. 自主Agent设计
- Agent架构设计（分层/BDI/认知）
- 决策机制与学习能力
- 安全性与可解释性

## 🎯 学习目标

- [ ] 掌握 OpenAI Agents SDK 构建生产级 Agent
- [ ] 能够开发 MCP Server 并集成到各平台
- [ ] 理解 A2A 协议，实现跨框架 Agent 协作
- [ ] 设计和实现 Agent 记忆系统
- [ ] 能够使用 CrewAI 构建多 Agent 系统
- [ ] 理解 Agent 协作机制和自主 Agent 设计

## 📖 子模块

### 🔥 前沿技术（推荐优先学习）
1. [OpenAI Agents SDK 完整教程](OpenAI-Agents-SDK-完整教程.md) ✅ 新增
2. [MCP 协议开发完整教程](MCP协议开发-完整教程.md) ✅ 新增
3. [A2A 协议与 Google ADK 完整教程](A2A协议与Google-ADK-完整教程.md) ✅ 新增
4. [Agent 记忆系统完整教程](Agent记忆系统-完整教程.md) ✅ 新增
5. [Function Calling 与工具使用完整教程](Function-Calling与工具使用-完整教程.md) ✅ 新增

### Agent 平台与框架
6. [Microsoft AutoGen 完整教程](AutoGen-完整教程.md) ✅ 新增
7. [Dify 平台实战完整教程](Dify平台实战-完整教程.md) ✅ 新增
8. [CrewAI 完整教程](CrewAI-完整教程.md) ✅

### 生产治理
13. [Agent 评估与安全护栏完整教程](Agent评估与安全护栏-完整教程.md) ✅ 新增

### 经典框架与理论
14. [AutoGPT 完整教程](AutoGPT-完整教程.md) ✅
15. [BabyAGI 完整教程](BabyAGI-完整教程.md) ✅
16. [Agent 协作框架完整教程](Agent协作框架-完整教程.md) ✅
17. [自主 Agent 设计完整教程](自主Agent设计-完整教程.md) ✅

## ⏱️ 预计学习时长

| 教程 | 学习时长 | 优先级 |
|------|----------|--------|
| OpenAI Agents SDK | 15-20小时 | P0 🔥 |
| MCP 协议开发 | 20-25小时 | P0 🔥 |
| A2A 协议与 Google ADK | 15-20小时 | P1 |
| Agent 记忆系统 | 15-20小时 | P0 🔥 |
| CrewAI | 20-25小时 | P1 |
| AutoGPT | 15-20小时 | P2 |
| BabyAGI | 10-15小时 | P2 |
| Agent 协作框架 | 15-20小时 | P1 |
| 自主 Agent 设计 | 20-25小时 | P1 |
| **总计** | **145-190小时** | |

## 📝 推荐学习路径

```
OpenAI Agents SDK → MCP 协议开发 → Agent 记忆系统 →
CrewAI → LangGraph（见05-大模型应用） → A2A 协议 →
Agent 协作框架 → 自主 Agent 设计
```

## 📝 前置知识

- Python 异步编程（async/await）
- LangChain / LangGraph 基础
- Prompt Engineering
- OpenAI API 使用经验

## 🔗 相关资源

- [OpenAI Agents SDK](https://github.com/openai/openai-agents-python)
- [MCP 协议](https://modelcontextprotocol.io/)
- [Google A2A 协议](https://github.com/google/A2A)
- [Google ADK](https://github.com/google/adk-python)
- [CrewAI 官方文档](https://docs.crewai.com/)
- [Mem0 记忆框架](https://docs.mem0.ai/)
- [LangGraph](https://langchain-ai.github.io/langgraph/)

---

**@author erik.zhou**
