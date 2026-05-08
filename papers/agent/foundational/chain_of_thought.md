---
title: "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models"
authors: Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, et al.
venue: NeurIPS 2022
year: 2022
url: https://arxiv.org/abs/2201.11903
tags: [reasoning, prompting, chain-of-thought, foundational]
status: done
---

## TL;DR

Prompting LLMs to produce intermediate reasoning steps dramatically improves performance on arithmetic, commonsense, and symbolic reasoning tasks, unlocking latent reasoning abilities.

## Problem

LLMs struggle with multi-step reasoning when asked to produce answers directly.

## Method

Augment few-shot prompting by including step-by-step reasoning in exemplars. Instead of input-output pairs, each exemplar shows the intermediate reasoning process. At inference, the model generates its own reasoning chain before arriving at the answer.

- No fine-tuning required -- works purely through prompting
- **Emergent property**: CoT only helps models above ~100B parameters

## Results

- Dramatic improvements on GSM8K (math), CommonsenseQA, StrategyQA
- PaLM 540B + CoT matches or exceeds fine-tuned models

## Why It Matters for Agents

Chain-of-thought is the intellectual foundation for the "Thought" step in ReAct and all subsequent agent reasoning. Without CoT, agents cannot plan or reason about their actions. Every modern agent relies on some form of CoT for planning and decision-making.

## Variants

- **Zero-shot CoT**: "Let's think step by step" (Kojima et al., 2022)
- **Self-Consistency**: Sample multiple CoT paths, vote on answer (Wang et al., 2022)
- **Tree of Thoughts**: Branch into multiple reasoning paths with backtracking
