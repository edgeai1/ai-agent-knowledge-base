---
title: "A Survey on Large Language Model based Autonomous Agents"
authors:
  - Lei Wang
  - Chen Ma
  - Xueyang Feng
  - Zeyu Zhang
  - Hao Yang
  - Jingsen Zhang
  - Zhiyuan Chen
  - Jiakai Tang
  - Xu Chen
  - Yankai Lin
  - Wayne Xin Zhao
  - Zhewei Wei
  - Ji-Rong Wen
venue: Frontiers of Computer Science (first published survey in LLM-based agents)
year: 2023
url: https://arxiv.org/abs/2308.11432
tags:
  - llm-agents
  - survey
  - autonomous-agents
  - agent-architecture
  - profiling
  - memory
  - planning
  - action
status: done
---

# A Survey on Large Language Model based Autonomous Agents

## TL;DR

This landmark survey from Fudan/Renmin NLP groups proposes a unified four-module framework for constructing LLM-based autonomous agents: **profiling**, **memory**, **planning**, and **action**. It provides the first comprehensive taxonomy of agent architectures, catalogs application domains (social simulation, natural science, engineering), and evaluates existing approaches across subjective and objective dimensions. Accepted by Frontiers of Computer Science, it is widely regarded as the foundational survey in the LLM-agent field.

---

## Motivation & Problem

Prior to this survey, the landscape of LLM-based agents was fragmented. Researchers were building agent systems (AutoGPT, BabyAGI, HuggingGPT, Voyager, Generative Agents, etc.) without a shared vocabulary or architectural reference. The paper identifies three core gaps:

1. **No unified framework** -- diverse agent designs lacked a common structural decomposition, making comparison difficult.
2. **Scattered application domains** -- agents were deployed in social simulation, scientific discovery, and software engineering, but no work had mapped these systematically.
3. **Evaluation vacuum** -- no consensus on how to assess agent capability, safety, or robustness.

The authors aim to provide a "comprehensive and systematic" survey that can serve as both a reference and a roadmap for the field.

---

## Method / Framework: The Four-Module Agent Architecture

### 1. Profiling Module

The profiling module determines the agent's role, persona, and behavioral characteristics. Three strategies are identified:

- **Handcrafting**: Manually specifying agent profiles with demographic details, personality traits, and domain expertise. Example: Generative Agents manually assign occupations, relationships, and daily routines to 25 agents in a sandbox town.
- **LLM-Generation**: Using the LLM itself to generate diverse profiles. Example: RecAgent creates seed profiles by hand and then prompts ChatGPT to synthesize additional profiles with varied backgrounds, expanding the agent population efficiently.
- **Dataset-Alignment**: Deriving profiles from real-world datasets. Example: using census data or social network profiles to ground agent personas in empirically observed distributions.

The profiling module answers "who is this agent?" and conditions all downstream behavior -- memory encoding, planning style, and action preferences.

### 2. Memory Module

The memory module provides agents with the ability to store, retrieve, and reason over past experiences. The survey decomposes memory along two axes:

**By temporal scope:**
- **Short-term memory**: Implemented via the LLM's finite context window. Captures the immediate interaction history and recent observations. Inherently limited by context length, creating a recency bias.
- **Long-term memory**: Implemented via external storage (vector databases, structured knowledge bases, or text files). Allows retention of experiences across sessions and beyond context-window limits.

**By representational format:**
- **Natural language**: Storing raw or summarized text observations (e.g., Generative Agents' memory stream of timestamped natural-language records).
- **Embeddings**: Dense vector representations stored in vector databases for similarity-based retrieval (e.g., using FAISS or Pinecone).
- **Databases**: Structured storage in relational or graph databases for precise querying.
- **Structured lists**: Maintaining organized inventories, skill libraries, or action histories (e.g., Voyager's skill library stored as executable code snippets).

**Memory operations:**
- **Reading (Retrieval)**: Accessing relevant memories, typically via recency, relevance (embedding similarity), and importance scoring. Generative Agents combine all three factors in a weighted retrieval function.
- **Writing (Storage)**: Encoding new observations. Key challenge: deciding what to store versus discard.
- **Reflection**: Higher-order synthesis where agents generate abstract insights from accumulated memories (e.g., "I tend to enjoy conversations about art" derived from multiple specific memory entries).

### 3. Planning Module

The planning module enables agents to decompose complex tasks and formulate action sequences. The survey distinguishes two paradigms:

**Planning without feedback:**
- **Single-path reasoning**: Chain-of-Thought (CoT) prompting, Zero-shot CoT ("Let's think step by step"), producing a linear sequence of reasoning steps.
- **Multi-path reasoning**: Tree-of-Thoughts (ToT), which explores multiple reasoning branches and uses search algorithms (BFS/DFS) to select the best path. Self-Consistent CoT samples multiple reasoning chains and takes a majority vote.
- **External planner integration**: Offloading planning to classical AI planners (e.g., PDDL-based planners) that guarantee optimality in well-defined domains, then translating plans back to natural-language actions.

**Planning with feedback:**
- **Environmental feedback**: The agent observes outcomes of its actions in the environment and adjusts plans accordingly. Example: ReAct interleaves reasoning and acting, using environment observations to guide next steps.
- **Human feedback**: Humans provide corrections, preferences, or guidance during execution. Example: Inner Monologue integrates human verbal feedback into the agent's planning loop.
- **Model feedback**: The LLM critiques its own plans or another LLM provides evaluation. Example: Reflexion stores linguistic feedback from failed attempts in an episodic memory buffer, enabling the agent to avoid repeating mistakes.

### 4. Action Module

The action module translates plans into concrete operations. Three dimensions characterize actions:

**Action targets:**
- **Task completion**: Direct actions to accomplish assigned goals (answering questions, generating code, producing artifacts).
- **Communication**: Engaging with other agents or humans via natural language dialogue.
- **Environment interaction**: Manipulating external environments (APIs, browsers, operating systems, physical robotics).

**Action production strategies:**
- **Memory-based**: Retrieving previously successful action sequences from long-term memory.
- **Plan-following**: Executing steps from a pre-generated plan.
- **Interactive generation**: Producing actions in real-time based on current observations, using ReAct-style interleaving.

**Action space:**
- **External tools**: Search engines (Google, Bing), calculators, code interpreters, APIs (Wolfram Alpha, weather services). HuggingGPT connects to Hugging Face model zoo; ToolFormer learns when and how to call APIs.
- **Internal knowledge**: Relying on the LLM's parametric knowledge to generate responses without external tools.
- **Embodied actions**: Physical or simulated-world actions (navigation, object manipulation). Voyager in Minecraft uses a code-based action space.

---

## Key Contributions

1. **First unified architectural framework** decomposing LLM agents into four orthogonal modules (profiling, memory, planning, action), enabling systematic comparison across diverse systems.
2. **Comprehensive taxonomy** of sub-approaches within each module, with concrete examples from published systems.
3. **Application domain mapping** covering three major areas:
   - **Social simulation**: Generative Agents (Stanford smallville), Social Simulacra, AgentSims -- studying emergent social phenomena, opinion dynamics, and collective behavior.
   - **Natural science**: ChemCrow (chemistry), Boiko et al. (autonomous lab experiments), protein design agents -- accelerating scientific discovery.
   - **Engineering**: MetaGPT, ChatDev (software development), AutoGPT (general-purpose task automation), data analysis agents.
4. **Evaluation framework** analyzing both **subjective** (human evaluation of agent behavior quality, coherence, believability) and **objective** (task success rate, efficiency metrics, benchmark performance) assessment approaches.
5. **Community resource**: the companion GitHub repository (Paitesanshi/LLM-Agent-Survey) maintaining an updated paper list that has become a standard reference.

---

## Coverage / Scope

### Application Domains in Detail

**Social Simulation:**
- Simulating human-like behavior in sandbox environments to study emergent social dynamics.
- Generative Agents: 25 agents in a virtual town exhibiting information diffusion, relationship formation, and coordination for events (e.g., spontaneously organizing a Valentine's Day party).
- S^3 (Social network Simulation System): studying information/opinion propagation at scale.
- Uses: policy testing, social science hypothesis generation, understanding collective behavior.

**Natural Science Discovery:**
- ChemCrow: 18 expert-designed chemistry tools integrated with an LLM for tasks like drug discovery, materials design, and reaction planning.
- Boiko et al.: fully autonomous agents performing wet-lab experiments including internet search, documentation reading, code execution, and robotic lab equipment control.
- Coscientist: multi-agent system for autonomous scientific research.

**Engineering Applications:**
- ChatDev and MetaGPT: multi-agent software development where agents assume roles (CEO, CTO, programmer, tester) and collaborate through structured communication protocols.
- Voyager: lifelong learning agent in Minecraft that autonomously explores, acquires skills, and builds a reusable skill library.
- AutoGPT/BabyAGI: general-purpose agents that recursively create and execute task lists.

### Evaluation Approaches

**Subjective evaluation:**
- Human ratings of believability, consistency, naturalness of agent behavior.
- Turing-test-style assessments where evaluators distinguish agent-generated from human-generated behavior.
- Expert evaluation in domain-specific tasks (chemical synthesis feasibility, code correctness).

**Objective evaluation:**
- Task success rates on standardized benchmarks (ALFWorld, WebShop, HotPotQA).
- Efficiency metrics (number of steps, API calls, tokens consumed).
- Multi-dimensional metrics including safety (harmful action avoidance), robustness (performance under perturbation), and calibration (confidence vs. accuracy).

---

## Limitations

1. **Static taxonomy**: The four-module decomposition, while useful, may not capture emergent architectural patterns (e.g., agents that blur the line between planning and action, or systems with no explicit profiling).
2. **LLM-centric bias**: The survey focuses exclusively on LLM-based agents, potentially missing insights from classical agent architectures (BDI, reactive agents) that could inform hybrid designs.
3. **Limited quantitative comparison**: The survey is primarily descriptive rather than empirical -- it catalogs approaches but does not run controlled experiments comparing them.
4. **Rapid obsolescence risk**: Given the pace of the field (the paper covers work through mid-2023, with updates through early 2025), specific system comparisons may become outdated quickly.
5. **Underexplored safety/alignment**: While evaluation is discussed, the survey gives relatively light treatment to adversarial robustness, prompt injection risks, and value alignment -- topics that have become critical since publication.
6. **Scalability questions**: The survey does not deeply address how these architectures scale to long-horizon tasks with hundreds of steps or multi-day execution timelines.

---

## Key Takeaways

1. **The four-module framework (profiling, memory, planning, action) has become the de facto reference architecture** for discussing and designing LLM-based agents. Most subsequent surveys (including Xi et al. 2023) build on or respond to this decomposition.
2. **Memory is the critical differentiator**: Agents with well-designed memory systems (combining short-term context, long-term storage, and reflection) consistently outperform those without, especially on multi-step and multi-session tasks.
3. **Planning with feedback dominates planning without feedback** for complex real-world tasks. The ReAct and Reflexion paradigms (environment and model feedback respectively) have become standard patterns.
4. **The tool-use action space is the primary extensibility mechanism**: An agent's capability is largely determined by the tools it can access and invoke correctly.
5. **Social simulation emerged as a surprisingly rich application domain**, demonstrating that LLM agents can exhibit emergent collective behaviors not explicitly programmed -- a finding with implications for both social science and AI safety.
6. **Evaluation remains the weakest link**: The field lacks standardized, comprehensive benchmarks that jointly assess capability, safety, efficiency, and robustness.
7. **This survey's influence**: With 2000+ citations, it established the shared vocabulary (profiling/memory/planning/action modules) that the field continues to use, making it essential reading for anyone entering the LLM agent space.

---

## Relationship to Other Surveys

- **Complements Xi et al. (2309.07864)**: The Fudan survey focuses on *construction* (how to build agents), while Xi et al. emphasizes *application* (what agents can do) and *society* (how agents interact). Together they provide comprehensive coverage of the 2023 landscape.
- **Extended by Ferrag et al. (2504.19678)**: The 2025 review builds on this taxonomy while adding benchmark comparison and inter-agent protocols (MCP, A2A).
- **Memory depth provided by Du (2603.07670)**: The memory module discussed here is expanded into a full treatment in the 2026 memory survey.
