---
title: "SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering"
authors: John Yang, Carlos E. Jimenez, Alexander Wettig, Kilian Lieret, Shunyu Yao, Karthik Narasimhan, Ofir Press
venue: NeurIPS 2024
year: 2024
url: https://arxiv.org/abs/2405.15793
code: https://github.com/SWE-agent/SWE-agent
tags: [coding, swe-bench, agent-computer-interface, tool-design, software-engineering]
status: done
---

## TL;DR

Introduces the Agent-Computer Interface (ACI) concept -- custom-designed terminal interfaces that reshape how LLM agents interact with code repositories, achieving 12.47% on SWE-bench (3x prior SOTA) and demonstrating that interface design is as critical as model capability.

## Motivation & Problem

Prior approaches to automated software engineering gave LLM agents raw shell access to code repositories. This created several fundamental problems:
- **Information overload**: `cat` on a large file dumps thousands of lines, exceeding context windows
- **Error-prone editing**: raw `sed`/`echo` commands lead to syntax errors and malformed patches
- **No guardrails**: agents could corrupt files without any feedback, compounding errors
- **Poor navigation**: `find` and `grep` produce unstructured output difficult for LLMs to parse

The core insight is that LLMs are a new class of "end users" with distinct cognitive limitations (finite context window, sensitivity to output formatting, tendency toward cascading errors) and strengths (ability to understand natural language documentation, flexible reasoning). Just as GUIs were designed for human cognition, ACIs should be designed for LLM cognition.

## Method

### The Agent-Computer Interface (ACI)

The ACI is a principled set of custom commands that replace raw shell interactions. The design philosophy follows four principles:

1. **Actions should be simple and composable**: each command does one thing well
2. **Actions should mirror existing conventions**: similar to familiar CLI tools
3. **Feedback should be informative and concise**: structured, bounded output
4. **Error recovery should be built-in**: guardrails prevent irreversible mistakes

### Core ACI Components

```
+--------------------------------------------------+
|                  SWE-agent ACI                    |
+--------------------------------------------------+
|                                                   |
|  +-------------+  +------------+  +------------+  |
|  | File Viewer |  |  Search    |  |   Editor   |  |
|  +-------------+  +------------+  +------------+  |
|  | open <file> |  | find_file  |  | edit n:m   |  |
|  | goto <line> |  | search_file|  | (+ linter) |  |
|  | scroll_up   |  | search_dir |  |            |  |
|  | scroll_down |  |            |  |            |  |
|  +-------------+  +------------+  +------------+  |
|                                                   |
|  +-------------+  +---------------------------+   |
|  |  Context    |  |    Error Recovery          |   |
|  +-------------+  +---------------------------+   |
|  | 100-line    |  | Syntax lint on every edit  |   |
|  | window      |  | Malformed cmd retry loop   |   |
|  | line nums   |  | History compaction         |   |
|  +-------------+  +---------------------------+   |
+--------------------------------------------------+
```

#### File Viewer
- `open <path>`: opens file at line 1, displays a **100-line window** with line numbers
- `goto <line>`: jumps to a specific line number
- `scroll_up` / `scroll_down`: moves the window by the configured number of lines
- Window size is fixed at 100 lines -- small enough to fit in context, large enough for meaningful code blocks
- Line numbers shown in every display, enabling precise edit references

#### Search Tools
- `find_file <name> [dir]`: locates files by name pattern, returns a summary list
- `search_file <pattern> [file]`: searches within the currently open file or a specified file
- `search_dir <pattern> [dir]`: searches across a directory, outputs grouped results with file paths and line numbers
- Results are **summarized** (not raw grep output) to prevent context overflow
- Critical design choice: search returns summary counts and locations, not full content

#### File Editor
- `edit <start_line>:<end_line>\n<replacement_text>\nend_of_edit`: replaces lines start through end with new text
- Every edit triggers an **automatic linter** that checks syntax
- If the linter detects errors, the edit is **rejected** and the agent receives an error message with the specific syntax issue
- This prevents cascading corruption -- a single malformed edit cannot break the file

#### Error Recovery Mechanisms
- **Syntax linting**: every `edit` command runs a language-appropriate linter; edits producing syntax errors are rolled back
- **Malformed command retry**: if the agent generates an unparseable command, it receives an error and is asked to retry; all intermediate error messages except the first are removed from history to conserve context
- **Context window management**: the 100-line viewer prevents context overflow; search results are summarized

### ACI vs. Raw Terminal Access

```
+------------------+-------------------+---------------------+
| Aspect           | Raw Terminal      | SWE-agent ACI       |
+------------------+-------------------+---------------------+
| File viewing     | cat (full dump)   | 100-line window     |
| Searching        | grep (raw output) | Summarized results  |
| Editing          | sed/echo (fragile)| Structured edit+lint|
| Error handling   | None              | Auto-rollback+retry |
| Navigation       | cd + ls           | find_file + open    |
| Output format    | Unstructured      | Formatted + bounded |
+------------------+-------------------+---------------------+
```

### Agent Loop

```
Input: GitHub issue description, repository snapshot
Output: patch file

while budget_remaining:
    observation = format_window(current_file, current_line, window_size=100)
    thought = LLM.generate(system_prompt + history + observation)
    action = parse_action(thought)

    if action is malformed:
        history.append(error_message)
        continue  # retry with error feedback

    result = execute(action)  # runs command in container

    if action.type == "edit":
        lint_result = run_linter(edited_file)
        if lint_result.has_errors:
            rollback(edited_file)
            result = lint_error_message
        else:
            result = show_updated_window()

    history.append((thought, action, result))
    compact_history_if_needed(history)

    if action == "submit":
        break
```

### mini-SWE-agent

A minimal reimplementation of the core ACI concept in approximately 100 lines of Python. Despite its simplicity, mini-SWE-agent demonstrates the power of good interface design by achieving >74% on SWE-bench Verified when paired with strong models. It includes the essential components: file viewer with windowed display, basic edit with linting, and summarized search.

## Key Innovations

1. **ACI as a first-class concept**: formalizing that agent-environment interface design is a research problem distinct from model training
2. **Linting as a guardrail**: preventing error cascading through automatic syntax validation
3. **Bounded observation windows**: the 100-line viewer as a principled solution to context overflow
4. **Summarized search**: transforming verbose tool output into LLM-digestible summaries
5. **Error recovery protocol**: structured retry with history compaction

## Experimental Setup

- **Benchmark**: SWE-bench (2,294 task instances from 12 Python repos) and HumanEvalFix
- **Base models**: GPT-4 Turbo (primary), Claude 3 Opus
- **Ablation subset**: SWE-bench Lite (300 instances) for component analysis
- **Baselines**: RAG-based approach (previous SOTA at 3.8%), raw shell agent (no ACI)
- **Evaluation**: pass@1 -- does the generated patch pass all relevant unit tests?
- **Compute**: each instance runs in an isolated Docker container

## Results

### Main Results

| System                    | SWE-bench (%) | HumanEvalFix (%) |
|---------------------------|---------------|-------------------|
| RAG (prior SOTA)          | 3.80          | --                |
| Shell-only Agent (GPT-4)  | ~1.8          | --                |
| SWE-agent (GPT-4 Turbo)  | **12.47**     | **87.7**          |

### Ablation Study (SWE-bench Lite, 300 instances)

| Configuration                    | Resolve Rate (%) | Delta     |
|----------------------------------|------------------|-----------|
| Full SWE-agent                   | 18.0             | --        |
| No search tools                  | 15.7             | -2.3      |
| Iterative search (not summarized)| 12.0             | -6.0      |
| No linting guardrail             | ~15.0            | -3.0      |
| Raw shell only (no ACI)          | 7.3              | -10.7     |

Key ablation findings:
- ACI provides a **10.7 percentage point improvement** over raw shell
- Iterative search (showing each match one by one) is **worse** than no search at all, because agents exhaust budget/context cycling through results
- Summarized search is critical: the format of results matters more than their availability
- Linting prevents ~3% of instances from entering unrecoverable error states

### Performance by Repository (SWE-bench Lite)

Harder repositories (e.g., matplotlib, sympy) have lower resolve rates due to complex multi-file dependencies. Simpler bug-fix repos show higher success rates.

## Analysis & Insights

- **Interface design >> model size**: a well-designed ACI with GPT-4 dramatically outperforms poorly-interfaced stronger systems
- **Search design is subtle**: naive "give the agent grep" can hurt performance by encouraging exhaustive, budget-consuming searches
- **Linting as training signal**: error messages from the linter provide implicit feedback that guides the agent toward correct edits
- **Cost-performance tradeoff**: most budget is spent on exploration (search and navigation), not on the final edit
- **Common failure modes**: (1) agent cannot locate the relevant code, (2) agent finds the code but generates an incorrect fix, (3) agent runs out of budget during exploration

## Limitations & Critiques

- Results evaluated only on Python repositories; generalization to other languages untested at time of publication
- The 12.47% resolve rate, while 3x SOTA, still means 87.5% of issues remain unsolved
- ACI design was hand-crafted through iteration; no formal search over the design space
- Evaluation on SWE-bench may overestimate real-world performance due to benchmark-specific patterns
- Single-turn patch generation; no iterative debugging with test feedback in the original system
- On truly novel issues (not in curated benchmarks), agent performance drops to ~18-20%

## Follow-up Work

- **SWE-agent 2.0**: extended with more sophisticated search and multi-file editing
- **mini-SWE-agent**: 100-line distillation achieving >74% on SWE-bench Verified with frontier models
- **OpenHands/CodeAct**: adopted ACI principles with code-based action spaces
- **Agentless**: showed that simpler localize-then-patch pipelines can be competitive
- **SWE-bench Verified**: curated 500-instance subset for more reliable evaluation
- Leaderboard progression (as of early 2026): top systems exceed 80% on SWE-bench Verified

## Key Takeaways

1. The interface between an agent and its environment deserves as much research attention as the agent's underlying model
2. Less is more: bounded, summarized outputs outperform raw, verbose tool responses
3. Guardrails (linting) are not just safety features -- they are performance features that prevent error cascading
4. Search tool design requires careful thought; naive implementations can actively hurt performance
5. The ACI concept generalizes beyond coding to any domain where agents interact with complex software environments
