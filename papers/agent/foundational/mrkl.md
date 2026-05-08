---
title: "MRKL Systems: A Modular, Neuro-Symbolic Architecture that Combines Large Language Models, External Knowledge Sources and Discrete Reasoning"
authors:
  - Ehud Karpas
  - Omri Abend
  - Yonatan Belinkov
  - Barak Lenz
  - Opher Lieber
  - Nir Ratner
  - Yoav Shoham
  - Hofit Bata
  - Yoav Levine
  - Kevin Leyton-Brown
  - Dor Muhlgay
  - Noam Rozen
  - Erez Schwartz
  - Gal Shachaf
  - Shai Shalev-Shwartz
  - Amnon Shashua
  - Moshe Tenenholtz
venue: arXiv preprint (AI21 Labs)
year: 2022
url: https://arxiv.org/abs/2205.00445
institution: AI21 Labs
tags:
  - architecture
  - tool-routing
  - neuro-symbolic
  - modular-reasoning
  - agents
  - foundational
status: done
---

# MRKL Systems: Modular Reasoning, Knowledge and Language

## TL;DR

MRKL (pronounced "miracle") proposes a neuro-symbolic architecture where a large language model
serves as a central router that dispatches natural language queries to an extendable set of
specialized "expert" modules -- both neural (e.g., other LMs fine-tuned for specific tasks) and
symbolic (e.g., calculators, databases, APIs). The paper formalizes the concept that an LLM should
not try to do everything itself but should instead orchestrate a collection of specialized systems.
This conceptual framework directly preceded and inspired ReAct, LangChain agents, and the entire
modern tool-using agent paradigm.

---

## Motivation & Problem

### Limitations of Monolithic LLMs

Large language models like GPT-3 demonstrate impressive few-shot capabilities but suffer from
critical structural limitations:

1. **Arithmetic failure** -- LLMs fail on multi-digit arithmetic. GPT-3 accuracy on 5-digit
   addition drops from ~80% (3-digit) to under 10%. Multiplication is even worse.
2. **Stale knowledge** -- Parametric knowledge is frozen at training time and cannot reflect
   current events, prices, weather, or breaking news.
3. **No proprietary access** -- LLMs cannot query private databases, internal company knowledge
   bases, or paywalled content.
4. **Lack of interpretability** -- When an LLM produces an answer, there is no audit trail of
   which reasoning was applied or which data was consulted.
5. **Brittleness on symbolic tasks** -- Tasks requiring precise symbolic computation (dates,
   units, currencies, logic) degrade unpredictably as complexity increases.

### The Core Insight

Rather than building one massive model that does everything poorly, build a **system** where:
- A language model handles natural language understanding and routing
- Specialized modules handle tasks they are provably good at
- The system is modular and extensible -- new capabilities can be added without retraining

---

## Method

### The MRKL Architecture

```
                        +-------------------+
                        |   USER QUERY      |
                        | "What is 47392    |
                        |  times 8841?"     |
                        +--------+----------+
                                 |
                                 v
                    +------------+------------+
                    |        ROUTER           |
                    | (Large Language Model)  |
                    | - Parses intent         |
                    | - Selects expert module |
                    | - Formats input         |
                    | - Integrates output     |
                    +--+------+------+-------+
                       |      |      |
              +--------+  +---+---+  +--------+
              |           |       |           |
              v           v       v           v
    +---------+--+ +------+----+ ++---------+ ++----------+
    | Calculator | | Weather   | | Database | | Knowledge |
    | (Symbolic) | | API       | | Query    | | Base      |
    |            | | (Symbolic)| | (Symb.)  | | (Neural)  |
    | 47392*8841 | |           | |          | |           |
    | = 419,059  | |           | |          | |           |
    +------------+ +-----------+ +----------+ +-----------+

    [Module 1]     [Module 2]    [Module 3]   [Module N]
     Symbolic       Symbolic      Symbolic      Neural
     Discrete       Continuous    Discrete      Continuous
```

### Component Taxonomy

MRKL classifies expert modules along two independent dimensions:

```
                      IMPLEMENTATION
                   Neural        Symbolic
              +-------------+---------------+
   Discrete   | Fine-tuned  | Calculator,   |
   (finite    | classifier  | Lookup table, |
    output)   | for entity  | Database      |
              | types       | query engine  |
   DOMAIN     +-------------+---------------+
              | Language    | Physics       |
   Continuous | model for   | simulator,    |
   (open-ended| open QA,    | Weather model,|
    output)   | translation | Optimization  |
              +-------------+---------------+
```

**Four categories of expert modules:**

1. **Neural + Discrete** -- Neural models that classify into a finite set of outputs
   (e.g., sentiment classification, named entity recognition)
2. **Neural + Continuous** -- Neural models that generate open-ended outputs
   (e.g., a fine-tuned LM for specific domain QA, a translation model)
3. **Symbolic + Discrete** -- Rule-based systems with finite outputs
   (e.g., calculator, date arithmetic, unit converter, lookup tables)
4. **Symbolic + Continuous** -- Algorithmic systems with open-ended outputs
   (e.g., search engine, database query engine, weather simulator)

### The Routing Mechanism

The router is the critical component. It is itself a language model (or fine-tuned LM) that
performs the following steps:

```
Algorithm: MRKL Routing

Input:  User query q, Set of expert modules E = {e_1,...,e_N}
Output: Final answer a

1. PARSE:     Extract intent and entities from q using the router LM
2. CLASSIFY:  Determine which expert module e_k best handles q
              - If q involves math -> route to Calculator
              - If q involves dates -> route to Date module
              - If q involves weather -> route to Weather API
              - If q involves facts -> route to Knowledge Base
              - If no expert matches -> FALLBACK to router LM itself
3. FORMAT:    Transform q into the input format expected by e_k
              - For Calculator: extract numbers and operations
              - For Database: generate SQL or structured query
              - For API: format as API request
4. EXECUTE:   Send formatted input to e_k, receive result r_k
5. INTEGRATE: Combine r_k with original context to produce answer a
6. RETURN:    Present a to user
```

The routing can be implemented via:
- **Fine-tuning:** Train the router LM to predict which expert to call (the approach used
  in the Jurassic-X experiments)
- **Few-shot prompting:** Provide the router with descriptions and examples for each expert
- **Learned embedding similarity:** Embed the query and expert descriptions, route by
  nearest neighbor

### Safe Fallback

A critical design property: if the router cannot confidently assign the query to any expert
module, it falls back to the base LLM's own generation. This ensures the system is never
*worse* than a standalone LLM -- it can only be better.

---

## Key Innovations

1. **Formal neuro-symbolic framework for LLM augmentation** -- MRKL was the first paper to
   formally articulate the architecture pattern of "LLM as router + specialized expert modules."
   This conceptual framework became the blueprint for all subsequent agent systems.

2. **Module taxonomy** -- The two-dimensional classification (neural/symbolic x discrete/continuous)
   provides a principled way to think about what types of tools an agent system should include.

3. **Desiderata for augmented LM systems** -- The paper identifies six essential properties
   that any such system should satisfy (see below).

4. **Compositionality through routing** -- By decomposing compound queries and routing
   sub-problems to different experts, the system can handle multi-hop reasoning that no
   single module could solve alone.

5. **Extensibility without retraining** -- New expert modules can be added to the system
   without modifying or retraining the existing modules; only the router needs updating.

### Six Desiderata

The paper formally defines six properties a MRKL system should satisfy:

| Desideratum          | Description                                                    |
|----------------------|----------------------------------------------------------------|
| **Safe fallback**    | If no expert matches, the system defaults to the LLM itself   |
| **Extensibility**    | New experts can be added cheaply without breaking existing ones|
| **Interpretability** | Routing decisions provide an audit trail for the answer        |
| **Up-to-date info**  | External modules can access current data                       |
| **Proprietary access**| Modules can connect to private data and internal systems      |
| **Compositionality** | Multi-hop queries can be decomposed across multiple experts    |

---

## Experimental Setup

### Implementation: Jurassic-X

AI21 Labs implemented MRKL as **Jurassic-X**, combining their Jurassic-1 LLM with approximately
55 specialized expert modules covering domains including:

- Arithmetic (calculator)
- Currency conversion
- Date/time reasoning
- Weather forecasting
- General knowledge retrieval
- And many others

### Focused Experiments: Arithmetic

The paper's primary experimental contribution is a deep analysis of arithmetic routing. The
experiments test whether a router fine-tuned to extract math problems from natural language
and send them to a calculator can dramatically outperform an LLM doing arithmetic alone.

**Training setup:**
- Router model fine-tuned on the Jurassic-1 LM
- Task: given a natural language math problem, extract the arithmetic expression and send
  it to a symbolic calculator module
- Training data: math problems with single-digit operands only
- Test data: math problems with operands ranging from 1 to 9 digits

**Problem types tested:**
- Single-operation: addition, subtraction, multiplication
- Two-operation: combinations (e.g., "add X and Y, then multiply by Z")
- Digit-format numbers (e.g., "1234") vs word-format numbers (e.g., "one thousand two
  hundred thirty-four")

### Baselines

- **GPT-3 (175B, text-davinci-002)** -- direct arithmetic via prompting
- **Jurassic-1 LLM** -- direct arithmetic via prompting
- **MRKL (Jurassic-X)** -- router + symbolic calculator

---

## Results

### Arithmetic Accuracy: MRKL vs. GPT-3

**Addition accuracy by number of digits:**

| # Digits | GPT-3 175B | MRKL (Jurassic-X) |
|----------|-----------|---------------------|
| 1        | ~1.000    | 1.000               |
| 2        | ~0.990    | 1.000               |
| 3        | 0.804     | 1.000               |
| 4        | 0.414     | 1.000               |
| 5        | 0.093     | 1.000               |
| 6        | --        | 1.000               |
| 7        | --        | 1.000               |
| 8        | --        | 1.000               |
| 9        | --        | 1.000               |

**Multiplication accuracy by number of digits:**

| # Digits | GPT-3 175B | MRKL (Jurassic-X) |
|----------|-----------|---------------------|
| 1        | ~1.000    | 0.98-1.00           |
| 2        | ~0.60     | 0.98-1.00           |
| 3        | ~0.20     | 0.98-1.00           |
| 4        | ~0.05     | 0.98-1.00           |
| 5        | ~0.00     | 0.98-1.00           |
| 6-9      | ~0.00     | 0.98-1.00           |

Key insight: GPT-3 performance **degrades catastrophically** as digit count increases (dropping
from ~0.80 on 3-digit addition to ~0.09 on 5-digit). The MRKL system maintains near-perfect
accuracy (1.00 for addition, 0.98-1.00 for multiplication) across all digit lengths up to 9
digits, because it routes the computation to a symbolic calculator.

### Generalization Results

**Training on word-format, testing on digit-format:**

| Test Format | Accuracy |
|-------------|----------|
| Words -> Words   | ~1.00  |
| Words -> Digits  | 0.987  |
| Digits -> Digits | ~1.00  |
| Digits -> Words  | 0.95+  |

The router trained on word-based number representations ("forty-seven thousand three hundred
ninety-two") generalized exceptionally well to digit-based representations ("47392"), achieving
0.987 accuracy. This demonstrates that the router learns the abstract concept of "this is an
arithmetic problem" rather than memorizing surface patterns.

### Two-Operation Composition

The system trained only on single-operation problems successfully handled most two-operation
problems, achieving high accuracy (often 1.0 or high 0.9s) across different operation
combinations (add-then-multiply, subtract-then-divide, etc.).

### Where Routing Fails

The experiments also reveal failure modes:
- The router occasionally misparses complex natural language phrasing
- Ambiguous queries (e.g., those that could be arithmetic or factual) sometimes route to the
  wrong module
- The composition of more than two operations was not tested extensively

---

## Analysis & Insights

### Why MRKL Matters as Intellectual Precursor

MRKL is historically significant not for its experimental results (which are narrow) but for
its **conceptual contribution**. The paper was the first to clearly articulate several ideas
that became foundational:

1. **LLM as orchestrator, not solver** -- The LLM's job is to understand the user's intent
   and delegate, not to compute the answer itself. This is now the default assumption in
   agent design.

2. **Modular extensibility** -- The idea that you can add new capabilities by adding new
   modules without retraining the core system. This directly maps to how tools are added
   to modern agent frameworks.

3. **The routing problem** -- MRKL identified that the hard problem is not building the
   expert modules (calculators exist) but teaching the LLM to route correctly. This remains
   an active research challenge.

4. **Neuro-symbolic hybrid** -- The explicit combination of neural (LLM) and symbolic
   (calculator, database) components was prescient. Modern agents routinely mix neural
   generation with symbolic execution.

### The Neuro-Symbolic Chasm

The paper coins the term "neuro-symbolic chasm" to describe the gap between:
- What LLMs can do well (language understanding, few-shot reasoning, creative generation)
- What symbolic systems can do well (precise computation, database lookup, formal logic)

MRKL proposes bridging this chasm through natural language as the interface layer -- the LLM
translates between the user's natural language and the expert module's expected input format.

### Compositional Reasoning

A key insight is that real-world queries often require **composing** multiple expert modules.
For example: "What was the GDP of France last year in Japanese yen?" requires:
1. Knowledge retrieval (GDP of France)
2. Date reasoning (what year is "last year"?)
3. Currency conversion (EUR to JPY)
4. Arithmetic (the conversion calculation)

MRKL argues that a modular system can handle such queries by routing sub-problems to the
appropriate experts and composing their outputs.

---

## Limitations & Critiques

1. **Narrow experimental scope** -- The paper only deeply evaluates arithmetic routing. Claims
   about general-purpose routing, multi-hop composition, and extensibility are conceptual
   arguments, not experimentally validated.

2. **Routing accuracy not fully measured** -- The paper does not provide comprehensive metrics
   on routing accuracy across diverse query types. The router is only evaluated on arithmetic
   extraction.

3. **No comparison with prompting-based approaches** -- The paper does not compare MRKL routing
   with simpler approaches like few-shot prompting the LLM to use tools (which later proved
   effective in ReAct and Toolformer).

4. **Closed-source implementation** -- Jurassic-X and the full 55-module system were not released,
   making it impossible to reproduce or build upon the full system.

5. **Scalability of routing** -- With 55+ modules, how does the router scale? The paper does
   not address routing accuracy degradation as the number of modules increases.

6. **Sequential routing only** -- The paper discusses compositionality as a desideratum but
   does not demonstrate a working implementation of multi-hop, multi-module query resolution.

7. **No learning from failures** -- When the router makes a mistake, there is no mechanism for
   the system to detect the error and try a different module.

8. **Training cost of router** -- Fine-tuning the router for each new expert module may not
   scale as well as few-shot or zero-shot routing approaches.

---

## Connection to LangChain's MRKL Agent

LangChain, the dominant agent framework from 2023 onward, directly implemented a "MRKL agent"
as one of its core agent types:

```python
# LangChain's MRKL agent implementation (simplified)
from langchain.agents import initialize_agent, AgentType

agent = initialize_agent(
    tools=[calculator_tool, search_tool, database_tool],
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,  # MRKL-style routing
    verbose=True
)

# The agent:
# 1. Reads the user query
# 2. Examines tool descriptions
# 3. Selects the appropriate tool (routing)
# 4. Formats the tool input
# 5. Executes the tool
# 6. Returns the result
```

Key connections:
- LangChain's `MRKLChain` is directly named after this paper
- The `ZERO_SHOT_REACT_DESCRIPTION` agent type combines MRKL routing with ReAct reasoning
- Tool descriptions in LangChain serve as the "expert module specifications" in MRKL
- LangChain's `AgentExecutor` implements the router + execute + integrate loop
- The safe fallback desideratum maps to LangChain's fallback handling

The evolution: MRKL (concept) -> ReAct (reasoning traces) -> LangChain (practical implementation)
-> modern agent frameworks (OpenAI function calling, Claude tool use, etc.)

---

## Follow-up Work

- **ReAct (Yao et al., 2022)** -- Extended MRKL routing with explicit reasoning traces
  (Thought/Action/Observation loops), creating the dominant agent paradigm.
- **Toolformer (Schick et al., 2023)** -- Approached the same problem from a training
  perspective: teach the model to route via self-supervision rather than prompting.
- **HuggingGPT (Shen et al., 2023)** -- Scaled the MRKL concept to hundreds of ML models
  on HuggingFace as expert modules.
- **LangChain (Chase, 2022-2023)** -- Made MRKL practical with an open-source framework
  implementing MRKL-style agents with dozens of tool integrations.
- **Gorilla (Patil et al., 2023)** -- Fine-tuned an LLM specifically for API routing across
  thousands of APIs.
- **OpenAI Function Calling (2023)** -- Productionized the MRKL routing concept with
  structured function schemas and native model support.
- **Claude Tool Use (Anthropic, 2023-2024)** -- Similar productionization with tool-use
  capabilities built into Claude models.

---

## Key Takeaways

1. **The paper is more important for its ideas than its experiments.** MRKL formalized the
   concept of "LLM as router to specialized modules" that now underpins every agent framework.
   The arithmetic experiments are a proof of concept, not the main contribution.

2. **Symbolic modules solve symbolic problems.** The contrast between GPT-3's catastrophic
   arithmetic degradation and MRKL's perfect accuracy via calculator routing demonstrates
   why neuro-symbolic hybrids are necessary. No amount of scaling will make LLMs reliable
   calculators.

3. **The routing problem is the hard problem.** Building a calculator is trivial; teaching
   an LLM to reliably extract arithmetic from natural language and route it correctly is
   the actual research challenge.

4. **Extensibility is a system design principle.** MRKL's insight that new capabilities
   should be addable without retraining the core system is now standard in agent design
   (tools as plugins).

5. **The six desiderata remain relevant.** Safe fallback, extensibility, interpretability,
   up-to-date information, proprietary access, and compositionality are still the goals
   that modern agent frameworks strive for.

6. **Natural language as universal interface.** MRKL's use of the LLM to translate between
   user queries and module-specific input formats established that natural language can serve
   as the glue layer in modular AI systems.

7. **Historical lineage matters.** Understanding MRKL clarifies why LangChain agents work
   the way they do, why tool descriptions matter, and why the router/executor pattern
   is universal. MRKL is the conceptual ancestor of all modern AI agent architectures.
