---
title: "Lightning OPD: Efficient Post-Training for Large Reasoning Models with Offline On-Policy Distillation"
authors:
  - Yecheng Wu
  - Song Han
  - Hai Cai
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2604.13010
tags:
  - OPD
  - offline-OPD
  - efficient-training
  - post-training
  - reasoning-models
  - infrastructure
status: done
---

# Lightning OPD：用离线在线策略蒸馏高效后训练大推理模型

## 摘要

Lightning OPD 解决 OPD 的**工程级限制**：标准 OPD 需要在训练时实时运行老师模型（teacher server），导致 GPU 资源需求翻倍。本工作发现**朴素的离线方法会失败**（"老师一致性"问题），并提出新的离线 OPD 方法：**在 SFT 与蒸馏阶段强制使用同一老师**。结果：训练效率提升 **4.0×**，Qwen3-8B 仅用 **30 GPU-小时**在 AIME 2024 上达到 69.9%，单台 8×H100 节点上 Qwen3-30B-A3B 达到 71.0%。

## 动机与问题

### 标准 OPD 的工程难题

在线 OPD（如 [[opsd]]、[[rethinking_opd]]）要求训练时**实时调用老师模型**：

```
每个 train step:
  1. 学生采样 rollout
  2. 老师对 rollout 评分      ← 老师必须在线
  3. 计算散度 + 反传
  4. 更新学生
```

工程后果：
- **资源翻倍**：老师 + 学生同时占用 GPU
- **基础设施复杂**：需 teacher server + 学生 server + 通信
- **延迟**：老师推理成为瓶颈
- **OoO 训练困难**：异步通信难调试

### 为什么不直接离线？

最自然的想法：**预先用老师跑一遍，缓存所有 logits**：

```
Offline phase: 老师在固定语料上跑，缓存 logits
Train phase:   学生从缓存读 logits，做蒸馏
```

但**朴素离线 OPD 失败** —— 这是 Lightning OPD 的核心发现。原因？**学生 rollout 与缓存的 rollout 不同**，缓存的 logits 对应的是"另一个学生"的状态。

## 方法：Lightning OPD

### 核心洞察：Teacher Consistency

研究者发现：

> 当 SFT 阶段用老师 A，蒸馏阶段用老师 B（即使都是 GPT-4），离线 OPD 失败。
> 当两阶段都用**完全相同的老师**（同模型 + 同权重 + 同采样温度），离线 OPD 成功。

**"老师一致性"是离线 OPD 的必要条件。**

### 训练流程

```
[Stage 1: SFT]
  数据:   {(x_i, y_i^T)} 其中 y_i^T = teacher.sample(x_i)
  目标:   学生学会模仿老师轨迹
  缓存:   保存老师在每个 (x_i, y_i^T) 上的完整 logits

[Stage 2: Offline OPD]
  数据:   缓存的 (x_i, y_i^T, logits_T)
  目标:   匹配学生 logits 与缓存的老师 logits
  
  ★ 关键：缓存的轨迹 y_i^T 来自 SFT 的同一老师 ★
        所以学生此时的分布与轨迹"几乎一致"
```

### 为什么 Teacher Consistency 关键

**bounded gradient discrepancy** 保证：

$$\| \nabla \mathcal{L}_{\text{offline}} - \nabla \mathcal{L}_{\text{online}} \| \leq C \cdot \| p_S - p_S^{\text{SFT}} \|$$

当学生经过 SFT 后，$p_S \approx p_S^{\text{SFT}}$，所以离线梯度 ≈ 在线梯度。但如果两阶段老师不同，$p_S^{\text{SFT}}$ 会远离最终 $p_S$，离线 OPD 偏离在线 OPD 的轨迹。

### 隐式正则化

Lightning OPD 还揭示了离线训练的额外好处：

```
On-line OPD:  学生不断采样新 rollout    → 高方差
Off-line OPD: 固定缓存的轨迹           → 低方差，隐式正则化
```

这一隐式正则化让 Lightning OPD 在小数据集上反而比在线 OPD 更稳定。

## 实验结果

### 训练效率

| 方法 | Qwen3-8B AIME 2024 达到 70% 时间 | GPU 节点 |
|------|---------------------------------|---------|
| 在线 OPD | ~120 GPU-小时 | 2 × 8×H100 |
| **Lightning OPD** | **30 GPU-小时** | **1 × 8×H100** |
| 节省 | **4.0×** | **2×** |

### 主基准

| 模型 | 方法 | AIME 2024 |
|------|------|-----------|
| Qwen3-8B-Base | SFT | 60.3% |
| Qwen3-8B-Base | 在线 OPD | 70.4% |
| Qwen3-8B-Base | **Lightning OPD** | **69.9%** |

性能**与在线 OPD 相当**（差距 < 0.5 pp），但训练快 4×。

### MoE 模型上的扩展

Qwen3-30B-A3B（30B 总参数，3B 激活，MoE 架构）：

```
Lightning OPD:  71.0% on AIME 2024
单台 8×H100 节点完成训练
```

证明 Lightning OPD 能扩展到 MoE 架构。

### 代码任务

不仅数学，代码生成（HumanEval、MBPP）上 Lightning OPD 也接近在线 OPD（差距 < 1 pp）。

## 为什么有效

### 1. 老师一致性 = 缓存有效性

只要 SFT 与离线 OPD 用同一老师，缓存的 logits 对学生而言"还是新鲜的"。

### 2. 隐式正则化 vs 学生漂移

在线 OPD 中学生不断更新，每次都看到新状态；离线 OPD 中学生看固定状态。这一约束反而避免了过拟合。

### 3. 节省的成本可以投入更长训练

```
在线 OPD: 2 × 8×H100 × 60 小时 = 16 GPU-小时/节点
Lightning OPD: 1 × 8×H100 × 30 小时 = 30 GPU-小时/节点 ÷ 4 = 7.5

省下的资源可训练更大模型 / 更多 step
```

### 4. 基础设施简化

无需 teacher server，无需 RPC 通信，无需异步管理。**学术团队可以在单卡上跑 OPD 实验**。

## 贡献

1. **可行性证明**：离线 OPD 是可能的，只要满足老师一致性条件。
2. **失败原因分析**：朴素离线方法失败的根本原因是 SFT 与蒸馏阶段老师不一致。
3. **训练效率突破**：4.0× 加速，单节点训练 8B 模型。
4. **MoE 兼容**：扩展到 30B MoE 架构。
5. **降低 OPD 门槛**：让学术团队无需大集群也能做 OPD 研究。

## 局限与未解决问题

- 老师一致性约束意味着不能跨阶段切换老师（如先 GPT-4 SFT，再 Claude 蒸馏）。
- 缓存的 logits 存储成本（vocab × seq_len × samples）不可忽略。
- 长上下文任务（>16K tokens）的离线缓存成本急剧上升。

## 与相关工作的关系

### 直接前身
- [[opsd]]、[[rethinking_opd]]：在线 OPD 的工作，是 Lightning OPD 的基础。
- [[opd_survey]]：将 Lightning OPD 归入"训练稳定化"维度的"mixing offline data"分支。

### 互补关系
- 与 [[opsd]] 互补：OPSD 节省外部老师，Lightning OPD 节省训练时老师在线运行。两者可叠加（OPSD + 离线 = 更节省）。
- 与 [[rethinking_opd]] 互补：Rethinking OPD 解决"老师选择"问题，Lightning OPD 解决"老师部署"问题。

### 后续方向
- 跨阶段老师演进（在保持一致性的前提下逐步升级老师）。
- 增量缓存（新数据加入时只算增量 logits）。
- 量化缓存（FP16/INT8 缓存进一步降低存储）。
