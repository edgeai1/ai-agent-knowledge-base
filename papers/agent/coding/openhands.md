---
title: "OpenHands: An Open Platform for AI Software Developers as Generalist Agents"
authors: Xingyao Wang, Boxuan Li, Yufan Song, Frank F. Xu, Xiangru Tang, Mingchen Zhuge, Jiayi Pan, Yueqi Song, Bowen Li, Jaskirat Singh, Hoang H. Tran, Fuqiang Li, Ren Ma, Mingzhang Zheng, Bill Qian, Yanjun Shao, Niklas Muennighoff, Ziwen Liu, Dong Guo, Qian Liu, Meng Cao, Haoran Zhang, Yanzhi Zhang, Shuai Lu, Wenwen Qu, Haochuan Wang, Jiahui Zhang, Zhao Xu, Tianyu Liu, Samuel Arcadinho, Zhoujun Cheng, Chengcheng Han, Yuchen Eleanor Jiang, Suyash Vardhan Mathur, Manish Shetty, Wen-Ding Li, Yixin Ou, Yunxiang Li, Robert Brennan, Jun Shern Chan, Jenny Ma, Boyuan Zheng, Xizhou Zhu, Zhiyuan Liu, Jie Tang, Graham Neubig, Yuxiao Dong
affiliation: All Hands AI, Carnegie Mellon University, University of Illinois, multiple institutions
venue: ICLR 2025
year: 2024-2025
url: https://arxiv.org/abs/2407.16741
code: https://github.com/OpenHands/OpenHands
project: https://www.openhands.dev
sdk_paper: https://arxiv.org/abs/2511.03690
tags: [coding-agent, open-source, platform, sandbox, model-agnostic, SWE-bench, event-stream]
status: done
license: MIT
stars: 68K+ GitHub stars
funding: $18.8M Series A (All Hands AI)
---

## TL;DR

OpenHands (formerly OpenDevin) is an MIT-licensed, open-source platform for building AI software development agents, published at ICLR 2025. Built on an event-stream architecture with Docker-sandboxed execution, it provides a model-agnostic framework supporting 100+ LLM providers via LiteLLM. The default CodeAct agent combines code execution with reasoning, achieving state-of-the-art results: 53% on SWE-bench Verified with CodeAct 2.1 (Claude 3.5 Sonnet), scaling to 72% with Claude Sonnet 4.5 extended thinking. OpenHands demonstrates that open-source, model-agnostic agent platforms can match or exceed proprietary vertical solutions like Devin.

## Motivation & Problem

By mid-2024, the coding agent landscape was dominated by two extremes:

1. **Proprietary vertical solutions (Devin, etc.)**: Powerful but closed-source, expensive, vendor-locked, and scientifically opaque. Researchers could not inspect, modify, or reproduce results.
2. **Research prototypes (SWE-Agent, etc.)**: Academically rigorous but narrow in scope, difficult to deploy in production, and tied to specific model architectures.

OpenHands addresses several critical gaps:

- **No open platform for agent development**: Building a coding agent required reimplementing sandboxing, tool use, web browsing, and evaluation infrastructure from scratch for every new project
- **Model lock-in**: Most agent systems were hardcoded to specific LLMs (typically GPT-4 or Claude), preventing comparison across models and limiting flexibility
- **No reusable agent SDK**: The common infrastructure needed by all software agents (sandbox, event logging, state management, API serving) was rebuilt independently by each project
- **Reproducibility crisis**: Proprietary agents (Devin) published benchmark numbers that could not be independently verified or reproduced
- **Community fragmentation**: Dozens of independent coding agent projects with incompatible interfaces, preventing collaboration and comparison

## Method: Architecture

### Three Core Components

OpenHands consists of three main components that together form a complete agent development and execution platform:

#### 1. Agent Abstraction Layer (AgentHub)

Multiple agent implementations can be contributed and compared within the same platform:

- **CodeAct Agent** (default): The primary agent combining code execution with reasoning. Uses function calling to interact with the environment through Python code and bash commands.
- **Browser Agent**: Specialized for web browsing tasks using BrowserGym
- **Delegate Agent**: Orchestrates sub-agents for complex tasks requiring delegation
- Community can contribute new agent implementations with minimal boilerplate

#### 2. Event Stream (State Management Core)

The event stream is the central architectural abstraction -- a chronological, append-only log of all actions and observations:

```
Event Stream Architecture:

  User Instruction -> [Event 1: MessageAction]
  Agent Response   -> [Event 2: AgentAction(code="ls -la")]
  Sandbox Result   -> [Event 3: Observation(stdout="...")]
  Agent Response   -> [Event 4: AgentAction(code="edit file.py")]
  Sandbox Result   -> [Event 5: Observation(file_edited=True)]
  ...

  State = f(Event Stream) -- state is always derivable from the event log
```

Properties of the event-sourced design:
- **Immutable history**: All actions and observations are permanently logged, enabling replay, debugging, and analysis
- **Deterministic state recovery**: The current state can always be reconstructed by replaying the event stream from the beginning
- **Audit trail**: Complete record of every agent decision and environmental change
- **Multi-agent coordination**: Multiple agents can read from and write to the same event stream
- **Session persistence**: Work can be paused and resumed by loading the event stream

The state data structure includes: the event stream itself, accumulative LLM call costs, metadata for multi-agent delegation tracking, and execution-related parameters.

#### 3. Runtime (Sandboxed Execution Environment)

Each agent session runs in a securely isolated Docker container providing:

**Bash Shell**: Full Linux environment for command execution
- Connected via a REST API server running inside the container
- Maintains a persistent bash shell session across the entire task
- Agent sends commands, receives stdout/stderr output

**IPython/Jupyter Kernel**: For Python code execution
- Enables running Python code with access to installed packages
- Supports data analysis, file manipulation, and programmatic operations
- Provides richer output than bash (DataFrames, plots, structured data)

**BrowserGym Interface**: For web automation
- Full browser interaction using a domain-specific language from BrowserGym
- Enables searching documentation, browsing websites, and interacting with web applications
- Supports page navigation, element interaction, and content extraction

**Workspace Mounting**: A configurable workspace directory is mounted into the sandbox
- Users specify which files/directories the agent should work on
- Files persist across the session and are accessible to the agent
- Changes made by the agent are reflected in the mounted directory

**Security Model**: Disposable Docker containers as the primary confinement mechanism
- Each session gets a fresh container with no access to the host system
- Containers are destroyed after session completion
- Network access can be restricted per security requirements
- SSH access available for debugging

### CodeAct Agent Design

The CodeAct agent is the default "strong generalist" agent that implements a specific paradigm for agent-environment interaction:

**Core principle**: Instead of generating structured tool calls or JSON actions, the agent writes and executes Python code or bash commands to interact with the environment. This is the "CodeAct" paradigm -- using code as the universal action language.

**CodeAct 1.0 -> 2.1 Evolution:**

| Version    | Key Changes                                           | SWE-bench Verified |
|------------|-------------------------------------------------------|--------------------|
| v1.0       | IPython-based code execution, prompt-based actions    | ~25%               |
| v1.8       | Claude 3.5 Sonnet, improved prompts                  | 26% (Lite)         |
| v2.1       | Function calling, Claude 3.5 (Oct), directory fixes   | **53%**            |
| + Claude 4 | Extended thinking, latest frontier model              | **72%**            |

**CodeAct 2.1 improvements** (November 2025):
1. Switched from prompt-based actions to function calling for more reliable action generation
2. Updated to Anthropic's new Claude 3.5 Sonnet model (October 2024 release)
3. Multiple fixes to make it easier for agents to traverse and understand directory structures
4. Better error handling and recovery mechanisms

### Model-Agnostic Design via LiteLLM

OpenHands uniquely supports any LLM provider through LiteLLM integration:

- **100+ providers supported**: OpenAI, Anthropic, Google, Mistral, Cohere, Together, Groq, local models (Ollama, vLLM), and many more
- **No custom integrations needed**: Adding a new provider requires only an API key, not code changes
- **Non-function-calling fallback**: First-class support for models that lack function calling capability via prompt-based action generation
- **Unified interface**: The same agent code works with any model, enabling direct comparison across providers
- **Cost tracking**: Automatic cost calculation across different providers and pricing models

This model-agnostic design is a key differentiator from Devin (locked to proprietary SWE-1/1.5) and most other agent frameworks.

### Server Architecture

The OpenHands Server provides three interfaces:
- **Web UI**: Browser-based interaction with the agent, including real-time workspace viewing
- **REST API**: Programmatic access for integration with CI/CD pipelines and external tools
- **WebSocket API**: Real-time streaming of agent actions and observations

Additional interfaces:
- **VS Code integration**: Remote development environment access within the sandbox
- **VNC access**: Visual desktop access for GUI-based tasks
- **Browser view**: Live browser state for web automation tasks

## Key Innovations

1. **Event-sourced agent architecture**: Immutable event stream as the single source of truth enables replay, debugging, audit, and multi-agent coordination
2. **Model-agnostic design**: First major agent platform to support 100+ LLM providers without custom integration work, including fallbacks for non-function-calling models
3. **CodeAct paradigm**: Using executable code as the universal action language rather than structured tool calls -- more flexible and expressive
4. **Open platform for agent research**: Community-contributed agents, benchmarks, and improvements; 188+ contributors, 68K+ GitHub stars
5. **Production-grade sandboxing**: Docker-based isolation with SSH, Jupyter, and browser access in a reusable, configurable runtime

## Experimental Setup

### Benchmarks Evaluated

The ICLR 2025 paper evaluates OpenHands agents on 15 challenging tasks:
- **SWE-bench**: Real-world GitHub issue resolution (primary coding benchmark)
- **WebArena**: Web navigation in realistic environments
- **GAIA**: General AI assistant tasks
- **Multiple additional benchmarks** spanning software engineering and web interaction

### Model Comparison (on SWE-bench Verified)

| Configuration                          | Resolve Rate |
|----------------------------------------|-------------|
| OpenHands CodeAct v1.8 + Claude 3.5    | 26% (Lite)  |
| OpenHands CodeAct 2.1 + Claude 3.5     | **53%**     |
| OpenHands + Claude 3.7 Sonnet          | ~43%*       |
| OpenHands + Claude Sonnet 4.5 (thinking)| **72%**    |
| OpenHands + Claude 4                   | ~72%        |

*Note: 43% on SWE-bench Verified subset re-run; 19.25% on SWE-bench Live

## Results

### SWE-bench Performance Trajectory

OpenHands has shown consistent improvement driven by both agent architecture refinements and underlying model improvements:

- **CodeAct 2.1 (Nov 2025)**: 53% on SWE-bench Verified -- first open-source agent to surpass 50%, surpassing Claude 3.5's standalone 49%
- **With Claude Sonnet 4.5 extended thinking**: 72% -- among the highest scores for any agent platform on SWE-bench Verified
- **SWE-bench Live**: ~19.25% (highlighting the static vs. live benchmark gap)

### Cross-Benchmark Generalization

OpenHands demonstrates strong results beyond just SWE-bench:
- Competitive performance on WebArena (web navigation)
- Strong results on GAIA (general assistant tasks)
- The architecture's generality enables it to operate consistently across diverse model providers and task types

### SWE-bench Live: The Reality Check

SWE-bench Live tests agents on new, previously unseen issues to prevent data contamination:
- OpenHands + Claude 3.7 Sonnet: 43.20% on SWE-bench Verified vs. 19.25% on SWE-bench Live
- This 24-point drop is the largest documented gap between static and live benchmark performance
- Suggests that part of high static benchmark scores may reflect train/test contamination or overfitting to the benchmark distribution

### OpenHands Index

Introduced as a standardized evaluation framework for coding agents that goes beyond SWE-bench:
- Evaluates agents across multiple dimensions (not just bug fixing)
- Includes code generation, refactoring, and understanding tasks
- Aims to provide a more comprehensive picture of agent coding capabilities
- Designed to reduce over-optimization on any single benchmark

## Analysis & Insights

### Why Model-Agnostic Matters

OpenHands' model-agnostic design has revealed several important findings:
- **Agent scaffolding matters as much as the base model**: The same Claude 3.5 model scores 49% standalone but 53% with CodeAct 2.1 scaffolding
- **Model improvements compound with agent improvements**: Upgrading from Claude 3.5 to Claude 4 within the same scaffolding jumps from 53% to 72%
- **Open-source can match commercial**: OpenHands' best results (72%) match or exceed Devin (45.8%), demonstrating that access to frontier models plus good scaffolding beats proprietary vertical integration

### The CodeAct Advantage

Using executable code as the action language (instead of structured JSON tool calls) provides:
- **Composability**: Multiple operations in a single action (variable assignment + file read + string manipulation)
- **Debuggability**: Standard programming debugging tools apply
- **Expressiveness**: Any Python or bash operation is a valid action, not limited to predefined tool schemas
- **Familiarity**: The agent operates in the same paradigm human developers use

### Production Deployment Patterns

The OpenHands Agent SDK (November 2025) extends the platform for production use:
- Immutable configuration prevents accidental state mutation
- Event-sourced state model enables reliable session management
- Workspace-level remote interfaces (VS Code, VNC, browser) for human oversight
- Composable agent architecture for building specialized agent workflows

## Limitations

- **Benchmark dependence on underlying model**: OpenHands' scores are heavily determined by the underlying LLM quality; the platform amplifies model capability but cannot compensate for fundamental model weaknesses
- **SWE-bench overfitting concerns**: The 24-point gap between static and live benchmark performance raises questions about how much of the reported performance is genuine vs. data contamination
- **Resource overhead**: Docker sandboxing adds startup latency and resource overhead per session
- **Complexity for simple tasks**: The full platform is over-engineered for simple code generation; lightweight alternatives are more appropriate for single-file edits
- **Browser automation limitations**: BrowserGym-based web interaction is less robust than purpose-built browser automation tools
- **Security surface**: Even with Docker sandboxing, running arbitrary agent-generated code creates security considerations for production deployment
- **Community coordination challenges**: With 188+ contributors, maintaining quality and architectural coherence is an ongoing challenge

## Key Takeaways

1. **Open-source agent platforms can match or exceed proprietary solutions**: OpenHands' 72% on SWE-bench Verified vs. Devin's 45.8% demonstrates that model-agnostic platforms with access to frontier models outperform proprietary vertical integration.
2. **Event-stream architecture is a principled foundation for agent systems**: The immutable, append-only event log provides replay, debugging, audit, and multi-agent coordination capabilities that ad-hoc state management cannot match.
3. **Model-agnostic design is essential for longevity**: As models improve rapidly (Claude 3.5 -> 4 doubled SWE-bench scores), platforms locked to specific models become obsolete. OpenHands' 100+ provider support ensures it benefits from every model improvement.
4. **CodeAct (code as action) is more expressive than structured tool calls**: Executing Python/bash code provides composability, debuggability, and expressiveness that predefined tool schemas cannot match.
5. **The gap between static and live benchmarks (24 points) is a warning**: The community should weight SWE-bench Live results more heavily than static verified results when assessing real-world agent capability.
6. **Agent scaffolding amplifies model capability**: The same underlying model scores significantly higher with good agent architecture (49% standalone vs. 53% with CodeAct 2.1), demonstrating that both model quality and agent design matter.
7. **Community-driven development works for agent platforms**: 68K+ stars, 188+ contributors, and an ICLR 2025 publication demonstrate that open-source agent development can achieve both research excellence and production quality simultaneously.
8. **Docker-based sandboxing provides adequate security for most use cases**: Disposable containers as the primary confinement mechanism balance security with usability, though production deployments require additional hardening.
