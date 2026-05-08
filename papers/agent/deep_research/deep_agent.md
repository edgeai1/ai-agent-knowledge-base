---
title: "DeepAgent: A General Reasoning Agent with Scalable Toolsets"
authors:
  - Xiaoxi Li
  - Wenxiang Jiao
  - Jiarui Jin
  - Guanting Dong
  - Jiajie Jin
  - Yinuo Wang
  - Hao Wang
  - Yutao Zhu
  - Ji-Rong Wen
  - Yuan Lu
  - Zhicheng Dou
venue: WWW 2026 (ACM Web Conference 2026, Oral)
year: 2026
url: https://arxiv.org/abs/2510.21618
tags:
  - deep-agent
  - tool-use
  - reinforcement-learning
  - memory-folding
  - reasoning-agent
  - ToolPO
  - scalable-toolsets
status: done
---

## 摘要

DeepAgent 是一个端到端深度推理智能体，在单一连贯的推理过程中执行自主思考、工具发现和动作执行。它通过自主记忆折叠机制（情景记忆、工作记忆和工具记忆）扩展到 16,000 多个 RapidAPI 工具，并使用 ToolPO——一种新型强化学习方法进行训练，该方法利用 LLM 模拟的 API 和工具调用优势归因实现对单个工具调用的细粒度信用分配。DeepAgent-32B-RL 在 8 个基准测试上达到最先进水平，包括 TMDB 上的 89.0%、ALFWorld 上的 91.8% 和 GAIA 上的 53.3。

## 动机与问题

当前工具增强的 LLM 智能体面临三个核心挑战：

1. **大规模工具发现**：大多数智能体假设有一组小型、预定义的工具集。真实场景需要从数千个可用 API 中发现和选择（例如 RapidAPI 上的 16,000 多个），现有方法无法处理。

2. **长程上下文崩溃**：多步工具使用生成冗长的交互历史，溢出上下文窗口，导致错误累积和关键信息丢失。

3. **稀疏奖励归因**：标准强化学习方法仅对最终结果分配奖励，但在多工具推理链中，信用必须归因到单个工具调用，以学习哪些调用对成功有贡献。

## 方法

### 系统架构

```
+================================================================+
|                    DeepAgent Architecture                        |
+================================================================+
|                                                                  |
|  +------------------+    +------------------+                    |
|  |   Query Input    | -> | Deep Reasoning   |                    |
|  +------------------+    | Process          |                    |
|                          | (unified think + |                    |
|                          |  discover + act) |                    |
|                          +--------+---------+                    |
|                                   |                              |
|              +--------------------+--------------------+         |
|              |                    |                    |          |
|     +--------v-------+  +--------v-------+  +---------v------+  |
|     | TOOL DISCOVERY |  | ACTION         |  | MEMORY         |  |
|     | - Search 16K+  |  | EXECUTION      |  | FOLDING        |  |
|     |   RapidAPIs    |  | - API calls    |  | - Episodic     |  |
|     | - Match by     |  | - Parse results|  | - Working      |  |
|     |   description  |  | - Error handle |  | - Tool         |  |
|     +----------------+  +----------------+  +----------------+  |
|                                                                  |
+================================================================+
```

### 自主记忆折叠

当交互历史超过阈值时，DeepAgent 将过去的交互压缩为三种结构化记忆类型：

```
MEMORY_FOLD(interaction_history):
    if len(interaction_history) > threshold:

        episodic_memory = compress(
            key_events, decisions, sub-task_completions
        )
        # High-level log of what happened and what was decided

        working_memory = extract(
            current_sub_goal, near_term_plans, active_constraints
        )
        # Short-term focus: what the agent is currently doing

        tool_memory = consolidate(
            tool_calls, API_responses, success/failure_patterns
        )
        # Tool-specific learning: which tools worked for what

        return FOLDED_CONTEXT(episodic, working, tool)
```

```
+------------------------------------------------------------------+
|              Memory Folding Mechanism                              |
+------------------------------------------------------------------+
|                                                                    |
|  Raw History:  [Turn1][Turn2][Turn3]...[Turn_N]  (context overflow)|
|                          |                                         |
|                    FOLD TRIGGER                                    |
|                          |                                         |
|                          v                                         |
|  +------------------+------------------+-------------------+       |
|  | Episodic Memory  | Working Memory   | Tool Memory       |      |
|  |------------------|------------------|-------------------|      |
|  | - Completed      | - Current goal   | - API call log    |      |
|  |   sub-tasks      | - Next steps     | - Success/fail    |      |
|  | - Key decisions  | - Constraints    |   patterns        |      |
|  | - Milestones     | - Active vars    | - API schemas     |      |
|  +------------------+------------------+-------------------+       |
|                          |                                         |
|                          v                                         |
|             Compressed context << original length                  |
+------------------------------------------------------------------+
```

### ToolPO：工具调用优势归因强化学习

标准强化学习（如 PPO、GRPO）对整个轨迹分配单一奖励。ToolPO 在工具调用级别引入细粒度信用分配：

```
ToolPO Training Process:

1. TOOL SIMULATION
   - Use LLM to simulate API responses for large-scale tool corpus
   - Enables training without hitting real APIs (cost, rate limits)
   - Simulated APIs cover diverse domains and failure modes

2. TRAJECTORY COLLECTION
   - Agent generates reasoning traces with tool calls
   - Each trajectory: [think_1, tool_call_1, result_1, ..., answer]

3. TOOL-CALL ADVANTAGE ATTRIBUTION
   For each tool call t_i in trajectory T:
       advantage(t_i) = R(T) * credit(t_i)

   where credit(t_i) is computed by:
       - Comparing trajectories with/without t_i
       - Measuring marginal contribution of t_i to final success
       - Assigning higher weight to tokens within tool invocations

4. POLICY UPDATE
   - Update policy using attributed advantages
   - Tool invocation tokens receive fine-grained gradient signal
   - Non-tool tokens receive standard trajectory-level reward
```

### 训练流水线

```
Phase 1: Supervised Fine-Tuning (SFT)
    Base model (QwQ-32B) + curated tool-use demonstrations
    --> SFT model with basic tool-calling ability

Phase 2: ToolPO Reinforcement Learning
    SFT model + LLM-simulated APIs + tool-call attribution
    --> DeepAgent-32B-RL with refined tool-use reasoning

Backbone: QwQ-32B (reasoning model)
Auxiliary: Qwen2.5-32B-Instruct (for tool simulation)
```

## 关键创新

1. **统一的推理-发现-执行循环**：DeepAgent 将思考、工具发现和动作执行集成到单一连贯的推理过程中，而非作为独立的流水线阶段处理。

2. **记忆折叠**：一种有原则的上下文压缩方法，通过三种互补的记忆类型（情景、工作、工具）保留任务关键信息，在不丢失关键信息的情况下实现长程交互。

3. **带有工具调用优势归因的 ToolPO**：首个将细粒度信用分配到多步推理链中单个工具调用的强化学习方法，而非仅依赖稀疏的轨迹级奖励。

4. **扩展至 16K 以上工具**：证明了从大规模工具语料库中进行开放集工具发现的可行性，超越了智能体使用小型预定义工具集的假设。

5. **LLM 模拟的 API**：使用 LLM 模拟 API 响应，实现了无需真实 API 调用成本和可靠性问题的可扩展强化学习训练。

## 实验设置

- **模型**：QwQ-32B 骨干网络，Qwen2.5-32B-Instruct 辅助
- **基准测试**（共 8 个）：
  - 通用工具使用（标注工具）：TMDB、Spotify、API-Bank
  - 通用工具使用（开放集发现）：ToolBench、ToolHop
  - 下游应用：ALFWorld、WebShop、GAIA、HLE
- **基线**：GPT-4o、Claude、Qwen-Agent、ReAct、ToolLlama 及其他 32B 级推理智能体
- **设置**：标注工具（给定工具）和开放集（智能体必须自行发现工具）

## 结果

### 标注工具任务

| 基准测试 | DeepAgent-32B-RL | 最佳 32B 基线 | 提升 |
|-----------|------------------|-------------------|-------------|
| TMDB      | 89.0%            | 55.0%             | +34.0       |
| Spotify   | 75.4%            | 52.6%             | +22.8       |
| API-Bank  | 强               | --                | --          |

### 开放集工具发现

| 基准测试 | DeepAgent-32B-RL | 最佳基线 | 提升 |
|-----------|------------------|---------------|-------------|
| ToolBench | 64.0%            | 54.0%         | +10.0       |
| ToolHop   | 40.6%            | 29.0%         | +11.6       |

### 下游应用

| 基准测试 | DeepAgent-32B-RL | 备注                        |
|-----------|------------------|------------------------------|
| ALFWorld  | 91.8%            | 成功率                 |
| WebShop   | 34.4% / 56.3     | 成功率 / 得分         |
| GAIA      | 53.3             | 优于工作流智能体  |
| HLE       | 高于 WF   | 对比基于工作流的智能体    |

### 消融实验结果

- 记忆折叠在长程任务（ALFWorld、GAIA）上通过减少错误累积显著提升了性能
- ToolPO 在工具密集型基准测试上通过更好的工具调用词元信用分配比标准 GRPO 高出 5-10%
- 工具模拟实现了无需真实 API 成本的大规模训练

## 局限性

1. **基础模型依赖**：性能与 QwQ-32B 的推理能力密切相关；较弱的基础模型可能无法从 ToolPO 训练中获得同等收益。

2. **模拟保真度**：LLM 模拟的 API 可能无法完美捕捉真实 API 的行为，特别是在边缘情况、错误和速率限制方面。

3. **记忆折叠的启发式方法**：触发阈值和记忆折叠的压缩策略涉及可能需要任务特定调优的设计选择。

4. **工具发现噪声**：在 16K 以上的工具中，搜索空间巨大；误匹配的工具可能导致智能体走上无效的路径。

5. **评估范围**：虽然涵盖了 8 个基准测试，但任务主要是基于文本的；多模态工具使用（视觉 API、音频处理）未被评估。

6. **复现成本**：大规模使用模拟 API 训练 ToolPO 需要大量计算资源和辅助 LLM 的访问。

## 核心要点

1. 统一的思考-发现-执行范式比将推理与工具选择和执行分离的流水线方法更有效。集成使推理过程能够指导工具选择，反之亦然。

2. 记忆折叠是长程智能体交互的关键机制。三重分解（情景/工作/工具）捕获了交互历史的互补方面，在不丢失关键信息的情况下实现有意义的压缩。

3. 工具调用优势归因（ToolPO）解决了工具增强强化学习中的一个根本性信用分配问题。通过将奖励归因到单个工具调用而非整个轨迹，智能体学会了哪些工具调用是有价值的。

4. 从大规模语料库（16K 以上 API）中进行开放集工具发现是可行且实用的。这将该领域从智能体使用小型策划工具集的假设推进到更实际的场景。

5. LLM 模拟的 API 是强化学习训练中真实 API 的可行且成本有效的替代，实现了无需真实 API 成本和可靠性担忧的可扩展实验。

6. 在 TMDB（+34%）和 Spotify（+22.8%）上相对基线的巨大提升表明，当前工具使用智能体的训练严重不足，适当的强化学习配合细粒度信用分配可以释放显著的性能提升。
