---
title: "A Brief Overview: On-Policy Self-Distillation In Large Language Models"
authors:
  - Fangming Cui
  - Sunan Li
  - Jiahong Li
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.18141
tags:
  - OPSD
  - on-policy-self-distillation
  - overview
  - post-training
  - LLM-training
  - memory-efficiency
status: done
---

# OPSD 简明综述：大模型中的在线策略自蒸馏

## 摘要

本文为 **OPSD（On-Policy Self-Distillation）** 提供面向新研究者的简明综述。OPSD 是一个统一学习框架：**单个 LLM 同时充当老师和学生**。老师角色拥有特权信息（真实推理过程），学生角色仅看问题；训练目标是在学生采样的轨迹上**最小化每 token 分布散度**。OPSD 相比标准 OPD **节省 40-60% GPU 内存**，因为无需维护外部老师模型。本文从设计原则、方法学创新、最新进展三个维度梳理 OPSD 的核心思想。

## 动机：为什么需要 OPSD 综述？

[[opsd]] 原始论文（Self-Distilled Reasoner, 2601.18734）提出后，OPSD 迅速成为学术与工业界研究热点：

- **Qwen3** 部分变体采用 OPSD
- **MiMo** 训练流程包含 OPSD 阶段
- **GLM-5** 的后训练管线提到 OPSD 思想

但对**新手研究者**而言，OPSD 的核心思想散布在不同论文里。本综述以"入门视角"梳理，让初学者快速建立全局认知。

## OPSD 核心原理

### 一句话定义

> **同一个 LLM，根据是否能看到真实答案分为"老师"和"学生"两个角色，让学生在自己采样的轨迹上学习模仿老师的分布。**

### 三大支柱

1. **单模型双角色**：老师与学生是同一参数集，区别仅在输入上下文。
2. **特权信息不对称**：老师条件化真实答案 $y^*$，学生不知道。
3. **on-policy 训练**：散度计算在学生自己生成的轨迹上。

### 数学描述

$$\mathcal{L}_{\text{OPSD}}(\theta) = \mathbb{E}_{(x, y^*) \sim \mathcal{D}} \left[ \mathbb{E}_{\hat{y} \sim p_\theta(\cdot|x)} \left[ \sum_t D(p_\theta(\cdot | \hat{y}_{<t}, x, y^*) \| p_\theta(\cdot | \hat{y}_{<t}, x)) \right] \right]$$

关键点：

- $p_\theta(\cdot | \cdot, x, y^*)$：老师，看到 $y^*$
- $p_\theta(\cdot | \cdot, x)$：学生，不看 $y^*$
- $\hat{y}$：学生采样的轨迹
- $D$：散度（通常 forward KL）

## 关键设计选择

### 1. 特权信息形式

不同 OPSD 变体提供不同特权信息：

| 特权类型 | 例子 | 适用任务 |
|---------|------|---------|
| 真实答案 | 数学题最终数字 | 短答案问题 |
| 推理过程 | Chain-of-thought | 推理任务 |
| 中间步骤 | 部分代码片段 | 编码任务 |
| 提示词 | "Hint: 考虑 X" | 引导探索 |

### 2. 散度选择

```
Forward KL D(p_T || p_S):   "覆盖老师所有可能"
Reverse KL D(p_S || p_T):   "聚焦老师高概率"
Jensen-Shannon (JS):        对称版本
α-divergence:               可调参数
```

实验普遍发现 **forward KL** 在推理任务上最优。

### 3. 内存优化

标准 OPD 内存占用：

```
GPU Memory = M_student + M_teacher + M_KV_cache × 2
```

OPSD 内存占用：

```
GPU Memory = M_student + M_KV_cache × 1.2 (略多)
```

**节省 40-60%** —— 这是为什么 OPSD 在中等算力实验室特别受欢迎。

### 4. 长度控制

[[opsd]] 发现 **1024 tokens** 比 4096 tokens 更优：

```
1024 tokens: 早期 token 信号集中，learning curve 陡
4096 tokens: 信号稀释，长序列错误累积
```

实践建议：从 1024 开始训练，根据任务调整。

## 与标准 OPD 的对比

| 维度 | 标准 OPD | OPSD |
|------|---------|------|
| 外部老师 | 必需 | ❌ 无需 |
| 内存占用 | 高 | 低 (-40~60%) |
| 训练成本 | 高（两套模型） | 中 |
| 散度信号源 | 老师 logits | 自身在特权上下文下的 logits |
| 适用场景 | 跨模型知识迁移 | 自我提升 |

### 何时用 OPSD？

✅ **推荐场景**：
- 没有更强老师可用（前沿模型自我提升）
- 算力受限的学术团队
- 任务有清晰"正确答案"（数学、代码）

❌ **不推荐场景**：
- 想从更强模型蒸馏（用标准 OPD）
- 开放问题（无明确答案）
- 创意写作（无"特权信息"概念）

## 工业界采用情况

综述梳理了主要工业模型对 OPSD 的采用：

```
Qwen3-Math:   部分 OPSD 训练阶段
MiMo:         OPSD + RLHF 混合
GLM-5-Math:   OPSD 在数学方向
DeepSeek:     OPSD-like 自蒸馏作为辅助
```

OPSD 已成为推理模型后训练的标配工具之一。

## 开放问题与未来方向

### 1. 自动课程学习

如何动态选择哪些题目用 OPSD 训练？
- 题目太简单：学生本来就会，浪费训练
- 题目太难：学生即使看答案也不能 rationalize

可能方向：根据**学生当前在该题上的成功率**调节训练。

### 2. 多模态 OPSD

视觉问答中，特权信息可以是：
- 图像区域标注（bounding box）
- 中间视觉特征
- 真实答案

但散度计算需要适应多模态分布。

### 3. OPSD + RL 混合

```
Phase 1: OPSD 训练基础推理能力
Phase 2: GRPO/PPO 训练偏好对齐
Phase 3: OPSD 精修长尾错误
```

最佳切换时机尚未理论化。

### 4. 推理时的 OPSD

能否在推理时也用"假设答案 + 反向验证"思路？这与 [[t2po]] 的不确定性信号、[[hdri]] 的假设驱动思想呼应。

## 贡献

1. **入门友好**：为新研究者提供 OPSD 全景图。
2. **设计选择对比**：四大设计维度的优劣分析。
3. **工业采用追踪**：明确 OPSD 已是主流。
4. **开放问题清单**：4 个值得探索的方向。

## 局限

- 综述较短（精简版），缺乏完整文献覆盖。
- 未深入数学理论（如收敛性证明）。
- 工业模型的 OPSD 实现细节多为开放秘密。

## 与相关工作的关系

### 上下游关系
- **上游**：[[opsd]]（原始论文）、[[opd_survey]]（理论框架）
- **下游**：[[rethinking_opd]]（OPD 机制分析）、[[lightning_opd]]（离线高效版本）

### 互补
- 本文从**新手视角**入门，互补于 [[opd_survey]] 的**专家视角**。
- 提供 OPSD 与 OPD 的对比表格，帮助选型决策。

### 适用读者
- 刚入门 LLM 后训练的研究者
- 想用 OPSD 但算力有限的团队
- 工业实践者快速了解 OPSD 全貌
