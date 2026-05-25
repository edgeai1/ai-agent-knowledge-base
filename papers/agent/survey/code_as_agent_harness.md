---
title: "Code as Agent Harness"
authors:
  - Xuying Ning
  - Katherine Tieu
  - Dongqi Fu
  - Tianxin Wei
  - Zihao Li
  - "+ 37 co-authors"
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.18747
tags:
  - survey
  - code-as-harness
  - agent-infrastructure
  - tool-use
  - planning
  - multi-agent
status: done
---

# Code as Agent Harness：代码作为智能体执行框架的综述

## 摘要

这是 2026 年 5 月发布的**重要综述**：将代码（code）重新定义为智能体系统的**操作基础设施（operational substrate）**，而非仅仅是输出物。综述包含 42 位作者，系统梳理了代码在智能体推理、行动、环境建模、执行验证中的核心作用，组织为**三层框架**：harness 接口层、harness 机制层、规模化层。覆盖从单代理到多代理、从编程助手到科学发现、从 GUI 自动化到企业工作流的全景应用。

## 动机与问题

### 代码角色的演变

| 时期 | 代码的角色 |
|------|---------|
| 2020 前 | 工具调用的"语法糖"（Codex, Github Copilot） |
| 2022-2023 | LLM 的输出之一（Chain-of-Thought, CodeAct） |
| 2024-2025 | LLM 的"思考语言"（推理链用代码表达） |
| **2026 - 现在** | **智能体系统的"操作基础设施"** |

### 现状：代码无处不在但研究分散

近年来 LLM 代理的代码使用呈爆发增长，但研究分散在不同子领域：

- **CodeAct**：推理用代码表达
- **OpenHands、SWE-Agent**：代码修改代理
- **GPT Engineer**：代码生成代理
- **Voyager**：代码作为可学习的 skill
- **MOSS**（[[moss]]）：代码作为自演化对象

**缺乏统一视角**让研究者难以看清全局。

### 新视角：Code as Harness

> 代码不是智能体的"输出"，而是智能体**操作的基础**：
> - 推理通过代码执行被验证
> - 行动通过代码调用工具
> - 环境通过代码建模
> - 记忆通过代码结构化

这一视角统一了过去看似无关的工作。

## 方法：三层框架

### Layer 1: Harness Interface（接口层）

代码作为"思考与行动"之间的接口：

```
[Reasoning (LLM 思维)]
       ↕
[Code (Harness)]
       ↕
[Action (工具调用 / 环境交互)]
```

#### 子主题

- **Code as Reasoning**：LLM 用代码表达推理过程（CodeAct, PoT）。
- **Code as Action**：代码作为工具调用的统一接口（function calling 的代码化）。
- **Code as Plan**：代码作为可执行的计划（vs 自然语言计划）。

### Layer 2: Harness Mechanisms（机制层）

代码如何承载智能体的核心机制：

#### Planning

- 代码作为可执行 plan（任务分解 + 依赖图）
- 动态 plan 修改（运行时 plan 重写）
- Plan as Code Repository（Voyager 的 skill library）

#### Memory

- Code-structured memory（用数据结构组织记忆）
- 记忆压缩为代码片段（可执行的记忆）
- 跨会话代码持久化

#### Tool Use

- Tool 作为可调用的代码模块
- 动态工具发现（导入未知库）
- 工具组合（代码化的 tool chaining）

#### Feedback & Optimization

- 执行结果作为反馈信号
- 代码作为可微对象（如 DSPy）
- 代码自我修复（MOSS, Self-Edit）

### Layer 3: Scaling（规模化层）

从单代理到多代理：

#### 单代理 → 多代理

- 共享代码 artifacts（图书馆模型）
- 代码作为代理间通信协议
- 分布式代码执行（如 Ray）

#### 状态一致性

- 多代理修改同一代码的冲突解决
- 代码版本控制 + 协同编辑
- Git-style 多代理工作流

## 应用全景

综述梳理了 8 个主要应用域：

```
1. Coding Assistants:    GitHub Copilot, Cursor, Claude Code
2. GUI / OS Automation:  Anthropic Computer Use, GUI agents
3. Embodied Agents:      机器人代码生成
4. Scientific Discovery: AI Scientist, ResearchAgent
5. Personalization:      用户特定代码生成
6. Recommendation:       生成式推荐系统的代码控制
7. DevOps:               自动化运维代码
8. Enterprise Workflow:  企业流程自动化
```

### 跨应用的共同模式

```
所有应用都依赖：
  ✓ 代码生成质量（生成层）
  ✓ 代码执行环境（执行层）
  ✓ 代码错误处理（反馈层）
  ✓ 代码版本管理（演化层）
```

研究方向应聚焦这些**横切关注点**，而非各应用独立优化。

## 开放挑战

综述总结 6 大未解决问题：

### 1. Evaluation 方法学

- 如何评估"代码作为推理"的质量？传统代码评测（pass@1）不够。
- 如何评估代码 harness 的整体效率？

### 2. Verification with Incomplete Feedback

- 大部分代码执行只给"成功/失败"二值反馈。
- 如何用稀疏反馈训练高质量代码生成？

### 3. Regression Prevention

- 自演化代码（如 [[moss]]）容易引入回归。
- 如何形式化保证不破坏现有功能？

### 4. Multi-Agent State Consistency

- 多代理修改共享代码空间的一致性。
- Git 模型是否够用？

### 5. Human Safety Oversight

- 代理修改自己代码的安全边界。
- 哪些代码 / 库 / API 不能被代理修改？

### 6. Multimodal Environment

- 代码作为推理是否能扩展到图像、视频？
- 视觉编程语言是否会成为代理新接口？

## 综述的价值

### 1. 统一视角

将散落的研究（CodeAct、SWE-Agent、Voyager、MOSS、AI Scientist...）放入同一框架，让研究者**看清自己工作的位置**。

### 2. 揭示横切关注点

不再"每个应用单独优化"，而是**共同的核心问题**（评估、验证、规模化）。

### 3. 明确未来方向

6 个开放挑战指明研究优先级。

### 4. 跨领域桥梁

42 位作者来自 NLP、SE、HCI、Robotics、Systems 等领域，**综述本身就是跨学科合作的成果**。

## 贡献

1. **概念创新**：从"code as output" → "code as harness"。
2. **三层框架**：接口 / 机制 / 规模化的清晰组织。
3. **跨应用覆盖**：8 个主要应用域全景。
4. **开放挑战**：6 大未解决问题指引未来研究。
5. **广泛作者基础**：42 位作者跨多个领域。

## 局限与未解决问题

- 综述本身不提出新方法。
- 三层框架的边界在某些工作上模糊（如 Voyager 同时跨多层）。
- 工业实践（如 OpenAI Code Interpreter 等闭源系统）的细节缺失。

## 与相关工作的关系

### 综述位置
- 与 [[deep_research_survey]] 互补：DR Survey 关注研究流程，本综述关注代码基础设施。
- 与 [[deep_research_of_deep_research]] 互补：DRDR 从 Transformer 到 Agent，本综述从 Code 到 Agent。
- 与 [[opd_survey]] 互补：OPD Survey 关注模型训练，本综述关注代理执行。

### 引用密度
- 大量引用 [[moss]]（源代码自演化）、[[voyager]]（代码作为 skill）、[[codeact]]（代码作为推理）等核心工作。
- 是 2026 年代理研究者的**必读综述**。

### 后续方向
- 形式化"代码 harness"的理论（信息论 / 计算复杂性）。
- 跨语言代码 harness（不限于 Python）。
- 代码 harness 的工程标准化（如 ISO 标准）。
