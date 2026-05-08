---
title: "Agentic Retrieval-Augmented Generation: A Survey on Agentic RAG"
authors:
  - Aditi Singh
  - Abul Ehtesham
  - Saket Kumar
  - Tala Talaei Khoei
  - Athanasios V. Vasilakos
venue: arXiv preprint
year: 2025
url: https://arxiv.org/abs/2501.09136
tags:
  - survey
  - agentic-rag
  - retrieval-augmented-generation
  - self-rag
  - corrective-rag
  - adaptive-rag
  - multi-agent
  - LLM-agents
status: done
---

## TL;DR

Comprehensive survey on Agentic RAG -- the integration of autonomous AI agents into
Retrieval-Augmented Generation pipelines. Traces the evolution from Naive RAG (retrieve-
then-read) through Advanced/Modular RAG to fully Agentic RAG systems. Introduces a taxonomy
based on agent cardinality (single vs. multi-agent), control structure, autonomy level,
and knowledge representation. Covers key systems including Self-RAG, CRAG, and Adaptive
RAG, along with agentic design patterns: reflection, planning, tool use, and multi-agent
collaboration.

## Motivation & Problem

Traditional RAG suffers from several fundamental limitations:

1. **Indiscriminate retrieval**: Naive RAG retrieves a fixed number of passages for every
   query, regardless of whether retrieval is actually needed or helpful.
2. **No quality control**: Retrieved passages are passed to the generator without evaluation
   of relevance, support, or contradictions.
3. **Static pipelines**: The retrieve-then-read pipeline is rigid -- no iterative refinement,
   no adaptive routing, no self-correction.
4. **Single-step reasoning**: Complex queries requiring multi-hop reasoning or synthesis
   across multiple documents are poorly served by single-retrieval pipelines.

Agentic RAG addresses these limitations by embedding autonomous agents that can dynamically
decide *when* to retrieve, *what* to retrieve, *how* to evaluate retrieved content, and
*whether* to iterate.

## Method

### Evolution of RAG Paradigms

```
+============+     +=============+     +============+     +==============+
| Naive RAG  | --> | Advanced    | --> | Modular    | --> | Agentic RAG  |
|            |     | RAG         |     | RAG        |     |              |
+============+     +=============+     +============+     +==============+
| Query ->   |     | + Re-ranking|     | + Swappable|     | + Autonomous |
| Retrieve ->|     | + Query     |     |   components|    |   agents     |
| Generate   |     |   rewriting |     | + Routing  |     | + Reflection |
|            |     | + Chunk     |     | + Feedback |     | + Planning   |
|            |     |   optimization|   |   loops    |     | + Tool use   |
|            |     |             |     |            |     | + Multi-agent|
+============+     +=============+     +============+     +==============+
```

### Taxonomy of Agentic RAG Architectures

The survey classifies systems along four dimensions:

**1. Agent Cardinality**
```
Single-Agent RAG                    Multi-Agent RAG
+------------------+               +------------------+
| One agent manages|               | Specialized agents|
| all RAG steps:   |               | for each role:   |
| - Retrieval      |               | - Retriever Agent|
| - Evaluation     |               | - Evaluator Agent|
| - Generation     |               | - Generator Agent|
| - Routing        |               | - Router Agent   |
+------------------+               +------------------+
```

**2. Control Structure**
- Sequential: Fixed order of operations
- Router-based: Classifier routes to appropriate sub-pipeline
- Adaptive: Dynamic control flow based on intermediate results
- Hierarchical: Manager agent orchestrates worker agents

**3. Autonomy Level**
- Low: Human-defined rules dictate retrieval decisions
- Medium: Model-based classifiers guide pipeline depth
- High: Fully autonomous agents with self-reflection and planning

**4. Knowledge Representation**
- Dense vectors (embedding-based retrieval)
- Sparse vectors (BM25, TF-IDF)
- Knowledge graphs (structured relationships)
- Hybrid (multi-modal, multi-source)

### Key Agentic RAG Paradigms

#### Self-RAG (Asai et al., ICLR 2024)
- Model emits special reflection tokens (Retrieve, ISREL, ISSUP, ISUSE)
- Decides *on-demand* when to retrieve and how to critique
- Single model handles retrieval decision + generation + self-critique

#### Corrective RAG (CRAG) (Yan et al., 2024)
```
Query --> Retrieve --> Document Grader --> [Relevant]   --> Generate
                                      --> [Ambiguous]  --> Web Search + Generate
                                      --> [Irrelevant] --> Rephrase + Re-retrieve
```
- Introduces lightweight retrieval evaluator for relevance grading
- Three-way routing: relevant, ambiguous, irrelevant
- Falls back to web search when local retrieval fails

#### Adaptive RAG (Jeong et al., 2024)
```
Query --> T5 Classifier --> [Easy]   --> Direct generation (no retrieval)
                        --> [Medium] --> Single-step RAG
                        --> [Hard]   --> Multi-step iterative RAG
```
- Small T5-large classifier predicts query complexity
- Routes to appropriate pipeline depth
- Avoids unnecessary retrieval for simple queries

### Agentic Design Patterns

| Pattern         | Description                                           | Example System    |
|----------------|-------------------------------------------------------|-------------------|
| Reflection     | Agent evaluates own outputs and retrieval quality     | Self-RAG          |
| Planning       | Agent decomposes complex queries into sub-queries     | IRCoT, ReAct      |
| Tool Use       | Agent invokes external tools (search, calculators)    | Toolformer, CRAG  |
| Multi-Agent    | Multiple specialized agents collaborate               | AutoGen RAG       |
| Routing        | Agent selects appropriate retrieval strategy          | Adaptive RAG      |
| Iterative      | Agent performs multiple retrieval-generation cycles   | ITER-RETGEN       |

### Framework Implementations

The survey covers practical implementations across major frameworks:

- **LangChain/LangGraph**: Graph-based agentic RAG with state management, loops,
  and human-in-the-loop support
- **LlamaIndex**: Agentic Document Workflows (ADW) for end-to-end document processing
  with query routing and sub-question decomposition
- **AutoGen**: Multi-agent conversational RAG with specialized retriever/generator agents
- **CrewAI**: Role-based multi-agent RAG with predefined agent archetypes

## Key Innovations

1. **Principled taxonomy**: First systematic classification of Agentic RAG architectures
   along four orthogonal dimensions (cardinality, control, autonomy, knowledge).

2. **Evolution narrative**: Clear articulation of the progression from Naive -> Advanced
   -> Modular -> Agentic RAG, showing how each paradigm addresses the previous one's
   limitations.

3. **Design pattern catalog**: Identifies and categorizes reusable agentic patterns
   (reflection, planning, tool use, multi-agent, routing) that can be composed to
   build complex RAG systems.

4. **Cross-domain analysis**: Examines Agentic RAG applications across healthcare,
   finance, education, and enterprise document processing.

## Experimental Setup

As a survey, the paper does not present original experiments. Instead, it provides:

- **Comparative analysis** of 10+ Agentic RAG systems and their reported results
- **Architectural comparison** across implementation frameworks
- **Design trade-off analysis** for different agentic patterns and configurations

## Results

### Comparative System Analysis

| System       | Agent Type   | Retrieval Control | Self-Reflection | Multi-hop |
|-------------|-------------|-------------------|-----------------|-----------|
| Naive RAG   | None        | Always retrieve    | No              | No        |
| Self-RAG    | Single      | On-demand          | Yes (tokens)    | Limited   |
| CRAG        | Single      | Evaluate + route   | No              | Limited   |
| Adaptive    | Single      | Classify + route   | No              | Yes       |
| IRCoT       | Single      | Interleaved        | No              | Yes       |
| ITER-RETGEN | Single      | Iterative          | Implicit        | Yes       |
| AutoGen RAG | Multi       | Agent-decided      | Via agents      | Yes       |

### Key Findings

1. Self-RAG reduces hallucination rates to ~5.8% across evaluated configurations,
   significantly lower than standard RAG pipelines (12-14%).
2. Corrective RAG improves robustness by falling back to web search when local
   retrieval quality is low, particularly effective for time-sensitive queries.
3. Adaptive RAG reduces inference cost by 30-40% by skipping retrieval for simple queries
   that the model can answer from parametric knowledge alone.
4. Multi-agent RAG architectures show strongest performance on complex multi-hop
   questions but incur higher latency and cost due to inter-agent communication.

## Limitations

1. **Benchmark fragmentation**: Different Agentic RAG systems are evaluated on different
   benchmarks, making direct comparison difficult.
2. **Latency analysis missing**: The survey does not systematically compare inference
   latency across agentic RAG architectures, which is critical for production deployment.
3. **Cost modeling**: Limited analysis of compute and API costs for different agentic
   patterns, despite this being a key practical concern.
4. **Emerging systems**: Rapid development means some recent systems (2025) may not
   be fully covered.
5. **Evaluation metrics**: No unified evaluation framework is proposed for comparing
   Agentic RAG systems across accuracy, latency, cost, and robustness.

## Key Takeaways

1. The evolution from Naive to Agentic RAG represents a shift from static pipelines to
   dynamic, agent-driven systems that can reason about *when and how* to retrieve.

2. Reflection is the most impactful agentic pattern: Self-RAG's ability to self-critique
   retrieval relevance and generation support significantly reduces hallucination.

3. The choice between single-agent and multi-agent architectures involves clear
   trade-offs: single-agent is simpler and lower-latency; multi-agent is more capable
   for complex tasks but harder to debug and more expensive.

4. Router/classifier-based approaches (Adaptive RAG) offer a practical middle ground,
   adding minimal overhead while significantly reducing unnecessary retrieval.

5. Production Agentic RAG systems should compose multiple patterns (e.g., routing +
   reflection + iterative retrieval) rather than relying on a single pattern.

6. The field urgently needs standardized benchmarks and evaluation frameworks that
   assess Agentic RAG systems across multiple dimensions: accuracy, latency, cost,
   robustness to adversarial inputs, and hallucination rate.
