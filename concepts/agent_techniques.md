---
title: 关键智能体技术
tags: [agent, function-calling, rag, chain-of-thought, tree-of-thought, self-consistency, prompt-chaining, technique]
related: [llm_agent_fundamentals, agent_architecture_patterns, agent_frameworks]
---

## 定义

智能体技术是在 LLM 智能体系统中使用的特定方法、提示策略和算法方法，用于改善推理、接地、可靠性和能力。

---

## 技术 1：函数调用 / 工具使用

### 定义
一种允许 LLM 生成结构化输出的机制，指定要调用的函数/工具及其参数，而不是仅产生自然语言文本。运行时环境执行该函数并将结果返回给 LLM。

### 工作原理

```
1. 系统提示定义可用工具（名称、描述、参数的 JSON Schema）
2. 用户发送消息
3. LLM 决定是直接响应还是调用工具
4. 如果调用工具：LLM 输出包含工具名称 + 参数的结构化 JSON
5. 运行时执行工具并返回结果
6. LLM 接收结果并决定响应或调用另一个工具
7. 重复直到 LLM 产生最终响应
```

### 各提供商的实现

**OpenAI 函数调用 / 工具使用：**
```json
{
  "type": "function",
  "function": {
    "name": "get_weather",
    "description": "Get current weather for a location",
    "parameters": {
      "type": "object",
      "properties": {
        "location": {"type": "string", "description": "City and state"},
        "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
      },
      "required": ["location"]
    }
  }
}
```
- 支持并行工具调用（一轮中多个函数）
- `tool_choice`："auto"、"required"、"none" 或特定函数
- 响应包含 `tool_calls` 数组，含 `id`、`function.name`、`function.arguments`

**Anthropic 工具使用：**
```json
{
  "name": "get_weather",
  "description": "Get current weather for a location",
  "input_schema": {
    "type": "object",
    "properties": {
      "location": {"type": "string", "description": "City and state"},
      "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
    },
    "required": ["location"]
  }
}
```
- 使用内容块：`tool_use` 块（来自模型）和 `tool_result` 块（来自用户）
- 支持 `tool_choice`："auto"、"any" 或特定工具
- Claude 将推理文本与工具调用交织在一起

**模型上下文协议（MCP）：**
- Anthropic 的开放标准，用于将 LLM 连接到外部工具和数据源
- 客户端-服务器架构：智能体（客户端）连接到暴露工具的 MCP 服务器
- 服务器可以是本地进程或远程服务
- 标准化协议意味着工具可在任何 MCP 兼容客户端上使用
- 不断增长的预构建 MCP 服务器生态系统（文件系统、GitHub、数据库、Slack 等）

### 最佳实践
- 编写清晰、具体的工具描述（LLM 使用这些来决定何时/如何调用）
- 包含带示例的参数描述
- 对约束参数使用枚举
- 优雅地处理错误并返回信息丰富的错误消息
- 在执行前验证工具输入
- 实现超时和速率限制
- 考虑使用工具选择控制以提高可靠性

### 关键挑战
- 工具选择准确性（选择正确的工具）
- 参数提取准确性（解析正确的值）
- 错误处理和恢复
- 安全性（防止通过工具结果进行提示注入）
- 顺序工具调用带来的延迟

---

## 技术 2：RAG（检索增强生成）

### 定义
一种通过从外部知识库检索信息来增强 LLM 生成的技术，使模型能够在无需重新训练的情况下访问特定领域的、最新的或私有的信息。

### 起源
Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020), Meta AI

### 架构

```
                    +-------------------+
  用户查询 -------->| 嵌入模型          |-----> 查询向量
                    +-------------------+
                                                    |
                                                    v
                    +-------------------+    +--------------+
                    | 向量数据库        |<-->|   检索       |
                    |（知识库）         |    |  （Top-K）   |
                    +-------------------+    +--------------+
                                                    |
                                                    v
                    +-------------------+    +--------------+
                    |       LLM         |<---|  检索到的    |
                    |    （生成器）      |    |   上下文     |
                    +-------------------+    +--------------+
                           |
                           v
                      最终答案
```

### 流水线步骤

**索引（离线）：**
1. **加载**：摄取文档（PDF、HTML、Markdown、数据库、API）
2. **分块**：将文档分割为可管理的片段
   - 固定大小分块（例如 500 tokens，50 token 重叠）
   - 语义分块（在自然边界处分割：段落、章节）
   - 递归分块（先尝试最大分割器，然后逐步使用更小的）
   - 文档感知分块（尊重文档结构）
3. **嵌入**：使用嵌入模型将分块转换为密集向量
   - 常用模型：OpenAI text-embedding-3-small/large、Cohere embed-v3、BGE、E5
4. **存储**：将向量 + 元数据保存在向量数据库中
   - 数据库：Pinecone、Weaviate、ChromaDB、Qdrant、Milvus、pgvector、FAISS

**检索（在线）：**
1. **嵌入查询**：使用相同的嵌入模型将用户查询转换为向量
2. **搜索**：通过近似最近邻（ANN）搜索找到最相似的 Top-K 分块
   - 相似度度量：余弦相似度、点积、欧氏距离
3. **重排序**（可选）：使用交叉编码器重新评分和重新排列结果以提高相关性
   - 模型：Cohere Rerank、BGE-reranker、cross-encoder/ms-marco
4. **过滤**（可选）：应用元数据过滤（日期、来源、类别）

**生成：**
1. **构建提示**：将检索到的分块作为上下文插入 LLM 提示中
2. **生成**：LLM 基于检索到的上下文产生答案
3. **引用**（可选）：在响应中包含来源引用

### 高级 RAG 技术

| 技术 | 描述 |
|------|------|
| **混合搜索** | 结合密集（向量）和稀疏（BM25/关键词）检索 |
| **HyDE** | 假设文档嵌入——先生成假设答案，然后搜索相似的真实文档 |
| **多查询 RAG** | 生成多个查询变体并分别检索 |
| **父文档检索** | 检索小分块但返回其父文档以获得更多上下文 |
| **自查询** | LLM 在语义查询旁边生成结构化过滤器 |
| **上下文压缩** | 压缩检索到的文档以仅提取相关句子 |
| **智能体 RAG** | 智能体决定何时检索和检索什么，可查询多个来源 |
| **纠正性 RAG（CRAG）** | 评估检索质量；如果质量差，回退到网络搜索 |
| **图 RAG** | 从文档构建知识图谱，然后遍历它进行检索（Microsoft Research, 2024） |
| **延迟分块** | 先嵌入完整文档，然后对嵌入进行分块（保留跨分块上下文） |

### 评估指标
- **检索**：Precision@K、Recall@K、MRR、NDCG、命中率
- **生成**：忠实度、相关性、答案正确性
- **框架**：RAGAS、TruLens、LangSmith 评估

---

## 技术 3：思维链（CoT）提示

### 定义
一种提示技术，鼓励 LLM 在产生最终答案之前生成中间推理步骤，显著提高复杂推理任务的性能。

### 起源
Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (2022), Google Brain

### 变体

**少样本 CoT：**
提供包含逐步推理的示例：
```
Q: Roger has 5 tennis balls. He buys 2 cans of 3 each. How many does he have now?
A: Roger starts with 5 balls. 2 cans of 3 = 6 balls. 5 + 6 = 11. The answer is 11.

Q: [actual question]
A: [model generates reasoning + answer]
```

**零样本 CoT：**
只需在提示末尾添加"Let's think step by step"（Kojima et al., 2022）：
```
Q: [question]
A: Let's think step by step.
```

**Auto-CoT：**
通过聚类问题并在代表性示例上使用零样本 CoT 自动生成多样化的 CoT 演示（Zhang et al., 2022）。

### 为什么有效
- 迫使模型分解复杂问题
- 每个推理步骤都是一个更简单的任务
- 中间结果可用于后续步骤
- 暴露推理过程（可解释）
- 对数学、逻辑和多跳推理特别有效

---

## 技术 4：思维树（ToT）

### 定义
思维链的扩展，在树结构中探索多条推理路径，评估中间思想，并使用搜索算法（BFS、DFS）找到最佳推理路径。

### 起源
Yao et al., "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" (2023)

### 工作原理

```
                    [问题]
                   /    |    \
                  /     |     \
            [思想     [思想     [思想
              1a]      1b]      1c]
             /  \       |        X（剪枝）
            /    \      |
     [思想    [思想   [思想
       2a]     2b]     2d]
        |       X       |
        |    （剪枝）    |
     [解决方案        [解决方案
       A]               B]
```

1. **思想分解**：将问题分解为步骤，每步需要一个"思想"
2. **思想生成**：在每步生成多个候选思想
3. **思想评估**：使用 LLM 评估每个思想（投票、评分或分类为确定/可能/不可能）
4. **搜索**：使用 BFS 或带回溯的 DFS 探索树
5. **选择**：选择通往解决方案的最佳路径

### 何时使用
- 需要探索的问题（谜题、创意写作、游戏）
- 初始方法可能导致死路的任务
- 需要考虑多种策略时
- 不适用于简单直接的任务（过度使用）

---

## 技术 5：自一致性

### 定义
一种解码策略，从 LLM 中采样多条多样化推理路径，并通过多数投票选择最一致的答案。

### 起源
Wang et al., "Self-Consistency Improves Chain of Thought Reasoning in Language Models" (2022), Google Brain

### 工作原理

```
                      [问题]
                     /    |    \
                    /     |     \
              [CoT       [CoT       [CoT
              路径 1]    路径 2]    路径 3]
                |          |          |
            答案：42    答案：42    答案：37
                \          |          /
                 \         |         /
              +---v--------v--------v---+
              |      多数投票           |
              |    答案：42（2/3）      |
              +-------------------------+
```

1. 使用 temperature > 0 采样 N 条推理路径
2. 从每条路径中提取最终答案
3. 对所有答案进行多数投票

### 优势
- 实现简单（只需多次采样）
- 相比单路径 CoT 有显著的准确率提升
- 适用于任何 CoT 变体
- 无需训练

### 局限性
- 贵 N 倍（N 次采样）
- 只在有明确"答案"可投票时有效
- 对开放式生成任务效果较差

---

## 技术 6：提示链

### 定义
将复杂任务分解为一系列更简单的提示，其中一个提示的输出成为下一个提示的输入（或部分输入）。

### 工作原理

```
输入 --> [提示 1] --> 输出 1 --> [提示 2] --> 输出 2 --> [提示 3] --> 最终输出
         （提取）                  （分析）                  （格式化）
```

### 示例：研究报告生成

```
Step 1: "Extract the key claims from this article: {article}"
        -> claims_list

Step 2: "For each claim, assess its validity and find supporting evidence: {claims_list}"
        -> evidence_analysis

Step 3: "Write a balanced research summary based on this analysis: {evidence_analysis}"
        -> final_report
```

### 模式

| 模式 | 描述 | 示例 |
|------|------|------|
| **顺序** | 线性提示链 | 提取 -> 分析 -> 总结 |
| **条件** | 根据输出分支 | 分类 ->（如果正面：展开，如果负面：反驳） |
| **并行 + 合并** | 并行处理后合并 | 从视角 A、B、C 分析 -> 综合 |
| **迭代** | 循环直到达到质量阈值 | 写作 -> 批评 -> 修改 -> 批评 -> ... |
| **MapReduce** | 对项目映射提示，归约结果 | 总结每章 -> 合并为全书摘要 |

### 优势
- 每个步骤更简单、更可靠
- 中间结果可检查
- 每步可使用不同的模型/温度
- 容易测试和调试各个步骤
- 人工审查的自然检查点

### 局限性
- 延迟增加（顺序调用）
- 步骤之间的错误传播
- 需要管理更复杂的流水线
- 总成本可能高于单次调用方法

---

## 技术 7：其他值得注意的技术

### 结构化输出 / JSON 模式
强制 LLM 输出有效的 JSON 或符合特定模式：
- OpenAI：`response_format: { type: "json_object" }` 或带 JSON Schema 的结构化输出
- Anthropic：工具使用作为结构化输出，或约束生成
- 库：Instructor、Outlines、Guidance

### 智能体的少样本提示
提供成功的智能体交互示例（思想-行动-观察轨迹）来引导行为。

### 系统提示 / 元提示
编写详细的系统提示来定义：
- 智能体角色和能力
- 可用工具及其使用说明
- 输出格式要求
- 安全防护和约束
- 期望行为的示例

### 约束解码
限制 LLM 的输出空间为有效 token：
- 基于语法：仅生成产生有效 JSON、SQL、代码的 token
- 库：Outlines、LMQL、SGLang

### 混合智能体
根据复杂度将不同任务或子任务路由到不同模型：
- 简单任务 -> 小型/快速模型（Haiku、GPT-4o-mini）
- 复杂任务 -> 大型/强力模型（Opus、GPT-4o、o1）
- 成本和延迟优化

## 参考文献

- Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020)
- Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (2022)
- Wang et al., "Self-Consistency Improves Chain of Thought Reasoning in Language Models" (2022)
- Yao et al., "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" (2023)
- Kojima et al., "Large Language Models are Zero-Shot Reasoners" (2022)
- Schick et al., "Toolformer: Language Models Can Teach Themselves to Use Tools" (2023)
- Patil et al., "Gorilla: Large Language Model Connected with Massive APIs" (2023)
- Gao et al., "Retrieval-Augmented Generation for Large Language Models: A Survey" (2024)
- Edge et al., "From Local to Global: A Graph RAG Approach to Query-Focused Summarization" (2024)
