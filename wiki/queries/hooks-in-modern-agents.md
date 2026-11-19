---
title: "Hooks and Callbacks in Modern Agent Architecture"
type: query
created: 2026-05-17
updated: 2026-11-18
sources:
  - opencode-codebase-study-2026-11-18.md
  - claude-code-public-surface-study-2026-11-18.md
  - openclaw-codebase-study-2026-11-18.md
  - hermes-agent-codebase-study-2026-11-18.md
  - pi-codebase-study-2026-11-18.md
  - cursor-public-surface-study-2026-11-18.md
  - claude-code.md
  - opencode.md
  - pi-mono.md
  - hermes-agent.md
  - openclaw.md
  - comparison.md
status: complete
importance: high
tags:
  - 2026
---

# Hooks and Callbacks in Modern Agent Architecture

Synthesizing from all 5 documented 2026 agents ([[claude-code]], [[opencode]], [[pi-mono]], [[hermes-agent]], [[openclaw]]) and the [[cross-agent-comparison-2026]] — **~30 named hooks across these systems**. This page organizes them by lifecycle phase and surfaces the architectural decisions each agent made.

The hooks layer sits inside the [[agent-three-layer-model|L2 harness abstraction]] in the three-layer model and is a cross-cutter that also touches control flow.

## By category

### Tool lifecycle (universal — every agent has these)

| Hook | Fires at | Can do | Agents |
|---|---|---|---|
| **PreToolUse** / `before_tool_call` / `beforeToolCall` / `tool.execute.before` | Before each tool invocation | Block, modify args, auto-approve, audit | All 5 |
| **PostToolUse** / `after_tool_call` / `afterToolCall` / `tool.execute.after` / `PostToolUseFailure` | After tool returns (success + failure) | Transform results, gate continuation, trigger memory write | All 5 |
| **`tool_result_persist`** | Before tool result is persisted | Redact secrets, transform for storage | [[openclaw]] |

### Session lifecycle (3+ agents)

| Hook | Fires at | Agents |
|---|---|---|
| **SessionStart** / `session_start` / `on_session_start` | Once at session begin — init, restore memory, register tools | [[claude-code]], [[opencode]], [[hermes-agent]], [[openclaw]] |
| **SessionEnd** / `session_end` / `on_session_end` | Once at session end — cleanup, save memory, final log | [[claude-code]], [[hermes-agent]], [[openclaw]] |
| **Setup** | Initial system setup (pre-session) | [[claude-code]] |
| **`gateway_start` / `gateway_stop`** | Long-lived daemon lifecycle | [[openclaw]] (daemon mode only) |

### Message & prompt

| Hook | Fires at | Agents |
|---|---|---|
| **UserPromptSubmit** | User submits a prompt | [[claude-code]] (first-class) |
| **`chat.message` / `chat.params` / `chat.headers`** | Observe / mutate user message, LLM params, HTTP headers | [[opencode]] V1 only; V2 (2.3.0) renamed its hooks, see [[opencode-codebase-study-2026]] |
| **`message_received` / `message_sending` / `message_sent`** | Per-message lifecycle | [[openclaw]] |
| **`before_message_write`** | Before message persisted | [[openclaw]] |
| **`before_prompt_build`** | Before prompt assembly each turn | [[openclaw]] |
| **Notification** | Agent emits notification | [[claude-code]] |

### LLM-call lifecycle

| Hook | Fires at | Agents |
|---|---|---|
| **`pre_llm_call` / `post_llm_call`** | Every LLM API call | [[hermes-agent]] |
| **`llm_input` / `llm_output`** | LLM input + output points | [[openclaw]] |
| **`before_model_resolve`** | Before provider/model selected — can override | [[openclaw]] |

### Loop control / stopping

| Hook | Fires at | Agents |
|---|---|---|
| **Stop** / `StopFailure` | After a model turn with zero tool calls (loop wants to terminate) | [[claude-code]] (has anti-death-spiral logic). As of 2026-11-18, [[cursor]] `stop` (follow-up loop, limit 5), [[codex-cli]], [[openhands]], [[goose]] (capped at 8) and Antigravity also have one; see [[agent-harness-code-study-2026-11]] |
| **`agent_end`** | Loop exits | [[openclaw]] |

### Memory / compaction

| Hook | Fires at | Agents |
|---|---|---|
| **PreCompact** / `before_compaction` / `experimental.session.compacting` | Before context is compacted | [[claude-code]], [[openclaw]], [[opencode]] |
| **PostCompact** / `after_compaction` | After compaction | [[claude-code]], [[openclaw]] |

This connects to [[memory-management]] (the compaction lifecycle operation) and [[sleep-time-compute]] / [[offline-consolidation]] (which can be triggered from compaction hooks).

### Sub-agent / orchestration

| Hook | Fires at | Agents |
|---|---|---|
| **SubagentStart** / **SubagentStop** | Forked sub-agent lifecycle | [[claude-code]] (2 phases) |
| **`subagent_delivery_target` / `subagent_spawned` / `subagent_progress` / `subagent_ended`** | Sub-agent lifecycle | [[openclaw]] (4 phases — most granular; `subagent_spawning` was removed and `subagent_progress` added by 2026-11-18) |

### Permission

| Hook | Fires at | Agents |
|---|---|---|
| **PermissionRequest** / **PermissionDenied** | Permission system events | [[claude-code]] |
| **`permission.ask`** | Pre-empt permission dialogs | [[opencode]] V1 (V2 keeps one permission hook under a new name) |

### Environment events ([[claude-code]]; Cursor has `workspaceOpen`)

| Hook | Fires at |
|---|---|
| **ConfigChange** | Settings mutated |
| **CwdChanged** / **FileChanged** / **DirectoryAdded** | File-system / workspace state |
| **WorktreeCreate** / **WorktreeRemove** | Worktree lifecycle |
| **InstructionsLoaded** | `CLAUDE.md` or `.claude/rules/*.md` loaded |

## Hook surfaces across ten agents (2026-11-18)

| Agent | Hook events | Handler types / notes |
|---|---|---|
| [[claude-code]] | 33 | command, http, mcp_tool, prompt, agent; `updatedToolOutput`; `PostToolBatch` can stop the loop |
| [[cursor]] | 21 (18 agent, 2 Tab, 1 app) | 7 can block; `preToolUse` rewrites input; prompt-based hooks; imports Claude Code hooks |
| [[codex-cli]] | 14 | command, mcp_tool; engine named `ClaudeHooksEngine` with Claude Code's event names and JSON fields |
| [[goose]] | 12 (Open Plugins spec) | only PreToolUse and Stop block; no input rewrite |
| [[cline]] | 7 runtime + 10 file hooks | `PreCompact` declared but not dispatched |
| [[openhands]] | 6 | command, prompt, agent; exit 2 blocks, citing "Claude Code's hook contract" |
| Antigravity ([[google-antigravity]]) | 6 shell + 10 SDK | a fixed bug let `PreToolUse` `auto_approve` bypass admin review |
| [[hermes-agent]] | 48 + middleware + shell hooks | `pre_tool_call` blocks or modifies; `pre_verify` forces continuation |
| [[pi-mono]] | 41 extension events | plus loop callbacks in agent core |
| [[openclaw]] | 43 plugin + 15 internal | `before_tool_call` can require operator approval and then freezes params |

Claude Code's event names and exit-code-2 contract are now copied by Codex, OpenHands, Hermes and Cursor. Details: [[agent-harness-code-study-2026-11]].

## Two architectural axes

### 1. First-class harness hook vs. plugin-system hook

| Agent | Model | Hook count | Detail |
|---|---|---|---|
| [[claude-code]] | **First-class in harness** | 33 documented events (2026-11-18; was ~21) | Can affect permissions and rewrite tool output; 5 handler types: command, http, mcp_tool, prompt, agent |
| [[openclaw]] | **Plugin-system only** | 43 plugin hooks + 15 internal (2026-11-18; was ~24) | Modifying hooks run in priority order and can block, rewrite params or require operator approval |
| [[opencode]] | Plugin-system only | 21 hooks in V2 (2026-11-18; was ~18) | Only `tool.execute.before` can block; the V1 names `chat.*` and `permission.ask` are gone |
| [[hermes-agent]] | Plugin-system | 48 in `VALID_HOOKS` (2026-11-18; 46 on 2026-11-02) plus 4 middleware kinds and shell hooks (exit 2 blocks) | Includes `pre_verify`, which can keep the agent going; see [[agent-goal-loops-extension-docs-2026]] |
| [[pi-mono]] | **Extension layer** | 2 tool hooks plus ~6 loop callbacks in agent core; 41 extension events (2026-11-18) | `beforeToolCall` / `afterToolCall` in `packages/agent`; the coding agent documents the rest in `docs/extensions.md`, see [[agent-goal-loops-extension-docs-2026]] |

### 2. What hooks can DO — observational → modifying → blocking → compositional

- **Observational** (default for plugins) — read events for logging, telemetry
- **Modifying** — rewrite tool args/results, mutate prompts, inject context
- **Blocking** — abort a tool call, refuse a user prompt, terminate the loop
- **Compositional** — [[hermes-agent|Hermes]]'s curator forks an agent that rewrites its own skills with `skill_manage`, then fires `post_curator_run`. Hooks observe a self-modification mechanism rather than drive it.

> [!warning] Contradiction (2026-11-18)
> This page said Hermes's curator uses `post_tool_call` inside a forked sub-agent. The code at `0e8e421640` shows no `post_tool_call` use in the curator or background review; the only hook it fires is `post_curator_run` ([[hermes-agent-codebase-study-2026]]). The page also called OpenClaw's plugin hooks observational; at `9d6f4a94ad` its `before_tool_call` can block, rewrite or require approval ([[openclaw-codebase-study-2026]]).

## Cross-cutting framings

- The [[agent-three-layer-model]] classifies hooks as L3 implementations under L2 [[guardrailing|harness]], with cross-cutting edges to L2 control-flow.
- [[common-features-of-modern-agents]] does *not* count hooks among the 8 universal features — because [[pi-mono]] keeps them out of its agent core. Near-universal but not strictly baseline.
- The 5 hooks in the **Loop Hooks** graph view (`scripts/evolution-graph.html`) — SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop — are the **canonical subset**. [[pi-mono]] has close equivalents through extensions (`session_start`, `input`, `tool_call`, `tool_result`, `agent_before_settle`), a mapping made by this wiki. The 25+ others are agent-specific extensions of the same architectural idea.

## Gaps and interesting omissions

- **`on-error` / exception hooks** — now more common. [[claude-code]], [[cursor]] and [[goose]] have `PostToolUseFailure`; [[cline]] has `TaskError`. No `PostToolUseFailure`-style hook was found in [[openclaw]] (only `after_tool_call`).
- **`pre-respond` / `before-final-response`** — surprisingly absent across the wiki. The closest analogue is [[claude-code|Claude Code]]'s **Stop** hook (fires before exit when no follow-up tools).
- **Streaming-output hooks** — none of the 5 agents document a per-token or per-chunk output hook, even though all 5 stream output.

## Related queries

- [[common-features-of-modern-agents]] — the 8 universal features (hooks not baseline)
- [[from-transformer-to-modern-agents]] — historical track from Transformer → modern agents
- [[agent-three-layer-model]] — where hooks fit in the L1/L2/L3 architecture

> [!note] Update 2026-10-02 — a third kind of Claude Code hook
> Claude Code v2.1.287 added [[claude-code-mods|mods]]: plugin-shipped JS/TS modules that run in-process and stay resident. Unlike `settings.json` hooks, which run a stateless shell command per event, a mod is middleware. It can observe, rewrite or answer events (`tool.call`, `prompt.submit`, `turn.complete`, `ui.render` and others), keep state, draw UI and register its own tools and commands. The classification above predates mods.

## References

- [[claude-code-agent-doc-2026]] — May source for Claude Code's hook events (21; 33 documented on 2026-11-18, see [[claude-code-public-surface-study-2026]])
- [[opencode-agent-doc-2026]] — opencode's `opencode.json` plugin hook surface
- [[hermes-agent-doc-2026]] — Hermes's curator + plugin lifecycle
- [[openclaw-agent-doc-2026]] — May plugin hook taxonomy (24; 43 at [[openclaw-codebase-study-2026]])
- [[pi-mono-agent-doc-2026]] — minimal-core stance; 2 in core
- [[cross-agent-comparison-doc-2026]] — cross-cutting comparison source
- [[cross-agent-comparison-2026]] — synthesized comparison page

## Related 2026 sources

- [[getting-started-with-claude-code-mods-osmani-2026]] — Osmani's tutorial on Claude Code mods: in-process JS/TS hook modules shipped as plugins
- [[lessons-from-building-claude-code-how-we-use-skills-shihipar-2026]] — Anthropic's internal skill practice: categories, gotchas sections, trigger-style descriptions, on-demand hooks
- [[claude-code-mods]] — Claude Code mods: in-process, stateful hook modules shipped as plugins (Oct 2026)

## Lineage sources

- [[claude-code-changelog-2026]] — Claude Code hooks released in v1.0.38 (2025-06-30); mods in v2.1.287 (2026-10-01)
- [[langchain-history-2024-2025]] — LangChain 1.0 middleware (2025-10-22): hooks in the agent loop
- [[claude-code-mods-docs-2026]] — mods: the third kind of Claude Code hook. In-process middleware that can rewrite or answer events, including the system prompt, compaction and subagent spawns
