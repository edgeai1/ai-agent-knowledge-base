---
title: "Devin: The First AI Software Engineer (and OpenHands Comparison)"
authors: Cognition AI (Scott Wu, Steven Hao, Walden Yan)
venue: Product launch (March 2024), Devin 2.0 (2025)
year: 2024
url: https://devin.ai
blog: https://cognition.ai/blog/introducing-devin
technical_report: https://cognition.ai/blog/swe-bench-technical-report
tags: [coding-agent, autonomous-software-engineering, commercial, SWE-bench, product]
status: done
---

## TL;DR

Devin is the first commercially launched autonomous AI software engineering agent, developed by Cognition AI and announced March 2024. It operates as a multi-agent system with five specialized agents (planner, shell, editor, browser, verifier) within a sandboxed compute environment, executing observe-plan-act loops across multi-hour sessions. Devin's initial SWE-bench score of 13.86% (7x the prior SOTA of 1.96%) defined the product category, though subsequent competitors -- particularly OpenHands -- have far surpassed this on benchmarks. Devin 2.0 improved to 45.8% on SWE-bench Verified, while the broader field advanced to 70%+ by early 2026.

## Motivation & Problem

Before Devin, AI coding tools were primarily assistive -- code completion (Copilot), chat-based code generation (ChatGPT), or IDE-integrated suggestion engines. None attempted autonomous, end-to-end software engineering: taking a natural language description of a task and independently planning, coding, debugging, testing, and deploying a solution.

The fundamental problems Devin aimed to solve:

1. **Single-step vs. multi-step**: Existing tools generated code snippets but could not execute multi-step engineering workflows (read codebase -> understand context -> plan approach -> write code -> run tests -> debug failures -> iterate)
2. **No environment interaction**: Code generation tools operated in a text-in/text-out paradigm without access to a real development environment (shell, file system, browser for documentation)
3. **No error recovery**: When generated code failed tests, existing tools could not autonomously diagnose and fix the issue through iterative debugging
4. **No context gathering**: Real software engineering requires reading documentation, exploring codebases, and searching for relevant information -- capabilities no existing tool provided autonomously

## Method: Architecture

### Five-Agent Multi-Agent Architecture

Devin operates as a multi-agent orchestration framework where specialized agents handle different aspects of software engineering. The architecture decomposes software engineering into five functional roles:

**1. Planning Agent (the "Architectural Brain")**
- Receives a high-level task description (GitHub issue, feature request, bug report)
- Decomposes the task into a step-by-step execution plan
- Maps out the development path before any code is written
- Generates and maintains an evolving plan that adapts as execution proceeds
- In Devin 2.0: produces an interactive preliminary plan that users can review and modify before autonomous execution begins

**2. Shell Agent**
- Executes bash commands within the sandboxed Ubuntu environment
- Handles: dependency installation, build processes, running test suites, git operations, system configuration
- Processes stdout/stderr output and feeds results back to the planning loop
- Can execute complex multi-command workflows (piped commands, scripts, background processes)

**3. Editor Agent**
- Reads, writes, and modifies code files in the workspace
- Handles both new file creation and editing existing code
- Understands file structure and can navigate large codebases
- Applies targeted edits rather than rewriting entire files

**4. Browser Agent**
- Searches documentation and code examples when the agent needs external information
- Navigates official documentation sites, Stack Overflow, GitHub repositories
- Extracts relevant information and brings it back to the coding context
- Activated when the agent encounters unfamiliar frameworks, APIs, or error messages

**5. Verifier Agent**
- Runs test suites and validates that changes work correctly
- Analyzes test failures to diagnose issues
- Triggers re-planning when tests fail, closing the feedback loop
- Determines when the task is complete (stopping condition)

### Operational Loop: Observe-Plan-Act

Devin operates in agentic loops across potentially multi-hour sessions:

```
while not task_complete:
    1. OBSERVE: gather context (read code, check test results, examine errors)
    2. PLAN: update the execution plan based on current state
    3. ACT: execute the next step (edit code, run command, search docs)
    4. VERIFY: check if the action succeeded (run tests, examine output)
    5. ADAPT: if verification fails, diagnose the issue and re-plan
```

Each loop iteration involves thousands of decisions. Devin can recall relevant context at every step, learn from within-session experience, and fix its own mistakes through iterative debugging.

### Sandboxed Compute Environment

- Devin operates within a sandboxed Docker container providing a full Linux environment
- Equipped with: bash shell, code editor, web browser, file system access
- Isolated from the host system for security
- Persistent workspace across the entire session
- Git integration for version control operations

### Underlying Models: SWE Model Family

**SWE-1 (2024)**: Cognition's first internally developed coding model. Designed specifically for the multi-step reasoning and tool use required by autonomous software engineering agents. Powers both Devin and the Windsurf IDE.

**SWE-1.5 (October 2025)**: Frontier-scale model with hundreds of billions of parameters. Trained on a cluster of thousands of NVIDIA GB200 NVL72 chips. Partnership with Cerebras provides wafer-scale inference hardware for high token throughput, enabling the speed necessary for real-time autonomous coding. Scored 40.08% on SWE-Bench Pro.

## Key Innovations

1. **Product category creation**: Devin defined "AI software engineer" as a product category, distinct from code completion tools or chat assistants
2. **Multi-agent specialization**: Instead of a single monolithic model, Devin uses specialized agents for different engineering tasks, enabling more focused and effective behavior per subtask
3. **Interactive planning (Devin 2.0)**: Users review and modify the plan before autonomous execution, combining human judgment with AI execution
4. **Multi-hour autonomous sessions**: Can work on complex tasks for hours without human intervention, managing state and context across extended sessions
5. **In-session learning**: Improves within a session by learning from failures and adapting approach

## SWE-bench Evolution: 13.86% -> 45.8%

### The SWE-bench Trajectory

| Date         | System          | SWE-bench Score  | Notes                                        |
|--------------|-----------------|------------------|----------------------------------------------|
| Pre-Devin    | Best baseline   | 1.96%            | Prior unassisted SOTA                        |
| Mar 2024     | Devin 1.0       | **13.86%**       | 7x improvement; defined the category         |
| Mar 2024     | Devin (w/ test) | 23.0%            | With access to final unit test               |
| Late 2024    | Devin 2.0       | **45.8%**        | SWE-bench Verified; major architecture update|
| Oct 2025     | SWE-1.5         | 40.08%           | SWE-Bench Pro (harder benchmark)             |

### Industry Context: The SWE-bench Race

Devin's 13.86% sparked an intense competition. The benchmark leaderboard evolved rapidly:

| Date          | Leading System                    | SWE-bench Verified |
|---------------|-----------------------------------|--------------------|
| Mar 2024      | Devin                             | 13.86%             |
| Mid 2024      | SWE-Agent, AutoCodeRover          | ~20-25%            |
| Late 2024     | OpenHands CodeAct 2.1             | 53%                |
| Early 2025    | Claude 3.5 Sonnet + scaffolding   | ~49%               |
| Mid 2025      | Claude 3.7 Sonnet systems         | ~55-65%            |
| Early 2026    | Claude 4 + OpenHands              | **72%**            |
| Early 2026    | Leading systems                   | **75%+**           |

Devin's initial 13.86% was rapidly surpassed, but it catalyzed the entire field.

## Comparison with OpenHands

### Philosophical Differences

| Dimension              | Devin                                    | OpenHands                               |
|------------------------|------------------------------------------|-----------------------------------------|
| **Model**              | Proprietary SWE-1/1.5 models             | Model-agnostic (Claude, GPT, etc.)      |
| **Access**             | Commercial SaaS ($20/mo + per-ACU)       | Open-source (MIT license)               |
| **Architecture**       | Multi-agent orchestration                | Event-stream + CodeAct agent            |
| **Planning**           | Interactive human-in-the-loop            | Autonomous with optional human feedback |
| **SWE-bench best**     | 45.8% (Verified)                         | 72% (Verified, with Claude 4)           |
| **Extensibility**      | Closed platform                          | Open plugin/agent architecture          |
| **Community**          | Commercial user base                     | 68K+ GitHub stars, 188+ contributors    |

### Benchmark vs. Real-World Performance Gap

A critical insight from comparing Devin and OpenHands is the gap between benchmark and real-world performance:

- **SWE-bench** tests isolated bug fixes on well-defined open-source issues with clear test suites
- **Real-world tasks** involve ambiguous requirements, complex codebases, CI/CD pipelines, deployment, and communication
- **SWE-EVO benchmark** (2025) revealed this gap starkly: agents achieving 65% on SWE-bench Verified score only ~21% on SWE-EVO, which requires multi-file changes across an average of 21 files per task
- Devin has invested more in real-world usability (interactive planning, Slack integration, team workflows), while OpenHands excels on benchmarks due to its flexibility in underlying model selection

### SWE-bench Live: The Static vs. Dynamic Gap

SWE-bench Live (2025) tests agents on new, unseen issues in real-time, preventing contamination:
- OpenHands with Claude 3.7 Sonnet: ~43% on SWE-bench Verified but only ~19.25% on SWE-bench Live
- This ~24 percentage point drop highlights concerns about data contamination and overfitting to the static benchmark

## Experimental Setup & Benchmarks

### Devin's Original SWE-bench Evaluation (March 2024)

- **Benchmark**: SWE-bench (full set) -- 2,294 real-world GitHub issues from 12 Python repositories including Django, scikit-learn, Flask, and others
- **Task**: Given an issue description and the repository, autonomously produce a patch that resolves the issue
- **Evaluation**: The patch must pass the issue's associated test suite
- **Result**: 13.86% of issues resolved (318 of 2,294) compared to prior best of 1.96%
- **With test access**: When also given the final unit test alongside the problem statement, success increases to 23% (100 sampled)

### Devin 2.0 Pricing Model

- Core plan: $20/month base + $2.25 per Agent Compute Unit (ACU)
- Team plan: $20/month base + $2.00 per ACU
- One hour of Devin's time costs approximately $8-9 in ACUs
- Enterprise plans available with custom pricing

## Limitations

- **Closed source**: No ability to inspect, modify, or understand the underlying system; makes scientific evaluation and reproducibility impossible
- **Benchmark-reality gap**: Strong benchmark numbers may not reflect real-world engineering capability; SWE-bench tasks are well-defined, whereas real tasks are ambiguous
- **Cost**: At $8-9 per hour of compute, Devin is expensive for extended autonomous sessions; costs accumulate rapidly for complex tasks
- **Latency**: Multi-hour sessions with thousands of model calls are slow compared to human developers for many tasks
- **Vendor lock-in**: Proprietary platform with no portability; users cannot switch to alternative systems without losing workflow integration
- **Limited transparency**: No published technical paper with reproducible methodology; claims rely on blog posts and marketing materials
- **Scaling limitations**: Performance degrades on very large codebases or tasks requiring deep domain expertise
- **Error compounding**: In long autonomous sessions, small errors early in the process can cascade into larger failures that are difficult to recover from

## Key Takeaways

1. **Devin defined the product category** of "AI software engineer" -- a fully autonomous system with its own development environment, distinct from code completion or chat-based assistants. This framing catalyzed over $175M in Cognition AI funding and inspired numerous competitors.
2. **The multi-agent architecture with specialized roles** (planner, shell, editor, browser, verifier) has proven effective and influenced subsequent systems. The observe-plan-act loop with verification is now the standard pattern for coding agents.
3. **The SWE-bench trajectory from 1.96% to 75%+** over two years demonstrates explosive capability improvement in autonomous coding, though the real-world impact lags behind benchmark numbers.
4. **Interactive planning (Devin 2.0)** represents an important design evolution: rather than fully autonomous execution, combining human judgment on the plan with AI execution on the implementation yields better practical results.
5. **The benchmark vs. real-world gap is significant**: SWE-EVO and SWE-bench Live show that static benchmark performance substantially overstates real-world capability, with drops of 24-44 percentage points in more realistic settings.
6. **Open-source competitors (OpenHands) have surpassed Devin on benchmarks** (72% vs. 45.8% on SWE-bench Verified), demonstrating that open, model-agnostic platforms with access to frontier models can outperform proprietary, vertically integrated solutions.
7. **Cognition AI's strategic pivot** with SWE-1.5 and Devin 2.0 emphasizes real-world usability, interactive workflows, and team integration over raw benchmark scores -- a recognition that benchmark performance alone does not drive adoption.
