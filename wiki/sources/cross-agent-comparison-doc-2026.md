---
title: "Cross-Agent Comparison (claude-code / opencode / pi-mono / hermes / openclaw)"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - comparison.md
arxiv_id: null
venue: Technical write-up (latest-agents research corpus)
authors:
  - "(unattributed; internal technical doc)"
year: 2026
tags:
  - 2026
status: complete
importance: high
introduces:
  - "[[cross-agent-comparison-2026]]"
---

# Cross-Agent Comparison (claude-code / opencode / pi-mono / hermes / openclaw)

**Source**: `sources/comparison.md` (29KB technical document)
**Origin**: Side-by-side comparison of the five agent systems documented in `latest-agents/docs/agents/`, May 2026.

## Summary

A structured comparison across five 2026-era LLM agent systems:

- **[[claude-code]]** — Anthropic's official terminal coding agent (TS/Ink)
- **[[opencode]]** — sst/opencode (TS/Bun, HTTP/SSE server + multiple clients)
- **[[pi-mono]]** — badlogic's minimalist coding agent (TS/Node, 7 tools, "no X" manifesto)
- **[[hermes-agent]]** — Nous Research's self-improving multi-surface agent (Python)
- **[[openclaw]]** — single-operator personal assistant gateway (TS/Node, embeds pi-mono)

Compared along axes including: maker, language, runtime, process model, primary surface, provider support, tool count, MCP support, ACP support, sub-agents, sandbox, permissions, memory, and skills format. The full source is a 29KB document with detailed comparison tables and prose contrasts — see `sources/comparison.md` for full evidence.

## Key contrasts at a glance

| Axis | claude-code | opencode | pi-mono | hermes-agent | openclaw |
|---|---|---|---|---|---|
| **Language** | TypeScript | TypeScript | TypeScript | Python | TypeScript |
| **Runtime** | Node (Ink) | Bun (Effect) | Node | CPython | Node |
| **Process model** | single binary | server + clients | single binary | multi-process | one daemon + clients |
| **Primary surface** | terminal REPL | terminal TUI (also web/desktop/mobile) | terminal TUI | terminal + chat | chat + menu-bar + CLI |
| **Tool count** | ~40 | ~17 | **7** | 25+ | pi-mono's 7 + ~13 OpenClaw |
| **MCP** | first-class | first-class | refused | first-class | refused in core |
| **Sub-agents** | first-class | first-class | refused | yes | yes-but-discouraged |
| **Sandbox** | OS-level | none in core | none in core | 7 backends | tiered Docker |
| **Memory** | nested CLAUDE.md + auto-memory + "dream" consolidator | session summaries + snapshot git | AGENTS.md + JSONL sessions | 4 layers (MEMORY.md + plugin + FTS5 + curator) | inherited from pi |
| **Skills** | SKILL.md (Anthropic) | SKILL.md (walks multiple paths) | SKILL.md (Agent Skills v1) | SKILL.md (Anthropic-inspired + agentskills.io) | SKILL.md (inherited from pi) |

## Major architectural splits the comparison reveals

1. **Tool count: maximalist vs. minimalist**. Claude Code at ~40 vs. pi at 7. The Anthropic stance is "anticipate the user's needs"; the pi stance is "user builds what they need."
2. **MCP: in-tree vs. refused**. Anthropic, opencode, Hermes are pro-MCP. pi and OpenClaw are anti-MCP-in-core ("if you want it, bridge it").
3. **Process model: binary vs. server vs. daemon-and-clients**. Claude Code is one binary; opencode is a server with many clients; OpenClaw is a daemon with many clients (one of which is the *user*'s persistent assistant).
4. **Sub-agents: first-class vs. refused**. Claude Code and opencode embrace; pi explicitly refuses; OpenClaw allows but discourages.
5. **Multi-channel: code-focused vs. multi-channel**. Claude Code, opencode, pi are coding agents. Hermes and OpenClaw are *personal assistants* that show up on chat platforms and outside the terminal.
6. **Memory: lightweight vs. multi-layer**. pi keeps memory out of core; Hermes has 4 distinct layers and treats memory as a learning loop; OpenClaw has single-slot inherited memory.
7. **Skills as an interop format**: all five converge on SKILL.md from Anthropic's Agent Skills v1, which has become a de-facto cross-agent standard.

## Entities Mentioned

- [[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]] — the five agents being compared.

## Concepts Discussed

- [[llm-agent]] — five comparison points
- [[tool-use]] — varying philosophies
- [[skill-md-format]] — emergent cross-agent standard
- MCP — variable adoption
- ACP (Agent Client Protocol) — cross-IDE integration standard

## References

_Original source: `sources/comparison.md`_
