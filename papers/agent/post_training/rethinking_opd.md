---
title: "Rethinking On-Policy Distillation of Large Language Models: Phenomenology, Mechanism, and Recipe"
authors:
  - Yaxuan Li
  - Yuxin Zuo
  - Bingxiang He
  - Jinqian Zhang
  - Chaojun Xiao
  - Cheng Qian
  - Tianyu Yu
  - Huan-ang Gao
  - Wenkai Yang
  - Zhiyuan Liu
  - Ning Ding
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2604.13016
tags:
  - OPD
  - on-policy-distillation
  - post-training
  - LLM-reasoning
  - phenomenology
  - failure-mode-analysis
status: done
---

# Rethinking OPD：重新审视大模型的在线策略蒸馏 —— 现象、机制与配方

## 摘要

本文是 OPD（On-Policy Distillation）研究的**里程碑式工作**：尽管 Qwen3、MiMo、GLM-5 等业界顶尖模型都用 OPD，但**没人系统理解过它何时成功、何时失败**。本文揭示了一个反直觉的现象：**更强的老师可能完全无法提升学生，而更弱的老师反而成功**。论文给出两个成功条件 —— 思考模式兼容 + 老师有真正的新能力 —— 并提出两个**修复失败 OPD 的实用配方**：off-policy cold start 与 teacher-aligned prompt selection。这彻底改变了"OPD 玄学调参"的现状。

## 动机与问题

### OPD 已成行业标配，但机制成谜

OPD（On-Policy Distillation）已被 Qwen3、MiMo、GLM-5 等最强 LLM 的后训练流程采用。但学术界没有系统分析过：

1. **什么时候 OPD 有效？什么时候失败？**
2. **为什么更强的老师反而无法提升学生？**
3. **如何在失败时挽救训练？**

行业实践中 OPD 训练失败被认为"玄学"，团队靠经验调参。

### 一个反常实验暴露问题

研究者做了一个揭示性实验：

```
设置 A：用 7B-post-RL 模型 → 蒸馏 → 1.5B-pre-RL 学生   → 完全失败
设置 B：用 1.5B-post-RL 模型 → 蒸馏 → 1.5B-pre-RL 学生 → 成功
```

**更强的老师（7B post-RL）完全失败，更弱的老师（1.5B post-RL）反而成功** —— 这彻底打破了"老师越强越好"的常识。这一现象需要新的理论解释。

## 方法：现象学 → 机制 → 配方

### 第一部分：现象学（What happens）

研究者发现成功 OPD 与失败 OPD 在**早期就能区分**：

#### 成功 OPD 的特征

```
Overlap Ratio:  72%  →  91%   稳步上升
Top-k 概率质量: 持续在 97-99%
Entropy Gap:    持续缩小
Vocabulary 集中度: 共享 token 集小但占据绝大部分质量
```

#### 失败 OPD 的特征

```
Overlap Ratio:  低 + 停滞   从一开始就低
Top-k 概率质量: 远低于 97%
分布始终不重叠
```

**核心现象学发现**：

> 成功 OPD 中，**很小的共享 token 集（约几十个）集中了 97-99% 的概率质量**。
> 训练通过"强化这个小集合 + 弱化其他 token"形成自我增强循环。

### 第二部分：机制（Why it happens）

#### 反向蒸馏实验（决定性证据）

```
1.5B post-RL  →  1.5B pre-RL    回滚到训练前
完全退化（OPD 训练让模型变差）
```

将其替换为**同家族更大模型**（7B）：

```
7B post-RL  →  1.5B pre-RL    替换老师
完全相同的退化模式
```

**这证明**：

> OPD 的训练动态可以**与老师的 benchmark 性能完全无关**！

不是模型变强，而是分布**距离**和**共享 token 集**决定了 OPD 是否能转移知识。

#### 两个成功条件

**条件 1：思考模式兼容**
学生与老师的推理风格、token 分布要兼容。同家族不同尺寸的模型（1.5B 和 7B）"在学生看来分布上不可区分"。

**条件 2：老师有真正的新能力**
仅靠规模缩放的同家族大模型，对小模型来说**没有可转移的新知识**。

```
✅ 老师有新能力（post-RL，OOD 训练）：可转移
❌ 老师只是更大但同训练（scale-up only）：无可转移
```

### 第三部分：配方（How to fix）

#### 配方 A：Off-Policy Cold Start

**问题**：学生与老师的初始思考模式不兼容，Overlap Ratio 一开始就低。

**解决**：先用老师生成的轨迹做 SFT（off-policy），把学生"拉"到老师附近的分布，再开始 OPD。

```
Step 1 (Cold Start):  Off-policy SFT on teacher rollouts   →   提升初始 overlap
Step 2 (OPD):         On-policy distillation                →   稳定上升
```

实验：cold start 让最终准确率提升 **2-4 个百分点**。

#### 配方 B：Teacher-Aligned Prompt Selection

**问题**：训练 prompt 与老师后训练的 prompt 分布不同，老师在这些 prompt 上表现差。

**解决**：使用与老师后训练数据相似的 prompt 来训练 OPD。

```
✅ Teacher-aligned prompts:  Top-k 概率质量更集中
⚠️ 副作用：可能降低熵（多样性），需混入 OOD prompt 平衡
```

### 关键数学：Top-k Overlap Token Mass

设老师概率分布 $p_T$、学生 $p_S$，定义 overlap token 集：

$$\mathcal{O}_k = \text{TopK}(p_T) \cap \text{TopK}(p_S)$$

成功 OPD 的标志：

$$\sum_{v \in \mathcal{O}_k} p_T(v) + \sum_{v \in \mathcal{O}_k} p_S(v) \geq 1.94 \quad (97\% \times 2)$$

即两个分布都把 97%+ 概率质量放在共享 token 上 —— 这是 OPD 能学习的**必要条件**。

## 实验结果

### 实验设置

- **模型**：DeepSeek (R1-Distill-1.5B/7B, JustRL-1.5B)、Qwen (1.7B-4B)
- **任务**：AIME 2024/2025、AMC 2023（数学推理）
- **数据集**：DAPO-Math-17k（17,000 题）
- **评估**：avg@16（16 次采样多数表决）

### 关键发现（带数字）

| 现象 | 量化 |
|------|------|
| 同家族老师 vs 跨家族 post-train 老师 | 同家族恢复 20-40%，跨家族 60-80% |
| Cold-start SFT 提升 | +2-4 pp 最终准确率 |
| 长响应训练 | 3K-7K tokens 最优；>10K 性能退化 |
| Sampled-token OPD (top-1) vs top-16 | 性能相近，top-1 argmax 不稳定 |

### Overlap Ratio 与最终性能的相关性

```
Overlap @ step 100  →  Final Accuracy
   < 60%              <60% accuracy
   60-75%              中等
   > 80%              SOTA
```

**Overlap Ratio @ early step 是预测 OPD 成败的最强指标。**

## 为什么这项工作重要

### 1. 从玄学到科学

OPD 训练成败不再靠"试一下"。两个明确的可检查指标（思考模式兼容、新能力存在），让团队在训练前就能预测成败。

### 2. 反直觉但正确

"老师越强越好" → "老师与学生的 token 分布要兼容 + 老师要有 OOD 能力"。这是范式级转变。

### 3. 直接可落地的修复方法

- Cold start 是几乎零成本的预处理。
- Teacher-aligned prompts 可以通过简单 BM25 匹配实现。

### 4. 揭示了 LLM 后训练的本质

不是"信息量传递"，而是"**token 分布在共享子空间内的对齐与放大**"。这与传统知识蒸馏的理解完全不同。

## 贡献

1. **现象学**：首次系统刻画 OPD 成败的可观察特征（overlap ratio、97-99% 概率质量、entropy gap）。
2. **机制**：用反向蒸馏实验证明 OPD 与 benchmark 性能解耦，揭示同家族陷阱。
3. **配方**：cold start + teacher-aligned prompts 修复失败训练。
4. **可预测性**：早期 overlap ratio 可以预测 OPD 最终成败。
5. **数学框架**：Top-k overlap token mass 作为成败的量化判据。

## 局限与未解决问题

- 主要在数学推理任务上验证，编码、对话等领域未充分测试。
- 30 页正文 + 23 图，实验密度高但跨模型族泛化性有限。
- 与 RL（如 GRPO）的混合训练未探讨。

## 与相关工作的关系

### 直接前身
- [[opsd]]（自蒸馏，无外部老师）：本工作是其"外部老师 + 学生"版本的机制分析。
- 原始 OPD（[Distilling Step-by-Step](https://arxiv.org/abs/2305.02301) 等）：本工作首次系统分析。

### 同期工作
- [[opd_survey]]（2604.00626）：理论分类框架。
- [[lightning_opd]]（2604.13010）：离线版本提升效率。
- [[opsd_brief]]（2605.18141）：综合概述。

### 后续方向
- 推广到多模态 OPD（视觉 + 语言）。
- 老师选择算法：基于 overlap ratio 预测最佳老师。
- 与课程学习结合：根据 overlap 动态调整训练任务难度。
