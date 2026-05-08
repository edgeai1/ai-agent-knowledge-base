---
title: 智能体记忆系统
tags: [agent, memory, rag, vector-database, short-term, long-term, episodic, semantic]
related: [llm_agent_fundamentals, agent_techniques, agent_architecture_patterns]
---

## 定义

LLM 智能体中的记忆系统提供了跨交互存储、组织和检索信息的机制，突破 LLM 上下文窗口的限制。它们是构建能够从经验中学习、维护上下文和访问大型知识库的智能体的基础。

## 直觉理解

没有外部记忆的 LLM 就像一个拥有完美推理能力但没有长期记忆的人——他们可以解决呈现给他们的问题，但在对话之间会忘记一切。智能体记忆系统赋予 LLM 持久性，使其能够积累知识、从过去的错误中学习，并在长时间交互中保持一致的行为。

## 记忆分类体系

### 1. 短期 / 工作记忆

**实现方式**：LLM 的上下文窗口本身。

**特征：**
- 存储当前对话、近期观察、活跃计划
- 易失性：对话/会话结束时丢失
- 大小受限：上下文窗口（4K - 1M+ tokens）
- 零延迟：已经在提示中

**管理策略：**
- **滑动窗口**：仅保留最近 N 条消息
- **摘要化**：定期总结旧消息，用摘要替换全文
- **压缩**：从上下文中移除冗余或低价值内容
- **基于优先级**：保留高优先级项目（用户目标、当前计划、近期观察）

```
Context Window Layout (typical):
+-----------------------------------------+
| 系统提示（指令、工具）                    | <- 固定
| 长期记忆检索（如有）                      | <- 每轮检索
| 对话历史（旧消息已摘要化）                | <- 管理的
| 近期消息（完整细节）                      | <- 最新的
| 当前用户消息                              | <- 新的
+-----------------------------------------+
```

### 2. 长期记忆

**实现方式**：外部存储（向量数据库、键值存储、图数据库）。

**特征：**
- 跨会话持久化
- 存储容量几乎无限
- 需要检索机制（增加延迟）
- 必须显式写入和读取

**存储后端：**
| 后端 | 最适用于 | 示例 |
|------|----------|------|
| 向量数据库 | 非结构化文本的语义搜索 | Pinecone、Weaviate、ChromaDB、Qdrant、FAISS |
| 键值存储 | 精确查找、用户偏好 | Redis、DynamoDB |
| 图数据库 | 关系、实体网络 | Neo4j、Amazon Neptune |
| SQL 数据库 | 结构化数据、分析 | PostgreSQL、SQLite |
| 文件系统 | 文档、代码、工件 | 本地/云存储 |

### 3. 情景记忆

**存储内容**：特定的过去经验和交互序列。

**类似于**：人类的情景记忆（"我记得那次……"）

**应用场景：**
- 回忆之前如何解决类似问题
- 避免先前遇到的错误
- 基于过去成功的策略进行构建

**实现方法：**
```python
# Each episode is a structured record
episode = {
    "timestamp": "2024-01-15T10:30:00",
    "task": "Deploy new API version",
    "actions": ["checked tests", "updated config", "ran deploy script"],
    "outcome": "success",
    "observations": ["Tests passed", "Deployment took 3 minutes"],
    "lessons": ["Always run smoke tests after deploy"]
}
# Stored in vector DB, retrieved by semantic similarity to current task
```

### 4. 语义记忆

**存储内容**：通用知识、事实和概念（不与特定经验绑定）。

**类似于**：人类的语义记忆（"我知道 Python 是一种编程语言"）

**应用场景：**
- 领域知识库
- 公司文档和政策
- 技术参考资料
- 用户偏好和个人资料信息

**实现方法：**
- 基于文档知识库的 RAG
- 包含实体-关系三元组的知识图谱
- 用于事实查找的结构化数据库

### 5. 程序性记忆

**存储内容**："如何做"的知识——已学习的技能、行动序列、工作流。

**类似于**：人类的程序性记忆（"我知道如何骑自行车"）

**实现方法：**
- 为特定任务保存的提示模板
- 智能体已学会使用的代码库和脚本
- 微调的模型权重（将程序嵌入模型中）
- 智能体在需要时检索的文档化 SOP

### 6. 基于 RAG 的记忆

**定义**：将检索增强生成作为主要记忆机制。

**架构：**
```
Agent Interaction --> Store important info --> Vector DB
                                                  |
New query ------> Embed query -----> Retrieve relevant memories
                                                  |
                                          Inject into prompt
                                                  |
                                          LLM generates response
```

**存储内容：**
- 对话摘要
- 关键决策及其理由
- 在交互中了解到的用户偏好和事实
- 任务结果和结论
- 错误消息和解决方案

**检索策略：**
- **近期性加权**：近期记忆得分更高
- **重要性加权**：重要记忆（由 LLM 标记）得分更高
- **仅相关性**：纯语义相似度
- **综合评分**：alpha * 近期性 + beta * 重要性 + gamma * 相关性（用于 Generative Agents，Park et al., 2023）

## 公式化

**记忆增强智能体循环：**

```python
def memory_augmented_agent(query, memory_store, llm):
    # Retrieve relevant memories
    relevant_memories = memory_store.retrieve(
        query=query,
        top_k=5,
        strategy="combined"  # recency + importance + relevance
    )
    
    # Construct prompt with memories
    prompt = f"""
    System: You are an AI assistant with memory.
    
    Relevant memories:
    {format_memories(relevant_memories)}
    
    Current conversation:
    {format_conversation_history()}
    
    User: {query}
    """
    
    # Generate response
    response = llm.generate(prompt)
    
    # Store new memory
    memory_store.add(
        content=f"User asked: {query}\nAssistant responded: {response}",
        metadata={
            "timestamp": now(),
            "importance": llm.rate_importance(query, response),
            "type": "interaction"
        }
    )
    
    # Periodic memory maintenance
    if should_reflect():
        reflections = llm.reflect(memory_store.recent(n=20))
        memory_store.add_reflections(reflections)
    
    return response
```

## 变体

### 记忆反思（来自 Generative Agents）
定期从原始记忆中综合出更高层次的观察：
```
原始记忆：
  - "用户询问了 Python 装饰器"
  - "用户在闭包作用域方面遇到困难"
  - "用户成功编写了一个类装饰器"

反思：
  - "用户正在学习高级 Python 概念，特别是装饰器。
     他们理解基本语法，但在作用域/闭包概念方面需要帮助。"
```

### 遗忘 / 记忆衰减
并非所有记忆都应该永久保留：
- 应用基于时间的衰减来降低旧记忆的影响
- 合并相似记忆以防止冗余
- 定期删除或归档无关记忆

### 分层记忆
```
第 3 层：抽象知识（"用户偏好函数式编程"）
   ^
第 2 层：摘要化的情景（"在我们最近 5 次关于 Python 的会话中……"）
   ^
第 1 层：原始交互日志（"用户：如何使用 map()？……"）
```

## 参考文献

- Park et al., "Generative Agents: Interactive Simulacra of Human Behavior" (2023) -- 具有影响力的带反思的记忆架构
- Zhang et al., "A Survey on the Memory Mechanism of Large Language Model based Agents" (2024)
- Packer et al., "MemGPT: Towards LLMs as Operating Systems" (2023) -- LLM 的虚拟内存管理
- Zhong et al., "MemoryBank: Enhancing Large Language Models with Long-Term Memory" (2023)
- Modarressi et al., "RET-LLM: Towards a General Read-Write Memory for Large Language Models" (2023)
