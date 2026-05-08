---
title: "MCP & A2A: Agent Interoperability Protocols"
tags: [mcp, a2a, protocol, interoperability, core-concept]
related: [tool_use, multi_agent_systems]
---

## Overview

Two complementary protocols define the 2026 agent ecosystem:
- **MCP** (Model Context Protocol): agent-to-tool communication
- **A2A** (Agent-to-Agent): agent-to-agent communication

```
┌─────────┐     A2A      ┌─────────┐
│ Agent A │ <──────────> │ Agent B │
└────┬────┘              └────┬────┘
     │ MCP                    │ MCP
┌────▼────┐              ┌────▼────┐
│  Tools  │              │  Tools  │
│  Data   │              │  Data   │
└─────────┘              └─────────┘
```

## MCP (Model Context Protocol)

**Created by**: Anthropic (Nov 2024), donated to Linux Foundation AAIF (Dec 2025)

**Purpose**: Standardize how an AI agent connects to external tools, data sources, and services. "USB-C for AI."

**Adoption (Feb 2026)**: 97M+ monthly SDK downloads (Python + TypeScript combined). Adopted by Anthropic, OpenAI, Google, Microsoft, Amazon.

### How It Works

MCP defines a client-server architecture:
- **MCP Host**: the AI application (Claude, ChatGPT, custom agent)
- **MCP Server**: exposes tools, resources, prompts via a standard protocol
- **Transport**: stdio, HTTP/SSE

### Key Concepts

- **Tools**: functions the agent can call (e.g., `search_database`, `send_email`)
- **Resources**: data the agent can read (e.g., files, database records)
- **Prompts**: reusable prompt templates

### Example

An MCP server for Slack provides tools like `send_message`, `search_messages`, `list_channels` -- any MCP-compatible agent can use it without custom integration code.

## A2A (Agent-to-Agent Protocol)

**Created by**: Google (Apr 2025), now under Linux Foundation

**Purpose**: Standardize secure, structured communication between autonomous AI agents from different providers/frameworks.

### How It Works

- **Agent Cards**: JSON documents describing an agent's capabilities, skills, and endpoint
- **Tasks**: units of work exchanged between agents
- **Streaming**: real-time updates on task progress
- **Push Notifications**: async completion alerts

### Key Features

- Framework-agnostic (works across LangGraph, AutoGen, CrewAI, etc.)
- Secure agent discovery and capability negotiation
- Support for long-running tasks

## How They Complement Each Other

| Layer | Protocol | Function |
|-------|----------|----------|
| Agent ↔ Tool | **MCP** | Connect agent to external tools & data |
| Agent ↔ Agent | **A2A** | Enable multi-agent collaboration |

Most modern agentic AI systems leverage **both** protocols:
- MCP for reliable tool and context integration
- A2A for orchestrating teamwork across agents

## Other Protocols (2026)

- **ACP** (Agent Communication Protocol): alternative by IBM
- **UCP**: emerging universal protocol attempting to unify MCP + A2A

## References

- MCP Spec: https://modelcontextprotocol.io
- A2A Spec: https://github.com/google/A2A
- Linux Foundation AAIF: Agentic AI Foundation
