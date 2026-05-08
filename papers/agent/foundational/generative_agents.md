---
title: "Generative Agents: Interactive Simulacra of Human Behavior"
authors: Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, Michael S. Bernstein
venue: UIST 2023 (Best Paper Award)
year: 2023
url: https://arxiv.org/abs/2304.03442
tags: [multi-agent, memory, reflection, simulation, cognitive-architecture, foundational]
status: done
---

## TL;DR

Twenty-five LLM-powered generative agents inhabiting a sandbox world ("Smallville") exhibit emergent social behaviors -- forming relationships, spreading information, remembering past interactions, and autonomously coordinating a Valentine's Day party -- all arising from a cognitive architecture built on memory streams, retrieval, reflection, and planning, without any explicit behavioral scripting.

## Motivation & Problem

Prior work on believable agents (the Sims, virtual characters in games, NPCs) relied on hand-authored finite state machines or rule-based behavior trees. These approaches produce brittle, repetitive behavior that fails to generalize across situations. Large language models demonstrate remarkable abilities to generate human-like text, but a single LLM call cannot maintain long-term memory, synthesize experiences into higher-level understanding, or produce coherent behavior over extended time horizons. The central question is: can an architecture built around an LLM produce agents that exhibit believable, emergent social behavior over days of simulated time?

The paper addresses three specific gaps:
1. **No architecture existed** for LLM-based agents that could maintain coherent identity and memory over long time horizons (days/weeks of simulated time).
2. **Emergent social dynamics** (gossip, relationship formation, group coordination) had not been demonstrated with LLM agents.
3. **No evaluation framework** existed to assess the "believability" of generative agent behavior.

## Method

### The Smallville Environment

Smallville is a sandbox environment inspired by The Sims, implemented as a web application. It contains:
- **Locations**: Homes, a cafe (Hobbs Cafe), a park, a college dorm, a pharmacy, a bar, a grocery store
- **25 agents**: Each initialized with a paragraph-length natural language description specifying their identity, occupation, relationships, and personality traits
- **Time**: Runs in simulated time; one full day cycle includes waking, working, socializing, sleeping
- **Interaction**: Agents perceive nearby agents and objects; conversations happen when agents are co-located

The environment is tile-based. Agents navigate between locations and interact with objects (e.g., stove, bed, desk). The environment tree is hierarchical:

```
World
├── The Rose and Crown Pub
│   ├── bar area
│   │   ├── bar stool
│   │   └── beer tap
│   └── dining area
├── Hobbs Cafe
│   ├── main area
│   │   ├── table
│   │   └── counter
│   └── kitchen
├── Johnson Park
│   └── park bench
├── Dorm (Oak Hill College)
│   ├── Room 101 (Mei Lin)
│   │   ├── bed
│   │   ├── desk
│   │   └── bookshelf
│   └── ...
├── Houses
│   ├── Lin Family House
│   ├── Moreno House
│   └── ...
└── Harvey Oak Supply Store
```

### Cognitive Architecture

Each agent operates with four interconnected modules:

```
┌─────────────────────────────────────────────────────┐
│                    AGENT LOOP                        │
│                                                      │
│  Perceive ──> Retrieve ──> Plan/React ──> Act        │
│      │            │             │                    │
│      ▼            ▼             ▼                    │
│  ┌────────┐  ┌──────────┐  ┌──────────┐             │
│  │ Memory │  │Retrieval │  │Planning/ │             │
│  │ Stream │◄─│ Function │  │Reflection│             │
│  │        │──►          │  │          │             │
│  └────────┘  └──────────┘  └──────────┘             │
│                                                      │
└─────────────────────────────────────────────────────┘
```

#### 1. Memory Stream

A temporally ordered list of memory objects, each containing:
- **Description**: natural language description of the observation (e.g., "Klaus Mueller is reading a book on gentrification at the library")
- **Creation timestamp**: when the memory was created
- **Last access timestamp**: when the memory was last retrieved
- **Importance score**: integer 1-10 assigned by the LLM at creation time

The memory stream captures both observations (what the agent sees/hears) and reflections (synthesized higher-level insights). All agent experiences -- perceiving objects, having conversations, internal reflections -- are stored as memory objects.

Prompt for importance scoring:
```
On the scale of 1 to 10, where 1 is purely mundane (e.g., brushing teeth)
and 10 is extremely poignant (e.g., a breakup, college acceptance),
rate the likely poignancy of the following piece of memory.
Memory: [memory description]
Rating: <fill in>
```

#### 2. Retrieval Function

When an agent needs to act, the system retrieves relevant memories using a scoring function that combines three factors:

```
score(memory) = alpha * recency(memory) + beta * importance(memory) + gamma * relevance(memory)
```

Where (with alpha = beta = gamma = 1 in practice):

- **Recency**: Exponential decay function based on hours since last access
  ```
  recency(m) = decay_factor ^ (hours_since_last_access)
  ```
  where `decay_factor = 0.99`. Recent memories score close to 1; old memories decay toward 0.

- **Importance**: The pre-assigned importance score, normalized to [0, 1] by dividing by 10.

- **Relevance**: Cosine similarity between the embedding of the query/current situation and the embedding of the memory description. Uses OpenAI's text-embedding-ada-002.

The top-k memories (typically k varies based on context window budget) are returned and included in the LLM prompt.

#### 3. Reflection

Reflections are higher-level, abstract thoughts synthesized periodically from accumulated memories. The reflection mechanism triggers when the sum of importance scores of recent memories exceeds a threshold (empirically set to 150).

Reflection generation is a two-step process:

**Step 1 -- Generate questions**: Given the 100 most recent memories, the LLM generates 3 high-level questions that can be answered from those memories.
```
Prompt: Given only the information above, what are 3 most salient
high-level questions we can answer about the subjects in the statements?
```

**Step 2 -- Generate reflections**: For each question, retrieve relevant memories and ask the LLM to synthesize insights.
```
Prompt: Statements about [agent name]:
[retrieved memories]
What 5 high-level insights can you infer from the above statements?
(example format: insight (because of 1, 5, 3))
```

Reflections are stored back into the memory stream and can themselves be retrieved and reflected upon, creating a hierarchy of increasingly abstract insights. Reflections cite their source memories (the "because of" pointers), forming a tree structure.

#### 4. Planning and Reacting

**Planning** operates at multiple temporal granularities:

1. **Daily plan**: At the start of each day, the agent generates a high-level plan (e.g., "wake up at 7am, eat breakfast, go to work at the pharmacy, have lunch, continue work, go home, read, sleep").
2. **Hourly decomposition**: Each high-level plan item is recursively decomposed into finer-grained actions (5-15 minute blocks).
3. **Reaction**: When an agent encounters another agent or a significant environmental change, the system decides whether to continue the current plan or react. This decision is made by the LLM given the agent's personality, current activity, and relationship with the encountered entity.

**Conversation generation**: When two agents decide to converse, a dialogue is generated turn-by-turn. Each agent's utterance is conditioned on their persona, retrieved memories about the other agent, and the current conversation context.

**Plan revision**: After significant events (especially conversations), agents may revise their remaining daily plans to incorporate new information.

### Agent Initialization

Each agent is bootstrapped with a seed memory paragraph, for example:
```
John Lin is a pharmacy owner who has been running the local pharmacy
for 20 years. He is married to Mei Lin, a retired professor. They
have a son, Eddy Lin, who is a music composition student. John is
known for being helpful and friendly to everyone in the community.
```

This seed is decomposed into individual memory entries and loaded into the memory stream at initialization.

## Key Innovations

1. **Memory stream architecture**: First comprehensive architecture combining observation logging, importance scoring, temporally-weighted retrieval, and hierarchical reflection for LLM agents.
2. **Reflection mechanism**: Allowing agents to form higher-level abstractions from raw observations, enabling long-horizon coherent behavior.
3. **Emergent social phenomena**: Demonstrated that complex group dynamics (party planning, information diffusion, opinion formation) arise without any explicit coordination mechanisms.
4. **Evaluation methodology**: Introduced a controlled human evaluation protocol for agent believability using interview-based assessment.

## Experimental Setup

### Evaluation Protocol

Two types of evaluation:

**1. Controlled evaluation (individual agent believability)**:
- Participants (n = 100, via Amazon Mechanical Turk) assessed agent responses in interview scenarios
- Five competency areas tested:
  - **Self-knowledge**: Does the agent know facts about itself?
  - **Memory**: Does it recall past events?
  - **Planning**: Are plans coherent and reasonable?
  - **Reactions**: Are responses to unexpected situations appropriate?
  - **Reflection**: Does the agent show higher-level understanding?
- Methodology: Participants rated believability of agent responses vs. human-authored baseline responses on a Likert scale

**2. End-to-end emergent behavior evaluation**:
- Observed the simulation running for two simulated days
- Tracked specific seeded information (e.g., one agent's Valentine's Day party plan, another agent's mayoral candidacy)
- Measured information diffusion, relationship changes, coordination behavior

### Ablation Conditions

Five conditions tested:
1. **Full architecture** (memory + retrieval + reflection + planning)
2. **No observation** (agent has no memory stream)
3. **No retrieval** (uses full memory but no selective retrieval)
4. **No reflection** (no higher-level synthesis)
5. **No planning** (reactive only, no daily plans)

### Models Used
- **GPT-3.5-turbo** for most agent interactions
- **GPT-4** mentioned for comparison but primarily GPT-3.5

## Results

### Believability Scores (Likert 1-10 scale)

| Condition          | Self-Knowledge | Memory | Planning | Reaction | Reflection | Overall |
|-------------------|---------------|--------|----------|----------|------------|---------|
| Full Architecture  | High          | High   | High     | High     | High       | ~4.5    |
| No Observation     | Low           | Low    | Low      | Low      | Low        | ~1.5    |
| No Retrieval       | Mid           | Low    | Mid      | Low      | Low        | ~2.5    |
| No Reflection      | Mid           | Mid    | Mid      | Mid      | Low        | ~3.5    |
| No Planning        | Mid           | Mid    | Low      | Mid      | Mid        | ~3.0    |
| Human baseline     | -             | -      | -        | -        | -          | ~4.1    |

Key finding: the full architecture outperformed the human-authored baseline in believability, demonstrating that generative agents can be more believable than hand-crafted NPCs.

### Ablation Analysis

- **Reflection** is the most critical component for emergent social behavior. Without reflection, agents failed to synthesize experiences into relationship understanding, leading to shallow interactions.
- **Retrieval** is essential for maintaining coherent long-term behavior. Without selective retrieval, agents would reference irrelevant memories and lose contextual coherence.
- **Planning** enables proactive behavior. Without planning, agents were purely reactive and produced disjointed activity sequences.

### Emergent Behaviors Observed

1. **Valentine's Day Party**: Isabella Rodriguez mentioned wanting to throw a Valentine's Day party. Over two simulated days, this information spread through conversations. Agents autonomously coordinated: invited friends, decorated the cafe, showed up at the planned time. 12 agents were aware by end of day 2; 5 attended the party. No explicit coordination was scripted.
2. **Information Diffusion**: Sam Moore's mayoral candidacy spread from 4 agents (who were directly told) to 8 agents by end of simulation through organic conversation.
3. **Relationship Formation**: Agents developed new relationships based on shared interests discovered through conversation (e.g., two agents bonded over shared interest in research).

### Cost and Compute Analysis

- **Cost**: Running the 25-agent simulation for two game days cost approximately **$1,000 in API fees** (using GPT-3.5-turbo pricing at the time, ~$0.002/1K tokens).
- **Token usage**: Thousands of LLM calls per simulated day. Each agent makes dozens of calls for perception, retrieval, planning, and conversation.
- **Latency**: The simulation ran slower than real-time due to sequential API calls. The authors note that parallelization and caching could improve performance.
- **Memory growth**: The memory stream grows unboundedly; no explicit garbage collection. After two days, agents had hundreds of memories each.

## Analysis & Insights

1. **Architecture simplicity**: The architecture is remarkably simple -- three scoring functions for retrieval, periodic reflection triggers, hierarchical planning -- yet produces complex emergent behavior. This suggests that much of the complexity in agent behavior comes from the LLM's world knowledge rather than architectural sophistication.

2. **Reflection as compression**: Reflections serve as a form of lossy memory compression. Rather than retrieving dozens of raw observations, an agent can retrieve a single reflection that captures the essence of many interactions. This is analogous to how human episodic memory consolidates into semantic memory.

3. **Memory retrieval as attention**: The retrieval function serves a role analogous to attention in neural networks -- it determines which past experiences are relevant to the current situation. The three-factor scoring (recency, importance, relevance) provides a well-motivated prior over memory salience.

4. **Social simulation validity**: The emergent behaviors (information cascading, group coordination, relationship dynamics) mirror patterns studied in computational social science, suggesting generative agents could serve as tools for social science research.

5. **Prompt engineering load**: The system relies heavily on carefully crafted prompts for each module. The prompts encode significant domain knowledge about what constitutes good planning, appropriate social behavior, etc. The architecture is thus not fully "emergent" -- the prompts embed strong inductive biases.

## Limitations & Critiques

1. **Cost prohibitive**: ~$1,000 for two simulated days of 25 agents makes large-scale or long-duration studies impractical. Scaling to hundreds of agents or weeks of simulation would be orders of magnitude more expensive.

2. **No grounded perception**: Agents perceive via text descriptions of the environment, not actual sensory input. This sidesteps the perception problem and limits applicability to text-only simulations.

3. **Hallucination and fabrication**: Agents sometimes confabulate memories or relationships that were never established. The paper acknowledges "embellishments" in agent behavior that are plausible but never occurred.

4. **Bounded evaluation**: The evaluation covers only two simulated days with 25 agents. It is unclear whether agent behavior remains coherent over weeks or months, or whether the memory stream becomes unmanageable.

5. **No adversarial testing**: The paper does not explore how agents behave under adversarial conditions (e.g., deliberate misinformation injection, conflicting instructions).

6. **Retrieval bottleneck**: The fixed retrieval formula (alpha=beta=gamma=1) was not systematically tuned. Different tasks or agent types might benefit from different weightings.

7. **No learning or adaptation**: Agents do not improve their strategies over time. They can accumulate memories and reflections, but their underlying "reasoning" does not get better -- this is bounded by the LLM's fixed capabilities.

8. **Reproducibility concerns**: Due to LLM stochasticity and API dependence, exact reproduction of the emergent behaviors is not guaranteed.

## Follow-up Work

- **MemGPT** (Packer et al., 2023): Formalized LLM memory management as a virtual memory system with explicit paging, directly inspired by the memory stream concept.
- **CoALA** (Sumers et al., 2023): Proposed a formal cognitive architecture framework for language agents, abstracting the memory/reasoning/action loop introduced here.
- **AgentSims** (Lin et al., 2023): Extended multi-agent simulation with more complex economic and social dynamics.
- **CAMEL** (Li et al., 2023): Multi-agent communication framework for task solving, using role-playing agents.
- **Generative Agents for Social Science** (various 2023-2024): Multiple papers used generative agents to simulate surveys, elections, and social dynamics.
- **Project Sid** (Altera, 2024): Scaled generative agents to 1000+ agents in Minecraft, demonstrating emergent governance and economic systems.

## Key Takeaways

1. The **memory stream + retrieval + reflection** architecture is a foundational design pattern for LLM agents. Nearly every subsequent agent framework incorporates some variant of this memory architecture.
2. **Emergent behavior from simple rules** is possible with LLM agents -- complex social dynamics arise from individual agents following straightforward perceive-retrieve-plan-act loops.
3. The **retrieval scoring formula** (recency x importance x relevance) is a principled and influential approach to memory management that balances multiple factors.
4. **Reflection enables hierarchical abstraction** -- without it, agents remain trapped in surface-level observations and fail to develop deeper understanding of their social world.
5. The **cost of simulation** remains a fundamental barrier. The paper demonstrates feasibility but not scalability.
6. The evaluation methodology -- having humans assess believability through structured interviews -- set a standard for evaluating agent behavior that goes beyond task completion metrics.

## Related

- Memory architecture influenced: MemGPT, CoALA, modern agent memory systems
- See: `concepts/agent_memory.md`
- See: `concepts/multi_agent.md`
