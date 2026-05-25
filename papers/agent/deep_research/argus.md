---
title: "Argus: Evidence Assembly for Scalable Deep Research Agents"
authors:
  - Zhen Zhang
  - Liangcai Su
  - Zhuo Chen
  - Xiang Lin
  - Haotian Xu
  - Simon Shaolei Du
  - Kaiyu Yang
  - Bo An
  - Lidong Bing
  - Xinyu Wang
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.16217
tags:
  - deep-research
  - multi-agent
  - evidence-assembly
  - parallel-search
  - reinforcement-learning
  - context-efficiency
status: done
---

# Argus：可扩展深度研究智能体的证据汇总

## 摘要

Argus 提出了一个范式转变：**与其用并行 rollout 暴力探索多条轨迹（如 GPT-5 Pro、Kimi-Researcher 的做法），不如将信息搜索视为"用互补证据拼图"**。系统采用 Searcher-Navigator 双智能体协同架构，**Navigator 维护共享证据图，识别缺失片段并调度 Searcher**。结果令人震撼：单 Searcher 平均提升 5.5 点，8 个并行 Searcher 提升 12.7 点，**64 个 Searcher 在 BrowseComp 上达到 86.2，超越所有专有智能体**，且推理上下文仍控制在 21.5K tokens 以内。

## 动机与问题

### 现有并行 rollout 方案的核心缺陷

像 GPT-5 Pro、Kimi-Researcher 等顶级系统通过**并行运行多个独立 ReAct 轨迹**（rollout）然后投票/聚合来扩展深度研究。然而这种方案有三个本质问题：

1. **重复搜索浪费**：每个 rollout 都从零开始，搜索的内容高度重叠。
2. **缺乏协调**：无法系统地覆盖证据空间，靠"运气"找全关键证据。
3. **上下文爆炸**：聚合所有 rollout 结果导致主推理上下文超长，需要更大模型才能处理。

### 拼图视角的核心洞察

研究者重新审视问题本质：

> 找答案不像"广撒网希望某条轨迹成功"，而像**在多个不同地方找拼图碎片，然后把它们拼起来**。

如果有一个"导演"知道当前已收集了什么、还缺什么，它可以**精准派发"找特定碎片"的任务**，效率远高于让 64 个工人独立瞎找。

## 方法

### 架构：Searcher + Navigator

```
                  +-----------------------+
                  |       Navigator        |  ←→  Shared Evidence Graph
                  |  - 维护证据图          |       (节点 = 已知事实,
                  |  - 识别缺失片段        |        边 = 来源关系)
                  |  - 调度子查询          |
                  |  - 综合最终答案        |
                  +-----------------------+
                          |        ^
                Dispatch  |        | Evidence Trace
                          v        |
        +-----------+ +-----------+ +-----------+
        | Searcher1 | | Searcher2 | | Searcher_N|   并行
        |  (ReAct)  | |  (ReAct)  | |  (ReAct)  |   1 ~ 64 个
        +-----------+ +-----------+ +-----------+
            ↓             ↓             ↓
        Web API       Web API       Web API
```

### Searcher：标准 ReAct 智能体

- 接收 Navigator 指定的子查询。
- 自由探索网页：search → click → read → think → report。
- 输出**证据迹**（evidence trace）：发现的事实 + 来源 URL + 引用片段。

### Navigator：协调与综合（RL 训练）

Navigator 是系统的核心创新，负责四件事：

1. **证据图维护**：将所有 Searcher 返回的事实组织为图结构，节点是事实，边是"来源/关系"。
2. **缺失识别**：分析当前图与目标问题，决定哪些事实还需补充。
3. **任务派发**：生成下一轮 Searcher 的子查询（可派发 1 个或 64 个，灵活伸缩）。
4. **答案综合**：基于完整证据图给出带溯源的最终答案。

Navigator **独立用强化学习训练**，与 Searcher 解耦 —— 这使得同一 Navigator 可以驱动任意数量的 Searcher，**无需重训练即可在推理时扩展规模**。

### 关键设计：上下文隔离

每个 Searcher 处理自己的本地上下文（网页内容、长引文），**Navigator 只看汇总后的证据图节点**。这是 Argus 能在 64 路并行下仍保持 21.5K tokens 主上下文的关键。

```
Searcher 上下文：~100K tokens（含原始网页）
            ↓
Searcher 输出：~500 tokens（结构化证据迹）
            ↓
Navigator 看到的证据：64 × 500 = 32K
            ↓
Navigator 推理上下文：21.5K（去重 + 压缩）
```

## 实验结果

### 主基准（8 个数据集平均）

| 配置 | 提升 (vs. 单 ReAct 基线) |
|------|------------------------|
| 单 Searcher (Argus-1) | +5.5 pt |
| 8 并行 Searcher (Argus-8) | +12.7 pt |
| 64 并行 Searcher (Argus-64) | **+15+ pt** |

### BrowseComp 上的极限性能

```
Argus-64:                86.2%  ← 超越所有专有系统
GPT-5.5 Pro:             90.1%  (闭源)
GPT-5.4 Pro:             89.3%  (闭源)
Claude Mythos Preview:   86.9%  (闭源)
Argus-8:                 ~79%
WebDancer-72B:           ~75%
单 ReAct (35B-A3B MoE):  ~71%
```

**首个开源系统超越主流专有 deep research 服务**（不计 GPT-5.5/5.4 Pro 的最新版本）。

### 模型与参数

- **Backbone**：35B-A3B MoE（35B 总参数，3B 激活参数）。
- **训练数据**：Navigator RL 数据由 GPT-4o 标注 + 自蒸馏。
- **推理成本**：64 Searcher 单题约 $0.42，远低于 GPT-5 Pro deep research。

## 为什么有效

### 1. 协调取代蛮力

随机并行 ≠ 系统覆盖。Navigator 的"缺什么找什么"调度，让 64 个 Searcher 的实际证据覆盖远超 64 倍单 Searcher。

### 2. 上下文压缩 = 主推理能聚焦

主上下文 21.5K 让 Navigator 可以**深度推理**而非苦于上下文管理。这与 [[t2po]] 揭示的"长上下文会稀释信号"一致。

### 3. RL 训练 Navigator 而非 Searcher

Searcher 用通用 ReAct 智能体即可（甚至可换成 GPT-4o）；Navigator 是真正需要专门训练的"指挥官"。这种**分工训练**比端到端训练整个团队更高效。

## 贡献

1. **新范式**：用证据汇总取代并行投票，是后并行时代深度研究的方向。
2. **架构创新**：Searcher/Navigator 双智能体可独立扩展。
3. **效率突破**：64 路并行仍 < 22K tokens 主上下文。
4. **性能突破**：开源系统首次接近闭源前沿。
5. **训练独立性**：Navigator 与 Searcher 数量解耦，灵活伸缩。

## 局限与未解决问题

- Navigator 训练需大量高质量 trajectory，数据成本高。
- 64 并行下 Web API 调用成本仍高，需进一步优化。
- 证据图结构对新型多模态来源（视频、PDF 图表）支持有限。

## 与相关工作的关系

### 取代方向
- **取代并行投票**：GPT-5 Pro、Kimi-Researcher 风格的"多路独立 rollout"被证明效率低下。

### 范式延续
- 延续 [[mindsearch]] 的"图结构组织信息"思想。
- 延续 [[step_deep_research]]、[[webdancer]] 的"分阶段搜索"思想。

### 互补关系
- 与 [[t2po]] 互补：T²PO 解决单 trajectory 内的探索-利用平衡，Argus 解决多 trajectory 间的协调。
- 与 [[pi_serini]] 互补：Pi-Serini 证明检索器够用了，Argus 证明协调器是瓶颈。

### 后续方向
- 多模态证据图（图像、表格证据）
- 动态规模调整（根据问题难度决定 Searcher 数量）
- 用户反馈融入证据图
