---
title: Agent Memory Systems
tags: [agent, memory, rag, vector-database, short-term, long-term, episodic, semantic]
related: [llm_agent_fundamentals, agent_techniques, agent_architecture_patterns]
---

## Definition

Memory systems in LLM agents provide mechanisms for storing, organizing, and retrieving information across interactions and beyond the limits of the LLM's context window. They are fundamental to enabling agents that can learn from experience, maintain context, and access large knowledge bases.

## Intuition

An LLM without external memory is like a person with perfect reasoning but no long-term memory -- they can solve problems presented to them but forget everything between conversations. Agent memory systems give the LLM persistence, allowing it to accumulate knowledge, learn from past mistakes, and maintain coherent behavior over long interactions.

## Memory Taxonomy

### 1. Short-Term / Working Memory

**Implementation**: The LLM's context window itself.

**Characteristics:**
- Stores current conversation, recent observations, active plan
- Volatile: lost when the conversation/session ends
- Size-limited: context window (4K - 1M+ tokens)
- Zero latency: already in the prompt

**Management Strategies:**
- **Sliding window**: Keep only the most recent N messages
- **Summarization**: Periodically summarize older messages, replace full text with summary
- **Compaction**: Remove redundant or low-value content from context
- **Priority-based**: Keep high-priority items (user goal, current plan, recent observations)

```
Context Window Layout (typical):
+-----------------------------------------+
| System Prompt (instructions, tools)      | <- Fixed
| Long-term memory retrieval (if any)      | <- Retrieved per turn
| Conversation history (summarized older)  | <- Managed
| Recent messages (full detail)            | <- Most recent
| Current user message                     | <- New
+-----------------------------------------+
```

### 2. Long-Term Memory

**Implementation**: External storage (vector DB, key-value store, graph DB).

**Characteristics:**
- Persists across sessions
- Effectively unlimited storage
- Requires retrieval mechanism (adds latency)
- Must be explicitly written to and read from

**Storage Backends:**
| Backend | Best For | Examples |
|---------|----------|---------|
| Vector DB | Semantic search over unstructured text | Pinecone, Weaviate, ChromaDB, Qdrant, FAISS |
| Key-value store | Exact lookup, user preferences | Redis, DynamoDB |
| Graph DB | Relationships, entity networks | Neo4j, Amazon Neptune |
| SQL DB | Structured data, analytics | PostgreSQL, SQLite |
| File system | Documents, code, artifacts | Local/cloud storage |

### 3. Episodic Memory

**What it stores**: Specific past experiences and interaction sequences.

**Analogous to**: Human episodic memory ("I remember that time when...")

**Use Cases:**
- Recalling how a similar problem was solved before
- Avoiding previously encountered errors
- Building on past successful strategies

**Implementation Approaches:**
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

### 4. Semantic Memory

**What it stores**: General knowledge, facts, and concepts (not tied to specific experiences).

**Analogous to**: Human semantic memory ("I know that Python is a programming language")

**Use Cases:**
- Domain knowledge base
- Company documentation and policies
- Technical reference material
- User preferences and profile information

**Implementation Approaches:**
- RAG over a document knowledge base
- Knowledge graph with entity-relationship triples
- Structured databases for factual lookups

### 5. Procedural Memory

**What it stores**: "How to" knowledge -- learned skills, action sequences, workflows.

**Analogous to**: Human procedural memory ("I know how to ride a bike")

**Implementation Approaches:**
- Saved prompt templates for specific tasks
- Code libraries and scripts the agent has learned to use
- Fine-tuned model weights (embeds procedures into the model)
- Documented SOPs that the agent retrieves when needed

### 6. RAG-Based Memory

**What it is**: Using Retrieval-Augmented Generation as the primary memory mechanism.

**Architecture:**
```
Agent Interaction --> Store important info --> Vector DB
                                                  |
New query ------> Embed query -----> Retrieve relevant memories
                                                  |
                                          Inject into prompt
                                                  |
                                          LLM generates response
```

**What Gets Stored:**
- Conversation summaries
- Key decisions and their rationale
- User preferences and facts learned during interaction
- Task results and outcomes
- Error messages and resolutions

**Retrieval Strategies:**
- **Recency-weighted**: Recent memories score higher
- **Importance-weighted**: Important memories (flagged by LLM) score higher
- **Relevance-only**: Pure semantic similarity
- **Combined score**: alpha * recency + beta * importance + gamma * relevance (used in Generative Agents, Park et al., 2023)

## Formulation

**Memory-Augmented Agent Loop:**

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

## Variants

### Memory Reflection (from Generative Agents)
Periodically synthesize higher-level observations from raw memories:
```
Raw memories: 
  - "User asked about Python decorators"
  - "User struggled with closure scope"  
  - "User successfully wrote a class decorator"

Reflection:
  - "User is learning advanced Python concepts, specifically decorators. 
     They understand basic syntax but need help with scope/closure concepts."
```

### Forgetting / Memory Decay
Not all memories should be kept forever:
- Apply time-based decay to reduce the influence of old memories
- Consolidate similar memories to prevent redundancy
- Delete or archive irrelevant memories periodically

### Hierarchical Memory
```
Level 3: Abstract knowledge ("User prefers functional programming")
   ^
Level 2: Summarized episodes ("In our last 5 sessions about Python...")
   ^
Level 1: Raw interaction logs ("User: How do I use map()? ...")
```

## References

- Park et al., "Generative Agents: Interactive Simulacra of Human Behavior" (2023) -- Influential memory architecture with reflection
- Zhang et al., "A Survey on the Memory Mechanism of Large Language Model based Agents" (2024)
- Packer et al., "MemGPT: Towards LLMs as Operating Systems" (2023) -- Virtual memory management for LLMs
- Zhong et al., "MemoryBank: Enhancing Large Language Models with Long-Term Memory" (2023)
- Modarressi et al., "RET-LLM: Towards a General Read-Write Memory for Large Language Models" (2023)
