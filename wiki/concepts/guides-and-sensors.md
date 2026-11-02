---
title: Guides and Sensors
type: concept
created: 2026-11-02
updated: 2026-11-02
sources:
  - harness-engineering-for-coding-agent-users-bockeler-2026.html
status: draft
importance: medium
---

# Guides and Sensors

A model of a coding-agent harness as a system of controls, from [[harness-engineering-for-coding-agent-users-bockeler-2026|Böckeler (2026)]]. It uses two independent distinctions.

| | Computational (deterministic) | Inferential (LLM or other model) |
|---|---|---|
| **Guide** (feedforward, before the agent acts) | Code mods, language-server access | `AGENTS.md`, skills |
| **Sensor** (feedback, after the agent acts) | Tests, linters, structural checks | AI code review, "LLM as judge" |

The article argues that the two directions need each other. It treats computational sensors as cheap enough to run on every change and inferential ones as costlier and non-deterministic.

## In this wiki

- [[harness-extension-and-goal-loop-axes]] maps goal-command gates and judges onto the sensor row. That mapping is this wiki's, not Böckeler's.
- [[agentic-harness-engineering]] · [[context-engineering]] · [[guardrailing]]

## References

- [[harness-engineering-for-coding-agent-users-bockeler-2026]]
