---
title: "WebThinker: Empowering Large Reasoning Models with Deep Research Capability"
authors:
  - Xiaoxi Li
  - Jiajie Jin
  - Yujia Zhou
  - Yongfeng Zhang
  - (Renmin University of China, RUC-NLPIR)
venue: NeurIPS 2025 (also presented as WWW 2026 Oral)
year: 2025
url: https://arxiv.org/abs/2504.21776
tags:
  - deep-research
  - large-reasoning-models
  - web-exploration
  - report-generation
  - DPO
  - think-search-draft
status: done
---

# WebThinker: Empowering Large Reasoning Models with Deep Research Capability

## TL;DR

WebThinker empowers Large Reasoning Models (LRMs) with autonomous web search, navigation, and
report generation capabilities. It introduces a Deep Web Explorer module for multi-step web
navigation and an Autonomous Think-Search-and-Draft strategy for interleaved reasoning, search,
and writing. Trained via iterative online DPO, WebThinker-32B achieves +22.9% on WebWalkerQA and
+20.4% on HLE versus Search-o1, and scores 8.0 overall on Glaive report generation (surpassing
Gemini-Deep Research at 7.9).

## Motivation & Problem

Large Reasoning Models (e.g., DeepSeek-R1, QwQ) have demonstrated strong internal reasoning via
extended chain-of-thought. However, they suffer from critical limitations:

1. **Knowledge boundaries**: LRMs rely on static pre-training knowledge; they cannot access
   up-to-date or domain-specific information during reasoning.
2. **Shallow retrieval integration**: Standard RAG pipelines perform one-shot retrieval before
   reasoning, failing to support iterative, depth-first information gathering.
3. **No report generation**: Existing search-augmented reasoning systems produce short answers
   but cannot generate comprehensive, structured research reports.

WebThinker addresses all three by embedding web exploration and report drafting directly into the
LRM's reasoning process.

## Method

### System Architecture

```
+-------------------------------------------------------------------+
|                    WebThinker Reasoning Loop                       |
|                                                                    |
|  +-------------+    +------------------+    +-----------------+    |
|  | <think>     |--->| Deep Web Explorer|--->| <think>         |    |
|  | Reasoning   |    | Module           |    | Continue        |    |
|  | (identify   |    |                  |    | Reasoning       |    |
|  |  knowledge  |    | search(query)    |    | with new info   |    |
|  |  gaps)      |    | click(element)   |    |                 |    |
|  +-------------+    | extract(content) |    +-----------------+    |
|        |            | navigate(url)    |           |               |
|        v            +------------------+           v               |
|  +-----------+                              +-----------+          |
|  | Produce   |                              | Draft     |          |
|  | Answer    |                              | Report    |          |
|  | (QA mode) |                              | (report   |          |
|  +-----------+                              |  mode)    |          |
|                                             +-----------+          |
+-------------------------------------------------------------------+
```

### Deep Web Explorer Module

The Deep Web Explorer enables LRMs to go beyond single-query search:

```
Capability 1: search(query) -> ranked results
  - Issue search queries to web search engines
  - Receive ranked list of result snippets

Capability 2: click(link/button) -> page content
  - Navigate to specific URLs from search results
  - Click interactive elements (links, buttons) on pages

Capability 3: extract(selector) -> relevant content
  - Extract specific information from loaded pages
  - Parse and filter page content for relevance

Capability 4: navigate(url) -> page content
  - Follow deeper links discovered during exploration
  - Enables multi-hop web traversal
```

The key distinction from standard RAG: the LRM autonomously decides when, what, and how deeply
to search based on its evolving understanding during the reasoning process.

### Two Operating Modes

**Mode 1: Problem Solving (QA)**
```
while question not answered:
    <think> Reason about current knowledge state </think>
    if knowledge_gap_detected:
        <search> query </search>          # or click/navigate
        <result> web content </result>
    <think> Integrate new information </think>
<answer> final answer </answer>
```

**Mode 2: Report Generation (Think-Search-and-Draft)**
```
while report not complete:
    <think> Plan next section / identify gaps </think>
    <search> targeted query for section </search>
    <result> web content </result>
    <draft> Write/revise report section </draft>
    <think> Assess completeness, plan next steps </think>
<report> final structured report </report>
```

### RL Training via Iterative Online DPO

```
Algorithm: Iterative Online DPO for WebThinker
-------------------------------------------------
1. Start with base LRM (e.g., DeepSeek-R1-distill-Qwen-32B)
2. for each DPO iteration:
3.   Sample questions from training set
4.   Generate multiple reasoning trajectories (with web search)
5.   Evaluate trajectories by outcome quality
6.   Construct preference pairs: (better_trajectory, worse_trajectory)
7.   Update model via DPO loss:
8.     L_DPO = -log sigma(beta * (log pi(y_w|x) - log pi(y_l|x)))
9.   Use updated model as new reference for next iteration
```

The iterative online approach continuously generates fresh preference data from the improving
model, avoiding distribution shift from static preference datasets.

## Key Innovations

1. **Deep Web Explorer**: Moves beyond single-query RAG to multi-step web navigation with
   click, navigate, and extract capabilities embedded in the reasoning loop.

2. **Autonomous Think-Search-and-Draft**: First framework to unify reasoning, web exploration,
   and report writing in a single interleaved process.

3. **Iterative online DPO**: Continuously improves tool utilization through on-policy preference
   optimization, avoiding the instability of PPO/GRPO while still enabling exploration.

4. **Dual-mode operation**: Same architecture handles both short-answer QA and long-form report
   generation tasks.

## Experimental Setup

### Benchmarks

| Benchmark    | Task Type                   | Metric        |
|-------------|----------------------------|---------------|
| GPQA         | Graduate-level science QA  | Accuracy      |
| GAIA         | General AI assistant       | Accuracy      |
| WebWalkerQA  | Web navigation QA          | Accuracy      |
| HLE          | Hard long-form evaluation  | Accuracy      |
| Glaive       | Scientific report writing  | Multi-dim /10 |

### Model Variants

- **WebThinker-32B-Base**: DeepSeek-R1-distill-Qwen-32B with Deep Web Explorer (no DPO).
- **WebThinker-32B-RL**: After iterative online DPO training.
- **WebThinker-7B variants**: Using DeepSeek-R1-7B backbone.

### Baselines

- **Direct generation**: LRMs without any retrieval.
- **Standard RAG**: One-shot retrieval before reasoning.
- **Search-o1**: Prior search-augmented reasoning system.
- **Proprietary systems**: Gemini-Deep Research, other commercial systems.

## Results

### QA Benchmarks

| Model              | GPQA  | GAIA  | WebWalkerQA | HLE   |
|-------------------|-------|-------|-------------|-------|
| WebThinker-32B-RL | 70.7% | best  | best        | 15.8% |
| Search-o1         | lower | lower | -22.9%      | -20.4%|

- **+22.9%** over Search-o1 on WebWalkerQA.
- **+20.4%** over Search-o1 on HLE.
- **70.7%** accuracy on GPQA (graduate-level science).
- Up to **+21.5%** gains over baselines across benchmarks.

### With 7B Backbone (Relative Improvements)

- **+174.4%** on GAIA over direct generation.
- **+422.6%** on WebWalkerQA over direct generation.
- **+82.9%** on GAIA over standard RAG.
- **+161.3%** on WebWalkerQA over standard RAG.

### Report Generation (Glaive Benchmark)

| System              | Overall | Completeness | Thoroughness | Factuality | Coherence |
|--------------------|---------|-------------|-------------|-----------|----------|
| WebThinker-32B     | **8.0** | **8.4**     | **8.2**     | 7.7       | 7.8      |
| Gemini-Deep Research| 7.9    | lower       | lower       | ~7.7      | ~7.8     |

WebThinker achieves the top overall score, surpassing Gemini-Deep Research.

## Limitations

1. **LRM backbone dependency**: Performance is tightly coupled to the base LRM quality; weaker
   backbones show smaller absolute gains.
2. **Latency**: Multi-step web exploration adds significant inference time compared to
   single-query RAG approaches.
3. **Web page complexity**: The Deep Web Explorer may struggle with highly dynamic, JavaScript-
   heavy pages or pages requiring authentication.
4. **DPO training scope**: Iterative DPO improves tool use but may not fully explore novel
   research strategies that deviate from the initial policy.
5. **Report evaluation**: Glaive benchmark uses LLM-based scoring, which may not fully capture
   human preferences for report quality.

## Follow-up Work

- **Co-STORM** and **STORM**: Related systems for collaborative and structured article writing.
- **DeepResearcher**: Alternative RL-based approach using GRPO in live web environments.
- Integration with multimodal LRMs for research involving images, charts, and tables.
- Extension to domain-specific research (medical, legal, scientific literature).

## Key Takeaways

1. Embedding web exploration directly into the LRM's reasoning loop (rather than as a separate
   pre-processing step) enables significantly deeper and more targeted information gathering.
2. The Think-Search-and-Draft strategy demonstrates that reasoning, retrieval, and generation
   can be unified in a single interleaved process for report writing.
3. Iterative online DPO is an effective and stable alternative to PPO/GRPO for training
   tool-augmented reasoning agents.
4. Even relatively small models (7B) show dramatic improvements (+174-422%) when equipped with
   autonomous web exploration during reasoning.
5. WebThinker's report generation quality (8.0) matching or exceeding commercial systems like
   Gemini-Deep Research (7.9) validates the open-source approach to deep research.
