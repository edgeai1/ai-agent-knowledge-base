---
title: LLM Agent Fundamentals
tags: [agent, llm, architecture, planning, memory, tool-use, survey]
related: [agent_architecture_patterns, agent_memory_systems, agent_frameworks, agent_techniques]
---

## Definition

An LLM-based agent is an autonomous or semi-autonomous system that uses a Large Language Model as its central "brain" or controller to perceive its environment, reason about tasks, make decisions, and take actions to achieve goals. Unlike vanilla LLM usage (single prompt-in, response-out), agents operate in loops -- iterating through cycles of observation, reasoning, action, and feedback.

The canonical formulation (from the Fudan University survey, Xi et al. 2023) defines an agent as:

**Agent = LLM + Memory + Planning + Tool Use + Action**

## Intuition

The key insight is that LLMs, despite being trained as next-token predictors, exhibit emergent capabilities (reasoning, instruction following, in-context learning) that can be harnessed as a general-purpose controller. By wrapping an LLM in a loop with access to external tools, memory, and structured prompting strategies, we can create systems that solve complex, multi-step problems that no single LLM call could handle.

The agent paradigm transforms the LLM from a "question answerer" into a "problem solver" by giving it:
1. The ability to break down problems (planning)
2. The ability to remember context across steps (memory)
3. The ability to interact with external systems (tools/actions)
4. The ability to evaluate its own progress (feedback/reflection)

## Core Components

### 1. Brain / Controller (the LLM)

The LLM serves as the central decision-making engine. It:
- Interprets user goals and instructions
- Generates reasoning traces
- Decides which actions to take
- Synthesizes observations into coherent responses

Key LLM capabilities leveraged by agents:
- **In-context learning**: Adapting behavior from examples in the prompt
- **Instruction following**: Executing structured workflows described in system prompts
- **Reasoning**: Chain-of-thought, step-by-step problem decomposition
- **Code generation**: Writing executable code as actions
- **Natural language understanding**: Parsing tool outputs and user feedback

### 2. Planning

The ability to decompose complex tasks into manageable sub-tasks and determine execution order.

**Task Decomposition Strategies:**
- **Chain-of-Thought (CoT)**: Sequential step-by-step reasoning (Wei et al., 2022)
- **Tree-of-Thought (ToT)**: Branching exploration of multiple reasoning paths (Yao et al., 2023)
- **Graph-of-Thought (GoT)**: DAG-structured reasoning with merging and refinement
- **LLM+P**: Using classical planners (e.g., PDDL) with LLM as the problem translator
- **Hierarchical planning**: High-level plan decomposed into sub-plans recursively

**Subgoal Planning:**
- Break a goal into ordered subgoals
- Each subgoal may require different tools or approaches
- Plan can be static (generated once) or dynamic (revised after each step)

### 3. Memory

Enables the agent to maintain context, learn from experience, and access relevant knowledge.

**Short-term Memory (Working Memory):**
- The LLM's context window itself
- Stores the current conversation, recent observations, and immediate plan
- Limited by context length (4K to 1M+ tokens depending on model)
- Managed via summarization, sliding window, or compaction strategies

**Long-term Memory:**
- Persists across sessions and beyond context window limits
- Typically implemented via external vector databases (Pinecone, Weaviate, ChromaDB, FAISS)
- Information is embedded and retrieved via semantic similarity search
- Stores: past interactions, learned facts, user preferences, successful strategies

**Episodic Memory:**
- Records specific past experiences and interaction sequences
- Enables the agent to recall "what happened" in similar past situations
- Used in Reflexion and experience-replay patterns
- Example: remembering that a particular API call failed with a specific error

**Semantic Memory:**
- General knowledge and facts (not tied to specific episodes)
- Can be pre-loaded (knowledge base) or accumulated over time
- Often implemented as a structured knowledge graph or document store

**Procedural Memory:**
- Stores "how to do" knowledge: learned action sequences, tool usage patterns
- Can be implemented as code libraries, prompt templates, or fine-tuned weights

**RAG-based Memory (Retrieval-Augmented Generation):**
- Hybrid approach: external knowledge base + retrieval mechanism + LLM generation
- Documents are chunked, embedded, and stored in a vector DB
- At query time, relevant chunks are retrieved and injected into the prompt
- Enables agents to work with large knowledge bases that exceed context limits
- Key components: chunking strategy, embedding model, retrieval algorithm, reranking

### 4. Tool Use (Function Calling)

The ability to invoke external tools, APIs, and functions to extend the agent's capabilities.

**Mechanism:**
- The LLM generates structured output (JSON, function calls) specifying which tool to call and with what arguments
- The framework executes the tool and returns results to the LLM
- The LLM incorporates results into its reasoning

**Common Tool Categories:**
- **Search & retrieval**: Web search, database queries, knowledge base lookup
- **Code execution**: Python interpreter, shell commands, sandboxed environments
- **API calls**: REST/GraphQL APIs, third-party services
- **File operations**: Read, write, edit files
- **Communication**: Email, Slack, messaging APIs
- **Computation**: Calculators, data analysis, scientific computing
- **Browser/Web**: Navigate pages, fill forms, extract content

**Function Calling Protocols:**
- OpenAI function calling / tool use format
- Anthropic tool use format
- Open-source: Gorilla, ToolBench, API-Bank formats
- MCP (Model Context Protocol) by Anthropic -- standardized tool integration protocol

### 5. Action Execution

The agent's ability to affect its environment.

**Action Types:**
- **Internal actions**: Reasoning, planning, updating memory (no external side effects)
- **External actions**: Tool calls, code execution, API requests (affect the environment)
- **Communication actions**: Responding to the user, asking clarifying questions

### 6. Observation / Feedback Loop

The mechanism by which the agent perceives the results of its actions and the state of the environment.

**The Agent Loop:**
```
User Goal -> [Plan] -> [Act] -> [Observe] -> [Reason] -> [Act] -> ... -> [Final Answer]
```

**Observation Sources:**
- Tool/function return values
- Code execution output (stdout, stderr, return values)
- Environment state changes
- User feedback and corrections
- Self-evaluation (the LLM critiquing its own output)

**Feedback Integration:**
- Observations are appended to the agent's context
- The agent reasons about whether the observation satisfies the current subgoal
- If not, the agent may: retry, try a different approach, ask for help, or revise the plan

## Formulation

**Generic Agent Loop (Pseudocode):**

```python
def agent_loop(goal, tools, max_steps):
    memory = initialize_memory()
    plan = llm.plan(goal)
    
    for step in range(max_steps):
        # Reason about current state
        thought = llm.reason(goal, plan, memory)
        
        # Decide on action
        action = llm.select_action(thought, tools)
        
        if action.type == "finish":
            return action.result
        
        # Execute action
        observation = execute(action, tools)
        
        # Update memory
        memory.add(thought, action, observation)
        
        # Optionally revise plan
        if should_replan(observation, plan):
            plan = llm.replan(goal, memory)
    
    return "Max steps reached"
```

## Variants

- **Single-agent systems**: One LLM controller handling everything
- **Multi-agent systems**: Multiple specialized agents collaborating
- **Human-in-the-loop agents**: Agents that request human approval for critical actions
- **Autonomous agents**: Fully autonomous with minimal human oversight (e.g., AutoGPT, Devin)
- **Conversational agents**: Agents embedded in chat interfaces with interactive feedback

## References

- Xi et al., "The Rise and Potential of Large Language Model Based Agents: A Survey" (2023), arXiv:2309.07864 -- Fudan University comprehensive survey
- Wang et al., "A Survey on Large Language Model based Autonomous Agents" (2023), arXiv:2308.11432 -- Renmin University survey
- Weng, Lilian, "LLM Powered Autonomous Agents" (2023), lilianweng.github.io -- Influential blog post
- Sumers et al., "Cognitive Architectures for Language Agents" (CoALA, 2023), arXiv:2309.02427
- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2022), arXiv:2210.03629
- Shinn et al., "Reflexion: Language Agents with Verbal Reinforcement Learning" (2023), arXiv:2303.11366
