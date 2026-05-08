---
title: "OpenHands: An Open Platform for AI Software Developers as Generalist Agents"
authors: Xingyao Wang, Boxuan Li, Yufan Song, Frank F. Xu, Xiangru Tang, Mingchen Zhuge, Jiayi Pan, Yueqi Song, Bowen Li, Jaskirat Singh, Hoang H. Tran, Fuqiang Li, Ren Ma, Mingzhang Zheng, Bill Qian, Yanjun Shao, Niklas Muennighoff, Ziwen Liu, Dong Guo, Qian Liu, Meng Cao, Haoran Zhang, Yanzhi Zhang, Shuai Lu, Wenwen Qu, Haochuan Wang, Jiahui Zhang, Zhao Xu, Tianyu Liu, Samuel Arcadinho, Zhoujun Cheng, Chengcheng Han, Yuchen Eleanor Jiang, Suyash Vardhan Mathur, Manish Shetty, Wen-Ding Li, Yixin Ou, Yunxiang Li, Robert Brennan, Jun Shern Chan, Jenny Ma, Boyuan Zheng, Xizhou Zhu, Zhiyuan Liu, Jie Tang, Graham Neubig, Yuxiao Dong
affiliation: All Hands AI, Carnegie Mellon University, University of Illinois, multiple institutions
venue: ICLR 2025
year: 2024-2025
url: https://arxiv.org/abs/2407.16741
code: https://github.com/OpenHands/OpenHands
project: https://www.openhands.dev
sdk_paper: https://arxiv.org/abs/2511.03690
tags: [coding-agent, open-source, platform, sandbox, model-agnostic, SWE-bench, event-stream]
status: done
license: MIT
stars: 68K+ GitHub stars
funding: $18.8M Series A (All Hands AI)
---

## 摘要

OpenHands（原 OpenDevin）是一个 MIT 许可证的开源平台，用于构建 AI 软件开发智能体，发表于 ICLR 2025。基于事件流架构和 Docker 沙箱执行构建，它通过 LiteLLM 提供模型无关框架，支持 100+ LLM 提供商。默认的 CodeAct 智能体结合代码执行与推理，取得了最先进结果：使用 CodeAct 2.1（Claude 3.5 Sonnet）在 SWE-bench Verified 上达到 53%，配合 Claude Sonnet 4.5 扩展思考达到 72%。OpenHands 证明了开源、模型无关的智能体平台可以匹配或超越 Devin 等专有垂直解决方案。

## 研究动机与问题

到 2024 年中期，编码智能体领域被两个极端主导：

1. **专有垂直解决方案（Devin 等）**：功能强大但闭源、昂贵、供应商锁定，且科学上不透明。研究者无法检查、修改或重现结果。
2. **研究原型（SWE-Agent 等）**：学术上严谨但范围狭窄，难以在生产中部署，且绑定特定模型架构。

OpenHands 解决了几个关键缺口：

- **缺少开放的智能体开发平台**：构建编码智能体需要为每个新项目从零开始重新实现沙箱、工具使用、网络浏览和评估基础设施
- **模型锁定**：大多数智能体系统被硬编码到特定 LLM（通常是 GPT-4 或 Claude），阻止了跨模型比较和灵活性
- **缺少可复用的智能体 SDK**：所有软件智能体共需的通用基础设施（沙箱、事件日志、状态管理、API 服务）被每个项目独立重建
- **可重现性危机**：专有智能体（Devin）发布的基准数字无法被独立验证或重现
- **社区碎片化**：数十个独立编码智能体项目具有不兼容的接口，阻止了协作和比较

## 方法：架构

### 三个核心组件

OpenHands 由三个主要组件组成，共同构成完整的智能体开发和执行平台：

#### 1. 智能体抽象层（AgentHub）

多种智能体实现可以在同一平台内贡献和比较：

- **CodeAct 智能体**（默认）：主要智能体，结合代码执行与推理。使用函数调用通过 Python 代码和 bash 命令与环境交互。
- **浏览器智能体**：使用 BrowserGym 专注于网络浏览任务
- **委托智能体**：为需要委托的复杂任务编排子智能体
- 社区可以以最少的样板代码贡献新的智能体实现

#### 2. 事件流（状态管理核心）

事件流是核心架构抽象——所有动作和观察的按时间排序的、仅追加的日志：

```
事件流架构:

  用户指令     -> [事件 1: MessageAction]
  智能体响应   -> [事件 2: AgentAction(code="ls -la")]
  沙箱结果     -> [事件 3: Observation(stdout="...")]
  智能体响应   -> [事件 4: AgentAction(code="edit file.py")]
  沙箱结果     -> [事件 5: Observation(file_edited=True)]
  ...

  State = f(Event Stream) -- 状态始终可从事件日志中推导
```

事件溯源设计的属性：
- **不可变历史**：所有动作和观察被永久记录，支持重放、调试和分析
- **确定性状态恢复**：当前状态始终可通过从头重放事件流来重建
- **审计追踪**：每个智能体决策和环境变化的完整记录
- **多智能体协调**：多个智能体可以读写同一事件流
- **会话持久性**：工作可以通过加载事件流来暂停和恢复

状态数据结构包括：事件流本身、累积 LLM 调用成本、多智能体委托追踪的元数据以及执行相关参数。

#### 3. 运行时（沙箱执行环境）

每个智能体会话在安全隔离的 Docker 容器中运行，提供：

**Bash Shell**：完整 Linux 环境用于命令执行
- 通过容器内运行的 REST API 服务器连接
- 在整个任务中维护持久的 bash shell 会话
- 智能体发送命令，接收 stdout/stderr 输出

**IPython/Jupyter 内核**：用于 Python 代码执行
- 支持运行安装了包的 Python 代码
- 支持数据分析、文件操作和编程操作
- 提供比 bash 更丰富的输出（DataFrame、图表、结构化数据）

**BrowserGym 接口**：用于网络自动化
- 使用 BrowserGym 的领域特定语言进行完整浏览器交互
- 支持搜索文档、浏览网站和与 Web 应用交互
- 支持页面导航、元素交互和内容提取

**工作区挂载**：可配置的工作区目录被挂载到沙箱中
- 用户指定智能体应处理的文件/目录
- 文件在会话中持久存在并可被智能体访问
- 智能体所做的更改反映在挂载的目录中

**安全模型**：一次性 Docker 容器作为主要隔离机制
- 每个会话获得一个新容器，无法访问宿主系统
- 会话完成后容器被销毁
- 网络访问可按安全需求限制
- SSH 访问可用于调试

### CodeAct 智能体设计

CodeAct 智能体是默认的"强通用"智能体，实现了特定的智能体-环境交互范式：

**核心原则**：智能体不生成结构化工具调用或 JSON 动作，而是编写和执行 Python 代码或 bash 命令来与环境交互。这就是"CodeAct"范式——使用代码作为通用动作语言。

**CodeAct 1.0 -> 2.1 演进：**

| 版本      | 关键变更                                          | SWE-bench Verified |
|----------|---------------------------------------------------|--------------------|
| v1.0     | 基于 IPython 的代码执行，基于提示的动作            | ~25%               |
| v1.8     | Claude 3.5 Sonnet，改进的提示                      | 26%（Lite）        |
| v2.1     | 函数调用，Claude 3.5（10 月版），目录修复           | **53%**            |
| + Claude 4| 扩展思考，最新前沿模型                             | **72%**            |

**CodeAct 2.1 改进**（2025 年 11 月）：
1. 从基于提示的动作切换到函数调用，以更可靠地生成动作
2. 更新到 Anthropic 新的 Claude 3.5 Sonnet 模型（2024 年 10 月发布版）
3. 多项修复使智能体更容易遍历和理解目录结构
4. 更好的错误处理和恢复机制

### 通过 LiteLLM 实现模型无关设计

OpenHands 通过 LiteLLM 集成独特地支持任何 LLM 提供商：

- **支持 100+ 提供商**：OpenAI、Anthropic、Google、Mistral、Cohere、Together、Groq、本地模型（Ollama、vLLM）等
- **无需自定义集成**：添加新提供商只需 API 密钥，无需代码更改
- **非函数调用回退**：对缺乏函数调用能力的模型提供一等支持，通过基于提示的动作生成
- **统一接口**：相同的智能体代码适用于任何模型，支持跨提供商直接比较
- **成本追踪**：跨不同提供商和定价模型的自动成本计算

这种模型无关设计是与 Devin（锁定专有 SWE-1/1.5）和大多数其他智能体框架的关键差异化因素。

### 服务器架构

OpenHands 服务器提供三个接口：
- **Web UI**：基于浏览器的智能体交互，包括实时工作区查看
- **REST API**：用于与 CI/CD 流水线和外部工具集成的编程访问
- **WebSocket API**：智能体动作和观察的实时流

附加接口：
- **VS Code 集成**：沙箱内的远程开发环境访问
- **VNC 访问**：用于 GUI 任务的可视桌面访问
- **浏览器视图**：Web 自动化任务的实时浏览器状态

## 关键创新

1. **事件溯源的智能体架构**：不可变事件流作为单一事实来源，支持重放、调试、审计和多智能体协调
2. **模型无关设计**：首个支持 100+ LLM 提供商的主要智能体平台，无需自定义集成工作，包括非函数调用模型的回退
3. **CodeAct 范式**：使用可执行代码而非结构化工具调用作为通用动作语言——更灵活和富有表现力
4. **开放的智能体研究平台**：社区贡献的智能体、基准和改进；188+ 贡献者，68K+ GitHub star
5. **生产级沙箱**：基于 Docker 的隔离，在可复用、可配置的运行时中提供 SSH、Jupyter 和浏览器访问

## 实验设置

### 评估的基准

ICLR 2025 论文在 15 个挑战性任务上评估 OpenHands 智能体：
- **SWE-bench**：真实 GitHub issue 解决（主要编码基准）
- **WebArena**：真实环境中的网络导航
- **GAIA**：通用 AI 助手任务
- **多个附加基准**：涵盖软件工程和网络交互

### 模型比较（在 SWE-bench Verified 上）

| 配置                                    | 解决率      |
|----------------------------------------|-------------|
| OpenHands CodeAct v1.8 + Claude 3.5    | 26%（Lite） |
| OpenHands CodeAct 2.1 + Claude 3.5     | **53%**     |
| OpenHands + Claude 3.7 Sonnet          | ~43%*       |
| OpenHands + Claude Sonnet 4.5（思考）  | **72%**     |
| OpenHands + Claude 4                   | ~72%        |

*注：43% 为 SWE-bench Verified 子集重新运行；在 SWE-bench Live 上为 19.25%

## 结果

### SWE-bench 性能轨迹

OpenHands 展现了持续改进，由智能体架构优化和底层模型改进共同驱动：

- **CodeAct 2.1（2025 年 11 月）**：在 SWE-bench Verified 上 53%——首个开源智能体超越 50%，超过 Claude 3.5 独立的 49%
- **配合 Claude Sonnet 4.5 扩展思考**：72%——在 SWE-bench Verified 上任何智能体平台的最高分之列
- **SWE-bench Live**：~19.25%（凸显了静态 vs. 实时基准的差距）

### 跨基准泛化

OpenHands 在 SWE-bench 之外展现了强劲结果：
- 在 WebArena（网络导航）上有竞争力的表现
- 在 GAIA（通用助手任务）上的强劲结果
- 架构的通用性使其能在不同模型提供商和任务类型间一致运行

### SWE-bench Live：现实检验

SWE-bench Live 在新的、此前未见的 issue 上测试智能体，以防止数据污染：
- OpenHands + Claude 3.7 Sonnet：在 SWE-bench Verified 上 43.20%，在 SWE-bench Live 上 19.25%
- 这 24 个百分点的下降是静态和实时基准性能之间记录的最大差距
- 表明高静态基准分数的一部分可能反映了训练/测试数据污染或对基准分布的过度拟合

### OpenHands Index

作为编码智能体的标准化评估框架引入，超越 SWE-bench：
- 跨多个维度评估智能体（不仅是缺陷修复）
- 包括代码生成、重构和理解任务
- 旨在提供智能体编码能力的更全面图景
- 设计用于减少对单一基准的过度优化

## 分析与洞察

### 为什么模型无关很重要

OpenHands 的模型无关设计揭示了几个重要发现：
- **智能体脚手架与基础模型同样重要**：相同的 Claude 3.5 模型独立得分 49%，但使用 CodeAct 2.1 脚手架后达到 53%
- **模型改进与智能体改进叠加**：在相同脚手架内从 Claude 3.5 升级到 Claude 4 从 53% 跃升到 72%
- **开源可以匹配商业**：OpenHands 的最佳结果（72%）匹配或超越 Devin（45.8%），证明拥有前沿模型访问权加上良好脚手架击败专有垂直整合

### CodeAct 的优势

使用可执行代码作为动作语言（而非结构化 JSON 工具调用）提供：
- **组合性**：单个动作中的多个操作（变量赋值 + 文件读取 + 字符串处理）
- **可调试性**：标准编程调试工具适用
- **表现力**：任何 Python 或 bash 操作都是有效动作，不限于预定义的工具 schema
- **熟悉性**：智能体在与人类开发者相同的范式中运行

### 生产部署模式

OpenHands Agent SDK（2025 年 11 月）扩展平台用于生产使用：
- 不可变配置防止意外状态突变
- 事件溯源状态模型实现可靠的会话管理
- 工作区级别的远程接口（VS Code、VNC、浏览器）用于人类监督
- 可组合的智能体架构用于构建专业化智能体工作流

## 局限性

- **对底层模型的基准依赖**：OpenHands 的分数在很大程度上取决于底层 LLM 质量；平台放大模型能力但无法弥补基本的模型弱点
- **SWE-bench 过度拟合担忧**：静态和实时基准性能之间 24 个百分点的差距对报告性能中有多少是真实的 vs. 数据污染提出了质疑
- **资源开销**：Docker 沙箱为每个会话增加启动延迟和资源开销
- **简单任务的复杂度**：完整平台对简单代码生成来说过度工程化；轻量级替代方案更适合单文件编辑
- **浏览器自动化局限**：基于 BrowserGym 的网络交互不如专用浏览器自动化工具稳健
- **安全面**：即使有 Docker 沙箱，运行智能体生成的任意代码在生产部署中仍存在安全考量
- **社区协调挑战**：拥有 188+ 贡献者，维护质量和架构一致性是持续的挑战

## 核心要点

1. **开源智能体平台可以匹配或超越专有解决方案**：OpenHands 在 SWE-bench Verified 上的 72% vs. Devin 的 45.8% 证明了拥有前沿模型访问权的模型无关平台胜过专有垂直整合。
2. **事件流架构是智能体系统的有原则基础**：不可变、仅追加的事件日志提供了临时状态管理无法匹配的重放、调试、审计和多智能体协调能力。
3. **模型无关设计对长期生命力至关重要**：随着模型快速改进（Claude 3.5 -> 4 使 SWE-bench 分数翻倍），锁定特定模型的平台将变得过时。OpenHands 的 100+ 提供商支持确保它从每次模型改进中受益。
4. **CodeAct（代码即动作）比结构化工具调用更具表现力**：执行 Python/bash 代码提供了预定义工具 schema 无法匹配的组合性、可调试性和表现力。
5. **静态和实时基准之间的差距（24 个百分点）是一个警告**：社区在评估真实世界智能体能力时应更重视 SWE-bench Live 结果而非静态验证结果。
6. **智能体脚手架放大模型能力**：相同底层模型使用良好智能体架构后分数显著更高（独立 49% vs. CodeAct 2.1 的 53%），证明模型质量和智能体设计都很重要。
7. **社区驱动的开发适用于智能体平台**：68K+ star、188+ 贡献者和 ICLR 2025 论文证明开源智能体开发可以同时实现研究卓越和生产质量。
8. **基于 Docker 的沙箱为大多数用例提供了足够的安全性**：一次性容器作为主要隔离机制在安全性和可用性之间取得平衡，尽管生产部署需要额外加固。
