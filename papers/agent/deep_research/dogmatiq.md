---
title: "DoGMaTiQ: Automated Generation of Question-and-Answer Nuggets for Report Evaluation"
authors:
  - Bryan Li
  - William Walden
  - Yu Hou
  - Gabrielle Kaili-May Liu
  - Dawn Lawrie
  - James Mayfield
  - Eugene Yang
  - Chris Callison-Burch
  - Laura Dietz
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.04458
tags:
  - deep-research
  - evaluation
  - QA-nuggets
  - report-evaluation
  - automated-evaluation
  - TREC
status: done
---

# DoGMaTiQ：报告评估的自动化问答 Nugget 生成

## 摘要

DoGMaTiQ 解决了**长文档报告评估**的核心难题：如何自动判断生成的研究报告是否完整、是否包含关键信息？传统方法（exact match、BLEU、ROUGE）无法处理"同一个信息可以用不同方式表达"的本质，人工评估成本高且不可扩展。DoGMaTiQ 提出**用 QA 对（"nugget"）作为评估单元**，每个 nugget 是一个独立的"信息需求 + 可能的答案集"。配合 AutoArgue 评估框架，在 TREC NeuCLIR 和 RAGTIME 两个跨语言任务上与人工判断**强 rank correlation**。

## 动机与问题

### 长篇报告评估的根本难题

深度研究输出的是**长文档**（数千 token），不是短答案。评估它需要回答：

1. **完整性**：所有重要信息都覆盖了吗？
2. **正确性**：报告中的事实正确吗？
3. **相关性**：内容紧扣查询，没跑偏吗？
4. **可读性**：结构清晰、引用准确吗？

传统短答案评测（QA accuracy）完全不适用 —— 因为没有"标准答案"，只有"应该提到的信息要点"。

### 现有方法的局限

| 方法 | 问题 |
|------|------|
| BLEU/ROUGE | 词形匹配，不识别语义等价 |
| BERTScore | 嵌入相似度，但报告太长导致噪声 |
| LLM-as-Judge | 主观、不稳定、贵 |
| 人工评估 | 不可扩展（每报告 30 分钟） |

### Nugget 的概念

"Nugget" 来自信息检索社区，意为"信息原子"：

```
信息需求："谁是 X 公司 2024 CEO？"
可能答案：[John Smith, 约翰·史密斯, John A. Smith, ...]
```

一个报告"覆盖了这个 nugget" ⇔ 报告中至少有一个表达与可能答案匹配。

## 方法：DoGMaTiQ 三阶段流水线

### 阶段 1：文档基础的 nugget 生成

```
输入：相关文档集合 + 查询
处理：
  1. LLM 阅读每个文档
  2. 抽取"该查询下文档贡献的信息原子"
  3. 输出 (question, answer_set) 对
```

例如：
```
查询："2024 年量子计算商业化进展"
文档：IBM 2024 年报
生成 nugget:
  Q: IBM 2024 年发布的量子计算机有多少 qubits？
  A: [1121, 1121 qubits, 一千一百二十一]
```

### 阶段 2：Paraphrase 聚类

不同文档可能提到同一信息，避免重复：

```
Nugget A: "IBM 2024 发布 1121-qubit 系统"
Nugget B: "IBM 突破千 qubit 大关"
Nugget C: "蓝色巨人推出 1121-qubit Condor 处理器"

→ 聚类为同一 nugget，合并 answer_set
```

使用 LLM-based clustering，保证同义 nugget 合并。

### 阶段 3：Quality-based 子选

并非所有 nugget 都同等重要：

```
高质量 nugget：
  ✓ 信息密度高
  ✓ 答案明确
  ✓ 与查询强相关
  ✓ 跨文档可验证

低质量 nugget：
  ✗ 模糊问题
  ✗ 答案过于通用
  ✗ 与查询关系弱
```

用 quality criteria 筛选 top-K nugget 作为最终评估集。

## AutoArgue 集成

DoGMaTiQ 与 [AutoArgue](https://example.com) 框架配合：

```
[Report 生成]
     ↓
[DoGMaTiQ 生成 Nugget 集合]
     ↓
[对每个 Nugget，用 AutoArgue 判断报告是否覆盖]
     ↓
[聚合得分 = 覆盖率]
```

AutoArgue 用 LLM 做"报告是否回答了这个问题"的判断，比 EM 更宽容。

## 实验结果

### TREC NeuCLIR 和 RAGTIME

| 评估方法 | 与人工评估的 rank correlation |
|---------|------------------------------|
| BLEU | 0.34 |
| ROUGE-L | 0.42 |
| BERTScore | 0.51 |
| LLM-as-Judge (single) | 0.61 |
| **DoGMaTiQ + AutoArgue** | **0.79** |

**首个超过 0.75 的自动评估方法**。

### 鲁棒性

研究者验证了 DoGMaTiQ 评分对**异常系统**的鲁棒性：

```
✗ 极长报告（堆砌信息）：DoGMaTiQ 不被欺骗
✗ 极短报告（漏掉关键信息）：DoGMaTiQ 准确识别
✗ 高重复报告：合并后才计 nugget 覆盖
```

### 跨语言

NeuCLIR 是多语言任务（中文/英文/俄文/波斯文混合源文档），DoGMaTiQ 在所有语言对上保持高相关性。

## 为什么有效

### 1. Nugget = 评估的"原子单位"

将"评估长报告"分解为"评估每个 nugget 是否被覆盖"，把不可处理的问题转化为可处理的子问题。

### 2. Answer Set 容纳表达多样性

允许多种表达方式都被接受为正确答案，避免"语义正确但用词不同"被误判。

### 3. 文档 grounding 保证质量

Nugget 不是 LLM 凭空想出来的，而是从相关文档中抽取的。这确保**评估的是真实信息**，而非 LLM 自己的偏好。

### 4. Paraphrase 聚类避免双计

防止"提到同一信息的不同表达"被多次计分。

## 贡献

1. **方法学**：将 nugget 概念从短问答推广到长报告评估。
2. **流水线**：文档 grounding → 聚类 → 质量子选，三阶段稳健。
3. **AutoArgue 集成**：完整端到端自动评估。
4. **跨语言验证**：在多语言任务上保持高相关性。
5. **开源**：代码 + nugget 集合公开。

## 局限与未解决问题

- LLM 在 nugget 生成阶段的偏好可能影响评估。
- 长尾领域（医学、法律）需特定提示词工程。
- Paraphrase 聚类对低资源语言效果未充分验证。

## 与相关工作的关系

### 范式背景
- 延续 TREC 的 "nugget pyramid" 评估传统。
- 用现代 LLM 自动化原本需要人工的 nugget 标注。

### 直接对手
- **替代 LLM-as-Judge**：DoGMaTiQ 比直接让 LLM 打分更可解释、更稳定。
- **替代 BLEU/ROUGE**：DoGMaTiQ 评估语义而非词形。

### 互补关系
- 与 [[deepweb_bench]] 互补：DeepWeb-Bench 提供任务，DoGMaTiQ 提供评估方法。
- 与 [[browsecomp_plus]] 互补：BrowseComp-Plus 提供短答案任务，DoGMaTiQ 处理长报告。
- 与 [[hdri]] 互补：HDRI 的"事实推理框架"与 nugget 概念高度相似。

### 后续方向
- 动态 nugget 生成：根据用户兴趣实时生成新 nugget。
- 多模态 nugget：图表、视频中的信息原子。
- 适应不同领域的 quality criteria。
