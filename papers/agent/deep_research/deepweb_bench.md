---
title: "DeepWeb-Bench: A Deep Research Benchmark Demanding Massive Cross-Source Evidence and Long-Horizon Derivation"
authors:
  - Sixiong Xie
  - Zhuofan Shi
  - Haiyang Shen
  - Jiuzheng Wang
  - Siqi Zhong
  - Mugeng Liu
  - Chongyang Pan
  - Peilun Jia
  - Baoqing Sun
  - Xiang Jing
  - Yun Ma
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.21482
tags:
  - deep-research
  - benchmark
  - evidence-collection
  - cross-source
  - long-horizon
  - calibration
status: done
---

# DeepWeb-Bench：需要大规模跨源证据和长程推导的深度研究基准

## 摘要

DeepWeb-Bench 是 2026 年 5 月发布的深度研究 (Deep Research) 评测基准，专门针对**当前前沿系统已经触及饱和的现有基准（BrowseComp、GAIA、HLE）**的局限性而设计。研究通过对九个前沿模型的系统评测发现：**真正难住模型的不是检索本身（仅占 12-14% 错误），而是跨源调和与长程推导（占 70% 以上错误）**。这一发现颠覆了"深度研究 = 更强的搜索引擎"的传统认知，将研究重心引向证据整合与推理校准。

## 动机与问题

### 现有基准的饱和

- **BrowseComp**（OpenAI 发布）：GPT-5.5 Pro 已达 90.1%，GPT-5.4 Pro 89.3%，Claude Mythos 86.9%（2026 年 5 月数据），区分度严重下降。
- **GAIA**：单跳和双跳问题为主，无法测试真正的多源整合能力。
- **HLE (Humanity's Last Exam)**：偏重知识问答而非证据导向研究。

### 真实深度研究的需求未被衡量

学者、记者、产业研究员面对的真实研究任务通常涉及：
1. **大规模证据采集**：单题需查阅 20+ 来源；
2. **跨源调和**：不同来源数据相互矛盾，需要权威性判断；
3. **长程推导**：从原始数据到最终答案需要 5+ 步推理；
4. **置信度校准**：知道自己什么时候不确定，是否需要继续搜索。

DeepWeb-Bench 直接针对这四个维度构造任务。

## 方法：四大能力家族

基准将深度研究能力解耦为四个正交维度：

```
+---------------+    +--------------+    +--------------+    +--------------+
|  Retrieval    |    |  Derivation  |    |  Reasoning   |    | Calibration  |
| (能找到吗？) |    | (能推出吗？)|    | (能推对吗？)|    | (敢说吗？)  |
+---------------+    +--------------+    +--------------+    +--------------+
        |                   |                   |                   |
        v                   v                   v                   v
   检索失败 12-14%      推导失败 ~40%      推理失败 ~30%      校准失败 ~10%
```

### 任务设计

每个任务包含：
- **问题**：需要跨源证据的复杂查询。
- **参考答案**：附有**四级源溯证记录**（source-provenance records with four disclosure levels）：
  - L1：最终答案
  - L2：关键事实清单
  - L3：每个事实对应的来源 URL
  - L4：原文片段与发布时间戳
- **交叉源检验**（cross-source checks）：自动验证答案是否与多个独立来源一致。

### 评测指标

```
Score = w_R · Retrieval_F1 + w_D · Derivation_Accuracy 
      + w_Re · Reasoning_Consistency + w_C · Calibration_ECE
```

其中 ECE (Expected Calibration Error) 衡量模型置信度与实际正确率的差距 —— **首次将校准误差纳入深度研究基准**。

## 实验结果

### 跨模型表现

评估了 9 个前沿模型，**没有一个超过 50% 综合得分**（具体数字论文中给出）。两类典型错误模式：
- **强模型（如 GPT-5.5）**：表现为"不完整推导"（incomplete derivation）—— 找到了证据但推导链断裂。
- **弱模型**：表现为"幻觉精确"（hallucinated precision）—— 给出看似精确的数字，实际无据可查。

### 错误分布（关键发现）

| 错误类型 | 占比 | 含义 |
|---------|------|------|
| 检索失败 | 12-14% | 找不到关键来源 |
| 推导失败 | ~40% | 找到证据但无法整合 |
| 推理失败 | ~30% | 多步逻辑断裂 |
| 校准失败 | 10-15% | 应该承认不确定但坚持给答案 |

**核心洞察：检索不是瓶颈。** 这一发现直接挑战了过去三年"通过更好的检索器/RAG 来提升深度研究"的主流路线。

### 跨模型一致性

跨模型评估一致性 ρ = 0.61，最高分歧达 **18.8 个百分点**。这说明现有评测协议自身存在显著噪声，需要更严格的标准化。

## 贡献

1. **基准设计**：首个明确解耦四大能力的深度研究基准。
2. **源溯证机制**：四级披露 + 跨源检验，使评测可审计。
3. **错误归因**：定量证明"瓶颈在推导和校准，不在检索"。
4. **公开发布**：数据、评分细则、评测代码全开源。

## 为什么重要

DeepWeb-Bench 标志着深度研究评测进入新阶段：

- **从"能不能找到答案" → "能不能可靠地推导"**：评测的核心从信息获取转向证据整合。
- **校准首次成为一级指标**：模型必须知道自己什么时候不该回答。
- **指导后续研究方向**：[[argus]] 的证据汇总思路、[[pi_serini]] 的"检索已经足够好"结论、[[hdri]] 的假设驱动方法，都呼应 DeepWeb-Bench 揭示的"推导/校准瓶颈"。

## 局限与未解决问题

- 单审计员评分，未做多审计员一致性检验。
- 主要为英文任务，跨语言深度研究尚未覆盖。
- 长程推导评分仍依赖 LLM judge，存在偏差风险。

## 与相关工作的关系

- **取代基准**：BrowseComp、GAIA、HLE 在前沿模型上已经饱和，DeepWeb-Bench 提供新的难度梯度。
- **互补基准**：[[browsecomp_plus]] 隔离检索器与 LLM，DeepWeb-Bench 隔离能力维度，两者形成"检索器/能力"双视角评测体系。
- **方法启示**：促使 [[argus]]、[[t2po]]、[[lite_researcher]] 等转向证据整合与不确定性建模。
