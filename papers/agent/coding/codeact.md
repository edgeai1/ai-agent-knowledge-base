---
title: "Executable Code Actions Elicit Better LLM Agents"
authors: Xingyao Wang, Yangyi Chen, Lifan Yuan, Yizhe Zhang, Yunzhu Li, Hao Peng, Heng Ji
venue: ICML 2024
year: 2024
url: https://arxiv.org/abs/2402.01030
code: https://github.com/xingyaoww/code-act
tags: [coding, action-space, code-generation, agent-design, openhands]
status: done
---

## 摘要

提出 CodeAct，一种使用可执行 Python 代码作为 LLM 智能体统一动作空间的方法，在 17 个 LLM 上相比 JSON 或文本动作格式实现高达 20% 的成功率提升。

## 研究动机与问题

现有 LLM 智能体框架要求模型以受限格式生成动作：
- **JSON 动作**：`{"action": "search", "query": "..."}` —— 仅限于预定义的动作 schema
- **文本/自然语言动作**：`search for "..."` —— 解析模糊，无组合性
- **结构化格式**：ReAct 风格 `Action: tool_name[args]` —— 不灵活，每轮一个工具

这些方法共享根本性局限：
1. **受限的动作空间**：智能体只能调用预注册的工具，无法组合新操作
2. **无状态持久性**：每个动作无状态；中间结果无法存储或操作
3. **组合性有限**：链接多个工具调用需要多轮，增加延迟和错误风险
4. **无自我调试**：当动作失败时，智能体没有自省机制

关键洞察：LLM 已在大量代码语料上训练。Python 代码是 LLM 已经理解的自然、表达性强、可组合的动作语言。

## 方法

### CodeAct 动作格式

在 CodeAct 中，智能体每轮的动作是一个可执行的 Python 代码块。代码在持久的 IPython 解释器中运行，执行的 stdout/stderr 作为观察返回。

```
智能体轮次格式:
+--------------------------------------------+
| Thought: 我需要找到匹配模式的文件          |
|          并计数它们。                       |
|                                            |
| Action:                                    |
| ```python                                  |
| import os                                  |
| files = [f for f in os.listdir('.')        |
|          if f.endswith('.py')]              |
| print(f"Found {len(files)} Python files")  |
| for f in sorted(files)[:10]:               |
|     print(f"  - {f}")                      |
| ```                                        |
+--------------------------------------------+
            |
            v
+--------------------------------------------+
| 观察（来自解释器）:                         |
| Found 15 Python files                      |
|   - agent.py                               |
|   - config.py                              |
|   - ...                                    |
+--------------------------------------------+
```

### 为什么代码优于 JSON/文本

```
+------------------+--------+---------+-----------+
| 能力             | JSON   | 文本    | CodeAct   |
+------------------+--------+---------+-----------+
| 组合性           | 低     | 低      | 高        |
| 状态持久性       | 无     | 无      | 完整      |
| 自我调试         | 无     | 无      | 原生      |
| 工具组合         | 1/轮   | 1/轮    | N/轮      |
| 控制流           | 无     | 无      | 完整      |
| 数据操作         | 无     | 无      | 完整      |
| 新操作           | 否     | 否      | 是        |
| 图灵完备         | 否     | 否      | 是        |
+------------------+--------+---------+-----------+
```

具体优势：
1. **单轮多工具组合**：`result = tool_a(tool_b(x))` vs. 两轮分别调用
2. **变量持久性**：通过 Python 变量在跨轮中存储中间结果
3. **控制流**：在单个动作中使用循环、条件、异常处理
4. **自我调试**：`try/except` 块，错误检查，动态修正
5. **库访问**：导入任何 Python 库（pandas、requests 等）进行复杂操作
6. **数据转换**：使用原生 Python 处理、过滤和聚合工具输出

### 多轮交互协议

```
System Prompt: "You are a helpful assistant. You can use Python code
               to interact with the environment..."

轮次 1（智能体）:
  Thought: 让我搜索相关信息。
  Action: ```python
  result = search("climate change effects")
  print(result[:500])
  ```

轮次 1（环境）:
  Observation: [搜索结果...]

轮次 2（智能体）:
  Thought: 搜索返回了错误。让我修复查询。
  Action: ```python
  try:
      result = search("climate change effects 2024")
      filtered = [r for r in result if r['relevance'] > 0.8]
      print(f"Found {len(filtered)} relevant results")
  except Exception as e:
      print(f"Error: {e}")
      # 回退策略
      result = web_browse("en.wikipedia.org/wiki/Climate_change")
  ```
```

### 动作格式比较

**JSON 格式**（例如 OpenAI function calling）：
```json
{"name": "search", "arguments": {"query": "climate change", "limit": 5}}
```
局限：每轮一个工具，无后处理，无错误处理。

**文本格式**（例如 ReAct）：
```
Action: Search[climate change effects]
```
局限：解析模糊，无组合性，提取脆弱。

**CodeAct 格式**：
```python
results = search("climate change effects", limit=5)
relevant = [r for r in results if "temperature" in r["title"]]
if not relevant:
    results = search("global warming temperature data", limit=10)
print(json.dumps(relevant, indent=2))
```
优势：组合、过滤、回退逻辑——全在一轮中完成。

### CodeActInstruct 数据集

- **规模**：约 7,000 条多轮交互轨迹
- **来源**：从需要工具使用的多样任务中精选
- **选择标准**：优先选择智能体初始犯错但随后自我修正的轨迹，以促进自我调试能力
- **格式**：每条轨迹包含思考、代码动作、执行输出
- **目的**：对开源 LLM 进行指令微调以使用 CodeAct 格式

### CodeActAgent 模型

发布了两个微调模型：
- **CodeActAgent-Llama-2-7B**：从 Llama-2-7B-chat 微调
- **CodeActAgent-Mistral-7B**：从 Mistral-7B-v0.1 微调

两者都在 CodeActInstruct + 通用对话数据上训练，以在擅长智能体任务的同时保持广泛能力。

## 关键创新

1. **统一代码动作空间**：将可执行代码形式化为智能体动作格式
2. **持久解释器**：IPython 会话跨轮保持状态，支持有状态计算
3. **通过执行实现自我调试**：代码执行的错误消息为修正提供自然反馈
4. **CodeActInstruct**：强调错误恢复轨迹的精选数据集，用于微调
5. **格式无关评估**：在相同任务上系统比较 JSON、文本和代码格式

## 实验设置

### 基准测试
- **API-Bank**：跨多个难度级别的 API/工具使用能力评估
- **M3ToolEval**：82 个人工精选任务，需要多轮交互中的多工具组合；本文新引入

### 评估模型
共 17 个 LLM：
- **闭源（9 个）**：GPT-4、GPT-3.5-turbo、Claude-2、Claude-instant、Gemini Pro 等
- **开源（8 个）**：Llama-2-7B/13B/70B、CodeLlama、Mistral-7B 等

### 基线
- **JSON 格式**：结构化 JSON 工具调用（OpenAI function-calling 风格）
- **文本格式**：ReAct 风格的文本动作

## 结果

### API-Bank 结果（成功率 %）

| 模型（部分）          | 文本    | JSON    | CodeAct |
|----------------------|---------|---------|---------|
| GPT-4                | 82.1    | 85.3    | **86.7**|
| GPT-3.5-turbo        | 73.4    | 76.2    | **78.9**|
| Claude-2             | 70.8    | 71.5    | **74.3**|
| Llama-2-70B          | 45.2    | 42.1    | **51.6**|
| Llama-2-7B           | 25.8    | 11.3    | **28.8**|

CodeAct 在 API-Bank 上是 4/8 开源 LLM 和 4/9 闭源 LLM 的最佳格式。

### M3ToolEval 结果（多工具任务）

CodeAct 在复杂多工具任务上展现最大增益：
- 成功率**最高提升 20 个百分点**（绝对值），相比 JSON/文本
- 完成任务所需的交互轮次更少
- 初始错误后的成功自我修正率更高

### CodeActAgent 微调模型

| 模型                         | 智能体任务 | 通用任务 |
|-----------------------------|-----------|---------|
| Llama-2-7B（基础）           | 25.8%     | 基线    |
| CodeActAgent-Llama-2-7B     | **42.3%** | 可比    |
| Mistral-7B（基础）           | 38.1%     | 基线    |
| CodeActAgent-Mistral-7B     | **53.7%** | 可比    |

微调模型在域外智能体任务上展现强劲改进，同时保持通用对话能力。

## 分析与洞察

- **复杂度放大差距**：在简单单工具任务上，所有格式表现相似；在复杂多工具组合任务上，CodeAct 的优势大幅增长
- **自我调试是关键**：CodeAct 智能体更频繁地从错误中恢复，因为它们可以检查追踪信息并以编程方式修改方法
- **所需轮次更少**：通过在单个代码块中组合多个操作，CodeAct 智能体以更少的交互轮次完成任务
- **开源差距**：较小的开源模型在 JSON 解析上比代码生成更吃力，使 CodeAct 对它们相对更有利
- **状态持久性很重要**：跨轮在变量中存储中间结果的能力减少了冗余计算

## 局限性与批评

- **安全顾虑**：执行任意 Python 代码带来沙箱挑战；生产部署需要强大的隔离
- **代码质量差异**：较弱的模型可能生成语法正确但语义错误的代码，比格式错误的 JSON 更难检测
- **以 Python 为中心**：该方法与 Python 绑定；扩展到其他语言或非代码领域需要重新思考
- **数据集规模**：CodeActInstruct 仅 7K 示例，对指令微调来说相对较少
- **基准范围**：M3ToolEval 虽有用但仅 82 个任务，可能无法捕捉真实世界智能体场景的全部多样性
- **延迟**：代码执行相比生产系统中的简单 JSON 解析增加了额外开销

## 后续工作

- **OpenHands（原 OpenDevin）**：采用 CodeAct 作为其默认动作格式，成为领先的开源智能体平台之一
- **SWE-agent**：使用相关方法，在 shell 环境中执行自定义命令
- **Code-as-policy 方法**：机器人控制中的代码生成遵循类似原则
- **Apple 的采用**：CodeAct 被 Apple ML Research 认可和展示
- **多智能体系统**：CodeAct 格式支持共享 Python 运行时的智能体间更自然的通信

## 核心要点

1. 可执行代码是 LLM 智能体的自然且优越的动作格式，利用了预训练分布
2. 代码的组合性和有状态性提供了相比结构化格式的根本优势
3. 通过执行反馈的自我调试是代码动作空间独有的能力
4. 在错误恢复轨迹（CodeActInstruct）上的微调有效地教会智能体自我修正
5. 动作格式之间的差距随任务复杂度增加而扩大，使 CodeAct 在真实世界应用中尤为有价值
