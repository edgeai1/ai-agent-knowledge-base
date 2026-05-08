---
title: "AFlow: Automating Agentic Workflow Generation"
authors:
  - Jiayi Zhang
  - Jinyu Xiang
  - Zhaoyang Yu
  - Fengwei Teng
  - Xiong-Hui Chen
  - Jiaqi Chen
  - Mingchen Zhuge
  - Xin Cheng
  - Sirui Hong
  - Jinlin Wang
  - Bingnan Zheng
  - Bang Liu
  - Yuyu Luo
  - Chenglin Wu
venue: ICLR 2025 (Oral, top 1.8%)
year: 2025
url: https://arxiv.org/abs/2410.10762
tags:
  - agentic-workflow
  - workflow-optimization
  - monte-carlo-tree-search
  - code-generation
  - automated-workflow
  - LLM-agents
status: done
---

## TL;DR

AFlow reformulates agentic workflow optimization as a search problem over code-represented
workflows. It uses Monte Carlo Tree Search (MCTS) to iteratively explore, refine, and discover
effective workflows through code modification, tree-structured experience accumulation, and
execution feedback. AFlow-optimized workflows outperform manually designed methods by an
average of 5.7% and contemporary automatic methods by 19.5%, while enabling weaker models
to outperform GPT-4o at 4.55% of its inference cost.

## Motivation & Problem

Manually designing agentic workflows (chains of LLM calls with specific prompts, tools, and
control flow) is labor-intensive and requires deep expertise. Existing approaches fall into two
camps:

1. **Manual design**: Frameworks like Chain-of-Thought, Tree-of-Thought, and Self-Consistency
   require human experts to hand-craft workflows for each task domain.
2. **Automated optimization**: Prior methods (DSPy, ADAS, etc.) either optimize only prompts
   within fixed structures or search over natural-language-described workflows, limiting
   expressiveness and reproducibility.

The key insight is that workflows can be represented as executable Python code, where
LLM-invoking nodes are connected by edges (data flow), enabling the full power of programming
constructs (loops, conditionals, composition) while remaining machine-searchable.

## Method

### Core Formulation

AFlow represents each workflow as a Python class (a `Workflow`) containing:
- **Nodes**: Individual LLM invocations with specific prompts and configurations
- **Edges**: Data flow connections between nodes
- **Operators**: Predefined reusable node combinations (Generate, Format, Review, Revise,
  Ensemble, Test, Programmer)

### MCTS-Based Search

```
MCTS_AFLOW(initial_workflow, dataset, iterations):
    tree = initialize_tree(initial_workflow)

    for i in 1..iterations:
        # 1. SELECTION: soft mixed-probability node selection
        #    Balances exploitation (best-performing) and exploration (least-visited)
        node = SELECT(tree, UCB_score)

        # 2. EXPANSION: LLM-driven code modification
        #    An optimizer LLM proposes modifications to the workflow code
        new_workflow = LLM_MODIFY(node.workflow, tree.experience)

        # 3. EVALUATION: Execute workflow on validation samples
        score = EXECUTE(new_workflow, dataset.sample())

        # 4. BACKPROPAGATION: Update experience and scores
        BACKPROPAGATE(tree, node, score, new_workflow)

    return best_workflow(tree)
```

### Architecture Diagram

```
+-------------------+
|  Optimizer LLM    |  (e.g., GPT-4o proposes code changes)
+--------+----------+
         |
         v
+--------+----------+     +------------------+
|  MCTS Controller  |<--->| Tree Experience  |
|  - Selection      |     | - Past workflows |
|  - Expansion      |     | - Scores         |
|  - Backpropagation|     | - Code diffs     |
+--------+----------+     +------------------+
         |
         v
+--------+----------+
|  Workflow (Code)  |  Python class with Nodes + Edges
|  - Node_1(LLM)    |
|  - Node_2(LLM)    |
|  - Operators       |
+--------+----------+
         |
         v
+--------+----------+
|  Execution Engine |  Runs workflow on validation data
|  - Score feedback  |
+-------------------+
```

### Operators

Operators are predefined, reusable building blocks encapsulating common agentic patterns:

| Operator     | Description                                          |
|-------------|------------------------------------------------------|
| Generate    | Single LLM call with a prompt                       |
| Format      | Parse and structure LLM output                       |
| Review      | LLM evaluates quality of a previous generation       |
| Revise      | LLM improves output based on review feedback         |
| Ensemble    | Multiple LLM calls aggregated (voting/merging)       |
| Test        | Execute code and return test results                 |
| Programmer  | Generate and iteratively debug code                  |

### Key Mechanism: Tree-Structured Experience

Each MCTS node stores the complete workflow code and its execution history. When expanding,
the optimizer LLM receives:
- Current workflow code
- Execution scores from sibling and ancestor nodes
- Prior successful/failed modifications
This accumulated context guides the search toward productive regions of the workflow space.

## Key Innovations

1. **Code-as-workflow representation**: Unlike natural language descriptions, code-represented
   workflows are unambiguous, executable, and composable. They support loops, conditionals,
   and arbitrary Python logic.

2. **MCTS for workflow search**: First application of MCTS to the workflow optimization
   problem. The tree structure naturally handles the exploration-exploitation tradeoff and
   accumulates structured experience.

3. **Operator abstraction**: Predefined operators bootstrap the search, encoding human
   knowledge about effective agentic patterns without constraining the search space.

4. **Cross-model optimization**: The optimizer LLM (e.g., GPT-4o) discovers workflows
   that are executed by a potentially different, cheaper model (e.g., GPT-4o-mini),
   enabling cost-performance Pareto improvements.

## Experimental Setup

- **Benchmarks**: 6 datasets across 3 domains
  - Code: HumanEval (pass@1), MBPP (pass@1)
  - Math: GSM8K (solve rate %), MATH (solve rate %)
  - QA: HotpotQA (F1), DROP (F1)
- **Baselines**: Manual designs (IO, CoT, CoT-SC, LtM, ToT) and automated methods
  (DSPy, ADAS, and others)
- **Models**: GPT-4o-mini as executor, GPT-4o as optimizer
- **Search budget**: Multiple MCTS iterations per benchmark

## Results

### Aggregate Performance

| Method Category           | Avg. Performance | Comparison         |
|--------------------------|------------------|--------------------|
| AFlow (GPT-4o-mini)      | 80.3%            | --                 |
| Best manual design       | ~74.6%           | AFlow +5.7%        |
| Best automated method    | ~60.8%           | AFlow +19.5%       |

### Key Findings

- AFlow with GPT-4o-mini outperforms GPT-4o on multiple tasks at **4.55%** of
  GPT-4o's inference cost, demonstrating strong Pareto efficiency.
- On GSM8K, even without operators, AFlow achieves **93.1%**, surpassing all manual
  designs. With operators, performance improves further.
- AFlow autonomously discovers ensemble-like structures without being explicitly
  programmed to do so, validating the expressiveness of the code search space.
- Workflows are transferable: workflows discovered for one model can improve
  performance when applied to other models.

### Ablation Study (GSM8K)

| Configuration              | Solve Rate |
|---------------------------|------------|
| AFlow (full, with ops)    | Best       |
| AFlow (no operators)      | 93.1%      |
| Manual CoT-SC             | < 93.1%    |
| Manual CoT                | < 93.1%    |

## Limitations

1. **Search cost**: MCTS exploration requires many LLM calls for the optimizer, making
   the workflow discovery phase expensive (though the discovered workflow is cheap to run).
2. **Optimizer dependency**: Quality of discovered workflows depends on the capability of
   the optimizer LLM (GPT-4o). Weaker optimizers may yield inferior workflows.
3. **Benchmark-specific tuning**: Workflows are optimized per-benchmark; generalization
   across diverse task distributions is not fully explored.
4. **Operator design**: The predefined operator set encodes human assumptions about useful
   patterns, which may bias the search or miss novel compositions.
5. **Reproducibility of search**: MCTS with LLM-driven expansion introduces stochasticity;
   different runs may discover different workflows.

## Key Takeaways

1. Representing workflows as code unlocks a vastly richer search space compared to
   natural-language or fixed-topology approaches, supporting arbitrary control flow.
2. MCTS is an effective algorithm for workflow optimization because it balances
   exploration of novel structures with exploitation of known-good patterns.
3. The operator abstraction is a practical way to inject human knowledge while
   preserving automation -- operators bootstrap search without constraining it.
4. Cost-performance Pareto improvements are achievable: AFlow finds workflows where
   cheap models (GPT-4o-mini) outperform expensive models (GPT-4o), democratizing
   access to high-quality agentic workflows.
5. The framework is domain-agnostic, working across code generation, mathematical
   reasoning, and question answering without architecture changes.
