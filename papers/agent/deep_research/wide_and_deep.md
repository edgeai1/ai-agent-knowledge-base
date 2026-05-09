---
title: "W&D: Scaling Parallel Tool Calling for Efficient Deep Research Agents"
authors:
  - Xiaoqiang Lin
  - Jun Hao Liew
  - Silvio Savarese
  - Junnan Li
affiliations:
  - Salesforce AI Research
date: 2026-02-07
venue: arXiv
paper_url: https://arxiv.org/abs/2602.07359
project_url: https://xqlin98.github.io/wide-deep-research-agent/
code_url: https://github.com/SalesforceAIResearch/MCP-Universe
tags:
  - deep-research
  - parallel-tool-calling
  - width-scaling
  - depth-scaling
  - efficiency
  - agentic-search
---

# W&D: Scaling Parallel Tool Calling for Efficient Deep Research Agents

## 简要总结

W&D（Wide and Deep）是 Salesforce AI Research 提出的深度研究智能体框架，系统性地研究了**宽度扩展**（width scaling，即并行工具调用）对智能体性能的影响。该工作发现：在现有方法主要通过增加深度（sequential thinking and tool calls）提升性能的基础上，**通过并行工具调用扩展宽度**同样能显著提升性能，同时减少交互轮次、API 成本和端到端延迟。在 BrowseComp 上，使用 GPT-5-Medium 的 W&D 达到 **62.2% 准确率**，超过了 GPT-5-High Deep Research 的 54.9%。在 100 个 BrowseComp 任务上，并行调用（3 tools/turn）相比串行调用实现了约 **36% 的成本降低**和约 **41% 的时间降低**。

## 动机与问题

### 现有方法的偏向

近期研究主要通过扩展**深度**来增强深度研究智能体的能力：
- 增加顺序思考和工具调用的步数
- 增加推理链的长度
- 增加搜索的轮次

但是，**宽度维度**（即并行工具调用的数量）在很大程度上未被探索。

### 多智能体方案的局限

现有并行化方案主要依赖复杂的**多智能体编排**（multi-agent orchestration），带来了：
- 高系统复杂度
- 智能体间通信开销
- 任务分配和结果融合的挑战

### 研究问题

1. 并行工具调用能否在单个推理步骤内实现有效协调？
2. 宽度和深度的扩展如何影响深度研究智能体的性能？
3. 最优的工具调用调度策略是什么？

## 方法

### 1. 并行工具调用框架

**核心思想**：在单个推理步骤中，模型发出多个工具调用请求，这些调用并行执行，结果一同返回到智能体的推理轨迹中。

```
传统串行调用：
Step 1: Think → Tool Call 1 → Result 1
Step 2: Think → Tool Call 2 → Result 2
Step 3: Think → Tool Call 3 → Result 3

W&D 并行调用：
Step 1: Think → [Tool Call 1, Tool Call 2, Tool Call 3] → [Result 1, Result 2, Result 3]
```

**关键区别**：不同于多智能体系统，W&D 利用模型**内在的并行工具调用能力**（intrinsic parallel tool calling），在单一推理步骤中协调多个工具调用。

### 2. 工具调用调度策略

**固定调度（Fixed Scheduler）**：
- 配置模式："100: 3" — 每步固定 3 个并行工具调用，持续 100 轮迭代
- 简单且稳定的基准策略

**动态调度（Dynamic Scheduler）**：
- 模型自行决定每步的工具调用数量
- 范围：最少 1 个，最多 4 个
- 基于当前已收集的信息动态调整

**递减调度（Descending Scheduler）**：
- **先探索，后利用**（explore early, exploit later）
- 初期阶段发出更多并行调用以广泛探索
- 后期阶段减少并行调用以深入利用
- 相比固定调度额外提升约 **~6%**

### 3. 宽度与深度的扩展分析

**宽度扩展（Width Scaling）**：
- 增加每步的并行工具调用数量
- 最优并行数为 **3 tools/turn**
- 过多并行调用可能导致信息过载

**深度扩展（Depth Scaling）**：
- 增加总的推理和搜索步数
- 传统方法的主要扩展维度

**宽度 + 深度联合扩展**：
- W&D 证明两个维度可以互补
- 宽度扩展可以在减少深度的同时提升性能

### 4. 信源可信度提升

并行调用触发多个查询并聚合多样化的信息源，使模型能够：
- 比较不同来源的信息
- 选择最权威的信源（如官方联合国报告 vs. 第三方 API）
- 通过多源验证提升答案的可靠性

## 关键创新

1. **宽度维度的系统性研究**：首次系统性地研究了并行工具调用对深度研究智能体性能的影响
2. **内在并行调用**：利用模型原生的并行工具调用能力，无需多智能体编排
3. **递减调度策略**：提出 "先探索后利用" 的调度策略，额外提升 ~6%
4. **跨模型跨基准泛化**：证明宽度扩展的增益在 GPT-5、Gemini、Claude 等不同 LLM 和多个基准上均一致成立
5. **效率与性能双赢**：并行调用不仅提升准确率，还显著减少了成本和时间

## 实验结果

### BrowseComp 主实验

| 模型和设置 | 准确率 | 成本 | 时间 |
|-----------|--------|------|------|
| 串行调用（1 tool/turn） | 66% | ~$102.5 | ~1523s |
| **并行调用（3 tools/turn）** | **68%** | **~$65.7** | **~904s** |
| 成本/时间降低 | +2% 准确率 | **-36%** | **-41%** |

### GPT-5 BrowseComp 结果

| 设置 | 准确率 |
|------|--------|
| GPT-5-High Deep Research（OpenAI 基线） | 54.9% |
| **W&D + GPT-5-Medium** | **62.2%** |

W&D 使用 Medium 级别的 GPT-5 超越了 High 级别的 GPT-5 Deep Research，且无需上下文管理或其他技巧。

### 跨模型实验

| 模型 | 串行 | 并行（W&D） | 提升 |
|------|------|------------|------|
| GPT-5 | 基准 | 提升 | 一致 |
| Gemini 3.0 Pro | 基准 | 提升 | 一致 |
| Claude 4.5 Sonnet | 基准 | 提升 | 一致 |

所有测试的前沿 LLM 均从宽度扩展中获益。

### 跨基准实验

| 基准 | 串行 | 并行（W&D） |
|------|------|------------|
| BrowseComp | 提升 | 一致 |
| HLE（Humanity's Last Exam） | 提升 | 一致 |
| GAIA | 提升 | 一致 |

### 调度策略对比

| 调度策略 | 相对固定调度的提升 |
|---------|------------------|
| 固定调度（3/turn） | 基准 |
| 动态调度 | 小幅提升 |
| **递减调度** | **~6%** |
| 递增调度 | 较差 |

递减策略（先广后深）显著优于递增策略（先深后广），验证了"先探索后利用"的直觉。

### 轮次减少

并行工具调用显著减少了智能体找到正确答案所需的轮次数。宽度扩展让智能体在更少的交互轮次中覆盖更多的信息空间。

## 局限性

1. **上下文窗口压力**：并行工具调用返回的多个结果同时进入上下文，可能导致上下文窗口过载，尤其在长程任务中
2. **最优并行数的确定**：3 tools/turn 是经验性最优值，对不同任务类型和模型可能需要不同设置
3. **工具调用独立性假设**：并行调用假设工具调用之间相互独立，但在某些场景下后续调用可能依赖前续结果
4. **信息聚合挑战**：大量并行返回的信息需要模型具备更强的信息整合和筛选能力
5. **API 限流**：实际部署中，搜索 API 的并发限制可能制约并行调用的扩展
6. **缺乏 RL 训练**：当前工作主要基于 prompting 和推理，未探索通过 RL 训练优化并行调用策略
7. **成本分析的完整性**：成本分析主要考虑 API 调用成本，未充分考虑并行执行的基础设施成本

## 核心要点

1. 宽度扩展（并行工具调用）是深度研究智能体的一个重要但被忽视的性能提升维度
2. 利用模型内在的并行工具调用能力，无需复杂的多智能体编排即可实现有效并行化
3. 最优并行数为每步 3 个工具调用，既能覆盖足够信息又不至于信息过载
4. 递减调度策略（先探索后利用）比固定和递增策略更有效，额外提升约 6%
5. W&D + GPT-5-Medium（62.2%）超越 GPT-5-High Deep Research（54.9%），证明宽度扩展的巨大潜力
6. 并行调用在提升准确率的同时降低约 36% 的成本和 41% 的时间
7. 宽度扩展的增益跨模型（GPT-5、Gemini、Claude）和跨基准（BrowseComp、HLE、GAIA）一致成立
