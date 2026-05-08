---
title: "OSWorld: Benchmarking Multimodal Agents for Open-Ended Tasks in Real Computer Environments"
authors: "Tianbao Xie, Danyang Zhang, Jixuan Chen, Xiaochuan Li, Siheng Zhao, Ruisheng Cao, Toh Jing Hua, Zhoujun Cheng, Dongchan Shin, Fangyu Lei, Yitao Liu, Yiheng Xu, Shuyan Zhou, Silvio Savarese, Caiming Xiong, Victor Zhong, Tao Yu"
venue: "NeurIPS 2024 (Datasets and Benchmarks Track)"
year: 2024
url: "https://arxiv.org/abs/2404.07972"
code: "https://github.com/xlang-ai/OSWorld"
tags: [benchmark, desktop-agent, GUI-agent, multimodal, computer-use, operating-system, evaluation]
category: agent/benchmark
status: done
date_read: 2026-05-08
---

# OSWorld: Benchmarking Multimodal Agents for Open-Ended Tasks in Real Computer Environments

## TL;DR

OSWorld is the first scalable benchmark for evaluating multimodal agents in real computer
environments (Ubuntu, Windows, macOS). It provides 369 diverse computer tasks spanning desktop
applications, web browsing, file operations, and multi-app workflows, each with automated
execution-based evaluation via custom scripts. The initial evaluation reveals a dramatic capability
gap: human experts achieve 72.36% success rate while the best agent (GPT-4V) achieves only 12.24%,
primarily struggling with GUI grounding and operational knowledge. OSWorld established the definitive
benchmark for desktop/computer-use agent evaluation and has driven rapid progress in the field.

## Motivation & Problem

Prior benchmarks for evaluating computer-use agents suffered from critical limitations:

1. **Simulated environments**: Most benchmarks used simplified simulations or mock interfaces rather
   than real operating systems, failing to capture the complexity of actual desktop interactions
   (window management, multi-app coordination, system dialogs, etc.).
2. **Domain-specific scope**: Existing benchmarks focused on narrow domains -- web browsing only
   (WebArena), mobile only (AndroidWorld), or specific applications -- missing the full breadth
   of real computer use.
3. **Lack of cross-platform evaluation**: No benchmark tested agents across multiple operating
   systems (Ubuntu, Windows, macOS) to assess generalization.
4. **Reproducibility challenges**: Without standardized initial states and automated evaluation,
   results were difficult to reproduce and compare fairly across approaches.
5. **No multi-application workflows**: Real computer use frequently involves coordinating across
   multiple applications (download file from browser, edit in spreadsheet, email results), but
   existing benchmarks tested single-application tasks in isolation.

OSWorld's contribution: A unified, scalable, real-computer environment supporting task setup,
execution-based evaluation, and interactive learning across real operating systems.

## Method

### Environment Architecture

```
+-------------------------------------------------------------------+
|                      OSWorld Platform                              |
|                                                                   |
|  +-----------------------------+  +-----------------------------+ |
|  |    Virtual Machine Pool     |  |    Task Configuration       | |
|  |  +-------+ +-------+ +---+ |  |  - Initial state snapshots  | |
|  |  |Ubuntu | |Windows| |Mac| |  |  - Task instructions (NL)   | |
|  |  |  VM   | |  VM   | | VM| |  |  - Evaluation scripts       | |
|  |  +-------+ +-------+ +---+ |  |  - Setup automation          | |
|  +-----------------------------+  +-----------------------------+ |
|                                                                   |
|  +-----------------------------+  +-----------------------------+ |
|  |    Observation Space        |  |    Action Space              | |
|  |  - Screenshots (1920x1080) |  |  - pyautogui mouse/keyboard | |
|  |  - Accessibility tree (XML)|  |  - click(x, y)              | |
|  |  - Terminal output          |  |  - type(text)               | |
|  |  - Set-of-Marks overlay     |  |  - scroll(direction)        | |
|  +-----------------------------+  |  - hotkey(keys)             | |
|                                   |  - WAIT / FAIL / DONE       | |
|                                   +-----------------------------+ |
+-------------------------------------------------------------------+
```

### Observation Space

Agents receive observations through multiple modalities:

1. **Screenshots**: High-resolution (1920x1080) screen captures of the current desktop state
2. **Accessibility tree**: XML-formatted structural representation of UI elements with properties
   (role, name, state, bounding box)
3. **Terminal output**: Command-line feedback when terminal interactions are involved
4. **Set-of-Marks (SoM)**: Visual overlay that annotates interactable UI elements with numbered
   markers on screenshots

Four input configurations for agent evaluation:
- Accessibility tree only (text-based agents)
- Screenshot only (vision-based agents)
- Screenshot + Accessibility tree (multimodal agents)
- Set-of-Marks (screenshot with annotated markers)

### Action Space

Actions are formalized through pyautogui-style primitives:

```python
# Mouse actions
click(x, y)              # Left-click at coordinates
double_click(x, y)       # Double-click
right_click(x, y)        # Right-click context menu
drag(x1, y1, x2, y2)     # Drag from point to point
scroll(x, y, clicks)     # Scroll at position

# Keyboard actions
type(text)               # Type text string
hotkey(key1, key2, ...)   # Key combination (e.g., Ctrl+C)
press(key)               # Single key press

# Control actions
WAIT                     # Wait for environment response
FAIL                     # Agent declares task impossible
DONE                     # Agent declares task complete
```

### Task Design

369 tasks across multiple categories and complexity levels:

| Category              | Description                                            | Examples                                        |
|-----------------------|--------------------------------------------------------|-------------------------------------------------|
| Desktop Applications  | Office suites, image editors, text editors             | Edit spreadsheet formula, crop image             |
| Web Browsing          | Browser-based tasks on real websites                   | Fill form, find information, download file        |
| OS File Operations    | File management, system configuration                  | Organize files, change settings, install app      |
| Multi-App Workflows   | Tasks spanning 2+ applications                         | Download data, process in calc, email results     |
| Terminal/CLI          | Command-line operations                                | Run scripts, manage packages, configure services  |

Each task includes:
- **Natural language instruction**: Clear description of the desired outcome
- **Initial state configuration**: VM snapshot + automated setup scripts for reproducible starting state
- **Evaluation script**: Custom Python script that programmatically verifies task completion

### Evaluation Methodology

```
Algorithm: OSWorld Task Evaluation
------------------------------------
Input:  Task T with instruction I, initial state S_0, evaluation script E
Output: Binary success/failure

1:  VM = RestoreSnapshot(S_0)        // Restore to known initial state
2:  RunSetupScripts(VM, T)           // Configure task-specific environment
3:  agent_state = Initialize(VM, I)  // Provide instruction to agent
4:
5:  for step = 1 to MAX_STEPS:
6:      observation = Observe(VM)     // Screenshot + a11y tree
7:      action = Agent(observation, I, history)
8:      if action == DONE or action == FAIL:
9:          break
10:     Execute(VM, action)           // Apply action to real VM
11: end for
12:
13: result = RunEvaluationScript(VM, E)  // Check final VM state
14: return result  // 1 = success, 0 = failure
```

Key properties:
- **Execution-based**: Evaluation checks the actual VM state, not the action sequence
- **Reproducible**: Snapshot restoration ensures identical starting conditions
- **Flexible**: Agents can use any valid action sequence to achieve the goal

## Key Innovations

1. **Real computer environments**: First benchmark using actual VMs (Ubuntu, Windows, macOS) rather
   than simulations, capturing full OS complexity including window management, system dialogs,
   file systems, and multi-application coordination.
2. **Execution-based evaluation**: Custom evaluation scripts check the final state of the VM rather
   than comparing action sequences, allowing credit for any valid approach to completing a task.
3. **Multi-application workflows**: Tasks that require coordinating across multiple applications
   test a fundamentally harder capability than single-app benchmarks.
4. **Scalable infrastructure**: VM snapshot-based approach enables reproducible evaluation at scale
   with automated task setup and teardown.
5. **Multi-modal observation support**: Supporting screenshots, accessibility trees, and Set-of-Marks
   enables systematic comparison of vision-based vs. text-based vs. hybrid agent approaches.

## Experimental Setup

- **Models evaluated** (original paper, early 2024):
  - GPT-4V (screenshot-based and accessibility tree-based)
  - Claude 3 Opus
  - Gemini Pro Vision
  - Various open-source VLMs
- **Input configurations**: Screenshot only, Accessibility tree only, Screenshot + A11y tree, SoM
- **Human baseline**: Expert human evaluators performing the same tasks
- **Max steps**: Configurable per task (typically 15-30 steps)
- **Metrics**: Success Rate (SR) -- percentage of tasks completed correctly

## Results

### Overall Success Rates (Original Paper, 2024)

| Agent                              | Success Rate (%) |
|------------------------------------|:----------------:|
| Human experts                      | **72.36**        |
| GPT-4V (Screenshot + A11y tree)    | **12.24**        |
| GPT-4V (Screenshot only)           | ~10%             |
| GPT-4V (Accessibility tree only)   | ~8%              |
| Claude 3 Opus                      | ~6-8%            |
| Gemini Pro Vision                  | ~4-6%            |
| Open-source VLMs                   | ~1-4%            |

The **60-point gap** between human experts (72.36%) and the best agent (12.24%) highlighted the
enormous challenge of building capable computer-use agents.

### Performance by Input Modality

| Input Type               | GPT-4V Success Rate |
|--------------------------|:-------------------:|
| Screenshot + A11y tree   | **12.24%**          |
| Screenshot only          | ~10%                |
| Accessibility tree only  | ~8%                 |
| Set-of-Marks             | ~11%                |

Multimodal input (screenshot + accessibility tree) provides the best performance, confirming
that both visual and structural information contribute to agent capability.

### Error Analysis: Why Agents Fail

Primary failure modes identified:
1. **GUI grounding errors (~40%)**: Agents fail to correctly identify and locate UI elements,
   clicking wrong buttons, missing menus, or misinterpreting visual layouts.
2. **Operational knowledge gaps (~30%)**: Agents lack knowledge of application-specific workflows
   (e.g., how to access a specific feature in LibreOffice or GIMP).
3. **Multi-step planning failures (~20%)**: Agents lose track of multi-step procedures, skip
   steps, or fail to maintain coherent plans over extended interactions.
4. **Action execution errors (~10%)**: Correct intent but wrong action execution (wrong coordinates,
   typos, incorrect key combinations).

### Subsequent Leaderboard Progress (Post-Publication)

OSWorld became the standard benchmark for computer-use agents, driving rapid progress:

| Period    | Best Agent                | Success Rate |
|-----------|---------------------------|:------------:|
| Apr 2024  | GPT-4V                    | 12.24%       |
| Late 2024 | Claude 3.5 Sonnet         | ~22%         |
| Mid 2025  | Specialized desktop agents | ~40-50%      |
| 2026      | Claude Opus 4.6           | ~72.7%       |

## Limitations

1. **Ubuntu bias**: While the platform supports Windows and macOS, the majority of tasks are
   Ubuntu-based, creating a potential distribution bias toward Linux-centric evaluation.
2. **English-only instructions**: All task instructions are in English, limiting evaluation of
   multilingual agent capabilities.
3. **Static task set**: The fixed set of 369 tasks may not cover emerging application types or
   evolving OS features; task diversity is bounded.
4. **Binary evaluation**: Success is binary (pass/fail) with no partial credit for agents that
   complete most steps correctly but fail at the final step.
5. **VM overhead**: Real VM-based evaluation is computationally expensive (each task requires
   snapshot restore, execution, evaluation), limiting scalability for rapid iteration.
6. **8 Google Drive tasks**: Certain tasks involving Google Drive may encounter setup issues during
   initialization, requiring evaluators to optionally exclude these 8 tasks (running 361 instead).
7. **No efficiency metric**: The benchmark measures only success rate, not the number of steps or
   time required -- an agent that takes 100 steps to succeed scores the same as one taking 5 steps.
8. **Observation gap**: Screen resolution, rendering differences, and font aliasing across VMs
   can introduce visual noise that affects screenshot-based agents inconsistently.

## Key Takeaways

1. **Massive capability gap revealed**: The 72.36% vs. 12.24% gap between humans and the best
   agent (at time of publication) definitively quantified how far computer-use agents were from
   human capability, setting a clear research target.
2. **GUI grounding is the primary bottleneck**: ~40% of failures stem from inability to correctly
   identify and interact with visual UI elements, making vision-grounding a high-priority
   research direction for desktop agents.
3. **Real environments are essential**: The complexity of real OS environments (system dialogs,
   window management, application state) cannot be captured by simulated interfaces, validating
   the need for VM-based evaluation.
4. **Multimodal input helps**: Combining screenshots with accessibility tree information yields
   the best performance, suggesting that both visual and structural understanding are needed.
5. **Benchmark as catalyst**: OSWorld's clear metrics and leaderboard structure catalyzed rapid
   progress in desktop agent development, with success rates improving from 12% to 70%+ within
   two years as dedicated computer-use models (Claude Computer Use, etc.) emerged.
6. **Multi-app workflows are hardest**: Tasks requiring coordination across multiple applications
   have disproportionately low success rates, highlighting the challenge of long-horizon planning
   in complex desktop environments.
7. **Foundational benchmark for the field**: OSWorld established the standard evaluation protocol
   for computer-use agents, analogous to what ImageNet did for vision and SWE-bench for coding.
