---
title: 智能体框架对比
tags: [agent, framework, langchain, langgraph, autogen, crewai, metagpt, claude-sdk, openai-sdk, dify, coze]
related: [llm_agent_fundamentals, agent_architecture_patterns, agent_techniques]
---

## 定义

智能体框架是提供抽象、原语和基础设施的软件库和平台，用于构建基于 LLM 的智能体。它们处理智能体循环、工具集成、记忆管理和多智能体编排的样板代码。

---

## 框架 1：LangChain / LangGraph

### 概述
- **开发者**：LangChain Inc. (Harrison Chase)
- **语言**：Python、JavaScript/TypeScript
- **许可证**：MIT
- **仓库**：github.com/langchain-ai/langchain、github.com/langchain-ai/langgraph
- **首次发布**：2022 年 10 月（LangChain）、2024 年 1 月（LangGraph）

### 架构
LangChain 最初是一个基于链的框架（顺序 LLM 调用），后来演变为模块化生态系统：

- **LangChain Core**：基础抽象（LLM、提示、输出解析器、工具）
- **LangChain Community**：第三方集成（向量存储、工具、LLM 提供商）
- **LangGraph**：基于图的智能体编排框架（2024 年以后推荐的智能体构建方法）
- **LangSmith**：可观测性、测试和评估平台
- **LangServe**：部署为 REST API

### LangGraph（智能体框架）
LangGraph 将智能体建模为**有状态图**：
- **节点**：处理状态的函数（LLM 调用、工具执行、路由逻辑）
- **边**：节点之间的转换（条件或无条件）
- **状态**：在图中流动的共享类型化状态对象
- **检查点**：内置持久化，支持长时间运行的智能体、人在回路中和故障恢复

```python
# LangGraph conceptual example
from langgraph.graph import StateGraph, MessagesState

graph = StateGraph(MessagesState)
graph.add_node("agent", call_model)
graph.add_node("tools", call_tools)
graph.add_edge("__start__", "agent")
graph.add_conditional_edges("agent", should_continue, {"continue": "tools", "end": "__end__"})
graph.add_edge("tools", "agent")
app = graph.compile()
```

### 主要特性
- 模型无关（OpenAI、Anthropic、Google、开源）
- 庞大的集成生态系统（700+ 集成）
- LangGraph 支持循环、分支、并行执行
- 通过中断/恢复实现人在回路中
- 流式传输支持（token 级和事件级）
- LangGraph Cloud 用于托管部署

### 优势
- 最大的社区和生态系统
- 灵活且可组合
- LangGraph 提供对智能体行为的精细控制
- 优秀的文档和教程
- 通过 LangSmith 实现强大的可观测性

### 局限性
- LangChain（旧版链/智能体）被批评过度抽象
- LangGraph 的图范式学习曲线陡峭
- 历史上频繁的破坏性 API 更改
- 对于简单用例可能过于冗长

---

## 框架 2：AutoGen（Microsoft）

### 概述
- **开发者**：Microsoft Research
- **语言**：Python、.NET（AutoGen 0.4+）
- **许可证**：MIT（部分组件为 Creative Commons）
- **仓库**：github.com/microsoft/autogen
- **首次发布**：2023 年 9 月
- **重大重写**：AutoGen 0.4 (AgentChat) -- 2024 年底，完全重新设计

### 架构（AutoGen 0.4 / AgentChat）
AutoGen 0.4 是一个具有分层架构的完全重写版本：

- **核心层**：异步、事件驱动的智能体运行时，支持消息传递
- **AgentChat 层**：用于多智能体对话的高级 API
- **扩展**：模型客户端、工具、代码执行

关键原语：
- **智能体**：AssistantAgent、UserProxyAgent、自定义智能体
- **团队**：RoundRobinGroupChat、SelectorGroupChat、Swarm、MagenticOneGroupChat
- **终止条件**：MaxMessageTermination、TextMentionTermination 等
- **模型客户端**：OpenAI、Anthropic 等

```python
# AutoGen 0.4 conceptual example
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import RoundRobinGroupChat

agent1 = AssistantAgent("coder", model_client=model, tools=[...])
agent2 = AssistantAgent("reviewer", model_client=model)
team = RoundRobinGroupChat([agent1, agent2], max_turns=10)
result = await team.run(task="Build a web scraper")
```

### 主要特性
- 一流的多智能体对话
- 多种团队拓扑（轮询、基于选择器、群体、Magentic-One）
- 内置代码执行（Docker 沙箱、本地）
- 人在回路中支持
- 跨语言支持（.NET、Python）
- Magentic-One：预构建的多智能体团队，用于复杂的网页/文件任务

### 优势
- 多智能体系统方面表现优秀
- 干净的异步优先架构（0.4）
- 强大的代码执行能力
- 由 Microsoft Research 支持
- AutoGen Studio：用于构建智能体团队的无代码 UI

### 局限性
- 0.2 和 0.4 之间的破坏性更改（需要迁移）
- 0.4 版本的文档仍在完善中
- 社区比 LangChain 小
- 高级场景设置复杂

---

## 框架 3：CrewAI

### 概述
- **开发者**：CrewAI Inc. (Joao Moura)
- **语言**：Python
- **许可证**：MIT
- **仓库**：github.com/crewAIInc/crewAI
- **首次发布**：2023 年 12 月

### 架构
CrewAI 使用受现实世界团队结构启发的角色扮演隐喻：

- **智能体**：具有角色、目标、背景故事和工具的自主单元
- **任务**：具有描述、预期输出和指定智能体的具体分配
- **团队**：一组智能体共同处理一组任务
- **流程**：工作流类型（顺序、层级或共识）

```python
# CrewAI conceptual example
from crewai import Agent, Task, Crew, Process

researcher = Agent(
    role="Senior Research Analyst",
    goal="Find the latest AI trends",
    backstory="You are an expert researcher...",
    tools=[search_tool, scrape_tool]
)

writer = Agent(
    role="Tech Writer",
    goal="Write compelling articles",
    backstory="You are a skilled writer..."
)

research_task = Task(description="Research AI agent frameworks", agent=researcher)
write_task = Task(description="Write article based on research", agent=writer)

crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential
)
result = crew.kickoff()
```

### 主要特性
- 基于现实世界团队隐喻的简单直观 API
- 基于角色的智能体设计（角色、目标、背景故事）
- 顺序和层级流程类型
- 内置智能体之间的委派
- 记忆（短期、长期、实体记忆）
- CrewAI Enterprise 用于生产部署
- Flows：将团队与过程式代码结合的结构化工作流

### 优势
- 智能体框架中最容易学习
- 直观的心智模型（智能体作为团队成员）
- 快速原型制作
- 适合内容创建、研究、分析流水线
- 不断增长的预构建工具生态系统

### 局限性
- 对于复杂控制流不如 LangGraph 灵活
- 对高级智能体模式（如动态智能体创建）支持有限
- 相对年轻的框架
- 调试可能不够透明（智能体通信冗长）
- 冗长的智能体间通信带来性能开销

---

## 框架 4：MetaGPT

### 概述
- **开发者**：DeepWisdom (Alexander Wu et al.)
- **语言**：Python
- **许可证**：MIT
- **仓库**：github.com/geekan/MetaGPT
- **首次发布**：2023 年 6 月
- **论文**：Hong et al., "MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework" (2023)

### 架构
MetaGPT 围绕**标准操作流程（SOP）**构建多智能体协作——模拟真实软件公司的运作方式：

- **角色**：ProductManager、Architect、ProjectManager、Engineer、QA
- **行动**：每个角色有定义的行动（WritePRD、DesignAPI、WriteCode、RunTest）
- **消息总线**：智能体之间的发布-订阅通信
- **共享环境**：包含文档、代码和工件的共享工作区
- **SOP 工作流**：预定义的角色交互序列

### 主要特性
- 软件开发生命周期模拟
- 每个阶段的结构化输出（PRD、系统设计、API 规范、代码、测试）
- 基于角色的设计，职责明确
- 发布-订阅消息架构
- 带有迭代优化的增量开发

### 优势
- 产生高质量的结构化软件工件
- 通过角色约束和输出模式减少幻觉
- 新颖的 SOP 方法提供一致性
- 适合代码生成和软件工程任务

### 局限性
- 主要专注于软件开发用例
- 代码库复杂，难以扩展到其他领域
- token 消耗高（多个智能体，长输出）
- 不如 LangGraph 或 AutoGen 通用

---

## 框架 5：Claude Agent SDK（Anthropic）

### 概述
- **开发者**：Anthropic
- **语言**：Python、TypeScript
- **许可证**：MIT
- **首次发布**：2025 年 3 月（作为 Claude 生态系统的一部分）

### 架构
Claude Agent SDK 提供了一个极简、有主见的框架，用于使用 Claude 构建智能体：

- **智能体**：核心智能体类，包含模型、指令、工具和可选的切换目标
- **工具**：函数工具、Bash 工具、Computer 工具、MCP 服务器
- **切换**：在专业化智能体之间转移控制权
- **安全防护**：用于安全的输入/输出验证

```python
# Claude Agent SDK conceptual example
from claude_agent_sdk import Agent, Runner
from claude_agent_sdk.tools import FunctionTool

agent = Agent(
    name="research_assistant",
    model="claude-sonnet-4-20250514",
    instructions="You are a helpful research assistant...",
    tools=[search_tool, calculator_tool],
    handoffs=[specialist_agent]
)

result = Runner.run(agent, messages=[{"role": "user", "content": "..."}])
```

### 主要特性
- 与 Claude 模型紧密集成
- 通过模型上下文协议（MCP）实现标准化工具连接
- 多智能体切换（群体式智能体切换）
- 内置安全防护
- 计算机使用能力（GUI 交互）
- 追踪和可观测性

### 设计理念
- 最小抽象（薄封装，而非重量级框架）
- 智能体"只是"具有工具和指令的 Claude
- 强调简洁性和 Claude 原生模式
- MCP 作为通用工具集成标准

### 优势
- 简单、极简的 API
- 与 Claude 模型一流集成
- MCP 提供标准化、可复用的工具生态系统
- 计算机使用用于 GUI 自动化
- 强大的安全特性（安全防护）

### 局限性
- 仅限 Claude（非模型无关）
- 较新的框架，生态系统比 LangChain 小
- 比 LangGraph 内置的编排模式少
- MCP 生态系统仍在增长中

---

## 框架 6：OpenAI Agents SDK

### 概述
- **开发者**：OpenAI
- **语言**：Python
- **许可证**：MIT
- **首次发布**：2025 年 3 月（由 Swarm 框架演变而来）
- **仓库**：github.com/openai/openai-agents-python

### 架构
OpenAI Agents SDK 是一个轻量级、面向生产的框架：

- **智能体**：具有指令、工具和切换能力的 LLM
- **切换**：将对话转移给另一个智能体（继承自 Swarm）
- **安全防护**：用于安全的输入/输出验证器
- **追踪**：内置可观测性和调试

```python
# OpenAI Agents SDK conceptual example
from agents import Agent, Runner

agent = Agent(
    name="assistant",
    instructions="You are a helpful assistant...",
    tools=[search_tool, code_tool],
    handoffs=[triage_agent, specialist_agent]
)

result = Runner.run(agent, messages=[...])
```

### 主要特性
- 内置追踪和可观测性
- 多智能体切换（来自 Swarm）
- 输入/输出验证的安全防护
- 上下文管理
- 与 OpenAI 模型和工具集成

### 优势
- 简单、极简的 API（与 Claude Agent SDK 理念相似）
- 原生 OpenAI 模型集成
- 内置追踪
- 面向生产的设计

### 局限性
- 仅限 OpenAI（非模型无关）
- 与 LangGraph 相比编排能力有限
- 相对较新，社区资源有限

---

## 框架 7：平台型框架（Dify、Coze 等）

### Dify
- **类型**：开源 LLM 应用开发平台（附带云服务）
- **仓库**：github.com/langgenius/dify
- **主要特性**：
  - 可视化工作流构建器（拖拽式智能体设计）
  - 内置知识库管理的 RAG 流水线
  - 支持多个 LLM 提供商
  - 智能体和聊天机器人模板
  - API 优先设计，便于嵌入
  - 内置评估和监控
- **最适用于**：希望以可视化、无代码/低代码方式构建 AI 应用和智能体的团队
- **优势**：非开发人员可用、内置 RAG、良好的 UI
- **局限性**：不如代码优先的框架灵活、供应商依赖

### Coze（字节跳动）
- **类型**：机器人/智能体构建平台
- **主要特性**：
  - 带插件系统的可视化机器人构建器
  - 内置知识库、记忆和工作流能力
  - 常见集成的预构建插件
  - 多平台部署（Discord、Telegram、Slack、Web）
  - 带触发器的工作流自动化
- **最适用于**：快速构建和部署聊天机器人和简单智能体
- **优势**：易于使用、多平台部署、免费层
- **局限性**：可定制性较低、平台锁定、主要面向消费者

### 其他
- **Flowise**：开源拖拽式 LLM 流程构建器（基于 LangChain）
- **Botpress**：具有 LLM 智能体能力的聊天机器人平台
- **Wordware**：用于构建 LLM 智能体的协作 IDE
- **Relevance AI**：无代码 AI 智能体构建和部署平台

---

## 对比矩阵

| 特性 | LangGraph | AutoGen 0.4 | CrewAI | MetaGPT | Claude SDK | OpenAI SDK | Dify |
|------|-----------|-------------|--------|---------|------------|------------|------|
| **方法** | 基于图 | 多智能体对话 | 角色扮演 | 基于 SOP | 极简封装 | 极简封装 | 可视化构建器 |
| **多智能体** | 是 | 优秀 | 良好 | 优秀 | 切换 | 切换 | 基础 |
| **模型无关** | 是 | 是 | 是 | 是 | 否（Claude） | 否（OpenAI） | 是 |
| **学习曲线** | 高 | 中 | 低 | 中 | 低 | 低 | 非常低 |
| **灵活性** | 非常高 | 高 | 中 | 中 | 中 | 中 | 低 |
| **生产就绪** | 是 | 是 | 发展中 | 发展中 | 是 | 是 | 是 |
| **代码 vs 无代码** | 代码 | 代码 | 代码 | 代码 | 代码 | 代码 | 两者 |
| **最适用于** | 复杂工作流 | 多智能体 | 团队模拟 | 软件开发 | Claude 应用 | OpenAI 应用 | 快速原型 |
| **社区规模** | 非常大 | 大 | 大 | 中 | 增长中 | 增长中 | 大 |

## 选择指南

- **"我需要最大的灵活性和控制力"** -> LangGraph
- **"我需要多智能体协作"** -> AutoGen 或 CrewAI
- **"我想要最简单的智能体"** -> Claude SDK 或 OpenAI SDK
- **"我在构建软件工程智能体"** -> MetaGPT
- **"我想要无代码/可视化构建"** -> Dify、Coze 或 Flowise
- **"我专注于 Claude/Anthropic"** -> Claude Agent SDK + MCP
- **"我专注于 OpenAI"** -> OpenAI Agents SDK

## 参考文献

- LangChain docs: python.langchain.com
- LangGraph docs: langchain-ai.github.io/langgraph/
- AutoGen docs: microsoft.github.io/autogen/
- CrewAI docs: docs.crewai.com
- MetaGPT: github.com/geekan/MetaGPT
- Claude Agent SDK: docs.anthropic.com
- OpenAI Agents SDK: github.com/openai/openai-agents-python
- Dify: docs.dify.ai
