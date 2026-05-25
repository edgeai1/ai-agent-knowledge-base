---
title: "What Twelve LLM Agent Benchmark Papers Disclose About Themselves: A Pilot Audit and an Open Scoring Schema"
authors:
  - Mahdi Naser Moghadasi
  - Faezeh Ghaderi
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.21404
tags:
  - benchmark
  - meta-evaluation
  - reproducibility
  - audit
  - disclosure-standards
  - scoring-schema
status: done
---

# LLM Agent Benchmark 审计：12 篇基准论文的披露分析与开放评分体系

## 摘要

本文是 LLM 智能体评测领域的**警钟**：**12 篇知名基准论文披露分平均仅 0.38/1.0**（满分 1.0），远低于经典静态基准的 0.66。**没有一篇 agent benchmark 论文披露推理成本**，**没有一篇完整披露评测环境的容器镜像**。这意味着即使复刻论文报告的数字，也无法重现原始的实验条件。研究者提出**五字段审计模式**（基准身份、harness 规范、推理设置、成本报告、失败拆解）+ 评分守则 + 开放 CSV 数据，让社区可重复地审计任何新基准。

## 动机与问题

### 一个尴尬的事实

研究者经常遇到这种情况：

```
论文 A 报告：GPT-4o 在 AgentBench 上 65.3%
论文 B 报告：GPT-4o 在 AgentBench 上 51.8%
                              ↑↑↑
                  同基准 + 同模型，差 13.5 pp
                  
但你无法判断为什么 —— 因为：
  ✗ 不知道用的哪个 harness（ReAct? CodeAct? Reflexion?）
  ✗ 不知道 max_tokens / temperature / seed
  ✗ 不知道工具子集
  ✗ 不知道评测脚本版本
```

**当基准本身披露不足时，"在 X 基准上的 SOTA"成为一个无意义的声称。**

### LLM 智能体基准 vs 经典静态基准

静态基准（如 MMLU、GSM8K）：
```
输入 → 模型 → 输出 → EM 评分
```
披露需求低：只需说清评估指标。

智能体基准：
```
输入 → harness → 工具调用循环 → 多轮交互 → 评分
       ↑         ↑              ↑           ↑
       多种      子集            采样          复杂规则
```

每一步都有大量隐式选择，但论文常常不写。

## 方法：五字段审计模式

研究者将披露需求形式化为 5 个维度：

### 1. Benchmark Identity（基准身份）

```
✓ 数据集版本
✓ 任务子集（如果用了）
✓ 评测时间快照（数据集是否会变化？）
✓ License
```

### 2. Harness Specification（评测框架规范）

```
✓ Agent 类型（ReAct / Reflexion / CodeAct / ...）
✓ 提示词模板（完整文本）
✓ 工具列表（含 API 版本）
✓ 评测脚本（最好是容器镜像 + 哈希）
```

### 3. Inference Settings（推理设置）

```
✓ Temperature
✓ Top-p / Top-k
✓ Max tokens
✓ Seed（可复现性必需）
✓ 系统提示词
```

### 4. Cost Reporting（成本报告）

```
✓ Token 消耗（输入/输出，平均/总和）
✓ 推理调用次数
✓ Wall-clock 时间
✓ 美元成本（如果适用）
```

### 5. Failure Breakdown（失败拆解）

```
✓ 错误类型分布（工具调用错误 / 推理错误 / 评估错误 / ...）
✓ 部分成功的处理方式
✓ 超时 / API 限制的影响
```

### 评分守则

每个维度按 0/0.5/1.0 评分：

```
0:    完全不披露
0.5:  部分披露（够定性但不够复现）
1.0:  完整披露（可一键复现）
```

总分 = 5 个维度的平均。

## 审计结果

### 12 篇基准的得分

研究者审计了 12 篇知名基准论文：8 篇 agent benchmark + 4 篇 classical（作为对照）。

```
Agent Benchmarks (8 papers):     平均 0.38 / 1.0
Classical Benchmarks (4 papers): 平均 0.66 / 1.0
```

**Agent 基准比经典基准披露差近一倍。**

### 各维度细节

| 维度 | Agent 平均 | Classical 平均 |
|------|-----------|----------------|
| 基准身份 | 0.62 | 0.85 |
| Harness 规范 | 0.30 | 0.70 |
| 推理设置 | 0.45 | 0.72 |
| **成本报告** | **0.0** | 0.20 |
| 失败拆解 | 0.50 | 0.85 |

### 关键发现

#### Finding 1: 零成本披露

> **没有一篇 agent benchmark 论文以任何形式披露推理成本。**

这意味着：
- 你不知道一个"SOTA 系统"花了 $1 还是 $1000。
- 无法做公平的"准确率/成本"对比。
- 实际部署成本完全黑盒。

#### Finding 2: 零容器化评测环境

> **没有一篇完整披露评测环境的容器镜像（含内容哈希）。**

后果：
- 不同时间复现得到不同结果（API 漂移、工具版本变化）。
- 工具集差异导致 10+ pp 性能波动。

#### Finding 3: 不同评估之间分歧巨大

```
12 篇论文中，同一基准 + 同一模型的报告：
  最大分歧达 18.8 pp
  这一分歧无法用任何披露字段解释
```

这就是为什么"我们的方法比 X 论文的方法高 5 pp"经常没有意义 —— 噪声远大于信号。

## 评分模式的应用

研究者发布了完整的 JSON 模式：

```json
{
  "paper_id": "...",
  "benchmark_identity": {
    "dataset_version": null,
    "task_subset": "partial",
    "evaluation_date": null,
    "score": 0.5
  },
  "harness_specification": { ... },
  "inference_settings": { ... },
  "cost_reporting": {
    "input_tokens": null,
    "output_tokens": null,
    "wall_clock": null,
    "score": 0.0
  },
  "failure_breakdown": { ... },
  "total_score": 0.38
}
```

任何研究者都可以用这个模式审计新基准。

## 为什么有效

### 1. 量化披露问题

过去"披露不足"是定性印象，现在有了具体数字。**0.38 这个数字是震撼性的**。

### 2. 五字段是最小必要集

不要求论文披露一切，但**这五个字段是最低门槛**。无此无法复现。

### 3. 对照组揭示问题严重性

Classical benchmarks 也不完美（0.66），但 agent 是 0.38 —— 这说明 agent 社区有特别严重的披露文化问题。

### 4. 开放数据可审计

CSV 原始评分公开，任何人可以挑战评分。这本身就是"用披露解决披露问题"的元方法。

## 贡献

1. **首个 agent 基准的系统审计**：12 篇论文，5 维度评分。
2. **可重用模式**：JSON 模式 + 守则，任何人可用。
3. **关键发现**：零成本披露 + 零容器化评测环境。
4. **量化噪声**：18.8 pp 分歧揭示评测不可靠性。
5. **行业呼吁**：推动 agent 基准社区建立披露标准。

## 局限与未解决问题

- 单审计员评分，未做多审计员一致性。
- 12 篇为代表性样本，非全面普查。
- 评分守则在边界情况（如部分披露）可能不一致。

## 与相关工作的关系

### 直接呼应
- 与 [[biomed_arena]] 互补：本文揭示问题（评测不可复现），BioMedArena 提供解决方案（统一工具包）。
- 与 [[deepweb_bench]] 互补：DeepWeb-Bench 提供新基准（包含完整披露），本文设定了"什么是好披露"的标准。
- 与 [[browsecomp_plus]] 同思想：都强调"公平、透明、可复现"的评估。

### 行业意义
- 类似 ML reproducibility crisis 文献（如 [Pineau et al. 2021](https://arxiv.org/abs/2003.12206)）。
- 可能促使顶会（NeurIPS、ICML）要求基准论文的"披露清单"。

### 后续方向
- 多审计员审计 + 一致性分析。
- 自动化审计工具（用 LLM 评分论文披露）。
- 扩展到其他领域（如 LLM 安全基准、多模态基准）。
