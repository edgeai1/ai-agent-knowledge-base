---
title: "DeepResearcher: Scaling Deep Research via Reinforcement Learning in Real-world Environments"
authors:
  - Yuxiang Zheng
  - Lyumanshan Ye
  - Kaiwen Liu
  - Jianbo Yuan
  - Minghao Shao
  - Wenyuan Xu
  - Xuming Hu
  - Zhiwei Jia
  - Kevin Chen-Chuan Chang
  - Graham Neubig
  - Huan Sun
venue: EMNLP 2025
year: 2025
url: https://arxiv.org/abs/2504.03160
tags:
  - deep-research
  - reinforcement-learning
  - web-search-agent
  - GRPO
  - emergent-behaviors
  - end-to-end-training
status: done
---

# DeepResearcher：通过真实环境中的强化学习扩展深度研究

## 摘要

DeepResearcher 是首个端到端强化学习框架，用于训练与真实网络搜索环境交互的深度研究智能体。该系统基于 Qwen2.5-7B-Instruct 构建，通过 GRPO 和仅结果奖励进行训练，在开放域问答任务上比提示工程基线高出 +28.9 分，比基于 RAG 的强化学习智能体高出 +7.2 分。核心发现是：在真实网络环境中进行端到端强化学习能够产生涌现性认知行为（规划、交叉验证、自我反思、诚实弃权），这些行为无法仅通过监督微调实现。

## 动机与问题

现有构建深度研究智能体的方法分为两类：
1. **提示工程方法** -- 手工设计智能体工作流（如带工具使用指令的思维链），但无法通过经验进行适应或改进。
2. **基于 RAG 的强化学习方法** -- 使用强化学习训练，但采用静态的、策划好的检索语料库而非实时网络搜索，导致与实际部署之间存在分布偏差。

两种方法都无法端到端地学习进行真正的多步网络研究。DeepResearcher 通过仅使用基于结果的奖励在真实网络环境中直接训练智能体来弥补这一差距，让智能体自主发现研究策略。

## 方法

### 架构

```
+------------------+     +------------------+     +------------------+
| Qwen2.5-7B-Inst  |---->| Think / Reason   |---->| Generate Query   |
| (Policy Model)   |     | (internal CoT)   |     | <search>q</search>|
+------------------+     +------------------+     +------------------+
                                                          |
                                                          v
                                                  +------------------+
                                                  | Real Web Search  |
                                                  | (Bing/Google API)|
                                                  +------------------+
                                                          |
                                                          v
                                                  +------------------+
                                                  | Parse & Integrate|
                                                  | Search Results   |
                                                  +------------------+
                                                          |
                                                          v
                                                  +------------------+
                                                  | Continue Thinking|
                                                  | or Produce Answer|
                                                  +------------------+
```

### 训练流程（GRPO）

```
Algorithm: DeepResearcher Training via GRPO
-------------------------------------------------
Input: Base model pi_0, question set Q, reward fn R
Output: Trained policy pi_theta

1. for each training iteration t:
2.   Sample batch of questions q ~ Q
3.   for each q, generate G rollouts using pi_theta:
4.     Each rollout = interleaved (think, search, observe)* + answer
5.     Searches hit REAL web APIs (Bing/Google)
6.   Compute outcome reward r_i = R(answer_i, gold_answer)
7.   Compute group-relative advantages:
8.     A_i = (r_i - mean(r_1..G)) / std(r_1..G)
9.   Update pi_theta via clipped policy gradient (GRPO objective)
10.  Apply KL penalty against reference policy pi_ref
```

### 奖励设计

- **仅结果奖励**：二值正确性信号（F1 或与标准答案的精确匹配）。
- 不对搜索质量、推理步骤或工具使用提供中间奖励。
- 智能体必须纯粹从最终答案信号中发现有效的搜索策略。

### 数据策划（两阶段过滤）

1. **质量过滤**：移除时效性强、主观性或有害的问题。
2. **污染过滤**：在无搜索条件下测试基础模型（pass@10）。排除模型已能回答的问题，确保智能体学习真正的研究技能。

### 训练基础设施

- **框架**：veRL（分布式强化学习框架）。
- **搜索并行性**：50 个 CPU 节点集群，用于 GRPO 推演期间的并发网络搜索。
- **缓存**：相同搜索查询的 7 天 TTL 缓存，以降低 API 成本。
- **重试机制**：对速率限制和瞬时爬取失败的鲁棒处理。

## 关键创新

1. **首个在真实网络环境中的端到端强化学习**：与基于 RAG 的强化学习（Search-R1、R1-searcher）不同，DeepResearcher 在实时网络搜索上训练，消除了语料库与部署之间的分布偏差。

2. **涌现性认知行为**（未经显式训练）：
   - **规划**：智能体制定并动态调整多步研究计划；在适当时可以合并步骤。
   - **交叉验证**：在找到答案后，智能体在做出结论前会从独立来源搜索佐证。
   - **自我反思**：智能体识别搜索策略失败并重新调整研究方向。
   - **诚实弃权**：当证据不足时，智能体承认不确定性而非编造答案。

3. **仅使用结果奖励的可扩展训练**：证明了复杂的研究行为可以从简单的二值奖励信号中涌现，无需奖励塑形。

## 实验设置

### 数据集

| 划分        | 数据集                                    | 跳数类型    |
|-------------|---------------------------------------------|-------------|
| 域内   | NQ, TriviaQA, HotpotQA, 2WikiMultiHopQA    | 单跳/多跳|
| 域外| MuSiQue, Bamboogle, PopQA                  | 多跳   |

### 对比基线

- **基于提示的方法**：直接提示、ReAct、Search-o1
- **基于 SFT 的方法**：在搜索轨迹上微调
- **基于 RAG 的强化学习**：Search-R1、R1-Searcher（使用静态语料库，非实时网络）

### 评估指标

- **F1 分数**：与标准答案的词元级重叠。
- **基于模型的评估（MBE）**：以 LLM 作为评判者进行语义正确性评估（被认为更可靠）。

## 结果

### 主要性能（MBE 指标）

- 比最佳提示工程基线高出 **+28.9 分**（跨基准平均）。
- 比最佳基于 RAG 的强化学习智能体高出 **+7.2 分**（Search-R1 / R1-Searcher）。
- 在所有四个域内数据集（NQ、TQ、HotpotQA、2Wiki）上取得最高 MBE 分数。
- 在所有三个域外数据集上持续领先，展现了泛化能力。

### 关键对比

- DeepResearcher 对比基于提示的 ReAct：巨大差距表明强化学习习得的策略远超手工设计的提示工作流。
- DeepResearcher 对比基于 RAG 的强化学习智能体：7.2 分的差距验证了真实网络训练至关重要，而非仅在静态语料库上进行强化学习训练。
- 在需要信息综合的多跳任务（HotpotQA、2Wiki）上表现尤为突出。

## 局限性

1. **计算成本**：训练需要 50 个 CPU 节点进行并行网络搜索加上 GPU 集群进行 GRPO，使得复现成本高昂。
2. **搜索 API 依赖**：结果取决于搜索引擎质量；API 速率限制制约了推演吞吐量。
3. **7B 模型规模**：仅在 Qwen2.5-7B 上进行了验证；向更大模型的扩展行为未知。
4. **基准范围**：在事实型问答基准上评估；未在长文本报告生成或复杂分析任务上测试。
5. **奖励稀疏性**：仅结果奖励可能导致在更难任务上的收敛速度缓慢，而这些任务本可从部分奖励中受益。

## 后续工作

- **PokeeResearch**：通过额外的奖励塑形扩展基于强化学习的深度研究。
- **DeepDive**：将知识图谱与多轮强化学习相结合，实现更深层的搜索轨迹。
- **WebThinker**：在推理循环中添加报告生成和深度网络探索器。
- **OpenResearcher**：采用监督蒸馏方法，利用离线轨迹合成作为在线强化学习的替代方案。

## 核心要点

1. 在真实网络环境中的端到端强化学习"不仅是实现细节，更是构建鲁棒深度研究智能体的根本要求"。
2. 复杂认知行为（规划、交叉验证、自我反思）在仅结果强化学习中自然涌现，无需对这些技能进行显式监督。
3. 静态检索语料库与实时网络搜索之间的分布差距足够显著，证明了真实网络强化学习训练的额外复杂性是合理的。
4. 一个 7B 参数的模型在经过适当的强化学习训练后，可以在多步研究任务上匹配或超越更大的提示工程系统。
5. 带有分布式搜索并行性的 veRL 框架为使用工具智能体进行强化学习训练的扩展提供了实用模板。
