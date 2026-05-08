---
title: Tool Use & Function Calling
tags: [tool-use, function-calling, api, core-concept]
related: [agent_architecture, mcp_a2a_protocols]
---

## Definition

Tool use enables LLM agents to interact with external systems -- APIs, databases, code interpreters, web browsers, file systems -- extending their capabilities beyond text generation.

## Evolution

```
MRKL (2022)           -> route to expert modules (conceptual)
Toolformer (2023)     -> learn tool use via self-supervision
ChatGPT Plugins (2023) -> commercial tool use platform
GPT-4 Function Calling -> structured JSON tool calls
Claude Tool Use       -> structured tool use with schemas
MCP (2024-2026)       -> universal protocol for tool connection
```

## How It Works

1. **Tool Definition**: describe available tools with name, description, parameters (JSON Schema)
2. **Tool Selection**: LLM decides when and which tool to call based on task
3. **Parameter Generation**: LLM generates structured input for the tool
4. **Execution**: system executes the tool call
5. **Result Processing**: LLM processes the tool's output and continues

## Approaches

### Prompted Tool Use (ReAct-style)
```
Thought: I need to search for X
Action: search("query")
Observation: [results]
```

### Native Function Calling (modern APIs)
```json
{
  "type": "function",
  "function": {
    "name": "search",
    "arguments": {"query": "..."}
  }
}
```

### Learned Tool Use (Toolformer)
Model fine-tuned to insert API calls when beneficial.

## Agent-Computer Interface (ACI)

Key insight from SWE-Agent: the design of the tool interface matters as much as the model.
- Clear, concise output formats
- Error messages that help the LLM recover
- Tools designed for LLM ergonomics, not human ergonomics

## MCP (Model Context Protocol)

See: `mcp_a2a_protocols.md` for the universal protocol standardizing tool connections.
