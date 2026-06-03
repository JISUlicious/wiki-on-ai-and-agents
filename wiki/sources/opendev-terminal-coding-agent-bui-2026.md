---
title: "Building Effective AI Coding Agents for the Terminal: Scaffolding, Harness, Context Engineering, and Lessons Learned (OpenDev)"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - opendev-terminal-coding-agent-bui-2026.pdf
arxiv_id: "2603.05344"
authors:
  - Nghi D. Q. Bui
first_author: Nghi D. Q. Bui
year: 2026
tags: [2026]
status: complete
importance: medium
---

# Building Effective AI Coding Agents for the Terminal (OpenDev)

Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

This paper documents the architecture, design trade-offs, and lessons learned from building OpenDev, an open-source, terminal-native coding agent written in Rust. It frames the engineering response to long-running CLI agents around two phases: scaffolding (assembling system prompt, tool schemas, and a sub-agent registry before the first prompt) and the harness (orchestrating tool dispatch, context management, and safety at runtime). The central problems are managing finite context windows over long sessions, preventing destructive shell operations, and extending capabilities without overflowing the prompt budget.

OpenDev is built as a compound AI system with a four-level hierarchy (session → agent → workflow → LLM) that allows workload-specialized model routing, a dual-agent split separating planning from execution, lazy tool discovery via MCP, and Adaptive Context Compaction (ACC) — a five-stage pipeline that progressively reduces older observations. It adds an experience-driven memory pipeline that accumulates project knowledge across sessions and event-driven system reminders to counteract instruction fade-out.

## Key points

- Two-phase framing: **scaffolding** (static assembly) vs. **harness** (runtime orchestration of tool dispatch, context, safety).
- Compound AI system with per-workflow model routing; a compact model handles compaction/summarization, larger models handle reasoning.
- Dual-agent architecture separates "thinking" from "action," enforcing explicit reasoning phases.
- Adaptive Context Compaction (ACC): five-stage progressively aggressive reduction calibrated from API-reported token counts; large tool outputs are offloaded to the filesystem.
- Safety via capability gating and a runtime approval system; lesson "make unsafe tools invisible, not blocked."
- Transferable lessons: treat context as a budget not a buffer; inject reminders at the point of decision; design tools to absorb LLM imprecision; bound every resource that grows with session length.

## Concepts & entities

- [[code-as-harness]]
- [[agentic-harness-engineering]]
- [[context-assembly]]
- [[agentic-context-engineering]]
- [[memory-management]]
- [[tool-use]]
- [[mcp]]
- [[claude-code]]
- [[opencode]]
- [[swe-bench]]

## References

- [arXiv:2603.05344](https://arxiv.org/abs/2603.05344)
- OpenDev codebase: https://github.com/opendev-to/opendev
