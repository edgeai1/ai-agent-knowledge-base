---
title: "WebThinker: Empowering Large Reasoning Models with Deep Research Capability"
authors:
  - Xiaoxi Li
  - Jiajie Jin
  - Yujia Zhou
  - Yongfeng Zhang
  - (Renmin University of China, RUC-NLPIR)
venue: NeurIPS 2025 (also presented as WWW 2026 Oral)
year: 2025
url: https://arxiv.org/abs/2504.21776
tags:
  - deep-research
  - large-reasoning-models
  - web-exploration
  - report-generation
  - DPO
  - think-search-draft
status: done
---

# WebThinker：赋予大型推理模型深度研究能力

## 摘要

WebThinker 赋予大型推理模型（LRM）自主网络搜索、导航和报告生成能力。它引入了深度网络探索器模块用于多步网络导航，以及自主思考-搜索-起草策略用于交叉推理、搜索和写作。通过迭代在线 DPO 训练，WebThinker-32B 在 WebWalkerQA 上比 Search-o1 高出 +22.9%，在 HLE 上高出 +20.4%，并在 Glaive 报告生成评估中获得 8.0 的总分（超越 Gemini-Deep Research 的 7.9）。

## 动机与问题

大型推理模型（如 DeepSeek-R1、QwQ）通过扩展思维链展示了强大的内部推理能力。然而，它们存在关键局限：

1. **知识边界**：LRM 依赖静态预训练知识；推理过程中无法获取最新或特定领域的信息。
2. **浅层检索集成**：标准 RAG 流水线在推理前进行一次性检索，无法支持迭代式、深度优先的信息收集。
3. **缺乏报告生成**：现有的搜索增强推理系统只能产生简短答案，无法生成全面、结构化的研究报告。

WebThinker 通过将网络探索和报告起草直接嵌入 LRM 的推理过程来解决以上三个问题。

## 方法

### 系统架构

```
+-------------------------------------------------------------------+
|                    WebThinker Reasoning Loop                       |
|                                                                    |
|  +-------------+    +------------------+    +-----------------+    |
|  | <think>     |--->| Deep Web Explorer|--->| <think>         |    |
|  | Reasoning   |    | Module           |    | Continue        |    |
|  | (identify   |    |                  |    | Reasoning       |    |
|  |  knowledge  |    | search(query)    |    | with new info   |    |
|  |  gaps)      |    | click(element)   |    |                 |    |
|  +-------------+    | extract(content) |    +-----------------+    |
|        |            | navigate(url)    |           |               |
|        v            +------------------+           v               |
|  +-----------+                              +-----------+          |
|  | Produce   |                              | Draft     |          |
|  | Answer    |                              | Report    |          |
|  | (QA mode) |                              | (report   |          |
|  +-----------+                              |  mode)    |          |
|                                             +-----------+          |
+-------------------------------------------------------------------+
```

### 深度网络探索器模块

深度网络探索器使 LRM 能够超越单一查询搜索：

```
Capability 1: search(query) -> ranked results
  - Issue search queries to web search engines
  - Receive ranked list of result snippets

Capability 2: click(link/button) -> page content
  - Navigate to specific URLs from search results
  - Click interactive elements (links, buttons) on pages

Capability 3: extract(selector) -> relevant content
  - Extract specific information from loaded pages
  - Parse and filter page content for relevance

Capability 4: navigate(url) -> page content
  - Follow deeper links discovered during exploration
  - Enables multi-hop web traversal
```

与标准 RAG 的关键区别：LRM 在推理过程中基于其不断发展的理解，自主决定何时搜索、搜索什么以及搜索到多深。

### 两种运行模式

**模式一：问题求解（问答）**
```
while question not answered:
    <think> Reason about current knowledge state </think>
    if knowledge_gap_detected:
        <search> query </search>          # or click/navigate
        <result> web content </result>
    <think> Integrate new information </think>
<answer> final answer </answer>
```

**模式二：报告生成（思考-搜索-起草）**
```
while report not complete:
    <think> Plan next section / identify gaps </think>
    <search> targeted query for section </search>
    <result> web content </result>
    <draft> Write/revise report section </draft>
    <think> Assess completeness, plan next steps </think>
<report> final structured report </report>
```

### 通过迭代在线 DPO 进行强化学习训练

```
Algorithm: Iterative Online DPO for WebThinker
-------------------------------------------------
1. Start with base LRM (e.g., DeepSeek-R1-distill-Qwen-32B)
2. for each DPO iteration:
3.   Sample questions from training set
4.   Generate multiple reasoning trajectories (with web search)
5.   Evaluate trajectories by outcome quality
6.   Construct preference pairs: (better_trajectory, worse_trajectory)
7.   Update model via DPO loss:
8.     L_DPO = -log sigma(beta * (log pi(y_w|x) - log pi(y_l|x)))
9.   Use updated model as new reference for next iteration
```

迭代在线方法从不断改进的模型中持续生成新的偏好数据，避免了静态偏好数据集带来的分布偏移。

## 关键创新

1. **深度网络探索器**：从单一查询 RAG 进化到多步网络导航，将点击、导航和提取能力嵌入推理循环。

2. **自主思考-搜索-起草**：首个将推理、网络探索和报告写作统一到单一交叉过程中的框架。

3. **迭代在线 DPO**：通过基于策略的偏好优化持续改进工具使用，避免了 PPO/GRPO 的不稳定性同时仍然支持探索。

4. **双模式运行**：同一架构同时处理短答案问答和长文本报告生成任务。

## 实验设置

### 基准测试

| 基准测试    | 任务类型                   | 指标        |
|-------------|----------------------------|---------------|
| GPQA         | 研究生级别科学问答  | 准确率      |
| GAIA         | 通用 AI 助手       | 准确率      |
| WebWalkerQA  | 网络导航问答          | 准确率      |
| HLE          | 困难长文本评估  | 准确率      |
| Glaive       | 科学报告撰写  | 多维度 /10 |

### 模型变体

- **WebThinker-32B-Base**：带有深度网络探索器的 DeepSeek-R1-distill-Qwen-32B（无 DPO）。
- **WebThinker-32B-RL**：经过迭代在线 DPO 训练后。
- **WebThinker-7B 变体**：使用 DeepSeek-R1-7B 骨干网络。

### 基线

- **直接生成**：无任何检索的 LRM。
- **标准 RAG**：推理前一次性检索。
- **Search-o1**：先前的搜索增强推理系统。
- **专有系统**：Gemini-Deep Research 等商业系统。

## 结果

### 问答基准测试

| 模型              | GPQA  | GAIA  | WebWalkerQA | HLE   |
|-------------------|-------|-------|-------------|-------|
| WebThinker-32B-RL | 70.7% | 最佳  | 最佳        | 15.8% |
| Search-o1         | 较低 | 较低 | -22.9%      | -20.4%|

- 在 WebWalkerQA 上比 Search-o1 高出 **+22.9%**。
- 在 HLE 上比 Search-o1 高出 **+20.4%**。
- GPQA（研究生级别科学）准确率达 **70.7%**。
- 跨基准比基线最高提升 **+21.5%**。

### 使用 7B 骨干网络的相对提升

- GAIA 上比直接生成提升 **+174.4%**。
- WebWalkerQA 上比直接生成提升 **+422.6%**。
- GAIA 上比标准 RAG 提升 **+82.9%**。
- WebWalkerQA 上比标准 RAG 提升 **+161.3%**。

### 报告生成（Glaive 基准测试）

| 系统              | 总分 | 完整性 | 全面性 | 事实性 | 连贯性 |
|--------------------|---------|-------------|-------------|-----------|----------|
| WebThinker-32B     | **8.0** | **8.4**     | **8.2**     | 7.7       | 7.8      |
| Gemini-Deep Research| 7.9    | 较低       | 较低       | ~7.7      | ~7.8     |

WebThinker 取得最高总分，超越 Gemini-Deep Research。

## 局限性

1. **LRM 骨干依赖**：性能与基础 LRM 质量紧密相关；较弱的骨干网络显示较小的绝对提升。
2. **延迟**：多步网络探索比单一查询 RAG 方法增加了显著的推理时间。
3. **网页复杂性**：深度网络探索器可能难以处理高度动态、JavaScript 密集型的页面或需要认证的页面。
4. **DPO 训练范围**：迭代 DPO 改进了工具使用，但可能无法充分探索偏离初始策略的新研究策略。
5. **报告评估**：Glaive 基准测试使用基于 LLM 的评分，可能无法完全捕捉人类对报告质量的偏好。

## 后续工作

- **Co-STORM** 和 **STORM**：用于协作和结构化文章写作的相关系统。
- **DeepResearcher**：在实时网络环境中使用 GRPO 的替代强化学习方法。
- 与多模态 LRM 集成，用于涉及图像、图表和表格的研究。
- 扩展到特定领域的研究（医学、法律、科学文献）。

## 核心要点

1. 将网络探索直接嵌入 LRM 的推理循环（而非作为单独的预处理步骤）能够实现显著更深入和更有针对性的信息收集。
2. 思考-搜索-起草策略证明了推理、检索和生成可以在单一交叉过程中统一用于报告撰写。
3. 迭代在线 DPO 是训练工具增强推理智能体的有效且稳定的替代方案，可替代 PPO/GRPO。
4. 即使是相对较小的模型（7B），在配备推理过程中的自主网络探索时，也展现出戏剧性的提升（+174-422%）。
5. WebThinker 的报告生成质量（8.0）匹配或超越商业系统如 Gemini-Deep Research（7.9），验证了深度研究的开源方法。
