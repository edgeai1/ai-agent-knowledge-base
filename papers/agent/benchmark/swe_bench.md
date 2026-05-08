---
title: "SWE-bench: Can Language Models Resolve Real-World GitHub Issues?"
authors: Carlos E. Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, Karthik Narasimhan
venue: ICLR 2024
year: 2024
url: https://arxiv.org/abs/2310.06770
code: https://github.com/princeton-nlp/SWE-bench
tags: [benchmark, software-engineering, github, code-generation, evaluation]
status: done
---

## TL;DR

Introduces SWE-bench, a benchmark of 2,294 real-world software engineering tasks drawn from GitHub issues and pull requests across 12 popular Python repositories, establishing the first large-scale evaluation of LLMs on realistic, repository-level code modification tasks.

## Motivation & Problem

Prior code generation benchmarks had fundamental limitations:
- **HumanEval / MBPP**: self-contained function-level problems solvable in a few lines; no repository context
- **CodeContests**: competitive programming; algorithmic but not representative of real software engineering
- **DS-1000**: data science snippets; narrow domain

Real-world software engineering requires:
1. Understanding large codebases (thousands of files, complex dependencies)
2. Navigating across modules, classes, and files to locate relevant code
3. Reasoning about bug reports written in natural language with varying quality
4. Generating patches that modify existing code correctly without breaking other functionality
5. Handling diverse tasks: bug fixes, feature requests, refactoring, documentation

No existing benchmark captured this complexity. SWE-bench fills this gap by sourcing tasks from actual GitHub development activity.

## Method

### Dataset Construction Pipeline

```
GitHub Repository (e.g., django/django)
        |
        v
Collect merged PRs with associated issues
        |
        v
Filter: PR must modify source code AND test files
        |
        v
Extract: issue description, base commit, gold patch, test patch
        |
        v
Validate: gold patch must make failing tests pass
        |
        v
SWE-bench Task Instance
```

Step-by-step:

1. **Repository selection**: 12 popular, actively-maintained Python repositories chosen for diversity (web frameworks, scientific computing, ML libraries, utilities)
2. **PR collection**: all merged pull requests with linked issues are collected
3. **Filtering criteria**:
   - PR must modify at least one source file
   - PR must add or modify at least one test file
   - The test changes must include tests that FAIL on the pre-PR codebase and PASS after applying the PR
4. **Task instance creation**: for each qualifying PR, extract the issue description as the natural language specification and the code diff as the gold solution
5. **Validation**: verify that (a) tests fail before the patch, (b) tests pass after the patch, (c) existing tests still pass after the patch

### Task Instance Format

```
+--------------------------------------------------+
| SWE-bench Task Instance                          |
+--------------------------------------------------+
| repo:        django/django                       |
| instance_id: django__django-16379                |
| base_commit: a1b2c3d4...                         |
| issue_text:  "FileBasedCache has_key is          |
|              broken when cache key..."            |
| hints_text:  (optional additional context)        |
| patch:       (gold solution diff -- hidden)       |
| test_patch:  (tests that validate the fix)        |
| FAIL_TO_PASS: [test_cache.CacheTest.test_has_key]|
| PASS_TO_PASS: [test_cache.CacheTest.test_get...] |
| environment: Python 3.9, Django deps...           |
+--------------------------------------------------+
```

Key fields:
- **FAIL_TO_PASS**: tests that should fail before the fix and pass after -- the primary evaluation signal
- **PASS_TO_PASS**: existing tests that must continue passing -- ensures the fix does not introduce regressions
- **patch**: the gold-standard code change (hidden from the model at test time)
- **test_patch**: test code added/modified in the PR (used for evaluation only)

### Evaluation Methodology

```
Given: issue_text, repository at base_commit
Model generates: predicted_patch

Evaluation:
1. Apply predicted_patch to repository at base_commit
2. Apply test_patch (gold test changes)
3. Run FAIL_TO_PASS tests:
   - All must PASS  -->  candidate solution
4. Run PASS_TO_PASS tests:
   - All must PASS  -->  no regressions
5. If both pass: instance RESOLVED
```

The metric is **resolve rate**: percentage of instances where the model's patch causes all FAIL_TO_PASS tests to pass while maintaining all PASS_TO_PASS tests.

### Repository Distribution

| Repository        | Domain                | # Instances | % of Total |
|-------------------|-----------------------|-------------|------------|
| django            | Web framework         | 440         | 19.2%      |
| scikit-learn      | Machine learning      | 305         | 13.3%      |
| sympy             | Symbolic math         | 298         | 13.0%      |
| matplotlib        | Visualization         | 268         | 11.7%      |
| requests          | HTTP library          | 67          | 2.9%       |
| flask             | Web microframework    | 14          | 0.6%       |
| sphinx            | Documentation         | 187         | 8.2%       |
| astropy           | Astronomy             | 140         | 6.1%       |
| pytest            | Testing framework     | 118         | 5.1%       |
| pylint            | Linting               | 74          | 3.2%       |
| xarray            | Array computing       | 95          | 4.1%       |
| seaborn           | Statistical viz       | 48          | 2.1%       |
| **Total**         |                       | **2,294**   | **100%**   |

Note: Django and scikit-learn dominate the dataset, which creates a distributional bias.

## SWE-bench Variants

### SWE-bench Full
- 2,294 task instances
- Includes noise: some issues are ambiguously described, some test patches may be overly specific
- The original, unfiltered benchmark

### SWE-bench Lite
- **300 instances** selected for more tractable evaluation
- Criteria: excludes overly large patches, ambiguous descriptions, and instances requiring external knowledge
- Positioned as "easy-to-medium" difficulty
- Became the primary evaluation target for initial agent systems
- Activity has slowed since late 2024 as attention shifted to Verified

### SWE-bench Verified
- **500 instances** curated with human annotators (in collaboration with OpenAI)
- Each instance verified for: (1) clear problem description, (2) correct test patch, (3) solvability given available information
- Higher-quality subset designed to reduce noise and false negatives
- Released mid-2024; became the primary leaderboard by late 2024
- As of early 2025, ~99 entries on the Verified leaderboard vs ~79 on Lite

## Historical Performance Progression

| Date       | System                      | SWE-bench Full | Verified |
|------------|-----------------------------|--------------:|--------:|
| Oct 2023   | Claude 2 (RAG baseline)     | 1.96%          | --      |
| Oct 2023   | GPT-4 (RAG baseline)        | 3.80%          | --      |
| May 2024   | SWE-agent (GPT-4 Turbo)     | 12.47%         | --      |
| Jun 2024   | Devin                       | 13.86%         | --      |
| Aug 2024   | OpenHands + CodeAct          | --             | ~33%    |
| Oct 2024   | Various agentic systems      | --             | ~45-50% |
| Dec 2024   | Frontier models + agents     | --             | ~60%    |
| Mar 2025   | Claude Opus 4 systems        | --             | ~72%    |
| Early 2026 | Claude Opus 4.5 + scaffolds  | --             | ~80.9%  |
| 2026       | Claude Mythos Preview        | --             | ~93.9%  |

The benchmark has seen extraordinary performance growth from ~4% to 90%+ in under three years.

## Key Innovations

1. **Repository-level evaluation**: first benchmark requiring understanding of full codebases, not isolated functions
2. **Automated pipeline**: scalable construction from GitHub PRs enables continuous dataset expansion
3. **Execution-based evaluation**: unit tests provide unambiguous pass/fail signals, avoiding subjective quality judgments
4. **Real-world distribution**: tasks reflect actual developer work, not synthetic problems
5. **Version-controlled environments**: each instance specifies an exact commit, ensuring reproducible evaluation

## Analysis & Insights

- **Difficulty variance**: instances range from one-line typo fixes to complex multi-file refactoring; the distribution is heavily right-skewed (many hard instances)
- **Localization is half the battle**: many agent failures stem from inability to find the relevant code, not from inability to write the fix
- **Test quality matters**: some test patches are overly specific (testing implementation details rather than behavior), creating false negatives
- **Repository bias**: heavy representation of Django and scikit-learn may skew results toward agents that perform well on web/ML code patterns
- **Temporal leakage risk**: since data comes from public GitHub, there is a risk that models trained on post-PR code have seen solutions

## Limitations & Critiques

- **Python-only**: limited to Python repositories; real software engineering spans many languages
- **Test-dependent evaluation**: if the gold test patch is flawed (tests wrong behavior), correct solutions may be marked as failures
- **SWE-bench+**: follow-up work (arXiv:2410.06992) identified that some test patches in the original dataset are insufficient -- models can pass by making unrelated changes that happen to satisfy the tests
- **Benchmark saturation**: with top systems exceeding 90% on Verified, the benchmark's discriminative power is decreasing
- **Distribution shift**: curating Lite and Verified subsets may remove the hardest, most realistic instances
- **No partial credit**: a patch that fixes the bug but has a minor style issue fails; no gradient in evaluation
- **Single-attempt evaluation**: pass@1 does not capture models that can self-correct with feedback

## Follow-up Work

- **SWE-bench Lite**: 300-instance subset for faster iteration
- **SWE-bench Verified**: 500 human-validated instances for higher-quality evaluation
- **SWE-bench+**: identified and fixed flawed test patches in the original dataset
- **SWE-bench Pro**: extended to longer-horizon tasks
- **Multi-SWE-bench**: multilingual extension beyond Python
- **SWE-MERA**: dynamic benchmark that continuously generates new tasks to prevent contamination
- **LiveCodeBench**: complementary benchmark with temporal filtering to avoid data contamination

## Key Takeaways

1. Real-world software engineering remains fundamentally harder than function-level code generation
2. The benchmark's construction pipeline (GitHub PR to task instance) is a template for building scalable, realistic benchmarks
3. Execution-based evaluation via unit tests provides reliable, automated assessment
4. The rapid performance progression (4% to 90%+ in 3 years) may reflect benchmark-specific optimization as much as genuine capability gains
5. Repository-level understanding -- navigation, localization, cross-file reasoning -- is the key bottleneck that distinguishes SWE-bench from simpler coding benchmarks
