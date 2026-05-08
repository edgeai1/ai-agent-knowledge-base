---
title: "Voyager: An Open-Ended Embodied Agent with Large Language Models"
authors: Guanzhi Wang, Yuqi Xie, Yunfan Jiang, Ajay Mandlekar, Chaowei Xiao, Yuke Zhu, Linxi Fan, Anima Anandkumar
venue: NeurIPS 2023 (Spotlight)
year: 2023
url: https://arxiv.org/abs/2305.16291
tags: [embodied, lifelong-learning, skill-library, minecraft, code-generation, foundational]
status: done
---

## TL;DR

Voyager is the first LLM-powered lifelong learning agent in Minecraft that continuously explores, acquires diverse skills, and makes novel discoveries without human intervention, using an automatic curriculum, a skill library of executable code, and an iterative prompting mechanism for self-verified code generation.

## Motivation & Problem

Existing approaches to building agents in open-ended environments face fundamental limitations:

1. **Reinforcement learning agents** (e.g., DreamerV3, DEPS) require extensive environment interaction, reward shaping, and struggle to generalize across the vast skill space of open-ended worlds like Minecraft.
2. **LLM-based agents** at the time (e.g., ReAct, Inner Monologue) operated in a "one-shot" fashion -- they could plan and act but did not retain skills across episodes or build upon past successes.
3. **No lifelong learning**: No prior system demonstrated an agent that could continuously expand its capabilities over time, composing earlier skills into more complex behaviors.

The key insight is that LLMs possess vast world knowledge (including knowledge about Minecraft) but lack the ability to: (a) systematically explore an environment, (b) convert experiences into reusable skills, and (c) compose skills hierarchically. Voyager addresses all three gaps.

## Method

### System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        VOYAGER                                │
│                                                               │
│  ┌─────────────────┐    ┌──────────────────────────────────┐ │
│  │   Automatic      │    │    Iterative Prompting           │ │
│  │   Curriculum     │    │    Mechanism                     │ │
│  │                  │    │                                  │ │
│  │  Current state ──┼───►│  1. Generate code (GPT-4)       │ │
│  │  Exploration     │    │  2. Execute in Minecraft         │ │
│  │  progress        │    │  3. Get environment feedback     │ │
│  │  Skill inventory │    │  4. Self-verify success          │ │
│  │       │          │    │  5. Retry if failed (up to 4x)   │ │
│  │       ▼          │    │         │                        │ │
│  │  Propose next    │    │         ▼                        │ │
│  │  task/goal       │    │  Success?                        │ │
│  └─────────────────┘    │  ├─ Yes: Add to Skill Library    │ │
│                          │  └─ No:  Refine or skip          │ │
│                          └──────────────────────────────────┘ │
│                                                               │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                   Skill Library                           │ │
│  │                                                           │ │
│  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐         │ │
│  │  │ mine   │  │ craft  │  │ build  │  │ combat │         │ │
│  │  │ Wood   │  │ Planks │  │ House  │  │ Spider │  ...    │ │
│  │  │ (JS)   │  │ (JS)   │  │ (JS)   │  │ (JS)   │         │ │
│  │  └────────┘  └────────┘  └────────┘  └────────┘         │ │
│  │                                                           │ │
│  │  Indexed by: description embeddings (text-embedding-ada) │ │
│  │  Retrieved via: cosine similarity to current task         │ │
│  └──────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────┘
```

### Component 1: Automatic Curriculum

The curriculum module proposes tasks of appropriate difficulty based on the agent's current state. It uses GPT-4 with a prompt that includes:

- **Current inventory**: Items the agent currently holds
- **Current equipment**: What the agent is wearing/wielding
- **Nearby blocks and entities**: Environmental context
- **Current biome**: Desert, forest, ocean, etc.
- **Completed tasks**: History of successfully completed tasks
- **Failed tasks**: Tasks that were attempted but failed

The LLM generates the next task to attempt, following principles:
1. Tasks should be achievable given current resources and skill level
2. Tasks should promote exploration and discovery of new items/areas
3. Difficulty should gradually increase (curriculum learning)
4. Tasks should be diverse -- not repeating the same type endlessly

Example curriculum progression:
```
1. "Mine 1 wood log"           (basic resource gathering)
2. "Craft wooden planks"       (basic crafting)
3. "Craft a crafting table"    (tool prerequisite)
4. "Craft a wooden pickaxe"    (tool creation)
5. "Mine 8 cobblestone"        (using new tool)
6. "Craft a stone pickaxe"     (tool upgrade)
7. "Mine iron ore"             (deeper mining)
8. "Smelt iron ingot"          (processing)
9. "Craft iron sword"          (combat preparation)
10. "Defeat a zombie"          (combat)
...
```

### Component 2: Skill Library

The skill library is a persistent, growing collection of verified executable programs (JavaScript functions using the Mineflayer API for Minecraft).

**Skill structure**:
```javascript
// Skill: mineWoodLog
// Description: Mine 1 wood log from a nearby tree
async function mineWoodLog(bot) {
  const woodBlock = bot.findBlock({
    matching: block => block.name.includes('log'),
    maxDistance: 32
  });
  if (!woodBlock) {
    bot.chat("No wood logs nearby, exploring...");
    // exploration logic
    return;
  }
  await bot.dig(woodBlock);
  bot.chat("Mined a wood log!");
}
```

**Storage and indexing**:
- Each skill is stored as: `{code: string, description: string, embedding: vector}`
- Descriptions are embedded using OpenAI text-embedding-ada-002
- Skills are indexed in a vector database for retrieval

**Retrieval process**:
1. Given a new task, embed the task description
2. Compute cosine similarity with all skill description embeddings
3. Retrieve top-5 most relevant skills
4. Include retrieved skill code in the prompt as examples/building blocks

**Skill composition**: Later skills can call earlier skills. For example, `craftIronSword` might call `mineIronOre`, `smeltIronIngot`, and `craftOnCraftingTable` as sub-routines. This compositionality enables exponential growth in capability.

### Component 3: Iterative Prompting Mechanism

The code generation and refinement loop is the core execution engine:

```
Algorithm: Iterative Prompting for Skill Acquisition
─────────────────────────────────────────────────────
Input: task description T, max_retries = 4
Output: verified skill code or failure

1. Retrieve relevant skills from Skill Library
2. Construct prompt P with:
   - Agent state (inventory, position, biome)
   - Task description T
   - Retrieved skill code as examples
   - Mineflayer API documentation (curated subset)
   - Environment feedback (initially empty)
3. For i = 1 to max_retries:
   a. code = GPT-4(P)               // Generate JavaScript code
   b. result = Execute(code)         // Run in Minecraft
   c. feedback = GetFeedback()       // Execution errors, inventory changes,
                                     //   nearby entities, chat log
   d. success = SelfVerify(T, feedback)  // GPT-4 judges success
   e. If success:
        Store(code, T) in Skill Library
        Return code
      Else:
        P = P + feedback + error_info  // Augment prompt with feedback
4. Return failure
```

**Self-verification**: Rather than relying solely on hard-coded success criteria, Voyager uses GPT-4 to judge whether the task was completed. The LLM receives the task description, the execution feedback (inventory changes, errors, etc.), and decides if the goal was met. This allows flexible success criteria for open-ended tasks.

**Environment feedback includes**:
- JavaScript execution errors and stack traces
- Changes in inventory (before vs. after)
- Changes in nearby blocks/entities
- Bot chat messages and health/hunger status
- Minecraft game event logs

### Prompt Design

The prompts are carefully structured with several sections:

1. **System prompt**: Defines the agent as a Minecraft bot programmer
2. **Code generation guidelines**: Formatting rules, API usage patterns, error handling requirements
3. **Environment context**: Current game state
4. **Retrieved skills**: Code examples from the skill library
5. **Task specification**: What to accomplish
6. **Previous feedback** (if retrying): Error messages and observations from failed attempts

The Mineflayer API documentation is curated to include ~60 commonly used functions rather than the full API, reducing prompt length while covering most use cases.

## Key Innovations

1. **Lifelong learning through code**: Skills are stored as executable code, enabling perfect recall and zero-cost reuse. Unlike neural skill policies that degrade, code-based skills are deterministic and composable.
2. **Automatic curriculum with LLM world knowledge**: The curriculum leverages GPT-4's knowledge of Minecraft's tech tree and progression mechanics, eliminating the need for hand-designed reward functions.
3. **Self-verification**: Using the LLM to judge task completion enables handling of open-ended, hard-to-specify goals.
4. **Compositional skill growth**: Skills build on each other, enabling exponential capability expansion -- a key property missing from prior LLM agent work.

## Experimental Setup

### Environment
- **Minecraft** (Java Edition) via Mineflayer JavaScript API
- Agents spawned in random survival worlds
- No initial inventory, no creative mode advantages
- Continuous open-ended play (no episode resets)

### Baselines
1. **ReAct** (Yao et al., 2023): Reasoning + acting framework, adapted for Minecraft
2. **Reflexion** (Shinn et al., 2023): Self-reflection after failure, adapted for Minecraft
3. **Auto-GPT**: General-purpose autonomous GPT agent, adapted for Minecraft
4. **DEPS** (Wang et al., 2023): "Describe, Explain, Plan, Select" -- LLM-based planner with RL executor
5. **Voyager w/o skill library**: Ablation without persistent skill storage
6. **Voyager w/o curriculum**: Ablation with random task selection
7. **Voyager w/o self-verification**: Ablation using only hard-coded success checks

### Evaluation Metrics
1. **Unique items obtained**: Number of distinct items the agent obtains (out of ~400 in Minecraft)
2. **Distance traveled**: Total blocks traversed (exploration metric)
3. **Tech tree milestones**: Obtaining wooden tools, stone tools, iron tools, diamond tools
4. **Milestone completion time**: How quickly key milestones are reached
5. **Zero-shot generalization**: Performance on unseen tasks after skill accumulation

### Model
- **GPT-4** (gpt-4-0314) as the backbone LLM for all components
- **text-embedding-ada-002** for skill library indexing

## Results

### Unique Items Obtained (after 160 iterations)

| Method          | Unique Items | Relative to Voyager |
|----------------|-------------|-------------------|
| Voyager         | ~63         | 1.0x              |
| Auto-GPT        | ~19         | 0.30x             |
| ReAct           | ~18         | 0.29x             |
| Reflexion        | ~20         | 0.32x             |
| DEPS            | ~16         | 0.25x             |

Voyager obtains **3.3x more unique items** than the strongest baseline.

### Tech Tree Progression

| Milestone        | Voyager | ReAct | Reflexion | Auto-GPT |
|-----------------|---------|-------|-----------|----------|
| Wooden tools     | ~2 iter | ~5    | ~4        | ~6       |
| Stone tools      | ~5 iter | ~20   | ~18       | ~25      |
| Iron tools       | ~15 iter| ~50+  | ~50+      | ~50+     |
| Diamond tools    | ~40 iter| Fail  | Fail      | Fail     |

Key finding: **Only Voyager unlocks diamond-level tools**. All baselines plateau at iron level or earlier.

### Exploration (Distance Traveled)

Voyager traverses **2.3x longer distances** than baselines, demonstrating more effective exploration driven by the curriculum.

### Ablation Studies

| Variant                  | Unique Items (160 iter) | Notes                           |
|--------------------------|------------------------|---------------------------------|
| Full Voyager             | ~63                    | All components                  |
| w/o Skill Library        | ~42                    | 33% reduction; cannot reuse     |
| w/o Automatic Curriculum | ~35                    | 44% reduction; random goals     |
| w/o Self-Verification    | ~50                    | 20% reduction; noisy storage    |

All three components contribute meaningfully, with the **automatic curriculum** having the largest impact and the **skill library** close behind.

### Zero-Shot Generalization

After pre-training with Voyager's accumulated skill library, the agent was tested on unseen tasks:
- **Novel crafting tasks**: 80%+ success rate on crafting items not explicitly trained on
- **Combat tasks**: Successfully defeated unseen mobs by composing combat + crafting skills
- **Construction tasks**: Could build simple structures by composing block-placement skills

Baselines without skill libraries achieved near-zero performance on these generalization tasks.

### Skill Library Growth

The skill library grows roughly linearly with iterations:
- After 50 iterations: ~30 skills
- After 100 iterations: ~55 skills
- After 160 iterations: ~75 skills

Average skill length: ~15 lines of JavaScript code. Skills range from simple (3 lines for mining a single block) to complex (40+ lines for smelting workflows).

## Analysis & Insights

1. **Code as memory**: The skill library is a form of procedural memory that is perfectly faithful, deterministic, and composable. This is a significant advantage over neural memory systems that suffer from catastrophic forgetting.

2. **World knowledge bootstrapping**: The automatic curriculum works because GPT-4 already knows the Minecraft tech tree, item dependencies, and gameplay progression. This is effectively transferring commonsense and domain knowledge from pre-training into an exploration policy.

3. **Self-verification robustness**: The LLM-based self-verification is surprisingly accurate. It correctly identifies both successes and failures in ~90% of cases, handling nuanced situations like "I mined iron ore but it fell into lava" (failure despite mining action succeeding).

4. **Error feedback is critical**: The iterative refinement loop converges faster when execution errors (stack traces, type errors) are included in the feedback. Most successful skills are generated within 2-3 iterations.

5. **Emergent abstraction**: Without explicit instruction, the agent naturally creates utility functions that are reused across skills. For example, a `findAndGoTo(blockType)` helper emerges and is called by mining, crafting, and exploration skills.

6. **API knowledge matters**: The curated API documentation in the prompt is essential. Without it, GPT-4 generates plausible but incorrect Mineflayer API calls. This highlights the importance of grounding LLM code generation in actual API specifications.

## Limitations & Critiques

1. **GPT-4 dependency**: The entire system relies on GPT-4's world knowledge of Minecraft. For domains where the LLM has less pre-training exposure, the automatic curriculum and code generation quality would likely degrade significantly.

2. **Cost**: Each iteration involves multiple GPT-4 calls (curriculum proposal, code generation, self-verification, potential retries). A 160-iteration run costs hundreds of dollars in API fees.

3. **JavaScript-specific**: The skill library is tied to the Mineflayer JavaScript API. Transferring this approach to other environments requires a well-documented programmatic API, which many environments lack.

4. **No spatial reasoning**: The agent struggles with tasks requiring precise 3D spatial reasoning (e.g., building complex structures) because GPT-4 has limited spatial capabilities.

5. **No multi-agent interaction**: Voyager operates as a single agent. It cannot cooperate with other agents or players, limiting its applicability to multiplayer scenarios.

6. **Fragile skill composition**: When composed skills fail due to environmental changes (e.g., a needed resource is no longer nearby), the error recovery is limited. The agent may retry the same failing approach rather than adapting.

7. **Evaluation fairness**: Baselines like ReAct and Reflexion were designed for different settings (text games, QA) and adapted for Minecraft, potentially underrepresenting their capabilities. DEPS uses RL components that may need more training time.

8. **No continual model improvement**: The LLM itself does not learn from the agent's experiences. All improvement comes from the skill library accumulation, not from updating model weights.

## Follow-up Work

- **GITM** (Zhu et al., 2023): "Ghost in the Minecraft" -- similar LLM-based Minecraft agent with hierarchical goal decomposition and text-based knowledge base.
- **STEVE-1** (Lifshitz et al., 2023): Instruction-following agent in Minecraft using video pre-training, complementary approach (neural policy vs. code generation).
- **Creative Agents** (Zhang et al., 2023): Extended code-as-skill to creative tasks beyond Minecraft.
- **DEPS** (follow-up): Improved RL+LLM hybrid approaches citing Voyager's curriculum design.
- **Skill library concept adopted by**: JARVIS-1 (Wang et al., 2023), CodeAct (Wang et al., 2024), and numerous agent frameworks that store executable procedures.

## Key Takeaways

1. **Code is the ideal skill representation** for LLM agents: it is executable, verifiable, composable, and perfectly reproducible. This insight has influenced the entire field of agent design.
2. **LLM world knowledge can drive exploration**: The automatic curriculum demonstrates that pre-trained knowledge can substitute for reward engineering in open-ended environments.
3. **Iterative refinement with environment feedback** is essential -- single-pass code generation is insufficient for complex tasks. The retry mechanism with error feedback dramatically improves success rates.
4. **Lifelong learning is achievable** with LLM agents through persistent skill storage and retrieval, opening up possibilities for agents that grow more capable over time without retraining.
5. The **skill library pattern** (write, verify, store, retrieve, compose) has become a standard component in modern agent architectures.

## Related

- Skill library concept extends to: coding agents, tool creation
- See: `concepts/agent_memory.md` (procedural memory)
- See: `concepts/tool_use.md` (code as tools)
