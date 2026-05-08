---
title: "Search-o1: Agentic Search-Enhanced Large Reasoning Models"
authors: "Xiaoxi Li, Guanting Dong, and others"
venue: "EMNLP 2025"
year: 2025
url: "https://arxiv.org/abs/2501.05366"
code: "https://github.com/RUC-NLPIR/Search-o1"
tags: [reasoning, agentic-search, RAG, knowledge-retrieval, reasoning-models, chain-of-thought]
category: agent/deep_research
status: done
date_read: 2026-05-08
---

# Search-o1: Agentic Search-Enhanced Large Reasoning Models

## TL;DR

Search-o1 addresses knowledge insufficiency in large reasoning models (LRMs) like OpenAI-o1 and
QwQ by integrating an agentic search workflow into the reasoning process. When the model encounters
uncertain knowledge during its chain-of-thought, it dynamically triggers external retrieval. A
novel Reason-in-Documents (RiD) module deeply analyzes retrieved documents before injecting them
into the reasoning chain, minimizing noise and preserving coherent reasoning flow. Evaluated on
complex reasoning (GPQA, MATH, etc.) and open-domain QA benchmarks, Search-o1 outperforms both
vanilla reasoning models and naive RAG-augmented baselines, exceeding QwQ-32B by +3.1% and
RAgent-QwQ-32B by +4.7% on average.

## Motivation & Problem

Large reasoning models (LRMs) such as OpenAI-o1 and QwQ-32B-Preview have demonstrated impressive
long stepwise reasoning capabilities through large-scale reinforcement learning. However, their
extended reasoning processes suffer from critical limitations:

1. **Knowledge insufficiency**: During long chain-of-thought (CoT) reasoning, LRMs frequently
   encounter knowledge gaps -- facts, formulas, or domain-specific information not captured in
   model parameters. These gaps cause reasoning uncertainty and error propagation.
2. **Hallucination under uncertainty**: When LRMs lack knowledge, they tend to hallucinate plausible
   but incorrect facts, which then contaminate subsequent reasoning steps.
3. **Naive RAG disruption**: Simply augmenting reasoning with retrieved documents (standard RAG)
   introduces noisy, often irrelevant information that disrupts the coherent reasoning chain,
   sometimes degrading rather than improving performance.
4. **Static knowledge boundaries**: Unlike humans who seamlessly consult external sources mid-
   reasoning, LRMs operate entirely within parametric knowledge, unable to "look things up"
   during their thought process.

Search-o1's insight: **The retrieval trigger should be determined by the reasoning process itself**
(not by a separate system), and retrieved documents must be deeply analyzed and distilled before
injection to avoid disrupting the reasoning chain.

## Method

### Overall Framework Architecture

```
+-------------------------------------------------------------------+
|                    Search-o1 Framework                             |
|                                                                   |
|  User Query                                                       |
|      |                                                            |
|      v                                                            |
|  [LRM Reasoning Chain]                                            |
|      |                                                            |
|      +---> Step 1: "I know that..." (confident) --> continue      |
|      |                                                            |
|      +---> Step 2: "I need to verify..." (uncertain)              |
|      |         |                                                  |
|      |         v                                                  |
|      |    [SEARCH TRIGGER]                                        |
|      |         |                                                  |
|      |         v                                                  |
|      |    +---------------------------+                           |
|      |    | Agentic Search Module     |                           |
|      |    | - Query formulation       |                           |
|      |    | - Web/knowledge retrieval  |                           |
|      |    | - Document collection      |                           |
|      |    +---------------------------+                           |
|      |         |                                                  |
|      |         v                                                  |
|      |    +---------------------------+                           |
|      |    | Reason-in-Documents (RiD) |                           |
|      |    | - Deep document analysis   |                           |
|      |    | - Noise filtering          |                           |
|      |    | - Knowledge extraction     |                           |
|      |    +---------------------------+                           |
|      |         |                                                  |
|      |         v                                                  |
|      |    [Distilled knowledge injected back into reasoning]      |
|      |                                                            |
|      +---> Step 3: Reasoning continues with retrieved knowledge   |
|      |                                                            |
|      +---> ... (repeat search triggers as needed)                 |
|      |                                                            |
|      v                                                            |
|  Final Answer                                                     |
+-------------------------------------------------------------------+
```

### Agentic Search Workflow

The search component is triggered inline during the model's reasoning process:

```
Algorithm: Agentic Search Integration
---------------------------------------
Input:  Query Q, LRM with reasoning capability
Output: Answer A with grounded reasoning chain

1:  reasoning_chain = []
2:  state = START_REASONING(Q)
3:
4:  while not state.is_terminal():
5:      next_step = LRM.generate_next_step(state)
6:
7:      if DetectUncertainty(next_step):
8:          // Model signals knowledge gap
9:          search_query = FormulateQuery(next_step, Q)
10:         documents = RetrieveDocuments(search_query)  // Web search / KB
11:
12:         // Reason-in-Documents module
13:         distilled = RiD(documents, next_step, Q)
14:
15:         // Inject distilled knowledge back
16:         state.inject_knowledge(distilled)
17:         next_step = LRM.regenerate_step(state)
18:     end if
19:
20:     reasoning_chain.append(next_step)
21:     state.update(next_step)
22: end while
23:
24: A = ExtractAnswer(reasoning_chain)
25: return A
```

### Reason-in-Documents (RiD) Module

The RiD module is the key technical contribution that distinguishes Search-o1 from naive RAG:

```
Algorithm: Reason-in-Documents (RiD)
--------------------------------------
Input:  Retrieved documents D = {d_1, ..., d_k},
        Current reasoning context C,
        Original query Q
Output: Distilled knowledge K suitable for reasoning injection

1:  // Step 1: Document-level analysis
2:  for each document d_i in D:
3:      relevance_i = AssessRelevance(d_i, C, Q)
4:      key_facts_i = ExtractKeyFacts(d_i, C)
5:  end for
6:
7:  // Step 2: Cross-document reasoning
8:  consistent_facts = CrossValidate(all key_facts)
9:
10: // Step 3: Reasoning-aligned synthesis
11: K = Synthesize(consistent_facts, C)
12: // K is formatted to seamlessly continue the reasoning chain
13:
14: return K
```

The RiD module performs deep analysis rather than naive concatenation:
- **Relevance filtering**: Removes documents and passages irrelevant to the current reasoning step
- **Fact extraction**: Identifies specific facts, formulas, and data points needed
- **Cross-document reasoning**: Validates information consistency across multiple sources
- **Reasoning-aligned formatting**: Synthesizes extracted knowledge into a format that naturally
  continues the existing chain-of-thought, minimizing disruption to reasoning coherence

### Uncertainty Detection

Search triggers are activated when the reasoning model exhibits signals of knowledge uncertainty:
- Hedging language ("I think", "possibly", "I'm not sure")
- Self-contradiction within the reasoning chain
- Explicit knowledge-seeking statements ("I need to know", "Let me check")
- Low confidence indicators in the reasoning steps

## Key Innovations

1. **Inline agentic search**: Unlike standard RAG (retrieve-then-generate), Search-o1 triggers
   retrieval dynamically during the reasoning process itself, precisely when knowledge gaps appear.
2. **Reason-in-Documents module**: Deep analysis of retrieved documents before injection prevents
   the noise and coherence disruption that plagues naive RAG approaches applied to reasoning models.
3. **Reasoning-preserving injection**: Retrieved knowledge is formatted to seamlessly continue the
   chain-of-thought rather than being dumped as raw context, maintaining reasoning coherence.
4. **Self-aware knowledge gaps**: The model itself determines when to search, based on its own
   uncertainty signals, creating a more natural and efficient retrieval pattern.
5. **Applicable to existing LRMs**: Works as an augmentation to existing reasoning models (QwQ,
   DeepSeek-R1) without requiring model retraining.

## Experimental Setup

- **Base models**: QwQ-32B-Preview (primary), with comparisons to other reasoning models
- **Benchmarks**:
  - Complex reasoning: GPQA (graduate-level science QA), MATH (mathematical reasoning),
    LiveCodeBench (coding), ARC-Challenge (science reasoning)
  - Open-domain QA: TriviaQA, Natural Questions (NQ), HotpotQA, 2WikiMultiHopQA,
    MuSiQue, Bamboogle (single-hop and multi-hop QA)
- **Baselines**: Vanilla QwQ-32B, QwQ + naive RAG, RAgent-QwQ-32B, standard retrieval-augmented
  approaches
- **Metrics**: Accuracy (complex reasoning), F1/EM scores (open-domain QA)

## Results

### Complex Reasoning Tasks

Search-o1 with QwQ-32B-Preview backbone results:

| Benchmark       | QwQ-32B (vanilla) | QwQ + naive RAG | Search-o1    |
|-----------------|:-----------------:|:---------------:|:------------:|
| GPQA (extended) | ~54%              | degraded        | **57.9%**    |
| GPQA Physics    | --                | --              | **68.7%**    |
| GPQA Biology    | --                | --              | **69.5%**    |

Key finding: Naive RAG actually *degrades* reasoning model performance on complex tasks because
noisy retrieved documents disrupt the reasoning chain. Search-o1's RiD module prevents this.

### Open-Domain QA Tasks (6 benchmarks)

| Method               | Avg across 6 QA benchmarks |
|----------------------|:--------------------------:|
| QwQ-32B (vanilla)    | Baseline                   |
| RAgent-QwQ-32B       | Baseline + X               |
| **Search-o1**        | **+3.1% over QwQ-32B**     |
|                      | **+4.7% over RAgent**      |

Search-o1 outperforms both vanilla QwQ-32B and RAgent-QwQ-32B across the majority of open-domain
QA benchmarks, demonstrating that the RiD module provides consistent improvements.

### Ablation: Impact of Reason-in-Documents

| Configuration             | Performance delta vs vanilla |
|---------------------------|:----------------------------:|
| QwQ + naive RAG           | Sometimes negative (noise)   |
| QwQ + Search (no RiD)     | Modest improvement           |
| QwQ + Search + RiD        | **Best performance**         |

The RiD module is critical: without it, simply adding search to reasoning models yields
inconsistent results and can even hurt performance.

## Limitations

1. **Latency overhead**: Each search trigger adds retrieval and RiD processing latency to an
   already lengthy reasoning chain, potentially doubling or tripling total inference time.
2. **Search quality dependency**: Performance depends on the quality of retrieved documents; poor
   search results lead to poor or misleading knowledge injection.
3. **Uncertainty detection heuristics**: The mechanism for detecting when to trigger search is
   based on heuristic signals (hedging language, etc.) that may miss genuine knowledge gaps or
   trigger unnecessarily on rhetorical hedging.
4. **Single base model evaluation**: Primary results are on QwQ-32B-Preview only; generalization
   to other reasoning models (DeepSeek-R1, OpenAI-o1) is not extensively validated.
5. **Computational cost**: Multiple search-and-reason cycles per query make the approach
   significantly more expensive than single-pass reasoning.
6. **No training of RiD**: The RiD module relies on prompted reasoning rather than trained
   document analysis, leaving potential for improvement through specialized fine-tuning.
7. **Comparison gap**: Direct comparison with commercial systems (OpenAI-o1 with browsing) is
   limited due to API access constraints.

## Key Takeaways

1. **Naive RAG hurts reasoning models**: Simply prepending retrieved documents to reasoning model
   inputs can degrade performance by introducing noise that disrupts chain-of-thought coherence.
   This is a critical cautionary finding for the RAG community.
2. **Reason-in-Documents is essential**: Deep analysis of retrieved content before injection is
   the key innovation -- it bridges the gap between retrieval and reasoning by ensuring only
   clean, relevant knowledge enters the reasoning chain.
3. **Inline search > pre-search**: Triggering retrieval dynamically during reasoning (when
   uncertainty is detected) is more effective than retrieving before reasoning begins, because
   the model can identify precisely what knowledge it lacks.
4. **Complementary to reasoning scaling**: Search-o1 is orthogonal to improvements in reasoning
   model training (RL, long CoT) -- it augments any existing reasoning model with knowledge
   access, suggesting the two approaches are complementary.
5. **Foundation for search-augmented reasoning**: Search-o1, along with concurrent work like
   WebThinker, establishes the paradigm of integrating agentic search directly into the reasoning
   loop, influencing subsequent systems like Search-R1 and Tongyi DeepResearch.
