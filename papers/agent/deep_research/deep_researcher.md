---
title: "DeepResearcher: Scaling Deep Research via Reinforcement Learning in Real-world Environments"
authors:
  - Yuxiang Zheng
  - Lyumanshan Ye
  - Kaiwen Liu
  - Jianbo Yuan
  - Minghao Shao
  - Wenyuan Xu
  - Xuming Hu
  - Zhiwei Jia
  - Kevin Chen-Chuan Chang
  - Graham Neubig
  - Huan Sun
venue: EMNLP 2025
year: 2025
url: https://arxiv.org/abs/2504.03160
tags:
  - deep-research
  - reinforcement-learning
  - web-search-agent
  - GRPO
  - emergent-behaviors
  - end-to-end-training
status: done
---

# DeepResearcher: Scaling Deep Research via Reinforcement Learning in Real-world Environments

## TL;DR

DeepResearcher is the first end-to-end reinforcement learning framework for training deep research
agents that interact with real web search environments. Built on Qwen2.5-7B-Instruct and trained
via GRPO with outcome-only rewards, it achieves +28.9 points over prompt engineering baselines
and +7.2 over RAG-based RL agents on open-domain QA. The key finding is that end-to-end RL in
real web environments produces emergent cognitive behaviors (planning, cross-validation,
self-reflection, honest abstention) that cannot be achieved through supervised fine-tuning alone.

## Motivation & Problem

Existing approaches to building deep research agents fall into two categories:
1. **Prompt-engineering methods** -- hand-craft agentic workflows (e.g., chain-of-thought with
   tool-use instructions) but cannot adapt or improve through experience.
2. **RAG-based RL methods** -- train with RL but use a static, curated retrieval corpus rather than
   live web search, creating a distribution mismatch with real-world deployment.

Neither approach learns to conduct genuine multi-step web research end-to-end. DeepResearcher
addresses this gap by training an agent directly in real web environments using only outcome-based
rewards, letting the agent discover research strategies autonomously.

## Method

### Architecture

```
+------------------+     +------------------+     +------------------+
| Qwen2.5-7B-Inst  |---->| Think / Reason   |---->| Generate Query   |
| (Policy Model)   |     | (internal CoT)   |     | <search>q</search>|
+------------------+     +------------------+     +------------------+
                                                          |
                                                          v
                                                  +------------------+
                                                  | Real Web Search  |
                                                  | (Bing/Google API)|
                                                  +------------------+
                                                          |
                                                          v
                                                  +------------------+
                                                  | Parse & Integrate|
                                                  | Search Results   |
                                                  +------------------+
                                                          |
                                                          v
                                                  +------------------+
                                                  | Continue Thinking|
                                                  | or Produce Answer|
                                                  +------------------+
```

### Training Pipeline (GRPO)

```
Algorithm: DeepResearcher Training via GRPO
-------------------------------------------------
Input: Base model pi_0, question set Q, reward fn R
Output: Trained policy pi_theta

1. for each training iteration t:
2.   Sample batch of questions q ~ Q
3.   for each q, generate G rollouts using pi_theta:
4.     Each rollout = interleaved (think, search, observe)* + answer
5.     Searches hit REAL web APIs (Bing/Google)
6.   Compute outcome reward r_i = R(answer_i, gold_answer)
7.   Compute group-relative advantages:
8.     A_i = (r_i - mean(r_1..G)) / std(r_1..G)
9.   Update pi_theta via clipped policy gradient (GRPO objective)
10.  Apply KL penalty against reference policy pi_ref
```

### Reward Design

- **Outcome-only reward**: Binary correctness signal (F1 or exact match against gold answer).
- No intermediate rewards for search quality, reasoning steps, or tool usage.
- The agent must discover effective search strategies purely from the final-answer signal.

### Data Curation (Two-Stage Filtering)

1. **Quality filter**: Remove time-sensitive, subjective, or harmful questions.
2. **Contamination filter**: Test base model on questions without search (pass@10). Exclude
   questions the model can already answer, ensuring the agent learns genuine research skills.

### Training Infrastructure

- **Framework**: veRL (distributed RL framework).
- **Search parallelism**: 50 CPU node cluster for concurrent web search during GRPO rollouts.
- **Caching**: 7-day TTL for identical search queries to reduce API costs.
- **Retry mechanisms**: Robust handling of rate limits and transient crawling failures.

## Key Innovations

1. **First end-to-end RL in real web environments**: Unlike RAG-based RL (Search-R1, R1-searcher),
   DeepResearcher trains on live web search, eliminating corpus-deployment distribution mismatch.

2. **Emergent cognitive behaviors** (not explicitly trained):
   - **Planning**: Agent formulates and dynamically adjusts multi-step research plans; can merge
     steps when appropriate.
   - **Cross-validation**: After finding an answer, the agent searches for corroborating evidence
     from independent sources before committing.
   - **Self-reflection**: Agent recognizes when a search strategy fails and redirects research.
   - **Honest abstention**: When evidence is insufficient, the agent acknowledges uncertainty
     rather than fabricating answers.

3. **Scalable training with outcome-only rewards**: Demonstrates that complex research behaviors
   can emerge from simple binary reward signals without reward shaping.

## Experimental Setup

### Datasets

| Split        | Datasets                                    | Hop Type    |
|-------------|---------------------------------------------|-------------|
| In-domain   | NQ, TriviaQA, HotpotQA, 2WikiMultiHopQA    | Single/Multi|
| Out-of-domain| MuSiQue, Bamboogle, PopQA                  | Multi-hop   |

### Baselines Compared

- **Prompt-based**: Direct prompting, ReAct, Search-o1
- **SFT-based**: Fine-tuned on search trajectories
- **RAG-based RL**: Search-R1, R1-Searcher (use static corpus, not live web)

### Metrics

- **F1 score**: Token-level overlap with gold answer.
- **Model-Based Evaluation (MBE)**: LLM-as-judge for semantic correctness (considered more reliable).

## Results

### Main Performance (MBE metric)

- **+28.9 points** over best prompt engineering baseline (averaged across benchmarks).
- **+7.2 points** over best RAG-based RL agent (Search-R1 / R1-Searcher).
- Highest MBE scores across all four in-domain datasets (NQ, TQ, HotpotQA, 2Wiki).
- Consistent superiority on all three out-of-domain datasets, demonstrating generalization.

### Key Comparisons

- DeepResearcher vs. prompt-based ReAct: Large gap shows RL-learned strategies far outperform
  hand-crafted prompting workflows.
- DeepResearcher vs. RAG-RL agents: The 7.2-point gap validates that real web training is
  critical, not just RL training on static corpora.
- Particularly strong on multi-hop tasks (HotpotQA, 2Wiki) requiring information synthesis.

## Limitations

1. **Computational cost**: Training requires 50 CPU nodes for parallel web search plus GPU cluster
   for GRPO, making reproduction expensive.
2. **Search API dependency**: Results depend on search engine quality; API rate limits constrain
   rollout throughput.
3. **7B model scale**: Only demonstrated on Qwen2.5-7B; scaling behavior to larger models unknown.
4. **Benchmark scope**: Evaluated on factoid QA benchmarks; not tested on long-form report
   generation or complex analytical tasks.
5. **Reward sparsity**: Outcome-only reward may lead to slow convergence on harder tasks where
   partial credit would help.

## Follow-up Work

- **PokeeResearch**: Extends RL-based deep research with additional reward shaping.
- **DeepDive**: Combines knowledge graphs with multi-turn RL for deeper search trajectories.
- **WebThinker**: Adds report generation and Deep Web Explorer to the reasoning loop.
- **OpenResearcher**: Takes a supervised distillation approach with offline trajectory synthesis as
  an alternative to online RL.

## Key Takeaways

1. End-to-end RL in real web environments is "not merely an implementation detail but a fundamental
   requirement" for robust deep research agents.
2. Complex cognitive behaviors (planning, cross-validation, self-reflection) emerge naturally from
   outcome-only RL without explicit supervision on these skills.
3. The distribution gap between static retrieval corpora and live web search is significant enough
   to justify the added complexity of real-web RL training.
4. A 7B parameter model, when properly trained with RL, can match or exceed much larger
   prompt-engineered systems on multi-step research tasks.
5. The veRL framework with distributed search parallelism provides a practical template for
   scaling RL training with tool-use agents.
