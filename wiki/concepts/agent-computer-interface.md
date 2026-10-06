---
title: Agent-Computer Interface (ACI)
type: concept
created: 2026-06-19
updated: 2026-10-06
sources:
  - swe-agent-yang-2024.md
status: draft
importance: medium
tags:
  - 2026
---

# Agent-Computer Interface (ACI)

The Agent-Computer Interface (ACI) is the abstraction of a purpose-built tool and observation surface designed for an [[llm-agent|LLM agent]] rather than for a human operator. Carefully shaping the available commands and the feedback they return lets the agent act more reliably and with fewer mistakes. The concept was introduced and demonstrated by [[swe-agent]] as a core part of [[code-as-harness]] design.

## References

- [[swe-agent-yang-2024]]

## Related 2026 sources

- [[seeing-like-an-agent-shihipar-2026]] — Claude Code tool-design lessons: structured tools where the harness parses output, grep over RAG, fewer tools

## Lineage sources

- [[building-effective-agents-anthropic-2024]] — 'prompt engineering your tools': the ACI appendix (Dec 2024)
- [[aima-ch2-intelligent-agents-litman-pitt]] — sensors and actuators in classical agents
