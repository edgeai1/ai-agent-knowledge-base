---
title: "The Rise and Potential of Large Language Model Based Agents: A Survey"
authors:
  - Zhiheng Xi
  - Wenxiang Chen
  - Xin Guo
  - Wei He
  - Yiwen Ding
  - Boyang Hong
  - Ming Zhang
  - Junzhe Wang
  - Senjie Jin
  - Enyu Zhou
  - Rui Zheng
  - Xiaoran Fan
  - Xiao Wang
  - Limao Xiong
  - Yuhao Zhou
  - Weiran Wang
  - Changhao Jiang
  - Yicheng Zou
  - Xiangyang Liu
  - Zhangyue Yin
  - Shihan Dou
  - Rongxiang Weng
  - Wensen Cheng
  - Qi Zhang
  - Wenjuan Qin
  - Yongyan Zheng
  - Xipeng Qiu
  - Xuanjing Huang
  - Tao Gui
venue: Science China Information Sciences (86-page cover article)
year: 2023
url: https://arxiv.org/abs/2309.07864
tags:
  - llm-agents
  - survey
  - brain-perception-action
  - multi-agent
  - human-agent-cooperation
  - agent-society
status: done
---

# The Rise and Potential of Large Language Model Based Agents: A Survey

## TL;DR

This 86-page survey from the Renmin/Fudan NLP group presents an alternative agent framework centered on the **brain-perception-action** triad, inspired by cognitive science. It uniquely organizes the agent landscape into three interaction paradigms -- single-agent, multi-agent, and human-agent cooperation -- and includes a deep philosophical treatment of the agent concept tracing from Aristotle through Turing to modern LLMs. Published as the cover article of Science China Information Sciences, it provides the most comprehensive treatment of agent *applications* and *social phenomena* in the 2023 literature.

---

## Motivation & Problem

The authors identify several motivating observations:

1. **Philosophical gap**: The concept of "agent" spans philosophy, cognitive science, and computer science, but the AI community had not adequately situated LLM-based agents within this intellectual lineage.
2. **Why LLMs as agents?**: Previous agent paradigms (symbolic AI, reinforcement learning agents) had limited natural language understanding and generalization. LLMs offer emergent capabilities (in-context learning, instruction following, reasoning) that make them uniquely suited as the "brain" of autonomous agents.
3. **Application fragmentation**: Agent applications in task solving, social simulation, gaming, and scientific discovery needed systematic categorization.
4. **Social phenomena**: As agents interact in multi-agent settings, emergent social behaviors arise -- these needed documentation and analysis.

---

## Method / Framework: Brain-Perception-Action

### The Brain Module

The brain is the LLM itself, serving as the central controller. The survey decomposes brain capabilities into:

**Natural Language Interaction:**
- High-quality text generation enabling communication with humans and other agents.
- Multi-turn dialogue management maintaining coherence across extended interactions.
- Role-playing and persona adoption guided by system prompts.

**Knowledge:**
- **Linguistic knowledge**: Grammar, semantics, pragmatics internalized during pre-training.
- **Commonsense knowledge**: Everyday reasoning about physical causality, social norms, temporal relations.
- **Professional/domain knowledge**: Specialized expertise in law, medicine, coding, science (varying in reliability and depth).
- Knowledge limitations: hallucination, outdated information, knowledge boundary uncertainty.

**Memory:**
- **Raise-then-use paradigm**: Memories are first raised (recalled/retrieved) and then used in reasoning.
- **Memory formats**: natural language, code/structured data, embeddings.
- **Memory sources**: experiential memory from interaction histories, knowledge-base memory from external sources.
- Distinction between in-context memory (within the prompt) and external memory (retrieved from databases).

**Reasoning and Planning:**
- **Reasoning**: CoT, self-consistency, analogical reasoning, causal reasoning, counterfactual reasoning.
- **Planning**: task decomposition (Plan-and-Solve), hierarchical planning, plan verification, adaptive re-planning.
- Key distinction between *deliberative* planning (full plan before execution) and *reactive* planning (plan-as-you-go with environmental feedback).

**Transferability and Generalization:**
- Unseen task generalization via in-context learning and instruction tuning.
- Cross-domain transfer without architectural changes.
- Zero-shot and few-shot task adaptation.

### The Perception Module

The perception module handles multimodal input processing, extending agents beyond text:

- **Textual input**: Primary modality; task descriptions, user instructions, document content.
- **Visual input**: Image understanding via vision-language models (GPT-4V, LLaVA). Enables agents to interpret screenshots, diagrams, real-world scenes.
- **Auditory input**: Speech recognition and audio understanding (Whisper integration). Enables voice-controlled agents.
- **Other modalities**: Tactile data, sensor readings, 3D point clouds for embodied agents.
- **Multimodal fusion**: Combining multiple input streams for richer situational awareness. Crucial for robotics and embodied AI applications.

### The Action Module

The action module governs the agent's outputs and interactions with the external world:

- **Textual output**: Natural language responses, reports, summaries, code generation.
- **Tool use**: API calls, database queries, web searches, calculator operations. Formalized as the agent selecting from an available tool inventory based on the current task state.
- **Embodied actions**: Physical movement, object manipulation, navigation in simulated or real environments.
- **Communication actions**: Sending messages to other agents or humans in collaborative settings.
- **Action feedback loops**: The consequences of actions feed back into perception, creating the observe-think-act cycle.

---

## Three Interaction Paradigms

### Paradigm 1: Single-Agent Scenarios

Applications where a lone agent operates autonomously:

**Task-oriented deployment:**
- **Web agents**: Navigating websites, filling forms, making purchases (WebGPT, WebAgent, Mind2Web).
- **Coding agents**: Writing, debugging, and testing code (Code Interpreter, Codex-based systems).
- **Life assistants**: Managing schedules, emails, personal tasks.

**Innovation-oriented deployment:**
- Scientific hypothesis generation and experimental design.
- Creative writing, art generation, and content creation.
- Mathematical theorem proving and problem solving.

**Lifecycle in simulation:**
- Agents placed in persistent environments with long-horizon goals.
- Voyager in Minecraft: autonomous exploration, skill acquisition, and curriculum-driven self-improvement.
- Generative Agents: daily routines, relationship building, memory-driven behavioral evolution.

### Paradigm 2: Multi-Agent Scenarios

Settings where multiple agents collaborate, compete, or coexist:

**Cooperative interaction:**
- **Complementary cooperation**: Agents with different specializations work together (e.g., ChatDev's CEO-CTO-programmer-tester pipeline). Each agent contributes unique expertise to a shared goal.
- **Debate and discussion**: Agents engage in structured argumentation to improve solution quality. ChatEval uses multi-agent debate for text evaluation, achieving better calibration than single-agent scoring.
- **Collaborative reasoning**: Multiple agents share intermediate reasoning steps, catching errors the others miss. Improves accuracy on complex reasoning tasks compared to single-agent approaches.

**Adversarial interaction:**
- **Red-teaming**: Adversarial agents probe for vulnerabilities in target agents.
- **Negotiation**: Agents with conflicting objectives negotiate outcomes (price bargaining, resource allocation).
- **Competitive games**: Strategic gameplay in environments like Diplomacy, Werewolf, Avalon.

**Communication topologies:**
- **Layered**: Hierarchical communication with manager agents and worker agents.
- **Decentralized**: Peer-to-peer communication without central coordination.
- **Shared message pool**: All agents read from and write to a common communication channel.
- **Dynamic**: Communication structure evolves based on task requirements.

### Paradigm 3: Human-Agent Cooperation

Settings integrating human expertise with agent capabilities:

- **Instructor-executor**: Human provides high-level goals; agent decomposes and executes. Human may intervene for corrections.
- **Equal partnership**: Human and agent collaborate as peers, each contributing strengths (human judgment and world knowledge; agent speed and tirelessness).
- **Agent-as-mentor**: Agent guides humans through learning, problem-solving, or decision-making (educational tutoring, medical decision support).
- **Evaluation-in-the-loop**: Human evaluators continuously assess agent outputs, providing feedback that shapes subsequent behavior.

---

## Key Contributions

1. **Cognitive-science-grounded framework**: The brain-perception-action triad maps more naturally to cognitive science concepts than the Fudan survey's engineering-oriented four-module decomposition, offering complementary analytical power.
2. **Most comprehensive application coverage** in the 2023 literature, spanning web navigation, coding, scientific discovery, gaming, social simulation, education, and healthcare.
3. **Multi-agent and social analysis**: First survey to systematically categorize multi-agent interaction patterns (cooperative, adversarial) and communication topologies in the LLM agent context.
4. **Agent society analysis**: Documents emergent social phenomena in multi-agent simulations -- opinion formation, information cascading, trust dynamics, social norm emergence.
5. **Philosophical and historical context**: Traces the agent concept from Aristotle's notion of agency through Turing's computational agents to modern LLM-based systems, providing intellectual grounding for the field.
6. **Scale**: At 86 pages with 600+ references, it remains one of the most thorough treatments of LLM agents published.

---

## How This Survey Complements the Fudan Survey

| Dimension | Fudan (Wang et al.) | Renmin (Xi et al.) |
|-----------|--------------------|--------------------|
| **Framework** | Engineering: profiling + memory + planning + action | Cognitive: brain + perception + action |
| **Focus** | Agent *construction* -- how to build | Agent *application* -- what agents do |
| **Memory treatment** | Deep (short-term/long-term, formats, operations) | Embedded within brain module |
| **Perception** | Not a separate module | Explicit multimodal perception module |
| **Multi-agent** | Brief coverage | Extensive: cooperation, adversarial, topologies |
| **Human-agent** | Limited | Full paradigm with multiple interaction modes |
| **Social phenomena** | Mentioned | Dedicated analysis of emergent behaviors |
| **Philosophical roots** | Minimal | Extensive historical tracing |
| **Best used for** | Designing agent architectures | Understanding agent capabilities and ecosystems |

**Recommendation**: Read both. The Fudan survey provides the structural vocabulary (profiling/memory/planning/action) for designing agents, while this survey provides the application landscape and interaction paradigm analysis for deploying them.

---

## Limitations

1. **Perception module underspecified**: While identified as a first-class component, the perception module receives less detailed treatment than the brain and action modules, with limited discussion of how multimodal perception actually works in practice.
2. **Quantitative evaluation sparse**: Like the Fudan survey, this work is primarily qualitative and descriptive rather than benchmark-driven.
3. **Temporal scope**: Coverage extends to September 2023; the rapid pace of the field means many important systems (GPT-4 Turbo agents, Claude-based agents, open-source agent frameworks like CrewAI) postdate the survey.
4. **Multi-agent scalability**: The survey documents multi-agent patterns but does not deeply explore scalability challenges -- what happens with 100 or 1000 interacting agents versus 2-10.
5. **Safety and alignment**: Adversarial interaction is discussed from a functional perspective but not from a safety/alignment perspective (prompt injection, jailbreaking, value drift in agent societies).
6. **Reproducibility**: Many described systems lack open-source implementations or standardized evaluation, making it difficult to verify claims or compare approaches.

---

## Key Takeaways

1. **Brain-perception-action provides a cognitively intuitive decomposition** that is especially useful when designing multimodal or embodied agents where perception is a first-class concern.
2. **Multi-agent interaction is not merely a technical extension but a qualitatively different regime** -- emergent social phenomena, communication topology choices, and adversarial dynamics introduce challenges absent from single-agent settings.
3. **Human-agent cooperation deserves its own paradigm** rather than being treated as a special case of single-agent operation. The instructor-executor and equal-partnership modes have different implications for system design.
4. **LLMs are suitable agent brains because of four properties**: broad knowledge, language-based reasoning, in-context learning for fast adaptation, and instruction-following for controllability.
5. **The survey's emphasis on agent societies anticipates important future work** in multi-agent simulation, computational social science, and understanding emergent behaviors in agent populations.
6. **Together with the Fudan survey, these two papers defined the 2023 conceptual framework** that all subsequent LLM agent research builds upon or reacts against.
7. **The debate mechanism (multi-agent argumentation) is highlighted as particularly effective** for improving reasoning quality, foreshadowing the "society of minds" approach that gained prominence in 2024-2025.
