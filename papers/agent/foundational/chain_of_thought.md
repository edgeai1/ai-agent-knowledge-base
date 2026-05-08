---
title: "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models"
authors:
  - Jason Wei
  - Xuezhi Wang
  - Dale Schuurmans
  - Maarten Bosma
  - Brian Ichter
  - Fei Xia
  - Ed Chi
  - Quoc Le
  - Denny Zhou
venue: NeurIPS 2022
year: 2022
submitted: 2022-01-28
url: https://arxiv.org/abs/2201.11903
tags:
  - reasoning
  - prompting
  - chain-of-thought
  - emergent-abilities
  - arithmetic
  - commonsense
  - symbolic-reasoning
  - few-shot
  - foundational
status: done
---

# Chain-of-Thought Prompting Elicits Reasoning in Large Language Models

## TL;DR

Chain-of-thought (CoT) prompting augments few-shot exemplars with intermediate
reasoning steps, dramatically improving large language model performance on
arithmetic, commonsense, and symbolic reasoning tasks. This capability is an
**emergent property of scale** -- it only materializes in models with roughly
100B+ parameters. With just 8 CoT exemplars, PaLM-540B achieves state-of-the-art
on GSM8K (56.9%), surpassing fine-tuned GPT-3 175B with a trained verifier (55%),
and sets new SOTA on multiple other benchmarks without any fine-tuning.

---

## Motivation & Problem

### The Reasoning Gap

Despite remarkable capabilities in language understanding and generation, LLMs
at the time of this paper struggled with tasks requiring multi-step reasoning:

- **Arithmetic word problems** require parsing natural language, identifying
  relevant quantities, and executing a sequence of mathematical operations.
- **Commonsense reasoning** requires chaining together multiple pieces of
  world knowledge that are not explicitly stated.
- **Symbolic manipulation** requires applying abstract rules consistently
  over multiple steps.

Standard few-shot prompting (exemplars with input-output pairs) was ineffective
for these tasks, with performance often near random even at large scale.

### The Hypothesis

If the few-shot exemplars include not just the final answer but also the
**intermediate reasoning steps** (the "chain of thought"), the model will
learn to generate its own reasoning chains at inference time, decomposing
complex problems into manageable steps.

### Formal Definition

```
Standard Prompting:
  Exemplar format: (input, output)
  Model generates:  input -> output

Chain-of-Thought Prompting:
  Exemplar format: (input, chain-of-thought, output)
  Model generates:  input -> chain-of-thought -> output

Where chain-of-thought = [step_1, step_2, ..., step_n]
  Each step_i is a natural language sentence expressing
  an intermediate reasoning step toward the answer.
```

### Example: Standard vs. CoT Prompting

```
STANDARD PROMPTING:
====================
Q: Roger has 5 tennis balls. He buys 2 more cans of
   tennis balls. Each can has 3 tennis balls. How many
   tennis balls does he have now?
A: The answer is 11.

CHAIN-OF-THOUGHT PROMPTING:
============================
Q: Roger has 5 tennis balls. He buys 2 more cans of
   tennis balls. Each can has 3 tennis balls. How many
   tennis balls does he have now?
A: Roger started with 5 balls. 2 cans of 3 tennis balls
   each is 6 tennis balls. 5 + 6 = 11. The answer is 11.
```

The key difference: CoT exemplars show _how_ to arrive at the answer, not just
_what_ the answer is.

---

## Method

### The CoT Prompting Approach

The method is remarkably simple -- no architectural changes, no fine-tuning,
no reinforcement learning:

1. **Manually write** a small set of (question, chain-of-thought, answer)
   exemplars for the target task (typically 8 exemplars).
2. **Prepend** these exemplars to the test question as a few-shot prompt.
3. **Generate** the model's response, which will include a reasoning chain
   followed by a final answer.
4. **Extract** the final answer from the generated text.

### Pseudocode

```
Algorithm: Chain-of-Thought Prompting
==========================================
Input: test question q, set of exemplars E = {(x_i, c_i, y_i)}
       where x_i = question, c_i = chain of thought, y_i = answer
Output: predicted answer y

// Construct prompt
prompt <- ""
for each (x_i, c_i, y_i) in E:
    prompt <- prompt + "Q: " + x_i + "\n"
    prompt <- prompt + "A: " + c_i + " The answer is " + y_i + "\n\n"

// Append test question
prompt <- prompt + "Q: " + q + "\n" + "A: "

// Generate response
response <- LLM.generate(prompt)

// Parse answer from response
answer <- extract_final_answer(response)
return answer
```

### Diagram: Standard vs. CoT Information Flow

```
STANDARD FEW-SHOT PROMPTING:
+----------+          +-------+
| Question |--------->| Model |-------> Answer
+----------+          +-------+
   Direct mapping, no intermediate computation

CHAIN-OF-THOUGHT PROMPTING:
+----------+     +--------+     +--------+           +--------+     +--------+
| Question |---->| Step 1 |---->| Step 2 |---> ... ->| Step n |---->| Answer |
+----------+     +--------+     +--------+           +--------+     +--------+
   Decomposed reasoning, each step conditions on previous steps

EMERGENT ABILITY SCALING:
                                    CoT
Performance                        /
    ^                             /
    |                            /
    |         Standard          /
    |        ----------        /
    |       /                 /
    |      /         -------/
    |     /         /
    |    /         /
    |---/---------/--
    +--+----+----+----+----+---> Model Size
      1B   10B  100B  500B
           ^
           |
      Emergence threshold (~100B)
```

---

## Key Innovations

1. **Simplicity of the Approach**: No model architecture changes, no training
   procedure modifications, no learned verifiers -- just a change in the
   prompting format. This makes CoT immediately applicable to any LLM.

2. **Emergent Ability Discovery**: The finding that CoT reasoning is an emergent
   property of model scale (only appearing at ~100B+ parameters) was a landmark
   result in understanding LLM capabilities and limitations.

3. **Natural Language as Computation**: CoT showed that natural language reasoning
   steps can serve as a form of "intermediate computation" that extends the
   effective reasoning depth of a transformer, which is otherwise limited by
   its fixed depth.

4. **Bridging Prompting and Fine-tuning**: CoT prompting with PaLM-540B
   surpassed fine-tuned SOTA on GSM8K, demonstrating that prompting at
   sufficient scale can be competitive with or superior to task-specific
   fine-tuning.

5. **Decomposition Principle**: The implicit principle that complex reasoning
   should be decomposed into sequential sub-steps became foundational for
   all subsequent prompting research (Tree of Thoughts, ReAct, etc.).

---

## Experimental Setup

### Models Evaluated

```
+-------------------+----------------------------------+
| Model Family      | Sizes Tested                     |
+-------------------+----------------------------------+
| GPT-3             | 350M, 1.3B, 6.7B, 175B          |
|                   | (ada, babbage, curie, davinci)   |
+-------------------+----------------------------------+
| LaMDA             | 422M, 2B, 8B, 68B, 137B         |
+-------------------+----------------------------------+
| PaLM              | 8B, 62B, 540B                    |
+-------------------+----------------------------------+
| UL2               | 20B                              |
+-------------------+----------------------------------+
| Codex             | (code-davinci-002)               |
+-------------------+----------------------------------+
```

### Benchmarks

**Arithmetic Reasoning (5 benchmarks)**:

| Benchmark | Task Description | # Test | Answer Type |
|-----------|-----------------|--------|-------------|
| GSM8K | Grade school math word problems | 1,319 | Numeric |
| SVAMP | Math word problems with structural variation | 1,000 | Numeric |
| ASDiv | Diverse math word problems | 2,096 | Numeric |
| AQuA | Algebraic word problems (multiple choice) | 254 | Choice (A-E) |
| MAWPS | Math word problems (SingleEq+AddSub+MultiArith) | ~2,000 | Numeric |

**Commonsense Reasoning (2 benchmarks)**:

| Benchmark | Task Description | # Test | Answer Type |
|-----------|-----------------|--------|-------------|
| CommonsenseQA (CSQA) | 5-way multiple choice commonsense | 1,221 | Choice (A-E) |
| StrategyQA | Yes/no questions requiring multi-hop strategy | 2,290 | Yes/No |

**Symbolic Reasoning (2 benchmarks)**:

| Benchmark | Task Description | Example |
|-----------|-----------------|---------|
| Last Letter Concatenation | Concatenate last letters of words | "Amy Brown" -> "yn" |
| Coin Flip | Track coin state after sequence of flips/no-flips | heads -> flip -> no flip -> ? |

### Prompting Details

- **Number of exemplars**: 8 for all arithmetic and commonsense tasks
- **Exemplar selection**: Manually written by the authors
- **CoT style**: Natural language reasoning steps (not equations or code)
- **Decoding**: Greedy decoding (temperature=0) for main results
- **Answer extraction**: Parse final numeric/choice answer after "The answer is"

---

## Results

### Main Results: Arithmetic Reasoning (Accuracy %)

```
+----------+-----------+-----------+-----------+-----------+-----------+
|          |   GSM8K   |   SVAMP   |   ASDiv   |   AQuA    |   MAWPS   |
| Method   | Std | CoT | Std | CoT | Std | CoT | Std | CoT | Std | CoT |
+----------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| LaMDA    |     |     |     |     |     |     |     |     |     |     |
|  137B    | 17.1| 27.6| 39.9| 53.0|  --  | --  |  --  | --  | --  | --  |
+----------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| GPT-3    |     |     |     |     |     |     |     |     |     |     |
|  175B    | 18.1| 49.6| 65.7| 74.8| 71.3| 76.4| 24.8| 38.9| 78.7| 93.0|
+----------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| Codex    |     |     |     |     |     |     |     |     |     |     |
|  (175B)  | 19.7| 65.6| 69.9| 86.8|  -- |  -- |  -- |  -- |  -- |  -- |
+----------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| PaLM     |     |     |     |     |     |     |     |     |     |     |
|  540B    | 17.9| 56.9| 69.3| 79.0| 74.0| 78.7| 25.2| 35.8| 84.7| 93.3|
+----------+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
| Prior    |     |     |     |     |     |     |     |     |     |     |
| SOTA     |   55.0*   |   57.4    |   75.3    |   37.9    |   --      |
| (FT)     |  (ft+ver) | (ft)      | (ft)      | (ft)      |           |
+----------+-----------+-----------+-----------+-----------+-----------+

* Fine-tuned GPT-3 175B + learned verifier (Cobbe et al. 2021)
```

**Key findings**:
- PaLM 540B + CoT (56.9%) surpasses the prior fine-tuned SOTA on GSM8K (55.0%)
  that used GPT-3 175B fine-tuning + a specially trained solution verifier
- CoT provides the largest absolute gains on the hardest benchmarks (GSM8K:
  +39.0 for PaLM, SVAMP: +16.9 for Codex)
- GPT-3 175B + CoT on MAWPS (93.0%) approaches near-perfect accuracy
- PaLM 540B + CoT achieves new SOTA on GSM8K, SVAMP, and MAWPS

### Commonsense Reasoning Results (Accuracy %)

```
+----------+---------------------+--------------------+
|          |   CommonsenseQA     |    StrategyQA      |
| Method   | Standard |   CoT   | Standard |   CoT   |
+----------+----------+---------+----------+---------+
| LaMDA    |          |         |          |         |
|  137B    |   58.1   |  63.3   |   60.5   |  65.4   |
+----------+----------+---------+----------+---------+
| GPT-3    |          |         |          |         |
|  175B    |   73.5   |  74.7   |   63.6   |  65.4   |
+----------+----------+---------+----------+---------+
| PaLM     |          |         |          |         |
|  540B    |   65.5   |  74.4   |   68.6   |  75.6   |
+----------+----------+---------+----------+---------+
| Prior    |          |         |          |         |
| SOTA     |     79.0 (ft)     |     69.4 (ft)     |
+----------+---------------------+--------------------+
```

**Key findings**:
- PaLM 540B + CoT (75.6%) surpasses the prior fine-tuned SOTA on
  StrategyQA (69.4%) by a significant margin
- CommonsenseQA improvements are more modest, suggesting CoT has larger
  benefits for tasks requiring multi-hop strategic reasoning

### Additional Benchmark Results (PaLM 540B)

```
+---------------------------+----------+---------+
| Benchmark                 | Standard |   CoT   |
+---------------------------+----------+---------+
| Date Understanding        |   62.8   |  67.5   |
| Sports Understanding      |   91.4   |  95.4   |
| Last Letter (in-domain)   |   ~0     |  ~67    |
| Coin Flip (in-domain)     |  100.0   | 100.0   |
+---------------------------+----------+---------+
```

- Sports Understanding with CoT (95.4%) surpasses an unaided sports
  enthusiast (84%)
- Last Letter Concatenation shows CoT's ability to enable symbolic
  reasoning that is impossible with standard prompting

### Scaling Analysis: Emergent Ability of CoT

Performance across model sizes demonstrates the emergence phenomenon:

```
+------------+----------------------------+----------------------------+
| Model Size | GSM8K                      | MAWPS                      |
|            | Standard     | CoT         | Standard     | CoT         |
+------------+-------------+-------------+--------------+-------------+
| ~422M      |    2.3      |    3.0      |    17.3      |    19.2     |
| ~2B        |    3.8      |    4.1      |    33.2      |    32.4     |
| ~8B        |    5.0      |    4.8      |    51.3      |    47.1     |
| ~68B       |   10.4      |   12.4      |    68.0      |    72.3     |
| ~137B      |   17.1      |   27.6      |    77.4      |    85.1     |
| ~540B      |   17.9      |   56.9      |    84.7      |    93.3     |
+------------+-------------+-------------+--------------+-------------+
```

**Critical observations**:
- Below ~100B parameters, CoT provides no benefit or even slightly hurts
  performance (e.g., 8B PaLM: GSM8K standard 5.0 vs CoT 4.8)
- The emergence threshold is around 10^23 training FLOPs (~100B parameters)
- Above the threshold, CoT gains accelerate dramatically with scale
  (PaLM 540B: standard 17.9 -> CoT 56.9 = +39.0 absolute gain on GSM8K)
- For easier tasks (MAWPS), gains begin to appear at smaller scales (~68B)
  because less reasoning depth is required

---

## Ablation Studies

### What Components of CoT Matter?

The authors tested several ablation variants using LaMDA 137B on GSM8K:

```
+-------------------------------+----------+
| Ablation Variant              | Accuracy |
+-------------------------------+----------+
| Standard prompting (baseline) |   17.1   |
| Chain of thought (full CoT)   |   27.6   |
+-------------------------------+----------+
| Equation only                 |   18.1   |
| Variable computation only     |   20.5   |
| CoT after answer              |   18.5   |
+-------------------------------+----------+
```

**Findings from ablations**:

1. **Equation only**: Writing just the mathematical equation (e.g., "5 + 2*3 = 11")
   provides minimal benefit (+1.0). The natural language reasoning steps are
   essential -- the model cannot directly translate problem semantics into
   equations without them.

2. **Variable computation only**: Including variable assignments and computation
   steps but not natural language explanations gives modest improvement (+3.4),
   but significantly less than full CoT (+10.5).

3. **CoT after answer**: Placing the chain of thought _after_ the answer
   (instead of before) provides no meaningful benefit (+1.4). This demonstrates
   that the reasoning must precede the answer to be useful -- it is not merely
   a post-hoc rationalization but an active computation that guides the answer.

### Robustness Analyses

**Exemplar ordering**: Performance is robust to different orderings of the
few-shot exemplars. Standard deviation across 5 random orderings is small.

**Number of exemplars**: Performance improves with more exemplars but shows
diminishing returns. Even 1-2 exemplars provide substantial benefits at scale.

**Annotator variation**: Different annotators writing different chain-of-thought
exemplars yield similar performance, suggesting the method is not overly
sensitive to the specific writing style.

### Correctness of Generated Chains

Manual analysis of 50 randomly sampled CoT traces from LaMDA 137B on GSM8K:

```
+----------------------------------+----------+
| Category                         | % of 50  |
+----------------------------------+----------+
| Correct reasoning, correct ans.  |   ~50    |
| Correct reasoning, wrong answer  |    2     |
|   (arithmetic/extraction error)  |          |
| Wrong reasoning, correct answer  |    8     |
|   (lucky guess / right for wrong |          |
|    reasons)                      |          |
| Wrong reasoning, wrong answer    |   ~40    |
+----------------------------------+----------+
```

Key insight: most correct answers come from correct reasoning chains, not
lucky guesses. Only 8% of cases had correct answers with flawed reasoning.

---

## Analysis & Insights

### Why Does CoT Work?

The paper offers several explanations for CoT's effectiveness:

1. **Decomposition**: Multi-step problems are broken into manageable sub-steps,
   each within the model's capability even if the full problem is not.

2. **Intermediate variable binding**: Each step creates named intermediate
   results (e.g., "Roger has 5 balls") that the model can reference in
   subsequent steps, effectively extending working memory.

3. **Error localization**: When the model makes a mistake, it is often
   localized to one step rather than causing a cascade of errors.

4. **Allocation of computation**: The chain of thought effectively gives the
   model more "compute time" (more tokens to generate) before committing to
   an answer, similar to System 2 thinking in dual-process theory.

### Why Only at Scale?

The authors hypothesize that:

- Small models lack the **language modeling capability** to generate coherent
  multi-step reasoning chains, even when shown examples
- Small models may **copy the surface format** of CoT (generating text that
  looks like reasoning) without actually performing logical reasoning
- The emergence threshold corresponds to the point where models develop
  sufficient **compositional generalization** ability

### Comparison with Fine-tuning

```
+-----------------------------------+----------+-------------+
| Approach                          | GSM8K    | Training    |
|                                   | Accuracy | Data Needed |
+-----------------------------------+----------+-------------+
| GPT-3 175B (fine-tuned)           |   ~33    |  7,473      |
| GPT-3 175B (FT + verifier)       |   55.0   |  7,473+     |
| PaLM 540B (8-shot CoT prompting) |   56.9   |  0 (8 ex.)  |
+-----------------------------------+----------+-------------+
```

CoT prompting achieves SOTA on GSM8K with zero training data -- just 8
hand-written exemplars -- compared to fine-tuning on thousands of examples
plus training a separate verifier model.

---

## Limitations & Critiques

1. **Scale Dependency**: CoT only works at ~100B+ parameters, making it
   inaccessible for smaller, more deployable models. This limits practical
   applicability to organizations with access to the largest models.

2. **No Guarantee of Correct Reasoning**: The model can generate plausible-
   sounding but logically flawed reasoning chains. Correct final answers
   can result from incorrect reasoning (~8% of cases).

3. **Cost of Generation**: CoT requires generating many more tokens per
   query than standard prompting, increasing latency and cost proportionally
   to the length of the reasoning chain.

4. **Prompt Engineering Burden**: The quality of the few-shot exemplars
   matters significantly. Writing good chain-of-thought exemplars requires
   human effort and task understanding.

5. **Evaluation Difficulty**: It is hard to automatically evaluate whether
   a reasoning chain is correct vs. merely arriving at the right answer
   for the wrong reasons.

6. **Limited to Sequential Reasoning**: CoT generates a single linear chain.
   Problems requiring exploration, backtracking, or parallel reasoning
   paths are not well served (addressed by later work like Tree of Thoughts
   and Self-Consistency).

7. **Benchmark Saturation Concerns**: Some benchmarks (MAWPS at 93.3%)
   approach ceiling, making it hard to assess the true reasoning capability
   vs. pattern matching on these specific datasets.

8. **Hallucination in Chains**: While CoT reduces final-answer errors, the
   intermediate steps themselves can contain fabricated facts or flawed
   logic that happen to produce correct answers.

---

## Follow-up Work

- **Self-Consistency** (Wang et al., 2022): Sample multiple CoT paths and
  take majority vote on the answer. Achieves 74.4% on GSM8K with PaLM-540B
  (vs. 56.9% with greedy CoT).
- **Zero-shot CoT** (Kojima et al., 2022): Simply appending "Let's think
  step by step" to the prompt enables CoT without exemplars.
- **Least-to-Most Prompting** (Zhou et al., 2022): Decompose complex problems
  into simpler sub-problems, solve sequentially.
- **Tree of Thoughts** (Yao et al., 2023): Extend CoT to tree-structured
  exploration with backtracking.
- **ReAct** (Yao et al., 2022): Interleave CoT reasoning with grounded
  actions (tool use), addressing hallucination.
- **PAL** (Gao et al., 2022): Use code (not natural language) as the
  intermediate reasoning format.
- **Scratchpad** (Nye et al., 2021): Concurrent work on intermediate
  computation steps via fine-tuning (not prompting).
- **STaR** (Zelikman et al., 2022): Self-taught Reasoner -- bootstrap
  CoT capability through iterative self-training.
- **Faithful CoT** (Lyu et al., 2023): Ensure reasoning chains are
  logically faithful to the final answer.

---

## Key Takeaways

1. **Chain-of-thought prompting is a simple but transformative technique.**
   Adding intermediate reasoning steps to few-shot exemplars unlocks
   dramatic improvements in LLM reasoning (e.g., +39 points on GSM8K).

2. **CoT is an emergent ability of scale.** Below ~100B parameters, CoT
   provides no benefit. This is one of the clearest demonstrations of
   emergent capabilities in large language models.

3. **Natural language reasoning outperforms pure symbolic computation.**
   Equation-only and variable-only ablations show significantly less
   benefit than full natural language chains, suggesting the semantic
   content of the reasoning steps matters.

4. **Prompting can match fine-tuning.** PaLM 540B + 8-shot CoT surpasses
   GPT-3 175B fine-tuned on 7,473 examples + verifier on GSM8K, a
   striking result for the efficiency of prompting approaches.

5. **The reasoning must precede the answer.** The ablation showing CoT-after-
   answer fails confirms that the chain of thought serves as genuine
   intermediate computation, not post-hoc rationalization.

6. **CoT became the intellectual foundation for LLM agents.** Every modern
   agent framework (ReAct, Reflexion, AutoGPT) relies on some form of
   chain-of-thought reasoning for planning and decision-making. CoT
   established that LLMs can "think before they act."

7. **The quality of reasoning chains matters more than quantity.** Manual
   analysis shows most correct answers come from correct reasoning (~50%)
   rather than lucky guesses (~8%), validating that CoT genuinely elicits
   reasoning rather than exploiting superficial patterns.

8. **CoT's limitations directly motivated subsequent work.** The linearity
   limitation led to Tree of Thoughts; the hallucination problem led to
   ReAct's grounded actions; the single-path limitation led to Self-
   Consistency's multi-path voting.
