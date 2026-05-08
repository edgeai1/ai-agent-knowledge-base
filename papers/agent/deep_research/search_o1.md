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

# Search-o1：智能体搜索增强的大型推理模型

## 摘要

Search-o1 通过将智能体搜索工作流集成到推理过程中，解决了大型推理模型（LRM，如 OpenAI-o1 和 QwQ）的知识不足问题。当模型在思维链过程中遇到不确定的知识时，会动态触发外部检索。新颖的文档内推理（RiD）模块在将检索到的文档注入推理链之前进行深度分析，最大限度地减少噪声并保持连贯的推理流程。在复杂推理（GPQA、MATH 等）和开放域问答基准测试上的评估表明，Search-o1 优于原始推理模型和朴素 RAG 增强基线，平均比 QwQ-32B 高出 +3.1%，比 RAgent-QwQ-32B 高出 +4.7%。

## 动机与问题

大型推理模型（LRM），如 OpenAI-o1 和 QwQ-32B-Preview，通过大规模强化学习展示了令人印象深刻的长步骤推理能力。然而，其扩展推理过程存在关键局限：

1. **知识不足**：在长思维链（CoT）推理过程中，LRM 经常遇到知识缺口——模型参数中未捕获的事实、公式或特定领域信息。这些缺口导致推理不确定性和错误传播。
2. **不确定性下的幻觉**：当 LRM 缺乏知识时，倾向于产生看似合理但不正确的事实幻觉，随后污染后续推理步骤。
3. **朴素 RAG 干扰**：简单地用检索文档增强推理（标准 RAG）会引入噪声的、往往无关的信息，扰乱连贯的推理链，有时甚至降低而非提高性能。
4. **静态知识边界**：不同于人类在推理过程中可以无缝查阅外部资源，LRM 完全在参数化知识范围内运行，无法在思考过程中"查找信息"。

Search-o1 的洞察：**检索触发应由推理过程本身决定**（而非由单独的系统决定），且检索到的文档在注入前必须经过深度分析和提炼，以避免扰乱推理链。

## 方法

### 整体框架架构

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

### 智能体搜索工作流

搜索组件在模型推理过程中内联触发：

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

### 文档内推理（RiD）模块

RiD 模块是区分 Search-o1 和朴素 RAG 的关键技术贡献：

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

RiD 模块执行深度分析而非朴素拼接：
- **相关性过滤**：移除与当前推理步骤无关的文档和段落
- **事实提取**：识别所需的具体事实、公式和数据点
- **跨文档推理**：跨多个来源验证信息一致性
- **推理对齐格式化**：将提取的知识合成为自然延续现有思维链的格式，最大限度减少对推理连贯性的干扰

### 不确定性检测

搜索触发在推理模型表现出知识不确定性信号时激活：
- 模糊表述（"我认为"、"可能"、"我不确定"）
- 推理链中的自相矛盾
- 显式的知识寻求语句（"我需要知道"、"让我查一下"）
- 推理步骤中的低置信度指标

## 关键创新

1. **内联智能体搜索**：不同于标准 RAG（先检索后生成），Search-o1 在推理过程中动态触发检索，精确地在知识缺口出现时进行。
2. **文档内推理模块**：在注入前对检索文档进行深度分析，防止了困扰将朴素 RAG 应用于推理模型时的噪声和连贯性破坏。
3. **保持推理的知识注入**：检索到的知识被格式化为无缝延续思维链，而非作为原始上下文倾倒，维持推理连贯性。
4. **自感知知识缺口**：模型根据自身的不确定性信号决定何时搜索，创造了更自然和高效的检索模式。
5. **适用于现有 LRM**：作为现有推理模型（QwQ、DeepSeek-R1）的增强方案，无需重新训练模型。

## 实验设置

- **基础模型**：QwQ-32B-Preview（主要），与其他推理模型进行比较
- **基准测试**：
  - 复杂推理：GPQA（研究生级别科学问答）、MATH（数学推理）、LiveCodeBench（编程）、ARC-Challenge（科学推理）
  - 开放域问答：TriviaQA、Natural Questions（NQ）、HotpotQA、2WikiMultiHopQA、MuSiQue、Bamboogle（单跳和多跳问答）
- **基线**：原始 QwQ-32B、QwQ + 朴素 RAG、RAgent-QwQ-32B、标准检索增强方法
- **指标**：准确率（复杂推理）、F1/EM 分数（开放域问答）

## 结果

### 复杂推理任务

Search-o1 配合 QwQ-32B-Preview 骨干网络的结果：

| 基准测试       | QwQ-32B（原始） | QwQ + 朴素 RAG | Search-o1    |
|-----------------|:-----------------:|:---------------:|:------------:|
| GPQA（扩展） | 约 54%              | 下降        | **57.9%**    |
| GPQA 物理    | --                | --              | **68.7%**    |
| GPQA 生物    | --                | --              | **69.5%**    |

关键发现：朴素 RAG 实际上*降低了*推理模型在复杂任务上的性能，因为噪声的检索文档扰乱了推理链。Search-o1 的 RiD 模块防止了这一问题。

### 开放域问答任务（6 个基准测试）

| 方法               | 6 个问答基准测试的平均值 |
|----------------------|:--------------------------:|
| QwQ-32B（原始）    | 基线                   |
| RAgent-QwQ-32B       | 基线 + X               |
| **Search-o1**        | **比 QwQ-32B 高 +3.1%**     |
|                      | **比 RAgent 高 +4.7%**      |

Search-o1 在大多数开放域问答基准测试上优于原始 QwQ-32B 和 RAgent-QwQ-32B，证明了 RiD 模块提供了一致的改进。

### 消融实验：文档内推理的影响

| 配置             | 相对于原始模型的性能变化 |
|---------------------------|:----------------------------:|
| QwQ + 朴素 RAG           | 有时为负（噪声）   |
| QwQ + 搜索（无 RiD）     | 适度改善           |
| QwQ + 搜索 + RiD        | **最佳性能**         |

RiD 模块至关重要：没有它，简单地向推理模型添加搜索会产生不一致的结果，甚至可能损害性能。

## 局限性

1. **延迟开销**：每次搜索触发都会为已经很长的推理链增加检索和 RiD 处理延迟，可能使总推理时间翻倍或增至三倍。
2. **搜索质量依赖**：性能取决于检索文档的质量；较差的搜索结果导致较差或误导性的知识注入。
3. **不确定性检测的启发式方法**：检测何时触发搜索的机制基于启发式信号（模糊表述等），可能错过真正的知识缺口或在修辞性模糊表述上不必要地触发。
4. **单一基础模型评估**：主要结果仅在 QwQ-32B-Preview 上；对其他推理模型（DeepSeek-R1、OpenAI-o1）的泛化未得到广泛验证。
5. **计算成本**：每次查询的多次搜索-推理循环使该方法比单次推理显著更昂贵。
6. **未训练的 RiD**：RiD 模块依赖提示推理而非训练过的文档分析，留有通过专门微调改进的空间。
7. **对比差距**：由于 API 访问限制，与商业系统（带浏览功能的 OpenAI-o1）的直接比较有限。

## 核心要点

1. **朴素 RAG 损害推理模型**：简单地在推理模型输入前附加检索文档可能会通过引入扰乱思维链连贯性的噪声来降低性能。这是对 RAG 社区的重要警示。
2. **文档内推理至关重要**：在注入前对检索内容进行深度分析是关键创新——它通过确保只有干净、相关的知识进入推理链来弥合检索和推理之间的差距。
3. **内联搜索优于预搜索**：在推理过程中（当检测到不确定性时）动态触发检索比在推理开始前检索更有效，因为模型可以精确识别它缺乏哪些知识。
4. **与推理扩展互补**：Search-o1 与推理模型训练（强化学习、长思维链）的改进是正交的——它为任何现有推理模型增加知识访问能力，表明两种方法是互补的。
5. **搜索增强推理的基础**：Search-o1 与 WebThinker 等并发工作一起，建立了将智能体搜索直接集成到推理循环中的范式，影响了后续系统如 Search-R1 和 Tongyi DeepResearch。
