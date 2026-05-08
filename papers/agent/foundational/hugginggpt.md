---
title: "HuggingGPT: Solving AI Tasks with ChatGPT and Its Friends in Hugging Face"
authors: Yongliang Shen, Kaitao Song, Xu Tan, et al.
venue: NeurIPS 2023
year: 2023
url: https://arxiv.org/abs/2303.17580
tags: [orchestration, multi-model, task-planning, foundational]
status: done
---

## TL;DR

Uses ChatGPT as a controller to orchestrate calls to many specialist AI models hosted on Hugging Face, decomposing complex requests into sub-tasks handled by expert models.

## Method

Four-stage pipeline:
1. **Task Planning**: LLM parses request into dependency-aware task graph
2. **Model Selection**: LLM chooses best expert model per task from Hugging Face
3. **Task Execution**: selected models run (image gen, detection, speech, etc.)
4. **Response Generation**: LLM aggregates all results into coherent response

## Why It Matters

Demonstrated the vision of an LLM as general-purpose "brain" coordinating specialist models. This "controller + tools" architecture became a dominant paradigm. Related Microsoft paper (TaskMatrix.AI) generalized this to millions of APIs.
