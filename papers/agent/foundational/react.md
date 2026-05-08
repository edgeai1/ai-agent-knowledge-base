---
title: "ReAct: Synergizing Reasoning and Acting in Language Models"
authors: Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, Yuan Cao
venue: ICLR 2023
year: 2022
url: https://arxiv.org/abs/2210.03629
tags: [agent, reasoning, acting, react, foundational]
status: done
---

## TL;DR

Interleave chain-of-thought reasoning traces with action steps, forming the Thought-Action-Observation loop that became the foundation of nearly all modern LLM agents.

## Problem

LLMs either reason (CoT) or act (tool use) but not both simultaneously. Pure reasoning hallucinates; pure acting lacks planning.

## Method

The model generates a sequence of **Thought-Action-Observation** triplets:
- **Thought**: free-form reasoning trace (plan, track progress, handle exceptions)
- **Action**: call to external tool or API (e.g., Wikipedia search)
- **Observation**: environment's response

Loop continues until the model reaches a final answer. Implemented via few-shot prompting (PaLM, GPT-series).

```
Thought: I need to find the population of Tokyo.
Action: Search["population of Tokyo"]
Observation: Tokyo has a population of 13.96 million...
Thought: Now I have the answer.
Action: Finish["13.96 million"]
```

## Results

- Outperforms CoT-only and Act-only baselines on HotpotQA, FEVER, ALFWorld, WebShop
- Reasoning helps the model plan and handle exceptions
- Actions ground the model in real information, reducing hallucination

## Why It Matters

ReAct is arguably the single most influential agent framework. Nearly every modern agent system (LangChain, AutoGPT, Claude Agent SDK) descends from this Thought-Action-Observation loop pattern. It proved LLMs can be grounded decision-makers, not just text generators.

## Related

- Builds on: `concepts/chain_of_thought.md`, `papers/agent/foundational/mrkl.md`
- Extended by: Reflexion, Tree of Thoughts, AutoGPT, all modern agent frameworks
