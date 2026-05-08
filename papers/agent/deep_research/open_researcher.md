---
title: "OpenResearcher: A Fully Open Pipeline for Long-Horizon Deep Research Trajectory Synthesis"
authors:
  - TIGER-AI-Lab (full author list on paper)
venue: Preprint (under review)
year: 2025
url: https://arxiv.org/abs/2603.20278
tags:
  - deep-research
  - trajectory-synthesis
  - supervised-fine-tuning
  - browser-agent
  - open-source
  - BrowseComp
status: done
---

# OpenResearcher: A Fully Open Pipeline for Long-Horizon Deep Research Trajectory Synthesis

## TL;DR

OpenResearcher is a fully open, reproducible pipeline for synthesizing long-horizon deep research
trajectories at scale. It decouples one-time corpus bootstrapping from multi-turn trajectory
generation, using three browser primitives (search, open, find) over a 15M-document offline corpus.
GPT-OSS-120B serves as the teacher model to generate 97K trajectories (with 100+ tool calls in the
long tail). SFT on a 30B-A3B backbone achieves 54.8% on BrowseComp-Plus (+34.0 over base),
surpassing GPT-4.1, Claude-Opus-4, Gemini-2.5-Pro, and DeepSeek-R1.

## Motivation & Problem

Training deep research agents via online RL (as in DeepResearcher) is expensive and
non-reproducible due to:
1. **Search API dependency**: Live web results change over time; training is not deterministic.
2. **Cost and rate limits**: Massive parallel API calls during RL rollouts are prohibitively
   expensive.
3. **Lack of open data**: No large-scale, high-quality deep research trajectory datasets exist
   for supervised training.

OpenResearcher asks: can we generate high-quality research trajectories entirely offline and use
supervised fine-tuning to achieve competitive or superior performance?

## Method

### Pipeline Overview

```
+-------------------+     +--------------------+     +-------------------+
| Stage 1: Corpus   |     | Stage 2: Trajectory|     | Stage 3: SFT      |
| Bootstrapping     |---->| Synthesis          |---->| Training          |
| (one-time)        |     | (multi-turn)       |     | (30B-A3B)         |
+-------------------+     +--------------------+     +-------------------+
        |                         |                          |
  15M documents             97K trajectories           OpenResearcher
  ~11B tokens               100+ tool calls            -30B-A3B
  Self-built retriever      GPT-OSS-120B teacher       
```

### Stage 1: Corpus Bootstrapping

- Build a **15M-document corpus** (~11 billion tokens) from diverse web sources.
- Construct a **self-built retriever** over this corpus, eliminating dependency on external
  search APIs.
- One-time process: the corpus is indexed and reused across all trajectory synthesis runs.

### Stage 2: Trajectory Synthesis with Browser Primitives

Three explicit browser primitives define the action space:

```
PRIMITIVE: search(query) -> list[document_snippets]
  - Executes semantic search over the 15M-document corpus
  - Returns ranked list of relevant passages

PRIMITIVE: open(url/doc_id) -> full_document_content  
  - Opens and renders the full content of a specific document
  - Enables deep reading beyond snippet-level information

PRIMITIVE: find(pattern, document) -> matched_sections
  - Searches within an opened document for specific information
  - Enables targeted extraction from long documents
```

### Trajectory Generation Process

```
Algorithm: Long-Horizon Trajectory Synthesis
-------------------------------------------------
Input: Question q, corpus C, teacher model M (GPT-OSS-120B)
Output: Trajectory T = [(action_1, obs_1), ..., (action_n, obs_n), answer]

1. Initialize context with question q
2. while not done:
3.   M generates next action: search(q'), open(doc), or find(pattern)
4.   Execute action against offline corpus C
5.   Append (action, observation) to trajectory T
6.   M decides whether to continue or produce final answer
7. Return complete trajectory T with all intermediate steps
```

Key properties of generated trajectories:
- **Long-horizon**: Substantial tail with 100+ sequential tool calls per trajectory.
- **Multi-turn**: Agent iteratively refines its research strategy across many steps.
- **Grounded**: All observations come from the fixed corpus, ensuring reproducibility.

### Stage 3: Supervised Fine-Tuning

- **Backbone**: 30B-A3B (30 billion total parameters, 3 billion active via mixture-of-experts).
- **Training data**: 96K high-quality trajectories from Stage 2.
- **Distillation**: The student model learns the teacher's research strategies through standard
  next-token prediction on the trajectory sequences.

## Key Innovations

1. **Fully offline pipeline**: Eliminates search API dependency by building a self-contained
   corpus with a custom retriever. Trajectories are fully reproducible.

2. **Three-primitive action space**: The search/open/find abstraction is minimal yet sufficient to
   capture the full range of web research behaviors (broad search, deep reading, targeted
   extraction).

3. **Long-horizon trajectories at scale**: 97K trajectories with a substantial fraction containing
   100+ tool calls, far exceeding prior datasets in trajectory length and complexity.

4. **GPT-OSS-120B as teacher**: Uses a strong open-source teacher model with native browser tool
   support, enabling high-quality trajectory generation without proprietary API dependency.

5. **Compact student model**: The 30B-A3B architecture (only 3B active parameters via MoE)
   achieves state-of-the-art results despite its small active parameter count.

## Experimental Setup

### Benchmarks

| Benchmark         | Task Type                        | Difficulty    |
|-------------------|----------------------------------|---------------|
| BrowseComp-Plus   | Complex browsing comprehension   | Very hard     |
| BrowseComp        | Web browsing comprehension       | Hard          |
| GAIA              | General AI assistant tasks       | Medium-Hard   |
| xbench-DeepSearch | Deep search evaluation           | Hard          |

### Baselines

- **Proprietary models**: GPT-4.1, Claude-Opus-4, Gemini-2.5-Pro
- **Open models**: DeepSeek-R1, Tongyi-DeepResearch
- **Base model**: 30B-A3B without SFT (for ablation)

## Results

### BrowseComp-Plus (Primary Benchmark)

| Model                        | Accuracy |
|------------------------------|----------|
| OpenResearcher-30B-A3B       | **54.8%**|
| GPT-4.1                      | < 54.8%  |
| Claude-Opus-4                 | < 54.8%  |
| Gemini-2.5-Pro               | < 54.8%  |
| DeepSeek-R1                  | < 54.8%  |
| Tongyi-DeepResearch          | < 54.8%  |
| 30B-A3B base (no SFT)       | 20.8%    |

- **+34.0 absolute improvement** over the base model (20.8% -> 54.8%).
- Surpasses all listed proprietary and open-source baselines.

### Cross-Benchmark Generalization

- Competitive performance on BrowseComp, GAIA, and xbench-DeepSearch.
- Demonstrates that offline trajectory synthesis transfers to diverse evaluation settings.

## Limitations

1. **Corpus freshness**: The offline 15M-document corpus becomes stale over time; cannot handle
   queries about recent events.
2. **Teacher model dependency**: Quality of trajectories is bounded by GPT-OSS-120B's capability.
3. **SFT ceiling**: Supervised fine-tuning may not discover novel strategies beyond those
   demonstrated by the teacher (no exploration as in RL).
4. **Evaluation scope**: Primarily evaluated on browsing comprehension; less tested on open-ended
   report generation tasks.
5. **MoE complexity**: The 30B-A3B architecture adds engineering complexity for deployment despite
   having only 3B active parameters.

## Follow-up Work

- Potential integration with RL fine-tuning (SFT + RL hybrid as in DeepResearcher).
- Extension to multimodal research trajectories (images, tables, charts).
- Dynamic corpus updating for handling time-sensitive queries.
- Scaling the trajectory synthesis to even longer horizons (1000+ tool calls).

## Key Takeaways

1. Offline trajectory synthesis is a viable alternative to expensive online RL for training deep
   research agents, achieving state-of-the-art results through SFT alone.
2. The search/open/find primitive set is a powerful minimal abstraction for web research behavior.
3. Long-horizon trajectories (100+ tool calls) are critical for teaching agents to handle complex,
   multi-step research tasks.
4. A compact MoE architecture (3B active parameters) can match or exceed much larger dense models
   when trained on high-quality research trajectories.
5. Full open-sourcing of dataset, model, and training recipe enables reproducible research on
   deep research agents.
