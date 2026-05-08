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

# OpenScholar：使用检索增强语言模型合成科学文献

## 摘要

OpenScholar 是一个专门用于科学文献合成的检索增强语言模型。它在来自 Semantic Scholar 的包含 4500 万篇开放获取论文（2.36 亿段落嵌入）的数据存储中进行搜索，生成带有引用支持的回答。GPT-4o 的引用幻觉率高达 78-90%，而 OpenScholar 的引用准确率与人类专家持平。在专家评估（16 位博士级评审）中，OpenScholar-8B 的回答有 51% 的情况被优先选择于专家撰写的回答，GPT-4o 增强变体有 70% 的情况被优先选择。该成果发表于 Nature（2026），由 AI2 和华盛顿大学开发。

## 动机与问题

科学文献合成对研究者至关重要，但现有 LLM 存在灾难性缺陷：

1. **引用幻觉**：GPT-4o 有 78-90% 的时间在伪造引用，使其科学产出不可靠且可能对研究有害。
2. **规模挑战**：超过 4500 万篇开放获取论文存在；没有 LLM 仅通过预训练就能内化这些知识。
3. **多论文合成**：回答科学问题通常需要综合多篇论文的发现，而不仅是检索单个文档。
4. **成本**：基于商业 API 的解决方案（如 PaperQA2）日常使用成本高昂。

OpenScholar 通过在大规模科学文献数据存储上构建专门的检索增强流水线，配合训练过的检索器和微调的 8B 模型来解决这些问题。

## 方法

### 系统架构

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

### OpenScholar 数据存储（OSDS）

- **来源**：Semantic Scholar 开放获取论文。
- **规模**：4500 万篇论文，2.36 亿段落嵌入。
- **覆盖范围**：计算机科学、物理学、神经科学、生物医学等。
- **索引**：稠密段落嵌入用于高效相似性搜索 + BM25 用于关键词匹配。

### 检索流水线

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

### OpenScholar-8B 训练

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

### 自我优化循环

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

## 关键创新

1. **大规模科学数据存储**：来自 Semantic Scholar 的 4500 万篇论文/2.36 亿段落，提供对开放获取科学文献的近乎全面覆盖。

2. **引用准确率匹配人类专家**：首个在科学文献合成中达到专家级引用准确率的系统，解决了 GPT-4o 78-90% 的幻觉率问题。

3. **紧凑的开放权重模型**：OpenScholar-8B（微调 Llama 3.1 8B）尽管远小于 GPT-4o 但性能更优，证明了领域专精胜过模型规模。

4. **带有迭代检索的自我优化**：模型识别自身引用不足的论述并检索额外证据，迭代提升引用覆盖率。

5. **ScholarQABench**：首个用于科学文献合成的大规模多领域基准测试，包含专家撰写的查询和回答。

## 实验设置

### ScholarQABench 基准测试

| 属性         | 详情                                              |
|-----------------|------------------------------------------------------|
| 总查询数   | 2,967 条专家撰写                                 |
| 长文本回答| 208 条专家撰写                                  |
| 领域         | 计算机科学、物理学、神经科学、生物医学 |
| 评估      | 自动指标 + 人类专家评估          |

### 评估指标

- **正确性**：合成回答的事实准确性。
- **引用 F1**：引用参考文献相对于标准参考文献的精确率和召回率。
- **覆盖度**：涵盖相关方面的比例。
- **相关性**：回答与查询的关联度。
- **组织性**：回答的结构质量。
- **实用性**：领域专家评判的整体效用。

### 基线

- **GPT-4o**：最先进的商业 LLM（无专门检索）。
- **PaperQA2**：先前的专门科学问答系统。
- **Perplexity Pro**：商业搜索增强系统。
- **专家撰写的回答**：用于偏好评估的人类专家基线。

### 人类评估

- **评估者**：16 位博士级领域专家。
- **协议**：并排偏好对比（盲评）。

## 结果

### 自动评估

| 模型              | 正确性 | 引用 F1 |
|-------------------|-------------|-------------|
| OpenScholar-8B    | GPT-4o + 6.1% | 高     |
| OpenScholar-GPT4o | GPT-4o + 12%  | 39.5     |
| GPT-4o（原始）  | 基线    | 0.1        |
| PaperQA2          | OS-8B - 5.5%| --         |

- **OpenScholar-8B 在正确性上比 GPT-4o 高出 6.1%**，尽管模型小得多。
- **OpenScholar-GPT4o** 将 GPT-4o 的引用 F1 从 0.1 提升到 39.5（395 倍提升）。
- 比 PaperQA2 **节省 100 倍成本**。

### 引用幻觉

| 模型           | 引用幻觉率 |
|----------------|---------------------------|
| GPT-4o         | 78-90%                    |
| OpenScholar-8B | 专家级准确率     |

### 人类专家偏好（对比专家撰写的回答）

| 模型              | 优于专家的比例 |
|-------------------|----------------------|
| OpenScholar-GPT4o | **70%** 的情况  |
| OpenScholar-8B    | **51%** 的情况  |
| GPT-4o（原始）  | 32% 的情况      |

## 局限性

1. **开放获取偏差**：4500 万论文数据存储仅覆盖开放获取论文；专有期刊文章被排除，可能遗漏重要文献。
2. **时间滞后**：数据存储需要定期更新；最新论文可能尚未被索引。
3. **领域覆盖**：虽然跨领域，但在数据存储中代表性有限的小众子领域性能可能有差异。
4. **检索瓶颈**：回答质量受限于检索器的召回率；如果相关论文未被检索到，就无法被引用。
5. **单语言**：主要是英语论文；非英语科学文献代表不足。

## 后续工作

- 与类 STORM 系统集成，用于从文献中生成完整综述文章。
- 扩展到专有文献（需要适当的访问控制）。
- 多模态科学合成（图形、表格、公式）。
- 实时数据存储更新以应对新兴研究领域。
- 应用于系统综述和荟萃分析。

## 核心要点

1. 引用幻觉是通用 LLM 中一个关键的未解决问题（GPT-4o 为 78-90%）；专门的检索增强系统可以解决它。
2. 配有领域特定检索的微调 8B 模型在科学合成上优于 GPT-4o，证明了架构设计比原始规模更重要。
3. 使用开源模型可以实现专家级科学文献合成，使其对更广泛的研究社区可及。
4. 带有迭代检索的自我优化对于在多论文合成任务中实现全面的引用覆盖至关重要。
5. 发表于 Nature（2026），OpenScholar 验证了基于 RAG 的方法可以大规模产生受领域专家信赖的研究产出。
