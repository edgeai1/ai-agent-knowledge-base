---
title: "ReAct: Synergizing Reasoning and Acting in Language Models"
authors:
  - Shunyu Yao
  - Jeffrey Zhao
  - Dian Yu
  - Nan Du
  - Izhak Shafran
  - Karthik Narasimhan
  - Yuan Cao
venue: ICLR 2023 (Oral)
year: 2023
submitted: 2022-10-06
url: https://arxiv.org/abs/2210.03629
code: https://github.com/ysymyth/ReAct
tags:
  - agents
  - reasoning
  - acting
  - prompting
  - language-models
  - tool-use
  - grounded-reasoning
  - decision-making
  - foundational
status: done
---

# ReAct: Synergizing Reasoning and Acting in Language Models

## TL;DR

ReAct introduces a prompting paradigm that interleaves **reasoning traces** (Thoughts)
and **task-specific actions** (Actions) within a single language model generation loop.
By letting the LLM think _and_ act in alternation -- retrieving external information
via APIs or manipulating environments -- ReAct overcomes hallucination problems of
pure chain-of-thought reasoning and the lack of planning in action-only agents. It
achieves strong results on knowledge-intensive QA (HotpotQA), fact verification
(FEVER), and interactive decision-making benchmarks (ALFWorld, WebShop) using only
1-6 in-context examples with PaLM-540B, outperforming both reasoning-only and
action-only baselines.

---

## Motivation & Problem

### Core Insight from Cognitive Science

Human intelligence seamlessly integrates task-oriented **acting** (e.g., picking up
an object, typing a query) with **reasoning** (e.g., planning, adjusting strategy,
verifying facts). Prior to ReAct, LLM research treated these as separate tracks:

- **Reasoning-only** (e.g., Chain-of-Thought): The model generates an internal
  reasoning trace but cannot access external information, leading to hallucination
  and error propagation in knowledge-intensive tasks.
- **Action-only** (e.g., WebGPT, SayCan): The model issues actions to interact
  with environments or tools but lacks explicit reasoning about _why_ it takes
  those actions, leading to poor planning and inability to recover from errors.

### Problem Statement

Can we design a single prompting framework that allows an LLM to generate both
**verbal reasoning traces** and **grounded actions** in an interleaved manner, so
that reasoning informs action selection and actions provide grounding for reasoning?

### Key Hypotheses

1. Reasoning traces help the model **induce, track, and update action plans** as
   well as **handle exceptions** (e.g., when a search returns irrelevant results).
2. Actions allow the model to **interface with external sources** (knowledge bases,
   environments) to gather additional information that reasoning alone cannot provide.
3. The synergy between reasoning and acting yields trajectories that are more
   **interpretable**, **diagnosable**, and **controllable** than either alone.

---

## Method

### The ReAct Prompting Format

ReAct augments the action space of an agent with a **thought space**. At each step,
the model can choose to either produce a *Thought* (free-form reasoning) or an
*Action* (task-specific command). After each Action, the environment returns an
*Observation*.

```
Thought-Action-Observation Loop:
================================================

Question: <task/question>

Thought 1: <reasoning about what to do>
Action  1: <tool/environment action>
Observation 1: <result from environment>

Thought 2: <reasoning about observation>
Action  2: <next action based on reasoning>
Observation 2: <result from environment>

...

Thought n: <final reasoning>
Action  n: Finish[<answer>]
```

### Concrete Example: HotpotQA

```
Question: What is the elevation range for the area that the eastern
          sector of the Colorado orogeny extends into?

Thought 1: I need to search Colorado orogeny, find the area that the
           eastern sector extends into, then find the elevation range
           of that area.
Action  1: Search[Colorado orogeny]
Observation 1: The Colorado orogeny was an episode of mountain building
               (an orogeny) in Colorado and surrounding areas.

Thought 2: It does not mention the eastern sector. So I need to look
           up eastern sector.
Action  2: Lookup[eastern sector]
Observation 2: (Result 1 / 1) The eastern sector extends into the
               High Plains and is called the Central Plains orogeny.

Thought 3: The eastern sector of the Colorado orogeny extends into
           the High Plains. I need to search High Plains and find
           its elevation range.
Action  3: Search[High Plains]
Observation 3: High Plains refers to one of two distinct land regions:
               the High Plains (United States)... elevation from
               around 1,800 to 7,000 ft (550 to 2,130 m).

Thought 4: High Plains rise in elevation from around 1,800 to 7,000 ft,
           so the answer is 1,800 to 7,000 ft.
Action  4: Finish[1,800 to 7,000 ft]
```

### Available Actions by Task

```
+------------------+-----------------------------------------------+
| Task             | Action Space                                  |
+------------------+-----------------------------------------------+
| HotpotQA / FEVER | Search[entity] - find Wikipedia page           |
|                  | Lookup[keyword] - search within current page   |
|                  | Finish[answer]  - return final answer           |
+------------------+-----------------------------------------------+
| ALFWorld         | go to {receptacle}                             |
|                  | take {object} from {receptacle}                |
|                  | put {object} in/on {receptacle}                |
|                  | open/close {receptacle}                        |
|                  | toggle {object}                                |
|                  | clean/heat/cool {object} with {receptacle}     |
|                  | use {object}                                   |
|                  | examine {object/receptacle}                    |
|                  | look                                           |
+------------------+-----------------------------------------------+
| WebShop          | search[query]                                  |
|                  | click[element]                                 |
|                  | think[reasoning]                               |
+------------------+-----------------------------------------------+
```

### Algorithmic Framework (Pseudocode)

```
Algorithm: ReAct Agent Loop
==========================================
Input: question q, action space A, environment E, LM theta
Output: answer a

prompt <- load_few_shot_exemplars()  // 1-6 human-written trajectories
trajectory <- [("Question", q)]

for t = 1, 2, ... , T_max do:
    // Generate next thought or action
    output <- LM_theta(prompt + trajectory)

    if output is Thought:
        trajectory.append(("Thought t", output))

    else if output is Action:
        trajectory.append(("Action t", output))

        if output == "Finish[answer]":
            return answer

        // Execute action in environment
        observation <- E.execute(output)
        trajectory.append(("Observation t", observation))

    // Halt if trajectory exceeds max steps
    if t >= T_max:
        return FAIL
```

### Architecture Diagram

```
                    +-------------------+
                    |    LLM (PaLM)     |
                    |   (frozen, no     |
                    |    fine-tuning)    |
                    +--------+----------+
                             |
                    +--------v----------+
               +--->| Generate Thought  |---+
               |    | or Action         |   |
               |    +-------------------+   |
               |             |              |
               |    +--------v----------+   |
               |    |  Is it a Thought? |   |
               |    +---+----------+----+   |
               |        |          |        |
               |       Yes         No       |
               |        |          |        |
               |        v          v        |
               |    [Append to   [Execute   |
               |     context]    Action in  |
               |        |        Env/API]   |
               |        |          |        |
               |        |    +-----v-----+  |
               |        |    |Observation |  |
               |        |    |from Env    |  |
               |        |    +-----+------+  |
               |        |          |         |
               |        +----+-----+         |
               |             |               |
               |    +--------v----------+    |
               |    |  Finish[answer]?  |    |
               |    +---+----------+----+    |
               |        |          |         |
               |       Yes         No        |
               |        |          |         |
               |        v          +---------+
               |    [Return answer]
               |
               +--- [Loop continues]
```

### Comparison: ReAct vs. Other Prompting Paradigms

```
Standard Prompting:   Question -> Answer
Chain-of-Thought:     Question -> Thought_1 -> ... -> Thought_n -> Answer
Act-only:             Question -> Action_1 -> Obs_1 -> ... -> Action_n -> Answer
ReAct:                Question -> Thought_1 -> Action_1 -> Obs_1 ->
                                  Thought_2 -> Action_2 -> Obs_2 ->
                                  ... -> Thought_n -> Finish[Answer]
```

---

## Key Innovations

1. **Unified Reasoning-Acting Framework**: First work to show that interleaving
   free-form reasoning traces with grounded actions within a single LLM generation
   loop yields synergistic benefits over either alone.

2. **Thought as a Flexible Control Mechanism**: Thoughts serve multiple purposes
   -- decomposing goals into subgoals, tracking progress, handling exceptions,
   extracting relevant info from observations, and commonsense reasoning about
   when to switch strategies.

3. **Minimal Supervision**: Only 1-6 human-written exemplar trajectories are
   needed as in-context demonstrations (no training data, no fine-tuning for
   the prompting setup).

4. **Human Interpretability**: The explicit thought traces make the agent's
   reasoning transparent and diagnosable, unlike opaque action-only agents.

5. **Synergy Mechanism**: Acting addresses the hallucination and knowledge
   limitations of reasoning-only approaches; reasoning addresses the lack of
   planning and state-tracking in action-only approaches.

6. **Task Generality**: The same framework applies to both knowledge-intensive
   NLP tasks (QA, fact verification) and interactive decision-making tasks
   (text games, web navigation) with only the action space changing.

---

## Experimental Setup

### Tasks and Datasets

| Task | Dataset | Metric | # Eval | Domain |
|------|---------|--------|--------|--------|
| Multi-hop QA | HotpotQA (Yang et al. 2018) | Exact Match (EM), F1 | 500 (random dev) | Wikipedia |
| Fact Verification | FEVER (Thorne et al. 2018) | Label Accuracy | 500 (random dev) | Wikipedia |
| Text Game | ALFWorld (Shridhar et al. 2021) | Success Rate | 134 (unseen games) | Household |
| Web Navigation | WebShop (Yao et al. 2022) | Success Rate, Reward | 500 (test) | E-commerce |

### Models

- **Primary**: PaLM-540B (Chowdhery et al. 2022), used for all prompting experiments
- **Finetuning**: PaLM-8B, PaLM-62B, PaLM-540B for scaling analysis

### Baselines Compared

| Method | Description |
|--------|-------------|
| Standard | Direct answer prompting (no reasoning, no action) |
| CoT (Chain-of-Thought) | Reasoning traces only, no external actions |
| CoT-SC (Self-Consistency) | CoT with majority voting over k=21 samples |
| Act | Actions only, no reasoning thoughts |
| ReAct | Interleaved thoughts + actions (proposed) |
| ReAct -> CoT-SC | ReAct with fallback to CoT-SC on failure |
| CoT-SC -> ReAct | CoT-SC with fallback to ReAct on low confidence |

### Hyperparameters and Setup

- **Prompting**: 3-6 in-context exemplars for knowledge tasks; 1-2 for decision tasks
- **Decoding**: Greedy decoding for ReAct (temperature=0); sampling for CoT-SC
- **Wikipedia API**: Simple interface with Search[entity] and Lookup[keyword]
- **ALFWorld**: 6 task types, 1 in-context example per task type
- **WebShop**: 1-shot prompting with one exemplar trajectory

---

## Results

### Table 1: Knowledge-Intensive Reasoning (PaLM-540B)

```
+---------------------+-------------------------+-------------------------+
|                     |       HotpotQA          |         FEVER           |
| Method              |   EM    |     F1        |    Accuracy             |
+---------------------+---------+---------------+-------------------------+
| Standard            |  27.1   |      --       |      --                 |
| CoT (n=1)           |  29.4   |      --       |     56.3                |
| CoT-SC (n=21)       |  33.4   |      --       |     64.6                |
| Act                 |  25.7   |      --       |     58.9                |
| ReAct               |  27.4   |     35.1      |     60.9                |
| ReAct -> CoT-SC     |  35.1   |     40.1      |     64.6                |
| CoT-SC -> ReAct     |  34.2   |      --       |     65.4                |
+---------------------+---------+---------------+-------------------------+
```

**Key observations**:
- ReAct outperforms Act-only by 1.7 EM on HotpotQA and 2.0 accuracy on FEVER
- On HotpotQA, CoT (29.4) slightly outperforms ReAct (27.4) due to the
  flexibility of internal reasoning for multi-hop questions
- On FEVER, ReAct (60.9) significantly outperforms CoT (56.3) because fact
  verification benefits more from grounded evidence retrieval
- The combined approaches (ReAct -> CoT-SC, CoT-SC -> ReAct) achieve the
  best results, demonstrating complementarity of internal and external knowledge

### Table 2: Interactive Decision Making (PaLM-540B)

```
+---------------------+-------------------+-------------------------+
|                     |    ALFWorld       |       WebShop           |
| Method              | Success Rate (%)  | Success (%) | Reward    |
+---------------------+-------------------+-------------+-----------+
| BUTLER (IL, 10^5)   |      37           |     --      |    --     |
| IL (1,012 trajs)    |      --           |    29.1     |   59.9    |
| IL + RL (10,587)    |      --           |    28.7     |   62.4    |
| Act (best of 6)     |      45           |    30.1     |   62.3    |
| ReAct (best of 6)   |      71           |    40.0     |   66.6    |
+---------------------+-------------------+-------------+-----------+
```

**Key observations**:
- On ALFWorld, ReAct (71%) outperforms Act (45%) by +26% absolute and
  BUTLER (37%) by +34% absolute -- a massive improvement
- On WebShop, ReAct (40.0%) outperforms IL+RL (28.7%) by +11.3% absolute,
  despite using only 1-shot prompting vs. thousands of training trajectories
- Even the _worst_ ReAct trial (48%) on ALFWorld beats the _best_ Act trial (45%)
- Relative performance gain of ReAct over Act on ALFWorld ranges from 33% to
  90% across 6 controlled trials, averaging 62%

### ALFWorld Per-Task Breakdown

```
+------------------------+----------+----------+
| Task Type              | Act (%)  | ReAct (%)|
+------------------------+----------+----------+
| Pick & Place           |   45     |    74    |
| Examine in Light       |   50     |    83    |
| Clean & Place          |   39     |    65    |
| Heat & Place           |   74     |    91    |
| Cool & Place           |   58     |    75    |
| Pick Two & Place       |   24     |    43    |
+------------------------+----------+----------+
```

ReAct consistently outperforms Act across all 6 task categories in ALFWorld.

### Finetuning vs. Prompting (HotpotQA)

```
+---------------------+-------------+-------------+-------------+
| Method + Model      | PaLM-8B     | PaLM-62B    | PaLM-540B   |
+---------------------+-------------+-------------+-------------+
| Standard (prompt)   |    --       |    --       |   27.1      |
| CoT (prompt)        |    --       |    --       |   29.4      |
| Act (prompt)        |    --       |    --       |   25.7      |
| ReAct (prompt)      |    --       |    --       |   27.4      |
+---------------------+-------------+-------------+-------------+
| Standard (finetune) |   23.6      |   27.0      |    --       |
| CoT (finetune)      |   24.2      |   28.5      |    --       |
| Act (finetune)      |   25.3      |   28.0      |    --       |
| ReAct (finetune)    |   28.4      |   31.2      |    --       |
+---------------------+-------------+-------------+-------------+
```

**Key observations**:
- ReAct is the best finetuning format across all model sizes
- Finetuned PaLM-8B with ReAct (28.4) outperforms prompted PaLM-540B
  with Standard (27.1) -- a 67x smaller model outperforming a much larger one
- Finetuned PaLM-62B with ReAct (31.2) outperforms all PaLM-540B prompting methods
- This demonstrates ReAct's potential for efficient deployment via distillation

---

## Analysis & Insights

### Synergy Between Reasoning and Acting

The paper provides clear evidence that reasoning and acting are complementary:

1. **Acting helps reasoning**: In knowledge tasks, external retrieval corrects
   the model's internal hallucinations. CoT alone hallucinates facts 56% of the
   time in its failure cases; ReAct's search actions ground the reasoning.

2. **Reasoning helps acting**: In decision tasks, thoughts help the agent
   decompose goals, track progress, and recover from errors. Without thoughts
   (Act-only), the agent loses track of subgoals and repeats actions.

### Failure Mode Analysis (50 randomly sampled trajectories each)

```
+---------------------------+----------+----------+
| Error Category            | CoT (%)  | ReAct (%)|
+---------------------------+----------+----------+
| Hallucination             |   56     |    --    |
| Reasoning error           |   --     |   23    |
| Search failure (no info)  |   --     |   29    |
| Repetitive loop           |   --     |   ~10   |
| False positive rate       |   14     |    6    |
+---------------------------+----------+----------+
```

**CoT failure modes**:
- Hallucination is the dominant failure (56%): the model fabricates facts
  that sound plausible but are incorrect
- Higher false positive rate (14% vs. 6%): CoT is more likely to produce
  a confident-sounding but wrong answer

**ReAct failure modes**:
- Search failure (29%): the model retrieves information but it is not
  sufficiently informative to answer the question
- Reasoning error (23%): the model fails to properly synthesize retrieved
  information or gets stuck in repetitive thought-action loops
- ReAct's errors are more structural (bad retrieval) rather than factual
  (hallucination), making them more diagnosable and fixable

### Why the Combination Works Best

The ReAct -> CoT-SC and CoT-SC -> ReAct switching strategies achieve the
highest scores because they leverage complementary strengths:
- CoT excels when the answer is within the model's parametric knowledge
- ReAct excels when external evidence retrieval is needed
- The switching heuristic: use internal confidence of CoT-SC (majority vote
  agreement) to decide whether to fall back to ReAct for evidence gathering

---

## Limitations & Critiques

1. **Dependence on Retrieval Quality**: ReAct's performance on knowledge tasks
   is bottlenecked by the quality of the Wikipedia API. If Search returns
   irrelevant results, the reasoning cannot recover (29% of failures).

2. **Prompt Sensitivity**: Performance depends heavily on the quality and
   diversity of the few-shot exemplar trajectories. The human-written examples
   must demonstrate the desired reasoning style.

3. **Computational Cost**: Each ReAct trajectory involves multiple LLM calls
   (one per thought/action), significantly increasing inference cost compared
   to single-pass CoT.

4. **Limited Action Spaces**: The evaluated tasks use relatively simple action
   spaces (3 Wikipedia actions, basic text game commands). Scaling to complex
   real-world tool use remains unvalidated.

5. **Greedy Decoding Limitation**: ReAct uses greedy decoding, whereas CoT-SC
   benefits from sampling multiple paths. This comparison disadvantage could
   be addressed by applying self-consistency to ReAct as well.

6. **Repetitive Loop Problem**: ReAct sometimes enters loops where it generates
   the same thought-action sequence repeatedly, indicating a failure of the
   model to plan recovery strategies.

7. **Scale Requirements**: Like CoT, ReAct benefits most from very large models
   (540B parameters). Its effectiveness on smaller models via prompting is
   limited, though finetuning partially addresses this.

---

## Follow-up Work

- **Reflexion** (Shinn et al., 2023): Extends ReAct with verbal self-reflection,
  allowing agents to learn from past mistakes stored in episodic memory.
- **Tree of Thoughts** (Yao et al., 2023): Extends CoT to tree-structured
  exploration, by the same first author.
- **Toolformer** (Schick et al., 2023): Teaches LLMs to use tools via
  self-supervised learning rather than prompting.
- **FireAct** (Chen et al., 2023): Fine-tuning language agents with ReAct-style
  trajectories from multiple prompting methods.
- **LATS** (Zhou et al., 2023): Combines ReAct with Monte Carlo Tree Search for
  more systematic exploration.
- **AutoGPT / BabyAGI** (2023): Open-source agent systems heavily inspired by
  the ReAct loop.
- **LangChain** (Chase, 2022): Popular framework that implements ReAct as its
  default agent architecture.

---

## Key Takeaways

1. **Reasoning and acting are synergistic, not competing paradigms.** Combining
   them in a single framework yields strictly better results than either alone
   across diverse task types.

2. **Grounding through action is essential for knowledge-intensive tasks.**
   Pure reasoning (CoT) hallucinates; interleaving search actions reduces
   hallucination and provides factual grounding.

3. **Reasoning is essential for complex decision-making.** Pure action (Act)
   fails at long-horizon tasks because the agent cannot plan, decompose goals,
   or track state. Adding thoughts fixes this.

4. **The Thought-Action-Observation loop is a general agent architecture.**
   The same framework works for QA, fact verification, text games, and web
   navigation with minimal modification.

5. **Few-shot prompting can outperform extensively trained RL/IL agents.**
   ReAct with 1-2 examples beats agents trained on thousands of trajectories
   on ALFWorld (+34%) and WebShop (+10%).

6. **Finetuning amplifies ReAct's benefits.** ReAct-format finetuning is the
   best format across model sizes, and finetuned small models (8B) can
   outperform prompted large models (540B).

7. **Human interpretability is a practical advantage.** The explicit reasoning
   traces make ReAct agents easier to debug, audit, and improve compared to
   opaque action-only or end-to-end learned agents.

8. **ReAct established the foundational agent loop** that most subsequent
   LLM agent systems (LangChain, AutoGPT, etc.) adopted as their core
   architecture pattern.
