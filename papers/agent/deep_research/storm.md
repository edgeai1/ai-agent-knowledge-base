---
title: "Assisting in Writing Wikipedia-like Articles From Scratch with Large Language Models (STORM)"
authors:
  - Yijia Shao
  - Yucheng Jiang
  - Theodore A. Kanell
  - Peter Xu
  - Omar Khattab
  - Monica S. Lam
venue: NAACL 2024
year: 2024
url: https://arxiv.org/abs/2402.14207
tags:
  - article-generation
  - multi-perspective-qa
  - knowledge-curation
  - Wikipedia
  - outline-generation
  - retrieval-augmented
status: done
---

# STORM: Synthesis of Topic Outlines through Retrieval and Multi-perspective Question Asking

## TL;DR

STORM is an LLM-powered system from Stanford that generates comprehensive, Wikipedia-like articles
from scratch by simulating the pre-writing research process. It discovers diverse perspectives on
a topic, simulates multi-perspective conversations with a grounded topic expert, and curates the
information into a structured outline before generating the full article. Evaluated on FreshWiki
(100 recent Wikipedia articles), STORM produces articles with +25% better organization and +10%
broader coverage than RAG baselines, as judged by experienced Wikipedia editors.

## Motivation & Problem

Writing high-quality long-form articles (like Wikipedia entries) requires extensive pre-writing
research that involves:
1. **Discovering what to write about**: Understanding the full scope of a topic.
2. **Finding diverse perspectives**: Gathering information from multiple viewpoints.
3. **Organizing information**: Creating a coherent outline that structures the article.

Existing LLM approaches either:
- Generate articles directly (hallucination-prone, shallow coverage).
- Use simple RAG (retrieves once, no iterative deepening or perspective diversity).
- Require human-in-the-loop pre-writing (not scalable).

STORM automates the entire pre-writing pipeline to produce articles with breadth and depth
comparable to human-authored Wikipedia entries.

## Method

### Pipeline Overview

```
+-----------+     +------------------+     +---------------+     +-----------+
| Step 1:   |     | Step 2:          |     | Step 3:       |     | Step 4:   |
| Perspective|---->| Multi-Perspective|---->| Outline       |---->| Article   |
| Discovery |     | Conversation     |     | Generation    |     | Generation|
|           |     | Simulation       |     |               |     |           |
+-----------+     +------------------+     +---------------+     +-----------+
     |                    |                       |                    |
  Find diverse       Simulate Q&A with       Curate collected     Generate full
  viewpoints on      grounded expert for     info into a          article from
  the topic          each perspective        structured outline   outline + refs
```

### Step 1: Perspective Discovery

```
Algorithm: Discover Diverse Perspectives
-------------------------------------------------
Input: Topic T
Output: Set of perspectives P = {p_1, p_2, ..., p_k}

1. Search for related Wikipedia articles on topic T
2. Identify the types of authors, experts, or stakeholders
   who would write about T from different angles
3. Extract k diverse perspectives (e.g., historian, scientist,
   policy-maker, affected community member)
4. Each perspective p_i carries specific prior knowledge and interests
```

The system automatically mines perspectives from relevant existing Wikipedia articles, identifying
the range of viewpoints that a comprehensive article should address.

### Step 2: Multi-Perspective Conversation Simulation

```
Algorithm: Simulated Expert Conversation
-------------------------------------------------
Input: Topic T, perspective p_i, trusted web sources S
Output: Conversation transcript C_i with cited references

1. Initialize Writer agent with perspective p_i
2. Initialize Expert agent grounded in web sources S
3. for each conversation turn:
4.   Writer(p_i) generates a question based on:
5.     - Their perspective's interests and prior knowledge
6.     - Answers received so far (conversational context)
7.   Expert retrieves relevant passages from S
8.   Expert generates a grounded answer with citations
9.   Writer updates understanding, formulates follow-up
10. Return transcript C_i = [(q_1, a_1), ..., (q_m, a_m)]
```

Key insight: New questions arise naturally as answers update the writer's understanding, mirroring
how human researchers iteratively deepen their knowledge through dialogue.

### Step 3: Outline Generation

```
Algorithm: Information Curation -> Outline
-------------------------------------------------
Input: All conversation transcripts {C_1, ..., C_k}
Output: Hierarchical article outline O

1. Aggregate all Q&A pairs across all perspective conversations
2. Cluster related information into thematic groups
3. Generate hierarchical section structure:
   - Top-level sections (major themes)
   - Subsections (specific aspects)
   - Bullet points (key facts to cover)
4. Map citations from conversations to outline sections
5. Ensure balanced coverage across perspectives
```

### Step 4: Full Article Generation

- Generate each section following the outline using the curated information.
- Include inline citations from the grounded conversations.
- Ensure coherent transitions between sections.

## Key Innovations

1. **Perspective-Guided Question Asking**: Rather than asking generic questions, STORM includes
   specific perspectives in the prompt to provide focus, prior knowledge, and diversity of
   viewpoints. This is critical for achieving broad topic coverage.

2. **Simulated Conversations for Research**: The conversational format mirrors how human
   researchers learn -- new questions emerge from answers, enabling iterative deepening that
   static retrieval cannot achieve.

3. **Separation of Pre-writing and Writing**: By explicitly modeling the research/outlining
   phase before generation, STORM produces more organized articles than end-to-end approaches.

4. **FreshWiki Benchmark**: A carefully curated evaluation dataset of 100 high-quality Wikipedia
   articles from Feb 2022 - Sep 2023, designed to avoid data contamination from LLM pre-training.

## Experimental Setup

### FreshWiki Dataset

- **Size**: 100 high-quality Wikipedia articles.
- **Source**: Most-edited Wikipedia pages from February 2022 to September 2023.
- **Purpose**: Ensures topics are recent enough to avoid pre-training data leakage.
- **Quality filter**: Only articles meeting Wikipedia's quality standards included.

### Evaluation Dimensions

| Dimension      | What It Measures                                | Evaluator         |
|---------------|------------------------------------------------|-------------------|
| Organization  | Outline quality, section structure              | Wikipedia editors |
| Coverage      | Breadth of topic aspects addressed              | Wikipedia editors |
| Relevance     | Information pertinence to the topic             | Automatic + human |
| Citation quality| Accuracy and appropriateness of references     | Manual inspection |

### Baselines

- **Outline-driven RAG baseline**: Retrieve once, generate outline, write article.
- **Direct generation**: LLM generates article without explicit pre-writing stage.
- **Retrieval-only**: Standard RAG without perspective simulation or outlining.

### LLMs Used

- GPT-3.5-turbo and GPT-4 as backbone LLMs for the pipeline stages.

## Results

### Article Quality (Expert Wikipedia Editor Evaluation)

| Metric        | STORM vs. Baseline Improvement |
|--------------|-------------------------------|
| Organization | **+25% absolute increase**     |
| Coverage     | **+10% absolute increase**     |

- Expert Wikipedia editors consistently rated STORM articles as better organized and broader in
  coverage than the outline-driven RAG baseline.
- The multi-perspective conversation approach was the primary driver of coverage improvements.
- Outline quality improvements directly translated to better-organized final articles.

### Ablation Findings

- Removing perspective diversity degrades coverage significantly.
- Removing the conversational format (using single-turn Q&A instead) reduces depth.
- The outline generation stage is critical for organization quality.

## Limitations

1. **Internet bias propagation**: Articles inherit biases present in web sources; over-represented
   viewpoints may dominate generated content.
2. **Fabricated connections**: LLMs sometimes create plausible but incorrect connections between
   unrelated facts discovered during research.
3. **Scalability to niche topics**: Performance depends on availability of diverse web sources;
   obscure topics may have insufficient source material.
4. **Citation reliability**: While grounded in web sources, the system may sometimes misattribute
   or misinterpret source content.
5. **Evaluation scope**: FreshWiki contains 100 articles; larger-scale evaluation needed for
   statistical robustness.

## Follow-up Work

- **Co-STORM** (EMNLP 2024): Extends STORM to collaborative human-AI knowledge curation where
  users observe and steer conversations among LM agents, enabling discovery of "unknown unknowns."
- **STORM v2**: Improved pipeline with better retrieval and outline refinement.
- **WebThinker**: Extends the idea to integrated reasoning + search + drafting in a single loop.
- **Deep research agents**: STORM's perspective-based research approach influenced later systems
  like DeepResearcher and OpenResearcher.

## Key Takeaways

1. Explicitly modeling the pre-writing research process (perspective discovery, expert
   conversation, outline curation) produces dramatically better long-form articles than
   end-to-end generation.
2. Multi-perspective question asking is essential for achieving broad topic coverage; different
   viewpoints naturally surface different aspects of a topic.
3. Simulated conversations enable iterative deepening of knowledge that static retrieval cannot
   match, because follow-up questions emerge from prior answers.
4. The FreshWiki benchmark provides a rigorous, contamination-resistant evaluation framework for
   article generation systems.
5. Open-sourced at github.com/stanford-oval/storm, STORM has become a foundational system in
   the deep research pipeline ecosystem.
