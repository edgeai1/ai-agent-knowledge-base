---
title: Key Agent Techniques
tags: [agent, function-calling, rag, chain-of-thought, tree-of-thought, self-consistency, prompt-chaining, technique]
related: [llm_agent_fundamentals, agent_architecture_patterns, agent_frameworks]
---

## Definition

Agent techniques are specific methods, prompting strategies, and algorithmic approaches used within LLM agent systems to improve reasoning, grounding, reliability, and capability.

---

## Technique 1: Function Calling / Tool Use

### What It Is
A mechanism that allows LLMs to generate structured output specifying a function/tool to call, along with its arguments, rather than producing only natural language text. The runtime environment executes the function and returns results to the LLM.

### How It Works

```
1. System prompt defines available tools (name, description, parameters as JSON schema)
2. User sends a message
3. LLM decides whether to respond directly OR call a tool
4. If tool call: LLM outputs structured JSON with tool name + arguments
5. Runtime executes the tool and returns the result
6. LLM receives the result and either responds or calls another tool
7. Repeat until LLM produces a final response
```

### Provider Implementations

**OpenAI Function Calling / Tool Use:**
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
- Supports parallel tool calls (multiple functions in one turn)
- `tool_choice`: "auto", "required", "none", or specific function
- Response includes `tool_calls` array with `id`, `function.name`, `function.arguments`

**Anthropic Tool Use:**
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
- Uses content blocks: `tool_use` block (from model) and `tool_result` block (from user)
- Supports `tool_choice`: "auto", "any", or specific tool
- Claude interleaves reasoning text with tool calls

**Model Context Protocol (MCP):**
- Anthropic's open standard for connecting LLMs to external tools and data sources
- Client-server architecture: the agent (client) connects to MCP servers that expose tools
- Servers can be local processes or remote services
- Standardized protocol means tools work across any MCP-compatible client
- Growing ecosystem of pre-built MCP servers (filesystem, GitHub, databases, Slack, etc.)

### Best Practices
- Write clear, specific tool descriptions (the LLM uses these to decide when/how to call)
- Include parameter descriptions with examples
- Use enums for constrained parameters
- Handle errors gracefully and return informative error messages
- Validate tool inputs before execution
- Implement timeouts and rate limiting
- Consider tool choice control for reliability

### Key Challenges
- Tool selection accuracy (choosing the right tool)
- Argument extraction accuracy (parsing correct values)
- Error handling and recovery
- Security (preventing prompt injection via tool results)
- Latency from sequential tool calls

---

## Technique 2: RAG (Retrieval-Augmented Generation)

### What It Is
A technique that augments LLM generation with information retrieved from an external knowledge base, enabling the model to access domain-specific, up-to-date, or private information without retraining.

### Origin
Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020), Meta AI

### Architecture

```
                    +-------------------+
  User Query ------>| Embedding Model   |-----> Query Vector
                    +-------------------+
                                                    |
                                                    v
                    +-------------------+    +--------------+
                    | Vector Database   |<-->|   Retrieval  |
                    | (Knowledge Base)  |    |   (Top-K)    |
                    +-------------------+    +--------------+
                                                    |
                                                    v
                    +-------------------+    +--------------+
                    |       LLM         |<---|  Retrieved   |
                    |   (Generator)     |    |   Context    |
                    +-------------------+    +--------------+
                           |
                           v
                      Final Answer
```

### Pipeline Steps

**Indexing (Offline):**
1. **Load**: Ingest documents (PDF, HTML, Markdown, databases, APIs)
2. **Chunk**: Split documents into manageable pieces
   - Fixed-size chunking (e.g., 500 tokens with 50-token overlap)
   - Semantic chunking (split at natural boundaries: paragraphs, sections)
   - Recursive chunking (try largest splitter first, then smaller ones)
   - Document-aware chunking (respect document structure)
3. **Embed**: Convert chunks to dense vectors using an embedding model
   - Popular models: OpenAI text-embedding-3-small/large, Cohere embed-v3, BGE, E5
4. **Store**: Save vectors + metadata in a vector database
   - Databases: Pinecone, Weaviate, ChromaDB, Qdrant, Milvus, pgvector, FAISS

**Retrieval (Online):**
1. **Embed query**: Convert user query to a vector using the same embedding model
2. **Search**: Find top-K most similar chunks via approximate nearest neighbor (ANN) search
   - Similarity metrics: cosine similarity, dot product, Euclidean distance
3. **Rerank** (optional): Use a cross-encoder to re-score and reorder results for relevance
   - Models: Cohere Rerank, BGE-reranker, cross-encoder/ms-marco
4. **Filter** (optional): Apply metadata filters (date, source, category)

**Generation:**
1. **Construct prompt**: Insert retrieved chunks into the LLM prompt as context
2. **Generate**: LLM produces an answer grounded in the retrieved context
3. **Cite** (optional): Include source references in the response

### Advanced RAG Techniques

| Technique | Description |
|-----------|-------------|
| **Hybrid search** | Combine dense (vector) and sparse (BM25/keyword) retrieval |
| **HyDE** | Hypothetical Document Embedding -- generate a hypothetical answer first, then search for similar real documents |
| **Multi-query RAG** | Generate multiple query variations and retrieve for each |
| **Parent document retrieval** | Retrieve small chunks but return their parent documents for more context |
| **Self-query** | LLM generates structured filters alongside the semantic query |
| **Contextual compression** | Compress retrieved documents to extract only relevant sentences |
| **Agentic RAG** | Agent decides when and what to retrieve, may query multiple sources |
| **Corrective RAG (CRAG)** | Evaluate retrieval quality; if poor, fall back to web search |
| **Graph RAG** | Build a knowledge graph from documents, then traverse it for retrieval (Microsoft Research, 2024) |
| **Late chunking** | Embed full documents first, then chunk the embeddings (preserves cross-chunk context) |

### Evaluation Metrics
- **Retrieval**: Precision@K, Recall@K, MRR, NDCG, Hit Rate
- **Generation**: Faithfulness, Relevance, Answer Correctness
- **Frameworks**: RAGAS, TruLens, LangSmith evaluation

---

## Technique 3: Chain-of-Thought (CoT) Prompting

### What It Is
A prompting technique that encourages the LLM to generate intermediate reasoning steps before producing a final answer, dramatically improving performance on complex reasoning tasks.

### Origin
Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (2022), Google Brain

### Variants

**Few-shot CoT:**
Provide examples that include step-by-step reasoning:
```
Q: Roger has 5 tennis balls. He buys 2 cans of 3 each. How many does he have now?
A: Roger starts with 5 balls. 2 cans of 3 = 6 balls. 5 + 6 = 11. The answer is 11.

Q: [actual question]
A: [model generates reasoning + answer]
```

**Zero-shot CoT:**
Simply append "Let's think step by step" to the prompt (Kojima et al., 2022):
```
Q: [question]
A: Let's think step by step.
```

**Auto-CoT:**
Automatically generate diverse CoT demonstrations by clustering questions and using zero-shot CoT on representative examples (Zhang et al., 2022).

### Why It Works
- Forces the model to decompose complex problems
- Each reasoning step is a simpler task
- Intermediate results are available for subsequent steps
- Exposes the reasoning process (interpretable)
- Particularly effective for math, logic, and multi-hop reasoning

---

## Technique 4: Tree-of-Thought (ToT)

### What It Is
An extension of Chain-of-Thought that explores multiple reasoning paths in a tree structure, evaluates intermediate thoughts, and uses search algorithms (BFS, DFS) to find the best reasoning path.

### Origin
Yao et al., "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" (2023)

### How It Works

```
                    [Problem]
                   /    |    \
                  /     |     \
            [Thought   [Thought   [Thought
              1a]        1b]        1c]
             /  \         |          X (pruned)
            /    \        |
     [Thought  [Thought [Thought
       2a]      2b]      2d]
        |        X        |
        |     (pruned)    |
     [Solution         [Solution
       A]                B]
```

1. **Thought decomposition**: Break the problem into steps where each step requires a "thought"
2. **Thought generation**: Generate multiple candidate thoughts at each step
3. **Thought evaluation**: Use the LLM to evaluate each thought (vote, score, or classify as sure/maybe/impossible)
4. **Search**: Use BFS or DFS with backtracking to explore the tree
5. **Selection**: Choose the best path to the solution

### When to Use
- Problems requiring exploration (puzzles, creative writing, game playing)
- Tasks where initial approaches may lead to dead ends
- When you need to consider multiple strategies
- NOT for simple, straightforward tasks (overkill)

---

## Technique 5: Self-Consistency

### What It Is
A decoding strategy that samples multiple diverse reasoning paths from the LLM and selects the most consistent answer by majority voting.

### Origin
Wang et al., "Self-Consistency Improves Chain of Thought Reasoning in Language Models" (2022), Google Brain

### How It Works

```
                      [Question]
                     /    |    \
                    /     |     \
              [CoT       [CoT       [CoT
              Path 1]    Path 2]    Path 3]
                |          |          |
            Answer: 42  Answer: 42  Answer: 37
                \          |          /
                 \         |         /
              +---v--------v--------v---+
              |    Majority Vote        |
              |    Answer: 42 (2/3)     |
              +-------------------------+
```

1. Sample N reasoning paths using temperature > 0
2. Extract the final answer from each path
3. Take the majority vote across all answers

### Strengths
- Simple to implement (just sample multiple times)
- Significant accuracy improvement over single-path CoT
- Works with any CoT variant
- No training required

### Limitations
- N times more expensive (N samples)
- Only works when there's a definitive "answer" to vote on
- Less effective for open-ended generation tasks

---

## Technique 6: Prompt Chaining

### What It Is
Decomposing a complex task into a sequence of simpler prompts, where the output of one prompt becomes the input (or part of the input) to the next.

### How It Works

```
Input --> [Prompt 1] --> Output 1 --> [Prompt 2] --> Output 2 --> [Prompt 3] --> Final Output
          (Extract)                   (Analyze)                   (Format)
```

### Example: Research Report Generation

```
Step 1: "Extract the key claims from this article: {article}"
        -> claims_list

Step 2: "For each claim, assess its validity and find supporting evidence: {claims_list}"
        -> evidence_analysis

Step 3: "Write a balanced research summary based on this analysis: {evidence_analysis}"
        -> final_report
```

### Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| **Sequential** | Linear chain of prompts | Extract -> Analyze -> Summarize |
| **Conditional** | Branch based on output | Classify -> (if positive: elaborate, if negative: counter) |
| **Parallel + Merge** | Process in parallel, then combine | Analyze from perspective A, B, C -> Synthesize |
| **Iterative** | Loop until quality threshold met | Write -> Critique -> Revise -> Critique -> ... |
| **MapReduce** | Map prompt over items, reduce results | Summarize each chapter -> Combine into book summary |

### Strengths
- Each step is simpler and more reliable
- Intermediate results are inspectable
- Can use different models/temperatures per step
- Easy to test and debug individual steps
- Natural checkpoints for human review

### Limitations
- Increased latency (sequential calls)
- Error propagation between steps
- More complex pipeline to manage
- Total cost may be higher than single-call approach

---

## Technique 7: Additional Notable Techniques

### Structured Output / JSON Mode
Force the LLM to output valid JSON or conform to a specific schema:
- OpenAI: `response_format: { type: "json_object" }` or Structured Outputs with JSON Schema
- Anthropic: Tool use as structured output, or constrained generation
- Libraries: Instructor, Outlines, Guidance

### Few-Shot Prompting for Agents
Provide examples of successful agent interactions (thought-action-observation traces) to guide behavior.

### System Prompts / Meta-Prompting
Craft detailed system prompts that define:
- Agent persona and capabilities
- Available tools and their usage instructions
- Output format requirements
- Safety guardrails and constraints
- Examples of desired behavior

### Constrained Decoding
Restrict the LLM's output space to valid tokens:
- Grammar-based: only generate tokens that produce valid JSON, SQL, code
- Libraries: Outlines, LMQL, SGLang

### Mixture of Agents
Route different tasks or sub-tasks to different models based on complexity:
- Simple tasks -> small/fast model (Haiku, GPT-4o-mini)
- Complex tasks -> large/capable model (Opus, GPT-4o, o1)
- Cost and latency optimization

## References

- Lewis et al., "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks" (2020)
- Wei et al., "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" (2022)
- Wang et al., "Self-Consistency Improves Chain of Thought Reasoning in Language Models" (2022)
- Yao et al., "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" (2023)
- Kojima et al., "Large Language Models are Zero-Shot Reasoners" (2022)
- Schick et al., "Toolformer: Language Models Can Teach Themselves to Use Tools" (2023)
- Patil et al., "Gorilla: Large Language Model Connected with Massive APIs" (2023)
- Gao et al., "Retrieval-Augmented Generation for Large Language Models: A Survey" (2024)
- Edge et al., "From Local to Global: A Graph RAG Approach to Query-Focused Summarization" (2024)
