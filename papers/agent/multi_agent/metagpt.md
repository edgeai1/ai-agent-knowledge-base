---
title: "MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework"
authors: "Sirui Hong, Mingchen Zhuge, Jonathan Chen, Xiawu Zheng, Yuheng Cheng, Jinlin Wang, Ceyao Zhang, Zili Wang, Steven Ka Shing Yau, Zijuan Lin, Liyang Zhou, Chenyu Ran, Lingfeng Xiao, Chenglin Wu, Juergen Schmidhuber"
venue: "ICLR 2024 (Oral)"
year: 2024
arxiv: "https://arxiv.org/abs/2308.00352"
code: "https://github.com/geekan/MetaGPT"
tags: [multi-agent, software-engineering, SOP, meta-programming, code-generation, structured-communication]
category: agent/multi_agent
date_read: 2026-05-08
---

# MetaGPT: Meta Programming for A Multi-Agent Collaborative Framework

## TL;DR

MetaGPT encodes real-world Standardized Operating Procedures (SOPs) from software companies into a
multi-agent framework where five specialized agents (Product Manager, Architect, Project Manager,
Engineer, QA) collaborate through structured document artifacts via publish-subscribe communication.
An executable feedback mechanism enables runtime debugging. Achieves 85.9%/87.7% Pass@1 on
HumanEval/MBPP, and 3.75/4.00 executability on SoftwareDev (vs. ChatDev's 2.1 and AutoGPT's 1.0).

## Motivation & Problem

1. **Cascading hallucination**: In unstructured multi-agent dialogue, errors propagate and amplify
   across pipeline stages -- Agent A hallucinates, B builds on it, C implements the hallucination.
2. **Role confusion**: Without clear responsibilities, agents duplicate work or produce misaligned outputs.
3. **No quality gates**: Most frameworks lack intermediate verification before downstream propagation.
4. **Inefficient communication**: Free-form chat wastes tokens with low-information-density text.

Core insight: **Human software teams communicate through structured artifacts (PRDs, design docs,
API specs), not free-form chat. Encoding these SOPs constrains and focuses agent interactions.**

## Method

### SOP Encoding Mechanism

```
User Requirement --> Product Manager --> Architect --> Project Manager --> Engineer --> QA
                      (PRD)             (System       (Task List)        (Code)       (Tests)
                                         Design)
```

Each role is encoded with: profile (name, role, goal, constraints), actions (specific operations),
subscriptions (message types it listens to), and output schema (format requirements).

### Role Definitions and Prompt Templates

**Product Manager**: Receives raw user requirements. Produces structured PRD containing:
Original Requirements, Product Goals (3), User Stories, Competitive Analysis table,
Requirements Analysis, Requirement Pool (P0/P1/P2 priority), UI Design Draft.
Prompt: "Act as a professional product manager. Produce a structured PRD following template exactly."

**Architect**: Receives PRD. Produces system design with implementation approach (tech choices),
file list with descriptions, data structures and interface definitions, program call flow
(Mermaid sequence diagrams), class diagrams, and API specifications.

**Project Manager**: Receives system design. Produces task list with required packages, file
dependencies, full API spec per file, logic analysis per file, and task ordering with dependency graph.

**Engineer**: Receives task list + design + PRD (all upstream artifacts). Generates code
file-by-file in dependency order. Actions include WriteCode, WriteCodeReview, FixBug.
Must conform to API specs from Architect. Prompt enforces PEP8, modularity, type hints.

**QA Engineer**: Receives code + design + PRD. Writes comprehensive tests, runs them,
reports bugs and inconsistencies with design document.

### Communication Protocol: Publish-Subscribe

```
+---------------------------------------------------------------+
|                    SHARED MESSAGE POOL                         |
| Message { sender, role, content, cause_by, sent_to }          |
|                                                               |
| Subscriptions:                                                |
|   ProductManager  <-- UserRequirement                         |
|   Architect       <-- WritePRD output                         |
|   ProjectManager  <-- WriteDesign output                      |
|   Engineer        <-- WriteTasks + WriteDesign + PRD           |
|   QA              <-- WriteCode output                         |
+---------------------------------------------------------------+

Algorithm:
  for each role R in topological_order(SOP_graph):
      inputs <- message_pool.get(R.subscriptions)
      output <- R.act(inputs)              // LLM generates artifact
      validate(output, R.output_schema)    // schema validation
      message_pool.publish(output)         // add to shared pool
```

Key properties: no direct agent-to-agent calls; role-based filtering reduces noise;
document-centric (not conversational); schema validation before publishing; transparent audit trail.

### Executable Feedback Mechanism

```
+--------+    code    +----------+   execute   +---------+
|Engineer|----------->|Code Files|------------>| Runtime |
+--------+            +----------+             +---------+
    ^                                              |
    |         error messages (stderr)              |
    +----------------------------------------------+
    (revision cycle until pass or max_retries)
```

Runs generated code, catches runtime errors, provides actual error traces back to Engineer
with full upstream context (PRD, design, task spec) for debugging. Contributes +4.2% on
HumanEval and +5.4% on MBPP over the no-feedback version.

## Key Innovations

1. **SOP-as-code**: First framework encoding human organizational SOPs into multi-agent pipelines.
2. **Document-centric communication**: Structured artifacts replace dialogue, reducing hallucination.
3. **Publish-subscribe architecture**: Decoupled, extensible communication model.
4. **Executable feedback**: Runtime execution provides ground-truth error signals vs. LLM self-eval.
5. **Meta-programming paradigm**: "Programming" the LLM with organizational knowledge.

## Experimental Setup

- **Models**: GPT-4 (primary), GPT-3.5-Turbo (ablation)
- **Benchmarks**: HumanEval (164 problems), MBPP (427 problems), SoftwareDev (7 complex projects)
- **Baselines**: ChatDev, AutoGPT, LangChain+REPL, AgentVerse, GPT-Engineer, raw GPT-4

## Results

### HumanEval and MBPP (Pass@1 %)

| Method                | HumanEval | MBPP  |
|----------------------|-----------|-------|
| GPT-4 (direct)       | 67.0      | --    |
| AutoGPT              | ~64.0     | --    |
| ChatDev (GPT-3.5)    | 61.8      | 67.3  |
| MetaGPT (GPT-3.5)    | 82.3      | 82.0  |
| MetaGPT (no feedback)| 81.7      | 82.3  |
| **MetaGPT (GPT-4)**  | **85.9**  |**87.7**|

GPT-3.5 with MetaGPT (82.3%) outperforms raw GPT-4 (67.0%) -- framework compensates for model gap.

### SoftwareDev Benchmark

| Metric                | AutoGPT | ChatDev | MetaGPT  |
|------------------------|---------|---------|----------|
| Executability (1-4)    | 1.0     | 2.1     | **3.75** |
| Average Score (1-4)    | 1.0     | 2.1     | **3.9**  |
| Lines of Code (avg)    | --      | ~80     | ~179     |
| Running Time (sec)     | --      | ~600+   | ~503     |

### Cost Analysis

| Metric                     | MetaGPT  | ChatDev  | AgentCoder |
|----------------------------|----------|----------|------------|
| Tokens per HumanEval task  | ~138.2K  | ~183.7K  | ~56.9K     |
| Tokens per MBPP task       | ~206.5K  | ~259.3K  | ~66.3K     |
| Tokens per line of code    | 50% fewer than ChatDev | baseline | -- |

MetaGPT generates 2.24x more code with fewer tokens per line, but per-task cost exceeds leaner
approaches like AgentCoder (56.9K vs 138.2K).

### Ablation: Impact of SOP Components

| Configuration              | Executability (%) | Human Rating |
|----------------------------|:-----------------:|:------------:|
| Full MetaGPT               | 82                | 3.9          |
| Without PRD stage          | 71                | 3.4          |
| Without design doc stage   | 65                | 3.1          |
| Without QA stage           | 76                | 3.6          |
| Without schema validation  | 73                | 3.3          |
| Free-form chat (no SOP)    | 52                | 2.6          |

Design doc removal causes largest drop (-17%), making architectural planning the highest-leverage
intervention. Removing all structure drops below ChatDev levels.

## Analysis & Insights

1. **Structure prevents hallucination cascades**: Structured artifacts act as checkpoints;
   hallucinations become explicit and catchable rather than implicit in conversation.
2. **Document > dialogue**: Quality of intermediate artifacts (PRD, design) directly predicts
   final code quality more than number of conversation turns.
3. **Role specialization reduces cognitive load**: Decomposing into subtasks fits LLM reasoning
   capacity better than one monolithic prompt.
4. **Frameworks compensate for weaker models**: GPT-3.5 + MetaGPT > raw GPT-4 on HumanEval.
5. **Cost-effectiveness of structure**: 25-30% token reduction vs. dialogue approaches despite
   more comprehensive outputs.

## Limitations & Critiques

1. **Waterfall rigidity**: No support for iterative development or backtracking to earlier stages.
2. **GPT-4 dependency**: Performance degrades significantly with weaker base models.
3. **Limited project complexity**: SoftwareDev projects are simple; enterprise scaling unvalidated.
4. **High per-task token cost**: 138K tokens/task is expensive vs. single-agent approaches.
5. **No human-in-the-loop**: No mechanism for human review at intermediate stages.
6. **Fixed role structure**: Five-role pipeline is hard-coded; real projects need variable teams.
7. **Domain specificity**: Framework is specific to software development SOPs.

## Follow-up Work

- **MetaGPT v2** (2024): Added iterative refinement and human-in-the-loop review.
- **ChatDev** (Qian et al., 2023): Concurrent dialogue-based approach; MetaGPT proved more robust.
- **AgentCoder** (2024): Leaner multi-agent coding at 56.9K tokens (vs. 138.2K).
- **SWE-Agent** (2024): Strong single-agent results on SWE-bench questioning multi-agent necessity.

## Key Takeaways

1. **SOPs transfer organizational knowledge** into agent systems effectively.
2. **Structured artifacts are the key insight** -- documents over dialogue for reliability.
3. **Verify with execution**: Ground-truth feedback > LLM self-evaluation (+4-5%).
4. **Design docs are highest-leverage**: Largest ablation impact (-17%) from architecture stage.
5. **Frameworks > raw models**: GPT-3.5 + MetaGPT outperforms raw GPT-4.
6. **Multi-agent overhead must be justified** by quality gains; not always worth the cost.
