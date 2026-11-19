---
title: "Agent Harness Code Study (2026-11): ten coding agents compared"
type: comparison
created: 2026-11-18
updated: 2026-11-18
sources:
  - codex-cli-codebase-study-2026-11-18.md
  - openhands-codebase-study-2026-11-18.md
  - goose-codebase-study-2026-11-18.md
  - cline-codebase-study-2026-11-18.md
  - hermes-agent-codebase-study-2026-11-18.md
  - pi-codebase-study-2026-11-18.md
  - openclaw-codebase-study-2026-11-18.md
  - claude-code-public-surface-study-2026-11-18.md
  - antigravity-public-surface-study-2026-11-18.md
  - cursor-public-surface-study-2026-11-18.md
status: complete
importance: high
tags:
  - 2026
---

# Agent Harness Code Study (2026-11): ten coding agents compared

On 2026-11-18, ten agents were studied side by side for features, architecture, agent loop, hooks and guardrails. Seven are open source and were read at their latest commit: [[codex-cli]], [[openhands]], [[goose]], [[cline]], [[hermes-agent]], [[pi-mono|Pi]] and [[openclaw]]. Three are closed and were studied from official docs and public repos only: [[claude-code]], [[google-antigravity|Antigravity]] and [[cursor]]. Nothing was built or run, so every cell is what code or docs say, not observed behaviour. Closed-source rows are marked ◇.

[[opencode]] was studied separately later the same day at `79bd2c6f11` (V2, 2.3.0); see [[opencode-codebase-study-2026]].

Per-agent evidence: [[codex-cli-codebase-study-2026]], [[openhands-codebase-study-2026]], [[goose-codebase-study-2026]], [[cline-codebase-study-2026]], [[hermes-agent-codebase-study-2026]], [[pi-codebase-study-2026]], [[openclaw-codebase-study-2026]], [[claude-code-public-surface-study-2026]], [[antigravity-public-surface-study-2026]], [[cursor-public-surface-study-2026]].

## 1. Shape: who builds it and how it runs

| Agent | Maker | Language / runtime | Process model | ACP |
|---|---|---|---|---|
| Codex CLI | [[openai\|OpenAI]] (Apache-2.0) | Rust, ~150 crates | every surface is a client of a JSON-RPC app-server (in-process or daemon) | no |
| OpenHands | OpenHands (MIT) | Python SDK + TS UI | UI drives FastAPI/WebSocket agent servers; local or remote conversation | host: drives other agents |
| Goose | Block → AAIF / Linux Foundation | Rust + Electron | CLI in-process; desktop spawns `goose serve` and talks ACP | server and client |
| Cline | Cline Bot Inc. (Apache-2.0) | TS on Node ≥22 | SDK `AgentRuntime`; optional hub daemon shared by all clients | yes (CLI) |
| Hermes | [[nous-research\|Nous Research]] | Python + TS clients | one gateway process per profile; TUI, desktop, dashboard attach | yes |
| Pi | Mario Zechner / Earendil | TS on Node ≥22.19 | single process; codemode in worker threads | no |
| OpenClaw | openclaw org (MIT) | TS on Node ≥24.16 | Gateway daemon plus channel clients | server; also embeds agents |
| Claude Code ◇ | [[anthropic\|Anthropic]] | native binary | one engine; local, cloud or Remote Control | not documented |
| Antigravity ◇ | [[google\|Google]] | native binary | one engine shared by app, IDE, CLI and SDK | yes (`agy acp`) |
| Cursor ◇ | Anysphere | VS Code-derived | desktop agent, cloud VMs, CLI, self-hosted workers | yes (`agent acp`) |

**Pattern: the loop has moved out of the UI.** Codex, Cline, OpenHands, Goose, Hermes and Antigravity all now run one engine behind a protocol, with every surface as a client. Cline and OpenHands made this move in 2026. Cline's VS Code extension still ships the old loop as a fallback; OpenHands moved its engine into a separate SDK repo. A single terminal binary is now the exception (Pi).

## 2. Agent loop

| Agent | Parallel tool calls | Loop limit | Compaction trigger | Goal command |
|---|---|---|---|---|
| Codex CLI | yes; tools start while the reply streams | no turn cap; spend budget | min(limit, 90% of window) | `/goal`, the model marks completion; budget |
| OpenHands | opt-in (`tool_concurrency_limit=1`) | 500 iterations + cost budget; stuck detector | event count, token cap or overflow | `/goal` + judge model, 10 rounds |
| Goose | yes | max turns 1000 (`/grind`) | 80% of window | `/goal`: one nudge, no judge |
| Cline | yes, adjacent calls | turn without tool calls ends run | 90% of input budget → 70% | none |
| Hermes | yes, planned by file path | unlimited by default | agent 50% (75% floor <512K); gateway 85% | `/goal`: shell gates then judge, 20 turns |
| Pi | yes; edits queued per file | none in core | window − 16,384 | none in core |
| OpenClaw | yes; tools start while the reply streams | repeat guard stops at 20th identical call | min(window − 16,384, 75%) | `/goal`: model marks done; token budget |
| Claude Code ◇ | read-only tools only | Stop hooks; no-progress guard | near window (~967K on 1M) | `/goal` = prompt Stop hook, small judge model |
| Antigravity ◇ | not documented | hidden `max_steps` | automatic checkpoints | `/goal`: self-reported |
| Cursor ◇ | not documented | stop-hook follow-ups, limit 5 | background summarization; Composer RL-trained self-summary | `/goal` (CLI) |

**Pattern: goal loops split on who judges.** Separate judge: OpenHands, Hermes, Claude Code. The working model judges itself: Codex, OpenClaw, Antigravity. A nudge only: Goose. None in core: Cline, Pi. This extends the axes in [[harness-extension-and-goal-loop-axes]] and [[goal-command-lineage-repos-2026]].

**Pattern: compaction converges on "summary + keep the tail".** Pi and OpenClaw both trigger at window − 16,384 and keep about 20k recent tokens; Cline also keeps 20k. See [[context-compaction]].

## 3. Extension surface

| Agent | Built-in tools (how counted) | Hooks | Hook can block / rewrite | Sub-agents |
|---|---|---|---|---|
| Codex CLI | 22 core + 5–6 multi-agent + ~15 ext | 14 | yes; handlers: command, MCP tool | depth 1, 6 threads, roles only narrow permissions |
| OpenHands | 5 default (+14 browser) | 6 | Pre/Prompt/Stop block; no rewrite | `task` tool, 4 built-ins; workflow scripts |
| Goose | 25 in 11 extensions + 17 bundled MCP | 12 (Open Plugins) | PreToolUse and Stop block; no rewrite | 1 level, 5 at once |
| Cline | 9 default | 7 runtime + 10 file | yes | `spawn_agent`, agent teams |
| Hermes | 68 registered, 41 default | 48 + 4 middleware + shell hooks | yes (exit 2 blocks) | depth 1, 10 concurrent |
| Pi | 8 registered, 4 default | 41 extension events | yes | not in core |
| OpenClaw | 68 core + 7 runtime-only | 43 plugin + 15 internal | yes; can require approval | depth 5, swarm on by default |
| Claude Code ◇ | 45 | 33, 5 handler types | yes; can rewrite tool output | 3 levels, 20 concurrent |
| Antigravity ◇ | ~20 named | 6 shell (+10 SDK) | yes | `research`, `browser`, `self` + custom |
| Cursor ◇ | ~10 tool IDs | 21 | 7 block; `preToolUse` rewrites input | Explore, Browser + custom; depth 2 |

**Pattern: Claude Code's hook contract has become a de facto standard.** Codex names its engine `ClaudeHooksEngine` and reuses the event names and JSON fields. OpenHands' code cites "Claude Code's hook contract" for exit code 2. Hermes ships Claude Code-style shell hooks. Cursor imports Claude Code `settings.json` hooks. Goose follows the separate Open Plugins spec. Codex and OpenHands also load `.claude-plugin/plugin.json`, and Cursor and Goose read `.claude/skills`. See [[hooks-in-modern-agents]].

**Pattern: tool surfaces are shrinking in the prompt, not in the code.** Pi's codemode, OpenClaw's tool search and QuickJS code mode, Antigravity's lazy MCP, Cursor's "dynamic context discovery" and Claude Code's ToolSearch all show only names, or a code interface, to the model. Cursor reports a 46.9% token cut from this.

## 4. Guardrails

| Agent | Default approval | Model-based reviewer | Sandbox (default) | Notable |
|---|---|---|---|---|
| Codex CLI | 4 policies (untrusted, on-request, granular, never); reviewer defaults to the user | `auto_review` Guardian LLM (opt-in) | Seatbelt / bubblewrap+seccomp / Windows token+firewall; network off in read-only and workspace-write modes | proxy swaps placeholder secrets for real ones after checking the destination |
| OpenHands | confirmation policy | LLM risk label on every call + analyzers | chosen by workspace (Docker, K8s, remote…) | every tool schema carries `security_risk` |
| Goose | **auto** | smart-approve judge; adversary reviewer (opt-in) | none; opt-in container | OSV malware check on extensions |
| Cline | reads/edits/web/MCP auto; commands ask | none | none | checkpoints as git refs |
| Hermes | manual | smart mode (LLM guardian) | 6 backends | hardline patterns can never be approved |
| Pi | no per-tool approval | none | none for shell | project-trust prompt is the only consent gate |
| OpenClaw | **exec full, ask off** | exec auto-reviewer | off; docker/podman/ssh + plugins | turn taint: network results lock owner-only tools |
| Claude Code ◇ | auto mode (classifier) | auto-mode classifier | off; Seatbelt / bubblewrap; Windows none | deny → ask → allow rules |
| Antigravity ◇ | CLI `auto` mode since v1.5.7 (turns the sandbox on) | Auto reviews sandbox-escaping commands | Linux namespaces / Seatbelt; no network | permission grants compile into sandbox mounts and network allowlist |
| Cursor ◇ | Auto-review | classifier, "not a security boundary" | Seatbelt / Landlock+seccomp / WSL2 | hook results merge deny > ask > allow |

**Pattern: an LLM now sits in the approval path almost everywhere.** Eight of ten can send approval decisions to a model instead of the user. Codex and OpenClaw prompt their reviewers to treat injected claims as hostile. Cursor's docs say outright that its reviewer is not a security boundary, and Pi's docs say isolation, not prompts, is the boundary. This is the inferential-control half of [[guides-and-sensors]]; the computational half is the OS sandbox.

**Pattern: secure defaults are uneven.** Codex's two non-dangerous sandbox modes keep the network off, and Antigravity's default `auto` mode turns its sandbox on. Claude Code and OpenClaw ship the sandbox off. Goose and OpenClaw default to running commands without asking. See [[guardrailing]], [[prompt-injection]].

## Corrections to older pages

These studies corrected [[cross-agent-comparison-2026]], [[hooks-in-modern-agents]] and [[fundamental-components-of-harness]]. The main corrections:
- **Hook counts:** Claude Code 21 → 33, OpenClaw 24 → 43, Hermes 46 → 48, Pi 40 → 41.
- **Tool counts:** Pi 7 → 8, Hermes ~85 → 68.
- **Memory:** Claude Code's "auto-dream" consolidator is not documented.
- **Hooks:** OpenClaw's hooks are not merely observational.

Each fix is marked on those pages with a contradiction callout.

## References

- [[codex-cli-codebase-study-2026]] · [[openhands-codebase-study-2026]] · [[goose-codebase-study-2026]] · [[cline-codebase-study-2026]]
- [[hermes-agent-codebase-study-2026]] · [[pi-codebase-study-2026]] · [[openclaw-codebase-study-2026]]
- [[claude-code-public-surface-study-2026]] · [[antigravity-public-surface-study-2026]] · [[cursor-public-surface-study-2026]]
- [[cross-agent-comparison-2026]] · [[hooks-in-modern-agents]] · [[fundamental-components-of-harness]] · [[harness-extension-and-goal-loop-axes]]
