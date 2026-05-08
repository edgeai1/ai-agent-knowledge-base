---
title: "Tree of Thoughts: Deliberate Problem Solving with Large Language Models"
authors: Shunyu Yao, Dian Yu, Jeffrey Zhao, et al.
venue: NeurIPS 2023
year: 2023
url: https://arxiv.org/abs/2305.10601
tags: [reasoning, planning, search, tree-search, foundational]
status: done
---

## TL;DR

Generalizes chain-of-thought from a single linear chain into a tree-structured exploration of multiple reasoning paths, enabling LLMs to explore, evaluate, and backtrack.

## Method

1. **Thought decomposition**: break problem into intermediate "thought" steps
2. **Thought generation**: at each step, propose multiple candidate next thoughts (branching)
3. **State evaluation**: LLM evaluates how promising each partial solution is
4. **Search algorithm**: BFS or DFS to navigate the tree, prune unpromising branches

The LLM serves as both generator and evaluator of thoughts.

## Results

- Game of 24: 74% (vs. 4% for CoT)
- Creative Writing: significant quality improvement
- Mini Crosswords: much higher solve rate

## Why It Matters

Enables agents to consider multiple plans, evaluate them, and choose the best path. Bridges classical AI search methods with modern LLM capabilities. Critical for complex planning and decision-making tasks.
