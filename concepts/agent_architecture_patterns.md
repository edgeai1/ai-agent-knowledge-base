---
title: 智能体架构模式
tags: [agent, architecture, react, plan-and-execute, reflexion, multi-agent, design-pattern]
related: [llm_agent_fundamentals, agent_techniques, agent_frameworks]
---

## 定义

智能体架构模式是定义 LLM 智能体如何组织其推理、规划、行动和反馈循环的重复性结构设计。每种模式代表了自主性、准确性、效率和复杂性之间的不同权衡。

---

## 模式 1：ReAct（推理 + 行动）

### 起源
Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (ICLR 2023), arXiv:2210.03629

### 核心洞察
在单个提示/循环中交替进行思维链推理和具体行动，而非将推理和行动分离为独立阶段。推理轨迹帮助模型进行规划和解释结果；行动则将推理建立在真实观察的基础上。

### 工作原理

```
Loop:
  1. Thought: 智能体对当前状态和下一步行动进行推理
  2. Action: 智能体选择并执行一个工具/行动
  3. Observation: 行动的结果返回给智能体
  重复直到智能体决定结束
```

### 示例轨迹
```
Question: What is the elevation of the birthplace of the inventor of the telephone?

Thought 1: I need to find who invented the telephone, then find their birthplace,
            then find the elevation of that place.
Action 1: Search("inventor of the telephone")
Observation 1: Alexander Graham Bell is credited with inventing the telephone.

Thought 2: Alexander Graham Bell was the inventor. Now I need to find his birthplace.
Action 2: Search("Alexander Graham Bell birthplace")
Observation 2: Alexander Graham Bell was born in Edinburgh, Scotland.

Thought 3: His birthplace is Edinburgh. Now I need the elevation.
Action 3: Search("Edinburgh Scotland elevation")
Observation 3: Edinburgh has an elevation of approximately 47 meters (154 ft).

Thought 4: I now have all the information needed.
Action 4: Finish("The elevation of Edinburgh, the birthplace of Alexander Graham Bell,
          is approximately 47 meters (154 ft).")
```

### 优势
- 简单直观
- 推理轨迹提高了可解释性
- 基于真实数据的推理减少了幻觉（行动提供真实数据）
- 适用于问答、事实核查、交互式任务

### 局限性
- 顺序执行（一次一个行动）可能较慢
- 如果推理质量差，可能陷入循环
- 没有显式计划——是反应式的而非主动式的
- 长轨迹会快速填满上下文窗口

### 变体
- **带自一致性的 ReAct**：运行多个 ReAct 轨迹，取多数投票的答案
- **ReWOO（无观察推理）**：先规划所有行动，批量执行，再对所有观察进行推理（Xu et al., 2023）
- **LATS（语言智能体树搜索）**：将 ReAct 与蒙特卡洛树搜索结合用于探索

---

## 模式 2：规划与执行

### 起源
受经典 AI 规划启发；由 Wang et al., "Plan-and-Solve Prompting" (2023) 为 LLM 形式化，并在 LangGraph 的 Plan-and-Execute 智能体等框架中实现。

### 核心洞察
将规划与执行分离。首先生成完整的计划（步骤列表），然后按顺序执行每个步骤，可选择在每步之后根据结果重新规划。

### 工作原理

```
Phase 1 -- 规划：
  输入：用户目标
  输出：有序步骤列表 [Step1, Step2, ..., StepN]

Phase 2 -- 执行：
  对计划中的每个步骤：
    执行步骤（可能涉及 ReAct 风格的子循环）
    观察结果
    可选：根据新信息重新规划剩余步骤

Phase 3 -- 综合：
  将所有步骤的结果组合为最终答案
```

### 架构图
```
                    +-------------+
  用户目标 -------->|   规划器    |-----> [Step1, Step2, Step3, ...]
                    |   (LLM)     |              |
                    +-------------+              |
                          ^                      v
                          |              +---------------+
                   需要时重新规划        |   执行器      |
                          |              | (LLM + 工具)  |
                          ^              +---------------+
                          |                      |
                    +-------------+              v
                    |  重规划器   |<----- 观察/结果
                    +-------------+
```

### 优势
- 更适合复杂的多步骤任务
- 显式计划可检查和修改（对人在回路中友好）
- 规划器和执行器可以使用不同的模型（例如强模型规划，弱模型执行）
- 可以并行化独立步骤
- 重新规划允许适应意外结果

### 局限性
- 在不知道执行结果的情况下，初始计划可能不够优化
- 对简单任务，规划步骤带来额外开销
- 重新规划增加延迟和成本
- 质量高度依赖于规划器的任务分解能力

### 变体
- **静态规划与执行**：规划一次，执行所有步骤不再重新规划
- **动态规划与执行**：每步之后都重新规划
- **分层规划与执行**：计划包含子计划（递归分解）
- **规划与求解（零样本）**：在提示中添加"让我们先理解问题并制定计划"

---

## 模式 3：Reflexion / 自我反思

### 起源
Shinn et al., "Reflexion: Language Agents with Verbal Reinforcement Learning" (NeurIPS 2023), arXiv:2303.11366

### 核心洞察
在完成任务（或失败）后，让智能体反思其表现，生成关于哪里出错的语言反馈，并将该反思作为下一次尝试的额外上下文。这创建了一种无需更新权重的"语言强化学习"形式。

### 工作原理

```
Outer Loop (试验):
  For each trial t = 1, 2, ..., T:
    
    Inner Loop (回合):
      运行智能体（例如使用 ReAct）尝试完成任务
      接收评估信号（成功/失败、分数、测试结果）
    
    反思:
      如果任务失败或不够优化：
        生成语言反思："哪里出了问题？我应该有什么不同的做法？"
        将反思存储到记忆中
    
    下一次试验使用：原始任务 + 所有先前的反思
    
    如果任务成功：返回结果
```

### 示例
```
Trial 1:
  任务：编写一个查找最长回文子串的函数
  智能体编写代码 -> 测试失败（"babad" 输出错误）
  
  反思："我检查所有子串的方法是正确的，但循环边界有一个差一错误。
  我也没有处理多个相同长度回文存在的情况。下次我应该更仔细地
  测试边界情况，并使用中心扩展法。"

Trial 2:
  任务：[相同] + Trial 1 的反思
  智能体编写改进的代码 -> 测试通过
```

### 记忆组件
- **轨迹记忆**：每次试验的行动/观察序列
- **反思记忆**：语言自我批评和经验教训
- **评估信号**：二元（通过/失败）或标量（测试通过率、分数）

### 优势
- 无需微调即可从错误中学习
- 反思是可解释的（我们可以阅读智能体学到了什么）
- 与任何基础智能体架构兼容（ReAct、规划与执行等）
- 对编程任务、决策制定和推理有效

### 局限性
- 需要多次试验（更慢且更昂贵）
- 需要明确的评估信号（对开放式任务较难）
- 受限于 LLM 准确自我诊断错误的能力
- 反思可能是浅层的或错误的（垃圾进垃圾出）

### 相关模式
- **Self-Refine**（Madaan et al., 2023）：使用自我反馈迭代优化输出
- **CRITIC**（Gou et al., 2023）：使用工具验证和批评 LLM 输出
- **内省型智能体**：维护对自身能力的显式信念的智能体

---

## 模式 4：多智能体协作

### 起源
多项工作：CAMEL (Li et al., 2023)、AutoGen (Wu et al., 2023)、MetaGPT (Hong et al., 2023)、ChatDev (Qian et al., 2023)、Generative Agents (Park et al., 2023)

### 核心洞察
不使用单一的整体式智能体，而是使用多个专业化智能体，通过交流、辩论或协作来解决复杂任务。每个智能体可以有不同的角色、专业知识、提示、工具，甚至不同的底层模型。

### 协作拓扑

#### 4a. 顺序流水线（链式）
```
Agent A -> Agent B -> Agent C -> 最终输出
（研究）    （撰写）    （审查）
```
- 每个智能体处理一个阶段
- 一个智能体的输出是下一个的输入
- 简单、可预测、易于调试

#### 4b. 层级式（管理者-工作者）
```
        +----------+
        |  管理者   |
        |  智能体   |
        +----+-----+
             |
    +--------+--------+
    |        |        |
+---v--+ +---v--+ +---v--+
|工作者| |工作者| |工作者|
|  A   | |  B   | |  C   |
+------+ +------+ +------+
```
- 管理者分解任务并委派给工作者
- 工作者将结果报告给管理者
- 管理者综合和协调
- 示例：MetaGPT 的软件公司模拟、CrewAI 的团队结构

#### 4c. 辩论 / 对抗式
```
Agent A <-----> Agent B
（提议者）     （批评者）
     \          /
      v        v
   +------------+
   |   裁判     |
   +------------+
```
- 智能体就不同立场进行争论或互相批评
- 由裁判智能体或机制选择最佳结果
- 通过多样化视角提高准确性
- 示例："心智社会"方法、LLM 辩论用于事实性验证

#### 4d. 对等协作（扁平式）
```
Agent A <---> Agent B
  ^    \   /    ^
  |     \ /     |
  v      X      v
Agent C <---> Agent D
```
- 所有智能体都是对等的，自由交流
- 共享消息总线或黑板架构
- 最灵活但最难协调
- 示例：Generative Agents（Stanford 虚拟小镇）

#### 4e. 投票 / 集成
```
Agent A ---\
Agent B ----+---> 聚合 --> 最终答案
Agent C ---/
```
- 多个智能体独立解决同一问题
- 结果进行聚合（多数投票、最佳N选一等）
- 通过冗余提高可靠性

### 通信协议
- **自然语言消息**：最常见；智能体通过文本交流
- **结构化消息**：JSON、函数调用、类型化接口
- **共享记忆/黑板**：智能体读写共享知识库
- **事件驱动**：智能体订阅事件并异步响应

### 多智能体设计模式（来自实践）

| 模式 | 描述 | 应用场景 |
|------|------|----------|
| 监督者 | 一个智能体将任务路由给专家 | 客户服务、分诊 |
| 群体 | 智能体根据上下文传递控制权 | 具有分支的复杂工作流 |
| Map-Reduce | 并行处理 + 聚合 | 文档分析、研究 |
| 反思对 | 生成器 + 批评者/审查者 | 代码生成、写作 |
| 小组讨论 | 多个专家讨论 | 复杂分析、决策制定 |

### 优势
- 专业化：每个智能体可以针对其角色进行优化
- 可扩展性：添加更多智能体以获得更多能力
- 鲁棒性：一个智能体的失败不会导致系统崩溃
- 涌现行为：简单智能体产生复杂结果
- 自然映射到组织结构

### 局限性
- 通信开销（token 成本、延迟）
- 协调复杂性（死锁、无限循环、冲突行动）
- 更难调试和测试
- 可能出现回音室或群体思维
- 更昂贵（多次 LLM 调用）

---

## 模式比较矩阵

| 模式 | 复杂度 | 最适用于 | 延迟 | 成本 | 可解释性 |
|------|--------|----------|------|------|----------|
| ReAct | 低 | 简单工具使用任务、问答 | 低-中 | 低 | 高 |
| 规划与执行 | 中 | 多步骤任务、复杂工作流 | 中 | 中 | 高 |
| Reflexion | 中 | 具有明确成功标准的任务 | 高 | 高 | 非常高 |
| 多智能体 | 高 | 复杂项目、专业领域 | 高 | 高 | 中 |

## 参考文献

- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2023), ICLR
- Wang et al., "Plan-and-Solve Prompting" (2023), ACL
- Shinn et al., "Reflexion: Language Agents with Verbal Reinforcement Learning" (2023), NeurIPS
- Wu et al., "AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation" (2023)
- Hong et al., "MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework" (2023)
- Li et al., "CAMEL: Communicative Agents for 'Mind' Exploration of Large Language Model Society" (2023)
- Park et al., "Generative Agents: Interactive Simulacra of Human Behavior" (2023)
- Xu et al., "ReWOO: Decoupling Reasoning from Observations for Efficient Augmented Language Models" (2023)
- Zhou et al., "Language Agent Tree Search Unifies Reasoning, Acting, and Planning in Language Models" (2023)
- Madaan et al., "Self-Refine: Iterative Refinement with Self-Feedback" (2023)
