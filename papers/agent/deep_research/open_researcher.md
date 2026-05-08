---
title: "OpenResearcher: A Fully Open Pipeline for Long-Horizon Deep Research Trajectory Synthesis"
authors:
  - TIGER-AI-Lab (full author list on paper)
venue: Preprint (under review)
year: 2025
url: https://arxiv.org/abs/2603.20278
tags:
  - deep-research
  - trajectory-synthesis
  - supervised-fine-tuning
  - browser-agent
  - open-source
  - BrowseComp
status: done
---

# OpenResearcher：用于长程深度研究轨迹合成的完全开放流水线

## 摘要

OpenResearcher 是一个完全开放、可复现的流水线，用于大规模合成长程深度研究轨迹。它将一次性的语料库构建与多轮轨迹生成解耦，使用三个浏览器原语（搜索、打开、查找）操作一个包含 1500 万文档的离线语料库。GPT-OSS-120B 作为教师模型生成了 9.7 万条轨迹（长尾部分包含 100 次以上的工具调用）。在 30B-A3B 骨干网络上进行 SFT 后，在 BrowseComp-Plus 上达到 54.8%（比基础模型高 +34.0），超越了 GPT-4.1、Claude-Opus-4、Gemini-2.5-Pro 和 DeepSeek-R1。

## 动机与问题

通过在线强化学习（如 DeepResearcher）训练深度研究智能体成本高昂且不可复现，原因如下：
1. **搜索 API 依赖**：实时网络结果随时间变化；训练不具有确定性。
2. **成本和速率限制**：强化学习推演期间大规模并行 API 调用的成本过高。
3. **缺乏开放数据**：不存在大规模、高质量的深度研究轨迹数据集用于监督训练。

OpenResearcher 提出的问题是：我们能否完全离线生成高质量的研究轨迹，并通过监督微调实现有竞争力或更优的性能？

## 方法

### 流水线概览

```
+-------------------+     +--------------------+     +-------------------+
| Stage 1: Corpus   |     | Stage 2: Trajectory|     | Stage 3: SFT      |
| Bootstrapping     |---->| Synthesis          |---->| Training          |
| (one-time)        |     | (multi-turn)       |     | (30B-A3B)         |
+-------------------+     +--------------------+     +-------------------+
        |                         |                          |
  15M documents             97K trajectories           OpenResearcher
  ~11B tokens               100+ tool calls            -30B-A3B
  Self-built retriever      GPT-OSS-120B teacher       
```

### 阶段一：语料库构建

- 从多元化网络来源构建 **1500 万文档语料库**（约 110 亿词元）。
- 在该语料库上构建 **自建检索器**，消除对外部搜索 API 的依赖。
- 一次性处理：语料库完成索引后可在所有轨迹合成运行中重复使用。

### 阶段二：基于浏览器原语的轨迹合成

三个显式浏览器原语定义了动作空间：

```
PRIMITIVE: search(query) -> list[document_snippets]
  - Executes semantic search over the 15M-document corpus
  - Returns ranked list of relevant passages

PRIMITIVE: open(url/doc_id) -> full_document_content  
  - Opens and renders the full content of a specific document
  - Enables deep reading beyond snippet-level information

PRIMITIVE: find(pattern, document) -> matched_sections
  - Searches within an opened document for specific information
  - Enables targeted extraction from long documents
```

### 轨迹生成过程

```
Algorithm: Long-Horizon Trajectory Synthesis
-------------------------------------------------
Input: Question q, corpus C, teacher model M (GPT-OSS-120B)
Output: Trajectory T = [(action_1, obs_1), ..., (action_n, obs_n), answer]

1. Initialize context with question q
2. while not done:
3.   M generates next action: search(q'), open(doc), or find(pattern)
4.   Execute action against offline corpus C
5.   Append (action, observation) to trajectory T
6.   M decides whether to continue or produce final answer
7. Return complete trajectory T with all intermediate steps
```

生成轨迹的关键特性：
- **长程**：相当部分的尾部轨迹包含 100 次以上的序列工具调用。
- **多轮**：智能体在多个步骤中迭代优化其研究策略。
- **有据可依**：所有观察均来自固定语料库，确保可复现性。

### 阶段三：监督微调

- **骨干网络**：30B-A3B（总参数 300 亿，通过混合专家激活 30 亿）。
- **训练数据**：来自阶段二的 9.6 万条高质量轨迹。
- **蒸馏**：学生模型通过标准下一词预测学习教师的研究策略。

## 关键创新

1. **完全离线流水线**：通过构建自包含的语料库和自定义检索器消除搜索 API 依赖。轨迹完全可复现。

2. **三原语动作空间**：搜索/打开/查找的抽象既简洁又足以捕获全部网络研究行为（广泛搜索、深度阅读、定向提取）。

3. **大规模长程轨迹**：9.7 万条轨迹，其中相当比例包含 100 次以上的工具调用，在轨迹长度和复杂度上远超先前数据集。

4. **GPT-OSS-120B 作为教师**：使用具有原生浏览器工具支持的强大开源教师模型，实现高质量轨迹生成而无需依赖专有 API。

5. **紧凑的学生模型**：30B-A3B 架构（通过 MoE 仅 30 亿活跃参数）尽管活跃参数数量较少，但取得了最先进的结果。

## 实验设置

### 基准测试

| 基准测试         | 任务类型                        | 难度    |
|-------------------|----------------------------------|---------------|
| BrowseComp-Plus   | 复杂浏览理解   | 非常难     |
| BrowseComp        | 网络浏览理解       | 难          |
| GAIA              | 通用 AI 助手任务       | 中等-难   |
| xbench-DeepSearch | 深度搜索评估           | 难          |

### 基线

- **专有模型**：GPT-4.1、Claude-Opus-4、Gemini-2.5-Pro
- **开源模型**：DeepSeek-R1、Tongyi-DeepResearch
- **基础模型**：未经 SFT 的 30B-A3B（用于消融实验）

## 结果

### BrowseComp-Plus（主要基准测试）

| 模型                        | 准确率 |
|------------------------------|----------|
| OpenResearcher-30B-A3B       | **54.8%**|
| GPT-4.1                      | < 54.8%  |
| Claude-Opus-4                 | < 54.8%  |
| Gemini-2.5-Pro               | < 54.8%  |
| DeepSeek-R1                  | < 54.8%  |
| Tongyi-DeepResearch          | < 54.8%  |
| 30B-A3B 基础模型（无 SFT）       | 20.8%    |

- 比基础模型 **绝对提升 +34.0**（20.8% -> 54.8%）。
- 超越所有列出的专有和开源基线。

### 跨基准泛化

- 在 BrowseComp、GAIA 和 xbench-DeepSearch 上具有竞争力的性能。
- 证明了离线轨迹合成可以迁移到多样化的评估场景。

## 局限性

1. **语料库时效性**：离线的 1500 万文档语料库会随时间陈旧；无法处理关于近期事件的查询。
2. **教师模型依赖**：轨迹质量受限于 GPT-OSS-120B 的能力。
3. **SFT 上限**：监督微调可能无法发现教师所展示之外的新策略（不像强化学习那样有探索）。
4. **评估范围**：主要在浏览理解上评估；在开放式报告生成任务上测试较少。
5. **MoE 复杂性**：30B-A3B 架构尽管仅有 30 亿活跃参数，但为部署增加了工程复杂性。

## 后续工作

- 可能与强化学习微调集成（如 DeepResearcher 中的 SFT + RL 混合方法）。
- 扩展到多模态研究轨迹（图像、表格、图表）。
- 动态语料库更新以处理时效性查询。
- 将轨迹合成扩展到更长的范围（1000 次以上的工具调用）。

## 核心要点

1. 离线轨迹合成是训练深度研究智能体的可行替代方案，可替代昂贵的在线强化学习，仅通过 SFT 即可实现最先进的结果。
2. 搜索/打开/查找原语集是网络研究行为的一组强大的最小抽象。
3. 长程轨迹（100 次以上的工具调用）对于教授智能体处理复杂的多步研究任务至关重要。
4. 紧凑的 MoE 架构（30 亿活跃参数）在使用高质量研究轨迹训练时，可以匹配或超越更大的稠密模型。
5. 完全开源数据集、模型和训练方案，使深度研究智能体的可复现研究成为可能。
