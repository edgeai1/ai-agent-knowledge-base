---
title: Agent Frameworks Comparison
tags: [agent, framework, langchain, langgraph, autogen, crewai, metagpt, claude-sdk, openai-sdk, dify, coze]
related: [llm_agent_fundamentals, agent_architecture_patterns, agent_techniques]
---

## Definition

Agent frameworks are software libraries and platforms that provide abstractions, primitives, and infrastructure for building LLM-based agents. They handle the boilerplate of agent loops, tool integration, memory management, and multi-agent orchestration.

---

## Framework 1: LangChain / LangGraph

### Overview
- **Developer**: LangChain Inc. (Harrison Chase)
- **Language**: Python, JavaScript/TypeScript
- **License**: MIT
- **Repository**: github.com/langchain-ai/langchain, github.com/langchain-ai/langgraph
- **First Release**: October 2022 (LangChain), January 2024 (LangGraph)

### Architecture
LangChain started as a chain-based framework (sequential LLM calls) and evolved into a modular ecosystem:

- **LangChain Core**: Base abstractions (LLMs, prompts, output parsers, tools)
- **LangChain Community**: Third-party integrations (vector stores, tools, LLM providers)
- **LangGraph**: Graph-based agent orchestration framework (the recommended approach for agents as of 2024+)
- **LangSmith**: Observability, testing, and evaluation platform
- **LangServe**: Deployment as REST APIs

### LangGraph (Agent Framework)
LangGraph models agents as **stateful graphs**:
- **Nodes**: Functions that process state (LLM calls, tool execution, routing logic)
- **Edges**: Transitions between nodes (conditional or unconditional)
- **State**: A shared, typed state object that flows through the graph
- **Checkpointing**: Built-in persistence for long-running agents, human-in-the-loop, and recovery

```python
# LangGraph conceptual example
from langgraph.graph import StateGraph, MessagesState

graph = StateGraph(MessagesState)
graph.add_node("agent", call_model)
graph.add_node("tools", call_tools)
graph.add_edge("__start__", "agent")
graph.add_conditional_edges("agent", should_continue, {"continue": "tools", "end": "__end__"})
graph.add_edge("tools", "agent")
app = graph.compile()
```

### Key Features
- Model-agnostic (OpenAI, Anthropic, Google, open-source)
- Huge ecosystem of integrations (700+ integrations)
- LangGraph supports cycles, branching, parallel execution
- Human-in-the-loop via interrupt/resume
- Streaming support (token-level and event-level)
- LangGraph Cloud for managed deployment

### Strengths
- Largest community and ecosystem
- Flexible and composable
- LangGraph provides fine-grained control over agent behavior
- Excellent documentation and tutorials
- Strong observability via LangSmith

### Limitations
- LangChain (legacy chains/agents) has been criticized for over-abstraction
- Steep learning curve for LangGraph's graph paradigm
- Frequent breaking API changes (historically)
- Can be verbose for simple use cases

---

## Framework 2: AutoGen (Microsoft)

### Overview
- **Developer**: Microsoft Research
- **Language**: Python, .NET (AutoGen 0.4+)
- **License**: MIT (Creative Commons for some components)
- **Repository**: github.com/microsoft/autogen
- **First Release**: September 2023
- **Major Rewrite**: AutoGen 0.4 (AgentChat) -- late 2024, complete redesign

### Architecture (AutoGen 0.4 / AgentChat)
AutoGen 0.4 is a complete rewrite with a layered architecture:

- **Core Layer**: Asynchronous, event-driven agent runtime with message-passing
- **AgentChat Layer**: High-level API for multi-agent conversations
- **Extensions**: Model clients, tools, code execution

Key primitives:
- **Agents**: AssistantAgent, UserProxyAgent, custom agents
- **Teams**: RoundRobinGroupChat, SelectorGroupChat, Swarm, MagenticOneGroupChat
- **Termination conditions**: MaxMessageTermination, TextMentionTermination, etc.
- **Model clients**: OpenAI, Anthropic, etc.

```python
# AutoGen 0.4 conceptual example
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import RoundRobinGroupChat

agent1 = AssistantAgent("coder", model_client=model, tools=[...])
agent2 = AssistantAgent("reviewer", model_client=model)
team = RoundRobinGroupChat([agent1, agent2], max_turns=10)
result = await team.run(task="Build a web scraper")
```

### Key Features
- First-class multi-agent conversations
- Multiple team topologies (round-robin, selector-based, swarm, Magentic-One)
- Built-in code execution (Docker sandbox, local)
- Human-in-the-loop support
- Cross-language support (.NET, Python)
- Magentic-One: pre-built multi-agent team for complex web/file tasks

### Strengths
- Excellent for multi-agent systems
- Clean async-first architecture (0.4)
- Strong code execution capabilities
- Backed by Microsoft Research
- AutoGen Studio: no-code UI for building agent teams

### Limitations
- Breaking changes between 0.2 and 0.4 (migration required)
- Documentation still catching up for 0.4
- Smaller community than LangChain
- Complex setup for advanced scenarios

---

## Framework 3: CrewAI

### Overview
- **Developer**: CrewAI Inc. (Joao Moura)
- **Language**: Python
- **License**: MIT
- **Repository**: github.com/crewAIInc/crewAI
- **First Release**: December 2023

### Architecture
CrewAI uses a role-playing metaphor inspired by real-world team structures:

- **Agent**: An autonomous unit with a role, goal, backstory, and tools
- **Task**: A specific assignment with a description, expected output, and assigned agent
- **Crew**: A team of agents working together on a set of tasks
- **Process**: The workflow type (sequential, hierarchical, or consensual)

```python
# CrewAI conceptual example
from crewai import Agent, Task, Crew, Process

researcher = Agent(
    role="Senior Research Analyst",
    goal="Find the latest AI trends",
    backstory="You are an expert researcher...",
    tools=[search_tool, scrape_tool]
)

writer = Agent(
    role="Tech Writer",
    goal="Write compelling articles",
    backstory="You are a skilled writer..."
)

research_task = Task(description="Research AI agent frameworks", agent=researcher)
write_task = Task(description="Write article based on research", agent=writer)

crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential
)
result = crew.kickoff()
```

### Key Features
- Simple, intuitive API based on real-world team metaphors
- Role-based agent design (role, goal, backstory)
- Sequential and hierarchical process types
- Built-in delegation between agents
- Memory (short-term, long-term, entity memory)
- CrewAI Enterprise for production deployments
- Flows: structured workflows combining crews with procedural code

### Strengths
- Easiest to learn among agent frameworks
- Intuitive mental model (agents as team members)
- Quick to prototype
- Good for content creation, research, analysis pipelines
- Growing ecosystem of pre-built tools

### Limitations
- Less flexible than LangGraph for complex control flows
- Limited support for advanced agent patterns (e.g., dynamic agent creation)
- Relatively young framework
- Debugging can be opaque (agents are chatty)
- Performance overhead from verbose inter-agent communication

---

## Framework 4: MetaGPT

### Overview
- **Developer**: DeepWisdom (Alexander Wu et al.)
- **Language**: Python
- **License**: MIT
- **Repository**: github.com/geekan/MetaGPT
- **First Release**: June 2023
- **Paper**: Hong et al., "MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework" (2023)

### Architecture
MetaGPT structures multi-agent collaboration around **Standardized Operating Procedures (SOPs)** -- mimicking how real software companies operate:

- **Roles**: ProductManager, Architect, ProjectManager, Engineer, QA
- **Actions**: Each role has defined actions (WritePRD, DesignAPI, WriteCode, RunTest)
- **Message Bus**: Publish-subscribe communication between agents
- **Shared Environment**: Shared workspace with documents, code, and artifacts
- **SOP Workflow**: Predefined sequence of role interactions

### Key Features
- Software development lifecycle simulation
- Structured outputs at each stage (PRD, system design, API specs, code, tests)
- Role-based design with clear responsibilities
- Publish-subscribe messaging architecture
- Incremental development with iterative refinement

### Strengths
- Produces high-quality structured software artifacts
- Reduces hallucination through role constraints and output schemas
- Novel SOP approach provides consistency
- Good for code generation and software engineering tasks

### Limitations
- Primarily focused on software development use case
- Complex codebase, harder to extend to other domains
- High token consumption (multiple agents, long outputs)
- Less general-purpose than LangGraph or AutoGen

---

## Framework 5: Claude Agent SDK (Anthropic)

### Overview
- **Developer**: Anthropic
- **Language**: Python, TypeScript
- **License**: MIT
- **First Release**: March 2025 (as part of Claude ecosystem)

### Architecture
The Claude Agent SDK provides a minimal, opinionated framework for building agents with Claude:

- **Agent**: Core agent class with a model, instructions, tools, and optional handoff targets
- **Tools**: Function tools, Bash tool, Computer tool, MCP servers
- **Handoffs**: Transfer control between specialized agents
- **Guardrails**: Input/output validation for safety

```python
# Claude Agent SDK conceptual example
from claude_agent_sdk import Agent, Runner
from claude_agent_sdk.tools import FunctionTool

agent = Agent(
    name="research_assistant",
    model="claude-sonnet-4-20250514",
    instructions="You are a helpful research assistant...",
    tools=[search_tool, calculator_tool],
    handoffs=[specialist_agent]
)

result = Runner.run(agent, messages=[{"role": "user", "content": "..."}])
```

### Key Features
- Tight integration with Claude models
- Model Context Protocol (MCP) for standardized tool connections
- Multi-agent handoffs (swarm-style agent switching)
- Built-in guardrails for safety
- Computer use capability (GUI interaction)
- Tracing and observability

### Design Philosophy
- Minimal abstraction (thin wrapper, not a heavy framework)
- Agents are "just" Claude with tools and instructions
- Emphasize simplicity and Claude-native patterns
- MCP as the universal tool integration standard

### Strengths
- Simple, minimal API
- Best-in-class integration with Claude models
- MCP provides standardized, reusable tool ecosystem
- Computer use for GUI automation
- Strong safety features (guardrails)

### Limitations
- Claude-only (not model-agnostic)
- Newer framework, smaller ecosystem than LangChain
- Fewer built-in orchestration patterns than LangGraph
- MCP ecosystem still growing

---

## Framework 6: OpenAI Agents SDK

### Overview
- **Developer**: OpenAI
- **Language**: Python
- **License**: MIT
- **First Release**: March 2025 (evolution of Swarm framework)
- **Repository**: github.com/openai/openai-agents-python

### Architecture
The OpenAI Agents SDK is a lightweight, production-ready framework:

- **Agent**: An LLM with instructions, tools, and handoff capabilities
- **Handoffs**: Transfer conversation to another agent (inherited from Swarm)
- **Guardrails**: Input/output validators for safety
- **Tracing**: Built-in observability and debugging

```python
# OpenAI Agents SDK conceptual example
from agents import Agent, Runner

agent = Agent(
    name="assistant",
    instructions="You are a helpful assistant...",
    tools=[search_tool, code_tool],
    handoffs=[triage_agent, specialist_agent]
)

result = Runner.run(agent, messages=[...])
```

### Key Features
- Built-in tracing and observability
- Multi-agent handoffs (from Swarm)
- Guardrails for input/output validation
- Context management
- Integration with OpenAI models and tools

### Strengths
- Simple, minimal API (similar philosophy to Claude Agent SDK)
- Native OpenAI model integration
- Built-in tracing
- Production-ready design

### Limitations
- OpenAI-only (not model-agnostic)
- Limited orchestration capabilities compared to LangGraph
- Relatively new, limited community resources

---

## Framework 7: Platform-Based Frameworks (Dify, Coze, etc.)

### Dify
- **Type**: Open-source LLM app development platform (with cloud offering)
- **Repository**: github.com/langgenius/dify
- **Key Features**:
  - Visual workflow builder (drag-and-drop agent design)
  - RAG pipeline with built-in knowledge base management
  - Support for multiple LLM providers
  - Agent and chatbot templates
  - API-first design for embedding
  - Built-in evaluation and monitoring
- **Best For**: Teams wanting a visual, no-code/low-code approach to building AI apps and agents
- **Strengths**: Accessible to non-developers, built-in RAG, good UI
- **Limitations**: Less flexible than code-first frameworks, vendor dependency

### Coze (ByteDance)
- **Type**: Bot/agent building platform
- **Key Features**:
  - Visual bot builder with plugin system
  - Built-in knowledge base, memory, and workflow capabilities
  - Pre-built plugins for common integrations
  - Multi-platform deployment (Discord, Telegram, Slack, web)
  - Workflow automation with triggers
- **Best For**: Building and deploying chatbots and simple agents quickly
- **Strengths**: Easy to use, multi-platform deployment, free tier
- **Limitations**: Less customizable, platform lock-in, primarily consumer-focused

### Others
- **Flowise**: Open-source drag-and-drop LLM flow builder (based on LangChain)
- **Botpress**: Chatbot platform with LLM agent capabilities
- **Wordware**: Collaborative IDE for building LLM agents
- **Relevance AI**: No-code AI agent builder and deployment platform

---

## Comparison Matrix

| Feature | LangGraph | AutoGen 0.4 | CrewAI | MetaGPT | Claude SDK | OpenAI SDK | Dify |
|---------|-----------|-------------|--------|---------|------------|------------|------|
| **Approach** | Graph-based | Multi-agent chat | Role-playing | SOP-based | Minimal wrapper | Minimal wrapper | Visual builder |
| **Multi-agent** | Yes | Excellent | Good | Excellent | Handoffs | Handoffs | Basic |
| **Model-agnostic** | Yes | Yes | Yes | Yes | No (Claude) | No (OpenAI) | Yes |
| **Learning curve** | High | Medium | Low | Medium | Low | Low | Very Low |
| **Flexibility** | Very High | High | Medium | Medium | Medium | Medium | Low |
| **Production-ready** | Yes | Yes | Growing | Growing | Yes | Yes | Yes |
| **Code vs No-code** | Code | Code | Code | Code | Code | Code | Both |
| **Best for** | Complex workflows | Multi-agent | Team simulation | SW dev | Claude apps | OpenAI apps | Rapid prototyping |
| **Community size** | Very Large | Large | Large | Medium | Growing | Growing | Large |

## Decision Guide

- **"I need maximum flexibility and control"** -> LangGraph
- **"I need multi-agent collaboration"** -> AutoGen or CrewAI
- **"I want the simplest possible agent"** -> Claude SDK or OpenAI SDK
- **"I'm building a software engineering agent"** -> MetaGPT
- **"I want no-code/visual building"** -> Dify, Coze, or Flowise
- **"I'm committed to Claude/Anthropic"** -> Claude Agent SDK + MCP
- **"I'm committed to OpenAI"** -> OpenAI Agents SDK

## References

- LangChain docs: python.langchain.com
- LangGraph docs: langchain-ai.github.io/langgraph/
- AutoGen docs: microsoft.github.io/autogen/
- CrewAI docs: docs.crewai.com
- MetaGPT: github.com/geekan/MetaGPT
- Claude Agent SDK: docs.anthropic.com
- OpenAI Agents SDK: github.com/openai/openai-agents-python
- Dify: docs.dify.ai
