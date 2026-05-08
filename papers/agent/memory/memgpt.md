# MemGPT: Towards LLMs as Operating Systems

## Metadata
- **Title**: MemGPT: Towards LLMs as Operating Systems
- **Authors**: Charles Packer, Sarah Wooders, Kevin Lin, Vivian Fang, Shishir G. Patil, Ion Stoica, Joseph E. Gonzalez
- **Venue**: ICLR 2024 (Spotlight)
- **Year**: 2023
- **URL**: https://arxiv.org/abs/2310.08560
- **Code**: https://github.com/cpacker/MemGPT
- **Tags**: [memory, context-management, os-inspired, architecture, long-context]
- **Note**: Later evolved into the Letta framework (https://github.com/letta-ai/letta)

---

## TL;DR

MemGPT applies operating system concepts -- virtual memory, paging, interrupts -- to LLM context management, enabling agents to self-direct memory operations across a tiered storage hierarchy (main context, recall storage, archival storage) for unbounded conversational and analytical tasks.

---

## Motivation & Problem

LLMs have a **fixed context window** that creates hard limits on their capabilities:

1. **Multi-session conversations**: Information from early interactions is lost when context fills up. Traditional chatbots either truncate history or use naive summarization.
2. **Large document analysis**: Documents exceeding the context window cannot be processed holistically. Standard RAG retrieves fragments but loses global coherence.
3. **Persistent agent state**: Agents cannot maintain structured, evolving knowledge across interactions.

Existing solutions are inadequate:
- **Context window extension** (ALiBi, RoPE scaling): Increases length but not unboundedly; attention cost grows quadratically.
- **RAG (Retrieval-Augmented Generation)**: Retrieves relevant chunks but the LLM has no control over what is retrieved or when. The retrieval is **system-directed**, not **agent-directed**.
- **Summarization**: Lossy compression that discards details the agent may need later.

The key insight: Operating systems solved an analogous problem decades ago -- processes need more memory than physically available RAM. The solution was **virtual memory** with demand paging. MemGPT adapts this paradigm for LLMs.

---

## Method

### The OS Analogy

| OS Concept | MemGPT Analog |
|-----------|---------------|
| Physical RAM | LLM context window (main context) |
| Virtual memory / Disk | External databases (recall + archival storage) |
| Page table | Memory metadata / indexes |
| Page fault / demand paging | Agent requests data not in context |
| OS scheduler / interrupts | Event system triggering agent actions |
| Process | Conversation thread or task |

### Memory Hierarchy

MemGPT defines a three-tier memory hierarchy:

```
+============================================+
|            MAIN CONTEXT (in-LLM)           |
|  +--------------------------------------+  |
|  |  System Prompt (instructions, persona)|  |
|  +--------------------------------------+  |
|  |  Working Context (scratchpad)         |  |   Tier 0: "RAM"
|  |  - Key facts about current user       |  |   (within context window)
|  |  - Active task state                  |  |
|  +--------------------------------------+  |
|  |  FIFO Message Queue                   |  |
|  |  - Recent conversation messages       |  |
|  |  - Most recent first                  |  |
|  +--------------------------------------+  |
+============================================+
               |          ^
      page out |          | page in
               v          |
+============================================+
|        RECALL STORAGE (database)           |   Tier 1: "Disk"
|  - Full conversation history               |   (conversation history DB)
|  - All past messages, timestamped          |
|  - Searchable by text, date, keyword      |
+============================================+
               |          ^
      archive  |          | retrieve
               v          |
+============================================+
|       ARCHIVAL STORAGE (database)          |   Tier 2: "Cold Storage"
|  - Long-term knowledge base               |   (persistent knowledge DB)
|  - Documents, facts, user preferences     |
|  - Searchable via embedding similarity    |
|  - Unlimited capacity                     |
+============================================+
```

**Main Context** (Tier 0): Fixed-size, fits within the LLM's context window. Divided into:
- **System prompt**: Static instructions, persona definition (~1000 tokens)
- **Working context**: Mutable scratchpad for key facts and active state (~2000 tokens)
- **FIFO message queue**: Recent conversation turns, managed as a queue with oldest messages evicted first

**Recall Storage** (Tier 1): SQLite/Postgres database storing the complete conversation history. Every message ever sent or received is stored with timestamps and metadata. Searchable by text content, date range, or keyword.

**Archival Storage** (Tier 2): Vector database (e.g., using embeddings) for long-term knowledge. Stores documents, extracted facts, user preferences, and any information the agent decides is worth preserving. Searchable via embedding similarity. Effectively unlimited capacity.

### Self-Directed Memory Management Functions

The LLM agent is given explicit function-calling tools to manage its own memory. This is the critical innovation -- the agent **decides** when and what to page in/out:

```python
# Core memory functions exposed to the agent:

# --- Working Context (in main context) ---
core_memory_append(section: str, content: str)
    # Append to the working context scratchpad
    # e.g., core_memory_append("user", "User prefers formal tone")

core_memory_replace(section: str, old: str, new: str)
    # Edit existing working context content
    # e.g., core_memory_replace("user", "likes cats", "likes dogs")

# --- Recall Storage (conversation history) ---
conversation_search(query: str, page: int = 0)
    # Search past conversation history
    # Returns paginated results from recall storage

conversation_search_date(start: str, end: str, page: int = 0)
    # Search conversations by date range

# --- Archival Storage (knowledge base) ---
archival_memory_insert(content: str)
    # Store information in long-term archival storage
    # e.g., archival_memory_insert("Project X deadline is March 15")

archival_memory_search(query: str, page: int = 0)
    # Retrieve from archival storage via embedding search
    # Returns paginated results
```

### Agent Decision Loop (Paging Mechanism)

The agent operates in an event-driven loop analogous to an OS interrupt handler:

```
Algorithm: MemGPT Agent Loop
----------------------------------------------------
Input:  Event e (user message, system alert, timer)
Output: Response and/or memory operations

1:  while True do
2:      e <- wait_for_event()           // block until event
3:      inject(e, main_context)         // add event to FIFO queue
4:      if overflow(main_context) then
5:          evict_oldest(FIFO_queue)    // page out oldest messages
6:          store(evicted, recall_storage)
7:      end if
8:      response <- LLM(main_context)  // generate with full context
9:      parse function calls from response
10:     for each function_call in response do
11:         if is_memory_function(function_call) then
12:             execute memory operation     // page in/out
13:             update main_context with results
14:         elif is_send_message(function_call) then
15:             deliver message to user
16:         end if
17:     end for
18:     if agent requested "heartbeat" then
19:         continue                    // agent wants another turn
20:     end if
21: end while
```

Key mechanisms:

**Inner monologue**: Before responding, the agent generates internal thoughts (not shown to user) reasoning about whether it needs to access memory, what to search for, and what to store.

**Heartbeat mechanism**: The agent can request additional processing turns without user input, allowing it to perform multiple memory operations before responding. This is analogous to an OS process requesting more CPU time.

**Automatic FIFO eviction**: When the message queue exceeds its allocation within main context, the oldest messages are automatically moved to recall storage. The agent can later retrieve them via `conversation_search`.

### How the Agent Decides What to Page In/Out

The agent's memory management decisions are guided by:
1. **System prompt instructions**: Explicit guidance like "If you don't remember something, search your recall and archival memory before saying you don't know."
2. **Inner monologue reasoning**: The agent thinks step-by-step about what information it needs: "The user is asking about our conversation last week. I should search recall storage for messages from that date range."
3. **Working context as index**: The working context stores high-level summaries and pointers: "User previously discussed Project X (see archival memory for details)." This enables the agent to know what exists in deep storage without keeping full details in main context.

---

## Key Innovations

1. **Agent-directed memory management**: Unlike RAG where retrieval is triggered by the system, MemGPT lets the LLM itself decide when to read/write memory through function calls.
2. **Tiered memory hierarchy**: Explicit separation into working context, recall (conversation), and archival (knowledge) storage mirrors how humans organize memory.
3. **Heartbeat mechanism**: Allows multi-step memory operations within a single user interaction, giving the agent time to "think" and access external storage before responding.
4. **Inner monologue**: The agent's reasoning about memory operations is transparent and inspectable, enabling debugging of memory management decisions.
5. **Unbounded operation**: By paging context in and out, MemGPT can maintain conversations and analyze documents of arbitrary length.

---

## Experimental Setup

### Task 1: Multi-Session Conversations
- **Setup**: Conversations spanning many sessions where the agent must recall information from much earlier interactions
- **Dataset**: Synthetic multi-session dialogues with planted facts that must be recalled later
- **Models**: GPT-4, GPT-3.5-turbo as the base LLM
- **Baselines**:
  - Fixed context window (truncation at most recent messages)
  - RAG-augmented retrieval (retrieve top-k relevant past messages)
  - Summarization-based compression of conversation history

### Task 2: Document Analysis (Document QA)
- **Setup**: QA over documents far exceeding the context window
- **Dataset**: Long documents requiring multi-passage reasoning
- **Baselines**:
  - Naive chunking + RAG
  - Map-reduce summarization
  - Recursive summarization

### Metrics
- Conversation consistency (does the agent contradict earlier statements?)
- Fact recall accuracy (can the agent retrieve planted facts from much earlier?)
- Document QA accuracy
- User preference ratings (human evaluation)

---

## Results

### Multi-Session Conversation

| Method | Consistency (%) | Fact Recall (%) | User Preference |
|--------|:--------------:|:--------------:|:--------------:|
| Fixed window (truncate) | 45 | 12 | 2.1/5 |
| RAG (top-5 retrieval) | 72 | 58 | 3.2/5 |
| Summarization | 68 | 41 | 3.0/5 |
| **MemGPT** | **93** | **88** | **4.2/5** |

Key findings:
- MemGPT dramatically outperforms fixed-window approaches on fact recall because it can proactively search for relevant past information.
- RAG retrieval sometimes fails because the retrieval query (current message) does not lexically match the relevant past message. MemGPT's agent-directed search can reformulate queries.
- Summarization loses specific details that MemGPT preserves in archival storage.

### Document Analysis

| Method | Accuracy on Multi-Passage Questions (%) |
|--------|:---------------------------------------:|
| Naive RAG (chunk + retrieve) | 57 |
| Map-reduce summarize | 52 |
| Recursive summarize | 61 |
| **MemGPT** | **78** |

MemGPT's advantage: the agent can iteratively search different parts of the document, cross-reference passages, and build up understanding in its working context. Standard RAG retrieves fixed chunks without the ability to follow up.

### Comparison with Standard RAG

| Dimension | Standard RAG | MemGPT |
|-----------|:----------:|:-----:|
| Retrieval trigger | System (automatic on each query) | Agent (explicit function call) |
| Query formulation | User's raw input | Agent's reformulated search |
| Multi-step retrieval | No (single retrieval per turn) | Yes (heartbeat enables chaining) |
| Memory writing | No (read-only) | Yes (agent can insert/update) |
| Context management | Append retrieved chunks | Agent manages what stays/goes |
| Long-term personalization | Limited | Native (working context + archival) |

---

## Analysis & Insights

1. **Agent autonomy in memory management is key**: The biggest differentiator vs. RAG is not the storage mechanism but the agent's ability to decide *when* and *what* to retrieve. This mirrors how human memory is reconstructive, not just retrieval-based.

2. **Working context as cognitive register**: The mutable working context scratchpad functions like a CPU register file -- small, fast, and holding the most immediately relevant state. The agent learns to keep high-level summaries here and detailed data in archival storage.

3. **Inner monologue cost**: The heartbeat mechanism and inner monologue add latency and token cost. Each memory operation requires a full LLM inference call, making MemGPT significantly more expensive per interaction than simple prompting.

4. **Failure modes**: The agent sometimes (a) forgets to search memory when it should, (b) retrieves irrelevant results due to poor query formulation, or (c) overfills working context with details that should be in archival storage.

5. **Persona consistency**: By storing persona-relevant information in working context and archival storage, MemGPT maintains more consistent character/persona across long conversations than any baseline.

---

## Limitations & Critiques

1. **Latency**: Multiple LLM calls per interaction (inner monologue + memory ops + response) creates significant latency. Not suitable for real-time applications.
2. **Cost**: Each heartbeat/memory operation is a full inference call. A single user interaction may trigger 3-5 LLM calls.
3. **Prompt engineering fragility**: The system prompt must carefully instruct the agent on memory management. Poor instructions lead to underutilization of memory functions.
4. **No memory consolidation**: Unlike human memory, there is no automatic process for consolidating, compressing, or reorganizing stored memories over time.
5. **Single-agent focus**: The original MemGPT design is for single-agent scenarios. Multi-agent shared memory is not addressed (later partially addressed in Letta).
6. **Embedding quality dependency**: Archival storage search quality depends entirely on embedding model quality. Poor embeddings lead to retrieval failures.
7. **No forgetting mechanism**: All information is preserved indefinitely. There is no principled mechanism for pruning irrelevant or outdated information, which can degrade search quality over time.

---

## Evolution: MemGPT to Letta

The MemGPT project evolved into **Letta** (formerly MemGPT framework):
- **Letta Server**: Production-ready agent server with REST API
- **Multi-agent support**: Multiple agents sharing archival storage
- **Tool ecosystem**: Extended beyond memory functions to general tool use
- **State management**: Persistent agent state with serialization/deserialization
- **ADE (Agent Development Environment)**: Visual IDE for building MemGPT-style agents
- The core memory hierarchy and self-directed management paradigm remain central

---

## Follow-up Work

- **Letta/MemGPT framework**: Production evolution of the research prototype.
- **LongMem** (Wang et al., 2023): Augments LLMs with a retrieval-augmented memory bank, similar concept but system-directed.
- **Generative Agents** (Park et al., 2023): Uses memory streams with retrieval/reflection/planning, complementary to MemGPT's paging approach.
- **MemoryBank** (Zhong et al., 2024): Adds forgetting mechanisms inspired by Ebbinghaus curves.
- **A-MEM** (2025): Zettelkasten-inspired memory with dynamic note linking.
- **Mem0**: Production memory layer for AI applications, influenced by MemGPT concepts.

---

## Key Takeaways

1. The OS virtual memory analogy is powerful: tiered storage + agent-directed paging solves the fixed context window problem more flexibly than RAG or summarization.
2. Agent-directed memory management (the agent decides what to read/write) fundamentally outperforms system-directed retrieval because the agent can reason about what information it needs.
3. The working context scratchpad is critical -- it serves as the agent's "cognitive register file" for maintaining active state.
4. The heartbeat mechanism enables multi-step memory reasoning within a single interaction, analogous to OS interrupts allowing preemptive scheduling.
5. The approach has significant latency and cost overhead due to multiple LLM calls per interaction, creating a practical tradeoff between memory capability and responsiveness.
