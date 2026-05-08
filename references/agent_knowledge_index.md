---
title: AI 智能体知识库 - 交叉参考索引
tags: [agent, index, reference]
---

## 知识库概述

本索引提供了此仓库中创建的 AI 智能体知识库条目的交叉参考映射。

## 概念文件

| 文件 | 路径 | 涵盖主题 |
|------|------|----------|
| LLM 智能体基础 | `concepts/llm_agent_fundamentals.md` | 核心定义、组件（大脑、规划、记忆、工具、行动、反馈循环）、智能体循环伪代码 |
| 智能体架构（概述） | `concepts/agent_architecture.md` | 架构图、组件概述、设计决策（预先存在） |
| 智能体架构模式 | `concepts/agent_architecture_patterns.md` | ReAct、规划与执行、Reflexion、多智能体（顺序、层级、辩论、对等、投票） |
| 智能体框架 | `concepts/agent_frameworks.md` | LangChain/LangGraph、AutoGen、CrewAI、MetaGPT、Claude SDK、OpenAI SDK、Dify、Coze、对比矩阵 |
| 智能体技术 | `concepts/agent_techniques.md` | 函数调用、RAG、思维链、思维树、自一致性、提示链 |
| 智能体记忆（概述） | `concepts/agent_memory.md` | 记忆分类体系、生命周期、五大机制族、关键架构（预先存在） |
| 智能体记忆系统 | `concepts/agent_memory_systems.md` | 短期、长期、情景、语义、程序性、基于 RAG 的记忆、反思、遗忘 |

## 论文笔记

| 文件 | 路径 | 涵盖主题 |
|------|------|----------|
| LLM 智能体综述 | `papers/llm/llm_agent_surveys.md` | 10 篇带注释的综述论文、分类体系、阅读顺序、关键基准 |

## 主题交叉参考

| 主题 | 主要文件 | 支持文件 |
|------|----------|----------|
| ReAct 模式 | `agent_architecture_patterns.md` | `llm_agent_fundamentals.md` |
| 规划与执行 | `agent_architecture_patterns.md` | `agent_frameworks.md`（LangGraph） |
| Reflexion | `agent_architecture_patterns.md` | `llm_agent_surveys.md` |
| 多智能体系统 | `agent_architecture_patterns.md` | `agent_frameworks.md`、`llm_agent_surveys.md` |
| 记忆系统 | `agent_memory_systems.md` | `llm_agent_fundamentals.md`、`llm_agent_surveys.md` |
| RAG | `agent_techniques.md`、`agent_memory_systems.md` | `llm_agent_surveys.md` |
| 函数调用 / 工具使用 | `agent_techniques.md` | `llm_agent_fundamentals.md`、`llm_agent_surveys.md` |
| 思维链 | `agent_techniques.md` | `agent_architecture_patterns.md` |
| 思维树 | `agent_techniques.md` | `agent_architecture_patterns.md` |
| 框架对比 | `agent_frameworks.md` | -- |
| 智能体评估 / 基准 | `llm_agent_surveys.md` | -- |

## 关键论文参考（按 arXiv ID）

| arXiv ID | 论文 | 主题 |
|----------|------|------|
| 2210.03629 | ReAct (Yao et al.) | 推理 + 行动模式 |
| 2303.11366 | Reflexion (Shinn et al.) | 自我反思模式 |
| 2309.07864 | 复旦智能体综述 (Xi et al.) | 全面智能体综述 |
| 2308.11432 | 人大智能体综述 (Wang et al.) | 智能体构建/应用 |
| 2309.02427 | CoALA (Sumers et al.) | 认知架构框架 |
| 2401.03568 | Agent AI (Durante et al.) | 多模态智能体综述 |
| 2404.13501 | 记忆综述 (Zhang et al.) | 智能体记忆系统 |
| 2402.01680 | 多智能体综述 (Guo et al.) | 多智能体协作 |
| 2405.17935 | 工具学习综述 (Qu et al.) | LLM 的工具使用 |
| 2312.10997 | RAG 综述 (Gao et al.) | 检索增强生成 |
| 2404.11584 | 架构全景 (Masterman) | 架构选择指南 |
| 2305.10601 | 思维树 (Yao et al.) | 树结构推理 |
| 2203.11171 | 思维链 (Wei et al.) | 逐步推理 |
| 2203.11171 | 自一致性 (Wang et al.) | CoT 上的多数投票 |
