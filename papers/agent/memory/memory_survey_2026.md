---
title: "Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers"
authors: Multiple (survey)
venue: arXiv
year: 2026
url: https://arxiv.org/abs/2603.07670
tags: [memory, survey, evaluation, 2026]
status: done
---

## TL;DR

Comprehensive 2026 survey covering how memory is designed, implemented, and evaluated in modern LLM-based agents, spanning work from 2022 through early 2026.

## Key Taxonomy

Formalizes agent memory as a **write-manage-read loop** with three-dimensional taxonomy:

1. **Temporal scope**: short-term vs. long-term
2. **Representational substrate**: token-level, parametric, latent
3. **Control policy**: how memory is managed

Five mechanism families:
- Context-resident compression
- Retrieval-augmented stores
- Reflective self-improvement
- Hierarchical virtual context
- Policy-learned management

## Notable 2026 Papers Covered

- **FadeMem**: biologically-inspired forgetting for efficient agent memory
- **ShardMemo**: masked MoE routing for sharded agentic LLM memory
- **E-mem**: multi-agent episodic context reconstruction
- **A-Mem**: agentic memory with autonomous management
- **MemMachine**: ground-truth-preserving memory system

## Evaluation Benchmarks

- **MemBench**: factual vs. reflective memory testing
- **MemoryAgentBench**: cognitive-science-grounded evaluation (retrieval, test-time learning, selective forgetting)
- **LOCOMO**: long-term conversational memory benchmark

## Related

- Also see: "Memory in the Age of AI Agents" (arXiv:2512.13564)
- "A Survey on the Security of Long-Term Memory in LLM Agents" (2026)
