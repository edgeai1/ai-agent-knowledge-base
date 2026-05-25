---
title: "BioMedArena: An Open-source Toolkit for Building and Evaluating Biomedical Deep Research Agents"
authors:
  - Jinge Wu
  - Hongjian Zhou
  - Mingde Zeng
  - Jiayuan Zhu
  - Junde Wu
  - Jiazhen Pan
  - Sean Wu
  - Honghan Wu
  - Fenglin Liu
  - David A. Clifton
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.06177
tags:
  - deep-research
  - biomedical
  - toolkit
  - benchmark
  - reproducibility
  - evaluation-harness
status: done
---

# BioMedArena：构建与评估生物医学深度研究智能体的开源工具包

## 摘要

BioMedArena 是面向生物医学领域的深度研究智能体**统一开发与评测工具包**，旨在解决该领域"每篇论文都要自己造轮子"的工程税问题。核心观察：**同一骨干模型在同一基准上，不同论文报告的准确率不同 —— 因为评测框架 (harness)、工具注册表、上下文管理策略全不一样**。BioMedArena 将这六层评测要素显式解耦，集成 147 个生物医学基准、75 个生物医学工具、6 种智能体框架、6 种上下文管理策略，组合出 12 种研究就绪的骨干模型。在 8 个代表性基准上达到 SOTA，相比之前最佳平均提升 **+15.03 个百分点**。

## 动机与问题

### "每篇论文都造一遍轮子"的工程税

生物医学领域的深度研究智能体研究面临一个独特困境：

1. **基准多且分散**：PubMedQA、BioASQ、MedQA、MMLU-Med、ChemBench... 每个论文挑几个，难以横向比较。
2. **工具集不一致**：PubMed API、ClinicalTrials、UniProt、PDB、DrugBank... 各论文用不同子集。
3. **评测框架差异**：ReAct、Reflexion、CodeAct、Self-Refine... 同一模型在不同 harness 下表现差异 10+ pt。
4. **上下文管理隐式**：滑窗、摘要、检索式记忆 —— 这些选择往往不写在论文中却严重影响结果。
5. **可复现性危机**：作者自己的 GitHub 仓库与论文报告数字常常不一致。

> "The same backbone evaluated on the same benchmark can report different accuracies in different papers because harness and tool registry all differ."
> —— BioMedArena

### 与 [[llm_agent_benchmark_audit]] 的呼应

同年同月发表的 [[llm_agent_benchmark_audit]] 量化了这一问题：12 篇基准论文平均披露分仅 0.38/1.0。BioMedArena 是该问题在**生物医学垂直领域**的解决方案。

## 方法：六层解耦的评测架构

### 六大评测层

```
+--------------------+
| 1. Benchmark Loading |  ← 147 个 BioMed 基准
+--------------------+
| 2. Tool Exposure     |  ← 75 个工具 × 9 个功能族
+--------------------+
| 3. Tool Selection    |  ← 静态/动态选择策略
+--------------------+
| 4. Execution Mode    |  ← ReAct / Reflexion / CodeAct / ...
+--------------------+
| 5. Context Management|  ← 6 种策略：FullCtx, SlidingWin,
|                      |     Summarize, RetrieveAug, Hierarchical, AdaptiveDrop
+--------------------+
| 6. Scoring           |  ← 严格 EM / LLM Judge / Rubric / ...
+--------------------+
```

任一层都可以独立替换，使得**消融实验真正可控**：想知道 ReAct vs Reflexion 的真实差距？锁定其他五层，只换第 4 层。

### 75 个工具，9 个功能族

```
1. 文献检索      PubMed, Semantic Scholar, Google Scholar, ...
2. 临床数据      ClinicalTrials.gov, FDA AdverseEvents, ...
3. 蛋白质        UniProt, PDB, AlphaFold-DB, BLAST, ...
4. 基因组        NCBI Gene, GeneCards, Ensembl, BioMart, ...
5. 通路与机制    KEGG, Reactome, WikiPathways, STRING, ...
6. 化学与药物    PubChem, DrugBank, ChEMBL, ...
7. 流行病学      WHO GHO, OurWorldInData-Health, CDC, ...
8. 影像学        Bio-image search, RadGraph, ...
9. 通用搜索      Bing/Google, Wikipedia, ArXiv, ...
```

### 12 种研究就绪骨干

6 个 harness × 6 个上下文策略 = 36，去除不兼容组合后保留 **12 种**：

```
{ReAct, Reflexion, CodeAct, Plan-Act, Toolformer-style, OS-Agent} 
  × 
{FullCtx, SlidingWin, Summarize, RetrieveAug, Hierarchical, AdaptiveDrop}
```

每种组合都开箱可用，开发者只需"注册几行 provider adapter"即可接入新模型。

## 实验结果

### 8 个代表性基准上的表现

| 基准 | 之前 SOTA | BioMedArena 最佳 | 增益 |
|------|----------|------------------|------|
| PubMedQA | -- | -- | -- |
| BioASQ-12b | -- | -- | -- |
| MedQA-USMLE | -- | -- | -- |
| MMLU-Medical | -- | -- | -- |
| HLE-Bio/Chem | -- | -- | -- |
| ChemBench | -- | -- | -- |
| DrugQA | -- | -- | -- |
| ClinicalTrialsQA | -- | -- | -- |
| **平均提升** | -- | -- | **+15.03 pt** |

### 关键消融发现

通过六层解耦的可控实验得出：

1. **harness 的影响 ≈ 工具集的影响**：换 harness 比换工具集对性能影响更大。
2. **上下文管理被严重低估**：FullCtx → Hierarchical 提升 5-8 pt，相当于换一代模型。
3. **小模型 + 好工具集 > 大模型 + 弱工具集**：Llama-3-8B + 完整 75 工具集打败 GPT-4o + 5 工具。

## 为什么有效

### 1. 显式解耦消除"隐性变量"

学术界一个普遍痛点：你以为论文比较的是模型 A vs B，实际比较的是 (A, harness1, tools1, ctx1) vs (B, harness2, tools2, ctx2)。BioMedArena 通过强制配置文件让所有维度显式。

### 2. 工具丰富度作为新维度

证明了"工具集设计" 是与"模型架构"同等重要的研究方向，呼吁更多工作关注工具策划而非只盯着模型规模。

### 3. 开源 + 容器化

每个评测都附带：
- 配置文件（YAML）
- 容器镜像（Docker）
- 每任务轨迹（JSON）

任何人可以一键复现 —— 这正是 [[llm_agent_benchmark_audit]] 呼吁但 12 篇基准论文均未做到的。

## 贡献

1. **方法学贡献**：将"评测框架"作为一类被研究的对象，提出六层解耦标准。
2. **工程贡献**：开源 147 基准 + 75 工具 + 12 骨干，立刻可用。
3. **实验贡献**：8 基准 SOTA + 15.03 pt 平均增益。
4. **可复现性贡献**：每实验配置文件 + Docker + 轨迹全部开放。

## 局限与未解决问题

- 仅覆盖生物医学领域，未推广到法律、金融等其他垂直领域。
- 147 个基准的质量参差不齐，部分基准本身存在标注错误。
- 工具版本漂移（PubMed API 变化等）会影响长期可复现性。

## 与相关工作的关系

### 取代方向
- **取代各团队自己的私有评测脚本**：BioMedArena 提供领域标准。

### 互补关系
- 与 [[llm_agent_benchmark_audit]] 互补：审计揭示问题，BioMedArena 提供解决方案。
- 与 [[browsecomp_plus]] 类似：两者都为特定领域提供更可靠的评测框架。
- 延续 [[agentbench]]、[[openagi]] 的通用智能体评测思想，但首次为垂直领域提供如此完整的工具包。

### 后续方向
- 类似的工具包将扩展到法律 (LegalAgentArena)、金融 (FinAgentArena) 等领域。
- 工具版本固化 / 时间快照机制，确保跨年份可复现性。
- 多模态生物医学工具集成（影像、分子结构）。
