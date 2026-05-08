---
title: LLM Agent Survey Papers - Annotated Collection
authors: Various
venue: Various (arXiv, NeurIPS, ICLR, ACL)
year: 2023-2025
tags: [agent, survey, llm, taxonomy, comprehensive]
status: reference
---

## TL;DR

A curated collection of the most influential survey papers on LLM-based agents, covering taxonomies, architectures, capabilities, applications, evaluation, and open challenges.

---

## Survey 1: "The Rise and Potential of Large Language Model Based Agents: A Survey"

- **Authors**: Zhiheng Xi, Wenxiang Chen, Xin Guo, et al.
- **Affiliation**: Fudan University
- **arXiv**: 2309.07864
- **Date**: September 2023 (updated 2024)
- **Pages**: ~86 pages
- **Citations**: One of the most cited agent surveys

### Taxonomy
The paper proposes a three-module agent framework:
1. **Brain (LLM)**: Natural language interaction, knowledge, reasoning, planning, transferability
2. **Perception**: Textual, visual, auditory input processing
3. **Action**: Tool use, embodied actions, communication

### Key Contributions
- Comprehensive taxonomy of agent capabilities
- Detailed analysis of single-agent vs. multi-agent systems
- Application domains: social science, natural science, engineering
- Agent society analysis: cooperation, competition, social norms
- Discussion of safety and ethical considerations

### Agent Capability Taxonomy
```
Agent
├── Brain (LLM)
│   ├── Natural Language Interaction
│   ├── Knowledge (linguistic, commonsense, professional, domain)
│   ├── Memory (in-context, external, long-term)
│   ├── Reasoning (inductive, deductive, abductive)
│   └── Planning (w/o feedback, w/ env feedback, w/ human feedback)
├── Perception
│   ├── Textual Input
│   ├── Visual Input
│   └── Auditory Input
└── Action
    ├── Tool Use
    ├── Embodied Action
    └── Communication / Output
```

---

## Survey 2: "A Survey on Large Language Model based Autonomous Agents"

- **Authors**: Lei Wang, Chen Ma, Xueyang Feng, et al.
- **Affiliation**: Renmin University of China, Tencent
- **arXiv**: 2308.11432
- **Date**: August 2023 (updated 2024)
- **Pages**: ~45 pages

### Taxonomy
The paper organizes agents around a **construction-application** framework:

**Agent Construction:**
1. **Profile Module**: Agent identity, role, personality
2. **Memory Module**: Short-term, long-term, hybrid memory structures
3. **Planning Module**: 
   - Without feedback: single-path (CoT, zero-shot CoT), multi-path (ToT, GoT, CoT-SC)
   - With feedback: environmental, human, model-based
4. **Action Module**: Tool use, embodied actions, goal completion

**Agent Applications:**
- Single-agent: Task-oriented (code, data), simulation-based
- Multi-agent: Cooperative, adversarial, debate, society simulation
- Human-agent interaction: Education, health, work assistance

### Key Contributions
- Clear separation of agent construction and application
- Detailed comparison of memory architectures
- Planning module taxonomy (with/without feedback)
- Evaluation strategies for agents

---

## Survey 3: "Cognitive Architectures for Language Agents" (CoALA)

- **Authors**: Theodore R. Sumers, Shunyu Yao, Karthik Narasimhan, Thomas L. Griffiths
- **Affiliation**: Princeton University
- **arXiv**: 2309.02427
- **Date**: September 2023

### Framework
CoALA provides a formal cognitive architecture framework for language agents:

**Structural Components:**
- **Memory**: Working memory (context), episodic (interaction history), semantic (knowledge base), procedural (code/prompts)
- **Decision-making**: Planning and reasoning processes
- **Grounding**: Interfaces to external environments

**Processing Cycle:**
```
Observe -> Retrieve from memory -> Reason/Plan -> Act -> Update memory
```

### Key Insight
Language agents can be understood through the lens of cognitive science -- they implement (simplified) versions of cognitive architectures like ACT-R and SOAR, with the LLM serving as a flexible reasoning engine.

### Agent Design Space
The paper defines a systematic design space along dimensions:
- Memory type (what kinds of memory)
- Decision procedure (how decisions are made)
- Grounding (what external interfaces exist)
- Learning (how the agent improves over time)

---

## Survey 4: "Agent AI: Surveying the Horizons of Multimodal Interaction"

- **Authors**: Zane Durante, Qiuyuan Huang, Naoki Wake, et al.
- **Affiliation**: Microsoft Research, Stanford, CMU, et al.
- **arXiv**: 2401.03568
- **Date**: January 2024

### Focus
Broader view of "Agent AI" encompassing multimodal, embodied, and interactive agents:
- Vision-Language-Action models
- Gaming and virtual world agents
- Robotics and embodied agents
- Healthcare and scientific agents

### Key Contribution
Bridges the gap between LLM-based text agents and multimodal/embodied agents, providing a unified perspective.

---

## Survey 5: "A Survey on the Memory Mechanism of Large Language Model based Agents"

- **Authors**: Zeyu Zhang, Xiaohe Bo, Chen Ma, et al.
- **Affiliation**: Multiple institutions
- **arXiv**: 2404.13501
- **Date**: April 2024

### Focus
Deep dive into memory systems for LLM agents:

**Memory Taxonomy:**
```
Agent Memory
├── Inside-trial Memory (within a single task)
│   ├── Working Memory (context window)
│   ├── Episodic Buffer (observation cache)
│   └── Semantic Cache (intermediate knowledge)
├── Cross-trial Memory (across tasks)
│   ├── Episodic Memory (past experiences)
│   ├── Semantic Memory (accumulated knowledge)
│   └── Procedural Memory (learned strategies)
└── Memory Operations
    ├── Reading (retrieval)
    ├── Writing (storage)
    ├── Reflection (meta-cognition)
    └── Consolidation (compression, abstraction)
```

---

## Survey 6: "Large Language Model based Multi-Agents: A Survey of Progress and Challenges"

- **Authors**: Taicheng Guo, Xiuying Chen, Yaqi Wang, et al.
- **arXiv**: 2402.01680
- **Date**: February 2024

### Focus
Dedicated survey of multi-agent LLM systems:

**Taxonomy of Multi-Agent Interactions:**
1. **Cooperative**: Agents work together toward a shared goal
   - Ordered messaging (round-robin, chain)
   - Unordered messaging (blackboard, broadcast)
2. **Competitive/Adversarial**: Agents compete or debate
3. **Mixed**: Combination of cooperative and competitive elements

**Applications:**
- Software development (ChatDev, MetaGPT)
- Science and research (multi-agent scientific debate)
- Society simulation (Generative Agents, AgentSims)
- Gaming (Voyager, Ghost in the Minecraft)
- Reasoning (multi-agent debate for factuality)

---

## Survey 7: "Tool Learning with Large Language Models: A Survey"

- **Authors**: Changle Qu, Sunhao Dai, Xiaochi Wei, et al.
- **arXiv**: 2405.17935
- **Date**: May 2024

### Focus
Comprehensive survey of tool use by LLMs:

**Tool Learning Pipeline:**
1. **Tool Creation**: Creating new tools (code generation, API wrapping)
2. **Tool Selection**: Choosing appropriate tools for a task
3. **Tool Calling**: Generating correct function calls with proper arguments
4. **Tool Result Processing**: Interpreting and using tool outputs
5. **Tool Learning**: Improving tool use through experience

**Tool Types:**
- Perception tools (search, read, browse)
- Action tools (write, execute, communicate)
- Computation tools (calculate, analyze, transform)
- Domain-specific tools (scientific instruments, databases)

---

## Survey 8: "Retrieval-Augmented Generation for Large Language Models: A Survey"

- **Authors**: Yunfan Gao, Yun Xiong, Xinyu Gao, et al.
- **arXiv**: 2312.10997
- **Date**: December 2023 (updated 2024)

### RAG Paradigm Evolution
```
Naive RAG -> Advanced RAG -> Modular RAG
```

**Naive RAG**: Simple retrieve-then-generate pipeline
**Advanced RAG**: Pre-retrieval optimization + post-retrieval processing
**Modular RAG**: Flexible, reconfigurable RAG components

### RAG vs. Fine-tuning vs. Prompt Engineering
| Aspect | RAG | Fine-tuning | Prompt Engineering |
|--------|-----|-------------|-------------------|
| External knowledge | Yes | Embedded in weights | In-context only |
| Training required | No (retrieval model optional) | Yes | No |
| Updatable knowledge | Easy (update docs) | Requires retraining | Limited by context |
| Best for | Dynamic knowledge, citations | Style/format/domain adaptation | Simple tasks |

---

## Survey 9: "Personal LLM Agents: Insights and Survey about the Capability, Efficiency and Security"

- **Authors**: Yuanchun Li, Hao Wen, Weijun Wang, et al.
- **Affiliation**: Tsinghua University, Microsoft
- **arXiv**: 2401.05459
- **Date**: January 2024

### Focus
Agents that run on personal devices and interact with personal data:
- On-device agent capabilities
- Privacy and security considerations
- Efficiency challenges (running on limited hardware)
- Personal data management and consent

---

## Survey 10: "The Landscape of Emerging AI Agent Architectures for Reasoning, Planning, and Tool Calling"

- **Authors**: Tula Masterman, Sandi Besen, Mason Sawtell, Alex Chao
- **Affiliation**: Neudesic (an IBM Company)
- **arXiv**: 2404.11584
- **Date**: April 2024

### Architectural Classification
1. **Single Agent**: One LLM with tools (ReAct, Plan-Execute)
2. **Multi-Agent - Vertical**: Hierarchical delegation (manager-worker)
3. **Multi-Agent - Horizontal**: Peer-to-peer collaboration
4. **Hybrid**: Combination approaches

### Evaluation Framework
Proposes criteria for selecting agent architectures:
- Task complexity
- Required autonomy level
- Latency requirements
- Cost constraints
- Safety requirements

---

## Chronological Reading Order (Recommended)

1. **Start here**: CoALA (2309.02427) -- for the theoretical framework
2. **Comprehensive overview**: Fudan survey (2309.07864) -- most thorough
3. **Construction-focused**: Renmin survey (2308.11432) -- practical taxonomy
4. **Architecture patterns**: Masterman et al. (2404.11584) -- practical selection guide
5. **Multi-agent deep dive**: Guo et al. (2402.01680) -- if building multi-agent systems
6. **Memory deep dive**: Zhang et al. (2404.13501) -- if designing memory systems
7. **RAG deep dive**: Gao et al. (2312.10997) -- if building RAG-based agents
8. **Tool use deep dive**: Qu et al. (2405.17935) -- if focusing on tool integration

## Key Observations Across Surveys

1. **Convergent taxonomy**: All surveys converge on similar core components (planning, memory, tools, action) despite different nomenclature
2. **Memory is underexplored**: Most agent frameworks have rudimentary memory compared to what surveys recommend
3. **Evaluation is hard**: No standardized evaluation framework for agents exists; benchmarks are fragmented
4. **Multi-agent is promising but costly**: Multi-agent systems show strong results but at significant computational cost
5. **Safety is critical**: All surveys highlight safety/alignment as a major open challenge
6. **The field moves fast**: Many survey papers are already partially outdated due to the pace of development

## Notes

Key benchmarks mentioned across surveys for evaluating agents:
- **AgentBench**: Comprehensive multi-environment agent benchmark
- **ToolBench**: Tool use evaluation
- **WebArena**: Web browsing tasks
- **SWE-bench**: Software engineering tasks
- **GAIA**: General AI assistant evaluation
- **HumanEval / MBPP**: Code generation (limited agent evaluation)
- **ALFWorld**: Embodied household tasks
- **HotPotQA**: Multi-hop question answering

## Cross-References (this knowledge base)

This file provides a consolidated annotated overview. For detailed individual paper notes, see:
- `papers/agent/_index.md` -- master index of all agent papers in this repository
- `papers/agent/survey/llm_agent_survey_fudan.md` -- detailed Fudan survey notes
- `papers/agent/survey/llm_agent_survey_renmin.md` -- detailed Renmin survey notes
- `papers/agent/memory/coala.md` -- CoALA paper notes
- `papers/agent/memory/reflexion.md` -- Reflexion paper notes
- `papers/agent/memory/memgpt.md` -- MemGPT paper notes
- `papers/agent/foundational/react.md` -- ReAct paper notes
- `papers/agent/foundational/chain_of_thought.md` -- CoT paper notes
- `papers/agent/foundational/tree_of_thoughts.md` -- ToT paper notes
