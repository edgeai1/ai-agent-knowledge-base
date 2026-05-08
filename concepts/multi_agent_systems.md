---
title: 多智能体系统
tags: [multi-agent, collaboration, orchestration, core-concept]
related: [agent_architecture, mcp_a2a_protocols]
---

## 定义

多个基于 LLM 的智能体协作解决对单个智能体来说过于复杂的任务，每个智能体具有专业化的角色、知识或能力。

## 协作模式

### 1. 顺序 / 流水线
智能体按顺序工作，每个处理一个阶段。
```
Agent A (plan) -> Agent B (code) -> Agent C (test) -> Agent D (review)
```
示例：ChatDev、MetaGPT

### 2. 层级 / 委派
管理者智能体委派任务给工作者智能体。
```
      Manager
      /  |  \
    A    B    C  （专家）
```
示例：CrewAI、AutoGen 带管理者的群组聊天

### 3. 辩论 / 讨论
智能体讨论并批评彼此的输出。
```
Agent A <-> Agent B  （争论直到达成共识）
```
示例：CAMEL、多智能体辩论

### 4. 合作 / 群体
智能体作为对等方协调，没有显式层级。
```
A <-> B <-> C  （网状通信）
```
示例：OpenAI Swarm

## 关键框架（2026）

| 框架 | 模式 | 优势 |
|------|------|------|
| **LangGraph** | 基于图的状态机 | 灵活、生产就绪、LangChain 生态系统 |
| **AutoGen** | 基于对话 | 灵活的模式、Microsoft 支持 |
| **CrewAI** | 基于角色的委派 | 简单 API、面向生产 |
| **MetaGPT** | SOP 编码协作 | 结构化工作流、工件驱动 |
| **Claude Agent SDK** | 可编程智能体循环 | 与 Claude Code 相同的引擎、原生 MCP |
| **OpenAI Agents SDK** | 基于切换 | 简单的智能体间委派 |
| **Google ADK 1.0** | A2A 原生 | 多智能体标准、Gemini 集成 |

## 挑战

1. **协调开销**：更多智能体 != 更好的结果
2. **错误传播**：错误在智能体链中级联
3. **成本**：每次智能体调用 = API 成本
4. **评估**：多智能体系统更难基准测试
5. **可扩展性**：随着智能体数量增加，收益递减

## 2026 趋势

- **CORAL**：自进化多智能体系统，具有共享持久记忆（比固定基线提高 3-10 倍）
- **DyTopo**：动态拓扑路由——基于语义的智能体连接重新布线
- **标准化**：MCP + A2A 协议趋向融合，实现全栈多智能体互操作
