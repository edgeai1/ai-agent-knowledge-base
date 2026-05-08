---
title: "MRKL Systems: A Modular, Neuro-Symbolic Architecture"
authors: Ehud Karpas, Omri Abend, Yonatan Belinkov, et al.
venue: arXiv (AI21 Labs)
year: 2022
url: https://arxiv.org/abs/2205.00445
tags: [architecture, tool-routing, neuro-symbolic, foundational]
status: done
---

## TL;DR

Proposed the MRKL ("miracle") architecture -- one of the earliest formal frameworks for combining an LLM with a set of external expert modules (calculators, databases, APIs).

## Method

- LLM as central "router" interpreting natural language requests
- Collection of specialized "expert" modules (neural and symbolic)
- Router decides which expert(s) to invoke, generates input, integrates results

## Why It Matters

Conceptual precursor to ReAct, LangChain, and the entire tool-using agent paradigm. Formally articulated that LLMs should route sub-problems to specialized systems rather than do everything themselves. LangChain's original "MRKL agent" is directly named after this paper.

## Related

- Successors: ReAct, Toolformer, HuggingGPT, all modern agent frameworks
