---
title: "Rethinking the Design of Reinforcement Learning-Based Deep Research Agents"
authors:
  - Yi Wan
  - Jiuqi Wang
  - Liam Li
  - Jinsong Liu
  - Ruihao Zhu
  - Zheqing Zhu
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2510.15862
tags:
  - deep-research
  - reinforcement-learning
  - RLOO
  - GRPO
  - LLM-judge
  - design-space-analysis
status: done
---

# Rethinking RL-Based Deep Research：重新审视 RL 驱动的深度研究代理设计

## 摘要

本文是 **RL-based Deep Research** 领域的**系统性设计空间分析**。尽管 [[deep_researcher]]、[[search_r1]]、[[r1_searcher]] 等 RL 训练的 deep research 系统取得了实证成功，**关键设计选择的影响从未被系统量化**。研究者将深度研究形式化为 **episodic finite MDP**，构建竞争性基线，然后系统消融四大设计因素：**奖励信号、训练算法、数据筛选、推理策略**。结论：**四项改进合起来让 7B 级 agent 在 10 个广泛基准上达到 SOTA**。这是从 "RL 凭直觉" 到 "RL 凭原则" 的转折点。

## 动机与问题

### RL-based Deep Research 的设计混乱

过去 2 年，RL 训练 deep research agent 的方法快速涌现：

```
DeepResearcher:     GRPO + 结果奖励 + 真实 web
Search-R1:          PPO + 格式 + 结果奖励
R1-Searcher:        类 R1 的 RL，规则奖励
ZeroSearch:         零成本环境 + GRPO
WebDancer:          GRPO + 课程学习
```

每个工作选择不同的 RL 算法、奖励、数据 —— 但**没人系统比较过这些选择**。

研究者发现：

> 同一基准上，A 论文报告 65%，B 论文报告 51%。**是 A 方法更好，还是 A 的设计选择更好？**

这一区分至关重要 —— 区别"方法贡献"与"工程贡献"。

### 系统分析的需求

学界需要回答：

1. **奖励**：规则奖励 vs LLM-judge 奖励，哪个更好？
2. **算法**：GRPO vs RLOO vs PPO，哪个更好？
3. **数据**：所有训练样本是否同等价值？
4. **推理**：贪心 vs 采样 vs 投票，哪个更好？

## 方法：4 项设计因素的系统分析

### MDP 形式化

将 deep research 形式化为 episodic finite MDP：

$$\mathcal{M} = (\mathcal{S}, \mathcal{A}, P, R, \gamma)$$

其中：
- $\mathcal{S}$：状态 = (问题, 对话历史, 已检索证据)
- $\mathcal{A}$：动作 = (search query, browse URL, read doc, answer)
- $P$：环境转移（真实 web 搜索 + LLM 内部状态）
- $R$：奖励
- $\gamma$：折扣因子

这一形式化让 RL 算法的应用清晰可比。

### 因素 1：奖励信号

**对比**：

| 奖励类型 | 描述 |
|---------|------|
| **规则奖励** | EM 匹配 / F1 / 启发式 |
| **LLM-judge 奖励** | 用更强 LLM 评估答案质量 |

**研究结果**：

```
规则奖励:    51.2% (10 基准平均)
LLM-judge:   58.7% (+7.5 pp)
```

**LLM-judge 显著优于规则**。原因：
- 规则奖励对表述敏感（"100 美元" vs "$100" 可能不一致）
- LLM-judge 容忍语义等价
- LLM-judge 能识别部分正确（partial credit）

### 因素 2：训练算法

**对比**：

| 算法 | 类型 |
|------|------|
| **GRPO** | off-policy（重要性采样） |
| **RLOO** | on-policy |

**研究结果**：

```
GRPO:   54.3%
RLOO:   60.1% (+5.8 pp)
```

**On-policy RLOO 显著优于 off-policy GRPO**。原因：
- GRPO 的重要性采样在长 trajectory 上方差大
- RLOO 简单稳定（一次采样组内对比）
- Deep research 任务的 trajectory 很长（~10 turns），off-policy 偏差累积

### 因素 3：数据筛选

**对比**：

| 策略 | 描述 |
|------|------|
| **All Data** | 用所有训练样本 |
| **Filtered** | 过滤"过易"和"过难"样本 |

**过易**：基线模型已经会做 → 训练信号低
**过难**：基线模型从不能做 → 训练信号噪声大

**研究结果**：

```
All Data:    56.4%
Filtered:    62.3% (+5.9 pp)
```

**筛选样本带来 +5.9 pp**。原则：

```python
def keep_sample(success_rate):
  return 0.2 <= success_rate <= 0.8
  # 太低 → 过难，模型学不到
  # 太高 → 过易，浪费训练
```

### 因素 4：推理策略

**对比**：

| 策略 | 描述 |
|------|------|
| **Greedy** | 贪心解码 |
| **Sampling** | 随机采样 |
| **Error-Tolerant Rollout** | 容错采样（错误时回滚重试） |

**研究结果**：

```
Greedy:                   58.0%
Sampling (T=0.7):         61.2%
Error-Tolerant Rollout:   64.5% (+6.5 vs Greedy)
```

**容错策略大幅提升**。机制：

```
推理过程:
  Step 1: 搜索 X → 失败
  Step 2: 回滚到 Step 1 重新生成 → 新搜索 Y → 成功
```

不像传统 LLM "一旦生成就不回退"，容错策略允许在长 trajectory 中纠错。

### 综合效果

四项改进**累积**：

```
Baseline (GRPO + rule reward + all data + greedy):  46.8%
+ LLM-judge reward:                                 54.3% (+7.5)
+ RLOO algorithm:                                   60.1% (+5.8)
+ Filtered data:                                    66.0% (+5.9)
+ Error-tolerant rollout:                           72.5% (+6.5)
                                                    ↑↑↑
                                                    +25.7 pp 总提升
```

## 实验结果

### 10 个基准的 SOTA

研究者在 10 个广泛使用的 deep research 基准上评测：

```
HotpotQA, 2WikiMultihop, MuSiQue, TriviaQA, NaturalQuestions,
StrategyQA, BrowseComp, BrowseComp-Plus, HLE-Eval, GAIA
```

**7B 模型上，4 项改进后达到所有基准的 SOTA**（同规模下）。

### 与商业系统对比

```
Rethinking-DR (7B):       72.5%
GPT-4o + ReAct:           58.4%
Claude 3.5 + Self-Refine: 65.2%
Gemini Pro DR:            70.8%
```

**7B 开源模型超过部分闭源前沿系统**。

## 为什么重要

### 1. 从直觉到原则

过去 RL deep research 的设计选择多靠直觉（"GRPO 好像在 R1 里有效，那就用它"）。本文用严格消融**证明每个选择的价值**。

### 2. 颠覆性发现

```
✗ 直觉：GRPO 是 DeepSeek 用的，应该好
✓ 实际：GRPO 在 deep research 长 trajectory 上不如 RLOO

✗ 直觉：规则奖励客观，应该好
✓ 实际：LLM-judge 奖励虽然主观，但语义更准

✗ 直觉：更多数据总是好的
✓ 实际：筛选后的数据训练效率高 +5.9 pp
```

每个"反直觉"都有数据支撑。

### 3. 可复制的配方

任何研究者都可以按本文配方训练 deep research agent：

```
配方：
1. 用 LLM-judge 作为奖励信号（不要规则奖励）
2. 用 RLOO 算法（不要 GRPO）
3. 用难度过滤的数据（20% < success rate < 80%）
4. 用 error-tolerant rollout（不要 greedy）
```

### 4. 节省整个领域的算力

如果未来工作都按本文配方训练，**避免重复犯错**的成本极高。

## 贡献

1. **MDP 形式化**：deep research 的清晰形式化框架。
2. **4 项关键设计因素**：奖励、算法、数据、推理。
3. **累积 +25.7 pp 改进**：每项独立 5-8 pp。
4. **10 基准 SOTA**：7B 模型上的全面胜利。
5. **可复制配方**：让 RL deep research 训练标准化。

## 局限与未解决问题

- 仅在 7B 规模验证，更大模型的设计选择可能不同。
- LLM-judge 奖励依赖 judge LLM 质量（如 GPT-4o），不同 judge 可能给不同信号。
- 4 项因素的交互效应未完全分析（如 RLOO + LLM-judge 是否有协同？）。

## 与相关工作的关系

### 直接挑战
- **挑战 GRPO 主流地位**：在 deep research 长 trajectory 任务上 RLOO 更优。
- **挑战规则奖励的"客观性"**：语义对齐比表面匹配更重要。

### 直接前身
- [[deep_researcher]]：开创性 RL deep research，但用 GRPO + 规则奖励。
- [[search_r1]]：用 PPO 训练 search agent。
- [[odr_plus]]：开源 deep research 的 SOTA 基线。

### 互补关系
- 与 [[actfocus]] 互补：ActFocus 调整 token 权重，本文调整奖励/算法/数据。
- 与 [[opd_survey]] 互补：OPD 用于蒸馏，本文用于 RL —— 都是后训练优化。
- 与 [[t2po]] 互补：T²PO 改 token 级控制，本文改训练算法。

### 后续方向
- 推广到更大规模模型（70B+）
- 与多模态 deep research 结合
- 探索 RLOO 与 GRPO 的混合策略
- 自动化数据筛选（在线动态调整难度）
