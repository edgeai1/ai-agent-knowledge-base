---
title: "WebSailor: Navigating Super-human Reasoning for Web Agent"
authors:
  - Kuan Li
  - Zhongwang Zhang
  - Huifeng Yin
  - Liwen Zhang
  - Litu Ou
  - Jialong Wu
  - Wenbiao Yin
  - Baixuan Li
  - Zhengwei Tao
  - Xinyu Wang
  - Weizhou Shen
  - Junkai Zhang
  - Dingchu Zhang
  - Xixi Wu
  - Yong Jiang
  - Ming Yan
  - Pengjun Xie
  - Fei Huang
  - Jingren Zhou
year: 2025
month: 7
venue: "arXiv:2507.02592"
institution: "Tongyi Lab, Alibaba Group"
url: "https://arxiv.org/abs/2507.02592"
code: "https://github.com/Alibaba-NLP/WebAgent"
tags:
  - web-agent
  - information-seeking
  - deep-research
  - reinforcement-learning
  - uncertainty-reduction
  - BrowseComp
  - DUPO
  - agentic-RL
---

# WebSailor: Navigating Super-human Reasoning for Web Agent

## 简要总结

WebSailor 是由阿里巴巴通义实验室提出的面向超人类推理能力的网页智能体后训练方法论。该工作的核心洞察在于：闭源系统（如 DeepResearch）在 BrowseComp 等极端复杂信息搜寻基准上展现出超人类能力，其成功关键在于一种开源模型所缺乏的"系统性不确定性消减"推理模式。基于此洞察，WebSailor 提出了一套完整的后训练流程，包括 SailorFog-QA 数据合成管线、RFT 冷启动、以及 DUPO（Duplicating Sampling Policy Optimization）强化学习算法。WebSailor-72B 在 BrowseComp-en 上达到 12.0%、BrowseComp-zh 上达到 30.1%、GAIA 上达到 55.4%，全面超越所有开源智能体。

## 动机与问题

### 核心问题

1. **超人类推理差距**：闭源系统在 BrowseComp 等超高难度基准上展现出人类专家都难以企及的能力，而开源模型与之存在巨大差距
2. **不确定性处理能力缺失**：现有开源模型缺乏在巨大信息空间中系统性消减不确定性的关键推理模式
3. **训练效率问题**：智能体 RL 训练的采样效率极低，大量 rollout 的奖励方差为零（即全部成功或全部失败），无法产生有效梯度信号
4. **训练数据匮乏**：缺乏专门为高不确定性信息搜寻设计的训练数据

### 关键洞察

闭源 DeepResearch 系统的成功秘诀不在于更大的模型或更多的数据，而在于一种独特的推理模式：在面对巨大的信息不确定性时，能够系统性地通过多步搜索和推理逐步消减不确定性，最终精确定位目标信息。这种能力在开源模型中几乎完全缺失。

## 方法

### 整体框架

WebSailor 的完整后训练方法论包含三个核心阶段：

#### 阶段一：SailorFog-QA 数据合成管线

这是 WebSailor 最核心的创新之一。该管线通过结构化采样和信息模糊化来生成高不确定性任务：

1. **知识图谱构建**：构建稠密互联的知识图谱，反映现实世界中知识的关联方式
2. **问题模板生成**：基于图谱结构提取问题模板
3. **信息模糊化（Information Obfuscation）**：对问题进行系统性模糊处理：
   - 实体名称遮蔽（如将具体人名替换为描述性短语）
   - 时间信息模糊化（如将精确日期替换为模糊时间窗口）
   - 语义关系故意遮掩
   - 歧义事件引用
   - 组合性干扰项
   - 关键关系省略
4. **不确定性注入**：确保生成的问题具有极高的初始不确定性，要求模型进行创造性探索

#### 阶段二：RFT 冷启动

使用拒绝采样微调（Rejection Sampling Fine-Tuning, RFT）在少量高质量示例上进行冷启动：

- 从 SailorFog-QA 数据中选取高质量样本
- 通过拒绝采样获取成功的推理轨迹
- 对模型进行 SFT 建立基线能力

#### 阶段三：DUPO 强化学习

DUPO（Duplicating Sampling Policy Optimization）是专为智能体 RL 设计的高效训练算法：

- **核心问题**：在标准 RL 训练中，大量样本的奖励方差为零（全部正确或全部错误），这些样本无法产生梯度信号，导致采样效率极低
- **解决方案**：DUPO 通过复制具有非零奖励方差的样本，重新构建训练批次
- **效率提升**：相比传统方法，DUPO 将 rollout 效率提升 2-3 倍
- **稳定性**：通过动态调整采样策略，保持训练过程的稳定性

### 模型家族

WebSailor 提供了四种规模的模型：3B、7B、32B、72B，全部基于 Qwen 系列模型训练。

## 关键创新

1. **不确定性消减推理范式**：首次明确指出超人类信息搜寻能力的本质在于系统性不确定性消减，并设计了专门的训练方法来激发这种能力
2. **SailorFog-QA 数据管线**：通过知识图谱构建和信息模糊化，自动生成具有高不确定性的训练任务，是该工作最核心的方法论创新
3. **DUPO 算法**：针对智能体 RL 训练中的采样效率问题，提出通过复制有效样本来提升 rollout 效率的创新方案
4. **向下兼容性**：在复杂不确定性推理模式上训练的模型，在简单任务上同样表现出色，证明了该方法的通用性
5. **规模效应验证**：从 3B 到 72B 的完整模型家族验证了方法的可扩展性

## 实验结果

### BrowseComp 基准（核心评测）

| 模型 | BrowseComp-en | BrowseComp-zh |
|------|--------------|---------------|
| WebSailor-72B | **12.0%** | **30.1%** |
| WebSailor-7B | 6.7 | - |
| WebDancer-32B | 2.5 | - |
| WebThinker-RL | 2.8 | - |

### GAIA 基准

| 模型 | 得分 |
|------|------|
| WebSailor-72B | **55.4%** |

### 关键发现

- WebSailor-72B 全面超越所有开源智能体和基于智能体方法的模型
- WebSailor-7B（6.7 on BrowseComp-en）大幅超越基于更大 32B 模型的智能体（WebDancer-32B: 2.5, WebThinker-RL: 2.8）
- 在 XBench-DeepSearch 和 SimpleQA 等较简单任务上也展现出良好的"向下兼容"性能
- WebSailor 模型超越了 Grok-3 和 DouBao 等闭源推理模型（在搭配浏览能力后）
- DUPO 相比传统 RL 方法将 rollout 效率提升 2-3 倍

### 与闭源系统对比

WebSailor 在复杂信息搜寻任务上匹配了闭源智能体的性能水平，有效弥合了开源与闭源之间的能力差距。

## 局限性

1. **BrowseComp-en 绝对分数仍低**：虽然 12.0% 已大幅领先开源，但绝对分数仍远低于顶级闭源系统，说明极端不确定性推理仍有巨大提升空间
2. **训练资源需求**：72B 模型的 RL 训练需要大量 GPU 资源
3. **知识图谱依赖**：SailorFog-QA 的数据质量高度依赖底层知识图谱的构建质量
4. **评估偏差风险**：信息模糊化策略可能导致训练数据与真实查询分布之间存在偏差
5. **时效性问题**：知识图谱和训练数据的时效性限制了模型对最新信息的搜寻能力

## 核心要点

- WebSailor 的核心贡献是识别并解决了"系统性不确定性消减"这一闭源系统的关键能力差距
- SailorFog-QA 管线通过"知识图谱 + 信息模糊化"生成高不确定性训练任务，是方法论的核心支柱
- DUPO 算法通过复制有效样本将智能体 RL 的采样效率提升 2-3 倍，解决了长期困扰该领域的效率问题
- 该工作证明了"训练模式"比"模型规模"更重要：7B 模型在正确的训练范式下可以超越 32B 模型
- WebSailor 属于阿里通义 WebAgent 系列的第三代（WebWalker -> WebDancer -> WebSailor），标志着开源信息搜寻智能体达到了接近闭源系统的水平
- 后续工作 WebSailor-V2（arXiv:2509.13305）进一步通过合成数据和可扩展 RL 提升了性能
