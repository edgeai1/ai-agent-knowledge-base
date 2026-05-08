---
title: "Tongyi DeepResearch Technical Report"
authors: "Tongyi DeepResearch Team (Alibaba)"
venue: "arXiv preprint"
year: 2025
url: "https://arxiv.org/abs/2510.24701"
code: "https://github.com/Alibaba-NLP/DeepResearch"
tags: [deep-research, agentic-training, mixture-of-experts, reinforcement-learning, information-seeking, open-source]
category: agent/deep_research
status: done
date_read: 2026-05-08
---

# 通义深度研究技术报告

## 摘要

通义深度研究是一个基于 Qwen3-30B-A3B（总参数 305 亿，通过混合专家每个词元激活 33 亿）构建的智能体 LLM，专门设计用于长程深度信息搜索研究任务。它通过结合智能体持续预训练（Agentic CPT）、监督微调（SFT）冷启动和强化学习（RL）的新型流水线进行端到端训练，所有环节均由全自动数据合成流水线驱动。通义深度研究在 Humanity's Last Exam 上达到 32.9，BrowseComp 上达到 43.4，GAIA 上达到 70.9，FRAMES 上达到 90.6，优于包括 OpenAI-o3 和 DeepSeek-V3.1 在内的强基线。模型、框架和完整的工具实现均完全开源。

## 动机与问题

深度研究任务——用户提出需要广泛网络搜索、文档分析和综合的复杂问题——给 LLM 带来了独特挑战：

1. **长程智能体行为**：深度研究需要在扩展轨迹（100 步以上）上执行数十个顺序的搜索、阅读和推理动作，远超标准 LLM 的训练范围。
2. **训练-推理差距**：标准 LLM 在静态文本上的预训练无法培养交互式研究任务所需的智能体技能（工具使用、搜索策略、信息综合）。
3. **智能体训练的数据稀缺**：通过人工标注收集高质量的智能体搜索行为训练数据成本高昂，自动合成则困难重重。
4. **工具集成复杂性**：有效的深度研究需要在连贯的推理框架内无缝集成多种工具（网络搜索、页面访问、代码执行、文件解析、学术搜索）。
5. **规模与效率**：大型专有模型（GPT-4、o3）实现了强大的研究性能但成本高昂；构建高效的开源替代方案需要审慎的架构和训练选择。

通义深度研究的方法：**端到端智能体训练**——从持续预训练到强化学习——配合全自动数据合成流水线，将智能体能力直接构建到模型中，而非纯粹依赖提示工程。

## 方法

### 架构概览

```
+-------------------------------------------------------------------+
|              Tongyi DeepResearch Architecture                     |
|                                                                   |
|  Base: Qwen3-30B-A3B (MoE: 30.5B total, 3.3B active/token)      |
|                                                                   |
|  +-----------------------------------------------------------+   |
|  |                  ReAct Reasoning Loop                      |   |
|  |                                                            |   |
|  |  [Thought] --> [Action] --> [Observation] --> [Thought]    |   |
|  |      |             |              ^                        |   |
|  |      |             v              |                        |   |
|  |      |    +------------------+    |                        |   |
|  |      |    | Tool Execution   |    |                        |   |
|  |      |    | - Search         |----+                        |   |
|  |      |    | - Visit          |                             |   |
|  |      |    | - Python         |                             |   |
|  |      |    | - Scholar        |                             |   |
|  |      |    | - File Parser    |                             |   |
|  |      |    +------------------+                             |   |
|  |      |                                                     |   |
|  |      v                                                     |   |
|  |  +-------------------+    +--------------------+           |   |
|  |  | Context Manager   |    | Meta-Thinker Agent |           |   |
|  |  | (dynamic context  |    | (monitors trajectory|          |   |
|  |  |  compression &    |    |  detects anomalies, |          |   |
|  |  |  refresh)         |    |  issues corrections)|          |   |
|  |  +-------------------+    +--------------------+           |   |
|  +-----------------------------------------------------------+   |
+-------------------------------------------------------------------+
```

### 训练流水线

训练遵循三阶段流水线，每个阶段均由自动数据合成驱动：

```
Stage 1: Agentic Continual Pre-Training (Agentic CPT)
-------------------------------------------------------
Purpose: Bridge pre-trained model to agentic behavior
Method:  Two-phase CPT on synthesized agentic trajectories

  Phase 1: General agentic capability
    - Train on diverse tool-use trajectories
    - Develop basic ReAct pattern comprehension
    - Preserve broad linguistic competence

  Phase 2: Domain-specific agentic skills
    - Train on research-specific trajectories
    - Develop search strategy and synthesis skills
    - Build document analysis capabilities

Stage 2: Supervised Fine-Tuning (SFT) Cold Start
--------------------------------------------------
Purpose: Provide initial policy for RL training
Method:  Fine-tune on high-quality agentic demonstrations

  - Curated set of research task demonstrations
  - Complete trajectories: query -> search -> read -> synthesize -> answer
  - Cold-start policy enables stable RL training

Stage 3: Reinforcement Learning (RL)
--------------------------------------
Purpose: Optimize research strategy through reward feedback
Method:  RL with reward model trained on research quality

  Reward signals:
  - Answer correctness (verifiable against ground truth)
  - Research thoroughness (coverage of relevant sources)
  - Reasoning quality (coherence and depth of synthesis)
  - Efficiency (fewer unnecessary actions preferred)
```

```
Algorithm: Tongyi DeepResearch Training Pipeline
--------------------------------------------------
Input:  Qwen3-30B-A3B-Base model M_0
Output: Trained agentic model M_final

// Stage 1: Agentic CPT
1:  D_cpt = SynthesizeAgenticTrajectories()
2:  M_1 = ContinualPreTrain(M_0, D_cpt)

// Stage 2: SFT Cold Start
3:  D_sft = CurateHighQualityDemonstrations()
4:  M_2 = SupervisedFineTune(M_1, D_sft)

// Stage 3: RL
5:  R = TrainRewardModel(quality_labels)
6:  M_final = RLTraining(M_2, R, environment)

return M_final
```

### 数据合成流水线

一个关键贡献是全自动数据合成流水线：

```
Data Synthesis for Each Training Stage:
-----------------------------------------

For Agentic CPT:
  1. Generate diverse research questions across domains
  2. Use teacher model to produce reference trajectories
  3. Simulate tool interactions in sandboxed environment
  4. Filter for quality: trajectory coherence, answer correctness
  5. Scale: hundreds of thousands of trajectories

For SFT:
  1. Select challenging research tasks from curated set
  2. Generate gold-standard trajectories with expert teacher model
  3. Verify answer correctness against ground truth
  4. Quality filtering: only top-quality demonstrations retained

For RL:
  1. Generate diverse task prompts
  2. Construct verifiable reward environments
  3. Enable model to interact with real/simulated tools
  4. Reward based on final answer correctness + process quality
```

核心设计原则：**无需昂贵的人工标注**。从数据合成到训练的整个流水线自动运行，实现可扩展的迭代。

### 动作空间

通义深度研究使用五种工具类型操作：

| 工具          | 描述                                        | 使用场景                        |
|---------------|----------------------------------------------------|---------------------------------|
| **Search**    | 通过搜索引擎 API 进行网络搜索                   | 查找相关页面/信息 |
| **Visit**     | 获取和解析网页内容                   | 阅读详细页面内容      |
| **Python**    | 在沙箱中执行 Python 代码                     | 数据处理、计算    |
| **Scholar**   | 学术论文搜索                              | 查找研究论文/引用  |
| **File Parser**| 解析上传的文档（PDF、DOCX 等）        | 分析用户提供的文档 |

### 多智能体内部架构

通义深度研究采用内部多智能体设计：

- **战术智能体**：以 ReAct 风格循环运行，处理逐步推理和工具调用。使用来自上下文管理器的简洁、动态刷新的上下文。
- **上下文管理器**：维护研究状态的压缩表示，防止长程研究轨迹中上下文窗口溢出。
- **元思考者智能体**：异步监控研究轨迹，检测异常（如重复失败、推理偏移或死胡同搜索路径），并发出战略干预以重新引导研究过程。

## 关键创新

1. **端到端智能体训练**：三阶段流水线（Agentic CPT -> SFT -> RL）将智能体能力直接构建到模型权重中，而非仅依赖提示工程。这是与仅使用提示的系统根本不同的方法。
2. **智能体持续预训练**：新颖的中间训练阶段弥合了通用预训练模型和智能体后训练之间的差距，为智能体行为提供了强大的归纳偏置，同时保留了通用语言能力。
3. **全自动数据合成**：完整的训练数据流水线无需人工标注即可运行，实现了训练数据质量和多样性的快速迭代和扩展。
4. **高效 MoE 架构**：仅使用 305 亿总参数中的 33 亿活跃参数，以远低于同等能力稠密模型的推理成本提供了强大的性能。
5. **用于轨迹校正的元思考者**：异步元监控智能体检测和纠正研究轨迹异常，是在长程智能体任务中维持质量的新型架构贡献。
6. **完全开源发布**：模型权重、训练框架、工具实现、提示配置和复现脚本全部发布，支持社区研究迭代。

## 实验设置

- **基础模型**：Qwen3-30B-A3B-Base（总参数 305 亿，每词元激活 33 亿）
- **基准测试**：
  - Humanity's Last Exam（HLE）：跨所有领域的专家级问题
  - BrowseComp / BrowseComp-ZH：网络浏览理解（英文/中文）
  - WebWalkerQA：网络导航问答
  - GAIA：通用 AI 助手基准测试
  - FRAMES：事实研究与多步推理
  - xbench-DeepSearch / xbench-DeepSearch-2510：深度搜索评估
- **基线**：OpenAI-o3、DeepSeek-V3.1、Claude、Gemini 等强模型
- **工具访问**：完全访问网络搜索、页面访问、代码执行、学术搜索

## 结果

### 基准测试性能

| 基准测试                | 通义深度研究 | OpenAI-o3 | DeepSeek-V3.1 |
|--------------------------|:-------------------:|:---------:|:-------------:|
| Humanity's Last Exam     | **32.9**            | 较低     | 较低         |
| BrowseComp               | **43.4**            | 较低     | 较低         |
| BrowseComp-ZH            | **46.7**            | --        | 较低         |
| WebWalkerQA              | **72.2**            | --        | 较低         |
| GAIA                     | **70.9**            | 较低     | 较低         |
| FRAMES                   | **90.6**            | 较低     | 较低         |
| xbench-DeepSearch        | **75.0**            | --        | --            |
| xbench-DeepSearch-2510   | **55.0**            | --        | --            |

通义深度研究在所有评估基准测试上优于包括 OpenAI-o3 和 DeepSeek-V3.1 在内的强基线，展示了端到端智能体训练的有效性。

### 效率分析

| 属性                 | 数值                          |
|--------------------------|--------------------------------|
| 总参数         | 305 亿                   |
| 每词元活跃参数  | 33 亿                    |
| MoE 效率           | 10.8% 激活率         |
| 等效稠密模型      | 约 7-14B 稠密模型计算量     |

MoE 架构以仅使用每词元一小部分计算量的方式，提供了与更大稠密模型竞争的性能。

### 训练流水线影响（消融实验）

| 配置                    | 相对性能 |
|----------------------------------|:--------------------:|
| 基础模型（无智能体训练） | 基线             |
| + 仅 Agentic CPT              | 显著提升     |
| + Agentic CPT + SFT             | 进一步提升  |
| + Agentic CPT + SFT + RL        | **最佳性能** |

每个训练阶段都有意义地贡献，完整的三阶段流水线比任何子集都取得了显著更好的结果。

## 局限性

1. **MoE 架构限制**：虽然高效，但 MoE 架构需要的总内存多于 33 亿稠密模型（尽管每词元计算量相似），限制了在资源受限硬件上的部署。
2. **工具依赖**：性能取决于外部工具（搜索 API、网络访问）的质量和可用性。工具故障直接影响研究质量。
3. **训练数据质量上限**：自动数据合成流水线虽然可扩展，但在最具挑战性的任务上，质量可能不及专家人工标注。
4. **长轨迹不稳定性**：尽管有元思考者，非常长的研究轨迹（100 步以上）仍可能受到累积错误和上下文退化的影响。
5. **基准特定优化风险**：强大的基准测试性能可能无法完全泛化到用户实际提出的多样化真实世界研究任务。
6. **中英文聚焦**：主要在英文和中文基准测试上评估；其他语言的性能未得到广泛验证。
7. **强化学习的可复现性**：强化学习结果对超参数和随机种子敏感，使得精确复现报告的数值具有挑战性。
8. **比较公平性**：基线模型（OpenAI-o3、DeepSeek-V3.1）可能未针对相同基准测试进行优化，使得直接比较需要细致考量。

## 核心要点

1. **智能体训练具有变革性**：通过 CPT -> SFT -> RL 流水线将智能体能力直接构建到模型权重中，产生的研究智能体从根本上强于仅使用提示工程。这一三阶段方案很可能成为智能体模型开发的标准。
2. **Agentic CPT 弥合了训练差距**：在微调前让模型接触智能体轨迹的新型中间训练阶段是关键架构贡献——它提供了使后续 SFT 和 RL 训练有效的归纳偏置。
3. **自动数据合成实现规模化**：从训练流水线中移除人工标注允许快速迭代和扩展，是实际智能体模型开发的关键推动力。
4. **MoE 实现高效智能体模型**：305 亿/33 亿的 MoE 架构证明了强大的智能体能力不需要稠密大模型，为研究智能体的高性价比部署打开了大门。
5. **元监控改善长轨迹**：检测和纠正轨迹异常的元思考者智能体是任何执行长程智能体任务的系统的重要架构模式。
6. **开源领导力**：通过完全开源模型、框架和工具，通义深度研究支持了社区研究，并建立了与专有系统竞争的强大开源基线。
7. **端到端与模块化设计**：通义深度研究的端到端训练方法与组合独立训练组件的模块化系统（MindSearch、WebThinker）形成对比，表明对于深度智能体任务，集成训练可能更优。
