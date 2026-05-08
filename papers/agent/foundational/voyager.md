---
title: "Voyager: An Open-Ended Embodied Agent with Large Language Models"
authors: Guanzhi Wang, Yuqi Xie, Yunfan Jiang, et al.
venue: NeurIPS 2023 (Spotlight)
year: 2023
url: https://arxiv.org/abs/2305.16291
tags: [embodied, lifelong-learning, skill-library, minecraft, foundational]
status: done
---

## TL;DR

First LLM-powered lifelong learning agent in Minecraft that continuously explores, acquires new skills, and makes discoveries without human intervention.

## Method

Three key components:
1. **Automatic Curriculum**: LLM proposes increasingly complex goals based on current state and skill level
2. **Skill Library**: growing repository of executable code (JavaScript for Minecraft) that the agent writes, verifies, and stores; indexed by description embeddings for retrieval
3. **Iterative Prompting**: LLM writes code, executes in environment, receives feedback, iteratively refines until success

Uses GPT-4 as backbone.

## Results

- 3.3x more unique items obtained than baselines
- Traverses 2.3x longer distances
- Unlocks key tech tree milestones faster than all baselines
- Skills compose: later skills reuse earlier ones

## Why It Matters

Landmark paper for embodied LLM agents and continual skill acquisition. The concept of agents writing, testing, and storing reusable code as a skill library has influenced agent architectures well beyond game environments.

## Related

- Skill library concept extends to: coding agents, tool creation
- See: `concepts/agent_memory.md` (procedural memory)
