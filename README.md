# AI/ML Research Knowledge Repository

Personal knowledge base for AI/Machine Learning research.

## Structure

```
papers/                    # Paper reading notes, organized by topic
  agent/                   # AI Agent research (primary focus)
    foundational/          # ReAct, CoT, Toolformer, Voyager, etc.
    multi_agent/           # MetaGPT, AutoGen, CAMEL, etc.
    memory/                # MemGPT, Reflexion, CoALA, memory surveys
    coding/                # SWE-Agent, Devin, OpenHands
    gui/                   # Computer Use, CogAgent, OmegaUse
    benchmark/             # SWE-bench, AgentBench, WebArena
    safety/                # Agent security papers
    survey/                # Agent survey papers
  llm/                     # Large Language Models
  cv/                      # Computer Vision
  rl/                      # Reinforcement Learning
  diffusion/               # Diffusion Models
  multimodal/              # Multimodal Learning
concepts/                  # Core concepts and definitions
  agent_architecture.md    # Agent components and design
  agent_memory.md          # Memory systems taxonomy
  react_pattern.md         # ReAct loop pattern
  tool_use.md              # Function calling & tool use
  multi_agent_systems.md   # Multi-agent collaboration
  mcp_a2a_protocols.md     # MCP & A2A protocols
  agent_frameworks.md      # Framework comparison
  agent_techniques.md      # Key techniques (CoT, RAG, ToT, etc.)
methods/                   # Algorithms, training techniques
experiments/               # Experiment logs and results
notes/                     # Free-form notes and ideas
  2026_agent_landscape.md  # 2026 Agent industry landscape
references/                # Datasets, benchmarks, tools, links
  agent_benchmarks.md      # Agent evaluation benchmarks
templates/                 # Note templates
```

## Quick Start

- **Paper index**: `papers/agent/_index.md` -- complete paper list with dependency chain
- **Core concepts**: start with `concepts/agent_architecture.md`
- **2026 landscape**: `notes/2026_agent_landscape.md`
- **Search**: `grep -r "keyword" .` or ask Claude Code

## Tags Convention

Use consistent tags in YAML frontmatter:
`transformer`, `attention`, `pretraining`, `finetuning`, `rlhf`,
`scaling`, `alignment`, `benchmark`, `survey`, `architecture`,
`agent`, `react`, `tool-use`, `memory`, `multi-agent`, `mcp`,
`coding`, `gui`, `computer-use`, `foundational`
