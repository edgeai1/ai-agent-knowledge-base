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

# STORM：通过检索和多视角提问进行主题大纲合成

## 摘要

STORM 是斯坦福大学开发的 LLM 驱动系统，通过模拟写作前的研究过程从头生成全面的类维基百科文章。它发现主题的多元化视角，模拟与有据可依的领域专家的多视角对话，并将信息整理成结构化大纲后再生成完整文章。在 FreshWiki（100 篇近期维基百科文章）上的评估中，经验丰富的维基百科编辑评判 STORM 生成的文章在组织性上提升了 +25%，在覆盖广度上提升了 +10%，优于 RAG 基线。

## 动机与问题

撰写高质量长文本文章（如维基百科条目）需要广泛的写作前研究，包括：
1. **发现写作内容**：理解主题的完整范围。
2. **寻找多元视角**：从多个观点收集信息。
3. **组织信息**：创建组织文章的连贯大纲。

现有的 LLM 方法要么：
- 直接生成文章（容易产生幻觉，覆盖面浅）。
- 使用简单 RAG（一次检索，无迭代深化或视角多样性）。
- 需要人工参与写作前准备（不可扩展）。

STORM 自动化整个写作前流水线，生成与人工撰写的维基百科条目具有可比广度和深度的文章。

## 方法

### 流水线概览

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

### 步骤一：视角发现

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

系统自动从相关的现有维基百科文章中挖掘视角，识别全面文章应涵盖的观点范围。

### 步骤二：多视角对话模拟

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

核心洞察：随着答案更新写作者的理解，新问题自然产生，模仿了人类研究者通过对话迭代深化知识的方式。

### 步骤三：大纲生成

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

### 步骤四：完整文章生成

- 按照大纲使用整理的信息生成每个章节。
- 包含来自有据对话的内联引用。
- 确保章节之间的连贯过渡。

## 关键创新

1. **视角引导的提问**：STORM 不是提出通用问题，而是在提示中包含特定视角以提供聚焦、先验知识和观点多样性。这对于实现广泛的主题覆盖至关重要。

2. **模拟对话进行研究**：对话形式模仿了人类研究者的学习方式——新问题从答案中产生，实现了静态检索无法达到的迭代深化。

3. **写作前和写作的分离**：通过在生成前显式建模研究/大纲阶段，STORM 比端到端方法产生更有条理的文章。

4. **FreshWiki 基准测试**：精心策划的评估数据集，包含 2022 年 2 月至 2023 年 9 月的 100 篇高质量维基百科文章，旨在避免 LLM 预训练数据的污染。

## 实验设置

### FreshWiki 数据集

- **规模**：100 篇高质量维基百科文章。
- **来源**：2022 年 2 月至 2023 年 9 月编辑次数最多的维基百科页面。
- **目的**：确保主题足够新以避免预训练数据泄露。
- **质量筛选**：仅包含符合维基百科质量标准的文章。

### 评估维度

| 维度      | 衡量内容                                | 评估者         |
|---------------|------------------------------------------------|-------------------|
| 组织性  | 大纲质量、章节结构              | 维基百科编辑 |
| 覆盖度      | 涵盖主题方面的广度              | 维基百科编辑 |
| 相关性     | 信息与主题的关联度             | 自动 + 人工 |
| 引用质量| 参考文献的准确性和适当性     | 人工检查 |

### 基线

- **大纲驱动的 RAG 基线**：一次检索，生成大纲，撰写文章。
- **直接生成**：LLM 在没有显式写作前阶段的情况下生成文章。
- **仅检索**：无视角模拟或大纲的标准 RAG。

### 使用的 LLM

- GPT-3.5-turbo 和 GPT-4 作为流水线各阶段的骨干 LLM。

## 结果

### 文章质量（维基百科资深编辑评估）

| 指标        | STORM 相比基线的提升 |
|--------------|-------------------------------|
| 组织性 | **绝对提升 +25%**     |
| 覆盖度     | **绝对提升 +10%**     |

- 维基百科资深编辑一致评定 STORM 生成的文章比大纲驱动的 RAG 基线更有条理、覆盖面更广。
- 多视角对话方法是覆盖度提升的主要驱动力。
- 大纲质量的提升直接转化为最终文章更好的组织性。

### 消融实验发现

- 移除视角多样性会显著降低覆盖度。
- 移除对话形式（使用单轮问答替代）会降低深度。
- 大纲生成阶段对组织质量至关重要。

## 局限性

1. **互联网偏见传播**：文章继承了网络来源中存在的偏见；过度代表的观点可能主导生成的内容。
2. **虚构关联**：LLM 有时会在研究过程中发现的不相关事实之间创建看似合理但不正确的关联。
3. **小众主题的可扩展性**：性能取决于多元网络来源的可用性；冷门主题可能缺少足够的来源材料。
4. **引用可靠性**：虽然基于网络来源，但系统有时可能会错误归因或误解来源内容。
5. **评估范围**：FreshWiki 包含 100 篇文章；需要更大规模的评估以获得统计稳健性。

## 后续工作

- **Co-STORM**（EMNLP 2024）：将 STORM 扩展到协作式人机知识策划，用户可以观察并引导 LM 智能体之间的对话，实现"未知的未知"的发现。
- **STORM v2**：改进的流水线，具有更好的检索和大纲优化。
- **WebThinker**：将理念扩展到在单一循环中集成推理+搜索+起草。
- **深度研究智能体**：STORM 基于视角的研究方法影响了后来的系统，如 DeepResearcher 和 OpenResearcher。

## 核心要点

1. 显式建模写作前研究过程（视角发现、专家对话、大纲策划）比端到端生成产生更好的长文本文章。
2. 多视角提问对于实现广泛的主题覆盖至关重要；不同的观点自然地呈现主题的不同方面。
3. 模拟对话实现了静态检索无法匹配的知识迭代深化，因为后续问题从先前的答案中自然涌现。
4. FreshWiki 基准测试为文章生成系统提供了严格的、抗污染的评估框架。
5. 在 github.com/stanford-oval/storm 开源后，STORM 已成为深度研究流水线生态系统中的基础系统。
