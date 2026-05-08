---
title: Multi-Agent Systems
tags: [multi-agent, collaboration, orchestration, core-concept]
related: [agent_architecture, mcp_a2a_protocols]
---

## Definition

Multiple LLM-powered agents collaborate to solve tasks that are too complex for a single agent, each with specialized roles, knowledge, or capabilities.

## Collaboration Patterns

### 1. Sequential / Pipeline
Agents work in sequence, each handling a phase.
```
Agent A (plan) -> Agent B (code) -> Agent C (test) -> Agent D (review)
```
Example: ChatDev, MetaGPT

### 2. Hierarchical / Delegation
A manager agent delegates to worker agents.
```
      Manager
      /  |  \
    A    B    C  (specialists)
```
Example: CrewAI, AutoGen group chat with manager

### 3. Debate / Discussion
Agents discuss and critique each other's outputs.
```
Agent A <-> Agent B  (argue until consensus)
```
Example: CAMEL, multi-agent debate

### 4. Cooperative / Swarm
Agents coordinate as peers without explicit hierarchy.
```
A <-> B <-> C  (mesh communication)
```
Example: OpenAI Swarm

## Key Frameworks (2026)

| Framework | Pattern | Strengths |
|-----------|---------|-----------|
| **LangGraph** | Graph-based state machine | Flexible, production-ready, LangChain ecosystem |
| **AutoGen** | Conversation-based | Flexible patterns, Microsoft backing |
| **CrewAI** | Role-based delegation | Simple API, production-focused |
| **MetaGPT** | SOP-encoded collaboration | Structured workflows, artifact-driven |
| **Claude Agent SDK** | Programmable agent loop | Same engine as Claude Code, MCP native |
| **OpenAI Agents SDK** | Handoff-based | Simple agent-to-agent delegation |
| **Google ADK 1.0** | A2A-native | Multi-agent standard, Gemini integration |

## Challenges

1. **Coordination overhead**: more agents != better results
2. **Error propagation**: mistakes cascade through agent chains
3. **Cost**: each agent call = API cost
4. **Evaluation**: harder to benchmark multi-agent systems
5. **Scalability**: diminishing returns as agent count grows

## 2026 Trends

- **CORAL**: self-evolving multi-agent systems with shared persistent memory (3-10x improvement over fixed baselines)
- **DyTopo**: dynamic topology routing -- semantic-based agent connection rewiring
- **Standardization**: MCP + A2A protocols converging for full-stack multi-agent interoperability
