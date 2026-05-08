---
title: "ReAct: Synergizing Reasoning and Acting in Language Models (ReAct：协同语言模型中的推理与行动)"
authors:
  - Shunyu Yao
  - Jeffrey Zhao
  - Dian Yu
  - Nan Du
  - Izhak Shafran
  - Karthik Narasimhan
  - Yuan Cao
venue: ICLR 2023 (Oral)
year: 2023
submitted: 2022-10-06
url: https://arxiv.org/abs/2210.03629
code: https://github.com/ysymyth/ReAct
tags:
  - agents
  - reasoning
  - acting
  - prompting
  - language-models
  - tool-use
  - grounded-reasoning
  - decision-making
  - foundational
status: done
---

# ReAct：协同语言模型中的推理与行动

## 简述

ReAct 引入了一种提示范式，在单个语言模型生成循环中交替产生**推理轨迹**（思考）
和**任务特定动作**（行动）。通过让大语言模型交替地思考_和_行动——通过 API 检索
外部信息或操控环境——ReAct 克服了纯思维链推理的幻觉问题以及纯行动智能体缺乏
规划的不足。它在知识密集型问答（HotpotQA）、事实验证（FEVER）和交互式决策
基准测试（ALFWorld、WebShop）上取得了优异成绩，仅使用 PaLM-540B 的 1-6 个
上下文示例，就超越了纯推理和纯行动的基线方法。

---

## 动机与问题

### 来自认知科学的核心洞察

人类智能无缝地整合了面向任务的**行动**（例如拾取物体、输入查询）和**推理**
（例如规划、调整策略、验证事实）。在 ReAct 之前，大语言模型研究将二者视为
独立的方向：

- **纯推理**（例如 Chain-of-Thought）：模型生成内部推理轨迹，但无法访问
  外部信息，导致在知识密集型任务中出现幻觉和错误传播。
- **纯行动**（例如 WebGPT、SayCan）：模型发出动作与环境或工具交互，但缺乏
  关于_为何_采取这些动作的显式推理，导致规划能力差和无法从错误中恢复。

### 问题陈述

我们能否设计一个统一的提示框架，使大语言模型以交替方式同时生成**语言推理轨迹**
和**基于实际的行动**，从而使推理指导行动选择，行动为推理提供事实依据？

### 关键假设

1. 推理轨迹帮助模型**归纳、跟踪和更新行动计划**，以及**处理异常情况**
   （例如搜索返回不相关结果时）。
2. 行动使模型能够**与外部数据源交互**（知识库、环境），获取仅靠推理
   无法提供的额外信息。
3. 推理与行动的协同产生的轨迹比单独使用任一方法更具**可解释性**、
   **可诊断性**和**可控性**。

---

## 方法

### ReAct 提示格式

ReAct 通过添加**思考空间**来扩展智能体的动作空间。在每一步中，模型可以选择
产生一个*思考*（自由形式推理）或一个*动作*（任务特定命令）。每次动作之后，
环境返回一个*观察结果*。

```
Thought-Action-Observation Loop:
================================================

Question: <task/question>

Thought 1: <reasoning about what to do>
Action  1: <tool/environment action>
Observation 1: <result from environment>

Thought 2: <reasoning about observation>
Action  2: <next action based on reasoning>
Observation 2: <result from environment>

...

Thought n: <final reasoning>
Action  n: Finish[<answer>]
```

### 具体示例：HotpotQA

```
Question: What is the elevation range for the area that the eastern
          sector of the Colorado orogeny extends into?

Thought 1: I need to search Colorado orogeny, find the area that the
           eastern sector extends into, then find the elevation range
           of that area.
Action  1: Search[Colorado orogeny]
Observation 1: The Colorado orogeny was an episode of mountain building
               (an orogeny) in Colorado and surrounding areas.

Thought 2: It does not mention the eastern sector. So I need to look
           up eastern sector.
Action  2: Lookup[eastern sector]
Observation 2: (Result 1 / 1) The eastern sector extends into the
               High Plains and is called the Central Plains orogeny.

Thought 3: The eastern sector of the Colorado orogeny extends into
           the High Plains. I need to search High Plains and find
           its elevation range.
Action  3: Search[High Plains]
Observation 3: High Plains refers to one of two distinct land regions:
               the High Plains (United States)... elevation from
               around 1,800 to 7,000 ft (550 to 2,130 m).

Thought 4: High Plains rise in elevation from around 1,800 to 7,000 ft,
           so the answer is 1,800 to 7,000 ft.
Action  4: Finish[1,800 to 7,000 ft]
```

### 各任务可用动作

```
+------------------+-----------------------------------------------+
| 任务             | 动作空间                                      |
+------------------+-----------------------------------------------+
| HotpotQA / FEVER | Search[entity] - 查找维基百科页面               |
|                  | Lookup[keyword] - 在当前页面中搜索              |
|                  | Finish[answer]  - 返回最终答案                  |
+------------------+-----------------------------------------------+
| ALFWorld         | go to {receptacle}                             |
|                  | take {object} from {receptacle}                |
|                  | put {object} in/on {receptacle}                |
|                  | open/close {receptacle}                        |
|                  | toggle {object}                                |
|                  | clean/heat/cool {object} with {receptacle}     |
|                  | use {object}                                   |
|                  | examine {object/receptacle}                    |
|                  | look                                           |
+------------------+-----------------------------------------------+
| WebShop          | search[query]                                  |
|                  | click[element]                                 |
|                  | think[reasoning]                               |
+------------------+-----------------------------------------------+
```

### 算法框架（伪代码）

```
Algorithm: ReAct Agent Loop
==========================================
Input: question q, action space A, environment E, LM theta
Output: answer a

prompt <- load_few_shot_exemplars()  // 1-6 human-written trajectories
trajectory <- [("Question", q)]

for t = 1, 2, ... , T_max do:
    // Generate next thought or action
    output <- LM_theta(prompt + trajectory)

    if output is Thought:
        trajectory.append(("Thought t", output))

    else if output is Action:
        trajectory.append(("Action t", output))

        if output == "Finish[answer]":
            return answer

        // Execute action in environment
        observation <- E.execute(output)
        trajectory.append(("Observation t", observation))

    // Halt if trajectory exceeds max steps
    if t >= T_max:
        return FAIL
```

### 架构图

```
                    +-------------------+
                    |    LLM (PaLM)     |
                    |   (冻结，无       |
                    |    微调)          |
                    +--------+----------+
                             |
                    +--------v----------+
               +--->| 生成思考          |---+
               |    | 或动作            |   |
               |    +-------------------+   |
               |             |              |
               |    +--------v----------+   |
               |    | 是否为思考？      |   |
               |    +---+----------+----+   |
               |        |          |        |
               |       是          否       |
               |        |          |        |
               |        v          v        |
               |    [追加到      [在环境/   |
               |     上下文]     API中执行  |
               |        |        动作]      |
               |        |          |        |
               |        |    +-----v-----+  |
               |        |    |来自环境的  |  |
               |        |    |观察结果    |  |
               |        |    +-----+------+  |
               |        |          |         |
               |        +----+-----+         |
               |             |               |
               |    +--------v----------+    |
               |    | Finish[answer]？   |    |
               |    +---+----------+----+    |
               |        |          |         |
               |       是          否        |
               |        |          |         |
               |        v          +---------+
               |    [返回答案]
               |
               +--- [循环继续]
```

### 对比：ReAct 与其他提示范式

```
标准提示:         问题 -> 答案
思维链:           问题 -> 思考_1 -> ... -> 思考_n -> 答案
纯行动:           问题 -> 动作_1 -> 观察_1 -> ... -> 动作_n -> 答案
ReAct:            问题 -> 思考_1 -> 动作_1 -> 观察_1 ->
                          思考_2 -> 动作_2 -> 观察_2 ->
                          ... -> 思考_n -> Finish[答案]
```

---

## 关键创新

1. **统一的推理-行动框架**：首个展示在单一大语言模型生成循环中交替使用自由形式
   推理轨迹和基于实际的行动，能比单独使用任一方法产生协同优势的工作。

2. **思考作为灵活的控制机制**：思考服务于多种目的——将目标分解为子目标、跟踪
   进度、处理异常、从观察中提取相关信息，以及关于何时切换策略的常识推理。

3. **最小监督**：仅需 1-6 个人工编写的示例轨迹作为上下文演示（提示设置中
   无需训练数据、无需微调）。

4. **人类可解释性**：显式的思考轨迹使智能体的推理过程透明且可诊断，
   不同于不透明的纯行动智能体。

5. **协同机制**：行动解决了纯推理方法的幻觉和知识局限问题；推理解决了
   纯行动方法中缺乏规划和状态跟踪的问题。

6. **任务通用性**：同一框架适用于知识密集型 NLP 任务（问答、事实验证）和
   交互式决策任务（文字游戏、网页导航），仅需更改动作空间。

---

## 实验设置

### 任务与数据集

| 任务 | 数据集 | 指标 | 评估数量 | 领域 |
|------|---------|--------|--------|--------|
| 多跳问答 | HotpotQA (Yang et al. 2018) | 精确匹配 (EM)、F1 | 500（随机开发集） | Wikipedia |
| 事实验证 | FEVER (Thorne et al. 2018) | 标签准确率 | 500（随机开发集） | Wikipedia |
| 文字游戏 | ALFWorld (Shridhar et al. 2021) | 成功率 | 134（未见过的游戏） | 家庭场景 |
| 网页导航 | WebShop (Yao et al. 2022) | 成功率、奖励值 | 500（测试集） | 电子商务 |

### 模型

- **主要模型**：PaLM-540B (Chowdhery et al. 2022)，用于所有提示实验
- **微调模型**：PaLM-8B、PaLM-62B、PaLM-540B，用于规模分析

### 比较基线

| 方法 | 描述 |
|--------|-------------|
| Standard | 直接回答提示（无推理、无动作） |
| CoT (Chain-of-Thought) | 仅推理轨迹，无外部动作 |
| CoT-SC (Self-Consistency) | 带多数投票的 CoT，k=21 个样本 |
| Act | 仅动作，无推理思考 |
| ReAct | 交替的思考+动作（本文提出） |
| ReAct -> CoT-SC | ReAct 失败时回退到 CoT-SC |
| CoT-SC -> ReAct | CoT-SC 置信度低时回退到 ReAct |

### 超参数与设置

- **提示**：知识任务使用 3-6 个上下文示例；决策任务使用 1-2 个
- **解码**：ReAct 使用贪心解码（temperature=0）；CoT-SC 使用采样
- **Wikipedia API**：包含 Search[entity] 和 Lookup[keyword] 的简单接口
- **ALFWorld**：6 种任务类型，每种类型 1 个上下文示例
- **WebShop**：使用一个示例轨迹的 1-shot 提示

---

## 结果

### 表 1：知识密集型推理（PaLM-540B）

```
+---------------------+-------------------------+-------------------------+
|                     |       HotpotQA          |         FEVER           |
| 方法                |   EM    |     F1        |    准确率               |
+---------------------+---------+---------------+-------------------------+
| Standard            |  27.1   |      --       |      --                 |
| CoT (n=1)           |  29.4   |      --       |     56.3                |
| CoT-SC (n=21)       |  33.4   |      --       |     64.6                |
| Act                 |  25.7   |      --       |     58.9                |
| ReAct               |  27.4   |     35.1      |     60.9                |
| ReAct -> CoT-SC     |  35.1   |     40.1      |     64.6                |
| CoT-SC -> ReAct     |  34.2   |      --       |     65.4                |
+---------------------+---------+---------------+-------------------------+
```

**关键观察**：
- ReAct 在 HotpotQA 上比纯行动方法高 1.7 EM，在 FEVER 上高 2.0 准确率
- 在 HotpotQA 上，CoT（29.4）略优于 ReAct（27.4），因为内部推理在多跳
  问题上更灵活
- 在 FEVER 上，ReAct（60.9）显著优于 CoT（56.3），因为事实验证更多地
  受益于有依据的证据检索
- 组合方法（ReAct -> CoT-SC、CoT-SC -> ReAct）取得最佳结果，展示了
  内部知识和外部知识的互补性

### 表 2：交互式决策（PaLM-540B）

```
+---------------------+-------------------+-------------------------+
|                     |    ALFWorld       |       WebShop           |
| 方法                | 成功率 (%)        | 成功率 (%) | 奖励值     |
+---------------------+-------------------+-------------+-----------+
| BUTLER (IL, 10^5)   |      37           |     --      |    --     |
| IL (1,012 trajs)    |      --           |    29.1     |   59.9    |
| IL + RL (10,587)    |      --           |    28.7     |   62.4    |
| Act (best of 6)     |      45           |    30.1     |   62.3    |
| ReAct (best of 6)   |      71           |    40.0     |   66.6    |
+---------------------+-------------------+-------------+-----------+
```

**关键观察**：
- 在 ALFWorld 上，ReAct（71%）比 Act（45%）绝对提升 +26%，比 BUTLER（37%）
  绝对提升 +34%——提升巨大
- 在 WebShop 上，ReAct（40.0%）比 IL+RL（28.7%）绝对提升 +11.3%，
  尽管仅使用 1-shot 提示，而对手使用了数千条训练轨迹
- 即使 ReAct _最差_的试验（48%）在 ALFWorld 上也优于 Act _最好_的试验（45%）
- ReAct 相对 Act 在 ALFWorld 上的性能增益在 6 次受控试验中从 33% 到 90%
  不等，平均为 62%

### ALFWorld 各任务分解

```
+------------------------+----------+----------+
| 任务类型               | Act (%)  | ReAct (%)|
+------------------------+----------+----------+
| 拾取并放置             |   45     |    74    |
| 在灯下检查             |   50     |    83    |
| 清洁并放置             |   39     |    65    |
| 加热并放置             |   74     |    91    |
| 冷却并放置             |   58     |    75    |
| 拾取两个并放置         |   24     |    43    |
+------------------------+----------+----------+
```

ReAct 在 ALFWorld 所有 6 个任务类别上均持续优于 Act。

### 微调与提示对比（HotpotQA）

```
+---------------------+-------------+-------------+-------------+
| 方法 + 模型         | PaLM-8B     | PaLM-62B    | PaLM-540B   |
+---------------------+-------------+-------------+-------------+
| Standard（提示）    |    --       |    --       |   27.1      |
| CoT（提示）         |    --       |    --       |   29.4      |
| Act（提示）         |    --       |    --       |   25.7      |
| ReAct（提示）       |    --       |    --       |   27.4      |
+---------------------+-------------+-------------+-------------+
| Standard（微调）    |   23.6      |   27.0      |    --       |
| CoT（微调）         |   24.2      |   28.5      |    --       |
| Act（微调）         |   25.3      |   28.0      |    --       |
| ReAct（微调）       |   28.4      |   31.2      |    --       |
+---------------------+-------------+-------------+-------------+
```

**关键观察**：
- ReAct 在所有模型规模上都是最佳微调格式
- 使用 ReAct 微调的 PaLM-8B（28.4）超越了使用 Standard 提示的 PaLM-540B
  （27.1）——一个小 67 倍的模型超越了大得多的模型
- 使用 ReAct 微调的 PaLM-62B（31.2）超越了所有 PaLM-540B 的提示方法
- 这展示了 ReAct 通过蒸馏实现高效部署的潜力

---

## 分析与洞察

### 推理与行动的协同

论文提供了推理和行动互补的明确证据：

1. **行动帮助推理**：在知识任务中，外部检索纠正了模型的内部幻觉。纯 CoT
   在其失败案例中 56% 的时间会产生幻觉事实；ReAct 的搜索动作为推理提供了
   事实依据。

2. **推理帮助行动**：在决策任务中，思考帮助智能体分解目标、跟踪进度和
   从错误中恢复。没有思考（纯行动），智能体会丢失子目标并重复动作。

### 失败模式分析（各随机抽样 50 条轨迹）

```
+---------------------------+----------+----------+
| 错误类别                  | CoT (%)  | ReAct (%)|
+---------------------------+----------+----------+
| 幻觉                     |   56     |    --    |
| 推理错误                  |   --     |   23    |
| 搜索失败（无信息）        |   --     |   29    |
| 重复循环                  |   --     |   ~10   |
| 假阳性率                  |   14     |    6    |
+---------------------------+----------+----------+
```

**CoT 失败模式**：
- 幻觉是主要失败原因（56%）：模型编造听起来合理但实际错误的事实
- 假阳性率较高（14% vs. 6%）：CoT 更容易产生听起来自信但错误的答案

**ReAct 失败模式**：
- 搜索失败（29%）：模型检索到的信息不足以回答问题
- 推理错误（23%）：模型未能正确综合检索到的信息，或陷入重复的
  思考-行动循环
- ReAct 的错误更多是结构性的（检索不佳）而非事实性的（幻觉），
  使其更容易诊断和修复

### 为何组合方法效果最佳

ReAct -> CoT-SC 和 CoT-SC -> ReAct 切换策略取得最高分，因为它们
利用了互补优势：
- CoT 在答案存在于模型参数化知识中时表现出色
- ReAct 在需要外部证据检索时表现出色
- 切换启发式方法：利用 CoT-SC 的内部置信度（多数投票一致性）来决定
  是否回退到 ReAct 进行证据收集

---

## 局限性与批评

1. **依赖检索质量**：ReAct 在知识任务上的表现受 Wikipedia API 质量的
   制约。如果搜索返回不相关结果，推理无法恢复（29% 的失败案例）。

2. **提示敏感性**：性能在很大程度上取决于少样本示例轨迹的质量和多样性。
   人工编写的示例必须展示所期望的推理风格。

3. **计算成本**：每条 ReAct 轨迹涉及多次大语言模型调用（每次思考/动作
   一次），与单次 CoT 相比显著增加了推理成本。

4. **有限的动作空间**：评估的任务使用相对简单的动作空间（3 个 Wikipedia
   动作、基本文字游戏命令）。扩展到复杂的实际工具使用尚未得到验证。

5. **贪心解码限制**：ReAct 使用贪心解码，而 CoT-SC 受益于多路径采样。
   这种比较劣势可以通过将自一致性也应用到 ReAct 来解决。

6. **重复循环问题**：ReAct 有时会进入循环，重复生成相同的思考-动作序列，
   表明模型在规划恢复策略方面存在不足。

7. **规模要求**：与 CoT 类似，ReAct 从非常大的模型（540B 参数）中受益
   最多。在较小模型上通过提示使用效果有限，但微调可以部分解决这个问题。

---

## 后续工作

- **Reflexion** (Shinn et al., 2023)：扩展 ReAct，加入语言自我反思，
  使智能体能够从存储在情景记忆中的过去错误中学习。
- **Tree of Thoughts** (Yao et al., 2023)：将 CoT 扩展为树状结构
  探索，由同一第一作者完成。
- **Toolformer** (Schick et al., 2023)：通过自监督学习而非提示来
  教会大语言模型使用工具。
- **FireAct** (Chen et al., 2023)：使用来自多种提示方法的 ReAct 风格
  轨迹微调语言智能体。
- **LATS** (Zhou et al., 2023)：将 ReAct 与蒙特卡洛树搜索结合，
  实现更系统的探索。
- **AutoGPT / BabyAGI** (2023)：深受 ReAct 循环启发的开源智能体系统。
- **LangChain** (Chase, 2022)：将 ReAct 作为其默认智能体架构的
  流行框架。

---

## 核心要点

1. **推理和行动是协同而非竞争的范式。** 将二者结合在单一框架中，在各种
   任务类型上都比单独使用任一方法产生严格更优的结果。

2. **通过行动实现接地对知识密集型任务至关重要。** 纯推理（CoT）会产生
   幻觉；交替使用搜索动作减少了幻觉并提供了事实基础。

3. **推理对复杂决策至关重要。** 纯行动（Act）在长期任务上失败，因为
   智能体无法规划、分解目标或跟踪状态。添加思考解决了这个问题。

4. **思考-行动-观察循环是通用的智能体架构。** 同一框架仅需最小修改
   即可用于问答、事实验证、文字游戏和网页导航。

5. **少样本提示可以超越大量训练的 RL/IL 智能体。** 使用 1-2 个示例
   的 ReAct 超越了在数千条轨迹上训练的智能体，在 ALFWorld 上提升
   34%，在 WebShop 上提升 10%。

6. **微调放大了 ReAct 的优势。** ReAct 格式微调在所有模型规模上都是
   最佳格式，微调后的小模型（8B）可以超越提示的大模型（540B）。

7. **人类可解释性是实际优势。** 显式推理轨迹使 ReAct 智能体比不透明的
   纯行动或端到端学习的智能体更容易调试、审计和改进。

8. **ReAct 建立了基础性的智能体循环**，大多数后续大语言模型智能体系统
   （LangChain、AutoGPT 等）都采用了这一核心架构模式。
