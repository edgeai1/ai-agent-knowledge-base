---
title: "Generative Agents: Interactive Simulacra of Human Behavior"
authors: Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, Michael S. Bernstein
venue: UIST 2023 (Best Paper)
year: 2023
url: https://arxiv.org/abs/2304.03442
tags: [multi-agent, memory, reflection, simulation, foundational]
status: done
---

## TL;DR

25 LLM-powered agents in a sandbox world ("Smallville") exhibit emergent social behaviors -- forming relationships, spreading information, and autonomously organizing events -- without explicit scripting.

## Method

Each agent has a cognitive architecture with three components:

1. **Memory Stream**: comprehensive log of experiences in natural language
2. **Retrieval**: surface relevant memories based on recency + importance + relevance
3. **Reflection**: periodically synthesize memories into higher-level insights
4. **Planning & Reacting**: generate and update daily plans, react to environment

All powered by GPT-3.5/4 as backbone.

## Key Emergent Behaviors

- Agents autonomously organized a Valentine's Day party
- Information spread through social networks
- Agents formed opinions and relationships based on interactions

## Why It Matters

Foundational for multi-agent simulation and the design of believable AI agents. The **memory-retrieval-reflection** architecture became a reference design for agent memory systems. Proved that complex social behavior can emerge from simple LLM-based architectures.

## Related

- Memory architecture influenced: MemGPT, CoALA, modern agent memory systems
- See: `concepts/agent_memory.md`
