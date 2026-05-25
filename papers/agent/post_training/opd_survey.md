---
title: "A Survey of On-Policy Distillation for Large Language Models"
authors:
  - Mingyang Song
  - Mao Zheng
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2604.00626
tags:
  - OPD
  - survey
  - on-policy-distillation
  - f-divergence
  - post-training
  - LLM-training-theory
status: done
---

# OPD 综述：大模型在线策略蒸馏的统一理论框架

## 摘要

本综述是首个对 **On-Policy Distillation (OPD)** 进行系统性梳理的工作。论文将 OPD 形式化为**在学生采样轨迹上的 f-散度最小化**，统一了 KD（知识蒸馏）、RLHF、模仿学习等多条原本独立的研究线。沿三维分类法 —— **优化目标 / 反馈来源 / 训练稳定化** —— 整理近百篇文献，揭示 OPD 与 KL-约束 RL 的深层联系，明确成功条件、失败模式与开放问题。

## 动机与问题

### 三条研究线的隔阂

直到本综述前，三个本质相关的研究领域各自为政：

1. **知识蒸馏 (KD)**：将大模型知识迁移到小模型，主流是 off-policy（学生模仿老师生成的固定数据）。
2. **RLHF**：用人类偏好作为奖励训练 LLM，on-policy 但反馈极稀疏。
3. **模仿学习**：机器人/控制领域的概念，DAgger、AggreVaTe 等方法。

OPD 实际上**跨越了三个领域**，但研究者各自用不同术语描述同一现象，导致：
- 重复发明轮子（每个领域独立发现"on-policy 比 off-policy 好"）
- 缺乏统一理论（无法预测新方法的成败）
- 工程实践分散（每团队自己的 OPD 实现）

### Exposure Bias：核心痛点

```
Off-Policy 蒸馏的根本问题：
  训练时：学生看老师的轨迹    p_T(y_{<t})
  推理时：学生看自己的轨迹    p_S(y_{<t})
  
  分布失配 → 错误指数级累积
  错误 ∝ horizon²（序列长度的平方）
```

OPD 通过**让学生在自己的状态上学习**消除这一失配。

## 方法：统一的 f-散度框架

### 形式化

OPD 的通用目标：

$$\mathcal{L}_{\text{OPD}}(\theta) = \mathbb{E}_{x \sim \mathcal{D}} \left[ \mathbb{E}_{\hat{y} \sim p_S^\theta(\cdot | x)} \left[ D_f(p_T \| p_S^\theta)(\hat{y}|x) \right] \right]$$

其中：
- $p_T$：教师分布（可以是另一个 LLM、自身带特权信息、奖励模型等）
- $p_S$：学生分布
- $\hat{y}$：从学生采样的轨迹（on-policy）
- $D_f$：任意 f-散度（KL、reverse KL、JS、$\alpha$-divergence 等）

这一公式统一了所有 OPD 变体。

### 三维分类法

```
                        OPD 方法
                            |
        +-------------------+-------------------+
        |                   |                   |
   What to optimize?   Where signal?    How to stabilize?
   (优化目标)         (反馈来源)         (训练稳定化)
        |                   |                   |
   Forward KL         Token logits        Cold start
   Reverse KL         Outcome reward      Clipping
   JSD                Self (privileged)   Mixing offline
   α-divergence       Reward model        Temperature
   Other f-div        Human/preferences   Curriculum
```

### 第一维：优化目标（What）

| 散度 | 行为 | 适用场景 |
|------|------|---------|
| **Forward KL** $D(p_T \| p_S)$ | mass-covering，学生覆盖老师 | 推理任务 |
| **Reverse KL** $D(p_S \| p_T)$ | mode-seeking，学生聚焦 | 对话/风格 |
| **JSD** | 对称，平滑 | 训练稳定 |
| **α-divergence** | 可调 | 探索-利用平衡 |

[[opsd]] 实验验证 forward KL 在推理任务上最优。

### 第二维：反馈来源（Where）

```
1. Logit-based     teacher 给出每 token 概率（dense）
2. Outcome-based   仅最终 reward（sparse, like GRPO）
3. Self-play       老师就是学生自己（如 OPSD）
4. Reward Model    用 reward model 评分
5. Preference      pair-wise comparison
```

不同来源密度差异巨大，影响训练效率几个数量级。

### 第三维：训练稳定化（How）

- **Cold Start**：先 SFT 拉近分布（[[rethinking_opd]] 配方）。
- **Per-Token Clipping**：防止 stylistic token 主导梯度（[[opsd]] 关键技巧）。
- **Mixing Offline Data**：保持多样性。
- **Temperature Annealing**：从高温（探索）到低温（利用）。
- **Curriculum Learning**：从简单题目逐步增加难度。

## 关键理论联系

### OPD 与 KL-约束 RL

综述揭示了一个深刻联系：

$$\mathcal{L}_{\text{OPD}} = \mathcal{L}_{\text{KL-RL}} + \text{terms}$$

具体来说，**OPD 等价于带 KL 约束的 RL**，但：
- KL-RL 用奖励信号 r(x, y) 引导策略
- OPD 用老师分布 p_T 直接做软目标

这一联系解释了为什么 OPD 能达到与 RL 相当甚至更好的效果 —— 它本质是用稠密的"分布奖励"替代稀疏的"标量奖励"。

### OPD 与 Imitation Learning

经典模仿学习 (IL)：

$$\mathcal{L}_{\text{IL}} = \mathbb{E}_{(x, y) \sim \pi_{\text{expert}}} \left[ -\log p_S(y | x) \right]$$

OPD 是 IL 的 on-policy 推广：

$$\mathcal{L}_{\text{OPD}} = \mathbb{E}_{x \sim \mathcal{D}, \hat{y} \sim p_S} \left[ \text{soft-target}(p_T, \hat{y}) \right]$$

关键区别：
- IL：学生看专家（off-policy）
- OPD：学生看自己（on-policy）+ 老师给软目标

## 失败模式分类

综述系统总结 OPD 的失败模式：

### 1. 老师-学生分布不兼容
现象：overlap ratio 持续低
原因：跨家族模型、训练数据差异大
解药：cold start（[[rethinking_opd]]）

### 2. 老师无可转移知识
现象：训练正常但最终性能与基线相当
原因：老师只是规模大、能力相似
解药：用 OOD 训练的老师

### 3. Stylistic Token 主导梯度
现象：训练 loss 下降但任务性能不变
原因：标点、连接词等高频低信息 token 散度大
解药：per-token clipping（[[opsd]]）

### 4. 探索不足
现象：模型收敛到次优解
原因：on-policy 采样多样性低
解药：温度调度、mixing offline data

### 5. 长上下文性能崩塌
现象：长响应时模型乱掉
原因：早期 token 决定后续，错误累积
解药：截断长度（[[rethinking_opd]] 发现 3K-7K 最优）

## 开放问题

综述列出 10 个待解问题：

1. **最优 f-散度选择**：当前主要靠经验，是否有理论指导？
2. **课程学习**：如何动态调整任务难度？
3. **多模态 OPD**：视觉、音频信号如何融入？
4. **多老师 OPD**：多个老师如何组合？
5. **在线 OPD**：边推理边训练的可行性？
6. **OPD 与 RL 混合**：何时切换？
7. **跨语言 OPD**：低资源语言的迁移？
8. **理论收敛保证**：什么条件下收敛到最优？
9. **隐私 OPD**：联邦学习场景下的设计？
10. **可解释 OPD**：哪些 token 的学习最重要？

## 贡献

1. **首个统一框架**：将 KD、RLHF、IL 在 f-散度视角下统一。
2. **三维分类法**：让研究者快速定位自己工作在版图中的位置。
3. **失败模式分类**：5 类失败模式 + 对应解药。
4. **理论联系**：OPD ↔ KL-RL ↔ IL 的桥梁。
5. **开放问题清单**：明确未来方向。

## 与相关工作的关系

### 综述自身的核心地位
本综述将 [[opsd]]、[[rethinking_opd]]、[[lightning_opd]] 等独立工作放入统一图景：

```
            f-散度最小化 OPD 框架
                    |
        +-----------+-----------+
        |           |           |
   OPSD        Rethinking    Lightning
   (自蒸馏)    (机制分析)    (离线变体)
```

### 后续方向
- 综述呼吁的开放问题正在被逐步解决（如 [[opsd_brief]] 探讨 OPSD 的内存效率）。
- 多模态 OPD 是 2026 下半年的研究热点。
