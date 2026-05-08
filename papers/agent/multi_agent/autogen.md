---
title: "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation"
authors: "Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Beibin Li, Erkang Zhu, Li Jiang, Xiaoyun Zhang, Shaokun Zhang, Jiale Liu, Ahmed Hassan Awadallah, Ryen W. White, Doug Burger, Chi Wang"
venue: "COLM 2024"
year: 2024
arxiv: "https://arxiv.org/abs/2308.08155"
code: "https://github.com/microsoft/autogen"
institution: "Microsoft Research, Penn State, University of Washington, Xidian University"
tags: [multi-agent, conversation-programming, framework, code-execution, human-in-the-loop, tool-use, group-chat]
category: agent/multi_agent
date_read: 2026-05-08
---

# AutoGen：通过多智能体对话赋能下一代 LLM 应用

## 摘要

AutoGen 引入了"可对话智能体"抽象和"对话编程"范式，将多智能体 LLM 应用统一为可定制的多轮对话。智能体灵活组合 LLM 推理、人类输入和代码执行。支持双智能体、顺序式、群聊（动态发言者选择）以及嵌套对话模式。在 MATH 上提升约 6%，在 HumanEval 上从 68% 提升到 90%（采用自适应多模型策略，成本降低 18%），并在 GAIA 基准上排名第一。后续发展为 AutoGen 0.4（事件驱动架构）和 Magentic-One 多智能体系统。

## 研究动机与问题

1. **缺乏统一抽象**：每个多智能体应用都需要定制交互逻辑。
2. **流水线刚性**：基于链的框架（LangChain、LlamaIndex）难以适应多轮、多方的对话交互。
3. **人类集成不佳**：缺乏可配置人机协作交互的标准模式。
4. **不安全的代码执行**：LLM 生成的代码执行需要的沙箱处理都是临时方案。
5. **模式刚性**：固定拓扑（双智能体、链式）需要重写才能更改。

核心洞察：**将所有关注点统一到可对话智能体 + 对话编程下，多轮对话本身就是计算基底。**

## 方法

### 可对话智能体抽象

```
ConversableAgent
  |-- llm_config:      {model, temperature, system_message, functions}
  |-- human_input_mode: "ALWAYS" | "TERMINATE" | "NEVER"
  |-- code_execution:   {work_dir, use_docker, timeout}
  |-- function_map:     {name -> callable} (注册的工具)
  |-- max_consecutive_auto_reply: int
  |
  |-- generate_reply():  通过优先级管道生成响应:
  |     1. 自定义注册回复函数
  |     2. 函数/工具调用触发
  |     3. 代码执行触发
  |     4. 带对话历史的 LLM 调用
  |     5. 人类输入检查
  |-- send(message, recipient) / receive(message, sender)
  |-- register_reply(trigger, func, position)

专化类型:
  AssistantAgent  -- LLM 驱动，无代码执行，human_input=NEVER
  UserProxyAgent  -- 代码执行 + 人类代理，human_input=TERMINATE
```

### 对话模式类型

**模式 1：双智能体聊天** -- 最简单的来回对话直到终止：
```
AssistantAgent  <-- 消息/回复 -->  UserProxyAgent
(LLM 推理,                       (代码执行,
 代码生成)                        人类输入)
终止条件: max_turns, "TERMINATE" 关键词, 或人类停止
```

**模式 2：顺序聊天** -- 链式双智能体对话，带上下文传递：
```
Chat_1 (A<->B) --摘要--> Chat_2 (C<->D) --摘要--> Chat_3 (E<->F)
每次聊天的摘要作为上下文注入下一阶段。
```

**模式 3：群聊** -- 多个智能体，由 GroupChatManager 管理：
```
+------------------------------------------------------+
|               GroupChatManager                        |
|  发言者选择策略:                                       |
|    round_robin | random | auto (LLM) | manual | custom|
|------------------------------------------------------|
|  +------+   +------+   +------+   +------+           |
|  |编码者|   |评审者|   |测试者|   |人类  |            |
|  +------+   +------+   +------+   +------+           |
+------------------------------------------------------+

自定义转换图:
  allowed_transitions = {
    planner: [coder, critic],
    coder:   [tester, critic],
    critic:  [coder, planner],
    tester:  [coder, planner]
  }
```

基于 LLM 的选择：管理器用对话上下文和智能体描述提示 LLM，动态选择最合适的下一个发言者。

**模式 4：嵌套对话** -- 父对话中的子对话：
主聊天中的 Agent_B 触发子智能体 1-3 之间的子聊天以解决子问题；结果返回主对话。支持层次化任务分解。

### 人类代理智能体设计

| 模式        | 行为                                   | 使用场景                |
|-------------|----------------------------------------|-------------------------|
| `ALWAYS`    | 每轮提示人类输入                        | 全程监督、协作          |
| `TERMINATE` | 仅在终止时提示人类                      | 审查最终输出            |
| `NEVER`     | 完全自主                               | 批量处理、流水线        |

```
procedure USER_PROXY_RECEIVE(message):
    if message contains code blocks:
        return SANDBOX_EXECUTE(code)       // Docker 或本地子进程
    if message contains function_call:
        return function_map[name](**args)   // 注册工具执行
    switch human_input_mode:
        ALWAYS:    return get_human_input()
        TERMINATE: if is_termination(msg): return get_human_input()
        NEVER:     return auto_reply()
```

### 代码执行沙箱

```
配置: work_dir (隔离目录), use_docker (布尔值/镜像), timeout (秒)
流程: 解析代码块 -> 检测语言 -> Docker/子进程执行 -> 捕获 stdout/stderr
      -> 返回对话 -> 智能体出错时调试 -> 迭代
```

Docker 提供：进程隔离、文件系统沙箱（仅挂载 work_dir）、网络控制、资源限制、每次执行的干净环境。

### 工具使用集成

```python
@assistant.register_for_llm(description="Search the web")
@user_proxy.register_for_execution()
def web_search(query: str) -> str:
    return search_api.search(query)
# AutoGen 自动在 LLM 调用中包含 schema，解析 function_call，
# 路由到执行器，将结果返回对话
```

## 关键创新

1. **可对话智能体**：通过配置统一 LLM、人类、代码执行、工具的单一类型。
2. **对话编程**：工作流作为可编程对话，而非固定流水线。
3. **可配置人机协作**：ALWAYS/TERMINATE/NEVER 频谱作为一等特性。
4. **动态发言者选择**：基于 LLM 或自定义函数的群聊编排。
5. **嵌套对话**：层次化分解，关注点清晰分离。
6. **一等沙箱**：基于 Docker 的代码执行作为框架特性。

## 实验设置与用例

| 用例                    | 模式          | 智能体                        |
|-------------------------|---------------|-------------------------------|
| 数学问题求解            | 双智能体      | MathChat + UserProxy          |
| 检索增强聊天            | 双智能体      | RetrieveAssistant + Proxy     |
| 代码生成与调试          | 双智能体      | Assistant + CodeExecutor      |
| 多智能体编码            | 群聊          | Coder + Critic + Executor     |
| 对话式国际象棋          | 双智能体      | 2 个玩家 + 棋盘验证器         |
| 研究群聊                | 群聊          | 领域专家智能体                |

## 结果

### 数学问题求解（MATH 数据集）

| 方法                          | 准确率   | 代数     |
|-------------------------------|:--------:|:--------:|
| GPT-4（原始，零样本）          | ~50%     | ~65%     |
| Program of Thought (PoT)      | ~52%     | ~63%     |
| GPT-4 + 代码解释器            | 69.7%    | --       |
| **MathChat (AutoGen)**        | **~59%** | **~78%** |
| **AutoGen GPT-4 双智能体**    | **74.8%**| --       |

总体提升 +6%；代数提升 +15%，通过动态切换符号计算（代码执行）和逐步推理实现。

### 编码（HumanEval）

| 策略                           | Pass@1   | 成本对比 GPT-4 |
|--------------------------------|:--------:|:--------------:|
| GPT-4 单模型                   | 68%      | 基线           |
| GPT-4 + GPT-3.5 自适应         | ~82%     | -10%           |
| **AutoGen 多智能体自适应**     | **~90%** | **-18%**       |

自适应策略：简单问题用 GPT-3.5，仅在需要时升级到 GPT-4。

### GAIA 基准测试

AutoGen 多智能体配置在所有三个难度级别上以显著优势达到第一名准确率，展示了在复杂多步骤任务上的有效性。

### 详细数学结果

| 方法                          | MATH (%) | 平均轮次  |
|-------------------------------|:--------:|:---------:|
| GPT-4 (CoT)                   | 57.6     | 1         |
| GPT-4 + 代码解释器            | 69.7     | 1         |
| AutoGen (GPT-3.5, 双智能体)   | 55.2     | 4.1       |
| **AutoGen (GPT-4, 双智能体)** | **74.8** | **3.2**   |

带代码执行的多轮对话实现了迭代优化，通过执行反馈而非单次推理来捕获错误。

## 分析与洞察

1. **对话即计算**：多智能体对话是通用计算基底，不仅仅是独立计算步骤的协调胶水。
2. **灵活性与结构的权衡**：AutoGen（灵活，任意对任意）vs. MetaGPT（刚性，结构化）。更具适应性但需要更精心的编排设计。
3. **代码执行作为锚定**：沙箱提供纯 LLM 推理无法匹配的真实验证——智能体通过计算验证推理。
4. **多模型策略胜过规模扩展**：更智能的编排（GPT-3.5 + GPT-4 自适应）以更低成本超越统一 GPT-4。
5. **发言者选择至关重要**："下一个谁发言"的质量显著影响群聊结果；LLM 选择自然但成本高。

## Magentic-One 系统（Fourney et al., 2024）

基于 AutoGen 构建的生产级多智能体团队：

```
              +------------------+
              |   编排器          |
              | - 任务账本       | (计划、事实、推测)
              | - 进度账本       | (逐步自我反思)
              +--------+---------+
                       | 委派给专业化智能体
          +--------+---+----+---------+
          |        |        |         |
       +------+ +------+ +------+ +--------+
       | 网络 | | 文件 | |编码器| |终端    |
       |浏览器| |浏览器| |      | |        |
       +------+ +------+ +------+ +--------+
```

编排器创建任务账本，按步骤委派，收集结果，通过自我反思维护进度账本。在需要网络浏览 + 文件处理 + 代码执行的基准上达到最先进水平。

## AutoGen 0.4：事件驱动重新设计（2025 年 1 月）

```
v0.2（原版）                  v0.4（重新设计）
同步消息                 -->   异步事件驱动总线
仅 Python              -->   跨语言 (Python + .NET)
进程内智能体            -->   分布式智能体运行时
直接智能体引用          -->   基于主题的消息路由
无状态持久化            -->   内置状态管理
最小可观测性            -->   追踪 + 日志 + 调试
```

集成到 Microsoft Agent Framework 中，与 Semantic Kernel 并列。社区分支（AG2）保持 0.2 API 兼容性以支持向后兼容开发。

## 局限性与批评

1. **对话开销**：没有内置的效率优化来处理无效交流。
2. **发言者选择脆弱性**：基于 LLM 的选择可能使群聊偏离正轨。
3. **调试复杂性**：多智能体故障难以在交错消息中追踪。
4. **无结构化输出**：自由文本通信引入歧义（对比 MetaGPT）。
5. **成本不可预测**：可变的对话长度使单任务成本难以预测。
6. **无持久记忆**：v0.2 没有跨对话的智能体记忆。
7. **群聊扩展性**：5-6 个以上智能体随着历史增长变得缓慢且昂贵。
8. **学习曲线陡峭**：设计决策多但内置指导有限。

## 后续工作

- **Magentic-One**（2024）：基于 AutoGen 的生产级多智能体，采用账本式编排。
- **AutoGen Studio**：用于无代码多智能体工作流构建的可视化界面。
- **AutoGen 0.4**（2025）：Microsoft Agent Framework 中的事件驱动分布式重写。
- **CrewAI**（2024）：简化的多智能体框架，采用基于角色的设计。
- **LangGraph**（2024）：受 AutoGen 对话模式启发的图式编排。
- **Swarm**（OpenAI, 2024）：带交接机制的轻量级多智能体。

## 核心要点

1. **对话即程序**：统一抽象覆盖数学、编码、检索、研究。
2. **可配置自主性**：ALWAYS/TERMINATE/NEVER 人类输入至关重要——不同任务需要不同的监督级别。
3. **全面沙箱化**：一等 Docker 沙箱应成为框架要求。
4. **自适应编排有效**：多模型策略以 -18% 的成本实现 90%（对比 68%）。
5. **灵活性需要纪律**：开放的对话编程需要精心设计终止条件、角色定义和模式选择。
6. **框架必须演进**：v0.2->v0.4 的迁移表明生产级智能体需要与研究原型根本不同的（事件驱动、分布式）架构。
