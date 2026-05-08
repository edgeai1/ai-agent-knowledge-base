---
title: "Tongyi DeepResearch Technical Report"
authors: "Tongyi DeepResearch Team (Alibaba)"
venue: "arXiv preprint"
year: 2025
url: "https://arxiv.org/abs/2510.24701"
code: "https://github.com/Alibaba-NLP/DeepResearch"
tags: [deep-research, agentic-training, mixture-of-experts, reinforcement-learning, information-seeking, open-source]
category: agent/deep_research
status: done
date_read: 2026-05-08
---

# Tongyi DeepResearch Technical Report

## TL;DR

Tongyi DeepResearch is an agentic LLM built on Qwen3-30B-A3B (30.5B total parameters, 3.3B active
per token via Mixture-of-Experts) specifically designed for long-horizon, deep information-seeking
research tasks. It is trained end-to-end through a novel pipeline combining Agentic Continual
Pre-training (Agentic CPT), Supervised Fine-Tuning (SFT) cold-starting, and Reinforcement Learning
(RL), all powered by a fully automatic data synthesis pipeline. Tongyi DeepResearch achieves 32.9
on Humanity's Last Exam, 43.4 on BrowseComp, 70.9 on GAIA, and 90.6 on FRAMES, outperforming
strong baselines including OpenAI-o3 and DeepSeek-V3.1. The model, framework, and complete tool
implementations are fully open-sourced.

## Motivation & Problem

Deep research tasks -- where a user asks a complex question requiring extensive web search,
document analysis, and synthesis -- present unique challenges for LLMs:

1. **Long-horizon agentic behavior**: Deep research requires dozens of sequential search, read,
   and reasoning actions over extended trajectories (100+ steps), far beyond what standard LLMs
   are trained for.
2. **Training-inference gap for agents**: Standard LLM pre-training on static text does not
   develop the agentic skills (tool use, search strategy, information synthesis) needed for
   interactive research tasks.
3. **Data scarcity for agentic training**: High-quality training data for agentic search behavior
   is expensive to collect through human annotation and difficult to synthesize automatically.
4. **Tool integration complexity**: Effective deep research requires seamless integration of
   multiple tools (web search, page visiting, code execution, file parsing, scholar search)
   within a coherent reasoning framework.
5. **Scaling vs. efficiency**: Large proprietary models (GPT-4, o3) achieve strong research
   performance but are expensive; building an efficient open-source alternative requires
   careful architectural and training choices.

Tongyi DeepResearch's approach: **End-to-end agentic training** -- from continual pre-training
through RL -- with a fully automatic data synthesis pipeline, building agentic capability
directly into the model rather than relying purely on prompting.

## Method

### Architecture Overview

```
+-------------------------------------------------------------------+
|              Tongyi DeepResearch Architecture                     |
|                                                                   |
|  Base: Qwen3-30B-A3B (MoE: 30.5B total, 3.3B active/token)      |
|                                                                   |
|  +-----------------------------------------------------------+   |
|  |                  ReAct Reasoning Loop                      |   |
|  |                                                            |   |
|  |  [Thought] --> [Action] --> [Observation] --> [Thought]    |   |
|  |      |             |              ^                        |   |
|  |      |             v              |                        |   |
|  |      |    +------------------+    |                        |   |
|  |      |    | Tool Execution   |    |                        |   |
|  |      |    | - Search         |----+                        |   |
|  |      |    | - Visit          |                             |   |
|  |      |    | - Python         |                             |   |
|  |      |    | - Scholar        |                             |   |
|  |      |    | - File Parser    |                             |   |
|  |      |    +------------------+                             |   |
|  |      |                                                     |   |
|  |      v                                                     |   |
|  |  +-------------------+    +--------------------+           |   |
|  |  | Context Manager   |    | Meta-Thinker Agent |           |   |
|  |  | (dynamic context  |    | (monitors trajectory|          |   |
|  |  |  compression &    |    |  detects anomalies, |          |   |
|  |  |  refresh)         |    |  issues corrections)|          |   |
|  |  +-------------------+    +--------------------+           |   |
|  +-----------------------------------------------------------+   |
+-------------------------------------------------------------------+
```

### Training Pipeline

The training follows a three-stage pipeline, each powered by automatic data synthesis:

```
Stage 1: Agentic Continual Pre-Training (Agentic CPT)
-------------------------------------------------------
Purpose: Bridge pre-trained model to agentic behavior
Method:  Two-phase CPT on synthesized agentic trajectories

  Phase 1: General agentic capability
    - Train on diverse tool-use trajectories
    - Develop basic ReAct pattern comprehension
    - Preserve broad linguistic competence

  Phase 2: Domain-specific agentic skills
    - Train on research-specific trajectories
    - Develop search strategy and synthesis skills
    - Build document analysis capabilities

Stage 2: Supervised Fine-Tuning (SFT) Cold Start
--------------------------------------------------
Purpose: Provide initial policy for RL training
Method:  Fine-tune on high-quality agentic demonstrations

  - Curated set of research task demonstrations
  - Complete trajectories: query -> search -> read -> synthesize -> answer
  - Cold-start policy enables stable RL training

Stage 3: Reinforcement Learning (RL)
--------------------------------------
Purpose: Optimize research strategy through reward feedback
Method:  RL with reward model trained on research quality

  Reward signals:
  - Answer correctness (verifiable against ground truth)
  - Research thoroughness (coverage of relevant sources)
  - Reasoning quality (coherence and depth of synthesis)
  - Efficiency (fewer unnecessary actions preferred)
```

```
Algorithm: Tongyi DeepResearch Training Pipeline
--------------------------------------------------
Input:  Qwen3-30B-A3B-Base model M_0
Output: Trained agentic model M_final

// Stage 1: Agentic CPT
1:  D_cpt = SynthesizeAgenticTrajectories()
2:  M_1 = ContinualPreTrain(M_0, D_cpt)

// Stage 2: SFT Cold Start
3:  D_sft = CurateHighQualityDemonstrations()
4:  M_2 = SupervisedFineTune(M_1, D_sft)

// Stage 3: RL
5:  R = TrainRewardModel(quality_labels)
6:  M_final = RLTraining(M_2, R, environment)

return M_final
```

### Data Synthesis Pipeline

A key contribution is the fully automatic data synthesis pipeline:

```
Data Synthesis for Each Training Stage:
-----------------------------------------

For Agentic CPT:
  1. Generate diverse research questions across domains
  2. Use teacher model to produce reference trajectories
  3. Simulate tool interactions in sandboxed environment
  4. Filter for quality: trajectory coherence, answer correctness
  5. Scale: hundreds of thousands of trajectories

For SFT:
  1. Select challenging research tasks from curated set
  2. Generate gold-standard trajectories with expert teacher model
  3. Verify answer correctness against ground truth
  4. Quality filtering: only top-quality demonstrations retained

For RL:
  1. Generate diverse task prompts
  2. Construct verifiable reward environments
  3. Enable model to interact with real/simulated tools
  4. Reward based on final answer correctness + process quality
```

Key design principle: **No costly human annotation**. The entire pipeline from data synthesis
through training operates automatically, enabling scalable iteration.

### Action Space

Tongyi DeepResearch operates with five tool types:

| Tool          | Description                                        | Use Case                        |
|---------------|----------------------------------------------------|---------------------------------|
| **Search**    | Web search via search engine API                   | Find relevant pages/information |
| **Visit**     | Fetch and parse web page content                   | Read detailed page content      |
| **Python**    | Execute Python code in sandbox                     | Data processing, computation    |
| **Scholar**   | Academic paper search                              | Find research papers/citations  |
| **File Parser**| Parse uploaded documents (PDF, DOCX, etc.)        | Analyze user-provided documents |

### Multi-Agent Internal Architecture

Tongyi DeepResearch employs an internal multi-agent design:

- **Tactical Agent**: Operates in a ReAct-style loop, handling step-by-step reasoning and tool
  calls. Works with a concise, dynamically refreshed context from the Context Manager.
- **Context Manager**: Maintains a compressed representation of the evolving research state,
  preventing context window overflow during long research trajectories.
- **Meta-Thinker Agent**: Asynchronously monitors the research trajectory, detecting anomalies
  such as repetitive failures, reasoning drift, or dead-end search paths, and issues strategic
  interventions to redirect the research process.

## Key Innovations

1. **End-to-end agentic training**: The three-stage pipeline (Agentic CPT -> SFT -> RL) builds
   agentic capability directly into model weights, rather than relying solely on prompting.
   This is a fundamentally different approach from prompt-only systems.
2. **Agentic Continual Pre-training**: A novel mid-training stage that bridges the gap between
   generic pre-trained models and agentic post-training, providing a strong inductive bias for
   agentic behavior while preserving general language capabilities.
3. **Fully automatic data synthesis**: The complete training data pipeline operates without human
   annotation, enabling scalable and cost-effective iteration on training data quality and
   diversity.
4. **Efficient MoE architecture**: Using only 3.3B active parameters from a 30.5B total model
   provides strong performance at dramatically lower inference cost than dense models of
   comparable capability.
5. **Meta-Thinker for trajectory correction**: The asynchronous meta-monitoring agent that detects
   and corrects research trajectory anomalies is a novel architectural contribution for
   maintaining quality in long-horizon agentic tasks.
6. **Full open-source release**: Model weights, training framework, tool implementations, prompt
   configurations, and reproduction scripts are all released, enabling community iteration.

## Experimental Setup

- **Base model**: Qwen3-30B-A3B-Base (30.5B total, 3.3B active per token)
- **Benchmarks**:
  - Humanity's Last Exam (HLE): Expert-level questions across all domains
  - BrowseComp / BrowseComp-ZH: Web browsing comprehension (English / Chinese)
  - WebWalkerQA: Web navigation question answering
  - GAIA: General AI Assistants benchmark
  - FRAMES: Factual research and multi-step reasoning
  - xbench-DeepSearch / xbench-DeepSearch-2510: Deep search evaluation
- **Baselines**: OpenAI-o3, DeepSeek-V3.1, Claude, Gemini, and other strong models
- **Tool access**: Full access to web search, page visiting, code execution, scholar search

## Results

### Benchmark Performance

| Benchmark                | Tongyi DeepResearch | OpenAI-o3 | DeepSeek-V3.1 |
|--------------------------|:-------------------:|:---------:|:-------------:|
| Humanity's Last Exam     | **32.9**            | lower     | lower         |
| BrowseComp               | **43.4**            | lower     | lower         |
| BrowseComp-ZH            | **46.7**            | --        | lower         |
| WebWalkerQA              | **72.2**            | --        | lower         |
| GAIA                     | **70.9**            | lower     | lower         |
| FRAMES                   | **90.6**            | lower     | lower         |
| xbench-DeepSearch        | **75.0**            | --        | --            |
| xbench-DeepSearch-2510   | **55.0**            | --        | --            |

Tongyi DeepResearch outperforms strong baselines including OpenAI-o3 and DeepSeek-V3.1 across
all evaluated benchmarks, demonstrating the effectiveness of end-to-end agentic training.

### Efficiency Analysis

| Property                 | Value                          |
|--------------------------|--------------------------------|
| Total parameters         | 30.5 billion                   |
| Active parameters/token  | 3.3 billion                    |
| MoE efficiency           | 10.8% activation ratio         |
| Comparable to dense      | ~7-14B dense model compute     |

The MoE architecture provides performance competitive with much larger dense models while
using only a fraction of the compute per token.

### Training Pipeline Impact (Ablation)

| Configuration                    | Relative Performance |
|----------------------------------|:--------------------:|
| Base model (no agentic training) | Baseline             |
| + Agentic CPT only              | Significant gain     |
| + Agentic CPT + SFT             | Further improvement  |
| + Agentic CPT + SFT + RL        | **Best performance** |

Each training stage contributes meaningfully, with the full three-stage pipeline achieving
substantially better results than any subset.

## Limitations

1. **MoE architecture constraints**: While efficient, the MoE architecture requires more total
   memory than a 3.3B dense model despite similar compute per token, limiting deployment on
   resource-constrained hardware.
2. **Tool dependency**: Performance is contingent on the quality and availability of external
   tools (search APIs, web access). Tool failures directly impact research quality.
3. **Training data quality ceiling**: The automatic data synthesis pipeline, while scalable, may
   have quality limitations compared to expert human annotation for the most challenging tasks.
4. **Long trajectory instability**: Despite the Meta-Thinker, very long research trajectories
   (100+ steps) may still suffer from accumulated errors and context degradation.
5. **Benchmark-specific optimization risk**: Strong benchmark performance may not fully generalize
   to the diverse range of real-world research tasks users actually pose.
6. **Chinese-English focus**: Primarily evaluated on English and Chinese benchmarks; performance
   on other languages is not extensively validated.
7. **Reproducibility of RL**: Reinforcement learning results can be sensitive to hyperparameters
   and random seeds, making exact reproduction of reported numbers challenging.
8. **Comparison fairness**: Baseline models (OpenAI-o3, DeepSeek-V3.1) may not have been
   optimized for the same benchmarks, making direct comparison nuanced.

## Key Takeaways

1. **Agentic training is transformative**: Building agentic capability directly into model weights
   through a CPT -> SFT -> RL pipeline produces fundamentally stronger research agents than
   prompting alone. This three-stage recipe is likely to become standard for agentic model
   development.
2. **Agentic CPT bridges the training gap**: The novel mid-training stage that exposes models to
   agentic trajectories before fine-tuning is a key architectural contribution -- it provides the
   inductive bias that makes subsequent SFT and RL training effective.
3. **Automatic data synthesis enables scale**: Removing human annotation from the training
   pipeline allows rapid iteration and scaling, a critical enabler for practical agentic model
   development.
4. **MoE enables efficient agentic models**: The 30.5B/3.3B MoE architecture demonstrates that
   strong agentic capabilities do not require dense large models, opening the door to
   cost-effective deployment of research agents.
5. **Meta-monitoring improves long trajectories**: The Meta-Thinker agent that detects and
   corrects trajectory anomalies is an important architectural pattern for any system performing
   long-horizon agentic tasks.
6. **Open-source leadership**: By fully open-sourcing model, framework, and tools, Tongyi
   DeepResearch enables community research and establishes a strong open-source baseline that
   competes with proprietary systems.
7. **End-to-end vs. modular design**: Tongyi DeepResearch's end-to-end training approach
   contrasts with modular systems (MindSearch, WebThinker) that compose separately trained
   components, suggesting that integrated training may be superior for deeply agentic tasks.
