---
title: "Cross-Agent Comparison (2026): claude-code / opencode / pi-mono / hermes / openclaw"
type: comparison
created: 2026-05-14
updated: 2026-05-14
sources:
  - comparison.md
status: complete
importance: high
---

# Cross-Agent Comparison (2026)

A structured comparison across five 2026-era LLM agent systems:

- **[[claude-code]]** — Anthropic's official terminal coding agent (TS/Ink)
- **[[opencode]]** — sst/opencode (TS/Bun, HTTP/SSE server + many clients)
- **[[pi-mono|pi]]** — badlogic's minimalist coding agent (TS/Node, 7 tools, "no X" manifesto)
- **[[hermes-agent]]** — Nous Research's self-improving multi-surface agent (Python)
- **[[openclaw]]** — single-operator personal assistant gateway (TS/Node, embeds pi-mono)

Synthesizing [[cross-agent-comparison-doc-2026|the source comparison document]].

## At-a-glance table

| Axis | claude-code | opencode | pi | hermes | openclaw |
|---|---|---|---|---|---|
| **Maker** | [[anthropic|Anthropic]] | SST + community | Mario Zechner | Nous Research | community |
| **Language** | TypeScript | TypeScript | TypeScript | Python | TypeScript |
| **Runtime** | Node (Ink) | Bun (Effect) | Node | CPython | Node (≥22.12) |
| **Process model** | single binary | server + clients | single binary | multi-process | Gateway + clients |
| **Primary surface** | terminal REPL | terminal TUI (also web/desktop/mobile) | terminal TUI | terminal + chat platforms | chat + menu-bar + CLI |
| **Provider story** | Anthropic-first | Vercel AI SDK + models.dev (23 bundled) | own pi-ai (~25 providers) | OpenAI-format + ~20 providers | inherits pi-ai |
| **Tool count** | ~40 | ~17 | **7** | 25+ | pi's 7 + ~13 OpenClaw |
| **MCP** | first-class | first-class | refused | first-class | refused in core |
| **ACP** | not in tree | yes (Zed) | extension | yes | yes (server + client) |
| **Sub-agents** | first-class | first-class | **refused** | yes (delegate_task) | yes-but-discouraged |
| **Sandbox** | OS-level adapter | none in core | none in core | 7 backends | tiered Docker |
| **Permissions** | rules+classifier+plan+hooks | pattern-rule engine, SSE | **none in core** | per-tool approval | per-session approval |
| **Memory** | nested CLAUDE.md + auto + "dream" | session summaries + snapshot git | AGENTS.md + JSONL sessions | 4 layers | inherited from pi |
| **Skills** | SKILL.md (Anthropic) | SKILL.md (walks multiple) | SKILL.md (Agent Skills v1) | SKILL.md (Anthropic + agentskills.io) | SKILL.md (inherited from pi) |

## Architectural splits the comparison reveals

### 1. Maximalist vs. minimalist tool philosophy

[[claude-code|Claude Code]] (~40 tools) vs. [[pi-mono|pi]] (7 tools). Two opposite stances:

- **Anthropic's claim**: anticipate the user's needs; ship a rich, well-typed toolbox so most tasks are covered without per-user setup.
- **pi's claim**: keep the core small and predictable; users build what they need as extensions.

[[opencode]] sits in the middle (~17 tools), [[openclaw]] adds ~13 to pi's 7 for its specific use case, [[hermes-agent]] reaches 25+ with a heavy bias toward memory / session management.

### 2. MCP: in-tree vs. refused

Three of five agents (Claude Code, opencode, Hermes) embrace MCP as a first-class integration surface. Two (pi, OpenClaw) refuse it in core and prescribe a bridge if you want it. The disagreement is fundamentally about whether external tool integration belongs *in* the agent runtime or *adjacent* to it.

### 3. Process model: binary vs. server vs. daemon

- **Binary** (pi, Claude Code): one process, runs in your terminal, dies when you close it.
- **Server + clients** (opencode): the agent loop is a service; UIs are clients that connect to it. Enables shared state across surfaces.
- **Daemon + clients** (OpenClaw, Hermes-gateway-mode): a long-lived background process that maintains state and serves multiple clients. The "personal AI assistant that's always on" pattern.

Notable: this maps onto a long-term axis. *Coding tools* → terminal binaries. *Personal assistants* → daemons.

### 4. Sub-agents: first-class vs. refused

Claude Code and opencode treat sub-agents as core infrastructure with built-in roles (Explore, Plan, code-reviewer). pi explicitly refuses sub-agents — the user is expected to be the orchestrator. OpenClaw allows but discourages multi-agent setups. Hermes embraces sub-agents with explicit leaf/orchestrator typing.

The split tracks "how much agency does the system permit itself" — pi's lean stance is partly a refusal to delegate to recursive copies of itself; the multi-agent embracers are betting that compound agency is the future.

### 5. Memory as first-class subsystem

- **None in core** (pi): AGENTS.md walk + JSONL session tree. Memory is an extension.
- **Single layer** (OpenClaw, opencode): inherited or session-summaries-only.
- **Multi-layer with learning loop** (Hermes): 4 distinct layers, an auto-archive curator, FTS5 over sessions.
- **Auto-consolidating** (Claude Code): per-project file-based memory with a background "auto-dream" consolidator. Closest to Hermes's "learning loop" in production-grade form.

### 6. Skills as a de-facto interop standard

All five agents read `SKILL.md` files in some form. The format originated with Anthropic's Agent Skills v1 specification and has been adopted by pi-skills, Hermes (via agentskills.io), opencode (walks `~/.claude/skills/`), and OpenClaw (inherits from pi). This is the closest the LLM-agent landscape has come to an interoperability standard.

## Notes on the source

This comparison is synthesized from `sources/comparison.md`, a ~30KB cross-cut document that itself is read against the five per-agent docs in `sources/{claude-code, hermes-agent, openclaw, opencode, pi-mono}.md`. For full evidence and citations into specific source files, read those docs directly.

## References

- [[cross-agent-comparison-doc-2026]] (source summary)
- [[claude-code-agent-doc-2026]]
- [[hermes-agent-doc-2026]]
- [[openclaw-agent-doc-2026]]
- [[opencode-agent-doc-2026]]
- [[pi-mono-agent-doc-2026]]
