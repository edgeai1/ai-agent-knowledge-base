---
title: "Executable Code Actions Elicit Better LLM Agents"
authors: Xingyao Wang, Yangyi Chen, Lifan Yuan, Yizhe Zhang, Yunzhu Li, Hao Peng, Heng Ji
venue: ICML 2024
year: 2024
url: https://arxiv.org/abs/2402.01030
code: https://github.com/xingyaoww/code-act
tags: [coding, action-space, code-generation, agent-design, openhands]
status: done
---

## TL;DR

Proposes CodeAct, an approach that uses executable Python code as the unified action space for LLM agents, achieving up to 20% higher success rates than JSON or text-based action formats across 17 LLMs.

## Motivation & Problem

Existing LLM agent frameworks require models to produce actions in constrained formats:
- **JSON actions**: `{"action": "search", "query": "..."}` -- limited to predefined action schemas
- **Text/NL actions**: `search for "..."` -- ambiguous parsing, no compositionality
- **Structured formats**: ReAct-style `Action: tool_name[args]` -- inflexible, one tool per turn

These approaches share fundamental limitations:
1. **Constrained action space**: agents can only invoke pre-registered tools, cannot compose novel operations
2. **No state persistence**: each action is stateless; intermediate results cannot be stored or manipulated
3. **Limited compositionality**: chaining multiple tool calls requires multiple turns, increasing latency and error risk
4. **No self-debugging**: when an action fails, agents have no mechanism for introspection

The key insight: LLMs are already trained on massive code corpora. Python code is a natural, expressive, composable action language that LLMs already understand.

## Method

### The CodeAct Action Format

In CodeAct, the agent's action at each turn is an executable Python code block. The code runs in a persistent IPython interpreter, and stdout/stderr from execution is returned as the observation.

```
Agent Turn Format:
+--------------------------------------------+
| Thought: I need to find files matching     |
|          the pattern and count them.        |
|                                            |
| Action:                                    |
| ```python                                  |
| import os                                  |
| files = [f for f in os.listdir('.')        |
|          if f.endswith('.py')]              |
| print(f"Found {len(files)} Python files")  |
| for f in sorted(files)[:10]:               |
|     print(f"  - {f}")                      |
| ```                                        |
+--------------------------------------------+
            |
            v
+--------------------------------------------+
| Observation (from interpreter):            |
| Found 15 Python files                      |
|   - agent.py                               |
|   - config.py                              |
|   - ...                                    |
+--------------------------------------------+
```

### Why Code is Superior to JSON/Text

```
+------------------+--------+---------+-----------+
| Capability       | JSON   | Text    | CodeAct   |
+------------------+--------+---------+-----------+
| Compositionality | Low    | Low     | High      |
| State persistence| None   | None    | Full      |
| Self-debugging   | None   | None    | Native    |
| Tool composition | 1/turn | 1/turn  | N/turn    |
| Control flow     | None   | None    | Full      |
| Data manipulation| None   | None    | Full      |
| Novel operations | No     | No      | Yes       |
| Turing-complete  | No     | No      | Yes       |
+------------------+--------+---------+-----------+
```

Concrete advantages:
1. **Multi-tool composition in a single turn**: `result = tool_a(tool_b(x))` vs. two separate turns
2. **Variable persistence**: store intermediate results across turns via Python variables
3. **Control flow**: loops, conditionals, exception handling within a single action
4. **Self-debugging**: `try/except` blocks, error inspection, dynamic correction
5. **Library access**: import any Python library (pandas, requests, etc.) for complex operations
6. **Data transformation**: process, filter, and aggregate tool outputs using native Python

### Multi-Turn Interaction Protocol

```
System Prompt: "You are a helpful assistant. You can use Python code
               to interact with the environment..."

Turn 1 (Agent):
  Thought: Let me search for relevant information.
  Action: ```python
  result = search("climate change effects")
  print(result[:500])
  ```

Turn 1 (Environment):
  Observation: [search results...]

Turn 2 (Agent):
  Thought: The search returned an error. Let me fix the query.
  Action: ```python
  try:
      result = search("climate change effects 2024")
      filtered = [r for r in result if r['relevance'] > 0.8]
      print(f"Found {len(filtered)} relevant results")
  except Exception as e:
      print(f"Error: {e}")
      # Fallback strategy
      result = web_browse("en.wikipedia.org/wiki/Climate_change")
  ```
```

### Comparison of Action Formats

**JSON Format** (e.g., OpenAI function calling):
```json
{"name": "search", "arguments": {"query": "climate change", "limit": 5}}
```
Limitation: one tool per turn, no post-processing, no error handling.

**Text Format** (e.g., ReAct):
```
Action: Search[climate change effects]
```
Limitation: ambiguous parsing, no compositionality, fragile extraction.

**CodeAct Format**:
```python
results = search("climate change effects", limit=5)
relevant = [r for r in results if "temperature" in r["title"]]
if not relevant:
    results = search("global warming temperature data", limit=10)
print(json.dumps(relevant, indent=2))
```
Advantage: composition, filtering, fallback logic -- all in one turn.

### CodeActInstruct Dataset

- **Size**: ~7,000 multi-turn interaction trajectories
- **Sources**: curated from diverse tasks requiring tool use
- **Selection criteria**: prioritizes trajectories where the agent initially makes errors but then self-corrects, promoting self-debugging capability
- **Format**: each trajectory includes thought, code action, execution output
- **Purpose**: instruction-tuning open-source LLMs to use CodeAct format

### CodeActAgent Models

Two fine-tuned models released:
- **CodeActAgent-Llama-2-7B**: fine-tuned from Llama-2-7B-chat
- **CodeActAgent-Mistral-7B**: fine-tuned from Mistral-7B-v0.1

Both are trained on CodeActInstruct + general conversation data to maintain broad capabilities while excelling at agent tasks.

## Key Innovations

1. **Unified code action space**: formalizing executable code as the agent action format
2. **Persistent interpreter**: IPython session maintains state across turns, enabling stateful computation
3. **Self-debugging through execution**: error messages from code execution provide natural feedback for correction
4. **CodeActInstruct**: curated dataset emphasizing error-recovery trajectories for fine-tuning
5. **Format-agnostic evaluation**: systematic comparison across JSON, text, and code formats on identical tasks

## Experimental Setup

### Benchmarks
- **API-Bank**: evaluation of API/tool use capabilities across multiple difficulty levels
- **M3ToolEval**: 82 human-curated tasks requiring multi-tool composition in multi-turn interactions; newly introduced in this paper

### Models Evaluated
17 LLMs total:
- **Closed-source (9)**: GPT-4, GPT-3.5-turbo, Claude-2, Claude-instant, Gemini Pro, etc.
- **Open-source (8)**: Llama-2-7B/13B/70B, CodeLlama, Mistral-7B, etc.

### Baselines
- **JSON format**: structured JSON tool calls (OpenAI function-calling style)
- **Text format**: ReAct-style text-based actions

## Results

### API-Bank Results (Success Rate %)

| Model (selected)       | Text    | JSON    | CodeAct |
|------------------------|---------|---------|---------|
| GPT-4                  | 82.1    | 85.3    | **86.7**|
| GPT-3.5-turbo          | 73.4    | 76.2    | **78.9**|
| Claude-2               | 70.8    | 71.5    | **74.3**|
| Llama-2-70B            | 45.2    | 42.1    | **51.6**|
| Llama-2-7B             | 25.8    | 11.3    | **28.8**|

CodeAct was the best-performing format for 4/8 open-source LLMs and 4/9 closed-source LLMs on API-Bank.

### M3ToolEval Results (Multi-Tool Tasks)

CodeAct shows the largest gains on complex multi-tool tasks:
- Up to **20% absolute improvement** in success rate over JSON/text
- Fewer interaction turns needed to complete tasks
- Higher rate of successful self-correction after initial errors

### CodeActAgent Fine-tuned Models

| Model                       | Agent Tasks | General Tasks |
|-----------------------------|-------------|---------------|
| Llama-2-7B (base)           | 25.8%       | baseline      |
| CodeActAgent-Llama-2-7B     | **42.3%**   | comparable    |
| Mistral-7B (base)           | 38.1%       | baseline      |
| CodeActAgent-Mistral-7B     | **53.7%**   | comparable    |

Fine-tuned models show strong improvement on out-of-domain agent tasks while maintaining general conversation ability.

## Analysis & Insights

- **Complexity amplifies the gap**: on simple single-tool tasks, all formats perform similarly; on complex multi-tool composition tasks, CodeAct's advantage grows substantially
- **Self-debugging is key**: CodeAct agents recover from errors more frequently because they can inspect tracebacks and modify their approach programmatically
- **Fewer turns needed**: by composing multiple operations in a single code block, CodeAct agents complete tasks in fewer interaction rounds
- **Open-source gap**: smaller open-source models struggle more with JSON parsing than with code generation, making CodeAct relatively more beneficial for them
- **State persistence matters**: the ability to store intermediate results in variables across turns reduces redundant computation

## Limitations & Critiques

- **Security concerns**: executing arbitrary Python code poses sandboxing challenges; production deployment requires robust isolation
- **Code quality variance**: weaker models may generate syntactically valid but semantically incorrect code that is harder to detect than a malformed JSON
- **Python-centric**: the approach is tied to Python; extending to other languages or non-code domains requires rethinking
- **Dataset scale**: CodeActInstruct at 7K examples is relatively small for instruction tuning
- **Benchmark scope**: M3ToolEval, while useful, has only 82 tasks and may not capture the full diversity of real-world agent scenarios
- **Latency**: code execution adds overhead compared to simple JSON parsing in production systems

## Follow-up Work

- **OpenHands (formerly OpenDevin)**: adopted CodeAct as its default action format, becoming one of the leading open-source agent platforms
- **SWE-agent**: uses a related approach with custom commands executed in a shell environment
- **Code-as-policy approaches**: robotic control using code generation follows similar principles
- **Apple's adoption**: CodeAct recognized and featured in Apple ML Research
- **Multi-agent systems**: CodeAct format enables more natural communication between agents that share a Python runtime

## Key Takeaways

1. Executable code is a natural and superior action format for LLM agents, leveraging pre-training distribution
2. The compositionality and statefulness of code provide fundamental advantages over structured formats
3. Self-debugging through execution feedback is a capability unique to code-based action spaces
4. Fine-tuning on error-recovery trajectories (CodeActInstruct) effectively teaches agents to self-correct
5. The gap between action formats widens as task complexity increases, making CodeAct especially valuable for real-world applications
