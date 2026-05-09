---
title: "AgentIR: Reasoning-Aware Retrieval for Deep Research Agents"
authors:
  - Zijian Chen
  - Xueguang Ma
  - Shengyao Zhuang
  - Jimmy Lin
  - Akari Asai
  - Victor Zhong
affiliations:
  - University of Waterloo
  - University of Queensland
  - Carnegie Mellon University
date: 2026-03-04
venue: arXiv
paper_url: https://arxiv.org/abs/2603.04384
code_url: https://github.com/texttron/AgentIR
data_url: https://huggingface.co/datasets/Tevatron/AgentIR-data
project_url: https://texttron.github.io/AgentIR/
tags:
  - deep-research
  - retrieval
  - reasoning-aware
  - embedding-model
  - agentic-search
  - contrastive-learning
---

# AgentIR: Reasoning-Aware Retrieval for Deep Research Agents

## 简要总结

AgentIR 提出了一种面向深度研究智能体的**推理感知检索范式**（Reasoning-Aware Retrieval），核心思想是将智能体在搜索前产生的推理轨迹（reasoning trace）与查询（query）进行联合嵌入，从而大幅提升检索质量。该工作同时提出了 DR-Synth 数据合成方法，能够从标准问答数据集自动生成深度研究检索器的训练数据。两个组件的结合产生了 AgentIR-4B 嵌入模型，在 BrowseComp-Plus 基准上实现了 68% 的准确率，相比传统嵌入模型（50%）取得了显著提升，绝对改进达 +17.60%。

## 动机与问题

### 核心观察

深度研究智能体（Deep Research Agents）正在迅速成为现代检索系统的主要消费者。然而，现有检索系统的设计仍然以人类用户为中心：

1. **人类用户**发出查询并迭代修改，但不会记录中间思维过程
2. **深度研究智能体**在每次搜索调用前会生成**显式的自然语言推理**，揭示了丰富的意图和上下文信息

### 被忽视的信号

现有检索器完全忽略了智能体推理轨迹中蕴含的丰富语义信号。这些推理轨迹包含了：
- 智能体当前的搜索意图
- 已获取信息的总结
- 对下一步搜索方向的规划
- 对查询的深层理解和分解

### 研究目标

如何设计一种新的检索范式，能够利用智能体独有的推理轨迹信号，提升深度研究场景下的检索效果？

## 方法

### 1. Reasoning-Aware Retrieval（推理感知检索）

**核心架构**：将智能体的推理轨迹与查询进行联合编码，作为检索模型的输入。

传统检索模型的输入为：
```
input = query
```

推理感知检索的输入为：
```
input = [reasoning_trace; query]
```

其中 `reasoning_trace` 是智能体在发出搜索请求前生成的自然语言推理过程。该方法将推理轨迹视为查询的上下文增强信号，通过联合嵌入捕获更丰富的检索意图。

**骨干模型**：所有方法均使用 **Qwen3-Embed-4B** 作为嵌入模型的骨干网络。

**训练方式**：采用**对比学习**（contrastive learning）进行训练，使检索器学会利用推理-查询对（reasoning-query pairs）进行有效检索。

### 2. DR-Synth（深度研究数据合成管线）

DR-Synth 是一个数据合成管线，用于从传统 QA 数据集生成子查询级别的训练数据。

**合成流程**：
1. **输入**：标准 QA 数据集（包含问题和答案）
2. **智能体滚动（Agent Rollouts）**：运行深度研究智能体对 QA 问题进行多轮搜索
3. **推理轨迹收集**：记录智能体在每个搜索步骤前的推理过程
4. **Oracle 重排序**：使用 LLM "Oracle" 对检索结果进行重排序
5. **标签生成**：为中间子查询创建黄金标准标签
6. **输出**：包含 (推理轨迹, 查询, 正例文档, 负例文档) 的训练数据

**关键优势**：DR-Synth 无需人工标注，可以低成本地从现有 QA 数据集自动扩展训练数据。

### 3. AgentIR-4B 模型

AgentIR-4B 是结合推理感知检索和 DR-Synth 训练数据训练得到的最终嵌入模型：
- 基于 Qwen3-Embed-4B 骨干
- 使用 DR-Synth 合成的训练数据
- 通过对比学习优化
- 支持推理轨迹与查询的联合编码

## 关键创新

1. **推理轨迹作为检索信号**：首次提出利用深度研究智能体的推理轨迹作为检索增强信号，这是传统人类搜索场景中不存在的独特信号源
2. **DR-Synth 数据合成**：提出了一种低成本的自动化数据合成方法，能够将标准 QA 数据集转化为深度研究检索训练数据
3. **两个组件的独立有效性**：推理感知检索和 DR-Synth 分别独立有效，且组合后效果进一步提升
4. **跨智能体泛化能力**：AgentIR-4B 虽然仅在一个特定智能体（Tongyi-DR）上训练，但能够直接泛化到其他具有不同推理模式的智能体（如 gpt-oss-120B 和 GLM-4.7），无需额外微调

## 实验结果

### BrowseComp-Plus 主实验

| 方法 | 准确率 |
|------|--------|
| BM25 | 37% |
| 传统嵌入模型（2倍大小） | 50% |
| AgentIR-4B + Tongyi-DR | **68%** |
| AgentIR-4B + Tongyi-DR（对比骨干） | 66.27%（+17.60% 绝对提升） |

### 检索性能提升

- **Recall**：从 59% 提升至 78%（+19% 绝对提升）
- **多跳任务准确率**：+18% 绝对提升
- 在多跳复杂任务上的提升尤为显著

### 效率提升

AgentIR-4B 在提高准确率的同时，实际**减少了总搜索步数**，说明更好的检索质量能够帮助智能体更快找到目标信息。

### 跨智能体泛化

| 智能体 | 使用传统检索 | 使用 AgentIR-4B |
|--------|-------------|----------------|
| Tongyi-DR | 基准 | +17.60% |
| gpt-oss-120B | 基准 | 显著提升 |
| GLM-4.7 | 基准 | 显著提升 |

所有智能体均无需额外训练即可受益于 AgentIR-4B。

### 消融实验

- **仅推理感知检索**（不使用 DR-Synth 数据）：显著提升
- **仅 DR-Synth 数据**（不使用推理轨迹）：显著提升
- **两者结合**：最佳性能，证明两个组件互补

## 局限性

1. **推理轨迹依赖**：该方法的效果依赖于智能体生成高质量推理轨迹的能力。如果智能体的推理质量较差，推理感知检索的增益可能有限
2. **模型规模**：AgentIR-4B 基于 4B 参数的嵌入模型，在资源受限场景下可能部署成本较高
3. **DR-Synth 数据质量**：合成数据的质量受限于 Oracle LLM 的重排序能力和源 QA 数据集的覆盖范围
4. **评估范围**：主要在 BrowseComp-Plus 上评估，对其他深度研究基准的泛化性尚需进一步验证
5. **推理轨迹格式**：不同智能体的推理轨迹格式各异，虽然实验证明了跨智能体泛化能力，但对推理轨迹格式的鲁棒性分析仍不够充分

## 核心要点

1. 深度研究智能体在搜索前生成的推理轨迹是一个被现有检索系统完全忽视的强信号源
2. 推理感知检索通过联合嵌入推理轨迹和查询，显著提升了检索质量
3. DR-Synth 提供了一条低成本的数据合成路径，解决了深度研究检索训练数据稀缺的问题
4. AgentIR-4B 在 BrowseComp-Plus 上达到 68% 准确率，比传统方法提升 17.60%
5. 跨智能体泛化能力表明推理轨迹信号具有普适性，不依赖于特定智能体架构
6. 更好的检索质量不仅提升了最终准确率，还减少了所需的搜索步数，提高了整体效率
7. 该工作指出了深度研究检索系统设计的新方向：从人类中心转向智能体中心
