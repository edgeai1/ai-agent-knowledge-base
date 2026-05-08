# Reflexion: Language Agents with Verbal Reinforcement Learning

## Metadata
- **Title**: Reflexion: Language Agents with Verbal Reinforcement Learning
- **Authors**: Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik R. Narasimhan, Shunyu Yao
- **Venue**: NeurIPS 2023
- **Year**: 2023
- **URL**: https://arxiv.org/abs/2303.11366
- **Code**: https://github.com/noahshinn/reflexion
- **Tags**: [self-improvement, reflection, memory, reinforcement-learning, foundational]

---

## TL;DR

Reflexion enables LLM agents to learn from failures by generating verbal self-reflections stored in an episodic memory buffer, achieving substantial improvements on sequential decision-making, coding, and reasoning tasks without any weight updates.

---

## Motivation & Problem

Traditional reinforcement learning requires expensive gradient-based optimization over massive numbers of environment interactions. LLM-based agents using techniques like ReAct can perform multi-step reasoning and acting, but they lack a mechanism to **learn from mistakes across trials**. When an agent fails at a task, it simply retries with no structured memory of what went wrong.

Existing approaches either:
1. **Fine-tune models** -- expensive, requires differentiable reward signals, and changes model capabilities globally.
2. **Simple retry** -- re-runs the agent with no accumulated knowledge, relying on sampling variance.
3. **Self-Refine** (Madaan et al., 2023) -- iterative refinement within a single generation episode, but no persistent memory across separate trial episodes.

The core question: **Can an LLM agent improve its behavior over successive trials using only natural language feedback, without any parameter updates?**

This connects to cognitive science concepts of metacognition and reflective practice -- humans improve through deliberate reflection on failures, not just repeated exposure.

---

## Method

### Core Architecture

Reflexion introduces three distinct components working in a trial loop:

```
+------------------+     +------------------+     +-------------------+
|                  |     |                  |     |                   |
|   Actor Agent    |---->|   Evaluator      |---->|  Self-Reflection  |
|  (LLM + ReAct)  |     | (heuristic/LLM)  |     |    Generator      |
|                  |     |                  |     |                   |
+------------------+     +------------------+     +-------------------+
        ^                                                  |
        |                                                  |
        |            +---------------------+               |
        +------------|  Episodic Memory    |<--------------+
                     |  (reflection buffer)|
                     +---------------------+
```

1. **Actor**: An LLM-based agent (e.g., ReAct-style) that generates actions and interacts with an environment. At each trial, the actor receives the task description plus the episodic memory containing reflections from prior failed attempts.

2. **Evaluator**: Computes a scalar or binary reward signal after a trajectory is completed. Can be a heuristic function (exact match for QA, unit test pass/fail for code, task completion for AlfWorld) or an LLM-based evaluator.

3. **Self-Reflection Model**: An LLM that takes as input the failed trajectory, the reward signal, and the current memory, producing a natural language reflection describing what went wrong and how to improve.

### Reflexion Algorithm -- Pseudocode

```
Algorithm: REFLEXION
-------------------------------------------------------
Input:  Environment E, Actor A (LLM agent),
        Evaluator Eval, Self-Reflector SR,
        Max trials T
Output: Successful trajectory or best trajectory

1:  mem <- []                              // episodic memory buffer
2:  for t = 1 to T do
3:      tau_t <- A.run(E, mem)             // execute actor with memory context
4:      r_t   <- Eval(tau_t, E.ground_truth)  // evaluate trajectory
5:      if r_t == SUCCESS then
6:          return tau_t                   // task solved, exit
7:      end if
8:      sr_t  <- SR.reflect(tau_t, r_t, mem)  // generate verbal reflection
9:      mem   <- mem + [sr_t]             // append to episodic memory
10:     if |mem| > K then                 // sliding window (K=3 typical)
11:         mem <- mem[-K:]               // keep most recent K reflections
12:     end if
13: end for
14: return argmax_{tau_t} Eval(tau_t)     // return best trajectory
```

### Self-Reflection Prompt Design

The self-reflection prompt is structured to elicit both **diagnosis** and **prescription**:

```
SYSTEM: You are an advanced reasoning agent that can improve
based on self-reflection.

USER: You will be given a previous reasoning trial in which
you were given access to tools and a question to answer. You
were unsuccessful in answering the question either because
you guessed the wrong answer with Finish[<answer>], or you
used up your set number of reasoning steps.

In a few sentences, diagnose a possible reason for failure
and devise a new, concise, high-level plan that aims to
mitigate the same failure. Use complete sentences.

Previous trial:
{full_trajectory_text}

[Previous reflections from memory, if any:]
{memory_contents}

Reflection:
```

Key design choices in the prompt:
- Reflections are kept **concise** (2-3 sentences) to avoid context window bloat
- The prompt explicitly requests both **diagnosis** (what went wrong) and **prescription** (what to do differently)
- Prior reflections are included so the new reflection builds on accumulated understanding
- Different prompt variants are used for different task types (QA, code, embodied)

### Episodic Memory Buffer Implementation

The buffer `mem` is a simple ordered list of reflection strings. At each new trial, buffer contents are injected into the actor's system prompt:

```
You have attempted this task before. Here are your reflections
on previous attempts:

Trial 1: I searched for "population of France" but should have
searched for "population of Paris" specifically. I need to
decompose the question more carefully before searching.

Trial 2: I found the correct population but made an arithmetic
error when computing the percentage. I should double-check all
calculations before submitting my final answer.

Now attempt the task again, incorporating these insights.
```

The buffer uses a **sliding window** (K=3 typically) -- older reflections are discarded FIFO. This is both a practical constraint (context length) and an empirical finding (K=3 outperforms unbounded).

### Task-Specific Adaptations

- **AlfWorld**: Actor uses ReAct with action space {go to, open, take, put, use, look}. Evaluator is binary task completion. Reflections focus on action sequencing errors.
- **HotPotQA**: Actor uses ReAct with Search/Lookup tools. Evaluator is exact match. Reflections address search strategy and reasoning chain errors.
- **HumanEval**: Actor generates Python code. Evaluator runs unit tests and returns pass/fail plus stderr. Reflections incorporate specific error messages and failing test cases.

---

## Key Innovations

1. **Verbal reinforcement learning**: Replaces scalar reward signals with rich natural language reflections, enabling nuanced credit assignment (e.g., "I failed because I searched the wrong entity" vs. reward = 0).
2. **Persistent episodic memory**: Unlike single-episode Self-Refine, reflections accumulate across completely separate trial episodes.
3. **Decoupled evaluation and reflection**: The evaluator provides the binary/scalar signal; the reflector provides the explanation. This allows using cheap heuristic evaluators while still generating rich feedback.
4. **No parameter updates**: All "learning" is in-context through the memory buffer, immediately applicable to any frozen LLM without fine-tuning infrastructure.

---

## Experimental Setup

### Benchmarks and Configurations

| Benchmark | Task Type | # Tasks | Agent Base | Evaluator | Max Trials |
|-----------|-----------|---------|------------|-----------|------------|
| AlfWorld  | Embodied decision-making (6 types) | 134 | ReAct | Heuristic (task success) | 12 |
| HotPotQA  | Multi-hop QA | 100 | ReAct (CoT + Act) | Exact match | 7 |
| HumanEval | Code generation (Python) | 164 | Direct generation | Unit tests (pass@1) | 11 |

### Models Used
- **Primary actor/reflector**: GPT-4, GPT-3.5-turbo
- **Baselines**: ReAct (single trial), ReAct + simple retry, Chain of Hindsight, Self-Refine
- **Ablation models**: text-davinci-003

---

## Results

### AlfWorld (Embodied Decision-Making)

| Method | Success Rate (%) |
|--------|:----------------:|
| ReAct (1 trial) | 75 |
| ReAct (retry, no reflection) | 85 |
| Reflexion (trial 2) | 91 |
| Reflexion (trial 3) | 94 |
| **Reflexion (trial 12)** | **97** |
| AdaPlanner | 94 |

Per-task breakdown at convergence (trial 12):
| Task Type | Success (%) | Avg Trials to Solve |
|-----------|:-----------:|:-------------------:|
| Pick      | 100         | 1.3                 |
| Clean     | 95          | 2.1                 |
| Heat      | 100         | 1.5                 |
| Cool      | 100         | 1.8                 |
| Examine   | 95          | 2.4                 |
| Pick Two  | 92          | 3.2                 |

Most tasks converge within 3-4 trials. Remaining failures involve complex multi-object interactions in "Pick Two" tasks.

### HotPotQA (Multi-hop Reasoning)

| Method | EM (%) |
|--------|:------:|
| CoT (1 trial) | 29 |
| ReAct (1 trial) | 34 |
| CoT + Reflexion (trial 2) | 47 |
| CoT + Reflexion (trial 3) | 54 |
| CoT + Reflexion (trial 5) | 63 |
| **CoT + Reflexion (trial 7)** | **68** |
| Chain-of-Hindsight | 43 |

Convergence: ~80% of final improvement reached by trial 3. Diminishing returns after trial 5, with only 5 additional percentage points gained in trials 5-7.

### HumanEval (Code Generation)

| Method | Pass@1 (%) |
|--------|:----------:|
| GPT-4 (baseline, zero-shot) | 80.1 |
| GPT-4 + simple retry | 83.5 |
| GPT-4 + Self-Refine | 84.1 |
| **GPT-4 + Reflexion** | **91.0** |
| GPT-3.5 (baseline) | 48.1 |
| GPT-3.5 + Reflexion | 68.1 |
| CodeT (GPT-3.5) | 65.8 |

Code tasks benefit most from Reflexion because unit test stderr provides highly informative evaluator feedback (stack traces, assertion errors, type errors).

### Ablation: Memory Buffer Size

| Configuration | HotPotQA EM (%) | HumanEval Pass@1 (%) |
|---------------|:---------------:|:--------------------:|
| Full Reflexion (K=3) | 68 | 91.0 |
| No reflection (retry only) | 43 | 83.5 |
| Reflection, no memory (K=0, single-trial refinement) | 51 | 86.2 |
| K=1 (last reflection only) | 62 | 89.0 |
| K=3 (last 3 reflections) | **68** | **91.0** |
| Unbounded (all reflections) | 66 | 90.5 |

K=3 is optimal. Unbounded memory causes slight degradation likely due to context pollution with stale, potentially contradictory reflections.

---

## Analysis & Insights

1. **Reflection quality > quantity**: The K=3 window outperforming unbounded memory shows that concise, relevant, recent reflections are more valuable than exhaustive history.

2. **Failure mode taxonomy**: Reflections implicitly categorize failures -- search strategy errors, reasoning errors, hallucination, action sequencing -- enabling targeted corrections in subsequent trials.

3. **Evaluator informativeness drives improvement**: Reflexion gains are largest for code (unit test feedback >> binary success), moderate for QA (exact match reveals wrong answers), and smallest where feedback is least informative.

4. **Diminishing returns**: ~80% of total improvement is achieved in the first 2-3 trials across all benchmarks, indicating easy failures are corrected first while harder structural issues persist.

5. **Not true learning**: Since no weights are updated, "learning" is bounded by context length and resets for every new task. Each task starts with an empty buffer.

---

## Limitations & Critiques

1. **No cross-task transfer**: Reflections are task-specific. The agent cannot learn a general principle (e.g., "always decompose multi-hop questions") and carry it to new tasks.
2. **Inference cost**: Each trial requires full actor inference + reflection generation. 7 HotPotQA trials = ~14x single-trial cost.
3. **Evaluator dependency**: Quality of reflections is bottlenecked by evaluator informativeness. Binary signals yield vague reflections; rich error messages yield actionable ones.
4. **Reflection hallucination**: The self-reflection model can produce plausible but incorrect diagnoses, sending the actor down wrong paths. No verification mechanism exists.
5. **Episodic structure required**: Reflexion assumes clean trial-reset-retry structure; it does not naturally apply to open-ended, continuous agent operation.
6. **Context window constraints**: The K=3 sliding window is a practical compromise, not necessarily optimal. Longer-context models might enable richer memory.

---

## Follow-up Work

- **Retroformer** (Yao et al., 2023): Fine-tunes a retrospective model to generate better reflections over time.
- **LATS** (Zhou et al., 2023): Combines reflection with Monte Carlo Tree Search for structured exploration.
- **ExpeL** (Zhao et al., 2023): Extracts cross-task generalizable insights from experience, addressing the no-transfer limitation.
- **Self-Discover** (Zhou et al., 2024): Uses self-reflection to discover task-specific reasoning structures.
- **Coding agents** (SWE-Agent, Devin, OpenHands): The reflection paradigm is widely adopted where unit test feedback provides natural evaluator signals.
- **Reflexion + long-context**: With models supporting 128K+ tokens, the sliding window constraint may be relaxed in future work.

---

## Key Takeaways

1. Natural language self-reflection is a powerful alternative to scalar rewards for in-context agent learning -- "verbal RL."
2. A small sliding window of recent reflections (K=3) outperforms both single-reflection and unbounded memory accumulation.
3. The approach is most effective when evaluator feedback is rich and structured (code >> QA >> embodied).
4. Reflexion achieves 91.0% pass@1 on HumanEval and 97% on AlfWorld, demonstrating broad applicability.
5. The key engineering decision is **reflection prompt design** -- the quality of the self-reflection prompt directly determines the quality of accumulated experience.
