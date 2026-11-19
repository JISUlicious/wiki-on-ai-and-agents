---
title: "Cross-Agent Comparison (2026): claude-code / opencode / pi-mono / hermes / openclaw"
type: comparison
created: 2026-05-14
updated: 2026-11-18
sources:
  - opencode-codebase-study-2026-11-18.md
  - comparison.md
  - claude-code-public-surface-study-2026-11-18.md
  - pi-codebase-study-2026-11-18.md
  - hermes-agent-codebase-study-2026-11-18.md
  - openclaw-codebase-study-2026-11-18.md
status: complete
importance: high
---

# Cross-Agent Comparison (2026)

A structured comparison across five 2026-era [[llm-agent|LLM agent]] systems:

- **[[claude-code]]** — Anthropic's official terminal coding agent (TS/Ink)
- **[[opencode]]** — anomalyco/opencode, formerly sst/opencode (TS/Bun, HTTP/SSE server + many clients)
- **[[pi-mono|pi]]** — badlogic's minimalist coding agent (TS/Node, 7 tools, small core plus extensions)
- **[[hermes-agent]]** — [[nous-research|Nous Research]]'s self-improving multi-surface agent (Python)
- **[[openclaw]]** — single-operator personal assistant gateway (TS/Node; embedded pi-mono until 2026-05-27)

Synthesizing [[cross-agent-comparison-doc-2026|the source comparison document]].

## At-a-glance table

> [!warning] Contradiction
> The May 2026 version of this table said Pi and OpenClaw refuse MCP, and that OpenClaw inherits its tools, providers, memory and skills from Pi. As of 2026-11-02, both document MCP support, and OpenClaw no longer embeds Pi. See [[agent-capabilities-check-2026-11]] and [[agent-products-repo-history-2026]].

> [!warning] Contradiction (2026-11-18)
> A code and docs restudy changed several cells, marked ‡. opencode V2 (2.3.0) replaced the Vercel AI SDK with its own `@opencode/ai` and now routes MCP tools through Code Mode. Hermes's tool count was "~85" and is 68 registered (41 default). Pi's was 7 and is 8 (4 default). Claude Code's runtime was "Node (Ink)" and is now a native binary, with npm deprecated. OpenClaw's default exec permission is full access with no prompt. The wider ten-agent table is [[agent-harness-code-study-2026-11]].

Rows marked † were rechecked on 2026-11-02 from docs and file listings, not by running the agents. Other rows date from May 2026.

| Axis | claude-code | opencode | pi | hermes | openclaw |
|---|---|---|---|---|---|
| **Maker** | [[anthropic\|Anthropic]] | `anomalyco` org (was `sst`) | Mario Zechner | Nous Research | community |
| **Language** | TypeScript | TypeScript | TypeScript | Python | TypeScript |
| **Runtime** | native binary ‡ | Bun (Effect) | Node ≥22.19 ‡ | CPython | Node ≥24.16 ‡ |
| **Process model** | one engine: local, cloud or Remote Control ‡ | server + clients | single binary | gateway process per profile + clients ‡ | Gateway + clients |
| **Primary surface** | terminal REPL | terminal TUI (also web/desktop/mobile) | terminal TUI | terminal + chat platforms | chat + menu-bar + CLI |
| **Provider story** | Anthropic-first | own `@opencode/ai` (AI SDK only as an adapter) ‡ | own pi-ai (43 provider files) ‡ | 39 provider plugins + native adapters ‡ | own `packages/ai` † |
| **Tool count** † | 45 ‡ | 13 direct + Code Mode catalog ‡ | **8**, 4 on by default ‡ | 68 registered, 41 default ‡ | 68 core ids ‡ |
| **MCP** † | first-class | first-class | supported (2026-09-29) | first-class | client registry + server |
| **ACP** | not on overview page | yes (Zed) | none in tree † | yes | yes (bridge) |
| **Sub-agents** † | first-class | first-class | **skipped in core** | yes (`delegate_task`) | yes (background runs) |
| **Sandbox** † | OS-level, shell commands only; off by default, none on native Windows ‡ | none for shell; Code Mode interpreter confined ‡ | none by default; isolation guide | 6 backends + plugins | off by default; Docker, Podman, SSH, OpenShell, MXC, Crabbox ‡ |
| **Permissions** | rules+classifier+plan+hooks | ordered allow/ask/deny rules, last match wins, no match = ask ‡ | **no per-tool approval**; project-trust prompt ‡ | manual / smart (LLM) / off modes ‡ | exec approvals, LLM reviewer, turn taint; default full + no prompt ‡ |
| **Memory** | CLAUDE.md hierarchy + auto memory † | AGENTS.md + SQLite session log + compaction summaries; no long-term store ‡ | AGENTS.md + JSONL sessions + compaction summaries ‡ | 4 layers | Markdown files + SQLite vector/FTS search, "dreaming" ‡ |
| **Skills** | SKILL.md (Anthropic) | SKILL.md (walks multiple) | SKILL.md (Agent Skills v1) | SKILL.md (Anthropic + agentskills.io) | SKILL.md, bundled + local † |
| **Goal command** † | Stop-hook evaluator | plugin + reviewer step | third-party `pi-goal` only | gates + judge | model or operator |

Goal-command details are in [[goal-command-lineage-repos-2026]].

## Architectural splits the comparison reveals

### 1. Maximalist vs. minimalist tool philosophy

[[claude-code|Claude Code]] (45 tools) vs. [[pi-mono|pi]] (8 tools, 4 on by default). Two opposite stances:

- **Anthropic's claim**: anticipate the user's needs; ship a rich, well-typed toolbox so most tasks are covered without per-user setup.
- **pi's claim**: keep the core small and predictable; users build what they need as extensions.

[[opencode]] documents 13 built-in tools. [[hermes-agent]] registers 68 (41 on by default) and [[openclaw]] 68 core ids; many depend on platform and credentials. These counts are not like for like, because each project counts its own registry.

### 2. MCP: the split has closed

In May 2026 three of five agents embraced MCP and two (pi, OpenClaw) refused it in core. All five now document MCP support. Pi reversed its stance on 2026-09-29, and OpenClaw describes itself as both an MCP server and a runtime integration surface. The earlier disagreement was about whether external tool integration belongs *in* the runtime or *adjacent* to it, and that is now a difference of degree.

### 3. Process model: binary vs. server vs. daemon

- **Binary** (pi): one process, runs in your terminal, dies when you close it. Claude Code started here but now also runs in the cloud and by Remote Control.
- **Server + clients** (opencode): the agent loop is a service; UIs are clients that connect to it. Enables shared state across surfaces.
- **Daemon + clients** (OpenClaw, Hermes-gateway-mode): a long-lived background process that maintains state and serves multiple clients. The "personal AI assistant that's always on" pattern.

Notable: this maps onto a long-term axis. *Coding tools* → terminal binaries. *Personal assistants* → daemons.

### 4. Sub-agents: first-class vs. refused

Claude Code and opencode treat sub-agents as core infrastructure with built-in roles (Claude Code: Explore, Plan, general-purpose). pi still skips sub-agents in core, leaving them to packages. OpenClaw documents sub-agents as background runs, and the "discouraged" wording from May was not found in its current sub-agent doc or README. Hermes embraces sub-agents with explicit leaf/orchestrator typing.

The split tracks "how much agency does the system permit itself" — pi's lean stance is partly a refusal to delegate to recursive copies of itself; the multi-agent embracers are betting that compound agency is the future.

### 5. Memory as first-class subsystem

- **None in core** (pi): AGENTS.md walk, a JSONL session tree and compaction summaries. Long-term memory is an extension.
- **Single layer** (opencode): session summaries.
- **Markdown workspace + search** (OpenClaw): plain files plus a SQLite vector and full-text index, a pre-compaction memory flush, and "dreaming" consolidation.
- **Multi-layer with learning loop** (Hermes): 4 distinct layers, an auto-archive curator, FTS5 over sessions.
- **File index** (Claude Code): a `MEMORY.md` index plus topic files. The May source described a background "auto-dream" consolidator; the 2026-11-18 docs document no consolidator, so treat that claim as unconfirmed.

### 6. Skills as a de-facto interop standard

All five agents read `SKILL.md` files in some form. The format originated with Anthropic's Agent Skills v1 specification and has been adopted by pi-skills, Hermes (via agentskills.io), opencode (walks `~/.claude/skills/`), and OpenClaw. This is the closest the LLM-agent landscape has come to an interoperability standard.

## Notes on the source

This comparison is synthesized from `sources/comparison.md`, a ~30KB cross-cut document that itself is read against the five per-agent docs in `sources/{claude-code, hermes-agent, openclaw, opencode, pi-mono}.md`. For full evidence and citations into specific source files, read those docs directly.

## References

- [[agent-harness-code-study-2026-11]] (ten-agent restudy, 2026-11-18)
- [[agent-capabilities-check-2026-11]]
- [[cross-agent-comparison-doc-2026]] (source summary)
- [[claude-code-agent-doc-2026]]
- [[hermes-agent-doc-2026]]
- [[openclaw-agent-doc-2026]]
- [[opencode-agent-doc-2026]]
- [[pi-mono-agent-doc-2026]]
