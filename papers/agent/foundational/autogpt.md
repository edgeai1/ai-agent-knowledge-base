---
title: "Auto-GPT: An Autonomous GPT-4 Experiment (Auto-GPT：一个自主 GPT-4 实验)"
authors: "Toran Bruce Richards (original); Yang et al. 2023 (benchmarks); Wang et al. 2023 (survey)"
venue: "Open-source (GitHub); Related: arXiv:2306.02224, arXiv:2308.11432"
year: 2023
github: "https://github.com/Significant-Gravitas/AutoGPT"
related_papers:
  - title: "Auto-GPT for Online Decision Making: Benchmarks and Additional Opinions"
    url: "https://arxiv.org/abs/2306.02224"
  - title: "A Survey on Large Language Model based Autonomous Agents"
    url: "https://arxiv.org/abs/2308.11432"
institution: "Significant Gravitas (community project)"
tags: [autonomous-agent, self-prompting, tool-use, memory, GPT-4, open-source, foundational]
status: done
date_reviewed: 2026-05-08
---

# Auto-GPT：一个自主 GPT-4 实验

## 简述

Auto-GPT（2023 年 3 月 30 日发布）是首个被广泛采用的自主大语言模型智能体，
展示了 GPT-4 可以被赋予目标、工具和自提示循环，在无需人工干预的情况下独立
运行。尽管存在严重的实际局限（无限循环、高成本、频繁失败、浅层推理），它
成为了历史上增长最快的 GitHub 仓库（数周内超过 100K 星标），并催化了整个
AI 智能体生态系统。没有单独的学术论文定义 Auto-GPT；本笔记综合了开源代码库、
社区分析和相关学术评估。

## 动机与问题

到 2023 年初，GPT-4 展示了卓越的推理和指令跟随能力，但仍然根本上是被动的——
仅响应人类提供的提示。Auto-GPT 提出了一个激进的问题：

**如果给 GPT-4 一个目标、一组工具，并让它在一个循环中自我提示，
直到目标实现，会发生什么？**

这不是一篇研究论文，而是一个工程实验——一个包装 OpenAI API 的 Python 脚本，
带有网络搜索、文件 I/O、代码执行和自提示循环。其意义在于展示了"智能体循环"
模式（感知-思考-行动-观察）仅通过提示工程就能实现。

## 方法

### 自主循环架构

```
初始化: 用户提供智能体名称、角色、最多 5 个目标
循环 (直到 task_complete 或用户干预):
  1. 构建提示: 系统消息 + 命令 + 记忆 + 历史 + 上次结果
  2. GPT-4 推理: 返回 JSON {thoughts: {text, reasoning, plan,
     criticism, speak}, command: {name, args}}
  3. 执行命令: 分派到工具处理器，捕获输出
  4. 更新记忆: 追加到短期记忆，嵌入到向量数据库，满时进行摘要
  -> 回到步骤 1
```

### 自提示机制（核心创新）

每次 GPT-4 响应必须遵循结构化 JSON 格式，强制在行动前进行推理：

```json
{
  "thoughts": {
    "text": "I need to research competitor pricing.",
    "reasoning": "Sub-goal 2 requires pricing data I haven't gathered yet.",
    "plan": ["- Search pricing data", "- Build comparison table", "- Draft recommendation"],
    "criticism": "I should verify from multiple sources, not just one search.",
    "speak": "Searching for competitor pricing now."
  },
  "command": {"name": "google_search", "args": {"query": "competitor SaaS pricing 2023"}}
}
```

- **`reasoning`**：强制显式说明行动与目标的关联
- **`plan`**：在迭代间维持多步连续性
- **`criticism`**：内置自反思，先于 Reflexion/LATS
- **`speak`**：面向用户的摘要，供可选的人类监控

### 系统提示结构

```
You are <agent_name>, <role_description>.
GOALS: [1-5 user-defined objectives]
CONSTRAINTS: ~4000 word memory limit, no user assistance, use listed commands only
COMMANDS: [numbered list with arg schemas]
RESPONSE FORMAT: [JSON schema above]
PERFORMANCE EVALUATION: Continuously self-criticize; every command has a cost.
```

### 记忆系统：双重架构

**短期记忆**：上下文窗口中的最后 N 条消息（GPT-4 发布时为 8K tokens）。
满时，较旧的消息通过单独的大语言模型调用进行摘要，丢失细节。

**长期记忆**：动作-结果对通过 OpenAI API 嵌入并存储在向量数据库
（Pinecone/Weaviate/Milvus/Redis）中。每次迭代时，当前上下文通过
余弦相似度查询存储以检索相关的过去经验。

**关键演变**：到 2023 年末，AutoGPT **完全移除了向量数据库支持**——
典型运行不会产生足够多的不同事实来证明索引开销的合理性。默认变为
使用简单的 JSON 文件进行记忆存储。

### 工具/命令集

按类别组织的核心命令：
- **信息**：google_search、browse_website
- **文件 I/O**：write_to_file、read_file、append_to_file、delete_file、list_files
- **代码**：execute_python、execute_shell、clone_repository
- **生成**：generate_image (DALL-E)
- **记忆**：memory_add、memory_search
- **控制**：do_nothing（跳过周期）、task_complete（标志完成）
- **通信**：send_tweet

可通过 `@command` 装饰器扩展社区插件（email、Slack、DB 等）。

## 关键创新

1. **自提示循环**：首个被广泛知晓的持续大语言模型自提示系统。
2. **结构化自反思**：强制的 `criticism`/`plan` 字段先于 Reflexion/LATS。
3. **联网自主**：当时最广泛的工具集（网络、文件、代码、API）。
4. **迭代目标分解**：计划在自提示中维护和更新。
5. **民主化**：使"AI 智能体"对数百万人变得具体，引发了一个生态系统。

## 实验设置（社区与学术基准）

### Auto-GPT for Online Decision Making (Yang et al., 2023, arXiv:2306.02224)

对 Auto-GPT 风格智能体最严格的学术评估：
- **基准测试**：WebShop（在线购物）、ALFWorld（家庭任务）
- **测试模型**：GPT-4、GPT-3.5、Claude、Vicuna
- **关键贡献**："Additional Opinions"算法——将基于监督/模仿的学习器作为
  顾问与大语言模型智能体一起使用，提供轻量级的专家指导而无需微调基础模型

### AgentBench (Liu et al., 2023)

跨 8 个环境的综合基准：网页浏览、在线购物、数据库操作、家庭任务、
数字卡牌游戏、横向思维、知识图谱、操作系统任务。GPT-4 显著优于
所有其他模型。

### Auto-GPT-Benchmarks（社区）

项目自身的基准套件，包含信息检索、代码生成、文件管理和多步规划任务。

## 结果

### 决策基准（Yang et al., 2023）

| 模型/配置                  | WebShop     | ALFWorld    |
|---------------------------|-------------|-------------|
| GPT-4（Auto-GPT 风格）    | 最高        | 最高        |
| GPT-3.5（Auto-GPT 风格）  | 中等        | 中等        |
| Claude（Auto-GPT 风格）   | 中等        | 中等        |
| Vicuna（Auto-GPT 风格）   | 低          | 低          |
| + Additional Opinions      | 所有模型均显著提升 |

"Additional Opinions"方法表明，将大语言模型智能体与轻量级监督学习器
混合使用，无需大语言模型微调即可显著提升性能。

### 实际性能（社区观察）

| 指标                    | 典型观察                            |
|------------------------|-------------------------------------|
| 目标完成率              | 复杂多步任务约 10-30%              |
| 平均循环迭代次数        | 完成或失败前 10-50+ 次             |
| 每次运行成本（GPT-4）   | API 调用 $1-50+                    |
| 无限循环频率            | 高——最常见的失败模式              |
| Token 消耗              | 比单次提示方法多 15-50 倍          |

### 已知失败模式

1. **无限循环**（最常见）：智能体重复动作或在两个状态间摇摆。原因包括
   模板化批评、无循环检测、模糊的完成标准。
2. **成本爆炸**：每次迭代=带大提示的完整 GPT-4 调用。按原始定价
   （输入 $0.03/1K、输出 $0.06/1K），运行花费 $10-50+ 但成果有限。
3. **上下文溢出**：8K tokens 在 5-10 次迭代内填满。摘要丢失细节，
   导致重复动作（"忘记"已经尝试过什么）。
4. **幻觉动作**：编造不存在的文件、虚构搜索结果。
5. **浅层探索**：坚持第一个看似合理的方法；无真正的回溯。
6. **目标漂移**：在多次迭代中逐渐追求偏离的子目标。

## 分析与洞察

### 为什么 Auto-GPT 在历史上如此重要

GitHub 星标增长速度超过任何此前的仓库。重要性在于展示而非性能：

1. **概念验证**：仅通过提示工程即可实现自主智能体。
2. **生态系统催化剂**：启发了 BabyAGI、AgentGPT、MetaGPT、CrewAI 等
   数十个项目。几个月内，"AI 智能体"成为主导的应用大语言模型范式。
3. **揭示了难题**：通过公开失败，确定了研究议程——可靠规划、记忆、
   成本控制、错误恢复、循环检测、目标跟踪。
4. **改变了 Overton 窗口**：智能体从学术好奇心变成了有风投资金和
   企业兴趣的主流工程追求。

### 架构分析（Wang et al., 2023 综述）

Wang et al. 将 Auto-GPT 放入三模块框架：**配置**（静态角色/目标）->
**记忆**（上下文+向量数据库，后来被移除）-> **行动**（固定命令，无学习）。
与后来的智能体系统相比，所有三个模块都非常简陋。

### 与同期系统的比较

| 特性              | Auto-GPT     | BabyAGI      | AgentGPT     | LangChain     |
|-------------------|--------------|--------------|--------------|---------------|
| 架构              | 完整循环     | 任务队列     | 完整循环     | 可配置        |
| 记忆系统          | 向量数据库   | 内存         | 内存         | 多种          |
| 规划              | 自提示       | 任务分解     | 自提示       | 链/智能体     |
| 网络访问          | 是           | 否           | 是           | 可插拔        |
| 代码执行          | 是           | 否           | 有限         | 是            |
| 人类监督          | 可选         | 可选         | 浏览器内     | 可配置        |
| 学习              | 无           | 无           | 无           | 无            |
| 主要受众          | 开发者       | 开发者       | 非技术人员   | 框架          |

## 局限性与批评

1. **可靠性**：对生产环境来说太低——频繁失败、循环、不正确的结果。
2. **成本低效**：比定向提示多 15-50 倍 token 消耗；无结果保证。
3. **无学习**：每次运行从头开始；无跨会话迁移。
4. **浅层自评估**：`criticism` 退化为泛泛的陈词滥调，非真正的纠正。
5. **安全性**：互联网+代码执行+最小沙箱构成真正风险。
6. **提示脆弱性**：目标措辞的微小变化导致截然不同的行为。
7. **无原则性的规划**：涌现的规划，无完整性/终止保证。
8. **评估困难**：发布时不存在标准化基准。

## 后续工作

- **BabyAGI** (Nakajima, 2023)：简化的智能体，带大语言模型驱动的任务队列优先级。
- **AgentGPT** (Reworkd, 2023)：面向非技术用户的网页版 Auto-GPT。
- **MetaGPT** (Hong et al., 2023)：模拟软件公司角色的多智能体框架。
- **Reflexion** (Shinn et al., 2023)：结构化自反思，带持久失败记忆。
- **LATS** (Zhou et al., 2023)：树搜索+智能体行动，实现系统性探索。
- **AutoGen** (Wu et al., 2023)：多智能体对话框架，带更好的监督。
- **CrewAI** (Moura, 2024)：基于角色的多智能体编排。
- **Devin** (Cognition, 2024)：带沙箱执行的软件工程智能体。
- **Claude Code, Codex CLI** (2025)：生产级智能体，可靠性改进。

## 核心要点

1. **自主循环模式原则上可行**：Auto-GPT 证明了大语言模型可以自提示、
   使用工具、并在无人干预的情况下朝目标推进，建立了所有后续系统
   使用的核心智能体架构。

2. **可靠性是根本障碍**："令人印象深刻的演示"与"可靠系统"之间的差距
   定义了随后整个智能体研究议程。2023 年以来的每个重大智能体进展
   都在解决 Auto-GPT 暴露的一个失败模式。

3. **记忆管理比看起来更难**：团队移除向量数据库支持的决定揭示，
   朴素的检索增强记忆不能解决智能体记忆问题——智能体记忆的正确
   抽象仍然是一个开放问题。

4. **自反思需要结构**：自由形式的 `criticism` 是不够的。后续工作
   （Reflexion、LATS）表明，带有显式失败分析和持久记忆的结构化
   反思要有效得多。

5. **Auto-GPT 最大的贡献是文化性的**：它使"AI 智能体"成为一个主流
   概念，吸引了大量社区关注，并设定了 2023-2025 年的研究议程——
   如何使自主智能体真正可靠？
