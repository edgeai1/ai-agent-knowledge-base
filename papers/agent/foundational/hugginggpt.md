---
title: "HuggingGPT: Solving AI Tasks with ChatGPT and Its Friends in Hugging Face"
authors: "Yongliang Shen, Kaitao Song, Xu Tan, Dongsheng Li, Weiming Lu, Yueting Zhuang"
venue: "NeurIPS 2023"
year: 2023
arxiv: "https://arxiv.org/abs/2303.17580"
code: "https://github.com/microsoft/JARVIS"
institution: "Zhejiang University, Microsoft Research Asia"
aliases: ["JARVIS"]
tags: [orchestration, multi-model, task-planning, tool-use, multi-modal, LLM-agent, foundational]
status: done
date_reviewed: 2026-05-08
---

# HuggingGPT: Solving AI Tasks with ChatGPT and Its Friends in Hugging Face

## TL;DR

HuggingGPT (codenamed JARVIS) uses an LLM (ChatGPT) as a central controller to orchestrate
hundreds of specialist AI models hosted on Hugging Face Hub. It decomposes user requests
into structured sub-tasks via a four-stage pipeline -- task planning (with dependency
parsing), model selection (download-count ranking + LLM reasoning), task execution (with
resource management and parallel scheduling), and response generation -- enabling a single
system to handle complex multi-modal tasks spanning language, vision, audio, and video
without any model fine-tuning.

## Motivation & Problem

By early 2023, a fundamental tension characterized the AI landscape:

1. **Specialist model proliferation**: Thousands of task-specific models on Hugging Face
   (object detection, image segmentation, speech recognition, text-to-image, etc.), each
   excelling narrowly but unable to handle multi-step or cross-modal requests.

2. **LLM generality vs. specialization**: ChatGPT/GPT-4 exhibit broad language
   understanding and planning ability but cannot directly perform vision, audio, or
   video tasks without additional tools.

3. **User burden**: Solving "generate an image of a cat, detect objects in it, and
   describe the scene in speech" requires manually identifying models, chaining I/O,
   and handling format conversions.

**Prior work limitations**:
- **Visual ChatGPT** (Wu et al., 2023): Connects ChatGPT to a fixed set of vision
  models -- limited to pre-defined visual tools, not extensible.
- **Toolformer** (Schick et al., 2023): Teaches LLMs to use tools via fine-tuning --
  supports only a handful of APIs and requires training.
- **TaskMatrix.AI** (Liang et al., 2023): Proposes connecting foundation models to
  millions of APIs -- remained a conceptual framework with no empirical evaluation.

HuggingGPT's insight: use the LLM as a **task planner and controller** (not executor),
leveraging Hugging Face's model descriptions as a dynamic, continuously growing tool
registry. The system's capabilities expand automatically as new models are published.

## Method

### Four-Stage Pipeline Architecture

```
  User Request: "Generate an image of a girl reading, detect objects, describe in speech"
       |
       v
  +---------------------------------------------------------------+
  | STAGE 1: TASK PLANNING (ChatGPT)                               |
  |   Parse request -> structured task DAG                         |
  |   Output: [{task, id, dep, args}, ...]                         |
  +---------------------------------------------------------------+
       |
       v
  +---------------------------------------------------------------+
  | STAGE 2: MODEL SELECTION (ChatGPT)                             |
  |   For each task: retrieve top-K models from HF Hub             |
  |   Rank by downloads -> LLM selects best fit                    |
  +---------------------------------------------------------------+
       |
       v
  +---------------------------------------------------------------+
  | STAGE 3: TASK EXECUTION (HF Inference API / Local)             |
  |   Execute in topological order of DAG                          |
  |   Independent tasks run in parallel                            |
  |   Handle resource passing between dependent tasks              |
  +---------------------------------------------------------------+
       |
       v
  +---------------------------------------------------------------+
  | STAGE 4: RESPONSE GENERATION (ChatGPT)                         |
  |   Aggregate execution logs + outputs                           |
  |   Generate coherent natural language response for user         |
  +---------------------------------------------------------------+
```

### Stage 1: Task Planning with Dependency Parsing

The LLM parses user requests into structured JSON. Each task is a tuple of four fields:

```json
[
  {"task": "text-to-image",      "id": 1, "dep": [],  "args": {"text": "a girl reading a book"}},
  {"task": "object-detection",   "id": 2, "dep": [1], "args": {"image": "<generated>-1"}},
  {"task": "image-to-text",      "id": 3, "dep": [1], "args": {"image": "<generated>-1"}},
  {"task": "text-to-speech",     "id": 4, "dep": [3], "args": {"text": "<generated>-3"}}
]
```

**Field semantics**:
- `task`: Task type from HF taxonomy (~30 categories: text-classification, object-detection,
  text-to-image, automatic-speech-recognition, video-classification, etc.)
- `id`: Unique integer identifier for topological ordering
- `dep`: List of prerequisite task IDs (defines DAG edges)
- `args`: Input arguments; `<generated>-N` references output of task N

**Dependency DAG patterns** the planner handles:
```
Sequential:  T1 -> T2 -> T3
Parallel:    T1 -> T2,  T1 -> T3    (T2, T3 independent)
Diamond:     T1 -> T2,  T1 -> T3,  T2+T3 -> T4
```

**Prompt template** combines specification-based instruction (JSON schema + dependency
format definition) with demonstration-based parsing (3-5 few-shot examples of complex
request-to-task-list decomposition).

### Stage 2: Model Selection with Ranking Criteria

Four-step selection pipeline per task:
1. **Filter**: Retrieve HF Hub models matching the task type tag
2. **Rank**: Sort by download count (primary quality proxy)
3. **Truncate**: Take top-K candidates (K=3-5, limited by context window)
4. **Select**: LLM chooses best fit from candidates with descriptions

```
score(m) = w1 * rank_downloads(m) + w2 * relevance(desc(m), task) + w3 * likes(m)
selected = argmax_{m in top-K} score(m)
```

In practice, the LLM makes the final selection via in-context reasoning over model
descriptions. Download-count pre-ranking ensures only well-tested models reach selection.

### Stage 3: Task Execution with Resource Management

**Execution**: Via HF Inference API (cloud, default) or local GPU deployment.
**Scheduling**: Topological order of DAG; independent tasks run in parallel (rate-limited).
**Resource passing**: Upstream outputs stored locally, referenced by path for downstream.
**Error cascade**: API failure -> retry (3x) -> fallback model -> format conversion ->
skip dependent tasks -> report partial results.

### Stage 4: Response Generation

LLM receives structured execution logs (task, model, status, output per task) and
generates a coherent natural language summary with artifact references and failure notes.

## Key Innovations

1. **LLM as controller, not executor**: ChatGPT orchestrates specialist models across
   modalities, combining LLM planning capability with expert model execution.

2. **Dynamic tool registry**: Unlike Visual ChatGPT's fixed tool set, HuggingGPT accesses
   the entire HF Hub -- a continuously growing registry. New capabilities require zero
   system changes, only new model uploads by the community.

3. **Structured dependency parsing**: JSON-based task DAG enables complex multi-step,
   multi-modal workflows with explicit dependency tracking and parallel execution.

4. **Cross-modal composition**: Naturally chains models across domain boundaries
   (text -> image -> detection -> speech) to handle requests no single model could.

5. **No fine-tuning**: Operates entirely through in-context learning, immediately
   deployable with any sufficiently capable LLM.

## Experimental Setup

- **Controller LLMs**: ChatGPT (GPT-3.5-turbo), GPT-4; also tested Alpaca-7B, Vicuna-7B
- **Model Hub**: Hugging Face Hub (~30 task categories, thousands of models)
- **Evaluation corpus**: 130 diverse user requests across:
  - Single-task requests (one model invocation)
  - Sequential multi-task (ordered chain)
  - Graph-structured requests (complex DAG dependencies)
- **Evaluation metrics**:
  - Task planning: F1, accuracy (single tasks); F1, normalized Edit Distance (sequential)
  - Model selection: passing rate (%), rationality (%)
  - End-to-end: success rate (%) -- whether user request is fully resolved
- **Human evaluation**: 130 requests assessed at each pipeline stage

## Results

### Task Planning Accuracy by LLM

| LLM         | Single-Task F1 | Sequential-Task F1 | Graph-Task F1 |
|-------------|----------------|---------------------|---------------|
| GPT-4       | ~98%           | ~92%                | ~85%          |
| GPT-3.5     | ~95%           | ~82%                | ~65%          |
| Vicuna-7B   | Low-Moderate   | Low                 | Very Low      |
| Alpaca-7B   | Low            | Very Low            | Very Low      |

GPT-3.5/GPT-4 show absolute predominance on complex tasks. Open-source models fail
on DAG-structured planning, producing malformed JSON or missing dependencies.

### Stage-wise Performance (GPT-3.5)

| Stage              | Passing Rate | Key Bottleneck             |
|--------------------|-------------|----------------------------|
| Task Planning      | ~80%+       | Complex DAG structures     |
| Model Selection    | ~85%        | Ambiguous task descriptions |
| Task Execution     | Model-dep.  | API latency, model quality |
| Response Generation| ~70-80%     | Error propagation          |

### End-to-End Success Rate

| Request Complexity | GPT-3.5 Success | GPT-4 Success |
|--------------------|----------------|---------------|
| Single-modal       | ~80%           | ~90%          |
| Cross-modal        | ~60%           | ~78%          |
| Complex DAG chain  | ~45%           | ~70%          |

### Error Cascading Analysis

Error probability compounds multiplicatively across stages:
```
P(end-to-end success) = P(plan) * P(select|plan) * P(execute|select) * P(respond|execute)

If each stage = 85%:  0.85^4 = 0.52  (only 52% end-to-end)
If each stage = 90%:  0.90^4 = 0.66  (only 66% end-to-end)
If each stage = 95%:  0.95^4 = 0.81  (still only 81%)
```

Primary error modes ranked by frequency:
1. API/inference failures (~30%): HF Inference API timeouts or errors
2. Task planning errors (~25%): Incorrect decomposition or missing tasks
3. Format mismatches (~20%): Output of one model incompatible with next
4. Model selection errors (~15%): Inadequate model for specific input
5. Response aggregation errors (~10%): LLM fails to coherently combine results

### Comparison with TaskMatrix.AI

| Aspect              | HuggingGPT              | TaskMatrix.AI            |
|---------------------|--------------------------|--------------------------|
| Tool registry       | HF Hub (~thousands)      | Millions of APIs (vision)|
| Implementation      | Fully built, open-source | Conceptual framework     |
| Model selection     | Download-count + LLM     | API spec matching        |
| Evaluation          | 130-request experiments  | No empirical evaluation  |
| Modalities          | Language, vision, audio  | Potentially unlimited    |
| Execution           | HF Inference API / local | Flexible (any endpoint)  |

HuggingGPT is narrower in scope but demonstrates the concept concretely. TaskMatrix.AI
is broader in vision but lacks empirical grounding.

## Analysis & Insights

1. **Controller quality is the primary scaling axis**: GPT-4 vs. GPT-3.5 yields ~20pp
   improvement. Better controllers directly translate to better orchestration.
2. **Model hub as dynamic tool inventory**: Capabilities grow automatically with community
   contributions, but reliability depends on third-party model quality.
3. **Error propagation is the central challenge**: 90% per-step across 4 steps = only 66%
   end-to-end. Motivates error recovery and alternative path planning.
4. **Context window is a hard bottleneck**: Top-K model descriptions per task quickly
   exhaust 4K-8K token windows. Complex pipelines force description truncation.
5. **Few-shot prompt sensitivity**: Planning accuracy shifts 10-15pp depending on examples.
6. **Latency accumulation**: 3-step pipeline at 5s/model = 15s+ total, plus LLM calls.

## Limitations & Critiques

1. **Context window constraint**: Tasks and candidate models bounded by context length.
2. **Error cascading**: Single planning mistake invalidates entire downstream pipeline.
3. **HF API dependency**: Not all models available; rate limits and unpredictable latency.
4. **Download count as quality proxy**: Popular models not always best for specific inputs.
5. **No learning**: No caching of patterns, model choices, or planning improvements.
6. **Security**: Arbitrary community models risk poisoning and adversarial outputs.
7. **Limited taxonomy**: ~30 categories miss 3D generation, RL, molecular simulation, etc.
8. **No interactive refinement**: "Make the dog larger" requires full re-execution.

## Follow-up Work

- **Visual ChatGPT** (Wu et al., 2023): Fixed vision tool set, same pattern but less dynamic.
- **Gorilla** (Patil et al., 2023): Fine-tunes LLMs for API call generation accuracy.
- **ToolLLM** (Qin et al., 2023): 16,000+ API benchmark with fine-tuned tool-use models.
- **TaskWeaver** (Qiao et al., 2023): Code-first planning, converting plans to executable code.
- **AutoGen** (Wu et al., 2023): Multi-agent framework generalizing the controller pattern.
- **OpenAGI** (Ge et al., 2023): Adds RLTF for self-improvement in orchestration.
- **AnyTool** (Du et al., 2024): Hierarchical API retrieval with self-reflection.

## Key Takeaways

1. **LLMs are effective orchestrators**: Even without fine-tuning, LLMs decompose complex
   requests, select tools, and synthesize multi-modal results. The "LLM as brain,
   specialist models as tools" architecture became the dominant agent paradigm.

2. **Community ecosystems are powerful tool registries**: Hugging Face's standardized
   model cards and APIs provide a natural, extensible inventory for LLM agents.

3. **Error cascading is the central unsolved problem**: Per-stage accuracy must be very
   high (>95%) for acceptable end-to-end reliability on multi-step pipelines.

4. **Context length directly limits scalability**: The framework cannot grow beyond
   what the controller LLM can process in a single context window.

5. **The "LLM as controller" pattern** pioneered by HuggingGPT became foundational,
   directly influencing ChatGPT Plugins, Semantic Kernel, LangChain tools, and modern
   function-calling architectures.
