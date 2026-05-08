---
title: "Tree of Thoughts: Deliberate Problem Solving with Large Language Models"
authors: "Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, Karthik Narasimhan"
venue: "NeurIPS 2023"
year: 2023
arxiv: "https://arxiv.org/abs/2305.10601"
code: "https://github.com/princeton-nlp/tree-of-thought-llm"
institution: "Princeton University, Google DeepMind"
tags: [reasoning, search, prompting, deliberate-planning, tree-search, LLM, foundational]
status: done
date_reviewed: 2026-05-08
---

# Tree of Thoughts: Deliberate Problem Solving with Large Language Models

## TL;DR

Tree of Thoughts (ToT) generalizes chain-of-thought prompting by framing LLM reasoning as
search over a tree of intermediate "thought" steps. The LLM itself generates candidate
thoughts, evaluates their promise via value or vote mechanisms, and a classical search
algorithm (BFS or DFS) navigates the tree -- enabling deliberate planning, exploration of
alternatives, and backtracking. On Game of 24, success rate jumps from 4% (CoT) to 74%
(ToT). The paper establishes a formal four-component framework (thought decomposition,
generation, evaluation, search) that subsumes IO, CoT, and CoT-SC as special cases.

## Motivation & Problem

The dominant paradigm for LLM reasoning circa early 2023:

1. **IO prompting**: Direct input-to-output, no intermediate reasoning
2. **Chain-of-Thought (CoT)** (Wei et al., 2022): Single linear chain of reasoning steps
3. **CoT with Self-Consistency (CoT-SC)** (Wang et al., 2023): Sample multiple independent
   chains and majority-vote on the final answer

All three share a critical limitation: **reasoning is left-to-right and irrevocable**.
Once a token is generated, the model cannot explore alternatives or backtrack -- unsuitable
for problems requiring exploration, strategic lookahead, or backtracking.

Drawing on Newell & Simon's (1972) problem space theory and Kahneman's (2011) dual-process
theory (CoT = System 1, ToT = System 2), the key insight is that the LLM can serve as both
**generator** and **evaluator**, unifying search in a single model without hand-crafted
heuristics.

## Method

### Formal Framework

ToT defines four independently configurable components:

**1. Thought Decomposition** -- granularity of intermediate reasoning steps:

| Task             | Thought Unit        | Depth (T) | Rationale                          |
|------------------|---------------------|-----------|------------------------------------|
| Game of 24       | An equation line    | 3         | 3 binary ops reduce 4 nums to 1   |
| Creative Writing | A paragraph plan    | 2         | Plan then write, each via voting   |
| Mini Crosswords  | A word fill         | 5-10      | 5 across + 5 down clues           |

Thoughts must be: small enough for diverse generation, large enough for meaningful
evaluation, and compositional (sequences form complete solutions).

**2. Thought Generation** G(p_theta, s, k) -- produce k candidates from state s:

(a) **i.i.d. Sampling**: Draw k thoughts independently via temperature sampling:
```
t_i ~ p_theta(thought | s),  i = 1..k
```
Best when thought space is rich (e.g., creative writing paragraphs).

(b) **Sequential Proposal**: Single LLM call proposes k distinct candidates:
```
[t_1, ..., t_k] = LLM("Given state s, propose k different next steps")
```
Better when diversity requires explicit prompting (e.g., Game of 24 arithmetic).
Avoids duplicates and is more token-efficient.

**3. State Evaluation** V(p_theta, S) -- assess promise of each partial solution:

(a) **Value Prompting** -- independently classify each state:
```
V(s) = LLM("Evaluate if this partial solution can reach the goal.
             Answer: sure / maybe / impossible")
```
For Game of 24: commonsense reasoning ("remaining numbers too large" -> impossible).
Each state evaluated independently, enabling caching and parallelism.

(b) **Deliberate Voting** -- comparative assessment across candidates:
```
best = LLM("Given states {s_1,...,s_k}, vote for the most promising one")
```
Repeated across multiple rounds; majority determines winner. Used for creative writing
where quality is comparative and subjective.

**4. Search Algorithm** -- navigate the tree structure:

**(a) Breadth-First Search (BFS)** -- used for Game of 24 and Creative Writing:

```
Algorithm: ToT-BFS(x, T, b, B)
  S_0 = {x}                           # initial state set
  for t = 1 to T:
    S_t' = {}
    for s in S_{t-1}:                  # expand each active state
      thoughts = Generate(s, b)        # b candidates per state
      for z in thoughts:
        S_t' = S_t' + {(s, z)}         # extend state
    V_t = {Evaluate(s') for s' in S_t'}  # evaluate all candidates
    S_t = top-B(S_t', V_t)            # keep best B states (beam)
  return best state in S_T
```

**(b) Depth-First Search (DFS)** -- used for Mini Crosswords:

```
Algorithm: ToT-DFS(s, t, T, v_th)
  if t > T: return s                   # max depth reached
  thoughts = Generate(s, b)
  for z in thoughts:
    v = Evaluate((s, z))               # evaluate candidate
    if v > v_th:                       # prune unpromising branches
      result = DFS((s, z), t+1, T, v_th)
      if result is solution: return result
    else: prune and backtrack
  return failure                       # all branches pruned
```

### ToT as Generalization

IO = single forward pass. CoT = single linear chain. CoT-SC = multiple independent chains
+ majority vote. ToT = deliberate tree search with generation, evaluation, and backtracking.
All prior methods are formally special cases of ToT.

## Key Innovations

1. **LLM as both generator and evaluator**: The same LLM proposes candidate reasoning
   steps AND evaluates their quality, eliminating external reward models or hand-crafted
   heuristics -- analogous to neural value functions in AlphaGo but zero-shot.

2. **Deliberate search over language**: First framework to apply classical AI search (BFS,
   DFS) to natural language reasoning within LLMs, enabling backtracking and exploration
   fundamentally absent from CoT.

3. **Modular and task-agnostic**: The four components (decompose, generate, evaluate,
   search) are independently configured per task, making ToT a general framework.

4. **Principled generalization**: IO, CoT, and CoT-SC are formally special cases of ToT
   (single branch = CoT; multiple independent branches with voting = CoT-SC).

## Experimental Setup

- **Model**: GPT-4 for all experiments
- **Baselines**: IO prompting, CoT, CoT-SC (k=100 samples for fair compute comparison)

**Task 1 -- Game of 24**: Use 4 numbers with +,-,*,/ to reach 24. Each number used exactly
once. 100 hard games (index 901-1000). Config: sequential proposal (b=5), value evaluation
({sure/maybe/impossible}), BFS with beam B=5, depth T=3.

**Task 2 -- Creative Writing**: Write a coherent 4-paragraph passage where each paragraph
ends with one of 4 given random sentences. 100 test inputs. Config: i.i.d. sampling (k=5),
vote evaluation (5 votes), BFS with B=1, depth T=2 (plan then passage).

**Task 3 -- Mini Crosswords (5x5)**: Fill a 5x5 crossword grid given 10 clues. 20 games
from NYT Mini archive. Config: sequential proposal, value evaluation, DFS with pruning
(prune if any clue deemed "impossible"), depth up to 10.

## Results

### Game of 24

| Method              | Success Rate | LLM Calls (approx) |
|---------------------|-------------|---------------------|
| IO prompting        | 7.3%        | 1                   |
| CoT prompting       | 4.0%        | 1                   |
| CoT-SC (k=100)      | 9.0%        | 100                 |
| Best of 100 CoT     | 49%         | 100                 |
| **ToT-BFS (b=1)**   | **45%**     | ~20                 |
| **ToT-BFS (b=5)**   | **74%**     | ~60                 |

ToT (b=5) achieves 74% -- an **18x improvement** over CoT. Even b=1 (no beam diversity)
yields 45% via evaluation-based pruning alone. Best-of-100 CoT (49%) samples 100
independent chains but still underperforms ToT (b=5), which explores far fewer total
paths but does so strategically via informed search.

### Creative Writing (Coherency, 1-10 scale)

| Method   | GPT-4 Score | Human Score | Human Preference vs IO |
|----------|-------------|-------------|------------------------|
| IO       | 6.19        | 3.43        | --                     |
| CoT      | 6.93        | 4.56        | --                     |
| **ToT**  | **7.56**    | **4.88**    | **67% preferred**      |

ToT preferred over CoT in 61% of pairwise human comparisons. The two-stage voting
process (vote on best plan, then vote on best passage) yields more coherent narratives.

### Mini Crosswords

| Method       | Word Success | Letter Success | Games Solved |
|--------------|-------------|----------------|--------------|
| IO           | 16.0%       | 33.6%          | 0/20         |
| CoT          | 15.6%       | 34.8%          | 0/20         |
| **ToT-DFS**  | **60.0%**   | **78.7%**      | **4/20**     |
| ToT + oracle | ~60%+       | ~78%+          | 7/20         |

DFS with backtracking enables the model to try a word, discover crossing-word conflicts,
backtrack, and try alternatives -- fundamentally impossible in left-to-right generation.
Oracle evaluation (ground-truth pruning) shows significant remaining headroom.

### Computational Cost Analysis

| Task             | Method  | Avg LLM Calls | Approx Tokens |
|------------------|---------|---------------|---------------|
| Game of 24       | CoT     | 1             | ~100          |
| Game of 24       | CoT-SC  | 100           | ~10,000       |
| Game of 24       | ToT-BFS | ~60           | ~6,500        |
| Creative Writing | CoT     | 1             | ~400          |
| Creative Writing | ToT     | ~25           | ~10,000       |
| Crosswords       | CoT     | 1             | ~200          |
| Crosswords       | ToT-DFS | ~50-200       | ~15,000       |

The cost-performance tradeoff is highly favorable for Game of 24: 10x performance gain
for ~60x compute increase, vs. CoT-SC's 2x gain for 100x compute.

## Analysis & Insights

1. **Evaluation quality is the bottleneck**: The value/vote accuracy directly determines
   search effectiveness. Game of 24's "sure/maybe/impossible" classification is ~80%
   precise, enabling effective pruning. Creative writing evaluation is noisier, hence
   smaller gains. Oracle experiments show significant headroom when evaluation is perfect.

2. **Thought granularity is a critical design choice**: Too fine (single tokens) creates
   intractable trees; too coarse (entire solutions) eliminates search benefit. The right
   granularity depends on problem decomposition structure.

3. **BFS vs. DFS tradeoffs**: BFS suits shallow search spaces with good early discrimination
   (Game of 24: depth 3). DFS suits deep constraint-satisfaction where violations are
   detectable early (crosswords: depth 10 with constraint conflicts).

4. **CoT failure modes ToT addresses**: (a) Commitment to wrong arithmetic path --
   ToT prunes impossible branches. (b) No global coherence view -- ToT's voting provides
   holistic comparison. (c) No constraint recovery -- DFS naturally backtracks on conflicts.

5. **LLM as heuristic function**: The evaluation component turns the LLM into a learned
   heuristic for search, zero-shot and prompt-specified rather than trained, previewing
   the "scaling test-time compute" paradigm.

## Limitations & Critiques

1. **Computational expense**: 10-200x more LLM calls than CoT. Impractical for
   latency-sensitive or cost-constrained applications.
2. **Task-specific engineering**: Each task requires manual configuration of all four
   components. No automatic adaptation or meta-learning.
3. **Limited task diversity**: Only three small-scale tasks. Scalability to deep trees
   (depth >> 10) or wide branching (b >> 10) is unknown.
4. **No learning from search**: Each instance starts fresh; no heuristic transfer.
5. **Evaluation reliability**: LLM self-evaluation is uncalibrated. False positives
   waste compute; false negatives lose valid paths.
6. **GPT-4 dependence**: Unclear if smaller/open-source models can serve as evaluators.
7. **No theoretical analysis**: Purely empirical; no formal guarantees on when ToT
   outperforms CoT or convergence properties.

## Follow-up Work

- **Graph of Thoughts** (Besta et al., 2023): DAG structure with thought merging/refinement.
- **Algorithm of Thoughts** (Sel et al., 2023): Simulates tree search in a single LLM call.
- **RAP** (Hao et al., 2023): MCTS + LLM world model with UCB-based exploration.
- **LATS** (Zhou et al., 2023): ToT + ReAct + MCTS for agent decision-making.
- **XoT** (Ding et al., 2023): RL-trained policy network to guide thought generation.
- **iToT** (2024): A*/D* informed search reducing nodes while preserving accuracy.
- **Scaling test-time compute**: ToT directly influenced OpenAI's o1/o3 paradigm.

## Key Takeaways

1. **Framing LLM reasoning as search** is a powerful paradigm that enables exploration and
   backtracking, dramatically expanding what LLMs can solve.

2. **Self-evaluation is surprisingly effective**: LLMs can assess their own partial solutions
   well enough to guide meaningful search, at least for well-structured problems.

3. **The compute-accuracy frontier is movable**: Additional inference-time tokens yield
   outsized improvements -- a principle now central to reasoning model design (o1, o3).

4. **Task decomposition is the key design choice**: The granularity and structure of
   "thoughts" determines the search space and thus the framework's effectiveness.

5. **ToT opened the door** to a family of inference-time reasoning methods (GoT, MCTS,
   RAP, eventually o1) that treat generation as search rather than a single forward pass.
