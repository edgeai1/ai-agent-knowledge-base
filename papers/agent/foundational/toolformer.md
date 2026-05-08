---
title: "Toolformer: Language Models Can Teach Themselves to Use Tools"
authors:
  - Timo Schick
  - Jane Dwivedi-Yu
  - Roberto Dessi
  - Roberta Raileanu
  - Maria Lomeli
  - Eric Hambro
  - Luke Zettlemoyer
  - Nicola Cancedda
  - Thomas Scialom
venue: NeurIPS 2023 (Advances in Neural Information Processing Systems 36)
year: 2023
url: https://arxiv.org/abs/2302.04761
institution: Meta AI Research
tags:
  - tool-use
  - self-supervised-learning
  - language-models
  - API-calls
  - augmented-LMs
  - agents
  - foundational
status: done
---

# Toolformer: Language Models Can Teach Themselves to Use Tools

## TL;DR

Toolformer is a self-supervised method that teaches a language model (GPT-J 6.7B) to autonomously
decide when and how to call external tool APIs (calculator, Q&A system, search engine, calendar,
translator) by embedding API calls directly into text sequences. The model generates candidate API
call annotations on a large corpus, filters them by whether they reduce perplexity on future tokens,
and fine-tunes on the surviving annotations. The resulting 6.7B model matches or exceeds GPT-3 175B
on several zero-shot benchmarks -- achieving this with 25x fewer parameters.

---

## Motivation & Problem

Large language models exhibit fundamental limitations that cannot be resolved by scaling alone:

1. **Inability to access up-to-date information** -- LMs are frozen at training time and cannot
   retrieve current facts, prices, weather, or news.
2. **Weak mathematical reasoning** -- even 175B-parameter models fail on basic multi-digit
   arithmetic and word problems.
3. **Lack of temporal awareness** -- LMs have no concept of the current date or time.
4. **Hallucination on factual recall** -- parametric memory is unreliable for long-tail knowledge.
5. **Limited multilinguality** -- translation quality degrades for low-resource language pairs.

Prior approaches to tool-augmented LMs (e.g., TALM, WebGPT, LaMDA) relied on extensive human
annotations or task-specific supervision. Toolformer asks: can a model teach *itself* when tools
are helpful, using only a few demonstrations per tool and no task-specific labels?

---

## Method

### High-Level Pipeline

The approach consists of three phases applied independently for each tool:

```
Phase 1: Annotate        Phase 2: Filter           Phase 3: Fine-tune
+-----------------+      +------------------+      +------------------+
| Sample candidate|      | Execute API calls|      | Merge all tool   |
| API call        | ---> | Compare loss with| ---> | datasets         |
| positions &     |      | vs without result|      | Fine-tune GPT-J  |
| arguments from  |      | Keep if loss     |      | on augmented     |
| corpus C        |      | reduction >= t_f |      | text + original C|
+-----------------+      +------------------+      +------------------+
```

### Step-by-Step Annotation Pipeline

**Step 1: Position Sampling**

Given a text sequence x = (x_1, ..., x_n) from corpus C (a subset of CCNet), compute for each
position i the probability that the model M assigns to generating the special `<API>` token:

```
p_i = P_M(<API> | x_1, ..., x_i)
```

Keep all positions where p_i > tau_s (sampling threshold). If more than k positions qualify,
retain only the top-k by probability. This is bootstrapped using few-shot prompting: the model
is given 2-3 examples of text with API calls for each tool, which biases it to assign non-trivial
probability to `<API>` at relevant positions.

- **tau_s = 0.05** (5% probability threshold)
- **k = 5** candidate positions per sequence (at most)

**Step 2: Candidate Generation**

For each selected position i, sample m candidate API calls by continuing generation from the
prefix [x_1, ..., x_i, <API>] with the tool-specific prompt format until the model generates
`</API>`. This yields candidate calls c_1, ..., c_m.

- **m = 5** candidates per position
- Sampling uses nucleus sampling or temperature-based decoding

**Step 3: API Execution**

Execute each candidate API call c_j to obtain the result r_j. Construct two augmented sequences:

```
e_i^+ = (x_1,...,x_{i-1}, <API> c_j -> r_j </API>, x_i,...,x_n)   [with result]
e_i^- = (x_1,...,x_{i-1}, <API> c_j ->  </API>, x_i,...,x_n)       [empty result]
```

**Step 4: Perplexity-Based Filtering**

Compute the weighted cross-entropy loss over tokens following the API call for both variants:

```
                   n
L_i(z) = - SUM   w_{t-i} * log P_M(x_t | z, x_{1:t-1})
               t=i
```

where:
- z is the inserted API call (with or without result)
- w_{t-i} are position-dependent weights (tokens closer to the API call weighted more heavily)
- The sum runs over all tokens after position i

Define two key quantities:

```
L_i^+ = L(e_i^+)    -- loss WITH the API result
L_i^- = L(e_i^-)    -- loss WITHOUT the API result (or with empty string)
```

**Filtering criterion:** Keep API call c_j at position i if and only if:

```
L_i^- - L_i^+ >= tau_f
```

where **tau_f = 1.0** (the API call must reduce loss by at least 1.0 nat).

This means: the call is retained only when having the tool's answer makes the model substantially
better at predicting the subsequent tokens compared to not having it. This is the core insight --
the model itself judges tool usefulness via its own loss function.

### Pseudocode: Full Annotation Pipeline

```
Algorithm: Toolformer Self-Supervised Annotation

Input:  Corpus C, Model M, Tool set T, thresholds tau_s, tau_f, limits k, m
Output: Augmented corpus C*

C* = {}
for each tool t in T:
    for each text x = (x_1,...,x_n) in C:
        # Step 1: Sample positions
        positions = {}
        for i = 1 to n:
            p_i = P_M(<API> | prompt_t, x_1,...,x_i)
            if p_i > tau_s:
                positions.add((i, p_i))
        positions = top_k(positions, k)  # keep top-k by probability

        # Step 2: Generate candidates
        for each position i in positions:
            candidates = {}
            for j = 1 to m:
                c_j = sample_continuation(M, [x_1,...,x_i, <API>], stop=</API>)
                candidates.add(c_j)

            # Step 3: Execute and filter
            for each c_j in candidates:
                r_j = execute_tool(t, c_j)
                L_plus  = weighted_loss(M, x, i, api_call=c_j, result=r_j)
                L_minus = weighted_loss(M, x, i, api_call=c_j, result="")
                if L_minus - L_plus >= tau_f:
                    x* = insert_annotation(x, i, c_j, r_j)
                    C*.add(x*)
                    break  # use first passing candidate

# Step 4: Fine-tune
C_final = C* UNION C  # merge annotated + original data
M* = finetune(M, C_final, lr=1e-5, warmup=10%)
return M*
```

### Tool API Formats

Each tool uses a consistent annotation syntax: `[ToolName(input) -> output]`

**1. Calculator (Calc)**
```
Syntax:   [Calculator(mathematical_expression) -> result]
Example:  Out of 1400 participants, 400 (or [Calculator(400/1400) -> 0.29] 29%) passed.
Scope:    Four basic arithmetic operations (+, -, *, /)
Output:   Rounded to two decimal places
```

**2. Question Answering (QA)**
```
Syntax:   [QA(natural_language_question) -> answer]
Example:  The NEJM is a trademark of [QA("Publisher of NEJM?") -> Massachusetts Medical
          Society] the MMS.
Backend:  Atlas retrieval-augmented LM (few-shot, no fine-tuning)
```

**3. Wikipedia Search (WikiSearch)**
```
Syntax:   [WikiSearch(query) -> first_sentence_of_article]
Example:  The Brown Act is [WikiSearch("Brown Act") -> The Ralph M. Brown Act is an act
          of the California State Legislature...] California's open meetings law.
Backend:  BM25 retrieval over Wikipedia, returns first sentence of top result
```

**4. Machine Translation (MT)**
```
Syntax:   [MT(text_in_source_language) -> translated_text]
Example:  "la tortuga", the Spanish word for [MT("tortuga") -> turtle] turtle.
Backend:  600M-parameter NLLB model (200 languages)
```

**5. Calendar**
```
Syntax:   [Calendar() -> current_date_string]
Example:  The WL opens on Friday, [Calendar() -> Today is Thursday, March 9, 2017.]
          March 10.
Backend:  Simple system call returning current date (no input arguments)
```

### Fine-Tuning Details

After filtering, the surviving annotated examples from all tools are merged with the original
unannotated corpus C. The model is then fine-tuned on this combined dataset.

- **Base model:** GPT-J 6.7B (EleutherAI)
- **Training corpus:** Subset of CCNet
- **Learning rate:** 1e-5 with linear warmup for the first 10% of training steps
- **Weight decay:** 0.01
- **Sequence length:** 1024 tokens
- **Objective:** Standard language modeling (next token prediction)
- **Dataset composition:** Augmented examples interleaved with original text to prevent
  catastrophic forgetting of general language modeling ability

At inference time, when the model generates `<API>`, generation pauses, the API call is executed,
the result is inserted, and generation continues from `</API>`.

---

## Key Innovations

1. **Self-supervised tool annotation** -- No human labeling of when to use tools. The model
   discovers useful tool invocations entirely through its own loss signal. Only a handful
   (2-3) of demonstrations per tool are needed to bootstrap the process.

2. **Perplexity-as-reward** -- Using the model's own next-token prediction loss as a filtering
   criterion is elegant: it directly measures whether a tool call helps the model do its job
   better. This avoids needing task-specific reward models.

3. **Tool-agnostic framework** -- The same annotate/filter/fine-tune pipeline works for any
   tool that can be expressed as a text-in/text-out API. Adding a new tool only requires
   writing a few demonstration examples.

4. **Inline API calls** -- Tool invocations are embedded directly in the token stream using
   special tokens, rather than requiring a separate action space or multi-turn protocol. This
   preserves the autoregressive generation paradigm.

5. **Decoupled tool training** -- Each tool's annotations are generated independently, allowing
   parallelism and modularity. The final fine-tuning merges all tools.

---

## Experimental Setup

### Datasets and Benchmarks

| Category   | Dataset     | Task                          | Metric   |
|------------|-------------|-------------------------------|----------|
| Math       | ASDiv       | Arithmetic word problems      | Accuracy |
| Math       | SVAMP       | Math word problems            | Accuracy |
| Math       | MAWPS       | Math word problems            | Accuracy |
| QA         | WebQS       | Open-domain QA (web)          | Accuracy |
| QA         | NQ          | Natural Questions             | Accuracy |
| QA         | TriviaQA    | Trivia question answering     | Accuracy |
| Knowledge  | LAMA-SQuAD  | Factual cloze (SQuAD subset)  | Accuracy |
| Knowledge  | LAMA-T-REx  | Factual cloze (T-REx subset)  | Accuracy |
| Knowledge  | LAMA-G-RE   | Factual cloze (Google-RE)     | Accuracy |
| Temporal   | TempLAMA    | Time-sensitive factual cloze  | Accuracy |
| Temporal   | DateSet     | Date understanding            | Accuracy |
| Translation| MLQA        | Multilingual QA (6 languages) | F1       |

### Baselines

- **GPT-J 6.7B** -- base model, no tools
- **GPT-J 6.7B fine-tuned on CCNet** -- same data, no tool annotations
- **OPT 66B** -- 10x larger, no tools
- **GPT-3 175B** -- 25x larger, no tools (via OpenAI API)
- **Toolformer disabled** -- fine-tuned with tool data but tool calls suppressed at inference

### Model Variants for Ablation

- GPT-2 124M, GPT-2 355M, GPT-2 775M, GPT-2 1.6B, GPT-J 6.7B

---

## Results

### Main Results Table

| Benchmark    | GPT-J 6.7B | OPT 66B | GPT-3 175B | Toolformer 6.7B |
|--------------|-----------|---------|------------|-----------------|
| ASDiv        | 7.5       | --      | 14.0       | **40.4**        |
| SVAMP        | 5.2       | 4.9     | 10.0       | **29.4**        |
| MAWPS        | 9.9       | --      | 19.8       | **44.0**        |
| SQuAD (LAMA) | 19.2      | --      | --         | **33.8**        |
| WebQS        | 18.5      | --      | --         | 26.3            |
| NQ           | 12.8      | --      | --         | 17.7            |
| TriviaQA     | 43.9      | --      | --         | 48.8            |
| TempLAMA     | 13.7      | --      | --         | 16.3            |
| DateSet      | 3.9       | --      | --         | **27.3**        |

Key observations:
- On math benchmarks (ASDiv, SVAMP, MAWPS), Toolformer **quadruples** GPT-J performance
  and **doubles to triples** GPT-3 performance using only 6.7B parameters
- On LAMA-SQuAD, Toolformer uses the QA tool in 98.1% of cases, boosting from 19.2 to 33.8
- On DateSet, Toolformer achieves a ~7x improvement by invoking the Calendar tool
- On QA benchmarks (WebQS, NQ, TriviaQA), gains are more modest since factual recall partially
  overlaps with the model's parametric knowledge
- Toolformer easily outperforms OPT 66B and GPT-3 175B on all math benchmarks
- On LAMA knowledge benchmarks, Toolformer outperforms all baselines of the same size and
  achieves results competitive with or exceeding GPT-3

### Tool Usage Analysis

| Tool          | Primary Benchmarks   | Usage Rate    | Benefit Level |
|---------------|----------------------|---------------|---------------|
| Calculator    | ASDiv, SVAMP, MAWPS  | Very high     | Very high     |
| QA (Atlas)    | LAMA, WebQS, NQ      | ~98% on LAMA  | High          |
| WikiSearch    | LAMA (some subsets)  | Moderate      | Moderate      |
| Calendar      | TempLAMA, DateSet    | High on date  | High          |
| MT (NLLB)     | MLQA                 | Variable      | Mixed         |

- The **Calculator** provides the single largest benefit -- math benchmarks see 4-5x gains
- The **QA tool** is used most frequently on knowledge benchmarks with consistent improvements
- **WikiSearch** shows moderate benefit; partially redundant with QA on some benchmarks
- **Calendar** is crucial for temporal reasoning but narrow in applicability
- **MT** provides benefits across most languages on MLQA, but CCNet fine-tuning can hurt
  multilingual performance, so Toolformer does not consistently outperform base GPT-J

### Ablation: Model Size and Tool Use Emergence

| Model Size | Uses Tools Effectively? | Notes                                         |
|------------|------------------------|-----------------------------------------------|
| 124M       | No                     | No improvement from tool availability         |
| 355M       | No                     | No improvement from tool availability         |
| 775M       | Partially              | First signs of tool use; WikiSearch works     |
| 1.6B       | Mostly                 | Reasonable tool use across most APIs          |
| 6.7B       | Yes                    | Full tool use capability; strong gains        |

Critical finding: **Tool use is an emergent capability** that only appears reliably at ~775M
parameters. Below this scale, models cannot learn when tools are helpful, even with the same
training signal. The exception is WikiSearch, which is "comparably easy to use" and shows some
benefit even at smaller scales.

---

## Analysis & Insights

### Why Self-Supervision Works

The perplexity filtering criterion acts as a natural reward signal. An API call is only useful if
it provides information the model could not have predicted on its own. This creates an automatic
curriculum: easy factual completions where the model already knows the answer get filtered out
(the API call does not reduce loss), while genuinely difficult predictions where the tool helps
are retained.

### The "Right" Level of Abstraction

Toolformer operates at the token level -- API calls are just special token sequences. This means:
- No separate "action space" is needed
- No RL or reward modeling is required
- Standard language modeling training works
- The model naturally learns to place calls at contextually appropriate positions

### Distribution of Tool Calls

The model learns intuitive placement patterns:
- Calculator calls appear right before numerical results in text
- QA calls appear before entity names or factual statements
- Calendar calls appear near date references
- MT calls appear near foreign-language terms

### Scaling Dynamics

While models become better at solving tasks without API calls as they grow in size, their ability
to make good use of provided APIs improves at the same time. The tool-use benefit and the raw
capability benefit are additive, not substitutive.

### Comparison with Contemporary Work

Toolformer predates and differs from subsequent agent frameworks (ReAct, function calling) in that
it does NOT use chain-of-thought reasoning or multi-step planning. Each API call is a single,
independent invocation. This is both a strength (simplicity) and a limitation (no multi-step
tool use).

---

## Limitations & Critiques

1. **No tool chaining** -- API calls are generated independently per tool. The output of one tool
   cannot be fed as input to another. This severely limits complex reasoning that requires
   multi-step tool use (e.g., "search for X, then calculate Y from the result").

2. **No interactive tool use** -- The model cannot browse search results, refine queries, or
   engage in multi-turn interactions with tools. It gets one shot per call.

3. **Limited to text-in/text-out APIs** -- Tools that require structured input (JSON schemas,
   images, files) or produce non-textual output cannot be integrated.

4. **Scale requirements** -- The approach only works for models >= 775M parameters, which limits
   applicability to smaller or edge-deployed models.

5. **CCNet fine-tuning trade-off** -- Fine-tuning on CCNet can hurt performance on some tasks
   (e.g., multilingual QA), making it hard to disentangle tool benefits from data effects.

6. **Static tool set** -- Adding new tools requires re-running the full annotation/filter/fine-tune
   pipeline. No plug-and-play tool addition at inference time.

7. **Evaluation gaps** -- The paper does not evaluate on tasks requiring genuine multi-step
   reasoning, planning, or complex tool orchestration -- the exact scenarios where agents are
   most needed.

8. **Single-tool per position** -- Each annotated position uses exactly one tool. The model cannot
   reason about which of multiple tools would be most appropriate for a given context at the
   same position.

9. **Annotation quality depends on base model** -- If the base model assigns low probability to
   `<API>` at genuinely useful positions (distribution mismatch), those positions are never
   sampled and potentially valuable tool uses are missed.

---

## Follow-up Work

- **Gorilla (2023)** -- Extended tool learning to thousands of APIs with retrieval-augmented
  generation for API documentation.
- **ToolLLM / ToolBench (2023)** -- Scaled to 16,000+ real-world APIs with multi-step tool use.
- **ART (Automatic Reasoning and Tool-use, 2023)** -- Combined chain-of-thought with tool use
  in a multi-step framework.
- **Chameleon (2023)** -- Plug-and-play tool composition with an LLM-based planner.
- **TaskMatrix.AI / HuggingGPT (2023)** -- Used LLMs to orchestrate many specialized models
  as tools.
- **Function Calling (OpenAI, 2023)** -- Commercial implementation of structured tool use with
  JSON schemas, directly inspired by the Toolformer paradigm.
- **ReAct (2022)** -- Interleaved reasoning and acting; complementary approach that adds
  chain-of-thought before tool calls.

---

## Key Takeaways

1. **Self-supervision suffices for tool learning** -- With only 2-3 demonstrations per tool, a
   model can learn when and how to use tools by filtering on its own loss. No human annotation
   of tool-use examples at scale is needed.

2. **Small models + tools > large models alone** -- Toolformer 6.7B + Calculator beats GPT-3
   175B on math by 2-3x. This suggests that tool augmentation can be more parameter-efficient
   than scaling.

3. **Perplexity is a universal tool-usefulness signal** -- The loss-reduction criterion
   (L_i^- - L_i^+ >= tau_f) is elegant, principled, and tool-agnostic. It can be applied
   to any text-in/text-out tool without modification.

4. **Tool use is emergent** -- It requires a minimum model scale (~775M) to appear, suggesting
   tool use depends on sufficient reasoning capability in the base model.

5. **The inline-API paradigm is powerful but limited** -- Embedding tool calls in the token
   stream is simple and effective for single-step tool use, but does not support the multi-step,
   interactive tool use that modern agent frameworks require.

6. **Foundational contribution** -- Toolformer established that LMs can autonomously learn
   tool use, directly inspiring the function-calling and tool-use capabilities now standard
   in commercial LLMs (GPT-4, Claude, Gemini).
