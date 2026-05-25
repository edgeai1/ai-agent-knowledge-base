---
title: "Improving and Evaluating Open Deep Research Agents"
authors:
  - Doaa Allabadi
  - Kyle Bradbury
  - Jordan M. Malof
venue: Preprint
year: 2025
url: https://arxiv.org/abs/2508.10152
tags:
  - deep-research
  - open-deep-research
  - ODR
  - ODR-plus
  - BrowseComp-Small
  - benchmark
status: done
---

# ODR / ODR+：开源深度研究代理的改进与评测

## 摘要

本文是 **Open Deep Research (ODR)** —— LangChain 生态唯一开源 Deep Research Agent —— 的首个学术评估。研究者提出 **BrowseComp-Small** 子集（60 题），让学术团队可以在有限算力下评测 deep research 系统。基线评估发现：**ODR、Anthropic、Google 在 BrowseComp-Small 上初始全部 0% 准确率**。研究者随后通过三处战略改进得到 **ODR+**，将开源/闭源系统的 SOTA 提升到 **10%**。这是 ODR 系列演进的起点，也是后续 RL-based deep research 研究的基础。

## 动机与问题

### Open Deep Research 的位置

LangChain 在 2025 年发布 [Open Deep Research](https://github.com/langchain-ai/open_deep_research)：

```
Open Deep Research:  唯一的开源、生产就绪 Deep Research 系统
                     基于 LangGraph，配置化，跨多 LLM 提供商
```

但发布时**没有学术评测** —— 学界不知道它有多好，研究者不知道在哪改进。

### 评测的算力门槛

BrowseComp 原始版本（OpenAI 发布）有 **1266 道题**，每题需要 5-15 美元运行成本：

```
1266 题 × $10 平均成本 = $12,660 一次评测
```

学术团队不可能反复运行。研究者意识到：**需要一个小规模、保留挑战性的子集**让学界能做实验。

## 方法

### BrowseComp-Small (BC-Small)

研究者从 BrowseComp 1266 题中精选 **60 道题**作为评测子集：

- **选题原则**：保持难度分布、覆盖主题、答案可验证
- **总成本**：约 $600 一次评测（20× 降低）
- **统计意义**：60 题足以区分 5-10 pp 的性能差异

### ODR+ 三大改进

研究者通过消融实验找到三个关键改进点：

#### 改进 1：搜索策略优化

```
原 ODR:    每次搜索独立，无规划
ODR+:      预先生成搜索 plan，分阶段执行
```

具体：在开始搜索前，让 LLM 生成 5-10 个子查询，按依赖关系排序。

#### 改进 2：上下文管理

```
原 ODR:   累积所有检索结果到 context，导致长上下文退化
ODR+:     每步骤摘要 + 关键事实保留 + 旧上下文压缩
```

引入 hierarchical context，将 token 消耗降低 60%。

#### 改进 3：引用与验证

```
原 ODR:   生成答案时不强制引用
ODR+:     每个事实必须有 source URL，无引用的事实丢弃
```

这一步**显著减少幻觉**，让答案可审计。

## 实验结果

### 基线评估（震撼性结果）

| 系统 | BC-Small 准确率 |
|------|----------------|
| ODR (原版) | **0%** |
| Anthropic Claude DR | **0%** |
| Google Gemini DR | **0%** |

**三个主流 deep research 服务在 60 题挑战上完全失败**。这一结果令人警醒。

### ODR+ 改进

| 系统 | BC-Small | 相对增益 |
|------|---------|---------|
| ODR | 0% | -- |
| ODR + 改进 1 | 3.3% | +3.3 |
| ODR + 改进 1+2 | 6.7% | +3.4 |
| **ODR+ (全部 3 项)** | **10%** | **+10** |

**SOTA among 开源/闭源 deep research 系统**，但仍只有 10%，说明 BC-Small 极具挑战性。

### 消融分析

```
Search Plan:       +3.3 pp
Context Mgmt:      +3.4 pp
Citation Enforce:  +3.3 pp
```

三个改进**贡献相当**，没有单一"银弹"。

## 为什么重要

### 1. 揭示前沿系统的脆弱性

主流商业 deep research 服务在 60 题上 0%，**实际研究质量远低于宣传**。

### 2. 提供学术研究基础

BC-Small 让学术团队能：
- 快速迭代实验
- 复现论文结果
- 公平比较新方法

### 3. 开启 ODR 后续研究链

ODR+ 成为后续工作的基础。例如：
- [[rethinking_rl_dr]]：在 ODR 基础上系统分析 RL 训练。
- [[intent_rl]]（2602.03468）：在 ODR 基础上加入用户意图建模。

### 4. 开源精神

完整代码 + BC-Small 数据集 + 评测脚本全部公开。

## 贡献

1. **BC-Small 基准**：算力友好的 BrowseComp 子集。
2. **ODR+ 系统**：开源 deep research 的 SOTA。
3. **三大改进点**：搜索 plan + 上下文管理 + 引用强制。
4. **消融分析**：明确每个改进的贡献。
5. **开源**：让后续研究基于 ODR+ 而非从零造轮子。

## 局限与未解决问题

- 10% 准确率仍非常低，说明系统还有大量改进空间。
- BC-Small 仅 60 题，可能不能代表所有任务分布。
- 主要为英文任务，多语言性能未评估。
- 改进 1-3 都是 inference-time 的，未涉及训练。

## 与相关工作的关系

### 直接前身
- **Open Deep Research (LangChain)**：原始开源系统。
- **BrowseComp (OpenAI)**：完整版基准。

### 直接后继
- [[rethinking_rl_dr]]（2510.15862）：ODR + RL 训练分析。
- [[intent_rl]]：ODR + 用户意图建模。
- [[deepresearcher]]、[[search_r1]]：RL 训练的 deep research，与 ODR 共同发展。

### 互补关系
- 与 [[browsecomp_plus]] 互补：BC-Small 解决算力问题，BC-Plus 解决检索-LLM 解耦问题。
- 与 [[deepweb_bench]] 互补：BC-Small 是简化版（60 题），DeepWeb-Bench 是新一代难度（多维度评估）。

### 后续方向
- ODR + RL 端到端训练（[[rethinking_rl_dr]] 已部分实现）。
- ODR 的多语言扩展。
- ODR 与 multi-modal source（图像、视频）的整合。
