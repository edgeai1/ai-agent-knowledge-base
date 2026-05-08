---
title: AutoGPT
authors: Toran Bruce Richards (Significant Gravitas)
venue: GitHub (open-source)
year: 2023
url: https://github.com/Significant-Gravitas/AutoGPT
tags: [autonomous, goal-driven, viral, foundational]
status: done
---

## TL;DR

First widely-viral fully autonomous LLM agent. Given a high-level goal, it autonomously plans, executes actions (web, files, code), manages memory, and iterates without human intervention.

## Method

Autonomous loop with GPT-4:
1. System prompt with persona and goals
2. Short-term memory (conversation context)
3. Long-term memory (vector DB for storing/retrieving experience)
4. Tool capabilities (web search, file I/O, code execution)
5. Self-prompting: agent critiques its own plan, decides next action

Loop: Think -> Plan -> Act -> Observe -> Reflect -> repeat

## Why It Matters

Not academically rigorous, but catalyzed massive public and research interest in autonomous agents. Exposed both the potential and limitations (runaway loops, high cost, unreliability) of fully autonomous LLM systems. Motivated research into better planning, memory, and self-correction.

## Limitations

- Frequent loops and dead ends
- Extremely high API costs
- Unreliable goal decomposition
- Difficulty knowing when to stop
