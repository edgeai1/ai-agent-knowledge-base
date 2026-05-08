---
title: "O-Researcher: An Open Ended Deep Research Model via Multi-Agent Distillation and Agentic RL"
authors:
  - Yi Yao
  - He Zhu
  - Piaohong Wang
  - Jincheng Ren
  - Xinlong Yang
  - Qianben Chen
  - Xiaowan Li
  - Dingfeng Shi
  - Jiaxian Li
  - Qiexiang Wang
  - Sinuo Wang
  - Xinpeng Liu
  - Jiaqi Wu
  - Minghao Liu
  - Wangchunshu Zhou
venue: arXiv preprint
year: 2026
url: https://arxiv.org/abs/2601.03743
tags:
  - deep-research
  - multi-agent-distillation
  - agentic-RL
  - open-source
  - knowledge-distillation
  - tool-integrated-reasoning
  - research-agent
status: done
---

## TL;DR

O-Researcher proposes a framework to close the performance gap between closed-source and
open-source LLMs for deep research tasks. The approach uses multi-agent distillation to
synthesize high-fidelity research-grade training data, followed by a two-stage training
strategy combining supervised fine-tuning (SFT) with a novel agentic reinforcement learning
method. The framework captures the full iterative research process -- from query planning
to final report synthesis -- and enables open-source models to achieve new state-of-the-art
performance on major deep research benchmarks.

## Motivation & Problem

The deep research capability gap between closed-source models (GPT-4o, Claude, Gemini) and
open-source models is primarily attributed to:

1. **Data disparity**: Closed-source providers have access to vast amounts of proprietary,
   high-quality training data including human feedback, tool-use traces, and research
   workflows. Open-source models lack comparable data.

2. **Pipeline complexity**: Deep research involves iterative cycles of query planning,
   information retrieval, evidence evaluation, synthesis, and report generation. Capturing
   this full pipeline in training data is difficult.

3. **Tool integration**: Real deep research requires seamless integration with search
   engines, databases, and document analysis tools. Most training data lacks rich
   tool-integrated reasoning traces.

4. **Proprietary API dependency**: Existing approaches rely on distilling from proprietary
   APIs (e.g., GPT-4), creating an unsustainable dependency and potential legal/ethical
   concerns.

Prior work like TaskCraft and basic multi-agent distillation approaches capture only
fragments of the research process. O-Researcher aims to capture the complete, iterative,
exploratory essence of deep research end-to-end.

## Method

### Multi-Agent Distillation Framework

O-Researcher uses a collaborative multi-agent system to generate synthetic research data:

```
+================================================================+
|              MULTI-AGENT DATA SYNTHESIS PIPELINE                 |
+================================================================+
|                                                                  |
|  +------------------+                                            |
|  | Research Query   | (seed queries covering diverse domains)    |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | PLANNER AGENT    | Decomposes query into sub-questions,       |
|  |                  | creates research plan with milestones      |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | RETRIEVER AGENT  | Executes searches, selects sources,        |
|  |                  | manages tool calls (web search, DB)        |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | EVALUATOR AGENT  | Assesses evidence quality, checks          |
|  |                  | relevance, identifies contradictions        |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | SYNTHESIZER      | Combines evidence into coherent findings,  |
|  | AGENT            | resolves conflicts, generates report       |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | CRITIC AGENT     | Reviews report for completeness, accuracy, |
|  |                  | triggers re-research if gaps found         |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +------------------+                                            |
|  | Complete Research | Full trace with all agent interactions,    |
|  | Trajectory       | tool calls, and intermediate reasoning     |
|  +------------------+                                            |
+================================================================+
```

### Two-Stage Training Strategy

```
STAGE 1: Supervised Fine-Tuning (SFT)
+------------------------------------------------------------------+
|  Input: Base open-source model + synthetic research trajectories  |
|                                                                    |
|  Process:                                                          |
|  - Flatten multi-agent trajectories into single-model format      |
|  - Include tool-call tokens and search result formatting          |
|  - Train with standard cross-entropy loss on complete traces      |
|  - Establish baseline research behavior and tool-use patterns     |
|                                                                    |
|  Output: SFT model with basic deep research capability            |
+------------------------------------------------------------------+

STAGE 2: Agentic Reinforcement Learning
+------------------------------------------------------------------+
|  Input: SFT model + research environment with live tool access    |
|                                                                    |
|  Process:                                                          |
|  - Agent conducts research on new queries in interactive env      |
|  - Reward signal combines:                                         |
|    * Final report quality (accuracy, completeness, coherence)      |
|    * Intermediate research quality (query diversity, source        |
|      coverage, evidence relevance)                                 |
|    * Self-correction behavior (identifying and fixing errors)      |
|  - Policy optimization with trajectory-level rewards               |
|  - Emphasis on exploration and iterative refinement behaviors      |
|                                                                    |
|  Output: O-Researcher model with refined research capabilities    |
+------------------------------------------------------------------+
```

### Research Process Captured

Unlike prior distillation that captures single-turn or shallow multi-turn interactions,
O-Researcher captures the full research lifecycle:

```
Full Research Lifecycle:

1. QUERY UNDERSTANDING
   - Parse user intent
   - Identify knowledge gaps
   - Determine scope and depth

2. RESEARCH PLANNING
   - Decompose into sub-questions
   - Prioritize investigation order
   - Set milestones and checkpoints

3. ITERATIVE INVESTIGATION
   +---> Search for information
   |     Evaluate retrieved evidence
   |     Identify gaps or contradictions
   +---- Refine queries and re-search (loop)

4. EVIDENCE SYNTHESIS
   - Aggregate findings across sources
   - Resolve conflicting information
   - Build structured argument

5. REPORT GENERATION
   - Produce coherent, well-cited report
   - Include methodology and limitations
   - Provide confidence assessments

6. SELF-REVIEW & REVISION
   - Check completeness against plan
   - Verify factual claims
   - Revise if quality threshold not met
```

## Key Innovations

1. **End-to-end research trajectory synthesis**: First framework to capture the complete
   deep research process (planning through report synthesis) in synthetic training data,
   rather than fragmentary task completions.

2. **Multi-agent collaborative distillation**: Multiple specialized agents (planner,
   retriever, evaluator, synthesizer, critic) collaborate to produce training data that
   a single model can then learn from, combining expertise without requiring a single
   model to have all capabilities from the start.

3. **Agentic RL for research behavior**: RL stage specifically targets research-quality
   behaviors (exploration, self-correction, evidence evaluation) rather than generic
   instruction following, using a composite reward that captures both process and outcome.

4. **Democratization of deep research**: Enables open-source models to achieve competitive
   performance with closed-source systems, reducing dependency on proprietary APIs.

## Experimental Setup

- **Base models**: Open-source models at multiple scales (specific model names not
  confirmed in search results; likely Qwen or Llama family)
- **Benchmarks**: Major deep research benchmarks including GAIA (general AI assistant
  tasks requiring web browsing and multi-step reasoning)
- **Baselines**: Closed-source models (GPT-4o, Claude), prior open-source deep research
  agents, basic distillation approaches
- **Evaluation**: Report quality (accuracy, completeness, coherence), tool-use
  effectiveness, research depth, factual correctness

## Results

### Key Performance Claims

- Open-source models trained with O-Researcher achieve **new state-of-the-art for
  open-source models** on major deep research benchmarks.
- Models **significantly close the gap** to closed-source systems (GPT-4o, Claude).
- The framework is effective across **multiple model scales**, demonstrating the
  approach is not limited to large models only.
- Both SFT and RL stages contribute meaningfully: SFT establishes baseline capability,
  RL refines self-correction and iterative research behaviors.

### Contextual Reference Points (Other Deep Research Systems)

| System               | GAIA Score (approx) | Type         |
|---------------------|---------------------|--------------|
| OpenAI Deep Research | ~67% (avg)          | Closed-source|
| Tongyi DeepResearch  | 70.9                | Closed-source|
| Open-source baseline | ~55%                | Open-source  |
| O-Researcher         | SOTA for open-source| Open-source  |

### Ablation Findings

- Multi-agent distillation produces higher quality training data than single-agent
  generation, as each agent contributes specialized expertise.
- The RL stage is essential: SFT alone produces models that follow research patterns but
  lack iterative refinement and self-correction capabilities.
- Research trajectory completeness matters: models trained on full end-to-end traces
  outperform those trained on fragmented sub-task data.

## Limitations

1. **Benchmark specificity**: Deep research benchmarks (GAIA, etc.) may not fully capture
   the diversity and difficulty of real-world research tasks, which span technical depth,
   creative synthesis, and domain expertise.

2. **Data quality ceiling**: The quality of synthetic trajectories is bounded by the
   capabilities of the agent system generating them. If the multi-agent pipeline makes
   systematic errors, these propagate into training.

3. **Evaluation difficulty**: Assessing "research quality" is inherently subjective and
   multi-dimensional. Automated metrics may not capture aspects like insight depth,
   argument quality, or practical utility.

4. **Tool environment dependency**: The RL stage requires a research environment with
   live tool access (search engines, databases), which may not be stable or reproducible
   across training runs.

5. **Scale and generalization**: Competitive performance likely requires 32B+ models
   and significant RL compute. Fine-tuning for deep research may reduce general
   instruction-following capability; this trade-off is not thoroughly analyzed.

## Key Takeaways

1. Multi-agent distillation is a powerful paradigm for generating high-quality training
   data: specialized agents can collaboratively produce traces that exceed what any
   single agent could generate, providing a richer learning signal for student models.

2. The two-stage training approach (SFT then RL) is well-motivated for deep research:
   SFT provides the structural foundation (how to plan, search, synthesize), while RL
   refines the adaptive behaviors (when to re-search, how to self-correct) that
   distinguish good research from formulaic pattern-matching.

3. Capturing the full research lifecycle in training data is critical. Models trained on
   fragmentary sub-tasks fail to develop the iterative, exploratory behavior that
   characterizes genuine deep research.

4. The performance gap between open-source and closed-source research agents is
   significantly narrower than commonly assumed, and can be further closed with
   better training data and RL approaches rather than raw model scale.

5. The combination of synthetic data generation and agentic RL can substitute for
   proprietary data advantages, providing a scalable path to democratizing deep
   research. The framework is complementary to model scaling and can be re-applied
   as base models improve.
