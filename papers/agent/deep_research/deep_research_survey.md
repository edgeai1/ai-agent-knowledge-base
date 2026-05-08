---
title: "Deep Research: A Survey of Autonomous Research Agents"
authors:
  - Wenlin Zhang
  - Xiaopeng Li
  - Yingyi Zhang
  - Pengyue Jia
  - Yichao Wang
  - Huifeng Guo
  - Yong Liu
  - Xiangyu Zhao
venue: arXiv preprint
year: 2025
url: https://arxiv.org/abs/2508.12752
tags:
  - survey
  - deep-research
  - autonomous-agents
  - taxonomy
  - web-exploration
  - report-generation
status: done
---

# 深度研究：自主研究智能体综述

## 摘要

本综述提供了自主深度研究智能体的首个系统性概述，围绕四阶段流水线组织：规划、问题开发、网络探索和报告生成。本文没有列举完整的系统流水线，而是剖析了深度研究底层的模块化能力，分析了技术瓶颈、协调挑战和新兴趋势（推理驱动的检索、结构化报告生成、自进化智能体）。涵盖的主要系统包括 OpenAI Deep Research、Gemini Deep Research、Perplexity Deep Research、DeepResearcher、WebThinker、STORM 等。

## 动机与问题

LLM 驱动的研究智能体的快速发展产生了碎片化的格局：

1. **系统激增**：OpenAI、Google、Perplexity 和学术团队都发布了深度研究智能体，但不存在理解其架构的统一框架。
2. **缺乏系统性比较**：现有工作聚焦于单个系统，缺乏对共享模块和设计决策的横向分析。
3. **评估标准不明确**：对如何评估深度研究能力没有共识；不同系统使用不同的指标和基准测试。
4. **快速演进**：该领域的发展速度超过传统综述的周期；需要模块化分析以适应新系统。

## 方法（综述方法论）

本综述采用模块化分解方法：

```
+------------------------------------------------------------------+
|                Deep Research Pipeline (4 Stages)                  |
|                                                                    |
|  +------------+  +---------------+  +--------------+  +----------+|
|  | Stage 1:   |  | Stage 2:      |  | Stage 3:     |  | Stage 4: ||
|  | Planning   |->| Question      |->| Web          |->| Report   ||
|  |            |  | Developing    |  | Exploration  |  | Generation||
|  +------------+  +---------------+  +--------------+  +----------+|
|                                                                    |
|  Each stage analyzed by:                                           |
|  - Training paradigm (RL, SFT, prompt-based)                     |
|  - Key modeling strategies                                        |
|  - Technical bottlenecks                                          |
|  - Cross-system comparisons                                       |
+------------------------------------------------------------------+
```

### 阶段一：规划

规划将用户查询转化为包含中间子目标的结构化研究计划。

```
Taxonomy of Planning Methods:
+-----------------------------+
| Planning Approaches         |
+-----------------------------+
| 1. Prompt-based planning    |
|    - Zero-shot decomposition|
|    - Few-shot with exemplars|
|    - Chain-of-thought plans |
+-----------------------------+
| 2. Supervised planning      |
|    - Trained on expert plans|
|    - Outline-based planning |
+-----------------------------+
| 3. RL-optimized planning    |
|    - Dynamic plan adjustment|
|    - Emergent planning      |
|    (e.g., DeepResearcher)   |
+-----------------------------+
| 4. World-model-guided       |
|    - LLMs as implicit world |
|      models for reasoning   |
+-----------------------------+
```

核心洞察：LLM 越来越多地充当隐式世界模型，为长程研究任务中的目标导向推理提供环境先验。

### 阶段二：问题开发

问题开发生成用于收集信息的具体查询。

```
Taxonomy of Question Developing:
+-----------------------------------+
| Reward-Optimized Methods          |
| - Trial-and-error query refinement|
| - RL-guided query generation      |
| - Outcome-reward-driven search    |
+-----------------------------------+
| Supervision-Driven Methods        |
| - SFT on expert query trajectories|
| - Manually designed query strategy|
| - Template-based query generation |
+-----------------------------------+
```

### 阶段三：网络探索

网络探索执行从网络来源收集信息的操作。

```
Evolution of Web Retrieval Agents:
+-------------------------------------------------+
| Generation 1: Simple Extractors                 |
|   - Single-query search + snippet extraction    |
|   - No navigation or interaction                |
+-------------------------------------------------+
| Generation 2: Interactive Agents                |
|   - Multi-step search refinement                |
|   - Page navigation (click, scroll)             |
|   - Cross-document information synthesis        |
+-------------------------------------------------+
| Generation 3: Multimodal Agents                 |
|   - Image/chart/table understanding             |
|   - Complex interactive content navigation      |
|   - Deep web exploration (WebThinker-style)     |
+-------------------------------------------------+
```

### 阶段四：报告生成

报告生成产出最终的结构化输出。

```
Report Generation Approaches:
+------------------------------------------+
| 1. Outline-first generation (STORM)      |
|    - Generate outline, then fill sections |
+------------------------------------------+
| 2. Interleaved drafting (WebThinker)     |
|    - Draft sections during research      |
+------------------------------------------+
| 3. Post-hoc synthesis (OpenScholar)      |
|    - Collect all info, then synthesize   |
+------------------------------------------+
| 4. Iterative refinement                  |
|    - Draft, evaluate, revise, repeat     |
+------------------------------------------+
```

## 对比的关键系统

### 商业系统

| 系统              | 开发者 | 核心方法                          | 优势           |
|--------------------|-----------|---------------------------------------|---------------------|
| OpenAI Deep Research| OpenAI    | 基于网络合成的推理智能体    | 最佳 F1（平均 0.55）  |
| Gemini Deep Research| Google    | 带有网络浏览的研究助手   | 最多引用数（111）|
| Perplexity Deep Research| Perplexity| 深度报告生成         | 最佳引用准确率（90%）|

### 学术系统

| 系统          | 方法                    | 核心创新                        |
|----------------|-----------------------------|------------------------------------- |
| DeepResearcher | 真实网络中的端到端 RL   | 涌现性认知行为          |
| WebThinker     | LRM + 深度网络探索器     | 思考-搜索-起草交叉       |
| STORM          | 多视角写作前准备| 视角引导的对话     |
| OpenScholar    | 基于 4500 万论文的 RAG         | 专家级引用准确率        |
| OpenResearcher | 离线轨迹合成 | 9.7 万轨迹，100+ 工具调用    |
| Search-R1      | 静态语料库上的 RL       | 基于 RAG 的 RL 基线                 |
| Search-o1      | 搜索增强推理  | 基础搜索+推理集成  |

## 评估方法

### 现有基准测试

| 基准测试        | 关注点                           | 指标                    |
|-----------------|--------------------------------|----------------------------|
| BrowseComp      | 网络浏览理解     | 准确率                   |
| BrowseComp-Plus | 复杂浏览任务         | 准确率                   |
| GAIA            | 通用 AI 辅助          | 准确率                   |
| GPQA            | 研究生级别科学问答      | 准确率                   |
| WebWalkerQA     | 网络导航问答              | 准确率                   |
| HLE             | 困难长文本评估      | 准确率                   |
| ScholarQABench  | 科学文献问答       | 正确性、引用 F1   |
| FreshWiki       | 文章生成质量     | 组织性、覆盖度     |
| Glaive          | 报告生成质量      | 多维度（1-10）   |
| LiveDRBench     | 实时深度研究评估  | F1                         |
| ReportBench     | 学术综述生成     | 精确率、质量指标 |

### 评估挑战

1. **缺乏统一基准**：不同系统使用不同的基准测试，使得直接比较困难。
2. **静态与实时评估**：网络内容变化；基准测试需要更新。
3. **多维度质量**：报告质量涵盖事实性、覆盖度、组织性、引用和连贯性——难以用单一指标捕获。
4. **人类评估成本**：专家评估昂贵且耗时。

## 开放挑战

1. **推理驱动的检索**：如何将推理深度整合到检索过程中，使搜索策略源于逻辑分析而非模式匹配。

2. **自进化智能体**：通过交互反馈、元学习或接触多样化任务分布来不断学习和优化其研究策略的智能体。

3. **长程一致性**：在 100 步以上的轨迹中维持连贯的研究计划，不丧失焦点或引入矛盾。

4. **多模态研究**：将智能体扩展到处理和综合来自图像、图表、表格、视频和其他非文本模态的信息。

5. **评估标准化**：开发统一的基准测试以捕获深度研究产出的多维度质量。

6. **模块协调**：优化规划、问题开发、网络探索和报告生成阶段之间的交互。

7. **可信性与验证**：确保生成的研究产出中的事实准确性和适当归因，特别是针对高风险的科学和政策应用。

8. **效率与延迟**：在保持质量的同时降低深度研究的计算成本和实际耗时。

## 局限性

1. **综述范围**：主要涵盖英语系统和出版物。
2. **快速过时**：该领域的发展速度超过综述的出版周期；讨论的某些系统在阅读时可能已被取代。
3. **基准碎片化**：由于不同的评估协议，无法提供所有系统的统一定量比较。
4. **商业系统不透明**：专有系统（OpenAI、Google、Perplexity）未公开完整的架构细节，限制了分析深度。

## 后续工作

- 开发统一的深度研究基准测试（DeepResearch Bench、ResearchRubrics）。
- 扩展到特定领域的深度研究（医学、法律、科学）。
- 将多模态能力集成到研究智能体中。
- 结合两种范式优势的混合 RL+SFT 训练方法。

## 核心要点

1. 四阶段流水线（规划、问题开发、网络探索、报告生成）为理解和比较深度研究智能体提供了有用的模块化框架。
2. 该领域正从基于提示的智能体快速转向基于强化学习训练的智能体，涌现性认知行为出现在端到端强化学习系统中。
3. 评估仍然是最薄弱的环节：不存在统一的基准测试，研究产出的多维度质量评估仍是一个开放问题。
4. 持续改进其研究策略的自进化智能体代表了该领域最有前景的前沿方向。
5. 在线强化学习（真实网络、昂贵、不可复现）与离线 SFT（静态语料库、便宜、可复现）之间的张力仍然是一个根本性的设计权衡。
