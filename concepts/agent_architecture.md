---
title: LLM Agent Architecture
tags: [agent, architecture, core-concept]
related: [react_pattern, agent_memory, tool_use, multi_agent_systems]
---

## Definition

An LLM Agent is a system that uses a large language model as its core reasoning engine, augmented with the ability to perceive its environment, plan actions, use tools, and learn from feedback.

## Core Components

```
                    ┌─────────────┐
                    │   User Goal │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │  Perception │  <- environment, user input, observations
                    └──────┬──────┘
                           │
              ┌────────────▼────────────┐
              │     Reasoning (LLM)     │  <- planning, decomposition, reflection
              │  ┌─────────────────┐    │
              │  │ Working Memory  │    │
              │  └─────────────────┘    │
              └────────────┬────────────┘
                     ┌─────┴─────┐
                     │           │
              ┌──────▼──┐  ┌────▼─────┐
              │  Memory │  │  Action  │  <- tool call, code exec, API, UI
              │  System │  │  System  │
              └─────────┘  └────┬─────┘
                                │
                         ┌──────▼──────┐
                         │ Observation │  <- tool result, env feedback
                         └──────┬──────┘
                                │
                         (loop back to Reasoning)
```

### 1. Perception
Converts environmental percepts into meaningful representations for the LLM.

### 2. Reasoning (Brain)
The LLM core. Handles:
- **Task decomposition**: breaking complex goals into sub-tasks
- **Planning**: generating action sequences
- **Decision-making**: choosing which action to take next
- **Reflection**: evaluating past actions and learning from mistakes

### 3. Memory
- **Working Memory**: current context window contents
- **Episodic Memory**: past experiences and interactions
- **Semantic Memory**: facts, knowledge, world model
- **Procedural Memory**: learned skills, code snippets, procedures
See: `agent_memory.md`

### 4. Action
Translates decisions into concrete actions:
- Tool/API calls (function calling)
- Code execution
- Web browsing
- File system operations
- GUI interaction (mouse, keyboard)

### 5. Observation
Feedback from the environment after an action. Closes the perception-action loop.

## Architecture Patterns

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **ReAct** | Thought-Action-Observation loop | General-purpose agent tasks |
| **Plan-and-Execute** | First plan all steps, then execute | Well-defined multi-step tasks |
| **Reflexion** | Attempt, reflect, retry with lessons | Tasks with clear success/failure signal |
| **Tree of Thoughts** | Explore multiple reasoning paths | Tasks requiring exploration/backtracking |
| **Multi-Agent** | Multiple specialized agents collaborate | Complex tasks requiring diverse expertise |

## Key Design Decisions

1. **How much autonomy?** Full autonomous loop vs. human-in-the-loop
2. **What memory structure?** Simple context vs. external retrieval vs. managed memory
3. **Single vs. multi-agent?** One agent with many tools vs. specialized agents
4. **Tool design (ACI)**: How the environment is presented to the agent matters as much as the model

## Detailed References (in this knowledge base)

- Detailed architecture patterns (ReAct, Plan-Execute, Reflexion, Multi-Agent): `agent_architecture_patterns.md`
- Comprehensive fundamentals and component deep-dive: `llm_agent_fundamentals.md`
- Memory systems deep-dive: `agent_memory_systems.md`, `agent_memory.md`
- Framework comparison: `agent_frameworks.md`
- Key techniques (function calling, RAG, CoT, ToT): `agent_techniques.md`
- Survey papers: `papers/llm/llm_agent_surveys.md`
