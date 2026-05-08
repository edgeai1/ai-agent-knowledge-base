# AI Agent Papers Index

## Foundational Papers (2022-2023)

| Paper | Year | Key Idea |
|-------|------|----------|
| [Chain-of-Thought](foundational/chain_of_thought.md) | 2022 | Step-by-step reasoning via prompting |
| [MRKL](foundational/mrkl.md) | 2022 | LLM routes to expert modules |
| [ReAct](foundational/react.md) | 2022 | Interleaved reasoning + action loops |
| [Toolformer](foundational/toolformer.md) | 2023 | Self-supervised tool-use learning |
| [Reflexion](memory/reflexion.md) | 2023 | Verbal self-reflection across episodes |
| [HuggingGPT](foundational/hugginggpt.md) | 2023 | LLM orchestrates specialist models |
| [Generative Agents](foundational/generative_agents.md) | 2023 | Believable social agent simulation |
| [Tree of Thoughts](foundational/tree_of_thoughts.md) | 2023 | Tree-structured reasoning search |
| [Voyager](foundational/voyager.md) | 2023 | Lifelong learning embodied agent |
| [AutoGPT](foundational/autogpt.md) | 2023 | Fully autonomous goal-driven agent |

## Multi-Agent Systems

| Paper | Year | Key Idea |
|-------|------|----------|
| [MetaGPT](multi_agent/metagpt.md) | 2023 | SOP-encoded multi-agent collaboration |
| [AutoGen](multi_agent/autogen.md) | 2023 | Multi-agent conversation framework |
| [CAMEL](multi_agent/camel.md) | 2023 | Role-playing multi-agent communication |
| [ChatDev](multi_agent/chatdev.md) | 2023 | Virtual software company agents |
| [CORAL](multi_agent/coral.md) | 2026 | Self-evolving multi-agent systems |

## Memory & Planning

| Paper | Year | Key Idea |
|-------|------|----------|
| [MemGPT / Letta](memory/memgpt.md) | 2023 | OS-inspired memory management |
| [CoALA](memory/coala.md) | 2024 | Cognitive architecture for agents |
| [Memory Survey 2026](memory/memory_survey_2026.md) | 2026 | Write-manage-read loop taxonomy |

## Coding Agents

| Paper | Year | Key Idea |
|-------|------|----------|
| [SWE-Agent](coding/swe_agent.md) | 2024 | Agent-Computer Interface design |
| [Devin](coding/devin.md) | 2024 | First AI software engineer |
| [OpenHands](coding/openhands.md) | 2024 | Open-source coding agent platform |
| [CodeAct](coding/codeact.md) | 2024 | Code as unified action space |

## GUI / Computer Use Agents

| Paper | Year | Key Idea |
|-------|------|----------|
| [Claude Computer Use](gui/claude_computer_use.md) | 2024 | First commercial computer use API |
| [CogAgent](gui/cogagent.md) | 2023 | Visual language model for GUI |
| [OmegaUse](gui/omegause.md) | 2026 | General-purpose GUI agent |
| [OSWorld](benchmark/osworld.md) | 2024 | Real OS benchmark for agents |

## Benchmarks

| Paper | Year | Key Idea |
|-------|------|----------|
| [SWE-bench](benchmark/swe_bench.md) | 2024 | Real GitHub issues benchmark |
| [AgentBench](benchmark/agentbench.md) | 2023 | Multi-environment agent eval |
| [WebArena](benchmark/webarena.md) | 2024 | Realistic web environment |
| [GAIA](benchmark/gaia.md) | 2023 | General AI assistant benchmark |

## Safety

| Paper | Year | Key Idea |
|-------|------|----------|
| [AgentDojo](safety/agentdojo.md) | 2024 | Prompt injection attack evaluation |
| [R-Judge](safety/r_judge.md) | 2024 | Safety risk awareness benchmark |

## Surveys

| Paper | Year | Scope |
|-------|------|-------|
| [LLM Agent Survey (Fudan)](survey/llm_agent_survey_fudan.md) | 2023 | Comprehensive agent taxonomy |
| [Agent Survey (Renmin)](survey/llm_agent_survey_renmin.md) | 2023 | Brain-perception-action framework |
| [From LLM Reasoning to Autonomous AI Agents](survey/reasoning_to_agents_2025.md) | 2025 | Reasoning-to-agent comprehensive review |

## Dependency Chain

```
Chain-of-Thought (reasoning) 
  -> MRKL (tool routing) 
    -> ReAct (reasoning + acting) 
      -> Toolformer (learned tool use)
      -> Reflexion (self-improvement)
      -> AutoGPT (full autonomy)
      -> HuggingGPT (orchestration)
      -> Generative Agents (memory + social)
      -> Tree of Thoughts (structured search)
      -> Voyager (embodied + skill library)
```
