---
title: "Reflexion: Language Agents with Verbal Reinforcement Learning"
authors: Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan, Shunyu Yao
venue: NeurIPS 2023
year: 2023
url: https://arxiv.org/abs/2303.11366
tags: [self-improvement, reflection, memory, reinforcement-learning, foundational]
status: done
---

## TL;DR

LLM agent reflects on its failures, stores verbal self-critiques in memory, and uses them to improve on subsequent attempts -- "verbal reinforcement learning" without weight updates.

## Method

The Reflexion loop:
1. Agent attempts a task (coding, decision-making, reasoning)
2. Receives binary/scalar feedback on success/failure
3. Self-reflection module generates natural-language analysis of what went wrong
4. Reflection stored in episodic memory buffer
5. On next attempt, agent is prompted with previous reflections to avoid past mistakes

## Results

- HumanEval: 91% pass@1 (vs GPT-4's ~67% at the time)
- AlfWorld: 97% success (vs. ReAct's 75%)
- HotPotQA: significant improvement over baselines

## Why It Matters

Foundational paper for self-improving agents. Showed agents can learn from trial and error without weight updates, using natural language as the medium for storing and applying experience. Underpins iterative coding agents, self-debugging systems.

## Related

- Related: Self-Refine (Madaan et al., 2023) -- iterative output refinement within single episode
- See: `concepts/agent_memory.md`
