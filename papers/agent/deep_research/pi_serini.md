---
title: "Rethinking Agentic Search with Pi-Serini: Is Lexical Retrieval Sufficient?"
authors:
  - Tz-Huan Hsu
  - Jheng-Hong Yang
  - Jimmy Lin
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.10848
tags:
  - deep-research
  - agentic-search
  - lexical-retrieval
  - BM25
  - retrieval-evaluation
  - BrowseComp-Plus
status: done
---

# Pi-Serini：重新思考代理式搜索 —— 词法检索是否足够？

## 摘要

Pi-Serini 提出一个**反直觉的发现**：随着 LLM 能力提升，**好好调过的 BM25 词法检索器配合前沿 LLM**，在深度研究任务上反而超过用密集检索器的同类系统。在 BrowseComp-Plus 基准上达到**答案准确率 83.1%、证据召回率 94.7%**。该结果颠覆了"必须用 dense retriever 才能做好 deep research"的共识，呼吁社区重新审视检索器与 LLM 的分工。

## 动机与问题

### 主流共识：密集检索器是 deep research 的标配

过去三年，深度研究领域几乎所有顶级系统都使用**密集检索器** (dense retriever)：
- OpenAI Deep Research：内部 embedding 模型
- WebDancer、WebSailor：DPR / Contriever
- DeepResearcher、Search-R1：训练专门的 dense retriever
- 多数学术系统：BGE、E5、Contriever

**默认假设**：BM25 这种 1990 年代的词法方法对深度研究"太弱了"。

### 但前沿 LLM 已经改变了游戏规则

研究者提出疑问：当 LLM 本身具有强大的**改写查询**、**判断相关性**、**多轮推理**能力时，检索器的"语义理解能力"是否还像过去那样重要？

具体来说：
- 过去：BM25 找不到含义相同但词形不同的文档（"汽车" vs "车辆"），需要 dense 来补。
- 现在：LLM 可以主动改写查询为多种词形（"汽车 OR 车辆 OR 机动车"），BM25 同样能找到。

如果上述假设成立，**对密集检索器的需求被过度夸大**，行业可以省下大量训练 retriever 的成本。

## 方法：Pi-Serini 系统

### 三大工具

Pi-Serini 是基于 [Pyserini](https://github.com/castorini/pyserini)（BM25 标准库）的搜索智能体：

```
+---------------+
|  Retrieve     |  ← BM25 检索 top-K 文档
+---------------+
|  Browse       |  ← 浏览文档元数据（标题、摘要）
+---------------+
|  Read         |  ← 读取文档全文
+---------------+
```

智能体（基于前沿 LLM）使用这三个工具，**多轮迭代**地查询、浏览、阅读，直到找到答案。

### 关键设计：检索深度

Pi-Serini 强调一个常被忽视的参数 —— **检索深度** (retrieval depth)：

```
top-K = 5  → 召回率有限，错过关键文档
top-K = 50 → 大幅提升召回率（+25.3%）
top-K = 100 → 边际收益递减
```

论文实验显示，**仅仅增加检索深度就能提升召回率 25.3%**，这一参数过去被严重低估。

### BM25 调参

Pi-Serini 仔细调优了 BM25 的两个关键参数：

```
BM25(q, d) = Σ IDF(q_i) × f(q_i, d) × (k1+1) / (f(q_i, d) + k1 × (1-b+b × |d|/avgdl))
```

- **k1**：词频饱和参数（默认 1.2，Pi-Serini 调到 ~0.9）
- **b**：文档长度归一化（默认 0.75，Pi-Serini 调到 ~0.4）

仅调参就带来 **+18.0% 准确率，+11.1% 召回率**。

## 实验结果

### BrowseComp-Plus 主结果

| 系统 | 检索器 | LLM | 准确率 | 召回率 |
|------|--------|-----|--------|--------|
| Pi-Serini | **BM25 (tuned)** | GPT-5.5 | **83.1%** | **94.7%** |
| OpenResearcher | Dense | GPT-4o | ~70% | ~85% |
| WebDancer | Dense | Qwen-72B | ~68% | ~82% |
| DeepResearcher | Dense | Qwen-7B | ~62% | ~78% |

### 消融实验：BM25 调参 vs 深度

| 设置 | 准确率 | 召回率 |
|------|-------|-------|
| BM25 默认 + top-10 | 60.0% | 60.0% |
| BM25 调参 + top-10 | 78.0% | 71.1% |
| BM25 调参 + top-50 | 81.5% | 89.0% |
| BM25 调参 + top-100 | **83.1%** | **94.7%** |

**单独调参提升 +18%，单独加深度提升 +25.3%，组合后达到 SOTA。**

### 跨 LLM 验证

Pi-Serini 配置在不同前沿 LLM 上都接近 SOTA：

```
GPT-5.5:         83.1%
Claude Mythos:   80.4%
Gemini Ultra:    78.9%
DeepSeek-R2:     77.3%
```

证明**这不是 GPT-5.5 的功劳，而是检索 + LLM 协同的胜利**。

## 为什么有效

### 1. LLM 是好的查询改写器

```
用户问："2024 年新能源车销量"
弱 LLM 用 BM25: 只搜 "2024 新能源车 销量" → 命中差
强 LLM 用 BM25: 改写为
  - "2024 新能源汽车 销量"
  - "2024 电动车 销售数据"  
  - "EV sales 2024 China"
  - ...
→ BM25 召回 == dense retriever 召回
```

LLM 的查询扩展能力**抹平了 BM25 与 dense 的语义差距**。

### 2. BM25 的优势被低估

- **零训练成本**：dense retriever 需要数百万对的对比学习数据。
- **可解释**：每个文档为何被检索清晰可查（哪些词命中）。
- **跨域稳定**：dense retriever 在 OOD（域外）查询上性能急剧下降，BM25 不受影响。
- **快**：BM25 索引检索快于 dense 数十倍。

### 3. 深度检索 + LLM 重排 == 最佳组合

```
BM25 (top-100) →  快速召回候选
       ↓
LLM (评估相关性)  →  精排取 top-5
       ↓
LLM (推理)        →  生成答案
```

这种"召回宽广 + LLM 精排"在前沿 LLM 时代效率最高。

## 贡献

1. **反直觉发现**：BM25 + 前沿 LLM ≥ dense retriever + LLM。
2. **方法论澄清**：BM25 调参和检索深度被严重低估，单独优化每个就能带来巨大提升。
3. **节省训练成本**：研究社区可以省下训练 dense retriever 的资源，将算力投入更核心问题（推理、规划）。
4. **可解释检索**：BM25 的命中可追溯性对审计和调试至关重要。
5. **开源**：完整代码 + 配置在 GitHub。

## 局限与未解决问题

- 仅在 BrowseComp-Plus 上验证，需要更多基准跨域验证。
- 对**低资源语言**和**专业术语稀少的领域**未做评估。
- 多模态检索（图、表）的可行性未讨论。

## 与相关工作的关系

### 直接挑战
- **挑战 Dense Retriever 路线**：以 DPR、Contriever、E5、BGE 为代表的密集检索流派。
- **呼应 BM25 复兴**：与近年来"BM25 + LLM rerank"的趋势一致。

### 互补关系
- 与 [[deepweb_bench]] 互补：DeepWeb-Bench 揭示"检索不是瓶颈"，Pi-Serini 给出具体方法 —— 因为 BM25 + LLM 已经能解决检索问题。
- 与 [[sira]] 互补：SIRA 也提倡"单轮高质量检索"，与 Pi-Serini 思想呼应。
- 与 [[argus]] 互补：Pi-Serini 解决"如何高效检索"，Argus 解决"如何协调多个检索"。

### 后续方向
- 多语言 BM25 调参
- 自适应检索深度（根据查询复杂度调整 K）
- BM25 与 LLM 反馈的闭环（用 LLM 评估来微调 BM25 参数）
