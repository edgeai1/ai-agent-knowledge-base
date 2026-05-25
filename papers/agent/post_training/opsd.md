---
title: "Self-Distilled Reasoner: On-Policy Self-Distillation for Large Language Models"
authors:
  - Siyan Zhao
  - Zhihui Xie
  - Mengchen Liu
  - Jing Huang
  - Guan Pang
  - Feiyu Chen
  - Aditya Grover
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2601.18734
tags:
  - OPSD
  - on-policy-self-distillation
  - post-training
  - LLM-reasoning
  - knowledge-distillation
  - math-reasoning
status: done
---

# OPSD：自蒸馏推理器 —— 大模型的在线策略自蒸馏

## 摘要

**OPSD (On-Policy Self-Distillation)** 是 LLM 后训练中的范式突破：**同一个 LLM 同时扮演老师和学生**。老师拥有"特权信息"（题目 + 真实推理过程），学生只看题目；训练目标是在**学生自己的 rollout 上**最小化两者的 token 级分布差异。这一巧妙设计同时解决了三大痛点：
1. **无需外部老师模型**（节省训练成本）；
2. **避免 off-policy 蒸馏的分布失配**（学生看到自己生成的状态）；
3. **稠密信号比 RL 的稀疏奖励训练效率高 10x**（每 token 都有梯度）。

在 AIME 2024/2025、HMMT 2025 上，OPSD **匹配或超过 GRPO**，且仅需 1/8 的 token 预算。

## 动机与问题

### 推理 LLM 后训练的两难

LLM 后训练（post-training）让基模型获得推理能力的主流方法：

| 方法 | 数据需求 | 反馈密度 | 分布失配 |
|------|---------|---------|---------|
| **SFT (off-policy)** | 专家轨迹 | 稠密但与学生分布不同 | 严重 |
| **RL/GRPO** | 仅需奖励 | 极稀疏（每条只 1 个标量） | 无 |
| **OPD (on-policy)** | 老师模型 | 稠密且在学生轨迹上 | 无 |
| **OPSD（本文）** | 真实答案 | 稠密 + 学生轨迹 + 无外部老师 | 无 |

### 关键洞察：合理化比生成简单

研究者从认知科学借来一个洞察：

> **生成一个正确答案很难；但如果已经知道答案，合理化（rationalize）这个答案要容易得多。**

如果让模型看到正确答案再"解释为什么对"，它可以产生比自己原本生成的更好的推理过程。这一**不对称的难度差**给了 OPSD 的存在基础。

### Exposure Bias 痛点

Off-policy 蒸馏（学生模仿老师的轨迹）有一个核心问题：

```
训练时学生看到的部分序列：来自老师的完美轨迹
推理时学生看到的部分序列：来自自己的（可能有错）轨迹
                          ↑↑↑
                  分布失配 → 错误累积 → 推理失败
```

**Exposure bias 随序列长度平方增长** —— 这是为什么"训练 loss 低，但推理时模型崩溃"。

## 方法：OPSD

### 两个角色

**老师策略**：

$$p_T(\cdot | x, y^*)$$

输入：问题 $x$ + 真实推理过程 $y^*$
输出：在已知答案后的"合理化"分布

**学生策略**：

$$p_S(\cdot | x)$$

输入：仅问题 $x$
输出：学生自己的推理分布

**关键点**：**两者是同一个 LLM**，区别仅在于输入上下文。

### 训练目标

最小化在学生自己采样的轨迹上的 token 级散度：

$$\mathcal{L}(\theta) = \mathbb{E}_{(x, y^*) \sim \mathcal{S}} \left[ \mathbb{E}_{\hat{y} \sim p_S(\cdot|x)} \left[ D(p_T \| p_S)(\hat{y} | x) \right] \right]$$

其中：
- $\hat{y}$：学生**自己生成**的轨迹（on-policy）
- $D$：散度（实验证明 **forward KL** 最优）
- 散度在每个位置的**全词表**上计算（稠密信号）

### 关键技巧：Per-Token Pointwise Clipping

如果不做处理，**风格性 token**（"the", ",", "Let's"）的散度会远大于**数学性 token**（"3.14", "≤", "lim"），训练信号被风格 token 主导。

```python
divergence_t = clip(per_token_kl[t], max=clip_threshold)
loss = mean(divergence_t)
```

这一步至关重要，否则训练崩塌。

## 实验结果

### 主实验：AIME / HMMT 数学竞赛

| 模型 | 方法 | AIME 2024 | AIME 2025 | HMMT 2025 |
|------|------|-----------|-----------|-----------|
| Qwen3-1.7B | SFT | 51.5 | 45.2 | 32.8 |
| Qwen3-1.7B | GRPO | 56.1 | 50.4 | 36.9 |
| Qwen3-1.7B | **OPSD** | **57.2** | **51.8** | **38.4** |
| Qwen3-8B | SFT | 73.6 | 65.4 | 49.2 |
| Qwen3-8B | GRPO | 76.4 | 68.9 | 53.5 |
| Qwen3-8B | **OPSD** | **77.8** | **70.8** | **55.1** |

OPSD 在所有模型规模上**匹配或超过 GRPO**，小模型上优势更明显。

### Token 效率（关键贡献）

```
GRPO 每步预算：8 个 rollout × 16K tokens = 128K tokens
OPSD 每步预算：1 个 rollout × 1K tokens =  1K tokens
                                          ↑↑↑
                                     128× token 效率
```

OPSD 在 100 步内的学习曲线比 GRPO 陡得多 —— 因为**每个 token 都给梯度**，而 GRPO 整条轨迹只给一个标量。

### GRPO 的"奖励多样性崩溃"

研究者揭示了 GRPO 的一个鲜为人知的故障模式：

> 当一个 batch 内所有样本都对（或都错）时，GRPO 的相对优势 = 0，梯度为 0。
> 实验显示**超过 50% 的 batch 出现零梯度**。

OPSD 没有这个问题：无论答案对错，**每个 token 的散度信号都有效**。

### 消融实验

| 消融维度 | 关键发现 |
|---------|---------|
| 散度类型 | Forward KL > Reverse KL > JS |
| 思考模式 | 学生不开思考模式 + 老师开思考模式 = 最优 |
| Clipping | 不做 clip 会训练崩塌 |
| 生成长度 | 1024 tokens > 4096 tokens（早期 token 信号更强） |
| 目标对比 | 全词表 logit (84.1%) > 采样 token 策略梯度 (82.1%) |

### vs. STaR

[STaR](https://arxiv.org/abs/2203.14465) 是 OPSD 的概念前身：
- STaR：用"rationalize 错误样本"产生数据，但奖励是序列级二值
- OPSD：每个 token 都有稠密散度信号

OPSD 在全错 batch 上**仍能学习**（每 token 都有信号），STaR 此时退化为 0 梯度。

## 为什么有效

### 1. 同模型 + 不同上下文 = 自然差异源

不需要外部老师 —— 同一个模型在两个不同上下文下产生不同分布，差异本身就是训练信号。这是 OPSD 最优雅的设计。

### 2. On-policy + 稠密信号 = 双优势

```
SFT:    on-policy?  ❌   稠密？ ✅
GRPO:   on-policy?  ✅   稠密？ ❌
OPSD:   on-policy?  ✅   稠密？ ✅   ← 全占
```

### 3. 真实答案 > 老师模型

OPSD 直接利用数据集中的真实答案作为"老师条件"，相比训练大模型当老师，**根本不需要任何外部老师模型**。

### 4. 节省 GPU 内存 40-60%

不需要同时跑老师 + 学生模型，单模型 + 两种上下文足够。

## 贡献

1. **新范式**：on-policy + 自蒸馏 + 答案条件化，首次三者结合。
2. **稠密 token 级信号**：训练效率比 RL 高 100×+。
3. **避免 GRPO 的奖励多样性崩溃**。
4. **节省 40-60% GPU 内存**：相比标准 OPD。
5. **简洁实现**：仅需 ~50 行 PyTorch 代码改造。

## 局限与未解决问题

- 实验仅到 8B 参数，更大模型行为未验证。
- 学生看不懂的题目（远超模型能力的）OPSD 无法解决。
- 对"无明确答案"的开放问题（如创意写作）不适用。
- 未探讨与推理模型（如 R1）的结合。

## 与相关工作的关系

### 范式前身
- **STaR**（2022）：rationalize 错误样本，但序列级奖励。
- **DPO**（2023）：成对偏好，无需 RL 但仍 off-policy。
- **GRPO**（DeepSeek R1）：on-policy RL，稀疏奖励。

### 直接后继
- [[rethinking_opd]]（2604.13016）：揭示 OPD/OPSD 何时失败、何时成功的相变条件。
- [[lightning_opd]]（2604.13010）：离线版本，进一步降低成本。
- [[opd_survey]]（2604.00626）：完整分类与理论框架。

### 互补关系
- 与 [[t2po]] 互补：T²PO 在推理时控制探索，OPSD 在训练时提供稠密信号。
- 与 RL 训练的 Deep Research（[[deep_researcher]]、[[search_r1]]）互补：可作为更高效的训练替代。

### 后续方向
- 课程学习：从简单题目开始，逐步增加难度。
- 多模态 OPSD：图像题、代码题。
- OPSD + RL 混合训练。
