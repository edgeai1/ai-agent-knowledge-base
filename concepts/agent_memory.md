---
title: Agent Memory Systems
tags: [memory, rag, context, core-concept]
related: [agent_architecture, react_pattern]
---

## Definition

Agent memory enables LLM agents to store, manage, and retrieve information beyond the immediate context window, supporting long-term learning and personalization.

## Memory Taxonomy (2026 Survey)

### By Temporal Scope
- **Short-term / Working Memory**: current context window contents, recent observations
- **Long-term Memory**: persisted knowledge across sessions

### By Representation
- **Token-level**: raw text stored and retrieved as tokens
- **Parametric**: knowledge encoded in model weights (via fine-tuning)
- **Latent**: compressed representations (embeddings, summaries)

### By Function
- **Factual Memory**: facts, knowledge, definitions
- **Experiential Memory**: past interactions, outcomes, reflections
- **Working Memory**: active task state, in-progress reasoning

## Memory Lifecycle: Write-Manage-Read

```
Write (Formation)    Manage (Evolution)    Read (Retrieval)
─────────────────    ──────────────────    ────────────────
Store experience  -> Update/consolidate -> Query relevant
                     Forget/compress       memories
                     Reflect/abstract      Score by relevance
```

## Five Mechanism Families

### 1. Context-Resident Compression
Compress information to fit more into context window.
- Summarization of past interactions
- Sliding window with compression

### 2. Retrieval-Augmented Stores (RAG)
External vector database for storing and retrieving memories.
- Embed experiences, retrieve by similarity
- Foundation of most production agent memory systems

### 3. Reflective Self-Improvement
Agent generates higher-level insights from raw experiences.
- Generative Agents: periodic reflection on memory stream
- Reflexion: verbal self-critique from failures

### 4. Hierarchical Virtual Context
OS-inspired memory management (MemGPT/Letta).
- Main context = RAM, external storage = disk
- Agent manages its own paging

### 5. Policy-Learned Management
Learned policies for when to read/write/forget.
- FadeMem: biologically-inspired forgetting
- A-Mem: autonomous memory management

## Key Architectures

| System | Approach | Key Innovation |
|--------|----------|----------------|
| Generative Agents | Memory stream + reflection | Recency-importance-relevance retrieval |
| MemGPT / Letta | Virtual memory management | Agent controls its own paging |
| Reflexion | Episodic failure memory | Verbal reinforcement learning |
| Voyager | Skill library (code) | Procedural memory as executable code |
| CoALA | Cognitive architecture | Unified framework from cognitive science |

## Evaluation Benchmarks (2026)

- **MemBench**: factual vs. reflective memory
- **MemoryAgentBench**: retrieval, learning, forgetting
- **LOCOMO**: long-term conversational memory
