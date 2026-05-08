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

# MindSearch：模仿人类思维激发深度 AI 搜索器

## 摘要

MindSearch 是一个用于深度网络信息搜索的多智能体框架，模仿人类认知搜索过程。它通过 WebPlanner 智能体将复杂查询分解为动态有向无环图（DAG）的原子子问题，然后调度并行的 WebSearcher 智能体在 3 分钟内从 300 多个网页中检索和总结信息——相当于人类约 3 小时的工作量。人类评估者在深度、广度和事实性指标上更偏好 MindSearch 的回答（即使使用 7B InternLM2.5 骨干网络），优于 ChatGPT-Web 和 Perplexity.ai Pro。

## 动机与问题

现有的 AI 搜索引擎（如 Perplexity.ai、带有浏览功能的 ChatGPT）在处理复杂的多跳查询时存在几个根本性局限：

1. **浅层检索**：单轮搜索查询检索到的是表面结果，缺少人类研究者在调查复杂主题时自然进行的迭代深化。
2. **信息范围有限**：传统系统每次查询仅处理少量网页，而进行深度研究的人类自然会浏览数十到数百个来源。
3. **缺乏认知分解**：复杂问题需要分解为原子子问题，但现有的搜索增强 LLM 试图在单次检索中回答所有内容。
4. **顺序处理瓶颈**：当系统确实尝试多步搜索时，它们按顺序进行，使得深度研究的速度变得极慢。

MindSearch 的核心洞察：**人类信息搜寻遵循认知图结构**——我们分解复杂问题、追踪并行调查线索、逐步综合发现。将这种认知过程编码为多智能体框架可以实现深度、广泛的搜索。

## 方法

### 架构概览

MindSearch 由分层多智能体框架中的两类智能体组成：

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

### WebPlanner：动态图构建

WebPlanner 将多步信息搜寻建模为动态有向无环图（DAG）的构建过程。算法如下：

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

DAG 的关键属性：
- **节点** 代表从原始复杂查询派生的原子子问题
- **边** 编码子问题之间的依赖关系
- **动态扩展**：图会随着搜索结果揭示新的信息缺口而增长
- **并行执行**：独立的子问题被并发搜索

### WebSearcher：分层信息检索

每个 WebSearcher 智能体通过分层搜索过程处理单个原子子问题：

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

WebSearcher 执行多轮搜索优化，生成多样化的查询表述以最大化覆盖范围，并通过相关性评分过滤结果。

### 多智能体并行性

一个关键的设计优势：多个 WebSearcher 智能体在独立子问题上并发执行。这使得在约 3 分钟内处理 300 多个网页成为可能，相比之下：
- 顺序处理：需要 30 分钟以上
- 人类研究者：相同深度大约需要 3 小时

## 关键创新

1. **认知图构建**：首个将信息搜寻建模为动态 DAG 构建的框架，直接模仿人类分解和调查复杂问题的方式。
2. **分层多智能体搜索**：两层架构（规划者+并行搜索者）将战略推理与战术检索分离，同时实现深度和效率。
3. **动态图扩展**：搜索图基于发现的信息缺口自适应增长，不同于静态查询分解一开始就固定计划。
4. **检索规模**：每次查询处理 300 多个网页，比通常检索 5-20 个文档的典型 RAG 系统高出一个数量级。
5. **开源竞争性能**：7B 参数模型（InternLM2.5-7B）配合 MindSearch 在人类偏好评分上与基于 GPT-4o 的专有搜索引擎具有竞争力。

## 实验设置

- **骨干 LLM**：InternLM2.5-7B-Chat、GPT-4o
- **搜索引擎**：与网络搜索 API 集成进行实时检索
- **评估基准测试**：
  - 封闭式问答：具有可验证答案的事实性问题
  - 开放式问答：需要全面、多方面回答的开放性问题
- **基线**：ChatGPT-Web（带浏览功能的 GPT-4o）、Perplexity.ai Pro、无搜索的原始 LLM
- **人类评估**：100 个真实世界查询由 5 位人类专家在三个维度上评估
- **评估维度**：深度、广度和事实性

## 结果

### 封闭式和开放式问答性能

MindSearch 显著优于原始基线：
- GPT-4o + MindSearch：在综合问答指标上比原始 GPT-4o **提升 +4.7%**
- InternLM2.5-7B + MindSearch：比原始 InternLM2.5-7B **提升 +6.3%**

### 人类偏好评估（100 个查询，5 位专家评估者）

在三个维度（深度、广度、事实性）上的评估：

| 系统                          | 优于 ChatGPT-Web | 优于 Perplexity Pro |
|---------------------------------|:--------------------------:|:-----------------------------:|
| MindSearch（GPT-4o）             | 胜出                        | 胜出                           |
| MindSearch（InternLM2.5-7B）     | 胜出                        | 胜出                           |

关键发现：**使用 InternLM2.5-7B（7B 开源模型）的 MindSearch 产生的回答在人类评估者中优于 ChatGPT-Web（GPT-4o）和 Perplexity.ai Pro**，证明了对于深度搜索任务，多智能体框架架构比原始模型规模更重要。

### 效率指标

| 指标                     | MindSearch    | 单智能体搜索 |
|----------------------------|---------------|---------------------|
| 处理的网页数        | 300+          | 10-20               |
| 每个复杂查询的时间     | 约 3 分钟    | 10-30 分钟       |
| 等效人类工作量    | 约 3 小时      | --                  |

### 深度和广度分析

经人类专家评估验证，MindSearch 的回答比单轮检索基线展现出显著更高的深度（具有支持证据的全面、详细分析）和广度（涵盖多个视角和相关方面）。

## 局限性

1. **简单查询的延迟**：多智能体图构建过程增加了额外开销，对于单轮检索即可的简单事实性问题而言是不必要的。
2. **搜索 API 依赖**：性能与底层网络搜索 API 的质量和可用性紧密相关；API 速率限制在实际中制约了并行性。
3. **图构建错误**：WebPlanner 可能会次优地分解查询，创建无关的子问题或遗漏重要方面，错误会传播到最终答案。
4. **成本扩展**：并行的 WebSearcher 调用使 API 成本与图的复杂度成比例增长；高度复杂的查询可能变得昂贵。
5. **实时信息时效性**：结果依赖于搜索引擎索引的新鲜度；最新事件可能覆盖不佳。
6. **评估规模有限**：对 100 个查询进行的人类评估虽然有信息价值，但对于关于一般性能的确定性结论来说样本量相对较小。
7. **缺乏事实验证**：虽然评估了事实性，但框架缺乏显式的事实核查或跨来源验证机制。

## 核心要点

1. **认知过程建模有效**：模仿人类实际进行深度研究的方式——分解、并行搜索、综合——比暴力检索产生更好的结果。
2. **搜索任务中框架优于模型规模**：带有正确多智能体框架的 7B 模型优于使用简单搜索集成的 GPT-4o，表明对于搜索密集型任务，架构创新比扩展模型参数更有影响力。
3. **动态图构建实现自适应深度**：与静态查询分解不同，DAG 基于发现的缺口增长，使系统仅在需要的地方深化调查。
4. **并行性对规模至关重要**：在 3 分钟内处理 300 多个页面只有通过并行智能体执行才可行；顺序方法面临根本性的时间约束。
5. **深度搜索范式的先驱**：MindSearch 建立了多智能体深度搜索模式，后来被 WebThinker、Search-o1 和 Tongyi DeepResearch 等系统采用和扩展，使其成为深度研究智能体领域的基础性参考。
