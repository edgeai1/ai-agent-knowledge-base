---
title: "SWE-Agent: Agent-Computer Interfaces Are All You Need"
authors: John Yang, Carlos E. Jimenez, et al.
venue: arXiv (Princeton)
year: 2024
url: https://arxiv.org/abs/2405.15793
tags: [coding, swe-bench, agent-computer-interface, tool-design]
status: done
---

## TL;DR

Introduces the Agent-Computer Interface (ACI) concept -- custom-designed interfaces that make it easier for LLMs to interact with code repositories. Key insight: how you present the environment to the agent matters enormously.

## Method

- Custom file viewer, search, and edit tools designed for LLM ergonomics
- Linting and syntax checking on edits
- Compact, structured output formats
- mini-SWE-agent (~100 lines Python) achieves >74% on SWE-bench Verified

## Results (2026 Update)

SWE-bench Verified leaderboard (March 2026):
- Claude Opus 4.5: 80.9%
- Claude Opus 4.6: 80.8%
- Gemini 3.1 Pro: 80.6%
- OpenHands (Claude 4): 72%
- mini-SWE-agent: >74%

On truly novel issues, agents solve ~18-20% -- far below curated benchmarks.

## Why It Matters

Demonstrated that the interface design between agent and environment is as important as the model itself. Spawned research into better ACIs for various domains.
