---
title: "O-Researcher: An Open Ended Deep Research Model via Multi-Agent Distillation and Agentic RL"
authors:
  - Yi Yao
  - He Zhu
  - Piaohong Wang
  - Jincheng Ren
  - Xinlong Yang
  - Qianben Chen
  - Xiaowan Li
  - Dingfeng Shi
  - Jiaxian Li
  - Qiexiang Wang
  - Sinuo Wang
  - Xinpeng Liu
  - Jiaqi Wu
  - Minghao Liu
  - Wangchunshu Zhou
venue: arXiv preprint
year: 2026
url: https://arxiv.org/abs/2601.03743
tags:
  - deep-research
  - multi-agent-distillation
  - agentic-RL
  - open-source
  - knowledge-distillation
  - tool-integrated-reasoning
  - research-agent
status: done
---

## 摘要

O-Researcher 提出了一个框架，旨在缩小闭源和开源 LLM 在深度研究任务上的性能差距。该方法使用多智能体蒸馏来合成高保真的研究级训练数据，随后采用两阶段训练策略，将监督微调（SFT）与新型智能体强化学习方法相结合。该框架捕获了完整的迭代研究过程——从查询规划到最终报告合成——使开源模型在主要深度研究基准测试上达到新的最先进水平。

## 动机与问题

闭源模型（GPT-4o、Claude、Gemini）和开源模型之间的深度研究能力差距主要归因于：

1. **数据差异**：闭源提供商拥有大量专有的高质量训练数据，包括人类反馈、工具使用轨迹和研究工作流。开源模型缺乏可比的数据。

2. **流水线复杂性**：深度研究涉及查询规划、信息检索、证据评估、综合和报告生成的迭代循环。在训练数据中捕获完整的流水线是困难的。

3. **工具集成**：真实的深度研究需要与搜索引擎、数据库和文档分析工具的无缝集成。大多数训练数据缺乏丰富的工具集成推理轨迹。

4. **专有 API 依赖**：现有方法依赖从专有 API（如 GPT-4）进行蒸馏，创造了不可持续的依赖关系和潜在的法律/伦理问题。

先前的工作如 TaskCraft 和基础多智能体蒸馏方法仅捕获了研究过程的片段。O-Researcher 旨在端到端地捕获深度研究的完整、迭代、探索性本质。

## 方法

### 多智能体蒸馏框架

O-Researcher 使用协作多智能体系统生成合成研究数据：

```
+================================================================+
|              MULTI-AGENT DATA SYNTHESIS PIPELINE                 |
+================================================================+
|                                                                  |
|  +------------------+                                            |
|  | Research Query   | (seed queries covering diverse domains)    |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | PLANNER AGENT    | Decomposes query into sub-questions,       |
|  |                  | creates research plan with milestones      |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | RETRIEVER AGENT  | Executes searches, selects sources,        |
|  |                  | manages tool calls (web search, DB)        |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | EVALUATOR AGENT  | Assesses evidence quality, checks          |
|  |                  | relevance, identifies contradictions        |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | SYNTHESIZER      | Combines evidence into coherent findings,  |
|  | AGENT            | resolves conflicts, generates report       |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +--------+---------+                                            |
|  | CRITIC AGENT     | Reviews report for completeness, accuracy, |
|  |                  | triggers re-research if gaps found         |
|  +--------+---------+                                            |
|           |                                                      |
|           v                                                      |
|  +------------------+                                            |
|  | Complete Research | Full trace with all agent interactions,    |
|  | Trajectory       | tool calls, and intermediate reasoning     |
|  +------------------+                                            |
+================================================================+
```

### 两阶段训练策略

```
STAGE 1: Supervised Fine-Tuning (SFT)
+------------------------------------------------------------------+
|  Input: Base open-source model + synthetic research trajectories  |
|                                                                    |
|  Process:                                                          |
|  - Flatten multi-agent trajectories into single-model format      |
|  - Include tool-call tokens and search result formatting          |
|  - Train with standard cross-entropy loss on complete traces      |
|  - Establish baseline research behavior and tool-use patterns     |
|                                                                    |
|  Output: SFT model with basic deep research capability            |
+------------------------------------------------------------------+

STAGE 2: Agentic Reinforcement Learning
+------------------------------------------------------------------+
|  Input: SFT model + research environment with live tool access    |
|                                                                    |
|  Process:                                                          |
|  - Agent conducts research on new queries in interactive env      |
|  - Reward signal combines:                                         |
|    * Final report quality (accuracy, completeness, coherence)      |
|    * Intermediate research quality (query diversity, source        |
|      coverage, evidence relevance)                                 |
|    * Self-correction behavior (identifying and fixing errors)      |
|  - Policy optimization with trajectory-level rewards               |
|  - Emphasis on exploration and iterative refinement behaviors      |
|                                                                    |
|  Output: O-Researcher model with refined research capabilities    |
+------------------------------------------------------------------+
```

### 捕获的研究过程

与先前仅捕获单轮或浅层多轮交互的蒸馏不同，O-Researcher 捕获了完整的研究生命周期：

```
Full Research Lifecycle:

1. QUERY UNDERSTANDING
   - Parse user intent
   - Identify knowledge gaps
   - Determine scope and depth

2. RESEARCH PLANNING
   - Decompose into sub-questions
   - Prioritize investigation order
   - Set milestones and checkpoints

3. ITERATIVE INVESTIGATION
   +---> Search for information
   |     Evaluate retrieved evidence
   |     Identify gaps or contradictions
   +---- Refine queries and re-search (loop)

4. EVIDENCE SYNTHESIS
   - Aggregate findings across sources
   - Resolve conflicting information
   - Build structured argument

5. REPORT GENERATION
   - Produce coherent, well-cited report
   - Include methodology and limitations
   - Provide confidence assessments

6. SELF-REVIEW & REVISION
   - Check completeness against plan
   - Verify factual claims
   - Revise if quality threshold not met
```

## 关键创新

1. **端到端研究轨迹合成**：首个在合成训练数据中捕获完整深度研究过程（从规划到报告合成）的框架，而非零散的子任务完成。

2. **多智能体协作蒸馏**：多个专门化智能体（规划者、检索者、评估者、综合者、评审者）协作产生训练数据，单一模型随后可以从中学习，在不要求单一模型从一开始就具备所有能力的情况下整合专业知识。

3. **针对研究行为的智能体强化学习**：强化学习阶段专门针对研究质量行为（探索、自纠正、证据评估），而非通用指令遵循，使用结合过程和结果的复合奖励。

4. **深度研究的民主化**：使开源模型达到与闭源系统竞争的性能，减少对专有 API 的依赖。

## 实验设置

- **基础模型**：多个规模的开源模型（具体模型名称在搜索结果中未确认；可能是 Qwen 或 Llama 系列）
- **基准测试**：主要深度研究基准测试，包括 GAIA（需要网络浏览和多步推理的通用 AI 助手任务）
- **基线**：闭源模型（GPT-4o、Claude）、先前的开源深度研究智能体、基础蒸馏方法
- **评估**：报告质量（准确性、完整性、连贯性）、工具使用有效性、研究深度、事实正确性

## 结果

### 关键性能声明

- 使用 O-Researcher 训练的开源模型在主要深度研究基准测试上达到了**开源模型的新最先进水平**。
- 模型**显著缩小了**与闭源系统（GPT-4o、Claude）的差距。
- 该框架在**多个模型规模**上有效，证明了该方法不局限于大型模型。
- SFT 和 RL 两个阶段都有意义地贡献：SFT 建立基础能力，RL 优化自纠正和迭代研究行为。

### 上下文参考点（其他深度研究系统）

| 系统               | GAIA 分数（近似） | 类型         |
|---------------------|---------------------|--------------|
| OpenAI Deep Research | 约 67%（平均）          | 闭源|
| 通义深度研究  | 70.9                | 闭源|
| 开源基线 | 约 55%                | 开源  |
| O-Researcher         | 开源最先进| 开源  |

### 消融实验发现

- 多智能体蒸馏比单智能体生成产生更高质量的训练数据，因为每个智能体贡献了专门的专业知识。
- 强化学习阶段至关重要：仅 SFT 产生的模型遵循研究模式但缺乏迭代优化和自纠正能力。
- 研究轨迹的完整性很重要：在完整端到端轨迹上训练的模型优于在零散子任务数据上训练的模型。

## 局限性

1. **基准特定性**：深度研究基准测试（GAIA 等）可能无法完全捕获真实世界研究任务的多样性和难度，这些任务涵盖技术深度、创造性综合和领域专业知识。

2. **数据质量上限**：合成轨迹的质量受限于生成它们的智能体系统的能力。如果多智能体流水线产生系统性错误，这些错误会传播到训练中。

3. **评估困难**：评估"研究质量"本质上是主观的和多维的。自动指标可能无法捕获洞察深度、论证质量或实际效用等方面。

4. **工具环境依赖**：强化学习阶段需要具有实时工具访问（搜索引擎、数据库）的研究环境，这在训练运行之间可能不稳定或不可复现。

5. **规模与泛化**：竞争性能可能需要 32B 以上的模型和大量的强化学习计算。针对深度研究的微调可能会降低通用指令遵循能力；这种权衡未得到充分分析。

## 核心要点

1. 多智能体蒸馏是生成高质量训练数据的强大范式：专门化智能体可以协作产生超越任何单一智能体所能生成的轨迹，为学生模型提供更丰富的学习信号。

2. 两阶段训练方法（SFT 然后 RL）对深度研究具有充分的动机：SFT 提供结构性基础（如何规划、搜索、综合），而 RL 优化自适应行为（何时重新搜索、如何自纠正），这些行为将优质研究与公式化模式匹配区分开来。

3. 在训练数据中捕获完整的研究生命周期至关重要。在零散子任务上训练的模型无法发展出表征真正深度研究的迭代、探索性行为。

4. 开源和闭源研究智能体之间的性能差距比通常认为的要窄得多，通过更好的训练数据和强化学习方法而非原始模型规模可以进一步缩小。

5. 合成数据生成和智能体强化学习的结合可以替代专有数据优势，为深度研究的民主化提供了可扩展的路径。该框架与模型规模互补，可以在基础模型改进时重新应用。
