---
title: "OpenScholar: Synthesizing Scientific Literature with Retrieval-Augmented LMs"
authors:
  - Akari Asai
  - Jacqueline He
  - Rulin Shao
  - Weijia Shi
  - Amanpreet Singh
  - Joseph Chee Chang
  - Kyle Lo
  - Luca Soldaini
  - Sergey Feldman
  - Mike D'Arcy
  - David Wadden
  - Matt Latzke
  - Minyang Tian
  - Pan Ji
  - Shengyan Liu
  - Hao Tong
  - Bohao Wu
  - Yanyu Xiong
  - Luke Zettlemoyer
  - Graham Neubig
  - Dan Weld
  - Doug Downey
  - Hannaneh Hajishirzi
venue: Nature (2026)
year: 2026
url: https://arxiv.org/abs/2411.14199
tags:
  - scientific-literature
  - retrieval-augmented-generation
  - citation-accuracy
  - hallucination-reduction
  - open-source
  - Semantic-Scholar
status: done
---

# OpenScholar: Synthesizing Scientific Literature with Retrieval-Augmented LMs

## TL;DR

OpenScholar is a retrieval-augmented LM specialized for scientific literature synthesis. It
searches a datastore of 45M open-access papers (236M passage embeddings) from Semantic Scholar,
generating citation-backed responses. While GPT-4o hallucinates 78-90% of citations, OpenScholar
achieves citation accuracy on par with human experts. In expert evaluations (16 PhD-level),
OpenScholar-8B responses were preferred over expert-written ones 51% of the time, and the
GPT-4o-augmented variant 70% of the time. Published in Nature (2026), developed by AI2 and UW.

## Motivation & Problem

Scientific literature synthesis is critical for researchers but existing LLMs fail catastrophically:

1. **Citation hallucination**: GPT-4o fabricates citations 78-90% of the time, making its
   scientific outputs unreliable and potentially harmful to research.
2. **Scale challenge**: Over 45 million open-access papers exist; no LLM can internalize this
   knowledge during pre-training alone.
3. **Multi-paper synthesis**: Answering scientific questions often requires synthesizing findings
   across multiple papers, not just retrieving a single document.
4. **Cost**: Commercial API-based solutions (e.g., PaperQA2) are expensive for routine use.

OpenScholar addresses these by building a specialized retrieval-augmented pipeline over a massive
scientific literature datastore with trained retrievers and a fine-tuned 8B model.

## Method

### System Architecture

```
+-------------------+     +--------------------+     +------------------+
| User Query        |     | OpenScholar        |     | Response with    |
| (scientific       |---->| DataStore (OSDS)   |---->| Inline Citations |
|  question)        |     | 45M papers         |     | [1], [2], ...    |
+-------------------+     | 236M passages      |     +------------------+
                          +--------------------+             ^
                                  |                          |
                                  v                          |
                          +--------------------+     +------------------+
                          | Trained Retriever  |     | OpenScholar-8B   |
                          | (passage retrieval)|---->| or               |
                          | BM25 + dense       |     | OpenScholar-GPT4o|
                          +--------------------+     | (synthesis LM)   |
                                                     +------------------+
                                                             |
                                                     +------------------+
                                                     | Self-Refinement  |
                                                     | (iterative       |
                                                     |  retrieval +     |
                                                     |  answer update)  |
                                                     +------------------+
```

### OpenScholar DataStore (OSDS)

- **Source**: Semantic Scholar open-access papers.
- **Scale**: 45 million papers, 236 million passage embeddings.
- **Coverage**: Computer science, physics, neuroscience, biomedicine, and more.
- **Indexing**: Dense passage embeddings for efficient similarity search + BM25 for keyword match.

### Retrieval Pipeline

```
Algorithm: Multi-Stage Retrieval
-------------------------------------------------
Input: Query q
Output: Ranked passages P = {p_1, ..., p_k}

1. Encode query q with trained bi-encoder
2. Retrieve top-N candidates via approximate nearest neighbor search
   over 236M passage embeddings
3. Re-rank candidates using cross-encoder or BM25 fusion
4. Return top-k most relevant passages with paper metadata
```

### OpenScholar-8B Training

```
Training Pipeline for OpenScholar-8B:
-------------------------------------------------
Base model: Llama 3.1 8B

1. Sample passages from the datastore
2. Generate synthetic queries and instructions from sampled passages
3. Create (query, retrieved_passages, gold_response) training triples
4. Fine-tune via standard next-token prediction on:
   - Query understanding
   - Multi-passage synthesis
   - Citation generation (inline references to retrieved passages)
5. Train retrieval models on the same data distribution
```

### Self-Refinement Loop

```
Algorithm: Iterative Self-Refinement
-------------------------------------------------
1. Generate initial response R_0 with citations from retrieved passages
2. Identify claims in R_0 that lack sufficient evidence
3. Issue follow-up retrieval queries for under-supported claims
4. Retrieve additional passages
5. Revise response: R_1 = refine(R_0, new_passages)
6. Repeat if needed until citation coverage is satisfactory
7. Return final response R_final
```

## Key Innovations

1. **Massive scientific datastore**: 45M papers / 236M passages from Semantic Scholar, providing
   near-comprehensive coverage of open-access scientific literature.

2. **Citation accuracy matching human experts**: First system to achieve expert-level citation
   accuracy in scientific literature synthesis, solving the 78-90% hallucination rate of GPT-4o.

3. **Compact open-weight model**: OpenScholar-8B (fine-tuned Llama 3.1 8B) outperforms GPT-4o
   despite being far smaller, demonstrating that domain specialization beats model scale.

4. **Self-refinement with iterative retrieval**: The model identifies its own under-cited claims
   and retrieves additional evidence, improving citation coverage iteratively.

5. **ScholarQABench**: First large-scale, multi-domain benchmark for scientific literature
   synthesis with expert-written queries and answers.

## Experimental Setup

### ScholarQABench Benchmark

| Property         | Details                                              |
|-----------------|------------------------------------------------------|
| Total queries   | 2,967 expert-written                                 |
| Long-form answers| 208 expert-written                                  |
| Domains         | Computer Science, Physics, Neuroscience, Biomedicine |
| Evaluation      | Automatic metrics + human expert evaluation          |

### Evaluation Metrics

- **Correctness**: Factual accuracy of synthesized responses.
- **Citation F1**: Precision and recall of cited references against gold references.
- **Coverage**: Proportion of relevant aspects addressed.
- **Relevance**: Pertinence of response to the query.
- **Organization**: Structural quality of the response.
- **Usefulness**: Overall utility as judged by domain experts.

### Baselines

- **GPT-4o**: State-of-the-art commercial LLM (without specialized retrieval).
- **PaperQA2**: Prior specialized system for scientific question answering.
- **Perplexity Pro**: Commercial search-augmented system.
- **Expert-written responses**: Human expert baselines for preference evaluation.

### Human Evaluation

- **Evaluators**: 16 PhD-level domain experts.
- **Protocol**: Side-by-side preference comparison (blind evaluation).

## Results

### Automated Evaluation

| Model              | Correctness | Citation F1 |
|-------------------|-------------|-------------|
| OpenScholar-8B    | GPT-4o + 6.1% | High     |
| OpenScholar-GPT4o | GPT-4o + 12%  | 39.5     |
| GPT-4o (vanilla)  | baseline    | 0.1        |
| PaperQA2          | OS-8B - 5.5%| --         |

- **OpenScholar-8B outperforms GPT-4o by 6.1%** in correctness despite being far smaller.
- **OpenScholar-GPT4o** improves GPT-4o's citation F1 from 0.1 to 39.5 (395x improvement).
- **100x more cost-efficient** than PaperQA2.

### Citation Hallucination

| Model           | Citation Hallucination Rate |
|----------------|---------------------------|
| GPT-4o         | 78-90%                    |
| OpenScholar-8B | Expert-level accuracy     |

### Human Expert Preferences (vs. Expert-Written)

| Model              | Preferred Over Experts |
|-------------------|----------------------|
| OpenScholar-GPT4o | **70%** of the time  |
| OpenScholar-8B    | **51%** of the time  |
| GPT-4o (vanilla)  | 32% of the time      |

## Limitations

1. **Open-access bias**: The 45M paper datastore only covers open-access papers; proprietary
   journal articles are excluded, potentially missing important literature.
2. **Temporal lag**: The datastore requires periodic updating; very recent papers may not be
   indexed.
3. **Domain coverage**: While multi-domain, performance may vary for niche subfields with
   limited representation in the datastore.
4. **Retrieval bottleneck**: Response quality is bounded by retriever recall; if relevant papers
   are not retrieved, they cannot be cited.
5. **Single-language**: Primarily English-language papers; non-English scientific literature
   is under-represented.

## Follow-up Work

- Integration with STORM-like systems for generating full survey articles from literature.
- Extension to proprietary literature (with appropriate access controls).
- Multimodal scientific synthesis (figures, tables, equations).
- Real-time datastore updating for emerging research areas.
- Application to systematic reviews and meta-analyses.

## Key Takeaways

1. Citation hallucination is a critical unsolved problem in general-purpose LLMs (78-90% for
   GPT-4o); specialized retrieval-augmented systems can solve it.
2. A fine-tuned 8B model with domain-specific retrieval outperforms GPT-4o on scientific
   synthesis, demonstrating that architecture design matters more than raw scale.
3. Expert-level scientific literature synthesis is achievable with open-source models, making
   it accessible to the broader research community.
4. Self-refinement with iterative retrieval is essential for achieving comprehensive citation
   coverage in multi-paper synthesis tasks.
5. Published in Nature (2026), OpenScholar validates that RAG-based approaches can produce
   research outputs trusted by domain experts at scale.
