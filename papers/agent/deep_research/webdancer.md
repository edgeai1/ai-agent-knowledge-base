---
title: "WebDancer: Towards Autonomous Information Seeking Agency"
authors:
  - Jialong Wu
  - Baixuan Li
  - Runnan Fang
  - Wenbiao Yin
  - et al.
year: 2025
month: 5
venue: "NeurIPS 2025 / arXiv:2505.22648"
institution: "Tongyi Lab, Alibaba Group"
url: "https://arxiv.org/abs/2505.22648"
code: "https://github.com/Alibaba-NLP/WebAgent"
tags:
  - web-agent
  - information-seeking
  - deep-research
  - reinforcement-learning
  - SFT
  - ReAct
  - agentic-search
---

# WebDancer: Towards Autonomous Information Seeking Agency

## 简要总结

WebDancer 是由阿里巴巴通义实验室提出的面向自主信息搜寻的原生智能体搜索模型（Native Agentic Search Model）。该工作提出了一套完整的端到端范式，从数据构建和训练两个维度系统性地构建信息搜寻智能体。核心流程包含四个阶段：浏览数据构建、轨迹采样、监督微调（SFT）冷启动、以及强化学习（RL）增强泛化。基于 ReAct 框架实例化的 WebDancer 在 GAIA 上达到 Pass@3 64.1%、WebWalkerQA 上达到 62.0%，展现出强大的复杂信息搜寻能力。

## 动机与问题

### 核心问题

当前大语言模型（LLM）在面对需要多步骤、多工具交互的复杂信息搜寻任务时，存在严重不足。现有方法通常依赖闭源模型（如 GPT-4o）作为后端，缺乏端到端的自主搜索能力。具体挑战包括：

1. **数据稀缺问题**：高质量的多步骤网页浏览轨迹数据极为匮乏，难以支撑大规模训练
2. **动作空间复杂**：网页浏览涉及搜索（search）、点击（click）、滚动（scroll）等多种操作，动作空间的组合爆炸使得策略学习极为困难
3. **长程推理困难**：信息搜寻任务需要模型在多个网页间进行长链条推理，现有模型在长轨迹上的表现退化严重
4. **泛化能力不足**：仅通过 SFT 训练的模型往往过拟合于训练分布，难以泛化到新的查询类型和网页结构

### 研究动机

该工作的核心动机是构建一个类似 Deep Research 的原生智能体搜索模型，使其能够自主地在互联网上进行信息搜寻和推理，而无需依赖外部闭源模型。WebDancer 属于阿里通义 WebAgent 系列的第二代工作，在前作 WebWalker 的基础上，从数据构建和训练两个维度进行了系统性升级。

## 方法

### 整体框架

WebDancer 提出了一套系统性的四阶段范式，涵盖从数据准备到模型训练的完整链路：

#### 阶段一：浏览数据构建

提出两种互补的数据合成策略：

- **CRAWLQA（爬取问答）**：通过递归爬取知识密集型网站（如 arXiv、GitHub、Wikipedia）的子页面，使用 GPT-4o 基于爬取内容合成问答对。问题类型涵盖 COUNT（计数）、MULTI-HOP（多跳）、INTERSECTION（交叉）等，模拟人类深度浏览行为
- **E2HQA（由易到难问答）**：从简单的初始问题 Q1 出发，在第 i 次迭代中利用从实体 Ei 检索到的新信息 Ci 对问题进行进化，使得任务复杂度从简单到困难逐步提升

#### 阶段二：轨迹采样

采用两种思维链（CoT）策略生成高质量的 "Thought-Action-Observation" 序列：

- **短 CoT**：直接使用 GPT-4o 生成推理轨迹
- **长 CoT**：使用大型推理模型（如 QwQ-Plus）生成更深入的推理轨迹

通过三阶段过滤框架确保质量：有效性控制、正确性验证、以及基于信息非冗余性和逻辑推理准确性的质量评估。

#### 阶段三：SFT 冷启动

使用过滤后的 ReAct 格式轨迹对策略模型进行微调。训练格式采用标准化标签：
- `<think>`, `</think>`：思考过程
- `<tool_call>`, `</tool_call>`：工具调用
- `<tool_response>`, `</tool_response>`：工具响应
- `<answer>`, `</answer>`：最终答案

#### 阶段四：RL 增强泛化

在 SFT 基础上，采用 DAPO（Decoupled Clip and Dynamic Sampling Policy Optimization）算法进行强化学习训练：

- 使用基于结果的奖励（outcome-based rewards）进行策略优化
- DAPO 通过解耦裁剪和动态采样机制，有效处理长轨迹上的稀疏奖励问题
- RL 阶段主要提升模型在复杂推理任务上的泛化能力和一致性

### 基础模型

以 QwQ-32B 作为骨干模型，基于 ReAct 框架构建 Web 智能体。

## 关键创新

1. **端到端数据驱动范式**：首次从数据构建和训练两个维度系统性地解决信息搜寻智能体的构建问题，形成完整的数据-训练闭环
2. **CRAWLQA 与 E2HQA 双重数据合成**：两种数据合成策略互补，CRAWLQA 通过爬取真实网页生成深层查询，E2HQA 通过迭代进化实现难度可控的数据增强
3. **多级质量过滤**：三阶段过滤框架（有效性-正确性-质量）确保训练数据的高质量
4. **SFT + RL 两阶段训练**：SFT 冷启动建立基本能力，RL（DAPO）提升泛化和一致性，两者协同互补
5. **原生智能体搜索模型**：不依赖外部闭源模型，实现端到端的自主信息搜寻
6. **ReAct 框架的标准化扩展**：通过标准化的标签体系（think/tool_call/tool_response/answer）对 ReAct 框架进行了规范化，提升了可复现性

## 实验结果

### 主要基准测试

| 基准测试 | 指标 | WebDancer | 原始 ReAct |
|----------|------|-----------|------------|
| GAIA | Pass@3 | **64.1%** | - |
| GAIA | 平均 | 51.5% | 37.8% |
| WebWalkerQA | Pass@3 | **62.0%** | - |
| WebWalkerQA | 平均 | 47.9% | 24.1% |

### 关键发现

- 相较于原始 ReAct 基线，WebDancer 在 GAIA 上提升约 13.7 个百分点，在 WebWalkerQA 上提升约 23.8 个百分点
- RL 训练对非推理模型的 Pass@3 和 Cons@3 指标提升显著
- 对于大型推理模型（LRM），RL 主要增强一致性而非 Pass@1，这是因为长轨迹上奖励稀疏
- WebDancer 在 BrowseComp 等更具挑战性的基准上也进行了评估
- SFT 阶段单独即可带来显著提升（GAIA 从 37.8% 到约 48%），但 RL 阶段进一步提升了泛化能力
- 数据量和数据质量的平衡是 SFT 阶段成功的关键因素
- 短 CoT 轨迹训练效率高但推理深度有限，长 CoT 轨迹推理深入但采样成本高，两者结合效果最优
- WebDancer 在处理需要跨多个网页整合信息的复杂查询时，相较于基线的优势最为显著

### 消融实验

消融分析涵盖以下关键问题：

- **数据构建策略的贡献**：CRAWLQA 和 E2HQA 两种策略在 SFT 阶段对智能体性能的贡献各有侧重，两者结合效果最优
- **SFT 冷启动的必要性**：SFT 对建立多步骤、多工具的指令遵循能力至关重要
- **RL 的增量价值**：RL 使模型能够进行更长的推理过程，支持更复杂的智能体动作

## 局限性

1. **骨干模型依赖**：WebDancer 基于 QwQ-32B，模型规模较大，部署成本高
2. **长轨迹奖励稀疏**：RL 阶段在长轨迹上的奖励信号稀疏，限制了 Pass@1 性能的提升
3. **数据合成成本**：CRAWLQA 和 E2HQA 的数据合成依赖 GPT-4o 等闭源模型，合成成本较高
4. **动态网页适应性**：当前方法主要针对静态网页内容，对动态交互式网页的适应能力有待验证
5. **评估基准有限**：主要在 GAIA 和 WebWalkerQA 上进行评估，对更多样化的信息搜寻场景覆盖有限
6. **多语言能力**：当前评估集中于英文场景，对中文等其他语言的信息搜寻能力有待验证
7. **安全性考量**：自主网页浏览涉及潜在的安全风险（如访问恶意网页），相关安全机制的讨论不够充分

## 核心要点

- WebDancer 是通义实验室推出的原生智能体搜索模型，代表了从"LLM + 工具调用"到"端到端智能体搜索"的范式转变
- 四阶段范式（数据构建 -> 轨迹采样 -> SFT 冷启动 -> RL 泛化）为构建信息搜寻智能体提供了可复现的系统性方法论
- CRAWLQA（爬取式）和 E2HQA（由易到难式）两种数据合成策略是该工作的核心技术贡献之一
- DAPO 算法在 RL 阶段有效处理了长轨迹稀疏奖励问题
- 该工作属于阿里通义 WebAgent 系列（WebWalker -> WebDancer -> WebSailor），代表了开源信息搜寻智能体的前沿水平
- 在 GAIA Pass@3 达到 64.1%，证明开源模型在信息搜寻任务上可以达到接近闭源系统的性能
- 数据质量是该工作的灵魂：三阶段过滤框架（有效性-正确性-质量）确保了只有高质量轨迹进入训练
- 长短 CoT 的结合（GPT-4o 短 CoT + QwQ-Plus 长 CoT）是一个巧妙的设计，兼顾效率和推理深度
- CRAWLQA 的问题类型设计（COUNT、MULTI-HOP、INTERSECTION）有效覆盖了信息搜寻的核心能力维度
- WebDancer 的成功验证了"数据驱动 + 训练驱动"的双轮驱动范式在构建自主智能体中的有效性
- 该工作被 NeurIPS 2025 接收，是开源信息搜寻智能体领域的重要里程碑
- 与同期工作 WebThinker、AutoAgent 等相比，WebDancer 的优势在于端到端的系统性设计而非单一技术突破
