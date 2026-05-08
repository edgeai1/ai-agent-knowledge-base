---
title: Claude Computer Use
authors: Anthropic
venue: Product release (Claude 3.5 Sonnet v2)
year: 2024
url: https://docs.anthropic.com/en/docs/computer-use
tags: [gui, computer-use, commercial, product]
status: done
---

## TL;DR

First major commercial API for computer use. Claude can view screenshots, move the mouse, click, type, and navigate desktop and web applications.

## Method

- Agent takes screenshots of the desktop
- Processes them via multimodal LLM (Claude 3.5 Sonnet v2)
- Outputs mouse/keyboard actions (click, type, scroll, etc.)
- Screenshot-based (not DOM-based) for security

## Safety Mitigations

- Screenshot-based approach (no direct DOM access)
- User confirmation for sensitive actions
- Sandboxed execution environments recommended

## Competitors

- OpenAI Operator / CUA (Jan 2025)
- CogAgent (Tsinghua/Zhipu)
- UFO (Microsoft Research)
- OmegaUse (2026)

## Why It Matters

Marked a major milestone in practical GUI agent deployment. Made computer use a recognized product category.
