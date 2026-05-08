---
title: "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation"
authors: "Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Beibin Li, Erkang Zhu, Li Jiang, Xiaoyun Zhang, Shaokun Zhang, Jiale Liu, Ahmed Hassan Awadallah, Ryen W. White, Doug Burger, Chi Wang"
venue: "COLM 2024"
year: 2024
arxiv: "https://arxiv.org/abs/2308.08155"
code: "https://github.com/microsoft/autogen"
institution: "Microsoft Research, Penn State, University of Washington, Xidian University"
tags: [multi-agent, conversation-programming, framework, code-execution, human-in-the-loop, tool-use, group-chat]
category: agent/multi_agent
date_read: 2026-05-08
---

# AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation

## TL;DR

AutoGen introduces the "conversable agent" abstraction and "conversation programming" paradigm,
unifying multi-agent LLM applications into customizable multi-turn conversations. Agents flexibly
combine LLM reasoning, human input, and code execution. Supports two-agent, sequential, group chat
(with dynamic speaker selection), and nested conversation patterns. Achieves ~6% improvement on MATH,
68%->90% on HumanEval with adaptive multi-model strategies at -18% cost, and No. 1 on GAIA benchmark.
Later evolved into AutoGen 0.4 (event-driven architecture) and Magentic-One multi-agent system.

## Motivation & Problem

1. **No unified abstraction**: Each multi-agent application required bespoke interaction logic.
2. **Rigid pipelines**: Chain-based frameworks (LangChain, LlamaIndex) poorly adapted to multi-turn,
   multi-party conversational interactions.
3. **Poor human integration**: No standard pattern for configurable human-in-the-loop interaction.
4. **Unsafe code execution**: LLM-generated code execution needed sandboxing handled ad hoc.
5. **Pattern rigidity**: Fixed topologies (two-agent, chain) required rewrites to change.

Core insight: **Unify all concerns under conversable agents + conversation programming, where the
multi-turn conversation itself IS the computational substrate.**

## Method

### Conversable Agent Abstraction

```
ConversableAgent
  |-- llm_config:      {model, temperature, system_message, functions}
  |-- human_input_mode: "ALWAYS" | "TERMINATE" | "NEVER"
  |-- code_execution:   {work_dir, use_docker, timeout}
  |-- function_map:     {name -> callable} (registered tools)
  |-- max_consecutive_auto_reply: int
  |
  |-- generate_reply():  Response via priority pipeline:
  |     1. Custom registered reply functions
  |     2. Function/tool call triggers
  |     3. Code execution triggers
  |     4. LLM call with conversation history
  |     5. Human input check
  |-- send(message, recipient) / receive(message, sender)
  |-- register_reply(trigger, func, position)

Specializations:
  AssistantAgent  -- LLM-backed, no code exec, human_input=NEVER
  UserProxyAgent  -- Code exec + human proxy, human_input=TERMINATE
```

### Conversation Pattern Types

**Pattern 1: Two-Agent Chat** -- Simplest back-and-forth until termination:
```
AssistantAgent  <-- message/reply -->  UserProxyAgent
(LLM reasoning,                       (code execution,
 code generation)                      human input)
Terminates: max_turns, "TERMINATE" keyword, or human stop
```

**Pattern 2: Sequential Chat** -- Chained two-agent conversations with carryover:
```
Chat_1 (A<->B) --summary--> Chat_2 (C<->D) --summary--> Chat_3 (E<->F)
Each chat's summary injected as context into the next stage.
```

**Pattern 3: Group Chat** -- Multiple agents, managed by GroupChatManager:
```
+------------------------------------------------------+
|               GroupChatManager                        |
|  Speaker selection strategies:                        |
|    round_robin | random | auto (LLM) | manual | custom|
|------------------------------------------------------|
|  +------+   +------+   +------+   +------+           |
|  |Coder |   |Critic|   |Tester|   |Human |           |
|  +------+   +------+   +------+   +------+           |
+------------------------------------------------------+

Custom transition graph:
  allowed_transitions = {
    planner: [coder, critic],
    coder:   [tester, critic],
    critic:  [coder, planner],
    tester:  [coder, planner]
  }
```

LLM-based selection: Manager prompts LLM with conversation context and agent descriptions
to pick the most appropriate next speaker dynamically.

**Pattern 4: Nested Conversations** -- Sub-conversations within parent:
Agent_B in main chat triggers sub-chat among Sub-Agents 1-3 to resolve a sub-problem;
result returned to main conversation. Enables hierarchical task decomposition.

### Human Proxy Agent Design

| Mode        | Behavior                               | Use Case                  |
|-------------|----------------------------------------|---------------------------|
| `ALWAYS`    | Human prompted every turn              | Full oversight, collab    |
| `TERMINATE` | Human prompted only at termination     | Review final output       |
| `NEVER`     | Fully autonomous                       | Batch processing, pipelines|

```
procedure USER_PROXY_RECEIVE(message):
    if message contains code blocks:
        return SANDBOX_EXECUTE(code)       // Docker or local subprocess
    if message contains function_call:
        return function_map[name](**args)   // registered tool execution
    switch human_input_mode:
        ALWAYS:    return get_human_input()
        TERMINATE: if is_termination(msg): return get_human_input()
        NEVER:     return auto_reply()
```

### Code Execution Sandbox

```
Config: work_dir (isolated dir), use_docker (bool/image), timeout (seconds)
Flow: Parse code blocks -> Detect language -> Docker/subprocess exec -> Capture stdout/stderr
      -> Return to conversation -> Agent debugs if error -> Iterate
```

Docker provides: process isolation, filesystem sandboxing (only work_dir mounted),
network control, resource limits, clean environment per execution.

### Tool Use Integration

```python
@assistant.register_for_llm(description="Search the web")
@user_proxy.register_for_execution()
def web_search(query: str) -> str:
    return search_api.search(query)
# AutoGen auto-includes schema in LLM calls, parses function_call,
# routes to executor, returns result to conversation
```

## Key Innovations

1. **Conversable agent**: Unified type combining LLM, human, code exec, tools via configuration.
2. **Conversation programming**: Workflows as programmable conversations, not fixed pipelines.
3. **Configurable human-in-the-loop**: ALWAYS/TERMINATE/NEVER spectrum as first-class feature.
4. **Dynamic speaker selection**: LLM-based or custom-function group chat orchestration.
5. **Nested conversations**: Hierarchical decomposition with clean concern separation.
6. **First-class sandboxing**: Docker-based code execution as framework feature.

## Experimental Setup & Use Cases

| Use Case                  | Pattern       | Agents                        |
|---------------------------|---------------|-------------------------------|
| Math problem solving      | Two-agent     | MathChat + UserProxy          |
| Retrieval-augmented chat  | Two-agent     | RetrieveAssistant + Proxy     |
| Code gen & debugging      | Two-agent     | Assistant + CodeExecutor      |
| Multi-agent coding        | Group chat    | Coder + Critic + Executor     |
| Conversational chess      | Two-agent     | 2 Players + Board validator   |
| Research group chat       | Group chat    | Domain expert agents          |

## Results

### Math Problem Solving (MATH Dataset)

| Method                          | Accuracy | Algebra  |
|--------------------------------|:--------:|:--------:|
| GPT-4 (vanilla, zero-shot)     | ~50%     | ~65%     |
| Program of Thought (PoT)       | ~52%     | ~63%     |
| GPT-4 + code interpreter       | 69.7%    | --       |
| **MathChat (AutoGen)**          | **~59%** | **~78%** |
| **AutoGen GPT-4 two-agent**    | **74.8%**| --       |

+6% overall improvement; +15% in Algebra via dynamic switching between symbolic
computation (code execution) and step-by-step reasoning.

### Coding (HumanEval)

| Strategy                         | Pass@1   | Cost vs. GPT-4 |
|---------------------------------|:--------:|:---------------:|
| GPT-4 single model              | 68%      | baseline        |
| GPT-4 + GPT-3.5 adaptive        | ~82%     | -10%            |
| **AutoGen multi-agent adaptive** | **~90%** | **-18%**        |

Adaptive strategy: GPT-3.5 for simple problems, escalate to GPT-4 only when needed.

### GAIA Benchmark

AutoGen multi-agent configuration achieved No. 1 accuracy across all three difficulty
levels with significant margins, demonstrating effectiveness on complex multi-step tasks.

### Detailed Math Results

| Method                        | MATH (%) | Avg Turns |
|------------------------------|:--------:|:---------:|
| GPT-4 (CoT)                  | 57.6     | 1         |
| GPT-4 + code interpreter     | 69.7     | 1         |
| AutoGen (GPT-3.5, two-agent) | 55.2     | 4.1       |
| **AutoGen (GPT-4, two-agent)**| **74.8** | **3.2**   |

Multi-turn conversation with code execution enables iterative refinement, catching errors
through execution feedback rather than single-pass reasoning.

## Analysis & Insights

1. **Conversation as computation**: Multi-agent conversation is a general-purpose computational
   substrate, not just coordination glue for separate computation steps.
2. **Flexibility vs. structure trade-off**: AutoGen (flexible, any-to-any) vs. MetaGPT (rigid,
   structured). More adaptable but requires more careful orchestration design.
3. **Code execution as grounding**: Sandbox provides ground-truth verification that pure LLM
   reasoning cannot match -- agents verify reasoning through computation.
4. **Multi-model strategies beat scaling**: Smarter orchestration (GPT-3.5 + GPT-4 adaptive)
   outperforms uniform GPT-4 at lower cost.
5. **Speaker selection is critical**: Quality of "who speaks next" significantly affects group
   chat outcomes; LLM selection is natural but costly.

## Magentic-One System (Fourney et al., 2024)

Production multi-agent team built on AutoGen:

```
              +------------------+
              |   Orchestrator   |
              | - Task Ledger    | (plan, facts, educated guesses)
              | - Progress Ledger| (step-by-step self-reflection)
              +--------+---------+
                       | delegates to specialized agents
          +--------+---+----+---------+
          |        |        |         |
       +------+ +------+ +------+ +--------+
       | Web  | | File | |Coder | |Terminal|
       |Surfer| |Surfer| |      | |        |
       +------+ +------+ +------+ +--------+
```

Orchestrator creates Task Ledger, delegates per step, collects results, maintains Progress
Ledger with self-reflection on completion. Achieved state-of-the-art on benchmarks requiring
web browsing + file handling + code execution.

## AutoGen 0.4: Event-Driven Redesign (Jan 2025)

```
v0.2 (Original)               v0.4 (Redesign)
Synchronous messaging    -->   Asynchronous event-driven bus
Python-only              -->   Cross-language (Python + .NET)
In-process agents        -->   Distributed agent runtime
Direct agent references  -->   Topic-based message routing
No state persistence     -->   Built-in state management
Minimal observability    -->   Tracing + logging + debugging
```

Integrated into Microsoft Agent Framework alongside Semantic Kernel. Community fork (AG2)
maintains 0.2 API compatibility for backward-compatible development.

## Limitations & Critiques

1. **Conversation overhead**: No built-in efficiency optimization for unproductive exchanges.
2. **Speaker selection fragility**: LLM-based selection can derail group conversations.
3. **Debugging complexity**: Multi-agent failures hard to trace through interleaved messages.
4. **No structured output**: Free-form text communication introduces ambiguity (vs. MetaGPT).
5. **Cost unpredictability**: Variable conversation length makes per-task cost unpredictable.
6. **No persistent memory**: v0.2 has no cross-conversation agent memory.
7. **Group chat scaling**: 5-6+ agents become slow and expensive with growing history.
8. **Steep learning curve**: Many design decisions with limited built-in guidance.

## Follow-up Work

- **Magentic-One** (2024): Production multi-agent with ledger-based orchestration on AutoGen.
- **AutoGen Studio**: Visual interface for no-code multi-agent workflow building.
- **AutoGen 0.4** (2025): Event-driven distributed rewrite in Microsoft Agent Framework.
- **CrewAI** (2024): Simplified multi-agent framework with role-based design.
- **LangGraph** (2024): Graph-based orchestration inspired by AutoGen conversation patterns.
- **Swarm** (OpenAI, 2024): Lightweight multi-agent with handoff mechanism.

## Key Takeaways

1. **Conversation IS the program**: Unified abstraction covers math, coding, retrieval, research.
2. **Configurable autonomy**: ALWAYS/TERMINATE/NEVER human input is essential -- different tasks
   need different oversight levels.
3. **Sandbox everything**: First-class Docker sandboxing should be a framework requirement.
4. **Adaptive orchestration works**: Multi-model strategies achieve 90% at -18% cost vs. 68%.
5. **Flexibility demands discipline**: Open conversation programming needs careful termination
   conditions, role definitions, and pattern selection.
6. **Frameworks must evolve**: v0.2->v0.4 migration shows production agents need fundamentally
   different (event-driven, distributed) architectures than research prototypes.
