---
title: Cursor
type: entity
created: 2026-11-01
updated: 2026-11-18
sources:
  - cursor-early-changelog-archive.md
  - cursor-semantic-search-blog-2025.md
  - cursor-public-surface-study-2026-11-18.md
status: draft
importance: high
tags:
  - product
---

# Cursor

An AI code editor. Its changelog goes back to at least March 2023; the first release date was not confirmed.

> [!warning] Contradiction
> The 2026-11-01 version of this page described Cursor only as "An AI code editor." As of 2026-11-18, the docs describe a much wider product from Anysphere, Inc.: the editor, an Agents Window, web and iOS agent clients, a CLI, cloud agents in VMs, Bugbot, Automations, an Agent SDK, Origin git hosting and Live Apps ([[cursor-public-surface-study-2026]]; https://cursor.com/docs). The editor is still the main desktop surface.

## Product surface (documented, 2026-11-18)

Everything below comes from public docs. The product is closed source, so none of it was verified in code ([[cursor-public-surface-study-2026]]).

- **Maker**: Anysphere, Inc. There is no `anysphere` page in this wiki yet.
- **Models**: third-party models, plus "Cursor models": Composer 2.5 (see [[composer-2-technical-report-cursor-2026]]) and Grok 4.5–4.7, listed as jointly trained with SpaceXAI.
- **Agent harness**: modes include Agent, Plan, Ask and Debug. Other parts:
  - [[subagents]]: Explore, Browser, and custom ones.
  - [[agent-skills|Skills]]: `SKILL.md`. Cursor also reads `.claude/` and `.codex/` skill and agent directories.
  - Rules: `.mdc` files and `AGENTS.md`.
  - [[mcp|MCP]] and an [[acp|ACP]] server through `agent acp`.
  - 21 `hooks.json` events, and it can import Claude Code hooks.
  - `/goal` and `/loop` commands.
- **Context**: compaction runs through `/summarize`. Composer is trained to self-summarize, and long tool outputs are written to files ("dynamic context discovery"). See [[context-compaction]].
- **Guardrails**: there are three Run Modes: Auto-review (an LLM classifier), Allowlist and Run Everything. The OS sandbox is Seatbelt on macOS, Landlock+seccomp on Linux and WSL2 on Windows. Network and read boundaries are set in `sandbox.json`. Privacy Mode provides zero data retention (ZDR). See [[guardrailing]].
- **Public GitHub**: the `cursor/cursor` repo is an issue tracker. The org's repos hold plugins, examples, an SDK protobuf bridge, a Terraform provider and research code. None of them holds editor or agent source.

## Search history

- June 2023: codebase-wide context ([[cursor-early-changelog]]).
- July 2023 nightly and November 2024 release: agents ([[cursor-early-changelog]]).
- November 2025: measures gains from semantic search with its own embeddings ([[cursor-semantic-search-blog-2025]]).
- March 2026: adds a case for regex search ([[cursor-fast-regex-search-2026]]).
- By September 2026: search docs say it does not store codebase embeddings for search ([[cursor-docs-search-history]]). No reason was found. The 2026-11-18 docs say the same.

See [[agentic-search]], [[composer-2-technical-report-cursor-2026]] and [[cursor-public-surface-study-2026]].
