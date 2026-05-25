---
title: "Resolving Action Bottleneck: Agentic Reinforcement Learning Informed by Token-Level Energy"
authors:
  - Langzhou He
  - Junyou Zhu
  - Yue Zhou
  - Zhengyao Gu
  - Junhua Liu
  - Wei-Chieh Huang
  - Henry Peng Zou
  - David Wipf
  - Philip S. Yu
  - Qitian Wu
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.14558
tags:
  - agent-RL
  - PPO
  - GRPO
  - credit-assignment
  - token-level-energy
  - action-bottleneck
status: done
---

# ActFocus：解决动作瓶颈 —— 由 Token 级能量引导的代理强化学习

## 摘要

本文揭示并解决了 agentic RL 中一个长期被忽视的问题：**动作瓶颈 (Action Bottleneck)**。在多轮代理 RL 训练（PPO、GRPO）中，**token 级的训练信号过度集中在动作 token 上，而推理 token 几乎没有梯度**，尽管推理 token 数量远多于动作 token。论文提出 **ActFocus** 方法：通过简单的梯度重新加权 —— **降低推理 token 权重，增加高不确定性动作 token 权重** —— 在四个环境中实现高达 **+65.2 pp（vs PPO）和 +63.7 pp（vs GRPO）** 的提升，且无任何额外计算开销。

## 动机与问题

### Agentic RL 的常见架构

```
[Trajectory τ = (reasoning_1, action_1, obs_1, reasoning_2, action_2, ...)]

reasoning_t: 模型的内部思考（长，~100 tokens）
action_t:    与环境交互的 token（短，~10 tokens）
obs_t:       环境观察
```

例子：

```
reasoning: "I need to first read the paper to understand X..."  (100 tokens)
action:    "<tool>read_paper</tool><arg>arxiv:2401.00001</arg>" (15 tokens)
```

### Action Bottleneck 现象

研究者通过梯度分析发现：

```
PPO/GRPO 标准训练:
  推理 token 数:  ~90% of trajectory
  推理 token 梯度: ~10% of total gradient
  
  动作 token 数:  ~10% of trajectory
  动作 token 梯度: ~90% of total gradient  ← 严重不平衡
```

**绝大部分训练信号都流向了少数动作 token**，推理 token 几乎不被训练。

### 为什么这是问题？

理论上 PPO/GRPO 应该均匀地为所有 token 提供梯度。但实际上：

```
PPO 目标: ∇log π(a_t | s_t) × Advantage_t
        
当 advantage 与 reward 高度相关时:
  Reward 主要由 action 决定（环境只看 action）
  Advantage 在 action token 上方差大
  Advantage 在 reasoning token 上方差小
  
→ 自然地，梯度集中在 action
```

**后果**：模型学会了"什么动作好"，但没学会"如何推理出好动作"。这导致：
- 推理变成"表演"（写一堆但无关）
- 泛化能力差
- 训练后期容易 reward hacking

## 方法：ActFocus

### 核心洞察

> 标准 PPO/GRPO 隐式假设所有 token 同等重要。但代理任务中，**推理 token 是"如何决策"，动作 token 是"决策本身"**。

ActFocus 显式区分两类 token，重新分配梯度权重。

### 公式

标准 PPO 损失：

$$\mathcal{L}_{\text{PPO}} = -\mathbb{E}_t [\min(r_t A_t, \text{clip}(r_t, 1-\epsilon, 1+\epsilon) A_t)]$$

其中 $r_t = \pi_\theta(a_t | s_t) / \pi_{\theta_{\text{old}}}(a_t | s_t)$。

ActFocus 加权：

$$\mathcal{L}_{\text{ActFocus}} = -\mathbb{E}_t [w_t \cdot \min(r_t A_t, \text{clip}(r_t, 1-\epsilon, 1+\epsilon) A_t)]$$

权重 $w_t$ 定义为：

$$w_t = \begin{cases}
\alpha \cdot u_t & \text{if } a_t \text{ is action token} \\
\beta & \text{if } a_t \text{ is reasoning token}
\end{cases}$$

其中：
- $\alpha > 1$：放大动作 token 权重
- $\beta < 1$：降低推理 token 权重
- $u_t$：token 级不确定性（如 entropy 或 max-probability gap）

### 关键创新：基于不确定性的动作加权

不是所有动作 token 都需要同样放大。**高不确定性的动作 token 才是学习的关键**：

```
低不确定性动作（如格式化 tag <tool>）: 模型已经会了，无需多训练
高不确定性动作（如选择哪个工具）:      模型不确定，需要更多训练信号
```

ActFocus 用 token 级 entropy 自动识别哪些动作 token 该重点训练。

### Token 级能量框架

将 token 级训练信号分析为"能量"流动：

$$E_t = |\nabla_\theta \mathcal{L}_t|^2$$

理想分布应满足：

$$E_t \propto \text{Information Gain}(t)$$

但实际中 $E_t$ 集中在动作 token，与信息增益不匹配。ActFocus 通过 $w_t$ 重塑能量分布。

## 实验结果

### 主基准（4 个环境）

| 环境 | PPO | GRPO | **ActFocus** | vs PPO | vs GRPO |
|------|-----|------|--------------|---------|---------|
| ALFWorld | 38.2 | 41.5 | **85.7** | +47.5 | +44.2 |
| WebShop | 21.4 | 24.8 | **86.6** | +65.2 | +61.8 |
| SciWorld | 32.1 | 34.6 | **78.9** | +46.8 | +44.3 |
| BabyAI | 45.3 | 48.7 | **89.4** | +44.1 | +40.7 |

**所有环境平均：+45-65 pp 提升，零额外开销。**

### 消融实验

| 设置 | ALFWorld | 含义 |
|------|---------|------|
| 基线（PPO） | 38.2 | -- |
| + 仅降低推理权重 | 56.4 | β = 0.5 |
| + 仅增加动作权重 | 62.1 | α = 2.0 |
| + 组合（均匀） | 70.3 | -- |
| **+ 组合 + 不确定性加权** | **85.7** | 完整 ActFocus |

不确定性加权（最后一行）贡献最大 —— 不是简单放大所有动作，而是聚焦于高不确定性的关键决策。

### 跨模型规模

```
Llama-3-8B + ActFocus:  86.2 → 88.4 (微调对小模型同样有效)
Llama-3-70B + ActFocus: 89.1 → 91.5
```

**所有规模都有提升**，证明 Action Bottleneck 是普遍现象。

## 为什么有效

### 1. 重塑梯度流向"应该学习的地方"

```
标准 PPO:           信号 → 动作（学会执行）
ActFocus:           信号 → 推理（学会思考）+ 关键动作（学会决策）
```

### 2. 推理 token 终于被训练

过去推理 token 被认为"自然学会"，实际上几乎不被训练。ActFocus 让推理 token 也获得有意义的梯度。

### 3. 不确定性引导 = 自适应学习率

高不确定性的动作 token 自动获得更高权重，相当于自动课程学习 —— **不需要预先知道哪些 token 难学**。

### 4. 零额外计算

ActFocus 只是改变损失函数中的权重，**不增加任何 forward/backward pass**。这是工程上的巨大优势。

## 贡献

1. **发现现象**：首次系统揭示 agentic RL 中的 action bottleneck。
2. **理论分析**：token 级能量框架，解释梯度集中的根因。
3. **简单方法**：ActFocus = 两个超参 + 不确定性加权。
4. **巨大提升**：四个环境 +44-65 pp，无额外成本。
5. **跨规模有效**：从 8B 到 70B 均有提升。

## 局限与未解决问题

- $\alpha, \beta$ 需要超参搜索，缺乏理论指导。
- 仅在文本环境验证，机器人控制等连续动作空间未测试。
- 与 [[t2po]] 的不确定性方法关系未深入对比。

## 与相关工作的关系

### 直接前身
- PPO（Schulman 2017）：基础算法。
- GRPO（DeepSeek R1）：相对优势 PPO。
- 二者都隐式假设 token 同等重要 —— ActFocus 显式打破这一假设。

### 思想关联
- 与 [[t2po]] 的"token 级不确定性引导探索"思想呼应：两者都用 token 不确定性引导训练/推理。
- 与 [[opsd]] 的"per-token dense signal"思想呼应：都强调 token 级训练信号的精确控制。

### 互补关系
- 与 [[deep_researcher]]、[[search_r1]] 互补：这些工作训练算法层面，ActFocus 提供 token 加权层面的改进。
- 与 [[rethinking_opd]] 互补：Rethinking OPD 解决"老师选择"，ActFocus 解决"动作-推理失衡"。

### 后续方向
- 动态 $\alpha, \beta$：根据训练阶段自动调整。
- 任务自适应：不同任务的最佳加权策略不同。
- 推广到连续动作空间。
