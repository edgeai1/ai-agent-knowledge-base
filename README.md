<p align="center">
  <h1 align="center">🤖 AI Agent 研究知识库</h1>
  <p align="center">
    <strong>系统化的 AI 智能体研究论文、核心概念与前沿追踪</strong>
  </p>
  <p align="center">
    📄 110 个文件 · 📝 23,317 行 · 🔬 43 篇 Deep Research · 📊 12 篇基准测试 · 🇨🇳 全中文
  </p>
</p>

---

## 📋 目录

- [快速导航](#-快速导航)
- [知识库结构](#-知识库结构)
- [论文索引](#-论文索引)
  - [奠基论文](#-奠基论文-2022-2023)
  - [深度研究智能体](#-深度研究智能体-38篇)
  - [多智能体系统](#-多智能体系统)
  - [记忆与规划](#-记忆与规划)
  - [编码智能体](#-编码智能体)
  - [GUI / 计算机使用](#-gui--计算机使用)
  - [工作流与编排](#-工作流与编排)
  - [Agentic RAG](#-agentic-rag)
  - [基准测试](#-基准测试-12篇)
  - [安全](#-安全)
  - [综述论文](#-综述论文)
- [核心概念](#-核心概念)
- [2026 全景](#-2026-智能体全景)
- [使用指南](#-使用指南)

---

## 🚀 快速导航

| 入口 | 说明 |
|------|------|
| 📑 [**论文总索引**](papers/agent/_index.md) | 全部论文列表、行数统计、知识依赖链 |
| 🏗️ [**智能体架构**](concepts/agent_architecture.md) | 核心组件、设计模式、架构选择 |
| 🧠 [**记忆系统**](concepts/agent_memory.md) | 记忆分类、五大机制、评估基准 |
| 🔧 [**工具使用**](concepts/tool_use.md) | 函数调用、ACI 设计、MCP 协议 |
| 🌐 [**MCP & A2A 协议**](concepts/mcp_a2a_protocols.md) | 2026 智能体互操作标准 |
| 📊 [**2026 全景**](notes/2026_agent_landscape.md) | 行业动态、SOTA 排行、研究前沿 |

---

## 📁 知识库结构

```
📂 papers/                         论文笔记（按方向分类）
├── 📂 agent/
│   ├── 🏛️ foundational/           奠基论文 (9篇)
│   ├── 🔬 deep_research/          深度研究智能体 (38篇) ⭐ 重点方向
│   ├── 👥 multi_agent/            多智能体系统 (4篇)
│   ├── 🧠 memory/                 记忆与规划 (4篇)
│   ├── 💻 coding/                 编码智能体 (4篇)
│   ├── 🖥️ gui/                    GUI/计算机使用 (3篇)
│   ├── ⚙️ workflow/               工作流与编排 (2篇)
│   ├── 🔍 agentic_rag/            智能体RAG (2篇)
│   ├── 📊 benchmark/              基准测试 (12篇)
│   ├── 🛡️ safety/                 安全 (1篇)
│   └── 📖 survey/                 综述 (4篇)
├── 📂 llm/                        LLM 基础论文 (2篇)
│
📂 concepts/                       核心概念 (11篇)
📂 notes/                          研究笔记
📂 references/                     参考资料
📂 templates/                      笔记模板
```

---

## 📄 论文索引

### 🏛️ 奠基论文 (2022-2023)

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **Chain-of-Thought** - 思维链提示 | 2022 | NeurIPS 2022 | [📝 笔记](papers/agent/foundational/chain_of_thought.md) |
| **MRKL** - 模块化推理知识语言系统 | 2022 | arXiv | [📝 笔记](papers/agent/foundational/mrkl.md) |
| **ReAct** - 推理与行动协同 | 2022 | ICLR 2023 Oral | [📝 笔记](papers/agent/foundational/react.md) |
| **Toolformer** - 自学工具使用 | 2023 | NeurIPS 2023 | [📝 笔记](papers/agent/foundational/toolformer.md) |
| **Reflexion** - 语言强化学习 | 2023 | NeurIPS 2023 | [📝 笔记](papers/agent/memory/reflexion.md) |
| **HuggingGPT** - LLM 编排专用模型 | 2023 | NeurIPS 2023 | [📝 笔记](papers/agent/foundational/hugginggpt.md) |
| **Generative Agents** - 生成式智能体 | 2023 | UIST 2023 Best Paper | [📝 笔记](papers/agent/foundational/generative_agents.md) |
| **Tree of Thoughts** - 思维树 | 2023 | NeurIPS 2023 | [📝 笔记](papers/agent/foundational/tree_of_thoughts.md) |
| **Voyager** - 终身学习具身智能体 | 2023 | NeurIPS 2023 Spotlight | [📝 笔记](papers/agent/foundational/voyager.md) |
| **AutoGPT** - 自主目标驱动智能体 | 2023 | 开源项目 | [📝 笔记](papers/agent/foundational/autogpt.md) |

### 🔬 深度研究智能体 (43篇)

#### RL 搜索系列

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **R1-Searcher** - RL 搜索能力激励 | 2025.03 | arXiv | [📝 笔记](papers/agent/deep_research/r1_searcher.md) |
| **Search-R1** - 推理+搜索 RL 框架 | 2025.03 | COLM 2025 | [📝 笔记](papers/agent/deep_research/search_r1.md) |
| **R1-Searcher++** - 动态知识获取 | 2025.05 | arXiv | [📝 笔记](papers/agent/deep_research/r1_searcher_plus.md) |
| **DeepResearcher** - 端到端 RL | 2025 | EMNLP 2025 | [📝 笔记](papers/agent/deep_research/deep_researcher.md) |
| **PANGU DeepDiver** - 搜索强度缩放 | 2025.05 | NeurIPS 2025 | [📝 笔记](papers/agent/deep_research/pangu_deepdiver.md) |
| **PokeeResearch** - RLAIF 7B | 2025.10 | arXiv | [📝 笔记](papers/agent/deep_research/pokee_research.md) |
| **ZeroSearch** - 无需真实搜索的 RL | 2025.05 | arXiv | [📝 笔记](papers/agent/deep_research/zerosearch.md) |
| **CoSearch** - 推理+排序联合 GRPO | 2026.04 | arXiv | [📝 笔记](papers/agent/deep_research/cosearch.md) |
| **T²PO** - 不确定性引导探索控制 | 2026.05 | ICML 2026 Spotlight | [📝 笔记](papers/agent/deep_research/t2po.md) |

#### Web Agent 系列

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **MindSearch** - 认知深度搜索 | 2024 | arXiv | [📝 笔记](papers/agent/deep_research/mindsearch.md) |
| **Search-o1** - 搜索增强推理 | 2025 | EMNLP 2025 | [📝 笔记](papers/agent/deep_research/search_o1.md) |
| **WebDancer** - 自主信息搜索 | 2025.05 | arXiv | [📝 笔记](papers/agent/deep_research/webdancer.md) |
| **WebSailor** - 超人级推理 72B | 2025.07 | arXiv | [📝 笔记](papers/agent/deep_research/websailor.md) |
| **WebThinker** - LRM 自主搜索报告 | 2026 | WWW 2026 Oral | [📝 笔记](papers/agent/deep_research/webthinker.md) |

#### 文献综合与报告生成

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **STORM** - 多视角文章写作 | 2024 | NAACL 2024 | [📝 笔记](papers/agent/deep_research/storm.md) |
| **OpenScholar** - 科学文献综合 | 2026 | Nature 2026 | [📝 笔记](papers/agent/deep_research/openscholar.md) |
| **OpenResearcher** - 开放研究轨迹合成 | 2026 | arXiv | [📝 笔记](papers/agent/deep_research/open_researcher.md) |
| **O-Researcher** - 多智能体蒸馏 | 2026 | arXiv | [📝 笔记](papers/agent/deep_research/o_researcher.md) |
| **Tongyi DeepResearch** - 通义深度研究 | 2025 | 技术报告 | [📝 笔记](papers/agent/deep_research/tongyi_deep_research.md) |
| **Marco DeepResearch** - 验证中心设计 | 2026.03 | arXiv | [📝 笔记](papers/agent/deep_research/marco_deep_research.md) |
| **Step-DeepResearch** - 原子能力分解 32B | 2025.12 | 技术报告 | [📝 笔记](papers/agent/deep_research/step_deep_research.md) |

#### 科研自动化

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **ResearchAgent** - 迭代想法生成 | 2024 | NAACL 2025 | [📝 笔记](papers/agent/deep_research/research_agent.md) |
| **AI Scientist** - 全自动科研 | 2024→2026 | Nature 2026 | [📝 笔记](papers/agent/deep_research/ai_scientist.md) |
| **AI Scientist v2** - 智能体树搜索 | 2025 | arXiv | [📝 笔记](papers/agent/deep_research/ai_scientist_v2.md) |
| **Agent Laboratory** - 自主研究工作流 | 2025 | EMNLP 2025 | [📝 笔记](papers/agent/deep_research/agent_laboratory.md) |
| **DeepScientist** - 渐进式前沿发现 | 2025.09 | arXiv | [📝 笔记](papers/agent/deep_research/deep_scientist.md) |
| **AgentRxiv** - 协作自主研究 | 2025 | arXiv | [📝 笔记](papers/agent/deep_research/agentrxiv.md) |
| **SPARK** - 癌症病理发现 | 2026 | Nature Medicine | [📝 笔记](papers/agent/deep_research/spark_pathology.md) |
| **SciResearcher** - 前沿科学推理 | 2026.05 | arXiv | [📝 笔记](papers/agent/deep_research/sci_researcher.md) |

#### 检索与训练基础设施

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **AgentIR** - 推理感知检索 | 2026.03 | arXiv | [📝 笔记](papers/agent/deep_research/agentir.md) |
| **DeepResearch-9K** - 9K 训练集 | 2026.03 | arXiv | [📝 笔记](papers/agent/deep_research/deepresearch_9k.md) |
| **LiteResearcher** - 零成本 RL 4B | 2026.04 | arXiv | [📝 笔记](papers/agent/deep_research/lite_researcher.md) |
| **OpenSearch-VL** - 多模态深搜 | 2026 | arXiv | [📝 笔记](papers/agent/deep_research/opensearch_vl.md) |

#### 架构与效率

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **DeepAgent** - 通用推理智能体 | 2026 | WWW 2026 Oral | [📝 笔记](papers/agent/deep_research/deep_agent.md) |
| **W&D** - 并行工具调用 | 2026.02 | Salesforce | [📝 笔记](papers/agent/deep_research/wide_and_deep.md) |
| **Self-Manager** - 并行 Agent 循环 | 2026.01 | arXiv | [📝 笔记](papers/agent/deep_research/self_manager.md) |
| **DeerFlow** - ByteDance 研究框架 | 2025 | 开源 (66K+ Stars) | [📝 笔记](papers/agent/deep_research/deerflow.md) |
| **EvoFSM** - FSM 可控自进化 | 2026.01 | arXiv | [📝 笔记](papers/agent/deep_research/evofsm.md) |
| **DeepVerifier** - 推理时验证缩放 | 2026.01 | ACL 2026 | [📝 笔记](papers/agent/deep_research/deep_verifier.md) |

#### Deep Research 综述

| 论文 | 年份 | 导航 |
|------|------|------|
| 📖 Deep Research Survey | 2025 | [📝 笔记](papers/agent/deep_research/deep_research_survey.md) |
| 📖 Deep Research Agents Roadmap | 2025 | [📝 笔记](papers/agent/deep_research/deep_research_agents_roadmap.md) |
| 📖 从搜索到深度研究演进 | 2025 | [📝 笔记](papers/agent/deep_research/web_search_to_deep_research.md) |
| 📖 Deep Research of Deep Research | 2026.03 | [📝 笔记](papers/agent/deep_research/deep_research_of_deep_research.md) |

### 👥 多智能体系统

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **MetaGPT** - SOP 编码协作 | 2023 | ICLR 2024 Oral | [📝 笔记](papers/agent/multi_agent/metagpt.md) |
| **AutoGen** - 多智能体对话 | 2023 | COLM 2024 | [📝 笔记](papers/agent/multi_agent/autogen.md) |
| **CAMEL** - 角色扮演通信 | 2023 | NeurIPS 2023 | [📝 笔记](papers/agent/multi_agent/camel.md) |
| **ChatDev** - 虚拟软件公司 | 2023 | ACL 2024 | [📝 笔记](papers/agent/multi_agent/chatdev.md) |

### 🧠 记忆与规划

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **MemGPT / Letta** - 虚拟内存管理 | 2023 | ICLR 2024 Spotlight | [📝 笔记](papers/agent/memory/memgpt.md) |
| **CoALA** - 认知架构框架 | 2024 | TMLR 2024 | [📝 笔记](papers/agent/memory/coala.md) |
| 📖 2026 记忆系统综述 | 2026 | arXiv | [📝 笔记](papers/agent/memory/memory_survey_2026.md) |

### 💻 编码智能体

| 论文 | 年份 | 会议 | 导航 |
|------|------|------|------|
| **SWE-Agent** - 智能体计算机接口 | 2024 | arXiv | [📝 笔记](papers/agent/coding/swe_agent.md) |
| **CodeAct** - 代码即行动空间 | 2024 | ICML 2024 | [📝 笔记](papers/agent/coding/codeact.md) |
| **Devin** - 首个 AI 软件工程师 | 2024 | 产品发布 | [📝 笔记](papers/agent/coding/devin.md) |
| **OpenHands** - 开源编码平台 | 2024 | ICLR 2025 | [📝 笔记](papers/agent/coding/openhands.md) |

### 🖥️ GUI / 计算机使用

| 论文 | 年份 | 导航 |
|------|------|------|
| **CogAgent** - GUI 视觉语言模型 | 2023 | [📝 笔记](papers/agent/gui/cogagent.md) |
| **Claude Computer Use** - 商业计算机使用 API | 2024 | [📝 笔记](papers/agent/gui/claude_computer_use.md) |
| **OmegaUse** - 通用 GUI 智能体 | 2026 | [📝 笔记](papers/agent/gui/omegause.md) |

### ⚙️ 工作流与编排

| 论文 | 年份 | 导航 |
|------|------|------|
| **AFlow** - MCTS 自动工作流生成 | 2025 | [📝 笔记](papers/agent/workflow/aflow.md) |
| 📖 Agent Workflow 综述 | 2025 | [📝 笔记](papers/agent/workflow/agent_workflow_survey.md) |

### 🔍 Agentic RAG

| 论文 | 年份 | 导航 |
|------|------|------|
| **Self-RAG** - 自反思检索增强生成 | 2024 | [📝 笔记](papers/agent/agentic_rag/self_rag.md) |
| 📖 Agentic RAG 综述 | 2025 | [📝 笔记](papers/agent/agentic_rag/agentic_rag_survey.md) |

### 📊 基准测试 (12篇)

| 基准 | 年份 | 关注点 | 导航 |
|------|------|--------|------|
| **SWE-bench** | 2024 | GitHub Issue 解决 | [📝 笔记](papers/agent/benchmark/swe_bench.md) |
| **AgentBench** | 2023 | 8 环境智能体评估 | [📝 笔记](papers/agent/benchmark/agentbench.md) |
| **WebArena** | 2024 | 真实网络环境 | [📝 笔记](papers/agent/benchmark/webarena.md) |
| **OSWorld** | 2024 | 真实操作系统 | [📝 笔记](papers/agent/benchmark/osworld.md) |
| **BrowseComp-Plus** | 2026 | 公平 DR 评估 (ACL 2026) | [📝 笔记](papers/agent/benchmark/browsecomp_plus.md) |
| **IDRBench** | 2026 | 交互式深度研究 | [📝 笔记](papers/agent/benchmark/idrbench.md) |
| **DeepResearch Bench** | 2026 | 9,430 细粒度指标 | [📝 笔记](papers/agent/benchmark/deep_research_bench.md) |
| **ReportBench** | 2025 | 学术综述任务 | [📝 笔记](papers/agent/benchmark/reportbench.md) |
| **DeepResearchEval** | 2026 | 自动化评估框架 | [📝 笔记](papers/agent/benchmark/deep_research_eval.md) |
| **SAGE** | 2026 | DR 检索基准 | [📝 笔记](papers/agent/benchmark/sage_retrieval.md) |
| **DeepSearchQA** | 2026 | Google DeepMind 900 提示 | [📝 笔记](papers/agent/benchmark/deepsearchqa.md) |
| **DRACO** | 2026 | Perplexity 跨领域 | [📝 笔记](papers/agent/benchmark/draco.md) |

### 🛡️ 安全

| 论文 | 年份 | 导航 |
|------|------|------|
| **AgentDojo** - 提示注入攻击评估 | 2024 | [📝 笔记](papers/agent/safety/agentdojo.md) |

### 📖 综述论文

| 论文 | 年份 | 导航 |
|------|------|------|
| LLM Agent 综述 (复旦) | 2023 | [📝 笔记](papers/agent/survey/llm_agent_survey_fudan.md) |
| LLM Agent 综述 (人大) | 2023 | [📝 笔记](papers/agent/survey/llm_agent_survey_renmin.md) |
| 从推理到智能体 | 2025 | [📝 笔记](papers/agent/survey/reasoning_to_agents_2025.md) |
| RL Agentic Search 综述 | 2025 | [📝 笔记](papers/agent/survey/rl_agentic_search_survey.md) |
| Deep Search Agents 综述 | ACL 2026 | [📝 笔记](papers/agent/survey/deep_search_agents_survey.md) |
| LLM Agent 综述合集 | 2023-2024 | [📝 笔记](papers/llm/llm_agent_surveys.md) |

---

## 💡 核心概念

| 概念 | 说明 | 导航 |
|------|------|------|
| 🏗️ **智能体架构** | 感知-推理-行动-观察循环 | [查看](concepts/agent_architecture.md) |
| 🔄 **架构模式** | ReAct、Plan-Execute、Reflexion、多智能体 | [查看](concepts/agent_architecture_patterns.md) |
| 🧠 **记忆系统** | 短期/长期、RAG、反思、虚拟上下文 | [查看](concepts/agent_memory.md) |
| 🧠 **记忆深入** | 存储后端、代码示例、反思与遗忘 | [查看](concepts/agent_memory_systems.md) |
| 🔧 **工具使用** | 函数调用、ACI 设计、MCP 协议 | [查看](concepts/tool_use.md) |
| 🌐 **MCP & A2A** | 2026 智能体互操作标准 | [查看](concepts/mcp_a2a_protocols.md) |
| 👥 **多智能体** | 协作模式、框架对比、趋势 | [查看](concepts/multi_agent_systems.md) |
| ⚡ **关键技术** | CoT, RAG, ToT, 自一致性, 提示链 | [查看](concepts/agent_techniques.md) |
| 📐 **框架对比** | LangGraph, AutoGen, CrewAI, MetaGPT | [查看](concepts/agent_frameworks.md) |
| 🔄 **ReAct 模式** | Thought-Action-Observation 循环 | [查看](concepts/react_pattern.md) |
| 📘 **智能体基础** | Agent = LLM + 记忆 + 规划 + 工具 | [查看](concepts/llm_agent_fundamentals.md) |

---

## 🌍 2026 智能体全景

> 详见 [2026 智能体全景报告](notes/2026_agent_landscape.md)

**SWE-bench Verified 排行榜（2026.03）**

| 排名 | 模型/智能体 | 分数 |
|------|------------|------|
| 🥇 | Claude Opus 4.5 | 80.9% |
| 🥈 | Claude Opus 4.6 | 80.8% |
| 🥉 | Gemini 3.1 Pro | 80.6% |
| 4 | mini-SWE-agent | >74% |
| 5 | OpenHands (Claude 4) | 72% |

**关键协议**

| 协议 | 创建者 | 用途 |
|------|--------|------|
| **MCP** | Anthropic → Linux Foundation | 智能体 ↔ 工具 |
| **A2A** | Google → Linux Foundation | 智能体 ↔ 智能体 |

---

## 📖 使用指南

### 阅读路径推荐

```
入门：Chain-of-Thought → ReAct → Toolformer → 智能体架构概念

├── 深度研究方向（重点）：
│   ├── 文献综合: STORM → OpenScholar → OpenResearcher
│   ├── RL 搜索: R1-Searcher → Search-R1 → DeepResearcher → PANGU DeepDiver
│   ├── Web Agent: MindSearch → WebDancer → WebSailor → WebThinker
│   ├── 科研自动化: AI Scientist → Agent Laboratory → SciResearcher
│   └── 训练基础设施: DeepResearch-9K → LiteResearcher → AgentIR

├── 多智能体方向：CAMEL → MetaGPT → AutoGen → ChatDev
├── 记忆方向：Generative Agents → Reflexion → MemGPT → CoALA
├── 编码智能体：SWE-Agent → CodeAct → Devin → OpenHands
└── 综合理解：各综述论文 → 2026 全景
```

### 搜索与检索

```bash
# 按关键词搜索
grep -r "强化学习" papers/

# 按标签搜索
grep -r "tags:.*deep-research" papers/

# 使用 Claude Code
# 直接问："知识库里有哪些关于记忆系统的论文？"
```

### 添加新论文

1. 复制模板：`templates/paper.md`
2. 填写到对应目录：`papers/agent/<方向>/`
3. 更新索引：`papers/agent/_index.md`

---

<p align="center">
  <sub>使用 Claude Code 构建 · 持续更新中 · 最后更新: 2026.05</sub>
</p>
