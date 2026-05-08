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

## TL;DR

DeepAgent is an end-to-end deep reasoning agent that performs autonomous thinking, tool
discovery, and action execution within a single coherent reasoning process. It scales to
16,000+ RapidAPI tools through an autonomous memory folding mechanism (episodic, working,
and tool memories) and is trained with ToolPO -- a novel RL method using LLM-simulated
APIs with tool-call advantage attribution for fine-grained credit assignment to individual
tool invocations. DeepAgent-32B-RL achieves state-of-the-art on 8 benchmarks, including
89.0% on TMDB, 91.8% on ALFWorld, and 53.3 on GAIA.

## Motivation & Problem

Current tool-augmented LLM agents face three core challenges:

1. **Tool discovery at scale**: Most agents assume a small, pre-defined toolset. Real-world
   scenarios require discovering and selecting from thousands of available APIs (e.g.,
   16,000+ on RapidAPI), which existing methods cannot handle.

2. **Long-horizon context collapse**: Multi-step tool use generates lengthy interaction
   histories that overflow context windows, leading to error accumulation and loss of
   critical information.

3. **Sparse reward attribution**: Standard RL methods assign reward only to the final
   outcome, but in multi-tool reasoning chains, credit must be attributed to individual
   tool calls to learn which invocations contributed to success.

## Method

### System Architecture

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

### Autonomous Memory Folding

When interaction history exceeds a threshold, DeepAgent compresses past interactions into
three structured memory types:

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

### ToolPO: Tool-Call Advantage Attribution RL

Standard RL (e.g., PPO, GRPO) assigns a single reward to the entire trajectory. ToolPO
introduces fine-grained credit assignment at the tool-call level:

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

### Training Pipeline

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

## Key Innovations

1. **Unified reasoning-discovery-action loop**: DeepAgent integrates thinking, tool
   discovery, and action execution into a single coherent reasoning process rather
   than treating them as separate pipeline stages.

2. **Memory folding**: A principled approach to context compression that preserves
   task-critical information across three complementary memory types (episodic,
   working, tool), enabling long-horizon interactions without context overflow.

3. **ToolPO with tool-call advantage attribution**: First RL method to assign
   fine-grained credit to individual tool invocations within multi-step reasoning
   chains, rather than relying solely on sparse trajectory-level rewards.

4. **Scale to 16K+ tools**: Demonstration that open-set tool discovery from massive
   tool corpora is feasible, moving beyond the assumption of small, pre-defined toolsets.

5. **LLM-simulated APIs**: Using LLMs to simulate API responses enables scalable RL
   training without the cost and reliability issues of real API calls.

## Experimental Setup

- **Model**: QwQ-32B backbone, Qwen2.5-32B-Instruct auxiliary
- **Benchmarks** (8 total):
  - General tool-use (labeled tools): TMDB, Spotify, API-Bank
  - General tool-use (open-set discovery): ToolBench, ToolHop
  - Downstream applications: ALFWorld, WebShop, GAIA, HLE
- **Baselines**: GPT-4o, Claude, Qwen-Agent, ReAct, ToolLlama, and other
  32B-class reasoning agents
- **Settings**: Both labeled-tool (tools given) and open-set (agent must discover tools)

## Results

### Labeled-Tool Tasks

| Benchmark | DeepAgent-32B-RL | Best 32B Baseline | Improvement |
|-----------|------------------|-------------------|-------------|
| TMDB      | 89.0%            | 55.0%             | +34.0       |
| Spotify   | 75.4%            | 52.6%             | +22.8       |
| API-Bank  | Strong           | --                | --          |

### Open-Set Tool Discovery

| Benchmark | DeepAgent-32B-RL | Best Baseline | Improvement |
|-----------|------------------|---------------|-------------|
| ToolBench | 64.0%            | 54.0%         | +10.0       |
| ToolHop   | 40.6%            | 29.0%         | +11.6       |

### Downstream Applications

| Benchmark | DeepAgent-32B-RL | Notes                        |
|-----------|------------------|------------------------------|
| ALFWorld  | 91.8%            | Success rate                 |
| WebShop   | 34.4% / 56.3     | Success rate / Score         |
| GAIA      | 53.3             | Outperforms workflow agents  |
| HLE       | Higher than WF   | Vs. workflow-based agents    |

### Ablation Results

- Memory folding improves performance significantly on long-horizon tasks
  (ALFWorld, GAIA) by reducing error accumulation
- ToolPO outperforms standard GRPO on tool-intensive benchmarks by 5-10%
  through better credit assignment to tool invocation tokens
- Tool simulation enables training at scale without real API costs

## Limitations

1. **Base model dependency**: Performance is tied to QwQ-32B's reasoning capability;
   weaker base models may not benefit as much from ToolPO training.

2. **Simulation fidelity**: LLM-simulated APIs may not perfectly capture the behavior
   of real APIs, particularly for edge cases, errors, and rate limiting.

3. **Memory folding heuristics**: The trigger threshold and compression strategy for
   memory folding involve design choices that may need task-specific tuning.

4. **Tool discovery noise**: With 16K+ tools, the search space is enormous; false
   positive tool matches can lead the agent down unproductive paths.

5. **Evaluation scope**: While 8 benchmarks are covered, the tasks are primarily
   text-based; multimodal tool use (vision APIs, audio processing) is not evaluated.

6. **Reproducibility cost**: Training ToolPO with simulated APIs at scale requires
   significant compute resources and access to auxiliary LLMs for simulation.

## Key Takeaways

1. The unified thinking-discovery-action paradigm is more effective than pipeline
   approaches that separate reasoning from tool selection and execution. Integration
   enables the reasoning process to inform tool selection and vice versa.

2. Memory folding is an essential mechanism for long-horizon agent interactions. The
   three-way decomposition (episodic/working/tool) captures complementary aspects of
   interaction history and enables meaningful compression without critical information
   loss.

3. Tool-call advantage attribution (ToolPO) solves a fundamental credit assignment
   problem in tool-augmented RL. By attributing reward to individual tool calls rather
   than entire trajectories, the agent learns which tool invocations are valuable.

4. Open-set tool discovery from massive corpora (16K+ APIs) is feasible and practically
   useful. This moves the field beyond the assumption that agents operate with small,
   curated toolsets.

5. LLM-simulated APIs are a viable and cost-effective proxy for real APIs during RL
   training, enabling scalable experimentation without real-world API costs and
   reliability concerns.

6. The massive gains over baselines on TMDB (+34%) and Spotify (+22.8%) suggest that
   current tool-use agents are substantially undertrained, and proper RL with fine-grained
   credit assignment can unlock significant performance improvements.
