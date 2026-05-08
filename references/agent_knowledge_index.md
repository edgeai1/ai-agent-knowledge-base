---
title: AI Agent Knowledge Base - Cross-Reference Index
tags: [agent, index, reference]
---

## Knowledge Base Overview

This index provides a cross-reference map for the AI Agent knowledge base entries created in this repository.

## Concept Files

| File | Path | Topics Covered |
|------|------|---------------|
| LLM Agent Fundamentals | `concepts/llm_agent_fundamentals.md` | Core definition, components (brain, planning, memory, tools, actions, feedback loop), agent loop pseudocode |
| Agent Architecture (Overview) | `concepts/agent_architecture.md` | Architecture diagram, component overview, design decisions (pre-existing) |
| Agent Architecture Patterns | `concepts/agent_architecture_patterns.md` | ReAct, Plan-and-Execute, Reflexion, Multi-Agent (sequential, hierarchical, debate, peer, voting) |
| Agent Frameworks | `concepts/agent_frameworks.md` | LangChain/LangGraph, AutoGen, CrewAI, MetaGPT, Claude SDK, OpenAI SDK, Dify, Coze, comparison matrix |
| Agent Techniques | `concepts/agent_techniques.md` | Function calling, RAG, Chain-of-Thought, Tree-of-Thought, Self-Consistency, Prompt Chaining |
| Agent Memory (Overview) | `concepts/agent_memory.md` | Memory taxonomy, lifecycle, five mechanism families, key architectures (pre-existing) |
| Agent Memory Systems | `concepts/agent_memory_systems.md` | Short-term, long-term, episodic, semantic, procedural, RAG-based memory, reflection, forgetting |

## Paper Notes

| File | Path | Topics Covered |
|------|------|---------------|
| LLM Agent Surveys | `papers/llm/llm_agent_surveys.md` | 10 annotated survey papers, taxonomies, reading order, key benchmarks |

## Topic Cross-Reference

| Topic | Primary File(s) | Supporting File(s) |
|-------|-----------------|-------------------|
| ReAct pattern | `agent_architecture_patterns.md` | `llm_agent_fundamentals.md` |
| Plan-and-Execute | `agent_architecture_patterns.md` | `agent_frameworks.md` (LangGraph) |
| Reflexion | `agent_architecture_patterns.md` | `llm_agent_surveys.md` |
| Multi-agent systems | `agent_architecture_patterns.md` | `agent_frameworks.md`, `llm_agent_surveys.md` |
| Memory systems | `agent_memory_systems.md` | `llm_agent_fundamentals.md`, `llm_agent_surveys.md` |
| RAG | `agent_techniques.md`, `agent_memory_systems.md` | `llm_agent_surveys.md` |
| Function calling / Tool use | `agent_techniques.md` | `llm_agent_fundamentals.md`, `llm_agent_surveys.md` |
| Chain-of-Thought | `agent_techniques.md` | `agent_architecture_patterns.md` |
| Tree-of-Thought | `agent_techniques.md` | `agent_architecture_patterns.md` |
| Framework comparison | `agent_frameworks.md` | -- |
| Agent evaluation / benchmarks | `llm_agent_surveys.md` | -- |

## Key Paper References (by arXiv ID)

| arXiv ID | Paper | Topic |
|----------|-------|-------|
| 2210.03629 | ReAct (Yao et al.) | Reasoning + Acting pattern |
| 2303.11366 | Reflexion (Shinn et al.) | Self-reflection pattern |
| 2309.07864 | Fudan Agent Survey (Xi et al.) | Comprehensive agent survey |
| 2308.11432 | Renmin Agent Survey (Wang et al.) | Agent construction/application |
| 2309.02427 | CoALA (Sumers et al.) | Cognitive architecture framework |
| 2401.03568 | Agent AI (Durante et al.) | Multimodal agent survey |
| 2404.13501 | Memory Survey (Zhang et al.) | Agent memory systems |
| 2402.01680 | Multi-Agent Survey (Guo et al.) | Multi-agent collaboration |
| 2405.17935 | Tool Learning Survey (Qu et al.) | Tool use by LLMs |
| 2312.10997 | RAG Survey (Gao et al.) | Retrieval-augmented generation |
| 2404.11584 | Architecture Landscape (Masterman) | Architecture selection guide |
| 2305.10601 | Tree of Thoughts (Yao et al.) | Tree-structured reasoning |
| 2203.11171 | Chain-of-Thought (Wei et al.) | Step-by-step reasoning |
| 2203.11171 | Self-Consistency (Wang et al.) | Majority voting over CoT |
