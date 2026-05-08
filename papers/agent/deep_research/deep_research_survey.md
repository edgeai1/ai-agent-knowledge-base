---
title: "Deep Research: A Survey of Autonomous Research Agents"
authors:
  - Wenlin Zhang
  - Xiaopeng Li
  - Yingyi Zhang
  - Pengyue Jia
  - Yichao Wang
  - Huifeng Guo
  - Yong Liu
  - Xiangyu Zhao
venue: arXiv preprint
year: 2025
url: https://arxiv.org/abs/2508.12752
tags:
  - survey
  - deep-research
  - autonomous-agents
  - taxonomy
  - web-exploration
  - report-generation
status: done
---

# Deep Research: A Survey of Autonomous Research Agents

## TL;DR

This survey provides the first systematic overview of autonomous deep research agents, organized
around a four-stage pipeline: planning, question developing, web exploration, and report
generation. Rather than enumerating full-system pipelines, it dissects the modular competencies
underlying deep research, analyzing technical bottlenecks, coordination challenges, and emerging
trends (reasoning-driven retrieval, structured report generation, self-evolving agents). It covers
major systems including OpenAI Deep Research, Gemini Deep Research, Perplexity Deep Research,
DeepResearcher, WebThinker, STORM, and others.

## Motivation & Problem

The rapid development of LLM-powered research agents has produced a fragmented landscape:

1. **Proliferation of systems**: OpenAI, Google, Perplexity, and academic groups have all released
   deep research agents, but no unified framework exists for understanding their architectures.
2. **Lack of systematic comparison**: Existing work focuses on individual systems without cross-
   cutting analysis of shared modules and design decisions.
3. **Unclear evaluation standards**: No consensus on how to benchmark deep research capabilities;
   different systems use different metrics and benchmarks.
4. **Rapid evolution**: The field moves faster than traditional survey cycles; a modular analysis
   is needed to accommodate new systems.

## Method (Survey Methodology)

The survey adopts a modular decomposition approach:

```
+------------------------------------------------------------------+
|                Deep Research Pipeline (4 Stages)                  |
|                                                                    |
|  +------------+  +---------------+  +--------------+  +----------+|
|  | Stage 1:   |  | Stage 2:      |  | Stage 3:     |  | Stage 4: ||
|  | Planning   |->| Question      |->| Web          |->| Report   ||
|  |            |  | Developing    |  | Exploration  |  | Generation||
|  +------------+  +---------------+  +--------------+  +----------+|
|                                                                    |
|  Each stage analyzed by:                                           |
|  - Training paradigm (RL, SFT, prompt-based)                     |
|  - Key modeling strategies                                        |
|  - Technical bottlenecks                                          |
|  - Cross-system comparisons                                       |
+------------------------------------------------------------------+
```

### Stage 1: Planning

Planning transforms a user query into a structured research plan of intermediate subgoals.

```
Taxonomy of Planning Methods:
+-----------------------------+
| Planning Approaches         |
+-----------------------------+
| 1. Prompt-based planning    |
|    - Zero-shot decomposition|
|    - Few-shot with exemplars|
|    - Chain-of-thought plans |
+-----------------------------+
| 2. Supervised planning      |
|    - Trained on expert plans|
|    - Outline-based planning |
+-----------------------------+
| 3. RL-optimized planning    |
|    - Dynamic plan adjustment|
|    - Emergent planning      |
|    (e.g., DeepResearcher)   |
+-----------------------------+
| 4. World-model-guided       |
|    - LLMs as implicit world |
|      models for reasoning   |
+-----------------------------+
```

Key insight: LLMs increasingly serve as implicit world models, providing environment priors for
goal-directed reasoning in long-horizon research tasks.

### Stage 2: Question Developing

Question developing generates the specific queries used to gather information.

```
Taxonomy of Question Developing:
+-----------------------------------+
| Reward-Optimized Methods          |
| - Trial-and-error query refinement|
| - RL-guided query generation      |
| - Outcome-reward-driven search    |
+-----------------------------------+
| Supervision-Driven Methods        |
| - SFT on expert query trajectories|
| - Manually designed query strategy|
| - Template-based query generation |
+-----------------------------------+
```

### Stage 3: Web Exploration

Web exploration executes information gathering from web sources.

```
Evolution of Web Retrieval Agents:
+-------------------------------------------------+
| Generation 1: Simple Extractors                 |
|   - Single-query search + snippet extraction    |
|   - No navigation or interaction                |
+-------------------------------------------------+
| Generation 2: Interactive Agents                |
|   - Multi-step search refinement                |
|   - Page navigation (click, scroll)             |
|   - Cross-document information synthesis        |
+-------------------------------------------------+
| Generation 3: Multimodal Agents                 |
|   - Image/chart/table understanding             |
|   - Complex interactive content navigation      |
|   - Deep web exploration (WebThinker-style)     |
+-------------------------------------------------+
```

### Stage 4: Report Generation

Report generation produces the final structured output.

```
Report Generation Approaches:
+------------------------------------------+
| 1. Outline-first generation (STORM)      |
|    - Generate outline, then fill sections |
+------------------------------------------+
| 2. Interleaved drafting (WebThinker)     |
|    - Draft sections during research      |
+------------------------------------------+
| 3. Post-hoc synthesis (OpenScholar)      |
|    - Collect all info, then synthesize   |
+------------------------------------------+
| 4. Iterative refinement                  |
|    - Draft, evaluate, revise, repeat     |
+------------------------------------------+
```

## Key Systems Compared

### Commercial Systems

| System              | Developer | Key Approach                          | Strengths           |
|--------------------|-----------|---------------------------------------|---------------------|
| OpenAI Deep Research| OpenAI    | Reasoning agent over web synthesis    | Best F1 (0.55 avg)  |
| Gemini Deep Research| Google    | Research assistant with web browsing   | Most citations (111)|
| Perplexity Deep Research| Perplexity| In-depth report generation         | Best citation acc (90%)|

### Academic Systems

| System          | Approach                    | Key Innovation                        |
|----------------|-----------------------------|------------------------------------- |
| DeepResearcher | End-to-end RL in real web   | Emergent cognitive behaviors          |
| WebThinker     | LRM + Deep Web Explorer     | Think-Search-Draft interleaving       |
| STORM          | Multi-perspective pre-writing| Perspective-guided conversations     |
| OpenScholar    | RAG over 45M papers         | Expert-level citation accuracy        |
| OpenResearcher | Offline trajectory synthesis | 97K trajectories, 100+ tool calls    |
| Search-R1      | RL with static corpus       | RAG-based RL baseline                 |
| Search-o1      | Search-augmented reasoning  | Basic search + reasoning integration  |

## Evaluation Methods

### Existing Benchmarks

| Benchmark        | Focus                           | Metrics                    |
|-----------------|--------------------------------|----------------------------|
| BrowseComp      | Web browsing comprehension     | Accuracy                   |
| BrowseComp-Plus | Complex browsing tasks         | Accuracy                   |
| GAIA            | General AI assistance          | Accuracy                   |
| GPQA            | Graduate-level science QA      | Accuracy                   |
| WebWalkerQA     | Web navigation QA              | Accuracy                   |
| HLE             | Hard long-form evaluation      | Accuracy                   |
| ScholarQABench  | Scientific literature QA       | Correctness, Citation F1   |
| FreshWiki       | Article generation quality     | Organization, Coverage     |
| Glaive          | Report generation quality      | Multi-dimensional (1-10)   |
| LiveDRBench     | Live deep research evaluation  | F1                         |
| ReportBench     | Academic survey generation     | Precision, quality metrics |

### Evaluation Challenges

1. **No unified benchmark**: Different systems use different benchmarks, making direct comparison
   difficult.
2. **Static vs. live evaluation**: Web content changes; benchmarks need updating.
3. **Multi-dimensional quality**: Report quality spans factuality, coverage, organization,
   citations, and coherence -- hard to capture in a single metric.
4. **Human evaluation cost**: Expert evaluation is expensive and time-consuming.

## Open Challenges

1. **Reasoning-driven retrieval**: How to deeply integrate reasoning into the retrieval process
   so that search strategies emerge from logical analysis rather than pattern matching.

2. **Self-evolving agents**: Agents that learn and refine their research strategies over time
   through interaction feedback, meta-learning, or exposure to diverse task distributions.

3. **Long-horizon consistency**: Maintaining coherent research plans over 100+ step trajectories
   without losing focus or introducing contradictions.

4. **Multimodal research**: Extending agents to process and synthesize information from images,
   charts, tables, videos, and other non-text modalities.

5. **Evaluation standardization**: Developing unified benchmarks that capture the multi-dimensional
   quality of deep research outputs.

6. **Coordination of modules**: Optimizing the interaction between planning, question developing,
   web exploration, and report generation stages.

7. **Trustworthiness and verification**: Ensuring factual accuracy and proper attribution in
   generated research outputs, especially for high-stakes scientific and policy applications.

8. **Efficiency and latency**: Reducing the computational cost and wall-clock time of deep
   research while maintaining quality.

## Limitations

1. **Survey scope**: Primarily covers English-language systems and publications.
2. **Rapid obsolescence**: The field evolves faster than survey publication cycles; some systems
   discussed may have been superseded by the time of reading.
3. **Benchmark fragmentation**: Unable to provide unified quantitative comparisons across all
   systems due to different evaluation protocols.
4. **Commercial system opacity**: Proprietary systems (OpenAI, Google, Perplexity) do not
   disclose full architectural details, limiting analysis depth.

## Follow-up Work

- Development of unified deep research benchmarks (DeepResearch Bench, ResearchRubrics).
- Extension to domain-specific deep research (medical, legal, scientific).
- Integration of multimodal capabilities into research agents.
- Hybrid RL+SFT training approaches combining the strengths of both paradigms.

## Key Takeaways

1. The four-stage pipeline (planning, question developing, web exploration, report generation)
   provides a useful modular framework for understanding and comparing deep research agents.
2. The field is rapidly moving from prompt-based to RL-trained agents, with emergent cognitive
   behaviors appearing in end-to-end RL systems.
3. Evaluation remains the weakest link: no unified benchmark exists, and multi-dimensional
   quality assessment of research outputs is an open problem.
4. Self-evolving agents that continuously improve their research strategies represent the most
   promising frontier in the field.
5. The tension between online RL (real web, expensive, non-reproducible) and offline SFT (static
   corpus, cheaper, reproducible) remains a fundamental design tradeoff.
