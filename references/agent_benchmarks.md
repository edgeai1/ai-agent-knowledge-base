---
title: Agent Benchmarks & Evaluation
tags: [benchmark, evaluation, leaderboard, reference]
related: []
---

## Coding Agent Benchmarks

### SWE-bench / SWE-bench Verified
- **Source**: Princeton (ICLR 2024)
- **URL**: https://www.swebench.com
- **Task**: resolve real GitHub issues from popular Python repos
- **Size**: 2,294 tasks (full), ~500 (Verified, human-confirmed solvable)
- **2026 SOTA**: Claude Opus 4.5 at 80.9% (Verified)
- **Reality check**: on truly novel issues, agents solve ~18-20%

### Aider Polyglot Benchmark
- **Source**: aider (Paul Gauthier)
- **Task**: multi-language code editing tasks
- **URL**: https://aider.chat/docs/leaderboards/

## Web / GUI Agent Benchmarks

### WebArena
- **Source**: CMU (ICLR 2024)
- **URL**: https://arxiv.org/abs/2307.13854
- **Task**: 812 complex tasks on self-hosted realistic web environments (Reddit, GitLab, shopping, Wikipedia)

### OSWorld
- **Source**: HKU/Edinburgh (NeurIPS 2024)
- **URL**: https://arxiv.org/abs/2404.07972
- **Task**: 369 tasks in real Ubuntu/Windows/macOS environments
- **Best agents scored under 12% at launch**

### VisualWebArena
- **Source**: CMU (ACL 2024)
- **Task**: visually-grounded web tasks requiring multimodal understanding

### ClawBench (2026)
- **Task**: 153 tasks across 144 live production websites
- **Focus**: real-world browser agent evaluation

## General Agent Benchmarks

### AgentBench
- **Source**: Tsinghua (ICLR 2024)
- **URL**: https://arxiv.org/abs/2308.03688
- **Task**: 8 environments (OS, DB, knowledge graph, card game, web shopping, etc.)

### GAIA
- **Source**: Meta AI (ICLR 2024)
- **URL**: https://arxiv.org/abs/2311.12983
- **Task**: real-world assistant tasks requiring multi-step reasoning + tool use
- **Human: ~92%, Best AI: ~15% initially**

### tau-bench
- **Source**: Sierra (2024)
- **Task**: customer service tasks requiring tool use and policy adherence

### MLE-bench
- **Source**: OpenAI (2024)
- **Task**: Kaggle-style ML engineering tasks end-to-end

## 2026 Benchmarks

### DevOps-Gym
- **Task**: 700+ real-world DevOps tasks
- **Focus**: practical infrastructure and deployment

### LUMINA
- **Task**: long-horizon understanding for multi-turn interactive agents

### MemBench / MemoryAgentBench
- **Task**: agent memory evaluation (factual vs. reflective, retrieval, forgetting)
