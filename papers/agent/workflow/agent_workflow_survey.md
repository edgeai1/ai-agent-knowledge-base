---
title: "A Survey on Agent Workflow -- Status and Future"
authors:
  - Chaojia Yu
  - Zihan Cheng
  - Hanwen Cui
  - Yishuo Gao
  - Zexu Luo
  - Yijin Wang
  - Hangbin Zheng
  - Yong Zhao
venue: ICAIBD 2025 (8th International Conference on Artificial Intelligence and Big Data)
year: 2025
url: https://arxiv.org/abs/2508.01186
tags:
  - survey
  - agent-workflow
  - orchestration
  - multi-agent
  - LLM-agents
  - workflow-patterns
status: done
---

## TL;DR

A comprehensive survey of agent workflow systems covering 20+ representative frameworks
(academic and industrial). The paper proposes a two-dimensional classification along
functional capabilities (planning, multi-agent collaboration, API integration) and
architectural features (agent roles, orchestration flows, specification languages).
It also introduces the HAWK framework -- a hierarchical five-layer architecture
(User, Workflow, Operator, Agent, Resource) with sixteen standardized interfaces for
end-to-end agent workflow orchestration.

## Motivation & Problem

As LLM-based autonomous agents grow in complexity, ad-hoc implementations become
unmaintainable. The field lacks:

1. **Systematic taxonomy**: No unified framework for comparing the growing zoo of agent
   workflow systems (LangChain, AutoGen, CrewAI, MetaGPT, etc.).
2. **Architectural clarity**: Systems conflate agent logic, orchestration, and resource
   management, making them hard to extend and debug.
3. **Standardization**: Each framework defines its own abstractions, making
   interoperability and migration difficult.
4. **Security considerations**: As workflows gain autonomy, the security implications of
   tool access, data flow, and agent delegation remain under-explored.

## Method

### Two-Dimensional Classification Framework

The survey organizes existing systems along two orthogonal axes:

```
+------------------------------------------------------------------+
|                    CLASSIFICATION FRAMEWORK                       |
+------------------------------------------------------------------+
|                                                                   |
|  Dimension 1: FUNCTIONAL CAPABILITIES                            |
|  +------------------------------------------------------------+  |
|  | - Planning (task decomposition, goal-driven)                |  |
|  | - Multi-Agent Collaboration (role assignment, delegation)   |  |
|  | - External API / Tool Integration                           |  |
|  | - Memory Management (short-term, long-term, shared)         |  |
|  | - Human-in-the-Loop (approval, feedback, override)          |  |
|  +------------------------------------------------------------+  |
|                                                                   |
|  Dimension 2: ARCHITECTURAL FEATURES                             |
|  +------------------------------------------------------------+  |
|  | - Agent Roles (single, specialized, hierarchical)           |  |
|  | - Orchestration Flows (sequential, parallel, DAG, cyclic)   |  |
|  | - Specification Languages (code, YAML, visual, natural lang)|  |
|  | - Control Type (centralized, decentralized, hybrid)         |  |
|  | - Deployment Model (local, cloud, hybrid)                   |  |
|  | - Memory Model (per-agent, shared, hierarchical)            |  |
|  +------------------------------------------------------------+  |
+------------------------------------------------------------------+
```

### Systems Compared (20+ frameworks)

The survey covers major frameworks across both academic and industrial settings:

| Category   | Systems                                                     |
|-----------|-------------------------------------------------------------|
| Academic  | MetaGPT, CAMEL, AgentVerse, AutoGen (MSR), ChatDev         |
| Industry  | LangChain/LangGraph, CrewAI, LlamaIndex, Semantic Kernel   |
| Hybrid    | AutoGPT, BabyAGI, OpenDevin, Dify, Coze                    |

Each system is evaluated across dimensions including:
- Planning approach (reactive vs. deliberative)
- Control type (centralized vs. decentralized)
- Configuration method (programmatic vs. declarative)
- Architecture (monolithic vs. modular)
- Specification language (Python, YAML, visual, etc.)
- Deployment model (local, cloud, serverless)
- Memory model (ephemeral, persistent, shared)

### HAWK: Hierarchical Agent Workflow Framework

The survey proposes HAWK as a reference architecture with five layers:

```
+================================================================+
|                       USER LAYER                                |
|  - Task submission interface                                    |
|  - Task translation and parsing                                 |
|  - User preference management                                   |
+================================================================+
                              |
+================================================================+
|                     WORKFLOW LAYER                               |
|  - Workflow definition and management                           |
|  - Adaptive scheduling and optimization                         |
|  - Real-time feedback and dynamic strategy adjustment           |
|  - DAG/graph-based workflow representation                      |
+================================================================+
                              |
+================================================================+
|                     OPERATOR LAYER                               |
|  - Atomic operation definitions                                 |
|  - Composition rules for complex operations                     |
|  - Pre/post-condition checking                                  |
+================================================================+
                              |
+================================================================+
|                      AGENT LAYER                                 |
|  - Agent instantiation and lifecycle                            |
|  - Role assignment and capability matching                      |
|  - Inter-agent communication protocols                          |
+================================================================+
                              |
+================================================================+
|                     RESOURCE LAYER                                |
|  - LLM provider abstraction                                    |
|  - Tool/API registry and access control                         |
|  - Memory stores (vector DBs, KV stores)                        |
|  - External service connectors                                  |
+================================================================+

Supported by 16 standardized interfaces across layers
```

### Workflow Pattern Taxonomy

The survey identifies several fundamental orchestration patterns:

1. **Sequential**: Linear chain of agent actions (A -> B -> C)
2. **Parallel**: Concurrent execution with synchronization (fan-out/fan-in)
3. **DAG (Directed Acyclic Graph)**: Complex dependencies without cycles
4. **Cyclic/Iterative**: Feedback loops for refinement (review-revise cycles)
5. **Hierarchical**: Manager agents delegate to worker agents
6. **Event-Driven**: Agents react to asynchronous events/triggers
7. **Conversational**: Multi-turn dialogue-based coordination

## Key Innovations

1. **Dual-axis taxonomy**: First survey to systematically classify agent workflow systems
   along both functional capabilities and architectural features simultaneously.

2. **HAWK reference architecture**: Proposes a concrete five-layer architecture with
   sixteen standardized interfaces, providing a blueprint for interoperable systems.

3. **Security analysis**: Explicitly addresses workflow security concerns including
   tool access control, data leakage through agent communication, and prompt injection
   attacks in multi-agent settings.

4. **Practical comparison**: Side-by-side evaluation of 20+ systems enables practitioners
   to make informed framework selection decisions.

## Experimental Setup

As a survey paper, the evaluation is primarily qualitative and comparative rather than
benchmark-driven. The methodology includes:

- **System selection**: 20+ representative frameworks from 2023-2025
- **Feature extraction**: Systematic analysis of documentation, source code, and
  published papers for each system
- **Comparison dimensions**: 7+ architectural and functional dimensions per system
- **Gap analysis**: Identification of under-served areas and open problems

## Results

### Key Comparative Findings

| Feature              | LangChain | AutoGen | CrewAI | MetaGPT | LlamaIndex |
|---------------------|-----------|---------|--------|---------|------------|
| Multi-agent          | Via Graph | Native  | Native | Native  | Limited    |
| Workflow spec        | Code      | Code    | YAML   | Code    | Code       |
| Planning             | Manual    | Auto    | Auto   | SOP     | Manual     |
| Memory               | Modular   | Shared  | Shared | Role    | Index      |
| Human-in-loop        | Yes       | Yes     | Yes    | Limited | Limited    |

### Identified Trends

1. **Convergence toward graph-based orchestration**: Most modern frameworks adopt
   DAG or state-graph representations for workflow definition.
2. **Declarative specification**: Movement from purely programmatic APIs toward
   YAML/JSON-based workflow definitions for non-developer accessibility.
3. **Role-based multi-agent**: Dominant paradigm assigns specialized roles to agents
   (coder, reviewer, planner) rather than using homogeneous agents.
4. **Tool ecosystem integration**: All major frameworks now support MCP/function-calling
   for external tool access.

## Limitations

1. **Snapshot bias**: The survey captures the state of a rapidly evolving field;
   some systems may have changed significantly since analysis.
2. **Quantitative gaps**: Limited empirical benchmarking across systems on standardized
   tasks; comparisons are primarily feature-based.
3. **HAWK validation**: The proposed reference architecture is not empirically validated
   against the surveyed systems through implementation or benchmarks.
4. **Industrial coverage**: Some proprietary/commercial systems (e.g., internal tools at
   major tech companies) are underrepresented due to access constraints.
5. **Evaluation methodology**: No standardized benchmark exists for comparing agent
   workflow systems end-to-end, making rigorous comparison difficult.

## Key Takeaways

1. Agent workflow systems are converging on common patterns: graph-based orchestration,
   role-based multi-agent coordination, and modular tool integration. Understanding these
   patterns helps practitioners select and design systems effectively.

2. The five-layer HAWK architecture (User, Workflow, Operator, Agent, Resource) provides
   a useful mental model for separating concerns in agent systems, even if not adopted
   as a literal implementation standard.

3. Standardization is the most critical open problem. The lack of common interfaces and
   specification languages across frameworks creates lock-in and limits interoperability.

4. Security in agent workflows remains under-explored. As agents gain tool access and
   autonomy, the attack surface (prompt injection, data exfiltration, unauthorized
   actions) grows significantly.

5. The gap between academic research frameworks and production-grade industrial systems
   is narrowing, but differences in reliability, observability, and error handling
   remain substantial.

6. Multimodal integration (vision, audio, structured data) in workflows is an emerging
   frontier that most current frameworks handle poorly or not at all.
