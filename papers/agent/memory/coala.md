---
title: "Cognitive Architectures for Language Agents (CoALA)"
authors: "Theodore R. Sumers, Shunyu Yao, Karthik Narasimhan, Thomas L. Griffiths"
venue: "Transactions on Machine Learning Research (TMLR)"
year: 2024
arxiv: "https://arxiv.org/abs/2309.02427"
code: "https://github.com/ysymyth/awesome-language-agents"
tags: [cognitive-architecture, language-agents, memory-systems, decision-making, framework, survey, production-systems]
category: agent/memory
date_read: 2026-05-08
---

# CoALA: Cognitive Architectures for Language Agents

## TL;DR

CoALA proposes a systematic framework for language agents grounded in cognitive science and symbolic
AI. It decomposes any language agent into three dimensions -- memory (working + long-term with
episodic/semantic/procedural subtypes), action space (internal reasoning/retrieval vs. external
grounding/learning), and a decision procedure (planning-then-execution loop). The framework unifies
prominent agents (ReAct, Reflexion, Voyager, Generative Agents, MemGPT) and connects LLM-based
agent design to classical cognitive architectures (Soar, ACT-R), revealing underexplored dimensions.

## Motivation & Problem

By mid-2023, the LLM agent landscape was fragmented with no shared vocabulary or design taxonomy:
1. **No common vocabulary**: Inconsistent terminology across papers for similar constructs.
2. **Ad hoc design**: Architectures designed case-by-case without principled component analysis.
3. **Disconnection from cognitive science**: Decades of Soar/ACT-R research largely ignored.
4. **No design guidance**: Practitioners lack a systematic framework for choosing memory types,
   action modules, and decision processes for their specific agent needs.

## Method

### The Full Cognitive Architecture

```
+=====================================================================+
|                        LANGUAGE AGENT (CoALA)                       |
+=====================================================================+
|  +---------------------------+  +--------------------------------+  |
|  |     WORKING MEMORY        |  |       LONG-TERM MEMORY         |  |
|  | - Perceptual inputs       |  |  Episodic  : past experiences  |  |
|  | - Active goals            |  |  Semantic  : factual knowledge |  |
|  | - Reasoning intermediates |  |  Procedural: skills/code/LLM   |  |
|  | - Retrieved context       |  |  (each: parametric + external) |  |
|  +---------------------------+  +--------------------------------+  |
|  +---------------------------------------------------------------+  |
|  |                       ACTION SPACE                            |  |
|  | Internal Actions              | External Actions              |  |
|  | - Reasoning (LLM inference)   | - Grounding (tool use, env)   |  |
|  | - Retrieval (memory reads)    | - Learning (memory writes)    |  |
|  +---------------------------------------------------------------+  |
|  +---------------------------------------------------------------+  |
|  |  DECISION PROCEDURE: loop { plan(internal) -> execute(ext.) } |  |
|  +---------------------------------------------------------------+  |
+=====================================================================+
```

### Memory Types (adapted from Tulving/Anderson cognitive psychology)

| Type       | Content               | Read              | Write             | Agent Example             |
|-----------|------------------------|-------------------|-------------------|---------------------------|
| Working   | Current state, goals   | Direct access     | Overwrite/cycle   | ReAct thought-obs chain   |
| Episodic  | Past experiences       | Similarity search | Append per episode| Reflexion reflections     |
| Semantic  | World knowledge        | Query lookup      | Summarize & store | Generative Agents reflects|
| Procedural| Skills, code, weights  | Pattern matching  | Code synthesis    | Voyager skill library     |

**Parametric vs. non-parametric distinction** (unique to LLM agents): LLM weights constitute
read-only parametric memory (semantic + procedural knowledge from pretraining). All runtime
learning must use non-parametric external storage (vector stores, code libraries, logs).

### Production System Formalization

```
Production System (Classical)        Language Agent (CoALA)
-------------------------------      ---------------------------
Working memory (symbols)       -->   Working memory (text variables)
Production rules (if-then)     -->   LLM reasoning (flexible inference)
Conflict resolution            -->   Decision procedure (plan + select)
External actions               -->   Grounding actions (tools, APIs)
Learning (rule compilation)    -->   Learning actions (memory writes)

Formal: Agent = (Memory, ActionSpace, DecisionProcedure)
  Memory = WorkingMem + {EpisodicLTM, SemanticLTM, ProceduralLTM}
  ActionSpace = Internal(reasoning, retrieval) + External(grounding, learning)
  DecisionProcedure = loop { plan(Internal) -> select -> execute(External) }
```

### Decision Procedure Pseudocode

```
procedure AGENT_LOOP(environment):
    working_memory.init(goals, percepts)
    while not terminated:
        // PLANNING PHASE (internal actions only)
        for i in 1..max_plan_steps:
            retrieved = RETRIEVE(long_term_memory, working_memory)
            thought = LLM_REASON(working_memory + retrieved)
            candidate = PROPOSE_ACTION(thought)
            score = EVALUATE(candidate, working_memory)
        selected = SELECT_BEST(candidates)
        // EXECUTION PHASE (external actions)
        if selected.type == GROUNDING:
            obs = environment.execute(selected)
            working_memory.update(obs)
        elif selected.type == LEARNING:
            long_term_memory.write(selected.content, selected.memory_type)
```

Instantiation levels: Simple (ReAct: single proposal, immediate execution), Planning
(Tree-of-Thought: multiple proposals, LLM scoring, best-first search), Learning (Reflexion:
standard cycle + episodic memory writes).

## Key Innovations

1. **Unified taxonomy** grounded in cognitive science for all language agent components.
2. **Memory typology** with parametric/non-parametric split unique to LLM-based agents.
3. **Internal/external action distinction** cleanly separating reasoning from grounding.
4. **Production system bridge** connecting to 50+ years of cognitive architecture research.
5. **Design space mapping** revealing unexplored regions via systematic agent comparison.

## Mapping Existing Agents onto CoALA

| Agent             | Episodic | Semantic | Procedural | Learning   | Decision Process       |
|-------------------|:--------:|:--------:|:----------:|:----------:|------------------------|
| ReAct             | No       | Param.   | Param.     | None       | Single proposal, exec  |
| Reflexion         | Yes      | Param.   | Param.     | Episodic   | Propose + reflect      |
| Voyager           | Log      | Param.   | Code lib   | Procedural | Propose + verify skill |
| Generative Agents | Stream   | Reflects | Param.     | Epi + Sem  | Retrieve-reflect-plan  |
| MemGPT            | Recall   | Archival | Functions  | All types  | Self-directed paging   |
| DEPS              | Errors   | Param.   | Param.     | None       | Describe-explain-plan  |
| SayCan            | No       | Param.   | Affordances| None       | Score + select         |

**ReAct vs. Reflexion**: Same basic architecture; Reflexion adds one episodic memory module and
one learning phase. CoALA reveals this as a minimal extension.
**Voyager vs. Generative Agents**: Both memory-rich but store different types -- procedural
(code skills) vs. episodic (events) + semantic (reflections). Different quadrants of memory space.

## Comparison with Classical Cognitive Architectures

| Dimension         | Soar                        | ACT-R                      | CoALA                      |
|-------------------|-----------------------------|-----------------------------|----------------------------|
| Working memory    | Symbolic WMEs               | Goal/problem state          | Text vars + LLM context   |
| LT memory types   | Procedural, semantic        | Declarative, procedural     | Episodic, semantic, proced.|
| Decision cycle    | Propose-decide-apply        | Production matching/firing  | Plan then execute          |
| Learning          | Chunking (rule compilation) | Activation learning         | Memory writes (any type)   |
| Conflict resoln.  | Preference semantics        | Utility-based selection     | LLM reasoning over cands.  |
| Flexibility       | Manual rule authoring       | Manual rule authoring       | LLM provides flexible inf. |

Key CoALA advantage: LLM replaces brittle hand-engineered rules with flexible reasoning.
Key CoALA limitation: Lacks formal decision theory (utility functions, preference semantics).

## Analysis & Insights

1. **Most agents underutilize memory**: Full space of 2^3=8 LTM combinations mostly unexplored.
2. **Procedural memory is most underexplored**: Only Voyager truly learns reusable skills.
3. **Learning is rare**: Few agents write to long-term memory, limiting self-improvement.
4. **Decision procedures are simple**: Most use single-step propose-execute, not full planning.
5. **LLM as dual-role engine**: Simultaneously serves as parametric memory store AND reasoning
   engine -- unique to LLM agents, no analog in classical architectures.
6. **Retrieval quality is critical**: Long-term memory is only as useful as retrieval quality.

## Limitations & Critiques

1. **Descriptive, not prescriptive**: Maps design space but does not navigate it.
2. **No empirical validation**: Purely theoretical; no experiments compare CoALA-designed agents.
3. **LLM as black box**: Parametric memory is not inspectable unlike Soar/ACT-R rules.
4. **No formal decision theory**: Lacks utility/reward framework for action selection.
5. **Missing multi-agent dimension**: Focuses on individual agents; coordination not addressed.
6. **Rapid obsolescence risk**: Static taxonomy in a fast-moving field.

## Follow-up Work

- **MemGPT** (2023): Operationalizes CoALA's memory hierarchy with virtual memory paging.
- **A-MEM** (2025): Zettelkasten-based memory blending episodic and semantic storage.
- **MemoryBank**: Implements forgetting curves for the "what to forget" question.
- **AgentBench/AgentBoard**: Evaluation suites aligned with CoALA capability dimensions.
- **LangGraph/LangChain memory**: Production frameworks adopting CoALA-style memory taxonomy.

## Key Takeaways

1. **Design modularly**: Separate memory, actions, and decision procedures for principled design.
2. **Use the memory checklist**: Explicitly consider episodic, semantic, procedural LTM needs.
3. **Invest in learning**: Memory writes enable self-improvement -- most underexplored dimension.
4. **LLM replaces rules, not architecture**: Soar/ACT-R structural insights remain valuable.
5. **All runtime learning = non-parametric writes**: Fundamental constraint shaping agent design.
6. **CoALA as diagnostic**: When agents fail, the framework identifies the bottleneck component.
