---
title: "MCP 与 A2A：智能体互操作协议"
tags: [mcp, a2a, protocol, interoperability, core-concept]
related: [tool_use, multi_agent_systems]
---

## 概述

两个互补的协议定义了 2026 年的智能体生态系统：
- **MCP**（模型上下文协议）：智能体到工具的通信
- **A2A**（Agent-to-Agent）：智能体到智能体的通信

```
┌─────────┐     A2A      ┌─────────┐
│ Agent A │ <──────────> │ Agent B │
└────┬────┘              └────┬────┘
     │ MCP                    │ MCP
┌────▼────┐              ┌────▼────┐
│  Tools  │              │  Tools  │
│  Data   │              │  Data   │
└─────────┘              └─────────┘
```

## MCP（模型上下文协议）

**创建者**：Anthropic（2024 年 11 月），捐赠给 Linux Foundation AAIF（2025 年 12 月）

**目的**：标准化 AI 智能体连接外部工具、数据源和服务的方式。"AI 的 USB-C。"

**采用情况（2026 年 2 月）**：每月 SDK 下载量超过 9700 万（Python + TypeScript 合计）。被 Anthropic、OpenAI、Google、Microsoft、Amazon 采用。

### 工作原理

MCP 定义了客户端-服务器架构：
- **MCP 主机**：AI 应用程序（Claude、ChatGPT、自定义智能体）
- **MCP 服务器**：通过标准协议暴露工具、资源和提示
- **传输层**：stdio、HTTP/SSE

### 关键概念

- **工具**：智能体可以调用的函数（例如 `search_database`、`send_email`）
- **资源**：智能体可以读取的数据（例如文件、数据库记录）
- **提示**：可复用的提示模板

### 示例

Slack 的 MCP 服务器提供 `send_message`、`search_messages`、`list_channels` 等工具——任何 MCP 兼容的智能体都可以使用它，无需自定义集成代码。

## A2A（Agent-to-Agent 协议）

**创建者**：Google（2025 年 4 月），现归 Linux Foundation

**目的**：标准化来自不同提供商/框架的自主 AI 智能体之间的安全、结构化通信。

### 工作原理

- **Agent Cards**：描述智能体能力、技能和端点的 JSON 文档
- **任务**：智能体之间交换的工作单元
- **流式传输**：任务进度的实时更新
- **推送通知**：异步完成提醒

### 关键特性

- 框架无关（适用于 LangGraph、AutoGen、CrewAI 等）
- 安全的智能体发现和能力协商
- 支持长时间运行的任务

## 它们如何互补

| 层级 | 协议 | 功能 |
|------|------|------|
| 智能体 <-> 工具 | **MCP** | 将智能体连接到外部工具和数据 |
| 智能体 <-> 智能体 | **A2A** | 实现多智能体协作 |

大多数现代智能体 AI 系统同时利用**两种**协议：
- MCP 用于可靠的工具和上下文集成
- A2A 用于编排智能体之间的协作

## 其他协议（2026）

- **ACP**（智能体通信协议）：IBM 的替代方案
- **UCP**：试图统一 MCP + A2A 的新兴通用协议

## 参考文献

- MCP Spec: https://modelcontextprotocol.io
- A2A Spec: https://github.com/google/A2A
- Linux Foundation AAIF: Agentic AI Foundation
