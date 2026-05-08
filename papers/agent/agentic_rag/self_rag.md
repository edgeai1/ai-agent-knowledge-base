---
title: "Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection"
authors:
  - Akari Asai
  - Zeqiu Wu
  - Yizhong Wang
  - Avirup Sil
  - Hannaneh Hajishirzi
venue: ICLR 2024 (Oral, top 1%)
year: 2024
url: https://arxiv.org/abs/2310.11511
tags:
  - self-rag
  - retrieval-augmented-generation
  - reflection-tokens
  - self-reflection
  - adaptive-retrieval
  - hallucination-reduction
  - ICLR
status: done
---

## TL;DR

Self-RAG trains a single LM to adaptively retrieve passages on-demand and self-evaluate
both retrieval quality and generation faithfulness using special reflection tokens. Unlike
standard RAG that always retrieves a fixed number of passages, Self-RAG decides *when* to
retrieve (or skip retrieval entirely) and *how* to critique retrieved content through four
types of reflection tokens: Retrieve, ISREL, ISSUP, and ISUSE. Trained on Llama2-7B/13B,
Self-RAG outperforms ChatGPT and retrieval-augmented Llama2-chat on open-domain QA,
reasoning, and fact verification while achieving the lowest hallucination rates among
evaluated RAG approaches (~5.8%).

## Motivation & Problem

Standard RAG approaches suffer from three key limitations:

1. **Indiscriminate retrieval**: Always retrieving K passages regardless of query type
   wastes compute on easy queries and may inject noise for queries the model already knows.
2. **No retrieval quality assessment**: Retrieved passages are blindly fed to the generator
   without checking relevance, potentially misleading the model.
3. **No generation faithfulness check**: The model has no mechanism to verify whether its
   generation is actually supported by the retrieved evidence.

Prior approaches to address these issues either require separate models (e.g., a retrieval
classifier + a NLI model) or use reinforcement learning (RLHF), which is unstable and
memory-intensive. Self-RAG unifies all these capabilities into a single model through
special tokens, trained with standard next-token prediction.

## Method

### Reflection Token Types

Self-RAG introduces four special token types appended to the model's vocabulary:

```
+------------------------------------------------------------------+
|  Token       | When Emitted        | Possible Values              |
+------------------------------------------------------------------+
|  Retrieve    | Before generation   | {yes, no, continue}          |
|              | segment             | Decides whether to retrieve   |
+------------------------------------------------------------------+
|  ISREL       | After retrieval     | {relevant, irrelevant}       |
|              |                     | Is passage relevant to query? |
+------------------------------------------------------------------+
|  ISSUP       | After generation    | {fully_supported,            |
|              | segment             |  partially_supported,         |
|              |                     |  no_support}                  |
|              |                     | Is output supported by passage|
+------------------------------------------------------------------+
|  ISUSE       | After complete      | {1, 2, 3, 4, 5}              |
|              | response            | Overall utility rating        |
+------------------------------------------------------------------+
```

### Training Pipeline (4 Steps)

```
Step 1: CRITIC DATA CREATION
+------------------+     +------------------+
| Input-output     | --> | GPT-4 annotates  | --> Reflection token
| pairs from       |     | with reflection  |     training data
| diverse sources  |     | token labels     |     (4K-20K per type)
+------------------+     +------------------+

Step 2: CRITIC MODEL TRAINING
+------------------+     +------------------+
| Reflection token | --> | Train Critic     | --> Critic_LM
| training data    |     | (expand vocab    |     (can label any
|                  |     |  with special    |     input with
|                  |     |  tokens, train   |     reflection tokens)
|                  |     |  next-token pred)|
+------------------+     +------------------+

Step 3: GENERATOR DATA CREATION
+------------------+     +------------------+     +------------------+
| 150K instruction | --> | Retriever adds   | --> | Critic_LM adds   |
| -output pairs    |     | relevant passages|     | reflection tokens |
|                  |     | where needed     |     | inline            |
+------------------+     +------------------+     +------------------+
                                                         |
                                                         v
                                                  Training data with
                                                  interleaved passages
                                                  + reflection tokens

Step 4: GENERATOR TRAINING
+------------------+     +------------------+
| Augmented data   | --> | Train Generator  | --> Self-RAG_LM
| with passages +  |     | (standard next-  |     (single model that
| reflection tokens|     |  token prediction|      retrieves, generates,
|                  |     |  loss over all   |      and critiques)
|                  |     |  tokens incl.    |
|                  |     |  reflection)     |
+------------------+     +------------------+
```

### Inference Algorithm

```
SELF_RAG_INFERENCE(query):
    segments = []
    context = query

    while not DONE:
        # Step 1: Decide whether to retrieve
        retrieve_token = model.generate_next_token(context)

        if retrieve_token == "yes":
            # Step 2: Retrieve passages
            passages = retriever.retrieve(query, top_k=5)

            # Step 3: Generate candidate segments for each passage
            candidates = []
            for p in passages:
                segment = model.generate(context + p)
                isrel  = model.predict_token(ISREL, p, query)
                issup  = model.predict_token(ISSUP, segment, p)
                isuse  = model.predict_token(ISUSE, segment, query)
                candidates.append((segment, isrel, issup, isuse))

            # Step 4: Rank and select best segment
            best = rank_by_weighted_score(candidates,
                       w_rel, w_sup, w_use)  # weights are tunable
            segments.append(best.segment)

        elif retrieve_token == "no" or "continue":
            # Generate without retrieval
            segment = model.generate(context)
            segments.append(segment)

        context = context + segments[-1]

    return join(segments)
```

### Key Design Choices

1. **Offline token insertion**: Reflection tokens are inserted by the Critic during
   training data creation, not predicted during training. This makes training as simple
   as standard next-token prediction (no RL needed).

2. **Segment-level granularity**: The model generates output in segments, deciding
   retrieval and critique at each segment boundary rather than per-token.

3. **Parallel candidate evaluation**: At inference, multiple passages are evaluated in
   parallel, and the best-supported generation is selected.

4. **Tunable critique weights**: The weights on ISREL, ISSUP, and ISUSE can be adjusted
   at inference time to trade off factuality vs. fluency without retraining.

## Key Innovations

1. **Unified retrieve-generate-critique model**: Single model handles all three functions
   through special tokens, eliminating the need for separate retrieval classifiers, NLI
   models, or reward models.

2. **On-demand retrieval**: The model learns to retrieve only when beneficial, skipping
   retrieval for queries it can answer from parametric knowledge. This reduces latency
   and avoids noise injection.

3. **Training efficiency**: By using offline token insertion instead of RL, Self-RAG
   training is as stable and memory-efficient as standard supervised fine-tuning.

4. **Inference-time controllability**: Critique weights can be tuned at inference time
   to shift the model's behavior along the factuality-creativity spectrum.

## Experimental Setup

- **Model sizes**: Llama2-7B and Llama2-13B
- **Training data**: 150K instruction-output instances augmented with reflection tokens
- **Critic training data**: 4K-20K instances per reflection token type, annotated by GPT-4
- **Retriever**: Contriever-MS MARCO
- **Benchmarks**:
  - Open-domain QA: PopQA, TriviaQA (accuracy)
  - Reasoning/Fact verification: PubHealth, ARC-Challenge (accuracy)
  - Long-form generation: ASQA (citation precision/recall, correctness), Biography
    generation (FactScore)
- **Baselines**: Llama2, Alpaca, ChatGPT, retrieval-augmented Llama2-chat, Ret-ChatGPT,
  standard RAG approaches

## Results

### Short-Form Tasks (Accuracy %)

| Model                    | PopQA | TriviaQA | PubHealth | ARC-C |
|-------------------------|-------|----------|-----------|-------|
| Llama2-7B (no retrieval)| ~21   | ~57      | ~60       | ~68   |
| Llama2-13B (no retrieval)| ~24  | ~63      | ~63       | ~72   |
| Alpaca-13B              | ~31   | ~55      | ~54       | ~55   |
| ChatGPT                 | ~29   | ~65      | ~70       | ~75   |
| Ret-ChatGPT             | ~51.8 | ~63      | ~54       | --    |
| Self-RAG-7B             | 54.9  | 68.0     | 72.4      | --    |
| Self-RAG-13B            | 55.8  | 69.3     | 74.5      | --    |

### Long-Form Generation

| Model            | ASQA (str-em) | ASQA (cite prec) | ASQA (cite recall) | FactScore |
|-----------------|---------------|-------------------|--------------------|-----------|
| Llama2-chat-13B | --            | --                | --                 | ~40       |
| ChatGPT         | --            | --                | --                 | ~58       |
| Self-RAG-13B    | --            | 70.3              | 71.3               | ~81       |

### Hallucination Rate

Self-RAG achieves approximately **5.8% hallucination rate**, the lowest among 12 RAG
variants evaluated -- significantly lower than standard agentic pipelines (12-14%).

## Limitations

1. **Training data dependency on GPT-4**: Critic training data is generated by GPT-4,
   creating a dependency on a proprietary model and potentially inheriting its biases.

2. **Segment-level granularity**: Fixed segment boundaries may not align with natural
   retrieval decision points; some queries need finer or coarser granularity.

3. **Retriever coupling**: The trained model is coupled to the specific retriever used
   during training (Contriever). Swapping retrievers may degrade performance.

4. **Inference cost**: Evaluating multiple candidate passages in parallel increases
   inference compute and latency, particularly when top_k is large.

5. **Limited multi-hop**: While Self-RAG supports multiple retrieval steps, it does not
   explicitly decompose complex queries into sub-questions, limiting multi-hop reasoning
   compared to chain-of-thought or interleaved approaches.

6. **Model scale**: Evaluated only on Llama2-7B/13B; effectiveness at larger scales
   and with newer model architectures is not established.

## Key Takeaways

1. Special reflection tokens are a remarkably simple yet effective mechanism for teaching
   a model to self-regulate retrieval and generation. The approach avoids the complexity
   and instability of RLHF while achieving strong self-critique capabilities.

2. On-demand retrieval is strictly superior to always-retrieve or never-retrieve
   strategies. The model learns meaningful retrieval patterns -- skipping retrieval for
   well-known facts and triggering it for long-tail or ambiguous queries.

3. The separation of training (offline token insertion) and inference (parallel candidate
   evaluation) is an elegant design that keeps training simple while enabling rich
   inference-time behavior.

4. Self-RAG's FactScore improvements on biography generation demonstrate that
   self-reflection tokens are particularly valuable for reducing hallucination in
   knowledge-intensive generation tasks.

5. The tunable inference-time weights (w_rel, w_sup, w_use) provide a practical
   mechanism for deploying a single model across applications with different
   factuality-creativity requirements, without retraining.

6. Self-RAG represents a foundational step toward the broader Agentic RAG paradigm,
   where the model is not just a consumer of retrieved passages but an active decision-
   maker about when and how to use external knowledge.
