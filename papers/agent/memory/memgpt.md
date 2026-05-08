---
title: "MemGPT: Towards LLMs as Operating Systems"
authors: Charles Packer, Sarah Wooders, Kevin Lin, et al.
venue: ICLR 2024 (Spotlight)
year: 2023
url: https://arxiv.org/abs/2310.08560
tags: [memory, context-management, os-inspired, architecture]
status: done
---

## TL;DR

OS-inspired memory management for LLM agents. Implements virtual context management with main context (RAM) and external storage (disk), enabling unbounded conversation and document analysis.

## Method

- **Main context** = working memory (fits in LLM context window)
- **External storage** = long-term memory (database)
- **Memory controller** = decides what to page in/out
- Agent can explicitly manage its own memory via function calls (push/pop/search)
- Inspired by operating system virtual memory and paging

## Results

- Enables multi-session conversations without context limit
- Document analysis over texts far exceeding context window
- Agent learns and recalls information across sessions

## Why It Matters

Pioneered the concept of LLMs managing their own memory hierarchies. Later evolved into the **Letta** framework. Influenced how modern agents handle long-term state and context management.

## Related

- Evolved into: Letta (commercial framework)
- See: `concepts/agent_memory.md`
