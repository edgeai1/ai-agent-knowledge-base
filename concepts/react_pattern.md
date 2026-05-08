---
title: ReAct Pattern (Reasoning + Acting)
tags: [react, pattern, agent-loop, core-concept]
related: [agent_architecture, tool_use]
---

## Definition

ReAct is the foundational agent loop pattern where an LLM interleaves reasoning (thinking) with actions (tool calls), processing observations from the environment to iteratively work toward a goal.

## The Loop

```
┌──────────────────────────────────┐
│                                  │
│  ┌─────────┐                    │
│  │ Thought │  "I need to..."    │
│  └────┬────┘                    │
│       │                         │
│  ┌────▼────┐                    │
│  │ Action  │  search/execute    │
│  └────┬────┘                    │
│       │                         │
│  ┌────▼───────────┐             │
│  │  Observation   │  result     │
│  └────┬───────────┘             │
│       │                         │
│       └─────────────────────────┘
│       (repeat until done)
```

## Implementation Variants

### Basic ReAct (few-shot prompting)
The original -- few-shot examples showing Thought/Action/Observation format.

### Function-Calling ReAct (modern)
Same loop but using native function calling:
1. LLM decides to call a tool (structured JSON)
2. System executes the tool
3. Result returned to LLM as tool response
4. LLM continues reasoning

### ReAct + Reflection (Reflexion)
Add a reflection step after failure to learn from mistakes.

### ReAct + Planning (Plan-and-Execute)
First generate a full plan, then execute each step with ReAct.

## When to Use

- General-purpose agent tasks
- Tasks requiring external information
- Multi-step reasoning with tool use
- When the number of steps is not known in advance

## Limitations

- Can get stuck in loops
- No exploration of alternative paths (vs. Tree of Thoughts)
- Linear reasoning only
- Quality depends heavily on the model's reasoning ability
