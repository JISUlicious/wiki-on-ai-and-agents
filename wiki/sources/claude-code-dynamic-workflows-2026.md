---
title: "Introducing Dynamic Workflows in Claude Code"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - claude-code-dynamic-workflows-2026.html
introduces:
  - [[dynamic-workflows]]
year: 2026
venue: claude.com blog
publisher: Anthropic
tags:
  - 2026
status: complete
importance: high
---

# Introducing Dynamic Workflows in Claude Code

**Source**: `sources/claude-code-dynamic-workflows-2026.html` (official [[anthropic|Anthropic]] blog)
**Date**: 2026-05-28
**URL**: https://claude.com/blog/introducing-dynamic-workflows-in-claude-code

## Summary

The announcement post for [[dynamic-workflows]] in [[claude-code|Claude Code]]. Claude can now write **orchestration scripts that run tens to hundreds of parallel [[multi-agent|subagents]] in a single session**, checking its own work before anything reaches the user — letting it take on tasks that would normally span quarters in days. A dynamic workflow is a JavaScript script that Claude writes for the task you describe; a runtime executes it in the background while your session stays responsive.

The post frames workflows as the answer to problems too large for single-pass processing: codebase-wide bug hunts, migrations touching hundreds of files, and plans that need stress-testing from multiple angles. It is light on internals (those are in the [[claude-code-dynamic-workflows-harness-2026|companion engineering post]]) and focused on the value proposition, availability, and how to start.

## Key points

- **What it is**: Claude dynamically plans from the prompt, breaks the task into subtasks, distributes them across parallel subagents, and verifies results before integration. Progress saves automatically, so interrupted jobs **resume** without restarting.
- **Two ways to start**: ask Claude directly to create a workflow, or enable the **`ultracode`** setting (effort menu) which sets effort to `xhigh` and lets Claude decide when to deploy a workflow automatically. Auto mode is recommended.
- **Real-world uses by early-access users**: codebase audits (security/auth/input-validation with independent verification), large-scale migrations (framework swaps, API deprecations, language ports across thousands of files), and high-stakes verification (independent solve attempts with adversarial testing).
- **Case study — Bun**: Jarred Sumner used dynamic workflows to port Bun from Zig to Rust — ~750,000 lines, 99.8% test-suite compatibility, in 11 days. One workflow mapped correct Rust lifetimes for every struct field; another produced behavior-identical `.rs` files from `.zig` with parallel agents and dual reviewers per file.
- **Availability**: research preview in the Claude Code CLI, Desktop, and VS Code extension (Max / Team / Enterprise, admin-enabled for Enterprise), and via the Claude API, Amazon Bedrock, Vertex AI, and Microsoft Foundry.
- **Caveat**: workflows consume substantially more tokens than typical sessions — start with scoped tasks.

## Concepts & entities

- [[dynamic-workflows]] — the concept this post introduces.
- [[claude-code]] — the host runtime.
- [[multi-agent]] — the parallel-subagent substrate.
- [[code-as-harness]] — the broader framing (self-authored harness).
- [[anthropic]] — publisher.

## References

- [[dynamic-workflows]]
- [[claude-code-dynamic-workflows-harness-2026]] — the engineering deep-dive companion.
- _Original source: `sources/claude-code-dynamic-workflows-2026.html`_
