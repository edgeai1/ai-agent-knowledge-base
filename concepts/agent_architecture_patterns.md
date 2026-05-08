---
title: Agent Architecture Patterns
tags: [agent, architecture, react, plan-and-execute, reflexion, multi-agent, design-pattern]
related: [llm_agent_fundamentals, agent_techniques, agent_frameworks]
---

## Definition

Agent architecture patterns are recurring structural designs that define how an LLM agent organizes its reasoning, planning, action, and feedback cycles. Each pattern represents a different tradeoff between autonomy, accuracy, efficiency, and complexity.

---

## Pattern 1: ReAct (Reasoning + Acting)

### Origin
Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (ICLR 2023), arXiv:2210.03629

### Key Insight
Interleave chain-of-thought reasoning traces with concrete actions in a single prompt/loop, rather than separating reasoning and acting into independent phases. The reasoning trace helps the model plan and interpret results; the actions ground the reasoning in real observations.

### How It Works

```
Loop:
  1. Thought: The agent reasons about the current state and what to do next
  2. Action: The agent selects and executes a tool/action
  3. Observation: The result of the action is returned to the agent
  Repeat until the agent decides to finish
```

### Example Trace
```
Question: What is the elevation of the birthplace of the inventor of the telephone?

Thought 1: I need to find who invented the telephone, then find their birthplace,
            then find the elevation of that place.
Action 1: Search("inventor of the telephone")
Observation 1: Alexander Graham Bell is credited with inventing the telephone.

Thought 2: Alexander Graham Bell was the inventor. Now I need to find his birthplace.
Action 2: Search("Alexander Graham Bell birthplace")
Observation 2: Alexander Graham Bell was born in Edinburgh, Scotland.

Thought 3: His birthplace is Edinburgh. Now I need the elevation.
Action 3: Search("Edinburgh Scotland elevation")
Observation 3: Edinburgh has an elevation of approximately 47 meters (154 ft).

Thought 4: I now have all the information needed.
Action 4: Finish("The elevation of Edinburgh, the birthplace of Alexander Graham Bell,
          is approximately 47 meters (154 ft).")
```

### Strengths
- Simple and intuitive
- Reasoning traces improve interpretability
- Grounded reasoning reduces hallucination (actions provide real data)
- Works well for question answering, fact-checking, interactive tasks

### Limitations
- Sequential execution (one action at a time) can be slow
- Can get stuck in loops if reasoning is poor
- No explicit plan -- reactive rather than proactive
- Context window fills up quickly with long traces

### Variants
- **ReAct with self-consistency**: Run multiple ReAct traces and take the majority answer
- **ReWOO (Reasoning WithOut Observation)**: Plan all actions first, execute in batch, then reason over all observations (Xu et al., 2023)
- **LATS (Language Agent Tree Search)**: Combine ReAct with Monte Carlo Tree Search for exploration

---

## Pattern 2: Plan-and-Execute

### Origin
Inspired by classical AI planning; formalized for LLMs in Wang et al., "Plan-and-Solve Prompting" (2023) and implemented in frameworks like LangGraph's Plan-and-Execute agent.

### Key Insight
Separate planning from execution. First, generate a complete plan (list of steps), then execute each step sequentially, optionally re-planning after each step based on results.

### How It Works

```
Phase 1 -- Planning:
  Input: User goal
  Output: Ordered list of steps [Step1, Step2, ..., StepN]

Phase 2 -- Execution:
  For each step in plan:
    Execute step (may involve ReAct-style sub-loops)
    Observe result
    Optionally: re-plan remaining steps based on new information

Phase 3 -- Synthesis:
  Combine results from all steps into final answer
```

### Architecture Diagram
```
                    +-------------+
  User Goal ------->|   Planner   |-----> [Step1, Step2, Step3, ...]
                    |   (LLM)     |              |
                    +-------------+              |
                          ^                      v
                          |              +---------------+
                   Re-plan if needed     |   Executor    |
                          |              | (LLM + Tools) |
                          ^              +---------------+
                          |                      |
                    +-------------+              v
                    |  Replanner  |<----- Observation/Result
                    +-------------+
```

### Strengths
- Better for complex multi-step tasks
- Explicit plan is inspectable and modifiable (human-in-the-loop friendly)
- Planner and executor can use different models (e.g., strong model plans, weaker model executes)
- Can parallelize independent steps
- Re-planning allows adaptation to unexpected results

### Limitations
- Initial plan may be suboptimal without knowing execution results
- Overhead of planning step for simple tasks
- Re-planning adds latency and cost
- Quality depends heavily on the planner's ability to decompose tasks

### Variants
- **Static Plan-and-Execute**: Plan once, execute all steps without re-planning
- **Dynamic Plan-and-Execute**: Re-plan after every step
- **Hierarchical Plan-and-Execute**: Plans contain sub-plans (recursive decomposition)
- **Plan-and-Solve (Zero-shot)**: Add "Let's first understand the problem and devise a plan" to the prompt

---

## Pattern 3: Reflexion / Self-Reflection

### Origin
Shinn et al., "Reflexion: Language Agents with Verbal Reinforcement Learning" (NeurIPS 2023), arXiv:2303.11366

### Key Insight
After completing a task (or failing), have the agent reflect on its performance, generate verbal feedback about what went wrong, and use that reflection as additional context for the next attempt. This creates a form of "verbal reinforcement learning" without weight updates.

### How It Works

```
Outer Loop (Trials):
  For each trial t = 1, 2, ..., T:
    
    Inner Loop (Episode):
      Run the agent (e.g., using ReAct) to attempt the task
      Receive evaluation signal (success/failure, score, test results)
    
    Reflection:
      If task failed or suboptimal:
        Generate verbal reflection: "What went wrong? What should I do differently?"
        Store reflection in memory
    
    Next trial uses: original task + all previous reflections
    
    If task succeeded: return result
```

### Example
```
Trial 1:
  Task: Write a function to find the longest palindromic substring
  Agent writes code -> Tests fail (wrong output for "babad")
  
  Reflection: "My approach of checking all substrings was correct but I had an
  off-by-one error in the loop bounds. I also didn't handle the case where
  multiple palindromes of the same length exist. Next time, I should test edge
  cases more carefully and use expand-around-center approach."

Trial 2:
  Task: [same] + Reflection from Trial 1
  Agent writes improved code -> Tests pass
```

### Memory Components
- **Trajectory memory**: The sequence of actions/observations from each trial
- **Reflection memory**: Verbal self-critiques and lessons learned
- **Evaluation signal**: Binary (pass/fail) or scalar (test pass rate, score)

### Strengths
- Learns from mistakes without fine-tuning
- Reflections are interpretable (we can read what the agent learned)
- Compatible with any base agent architecture (ReAct, Plan-and-Execute, etc.)
- Effective for coding tasks, decision-making, and reasoning

### Limitations
- Requires multiple trials (slower and more expensive)
- Needs a clear evaluation signal (hard for open-ended tasks)
- Limited by the LLM's ability to accurately self-diagnose errors
- Reflections can be superficial or wrong (garbage-in-garbage-out)

### Related Patterns
- **Self-Refine** (Madaan et al., 2023): Iteratively refine output using self-feedback
- **CRITIC** (Gou et al., 2023): Use tools to verify and critique LLM outputs
- **Introspective Agents**: Agents that maintain explicit beliefs about their own capabilities

---

## Pattern 4: Multi-Agent Collaboration

### Origin
Multiple works: CAMEL (Li et al., 2023), AutoGen (Wu et al., 2023), MetaGPT (Hong et al., 2023), ChatDev (Qian et al., 2023), Generative Agents (Park et al., 2023)

### Key Insight
Instead of one monolithic agent, use multiple specialized agents that communicate, debate, or collaborate to solve complex tasks. Each agent can have different roles, expertise, prompts, tools, and even different underlying models.

### Collaboration Topologies

#### 4a. Sequential Pipeline (Chain)
```
Agent A -> Agent B -> Agent C -> Final Output
(Research)  (Write)   (Review)
```
- Each agent handles one phase
- Output of one agent is input to the next
- Simple, predictable, easy to debug

#### 4b. Hierarchical (Manager-Worker)
```
        +----------+
        | Manager  |
        |  Agent   |
        +----+-----+
             |
    +--------+--------+
    |        |        |
+---v--+ +---v--+ +---v--+
|Worker| |Worker| |Worker|
|  A   | |  B   | |  C   |
+------+ +------+ +------+
```
- Manager decomposes tasks and delegates to workers
- Workers report results back to manager
- Manager synthesizes and coordinates
- Examples: MetaGPT's software company simulation, CrewAI's crew structure

#### 4c. Debate / Adversarial
```
Agent A <-----> Agent B
(Proposer)     (Critic)
     \          /
      v        v
   +------------+
   |   Judge    |
   +------------+
```
- Agents argue different positions or critique each other's work
- A judge agent or mechanism selects the best result
- Improves accuracy through diverse perspectives
- Example: "Society of Mind" approach, LLM debate for factuality

#### 4d. Peer Collaboration (Flat)
```
Agent A <---> Agent B
  ^    \   /    ^
  |     \ /     |
  v      X      v
Agent C <---> Agent D
```
- All agents are peers, communicating freely
- Shared message bus or blackboard architecture
- Most flexible but hardest to coordinate
- Example: Generative Agents (Stanford's virtual town)

#### 4e. Voting / Ensemble
```
Agent A ---\
Agent B ----+---> Aggregation --> Final Answer
Agent C ---/
```
- Multiple agents independently solve the same problem
- Results are aggregated (majority vote, best-of-N, etc.)
- Improves reliability through redundancy

### Communication Protocols
- **Natural language messages**: Most common; agents communicate via text
- **Structured messages**: JSON, function calls, typed interfaces
- **Shared memory/blackboard**: Agents read/write to a shared knowledge base
- **Event-driven**: Agents subscribe to events and react asynchronously

### Multi-Agent Design Patterns (from practice)

| Pattern | Description | Use Case |
|---------|-------------|----------|
| Supervisor | One agent routes tasks to specialists | Customer service, triage |
| Swarm | Agents hand off control based on context | Complex workflows with branching |
| Map-Reduce | Parallel processing + aggregation | Document analysis, research |
| Reflection pair | Generator + critic/reviewer | Code generation, writing |
| Panel discussion | Multiple experts discuss | Complex analysis, decision-making |

### Strengths
- Specialization: each agent can be optimized for its role
- Scalability: add more agents for more capabilities
- Robustness: failure of one agent doesn't kill the system
- Emergent behavior: complex outcomes from simple agents
- Natural mapping to organizational structures

### Limitations
- Communication overhead (token cost, latency)
- Coordination complexity (deadlocks, infinite loops, conflicting actions)
- Harder to debug and test
- Potential for echo chambers or groupthink
- More expensive (multiple LLM calls)

---

## Pattern Comparison Matrix

| Pattern | Complexity | Best For | Latency | Cost | Interpretability |
|---------|-----------|----------|---------|------|-----------------|
| ReAct | Low | Simple tool-use tasks, QA | Low-Med | Low | High |
| Plan-and-Execute | Medium | Multi-step tasks, complex workflows | Medium | Medium | High |
| Reflexion | Medium | Tasks with clear success criteria | High | High | Very High |
| Multi-Agent | High | Complex projects, specialized domains | High | High | Medium |

## References

- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2023), ICLR
- Wang et al., "Plan-and-Solve Prompting" (2023), ACL
- Shinn et al., "Reflexion: Language Agents with Verbal Reinforcement Learning" (2023), NeurIPS
- Wu et al., "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation" (2023)
- Hong et al., "MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework" (2023)
- Li et al., "CAMEL: Communicative Agents for 'Mind' Exploration of Large Language Model Society" (2023)
- Park et al., "Generative Agents: Interactive Simulacra of Human Behavior" (2023)
- Xu et al., "ReWOO: Decoupling Reasoning from Observations for Efficient Augmented Language Models" (2023)
- Zhou et al., "Language Agent Tree Search Unifies Reasoning, Acting, and Planning in Language Models" (2023)
- Madaan et al., "Self-Refine: Iterative Refinement with Self-Feedback" (2023)
