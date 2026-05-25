---
title: "Rethinking How to Remember: Beyond Atomic Facts in Lifelong LLM Agent Memory"
authors:
  - Jingwei Sun
  - Jianing Zhu
  - Jiangchao Yao
  - Tongliang Liu
  - Bo Han
venue: Preprint
year: 2026
url: https://arxiv.org/abs/2605.19952
tags:
  - memory
  - lifelong-learning
  - LLM-agent
  - multi-granularity
  - TextGrad
  - long-term-interaction
status: done
---

# TriMem：重新思考记忆 —— 超越原子事实的终身 LLM 智能体记忆

## 摘要

TriMem 提出**三粒度共存的记忆架构**，挑战当前主流"将对话压缩为原子事实"的记忆设计。研究者发现：**单纯的事实抽取会丢失上下文与推理脉络**，导致复杂推理失败。TriMem 同时维护三种表示：**原始对话片段（带来源）、抽取的原子事实、综合的人物画像**，并通过 **TextGrad 提示优化**实现无参数更新的持续改进。在 LoCoMo 和 PerLTQA 基准上超越所有现有记忆基线。

## 动机与问题

### 主流记忆架构的"原子事实陷阱"

[[memgpt]]、[[coala]] 等代表性记忆系统的设计思想：

```
对话 → 抽取事实 → 存入数据库 → 检索时按事实查询
```

例如：
```
对话："上周二老板让我准备 X 项目报告，但我妈妈生病了所以没做"
抽取事实：
  - 老板分配 X 项目报告任务
  - 任务时间：上周二
  - 完成状态：未完成
  - 原因：母亲生病
```

### 这一设计的根本缺陷

1. **上下文丢失**：原子事实独立后，无法重建"完整情境"。
2. **推理能力下降**：复杂推理需要事实间的关联模式，仅事实清单不够。
3. **冗余生成**：每次对话都抽取一遍，相似事实大量重复。
4. **同主体重复**：关于同一人/事的事实分散，无综合画像。

### 真实记忆是多粒度的

人类记忆同时存在多个粒度：
- **情景记忆**：完整事件（"那次开会的细节"）
- **语义记忆**：抽象事实（"老板叫张三"）
- **画像记忆**：综合判断（"他通常对项目报告很严"）

TriMem 把这一认知科学洞察引入 LLM 智能体。

## 方法：TriMem 三粒度架构

### 三种表示

```
+----------------------------+
|  Layer 1: Raw Segments     |
|  原始对话片段 + 来源锚定    |
|  ✓ 保留完整上下文            |
+----------------------------+
|  Layer 2: Atomic Facts     |
|  抽取的原子事实             |
|  ✓ 支持快速检索              |
+----------------------------+
|  Layer 3: Synthesized      |
|             Profiles       |
|  综合人物/事件画像           |
|  ✓ 跨片段聚合，支持推理     |
+----------------------------+
```

### Layer 1: Raw Dialogue Segments

完整对话片段，加上：
- 时间戳
- 参与者
- 来源 ID（如会话 ID、turn ID）

**用途**：当需要"重建情境"时检索，如"详细回忆我们上次讨论 X 时的过程"。

### Layer 2: Atomic Facts

从对话中抽取的可查询事实：

```python
fact = {
  "id": "fact_001",
  "subject": "李四",
  "predicate": "在公司任职",
  "object": "产品经理",
  "source_segment_id": "seg_042",
  "confidence": 0.95,
  "extracted_at": "2026-05-20"
}
```

**用途**：快速回答"李四是什么职位？"

### Layer 3: Synthesized Profiles

综合多个事实和片段，形成关于实体的画像：

```python
profile = {
  "subject": "李四",
  "role": "产品经理",
  "characteristics": ["注重细节", "倾向数据驱动", "周一例会主持"],
  "relationships": {"汇报对象": "王经理", "协作": ["张三"]},
  "communication_style": "正式但友好",
  "supporting_evidence": ["fact_001", "fact_017", "fact_028"]
}
```

**用途**：支持复杂推理，如"我应该怎么向李四汇报这个坏消息？"

## 自适应优化：TextGrad

记忆系统的难点：**抽取/综合的提示词如何持续改进？**

传统方法需要参数微调，TriMem 用 [TextGrad](https://arxiv.org/abs/2406.07496)：

```
1. 在新对话上运行当前提示词 → 生成事实/画像
2. 用 LLM 评估生成质量（"这个事实是否完整、准确？"）
3. TextGrad 反向计算"提示词的文本梯度"
4. 用 LLM 改写提示词 → 新提示词
```

**全程无参数更新**，仅通过提示词优化让记忆系统进化。

## 实验结果

### 基准

| 基准 | 描述 |
|------|------|
| LoCoMo | 长对话记忆 QA |
| PerLTQA | 个性化长程问答 |

### 主结果

| 方法 | LoCoMo | PerLTQA |
|------|--------|---------|
| RAG (vanilla) | 41.2 | 38.7 |
| [[memgpt]] | 53.8 | 48.4 |
| Multi-vector | 56.9 | 51.2 |
| Atomic Facts only | 58.4 | 53.6 |
| **TriMem (无 TextGrad)** | **65.1** | **60.2** |
| **TriMem (full)** | **68.4** | **63.8** |

每一层都贡献显著：
- 仅 Layer 1：49.2
- Layer 1 + 2：58.4
- Layer 1 + 2 + 3：65.1
- 全部 + TextGrad：**68.4**

### 跨 LLM 验证

```
GPT-4o:     68.4
Claude-3.5: 67.2
Llama-3-70B: 64.8
Qwen-72B:    65.6
```

证明 TriMem **不依赖特定 LLM**，是架构层面的改进。

### 长程衰减

```
对话长度        Atomic-only    TriMem
1K  turns       72.3           74.8
10K turns       58.4           67.9
100K turns      31.2           58.4   ← TriMem 优势随时长扩大
```

**TriMem 在超长对话中优势愈发明显**。

## 为什么有效

### 1. 信息无损 + 检索高效

- Layer 1 保证"信息不丢"
- Layer 2 保证"检索快"
- Layer 3 保证"推理准"

三者互补 —— 不像单层方案必须在 trade-off 之间选。

### 2. Profile 支持高阶推理

```
问题："我应该怎么向李四汇报项目延期？"

仅有原子事实:
  "李四是产品经理" + "项目延期" → 无法回答

有 Profile:
  "李四注重细节，倾向数据驱动，沟通风格正式" 
  → "提供详细数据 + 正式邮件 + 备好解决方案"
```

### 3. TextGrad 持续进化

记忆系统的"理解力"随使用增强，无需重训练。这是终身学习的核心。

### 4. 来源锚定 = 可审计

每个事实/画像都能追溯到原始对话片段，避免幻觉。

## 贡献

1. **认知科学启发**：将"情景/语义/画像"三层记忆引入 LLM 智能体。
2. **多粒度共存**：抛弃"压缩为事实"的单一思路。
3. **TextGrad 集成**：无参数更新的持续优化。
4. **长程优势**：100K turns 时仍保持高准确率。
5. **跨 LLM 通用**：架构创新，不依赖特定模型。

## 局限与未解决问题

- 三层记忆的存储成本是单层的 2-3 倍。
- Profile 综合质量依赖底层 LLM，弱模型生成低质量 Profile。
- TextGrad 优化收敛速度未充分分析。

## 与相关工作的关系

### 直接挑战
- **挑战 [[memgpt]] 的单层 hierarchical memory**：MemGPT 用 working/main memory 区分容量，TriMem 区分**信息粒度**。
- **挑战 atomic-only memory**：[[coala]]、Generative Agents 等系统的单一抽取范式。

### 范式启发
- 认知科学：Tulving 的 episodic/semantic 记忆分类。
- 知识图谱：Profile 类似 KG 中的实体节点 + 属性聚合。

### 互补关系
- 与 [[memgpt]] 互补：MemGPT 解决"容量"，TriMem 解决"粒度"。
- 与 [[reflexion]] 互补：Reflexion 反思单次任务，TriMem 累积长程记忆。
- 与 [[hdri]] 思想相通：HDRI 的"主体锁定"机制对应 TriMem 的 Profile 层。

### 后续方向
- 多模态 TriMem：图像、视频记忆。
- 联邦 TriMem：多智能体共享记忆。
- 衰减机制：不重要的事实自动遗忘。
