---
title: LLM Agent Survey Papers - Annotated Collection
authors: Various
venue: Various (arXiv, NeurIPS, ICLR, ACL)
year: 2023-2025
tags: [agent, survey, llm, taxonomy, comprehensive]
status: reference
---

## 简要总结

关于基于 LLM 的智能体最具影响力的综述论文精选集，涵盖分类体系、架构、能力、应用、评估和开放挑战。

---

## 综述 1："The Rise and Potential of Large Language Model Based Agents: A Survey"

- **作者**：Zhiheng Xi, Wenxiang Chen, Xin Guo 等
- **机构**：Fudan University
- **arXiv**：2309.07864
- **日期**：2023 年 9 月（2024 年更新）
- **页数**：约 86 页
- **引用**：智能体综述中引用最多的之一

### 分类体系
论文提出了三模块智能体框架：
1. **大脑 (LLM)**：自然语言交互、知识、推理、规划、迁移性
2. **感知**：文本、视觉、听觉输入处理
3. **行动**：工具使用、具身行动、通信

### 关键贡献
- 全面的智能体能力分类体系
- 单智能体 vs. 多智能体系统的详细分析
- 应用领域：社会科学、自然科学、工程
- 智能体社会分析：合作、竞争、社会规范
- 安全和伦理考量的讨论

### 智能体能力分类体系
```
Agent
├── 大脑 (LLM)
│   ├── 自然语言交互
│   ├── 知识（语言、常识、专业、领域）
│   ├── 记忆（上下文内、外部、长期）
│   ├── 推理（归纳、演绎、溯因）
│   └── 规划（无反馈、有环境反馈、有人类反馈）
├── 感知
│   ├── 文本输入
│   ├── 视觉输入
│   └── 听觉输入
└── 行动
    ├── 工具使用
    ├── 具身行动
    └── 通信 / 输出
```

---

## 综述 2："A Survey on Large Language Model based Autonomous Agents"

- **作者**：Lei Wang, Chen Ma, Xueyang Feng 等
- **机构**：Renmin University of China, Tencent
- **arXiv**：2308.11432
- **日期**：2023 年 8 月（2024 年更新）
- **页数**：约 45 页

### 分类体系
论文围绕**构建-应用**框架组织智能体：

**智能体构建：**
1. **角色模块**：智能体身份、角色、性格
2. **记忆模块**：短期、长期、混合记忆结构
3. **规划模块**：
   - 无反馈：单路径（CoT、零样本 CoT）、多路径（ToT、GoT、CoT-SC）
   - 有反馈：环境、人类、基于模型的
4. **行动模块**：工具使用、具身行动、目标完成

**智能体应用：**
- 单智能体：面向任务（代码、数据）、基于模拟
- 多智能体：合作、对抗、辩论、社会模拟
- 人机交互：教育、健康、工作辅助

### 关键贡献
- 智能体构建和应用的清晰分离
- 记忆架构的详细比较
- 规划模块分类体系（有/无反馈）
- 智能体评估策略

---

## 综述 3："Cognitive Architectures for Language Agents" (CoALA)

- **作者**：Theodore R. Sumers, Shunyu Yao, Karthik Narasimhan, Thomas L. Griffiths
- **机构**：Princeton University
- **arXiv**：2309.02427
- **日期**：2023 年 9 月

### 框架
CoALA 为语言智能体提供了一个形式化的认知架构框架：

**结构组件：**
- **记忆**：工作记忆（上下文）、情景记忆（交互历史）、语义记忆（知识库）、程序记忆（代码/提示）
- **决策**：规划和推理过程
- **接地**：与外部环境的接口

**处理循环：**
```
观察 -> 从记忆中检索 -> 推理/规划 -> 行动 -> 更新记忆
```

### 核心洞察
语言智能体可以通过认知科学的视角来理解 -- 它们实现了认知架构（如 ACT-R 和 SOAR）的（简化）版本，LLM 充当灵活的推理引擎。

### 智能体设计空间
论文定义了系统化的设计空间维度：
- 记忆类型（哪些种类的记忆）
- 决策过程（如何做出决策）
- 接地方式（哪些外部接口存在）
- 学习（智能体如何随时间改进）

---

## 综述 4："Agent AI: Surveying the Horizons of Multimodal Interaction"

- **作者**：Zane Durante, Qiuyuan Huang, Naoki Wake 等
- **机构**：Microsoft Research, Stanford, CMU 等
- **arXiv**：2401.03568
- **日期**：2024 年 1 月

### 重点
对"智能体 AI"的更广泛视角，涵盖多模态、具身和交互式智能体：
- 视觉-语言-行动模型
- 游戏和虚拟世界智能体
- 机器人和具身智能体
- 医疗和科学智能体

### 关键贡献
弥合了基于 LLM 的文本智能体与多模态/具身智能体之间的差距，提供了统一的视角。

---

## 综述 5："A Survey on the Memory Mechanism of Large Language Model based Agents"

- **作者**：Zeyu Zhang, Xiaohe Bo, Chen Ma 等
- **机构**：多个机构
- **arXiv**：2404.13501
- **日期**：2024 年 4 月

### 重点
深入研究 LLM 智能体的记忆系统：

**记忆分类体系：**
```
智能体记忆
├── 试验内记忆（单个任务内）
│   ├── 工作记忆（上下文窗口）
│   ├── 情景缓冲区（观察缓存）
│   └── 语义缓存（中间知识）
├── 跨试验记忆（跨任务）
│   ├── 情景记忆（过去经验）
│   ├── 语义记忆（积累的知识）
│   └── 程序记忆（学到的策略）
└── 记忆操作
    ├── 读取（检索）
    ├── 写入（存储）
    ├── 反思（元认知）
    └── 巩固（压缩、抽象）
```

---

## 综述 6："Large Language Model based Multi-Agents: A Survey of Progress and Challenges"

- **作者**：Taicheng Guo, Xiuying Chen, Yaqi Wang 等
- **arXiv**：2402.01680
- **日期**：2024 年 2 月

### 重点
专门的多智能体 LLM 系统综述：

**多智能体交互分类体系：**
1. **合作**：智能体共同朝着共享目标工作
   - 有序消息传递（轮替制、链式）
   - 无序消息传递（黑板、广播）
2. **竞争/对抗**：智能体竞争或辩论
3. **混合**：合作和竞争元素的组合

**应用：**
- 软件开发（ChatDev、MetaGPT）
- 科学与研究（多智能体科学辩论）
- 社会模拟（Generative Agents、AgentSims）
- 游戏（Voyager、Ghost in the Minecraft）
- 推理（多智能体辩论以提高事实性）

---

## 综述 7："Tool Learning with Large Language Models: A Survey"

- **作者**：Changle Qu, Sunhao Dai, Xiaochi Wei 等
- **arXiv**：2405.17935
- **日期**：2024 年 5 月

### 重点
LLM 工具使用的全面综述：

**工具学习流水线：**
1. **工具创建**：创建新工具（代码生成、API 封装）
2. **工具选择**：为任务选择合适的工具
3. **工具调用**：使用正确参数生成正确的函数调用
4. **工具结果处理**：解释和使用工具输出
5. **工具学习**：通过经验改进工具使用

**工具类型：**
- 感知工具（搜索、阅读、浏览）
- 行动工具（写入、执行、通信）
- 计算工具（计算、分析、转换）
- 领域特定工具（科学仪器、数据库）

---

## 综述 8："Retrieval-Augmented Generation for Large Language Models: A Survey"

- **作者**：Yunfan Gao, Yun Xiong, Xinyu Gao 等
- **arXiv**：2312.10997
- **日期**：2023 年 12 月（2024 年更新）

### RAG 范式演化
```
朴素 RAG -> 高级 RAG -> 模块化 RAG
```

**朴素 RAG**：简单的检索-然后-生成流水线
**高级 RAG**：检索前优化 + 检索后处理
**模块化 RAG**：灵活的、可重新配置的 RAG 组件

### RAG vs. 微调 vs. 提示工程
| 方面 | RAG | 微调 | 提示工程 |
|--------|-----|-------------|-------------------|
| 外部知识 | 是 | 嵌入在权重中 | 仅上下文内 |
| 是否需要训练 | 否（检索模型可选） | 是 | 否 |
| 可更新的知识 | 容易（更新文档） | 需要重新训练 | 受限于上下文 |
| 最适合 | 动态知识、引用 | 风格/格式/领域适应 | 简单任务 |

---

## 综述 9："Personal LLM Agents: Insights and Survey about the Capability, Efficiency and Security"

- **作者**：Yuanchun Li, Hao Wen, Weijun Wang 等
- **机构**：Tsinghua University, Microsoft
- **arXiv**：2401.05459
- **日期**：2024 年 1 月

### 重点
在个人设备上运行并与个人数据交互的智能体：
- 端侧智能体能力
- 隐私和安全考量
- 效率挑战（在有限硬件上运行）
- 个人数据管理和同意

---

## 综述 10："The Landscape of Emerging AI Agent Architectures for Reasoning, Planning, and Tool Calling"

- **作者**：Tula Masterman, Sandi Besen, Mason Sawtell, Alex Chao
- **机构**：Neudesic (an IBM Company)
- **arXiv**：2404.11584
- **日期**：2024 年 4 月

### 架构分类
1. **单智能体**：一个 LLM 配工具（ReAct、Plan-Execute）
2. **多智能体 - 纵向**：层次化委托（管理者-工作者）
3. **多智能体 - 横向**：点对点协作
4. **混合**：组合方法

### 评估框架
提出了选择智能体架构的标准：
- 任务复杂度
- 所需的自主级别
- 延迟要求
- 成本约束
- 安全要求

---

## 推荐阅读顺序（按时间）

1. **从这里开始**：CoALA (2309.02427) -- 获取理论框架
2. **全面概述**：复旦综述 (2309.07864) -- 最全面
3. **构建导向**：人民大学综述 (2308.11432) -- 实用分类体系
4. **架构模式**：Masterman et al. (2404.11584) -- 实用选择指南
5. **多智能体深入**：Guo et al. (2402.01680) -- 如果构建多智能体系统
6. **记忆深入**：Zhang et al. (2404.13501) -- 如果设计记忆系统
7. **RAG 深入**：Gao et al. (2312.10997) -- 如果构建基于 RAG 的智能体
8. **工具使用深入**：Qu et al. (2405.17935) -- 如果聚焦于工具集成

## 跨综述的关键观察

1. **分类体系趋同**：尽管命名不同，所有综述都趋向相似的核心组件（规划、记忆、工具、行动）
2. **记忆被低估**：大多数智能体框架的记忆实现远不如综述建议的复杂
3. **评估很难**：不存在用于智能体的标准化评估框架；基准测试是碎片化的
4. **多智能体前景光明但成本高**：多智能体系统显示出强结果但计算成本显著
5. **安全至关重要**：所有综述都将安全/对齐作为主要开放挑战
6. **领域发展迅速**：许多综述论文由于发展速度已经部分过时

## 笔记

跨综述提到的评估智能体的关键基准测试：
- **AgentBench**：全面的多环境智能体基准测试
- **ToolBench**：工具使用评估
- **WebArena**：网页浏览任务
- **SWE-bench**：软件工程任务
- **GAIA**：通用 AI 助手评估
- **HumanEval / MBPP**：代码生成（有限的智能体评估）
- **ALFWorld**：具身家务任务
- **HotPotQA**：多跳问答

## 交叉参考（本知识库）

本文件提供了统一的标注概述。详细的单篇论文笔记请参见：
- `papers/agent/_index.md` -- 本仓库所有智能体论文的主索引
- `papers/agent/survey/llm_agent_survey_fudan.md` -- 详细的复旦综述笔记
- `papers/agent/survey/llm_agent_survey_renmin.md` -- 详细的人民大学综述笔记
- `papers/agent/memory/coala.md` -- CoALA 论文笔记
- `papers/agent/memory/reflexion.md` -- Reflexion 论文笔记
- `papers/agent/memory/memgpt.md` -- MemGPT 论文笔记
- `papers/agent/foundational/react.md` -- ReAct 论文笔记
- `papers/agent/foundational/chain_of_thought.md` -- CoT 论文笔记
- `papers/agent/foundational/tree_of_thoughts.md` -- ToT 论文笔记
