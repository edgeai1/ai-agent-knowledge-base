---
title: LLM 智能体基础
tags: [agent, llm, architecture, planning, memory, tool-use, survey]
related: [agent_architecture_patterns, agent_memory_systems, agent_frameworks, agent_techniques]
---

## 定义

基于 LLM 的智能体是一种自主或半自主系统，使用大型语言模型作为其核心"大脑"或控制器来感知环境、推理任务、做出决策并采取行动以实现目标。与普通的 LLM 使用（单次提示输入、响应输出）不同，智能体以循环方式运行——迭代经历观察、推理、行动和反馈的循环。

经典定义（来自复旦大学综述，Xi et al. 2023）将智能体定义为：

**智能体 = LLM + 记忆 + 规划 + 工具使用 + 行动**

## 直觉理解

关键洞察是，尽管 LLM 是作为下一个 token 预测器训练的，但它们展现出涌现能力（推理、指令遵循、上下文学习），这些能力可以被利用作为通用控制器。通过将 LLM 包装在一个带有外部工具、记忆和结构化提示策略访问的循环中，我们可以创建解决复杂多步骤问题的系统，而单次 LLM 调用无法处理这些问题。

智能体范式将 LLM 从"问题回答者"转变为"问题解决者"，赋予它：
1. 分解问题的能力（规划）
2. 跨步骤记忆上下文的能力（记忆）
3. 与外部系统交互的能力（工具/行动）
4. 评估自身进展的能力（反馈/反思）

## 核心组件

### 1. 大脑 / 控制器（LLM）

LLM 作为中央决策引擎。它：
- 解释用户目标和指令
- 生成推理轨迹
- 决定采取哪些行动
- 将观察综合为连贯的响应

智能体利用的关键 LLM 能力：
- **上下文学习**：根据提示中的示例调整行为
- **指令遵循**：执行系统提示中描述的结构化工作流
- **推理**：思维链、逐步问题分解
- **代码生成**：编写可执行代码作为行动
- **自然语言理解**：解析工具输出和用户反馈

### 2. 规划

将复杂任务分解为可管理的子任务并确定执行顺序的能力。

**任务分解策略：**
- **思维链（CoT）**：顺序逐步推理（Wei et al., 2022）
- **思维树（ToT）**：分支探索多条推理路径（Yao et al., 2023）
- **思维图（GoT）**：DAG 结构推理，支持合并和优化
- **LLM+P**：使用经典规划器（如 PDDL），LLM 作为问题转换器
- **分层规划**：高层计划递归分解为子计划

**子目标规划：**
- 将目标分解为有序子目标
- 每个子目标可能需要不同的工具或方法
- 计划可以是静态的（一次生成）或动态的（每步之后修订）

### 3. 记忆

使智能体能够维护上下文、从经验中学习并访问相关知识。

**短期记忆（工作记忆）：**
- LLM 的上下文窗口本身
- 存储当前对话、近期观察和即时计划
- 受上下文长度限制（4K 到 1M+ tokens，取决于模型）
- 通过摘要化、滑动窗口或压缩策略管理

**长期记忆：**
- 跨会话持久化，突破上下文窗口限制
- 通常通过外部向量数据库实现（Pinecone、Weaviate、ChromaDB、FAISS）
- 信息被嵌入并通过语义相似性搜索检索
- 存储：过去的交互、学到的事实、用户偏好、成功的策略

**情景记忆：**
- 记录特定的过去经验和交互序列
- 使智能体能够回忆在类似过去情境中"发生了什么"
- 用于 Reflexion 和经验回放模式
- 示例：记住特定 API 调用因特定错误而失败

**语义记忆：**
- 通用知识和事实（不与特定情景绑定）
- 可以预加载（知识库）或随时间积累
- 通常实现为结构化知识图谱或文档存储

**程序性记忆：**
- 存储"如何做"的知识：学到的行动序列、工具使用模式
- 可实现为代码库、提示模板或微调的权重

**基于 RAG 的记忆（检索增强生成）：**
- 混合方法：外部知识库 + 检索机制 + LLM 生成
- 文档被分块、嵌入并存储在向量数据库中
- 查询时，检索相关分块并注入提示
- 使智能体能够处理超出上下文限制的大型知识库
- 关键组件：分块策略、嵌入模型、检索算法、重排序

### 4. 工具使用（函数调用）

调用外部工具、API 和函数以扩展智能体能力。

**机制：**
- LLM 生成结构化输出（JSON、函数调用），指定调用哪个工具及其参数
- 框架执行工具并将结果返回给 LLM
- LLM 将结果纳入其推理

**常见工具类别：**
- **搜索与检索**：网络搜索、数据库查询、知识库查找
- **代码执行**：Python 解释器、shell 命令、沙箱环境
- **API 调用**：REST/GraphQL API、第三方服务
- **文件操作**：读取、写入、编辑文件
- **通信**：电子邮件、Slack、消息 API
- **计算**：计算器、数据分析、科学计算
- **浏览器/网页**：导航页面、填写表单、提取内容

**函数调用协议：**
- OpenAI 函数调用 / 工具使用格式
- Anthropic 工具使用格式
- 开源：Gorilla、ToolBench、API-Bank 格式
- MCP（模型上下文协议）——Anthropic 的标准化工具集成协议

### 5. 行动执行

智能体影响其环境的能力。

**行动类型：**
- **内部行动**：推理、规划、更新记忆（无外部副作用）
- **外部行动**：工具调用、代码执行、API 请求（影响环境）
- **通信行动**：响应用户、提出澄清问题

### 6. 观察 / 反馈循环

智能体感知其行动结果和环境状态的机制。

**智能体循环：**
```
用户目标 -> [规划] -> [行动] -> [观察] -> [推理] -> [行动] -> ... -> [最终答案]
```

**观察来源：**
- 工具/函数返回值
- 代码执行输出（stdout、stderr、返回值）
- 环境状态变化
- 用户反馈和纠正
- 自我评估（LLM 批评自身输出）

**反馈整合：**
- 观察被附加到智能体的上下文中
- 智能体推理观察是否满足当前子目标
- 如果不满足，智能体可以：重试、尝试不同方法、请求帮助或修订计划

## 公式化

**通用智能体循环（伪代码）：**

```python
def agent_loop(goal, tools, max_steps):
    memory = initialize_memory()
    plan = llm.plan(goal)
    
    for step in range(max_steps):
        # Reason about current state
        thought = llm.reason(goal, plan, memory)
        
        # Decide on action
        action = llm.select_action(thought, tools)
        
        if action.type == "finish":
            return action.result
        
        # Execute action
        observation = execute(action, tools)
        
        # Update memory
        memory.add(thought, action, observation)
        
        # Optionally revise plan
        if should_replan(observation, plan):
            plan = llm.replan(goal, memory)
    
    return "Max steps reached"
```

## 变体

- **单智能体系统**：一个 LLM 控制器处理所有事务
- **多智能体系统**：多个专业化智能体协作
- **人在回路中的智能体**：为关键行动请求人类批准的智能体
- **自主智能体**：几乎不需要人类监督的完全自主智能体（如 AutoGPT、Devin）
- **对话智能体**：嵌入聊天界面中具有交互式反馈的智能体

## 参考文献

- Xi et al., "The Rise and Potential of Large Language Model Based Agents: A Survey" (2023), arXiv:2309.07864 -- 复旦大学全面综述
- Wang et al., "A Survey on Large Language Model based Autonomous Agents" (2023), arXiv:2308.11432 -- 中国人民大学综述
- Weng, Lilian, "LLM Powered Autonomous Agents" (2023), lilianweng.github.io -- 具有影响力的博客文章
- Sumers et al., "Cognitive Architectures for Language Agents" (CoALA, 2023), arXiv:2309.02427
- Yao et al., "ReAct: Synergizing Reasoning and Acting in Language Models" (2022), arXiv:2210.03629
- Shinn et al., "Reflexion: Language Agents with Verbal Reinforcement Learning" (2023), arXiv:2303.11366
