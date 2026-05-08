---
title: "Toolformer: Language Models Can Teach Themselves to Use Tools (Toolformer：语言模型可以自学使用工具)"
authors:
  - Timo Schick
  - Jane Dwivedi-Yu
  - Roberto Dessi
  - Roberta Raileanu
  - Maria Lomeli
  - Eric Hambro
  - Luke Zettlemoyer
  - Nicola Cancedda
  - Thomas Scialom
venue: NeurIPS 2023 (Advances in Neural Information Processing Systems 36)
year: 2023
url: https://arxiv.org/abs/2302.04761
institution: Meta AI Research
tags:
  - tool-use
  - self-supervised-learning
  - language-models
  - API-calls
  - augmented-LMs
  - agents
  - foundational
status: done
---

# Toolformer：语言模型可以自学使用工具

## 简述

Toolformer 是一种自监督方法，教会语言模型（GPT-J 6.7B）自主决定何时以及如何调用
外部工具 API（计算器、问答系统、搜索引擎、日历、翻译器），通过将 API 调用直接嵌入
文本序列中实现。模型在大规模语料库上生成候选 API 调用标注，通过判断其是否降低了后续
token 的困惑度来过滤候选标注，然后在保留的标注上进行微调。最终的 6.7B 模型在多个
零样本基准测试上匹配或超越了 GPT-3 175B——仅用 25 分之一的参数量实现了这一成绩。

---

## 动机与问题

大语言模型存在仅靠扩展规模无法解决的根本局限：

1. **无法访问最新信息**——语言模型在训练时被冻结，无法检索当前的事实、
   价格、天气或新闻。
2. **数学推理能力弱**——即使是 175B 参数的模型在基本的多位数算术和
   应用题上也会失败。
3. **缺乏时间意识**——语言模型没有当前日期或时间的概念。
4. **事实回忆中的幻觉**——参数化记忆在长尾知识上不可靠。
5. **多语言能力有限**——低资源语言对的翻译质量会退化。

先前的工具增强语言模型方法（例如 TALM、WebGPT、LaMDA）依赖于大量的人工标注或
任务特定的监督。Toolformer 提出：模型能否仅使用每个工具的少量演示、不需要任何
任务特定标签就_自行_学会工具何时有用？

---

## 方法

### 高层流水线

该方法对每个工具独立应用三个阶段：

```
阶段 1: 标注           阶段 2: 过滤            阶段 3: 微调
+-----------------+      +------------------+      +------------------+
| 从语料库 C 中   |      | 执行 API 调用    |      | 合并所有工具     |
| 采样候选 API    | ---> | 比较有结果和无    | ---> | 数据集           |
| 调用位置和      |      | 结果时的损失      |      | 在增强文本 +     |
| 参数            |      | 若损失减少 >= t_f |      | 原始 C 上微调    |
|                 |      | 则保留            |      | GPT-J            |
+-----------------+      +------------------+      +------------------+
```

### 逐步标注流水线

**步骤 1：位置采样**

给定语料库 C（CCNet 子集）中的文本序列 x = (x_1, ..., x_n)，对每个位置 i
计算模型 M 生成特殊 `<API>` token 的概率：

```
p_i = P_M(<API> | x_1, ..., x_i)
```

保留所有 p_i > tau_s（采样阈值）的位置。如果超过 k 个位置符合条件，仅保留
概率最高的 top-k 个。这通过少样本提示引导实现：为每个工具向模型提供 2-3 个
带 API 调用的文本示例，使其在相关位置对 `<API>` 赋予非零概率。

- **tau_s = 0.05**（5% 概率阈值）
- **k = 5** 每个序列最多候选位置数

**步骤 2：候选生成**

对每个选定位置 i，从前缀 [x_1, ..., x_i, <API>] 继续生成，采样 m 个候选
API 调用，直到模型生成 `</API>`。生成候选调用 c_1, ..., c_m。

- **m = 5** 每个位置的候选数
- 采样使用核采样或基于温度的解码

**步骤 3：API 执行**

执行每个候选 API 调用 c_j 以获取结果 r_j。构建两个增强序列：

```
e_i^+ = (x_1,...,x_{i-1}, <API> c_j -> r_j </API>, x_i,...,x_n)   [有结果]
e_i^- = (x_1,...,x_{i-1}, <API> c_j ->  </API>, x_i,...,x_n)       [无结果]
```

**步骤 4：基于困惑度的过滤**

计算 API 调用后续 token 的加权交叉熵损失：

```
                   n
L_i(z) = - SUM   w_{t-i} * log P_M(x_t | z, x_{1:t-1})
               t=i
```

其中：
- z 是插入的 API 调用（有或无结果）
- w_{t-i} 是位置相关的权重（距 API 调用越近的 token 权重越大）
- 求和遍历位置 i 之后的所有 token

定义两个关键量：

```
L_i^+ = L(e_i^+)    -- 有 API 结果时的损失
L_i^- = L(e_i^-)    -- 无 API 结果时的损失（或空字符串）
```

**过滤准则：** 仅当满足以下条件时保留位置 i 处的 API 调用 c_j：

```
L_i^- - L_i^+ >= tau_f
```

其中 **tau_f = 1.0**（API 调用必须至少降低 1.0 nat 的损失）。

这意味着：仅当工具的答案使模型在预测后续 token 时比没有工具答案时显著更好，
该调用才被保留。这是核心洞察——模型通过自身的损失函数判断工具的有用性。

### 伪代码：完整标注流水线

```
Algorithm: Toolformer Self-Supervised Annotation

Input:  Corpus C, Model M, Tool set T, thresholds tau_s, tau_f, limits k, m
Output: Augmented corpus C*

C* = {}
for each tool t in T:
    for each text x = (x_1,...,x_n) in C:
        # Step 1: Sample positions
        positions = {}
        for i = 1 to n:
            p_i = P_M(<API> | prompt_t, x_1,...,x_i)
            if p_i > tau_s:
                positions.add((i, p_i))
        positions = top_k(positions, k)  # keep top-k by probability

        # Step 2: Generate candidates
        for each position i in positions:
            candidates = {}
            for j = 1 to m:
                c_j = sample_continuation(M, [x_1,...,x_i, <API>], stop=</API>)
                candidates.add(c_j)

            # Step 3: Execute and filter
            for each c_j in candidates:
                r_j = execute_tool(t, c_j)
                L_plus  = weighted_loss(M, x, i, api_call=c_j, result=r_j)
                L_minus = weighted_loss(M, x, i, api_call=c_j, result="")
                if L_minus - L_plus >= tau_f:
                    x* = insert_annotation(x, i, c_j, r_j)
                    C*.add(x*)
                    break  # use first passing candidate

# Step 4: Fine-tune
C_final = C* UNION C  # merge annotated + original data
M* = finetune(M, C_final, lr=1e-5, warmup=10%)
return M*
```

### 工具 API 格式

每个工具使用一致的标注语法：`[ToolName(input) -> output]`

**1. 计算器 (Calc)**
```
语法:     [Calculator(mathematical_expression) -> result]
示例:     Out of 1400 participants, 400 (or [Calculator(400/1400) -> 0.29] 29%) passed.
范围:     四种基本算术运算 (+, -, *, /)
输出:     四舍五入到小数点后两位
```

**2. 问答 (QA)**
```
语法:     [QA(natural_language_question) -> answer]
示例:     The NEJM is a trademark of [QA("Publisher of NEJM?") -> Massachusetts Medical
          Society] the MMS.
后端:     Atlas 检索增强语言模型（少样本，未微调）
```

**3. 维基百科搜索 (WikiSearch)**
```
语法:     [WikiSearch(query) -> first_sentence_of_article]
示例:     The Brown Act is [WikiSearch("Brown Act") -> The Ralph M. Brown Act is an act
          of the California State Legislature...] California's open meetings law.
后端:     基于 Wikipedia 的 BM25 检索，返回排名第一结果的首句
```

**4. 机器翻译 (MT)**
```
语法:     [MT(text_in_source_language) -> translated_text]
示例:     "la tortuga", the Spanish word for [MT("tortuga") -> turtle] turtle.
后端:     600M 参数的 NLLB 模型（200 种语言）
```

**5. 日历**
```
语法:     [Calendar() -> current_date_string]
示例:     The WL opens on Friday, [Calendar() -> Today is Thursday, March 9, 2017.]
          March 10.
后端:     返回当前日期的简单系统调用（无输入参数）
```

### 微调细节

过滤后，来自所有工具的存活标注示例与原始未标注语料库 C 合并。然后在这个
组合数据集上微调模型。

- **基础模型：** GPT-J 6.7B (EleutherAI)
- **训练语料：** CCNet 子集
- **学习率：** 1e-5，前 10% 训练步骤线性预热
- **权重衰减：** 0.01
- **序列长度：** 1024 tokens
- **目标：** 标准语言建模（下一个 token 预测）
- **数据集组成：** 增强示例与原始文本交替排列，以防止灾难性遗忘
  通用语言建模能力

在推理时，当模型生成 `<API>` 时，生成暂停，API 调用被执行，结果被插入，
然后从 `</API>` 继续生成。

---

## 关键创新

1. **自监督工具标注**——无需人工标注何时使用工具。模型完全通过自身的损失信号
   发现有用的工具调用。每个工具仅需少量（2-3 个）演示即可引导流程。

2. **困惑度作为奖励**——使用模型自身的下一个 token 预测损失作为过滤标准
   非常优雅：它直接衡量工具调用是否帮助模型更好地完成任务。这避免了需要
   任务特定的奖励模型。

3. **工具无关框架**——相同的标注/过滤/微调流水线适用于任何可以表示为
   文本输入/文本输出 API 的工具。添加新工具仅需编写几个演示示例。

4. **内联 API 调用**——工具调用使用特殊 token 直接嵌入 token 流中，而非
   需要单独的动作空间或多轮协议。这保持了自回归生成范式。

5. **解耦的工具训练**——每个工具的标注独立生成，允许并行化和模块化。
   最终微调合并所有工具。

---

## 实验设置

### 数据集与基准测试

| 类别   | 数据集     | 任务                          | 指标     |
|--------|------------|-------------------------------|----------|
| 数学   | ASDiv      | 算术应用题                    | 准确率   |
| 数学   | SVAMP      | 数学应用题                    | 准确率   |
| 数学   | MAWPS      | 数学应用题                    | 准确率   |
| 问答   | WebQS      | 开放域问答（网络）            | 准确率   |
| 问答   | NQ         | Natural Questions             | 准确率   |
| 问答   | TriviaQA   | 知识问答                      | 准确率   |
| 知识   | LAMA-SQuAD | 事实完形填空（SQuAD 子集）    | 准确率   |
| 知识   | LAMA-T-REx | 事实完形填空（T-REx 子集）    | 准确率   |
| 知识   | LAMA-G-RE  | 事实完形填空（Google-RE）     | 准确率   |
| 时间   | TempLAMA   | 时间敏感事实完形填空          | 准确率   |
| 时间   | DateSet    | 日期理解                      | 准确率   |
| 翻译   | MLQA       | 多语言问答（6 种语言）        | F1       |

### 基线

- **GPT-J 6.7B**——基础模型，无工具
- **GPT-J 6.7B 在 CCNet 上微调**——相同数据，无工具标注
- **OPT 66B**——大 10 倍，无工具
- **GPT-3 175B**——大 25 倍，无工具（通过 OpenAI API）
- **Toolformer 禁用工具**——使用工具数据微调但在推理时禁止工具调用

### 消融模型变体

- GPT-2 124M、GPT-2 355M、GPT-2 775M、GPT-2 1.6B、GPT-J 6.7B

---

## 结果

### 主要结果表

| 基准测试     | GPT-J 6.7B | OPT 66B | GPT-3 175B | Toolformer 6.7B |
|--------------|-----------|---------|------------|-----------------|
| ASDiv        | 7.5       | --      | 14.0       | **40.4**        |
| SVAMP        | 5.2       | 4.9     | 10.0       | **29.4**        |
| MAWPS        | 9.9       | --      | 19.8       | **44.0**        |
| SQuAD (LAMA) | 19.2      | --      | --         | **33.8**        |
| WebQS        | 18.5      | --      | --         | 26.3            |
| NQ           | 12.8      | --      | --         | 17.7            |
| TriviaQA     | 43.9      | --      | --         | 48.8            |
| TempLAMA     | 13.7      | --      | --         | 16.3            |
| DateSet      | 3.9       | --      | --         | **27.3**        |

关键观察：
- 在数学基准测试（ASDiv、SVAMP、MAWPS）上，Toolformer 将 GPT-J 的表现
  **提高了四倍**，并将 GPT-3 的表现**提高了两到三倍**，仅使用 6.7B 参数
- 在 LAMA-SQuAD 上，Toolformer 在 98.1% 的案例中使用 QA 工具，从 19.2
  提升到 33.8
- 在 DateSet 上，Toolformer 通过调用日历工具实现了约 7 倍的改进
- 在问答基准测试（WebQS、NQ、TriviaQA）上，增益较为温和，因为事实回忆
  与模型的参数化知识部分重叠
- Toolformer 在所有数学基准测试上轻松超越 OPT 66B 和 GPT-3 175B
- 在 LAMA 知识基准测试上，Toolformer 超越了所有同规模基线，并取得了
  与 GPT-3 相当或更优的结果

### 工具使用分析

| 工具          | 主要基准测试         | 使用率        | 收益级别      |
|---------------|----------------------|---------------|---------------|
| 计算器        | ASDiv, SVAMP, MAWPS  | 非常高        | 非常高        |
| QA (Atlas)    | LAMA, WebQS, NQ      | LAMA 上约 98% | 高            |
| WikiSearch    | LAMA（部分子集）     | 中等          | 中等          |
| 日历          | TempLAMA, DateSet    | 日期相关上高  | 高            |
| MT (NLLB)     | MLQA                 | 变化较大      | 混合          |

- **计算器**提供了最大的单项收益——数学基准测试获得 4-5 倍的增益
- **QA 工具**在知识基准测试上使用最频繁，带来持续改进
- **WikiSearch** 显示适度收益；在某些基准测试上与 QA 部分冗余
- **日历**对时间推理至关重要但适用范围较窄
- **MT** 在 MLQA 的大多数语言上提供收益，但 CCNet 微调可能损害
  多语言性能，因此 Toolformer 并不总是优于基础 GPT-J

### 消融：模型规模与工具使用涌现

| 模型规模 | 能有效使用工具？ | 备注                                          |
|----------|------------------|-----------------------------------------------|
| 124M     | 否               | 工具可用性未带来改进                          |
| 355M     | 否               | 工具可用性未带来改进                          |
| 775M     | 部分             | 工具使用的初步迹象；WikiSearch 有效           |
| 1.6B     | 基本可以         | 在大多数 API 上有合理的工具使用               |
| 6.7B     | 是               | 完整的工具使用能力；强劲增益                  |

关键发现：**工具使用是一种涌现能力**，仅在约 775M 参数以上才可靠出现。
低于此规模的模型即使有相同的训练信号，也无法学会工具何时有用。例外是
WikiSearch，它"相对容易使用"，即使在较小规模上也显示出一些收益。

---

## 分析与洞察

### 为什么自监督有效

困惑度过滤标准作为自然的奖励信号发挥作用。API 调用仅在提供模型自身无法预测
的信息时才有用。这创建了自动课程：模型已知答案的简单事实完成被过滤掉（API
调用不降低损失），而工具确实有帮助的真正困难预测被保留。

### "正确"的抽象层级

Toolformer 在 token 层面运作——API 调用只是特殊的 token 序列。这意味着：
- 不需要单独的"动作空间"
- 不需要强化学习或奖励建模
- 标准语言建模训练即可
- 模型自然地学会在上下文合适的位置放置调用

### 工具调用的分布

模型学会了直观的放置模式：
- 计算器调用出现在文本中数字结果之前
- QA 调用出现在实体名称或事实陈述之前
- 日历调用出现在日期引用附近
- MT 调用出现在外语术语附近

### 规模动态

虽然模型随规模增长在不使用 API 调用的情况下解决任务的能力提高，但它们
有效利用所提供 API 的能力也同时提高。工具使用收益和原始能力收益是加性的，
而非替代性的。

### 与同期工作的比较

Toolformer 先于后续智能体框架（ReAct、function calling）出现，与之不同的
是它不使用思维链推理或多步规划。每次 API 调用都是单一的独立调用。这既是
优势（简单性），也是局限（不支持多步工具使用）。

---

## 局限性与批评

1. **不支持工具链式调用**——API 调用按工具独立生成。一个工具的输出不能作为
   另一个工具的输入。这严重限制了需要多步工具使用的复杂推理（例如"搜索 X，
   然后根据结果计算 Y"）。

2. **无交互式工具使用**——模型无法浏览搜索结果、优化查询或与工具进行多轮
   交互。每次调用只有一次机会。

3. **仅限文本输入/文本输出 API**——需要结构化输入（JSON 模式、图像、文件）
   或产生非文本输出的工具无法集成。

4. **规模要求**——该方法仅适用于 >= 775M 参数的模型，限制了对较小或
   边缘部署模型的适用性。

5. **CCNet 微调的权衡**——在 CCNet 上微调可能损害某些任务的表现（例如多语言
   问答），使得难以区分工具收益和数据效应。

6. **静态工具集**——添加新工具需要重新运行完整的标注/过滤/微调流水线。
   推理时无法即插即用地添加工具。

7. **评估不足**——论文未评估需要真正多步推理、规划或复杂工具编排的任务——
   正是智能体最需要的场景。

8. **每个位置单一工具**——每个标注位置仅使用一个工具。模型无法推理在同一
   位置哪个工具最合适。

9. **标注质量依赖基础模型**——如果基础模型在真正有用的位置对 `<API>` 赋予
   低概率（分布不匹配），这些位置永远不会被采样，潜在有价值的工具使用就会
   被遗漏。

---

## 后续工作

- **Gorilla (2023)**——将工具学习扩展到数千个 API，使用检索增强生成
  获取 API 文档。
- **ToolLLM / ToolBench (2023)**——扩展到 16,000+ 个真实世界 API，
  支持多步工具使用。
- **ART (Automatic Reasoning and Tool-use, 2023)**——将思维链与工具使用
  结合在多步框架中。
- **Chameleon (2023)**——使用基于大语言模型的规划器进行即插即用的工具组合。
- **TaskMatrix.AI / HuggingGPT (2023)**——使用大语言模型编排许多专业模型
  作为工具。
- **Function Calling (OpenAI, 2023)**——结构化工具使用的商业实现，使用
  JSON 模式，直接受 Toolformer 范式启发。
- **ReAct (2022)**——交替推理与行动；互补方法，在工具调用前添加思维链。

---

## 核心要点

1. **自监督足以进行工具学习**——每个工具仅需 2-3 个演示，模型就能通过
   自身损失过滤来学习何时以及如何使用工具。无需大规模的工具使用示例
   人工标注。

2. **小模型+工具 > 大模型**——Toolformer 6.7B + 计算器在数学上以 2-3 倍
   超越 GPT-3 175B。这表明工具增强比纯规模扩展更具参数效率。

3. **困惑度是通用的工具有用性信号**——损失降低标准（L_i^- - L_i^+ >= tau_f）
   优雅、有原则且工具无关。可以无需修改地应用于任何文本输入/文本输出工具。

4. **工具使用是涌现能力**——它需要最低模型规模（约 775M）才能出现，表明
   工具使用依赖于基础模型中足够的推理能力。

5. **内联 API 范式强大但有限**——将工具调用嵌入 token 流中对单步工具使用
   简单有效，但不支持现代智能体框架所需的多步、交互式工具使用。

6. **奠基性贡献**——Toolformer 确立了语言模型可以自主学习工具使用，直接
   启发了现在商业大语言模型（GPT-4、Claude、Gemini）中标准配置的函数调用
   和工具使用能力。
