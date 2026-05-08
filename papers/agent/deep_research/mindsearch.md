---
title: "MindSearch: Mimicking Human Minds Elicits Deep AI Searcher"
authors: "Zehui Chen, Kuikun Liu, Qiuchen Wang, Jiangning Liu, Wenwei Zhang, Kai Chen, Feng Zhao"
venue: "arXiv preprint"
year: 2024
url: "https://arxiv.org/abs/2407.20183"
code: "https://github.com/InternLM/MindSearch"
tags: [multi-agent, web-search, information-retrieval, graph-reasoning, deep-search, cognitive-search]
category: agent/deep_research
status: done
date_read: 2026-05-08
---

# MindSearch: Mimicking Human Minds Elicits Deep AI Searcher

## TL;DR

MindSearch is a multi-agent framework for deep web information seeking that mimics human cognitive
search processes. It decomposes complex queries into a dynamic directed acyclic graph (DAG) of
atomic sub-questions via a WebPlanner agent, then dispatches parallel WebSearcher agents to retrieve
and summarize information from 300+ web pages in under 3 minutes -- work equivalent to ~3 hours of
human effort. Human evaluators prefer MindSearch responses (even with the 7B InternLM2.5 backbone)
over ChatGPT-Web and Perplexity.ai Pro on depth, breadth, and factuality metrics.

## Motivation & Problem

Existing AI search engines (e.g., Perplexity.ai, ChatGPT with browsing) suffer from several
fundamental limitations when handling complex, multi-hop queries:

1. **Shallow retrieval**: Single-round search queries retrieve surface-level results, missing the
   iterative deepening that human researchers naturally perform when investigating complex topics.
2. **Limited information scope**: Conventional systems process only a handful of web pages per
   query, whereas humans conducting deep research naturally browse dozens to hundreds of sources.
3. **No cognitive decomposition**: Complex questions require breaking down into atomic sub-problems,
   but existing search-augmented LLMs attempt to answer everything in a single retrieval pass.
4. **Sequential bottleneck**: When systems do attempt multi-step search, they proceed sequentially,
   making deep research prohibitively slow.

MindSearch's core insight: **Human information-seeking follows a cognitive graph structure** -- we
decompose complex questions, pursue parallel investigation threads, and progressively synthesize
findings. Encoding this cognitive process into a multi-agent framework enables deep, broad search.

## Method

### Architecture Overview

MindSearch consists of two types of agents in a hierarchical multi-agent framework:

```
User Query
    |
    v
+------------------------------------------------------------------+
|                        WebPlanner Agent                           |
|  (High-level reasoning, query decomposition, graph construction) |
+------------------------------------------------------------------+
    |              |              |              |
    v              v              v              v
+----------+  +----------+  +----------+  +----------+
|WebSearcher| |WebSearcher| |WebSearcher| |WebSearcher|  (parallel)
|  Agent 1  | |  Agent 2  | |  Agent 3  | |  Agent N  |
+----------+  +----------+  +----------+  +----------+
    |              |              |              |
    v              v              v              v
[Search Engine API + Web Page Retrieval + Summarization]
    |              |              |              |
    +------+-------+------+------+------+-------+
           |              |              |
           v              v              v
    +--------------------------------------------------+
    |          WebPlanner: Synthesis & Response         |
    +--------------------------------------------------+
```

### WebPlanner: Dynamic Graph Construction

The WebPlanner models multi-step information seeking as a dynamic directed acyclic graph (DAG)
construction process. The algorithm proceeds as follows:

```
Algorithm: WebPlanner Dynamic Graph Construction
-------------------------------------------------
Input:  Complex user query Q
Output: Comprehensive answer A

1:  G = InitGraph(root=Q)           // Initialize DAG with root query
2:  sub_questions = Decompose(Q)    // LLM decomposes into atomic sub-questions
3:  for each sq in sub_questions:
4:      G.add_node(sq)              // Add sub-question as graph node
5:      G.add_edge(Q, sq)           // Connect to parent
6:  end for
7:
8:  while G has unresolved leaf nodes:
9:      parallel for each leaf node n in G:
10:         result = WebSearcher(n.question)    // Dispatch parallel searcher
11:         n.answer = result.summary
12:     end parallel
13:
14:     // Evaluate completeness and extend graph
15:     new_gaps = Evaluate(G, Q)
16:     for each gap in new_gaps:
17:         sq_new = FormulateSubQuestion(gap)
18:         G.add_node(sq_new)
19:         G.add_edge(parent_node, sq_new)    // Extend graph dynamically
20:     end for
21: end while
22:
23: A = Synthesize(G)  // Aggregate all node answers into final response
24: return A
```

Key properties of the DAG:
- **Nodes** represent atomic sub-questions derived from the original complex query
- **Edges** encode dependency relationships between sub-questions
- **Dynamic extension**: The graph grows as search results reveal new information gaps
- **Parallel execution**: Independent sub-questions are searched concurrently

### WebSearcher: Hierarchical Information Retrieval

Each WebSearcher agent handles a single atomic sub-question through a hierarchical search process:

```
Algorithm: WebSearcher Hierarchical Retrieval
----------------------------------------------
Input:  Atomic sub-question sq
Output: Summarized answer with citations

1:  queries = GenerateSearchQueries(sq)    // Multiple query formulations
2:  for each query q in queries:
3:      urls = SearchEngine(q)             // Retrieve top-K URLs
4:      for each url in urls:
5:          content = FetchAndParse(url)    // Extract page content
6:          relevance = ScoreRelevance(content, sq)
7:          if relevance > threshold:
8:              collected_info.append(content)
9:      end for
10: end for
11: summary = LLM_Summarize(collected_info, sq)  // Distill relevant facts
12: return summary
```

The WebSearcher performs multiple rounds of search refinement, generating diverse query
formulations to maximize coverage and filtering results by relevance scoring.

### Multi-Agent Parallelism

A critical design advantage: multiple WebSearcher agents execute concurrently on independent
sub-questions. This enables processing 300+ web pages in ~3 minutes, compared to:
- Sequential processing: would require 30+ minutes
- Human researcher: approximately 3 hours for equivalent depth

## Key Innovations

1. **Cognitive graph construction**: First framework to model information-seeking as dynamic DAG
   construction, directly mimicking how humans decompose and investigate complex questions.
2. **Hierarchical multi-agent search**: Two-tier architecture (planner + parallel searchers)
   separates strategic reasoning from tactical retrieval, enabling both depth and efficiency.
3. **Dynamic graph extension**: The search graph grows adaptively based on discovered information
   gaps, unlike static query decomposition that commits to a fixed plan upfront.
4. **Scale of retrieval**: Processing 300+ web pages per query is an order of magnitude beyond
   typical RAG systems that retrieve 5-20 documents.
5. **Open-source competitive performance**: A 7B-parameter model (InternLM2.5-7B) with MindSearch
   achieves human preference ratings competitive with GPT-4o-based proprietary search engines.

## Experimental Setup

- **Backbone LLMs**: InternLM2.5-7B-Chat, GPT-4o
- **Search engine**: Integration with web search APIs for real-time retrieval
- **Evaluation benchmarks**:
  - Closed-set QA: Factual questions with verifiable answers
  - Open-set QA: Open-ended questions requiring comprehensive, multi-faceted responses
- **Baselines**: ChatGPT-Web (GPT-4o with browsing), Perplexity.ai Pro, vanilla LLMs without search
- **Human evaluation**: 100 real-world queries evaluated by 5 human experts on three dimensions
- **Evaluation dimensions**: Depth, Breadth, and Factuality

## Results

### Closed-Set and Open-Set QA Performance

MindSearch significantly outperforms vanilla baselines:
- GPT-4o + MindSearch: **+4.7%** improvement over vanilla GPT-4o on combined QA metrics
- InternLM2.5-7B + MindSearch: **+6.3%** improvement over vanilla InternLM2.5-7B

### Human Preference Evaluation (100 queries, 5 expert evaluators)

Evaluation across three dimensions (depth, breadth, factuality):

| System                          | Preferred over ChatGPT-Web | Preferred over Perplexity Pro |
|---------------------------------|:--------------------------:|:-----------------------------:|
| MindSearch (GPT-4o)             | Win                        | Win                           |
| MindSearch (InternLM2.5-7B)     | Win                        | Win                           |

Key finding: **MindSearch with InternLM2.5-7B (a 7B open-source model) produces responses preferred
by human evaluators over ChatGPT-Web (GPT-4o) and Perplexity.ai Pro**, demonstrating that the
multi-agent framework architecture matters more than raw model scale for deep search tasks.

### Efficiency Metrics

| Metric                     | MindSearch    | Single-agent search |
|----------------------------|---------------|---------------------|
| Web pages processed        | 300+          | 10-20               |
| Time per complex query     | ~3 minutes    | 10-30 minutes       |
| Equivalent human effort    | ~3 hours      | --                  |

### Depth and Breadth Analysis

Responses from MindSearch show substantially greater depth (thorough, detailed analysis with
supporting evidence) and breadth (covering multiple perspectives and related aspects) compared
to single-round retrieval baselines, as validated by human expert evaluation.

## Limitations

1. **Latency for simple queries**: The multi-agent graph construction process adds overhead that
   is unnecessary for straightforward factual questions where single-round retrieval suffices.
2. **Search API dependency**: Performance is tightly coupled with the quality and availability of
   the underlying web search API; API rate limits constrain parallelism in practice.
3. **Graph construction errors**: The WebPlanner may decompose queries suboptimally, creating
   irrelevant sub-questions or missing important aspects, with errors propagating to final answers.
4. **Cost scaling**: Parallel WebSearcher invocations multiply API costs proportionally to graph
   complexity; highly complex queries can become expensive.
5. **Real-time information currency**: Results depend on search engine indexing freshness; very
   recent events may not be well-covered.
6. **Limited evaluation scale**: Human evaluation on 100 queries, while informative, is a
   relatively small sample for definitive conclusions about general performance.
7. **No factual verification**: While factuality is evaluated, the framework lacks explicit
   fact-checking or cross-source verification mechanisms.

## Key Takeaways

1. **Cognitive process modeling works**: Mimicking how humans actually conduct deep research --
   decompose, search in parallel, synthesize -- yields better results than brute-force retrieval.
2. **Framework > model scale for search**: A 7B model with the right multi-agent framework
   outperforms GPT-4o with naive search integration, suggesting architectural innovation is
   more impactful than scaling model parameters for search-heavy tasks.
3. **Dynamic graph construction enables adaptive depth**: Unlike static query decomposition, the
   DAG grows based on discovered gaps, allowing the system to deepen investigation only where
   needed.
4. **Parallelism is essential for scale**: Processing 300+ pages in 3 minutes is only feasible
   through parallel agent execution; sequential approaches hit fundamental time constraints.
5. **Pioneer of the deep search paradigm**: MindSearch established the multi-agent deep search
   pattern later adopted and extended by systems like WebThinker, Search-o1, and Tongyi
   DeepResearch, making it a foundational reference in the deep research agent space.
