---
title: 工具使用与函数调用
tags: [tool-use, function-calling, api, core-concept]
related: [agent_architecture, mcp_a2a_protocols]
---

## 定义

工具使用使 LLM 智能体能够与外部系统交互——API、数据库、代码解释器、网页浏览器、文件系统——将其能力扩展到文本生成之外。

## 演进历程

```
MRKL (2022)           -> 路由到专家模块（概念性）
Toolformer (2023)     -> 通过自监督学习工具使用
ChatGPT Plugins (2023) -> 商业化工具使用平台
GPT-4 Function Calling -> 结构化 JSON 工具调用
Claude Tool Use       -> 带模式的结构化工具使用
MCP (2024-2026)       -> 工具连接的通用协议
```

## 工作原理

1. **工具定义**：用名称、描述、参数（JSON Schema）描述可用工具
2. **工具选择**：LLM 根据任务决定何时调用哪个工具
3. **参数生成**：LLM 为工具生成结构化输入
4. **执行**：系统执行工具调用
5. **结果处理**：LLM 处理工具输出并继续

## 方法

### 基于提示的工具使用（ReAct 风格）
```
Thought: I need to search for X
Action: search("query")
Observation: [results]
```

### 原生函数调用（现代 API）
```json
{
  "type": "function",
  "function": {
    "name": "search",
    "arguments": {"query": "..."}
  }
}
```

### 学习式工具使用（Toolformer）
模型经过微调，在有益时插入 API 调用。

## 智能体-计算机接口（ACI）

来自 SWE-Agent 的关键洞察：工具接口的设计与模型同等重要。
- 清晰、简洁的输出格式
- 帮助 LLM 恢复的错误消息
- 为 LLM 人体工程学设计的工具，而非为人类人体工程学

## MCP（模型上下文协议）

参见：`mcp_a2a_protocols.md` 了解标准化工具连接的通用协议。
