---
title: "Hooks and Callbacks in Modern Agent Architecture"
type: query
created: 2026-05-17
updated: 2026-05-17
sources:
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
| **`chat.message` / `chat.params` / `chat.headers`** | Observe / mutate user message, LLM params, HTTP headers | [[opencode]] |
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
| **Stop** / `StopFailure` | After a model turn with zero tool calls (loop wants to terminate) | [[claude-code]] (unique — has anti-death-spiral logic) |
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
| **`subagent_spawning` / `subagent_delivery_target` / `subagent_spawned` / `subagent_ended`** | Sub-agent lifecycle | [[openclaw]] (4 phases — most granular) |

### Permission

| Hook | Fires at | Agents |
|---|---|---|
| **PermissionRequest** / **PermissionDenied** | Permission system events | [[claude-code]] |
| **`permission.ask`** | Pre-empt permission dialogs | [[opencode]] |

### Environment events ([[claude-code]] exclusive)

| Hook | Fires at |
|---|---|
| **ConfigChange** | Settings mutated |
| **CwdChanged** / **FileChanged** | File-system / workspace state |
| **WorktreeCreate** / **WorktreeRemove** | Worktree lifecycle |
| **InstructionsLoaded** | `CLAUDE.md` / skill frontmatter loaded |

## Two architectural axes

### 1. First-class harness hook vs. plugin-system hook

| Agent | Model | Hook count | Detail |
|---|---|---|---|
| [[claude-code]] | **First-class in harness** | ~21 named events | Can affect permissions; routes to user prompts, agent forks, or HTTP webhooks |
| [[openclaw]] | **Plugin-system only** | ~24 named events | Broadest surface but observational (synchronous handlers) |
| [[opencode]] | Plugin-system only | ~18 named events | Plugins registered via `opencode.json` |
| [[hermes-agent]] | Plugin-system | ~6 lifecycle hooks | Used internally for self-editing curator |
| [[pi-mono]] | **None in core** (deliberate) | 2 in core | Only `beforeToolCall` / `afterToolCall`; rest via extensions (the "no X" manifesto) |

### 2. What hooks can DO — observational → modifying → blocking → compositional

- **Observational** (default for plugins) — read events for logging, telemetry
- **Modifying** — rewrite tool args/results, mutate prompts, inject context
- **Blocking** — abort a tool call, refuse a user prompt, terminate the loop
- **Compositional** — [[hermes-agent|Hermes]]'s curator uses `post_tool_call` *inside a forked sub-agent* to modify its own skills during normal operation. Hooks become a self-modification mechanism.

## Cross-cutting framings

- The [[agent-three-layer-model]] classifies hooks as L3 implementations under L2 [[guardrailing|harness]], with cross-cutting edges to L2 control-flow.
- [[common-features-of-modern-agents]] does *not* count hooks among the 8 universal features — because [[pi-mono]] refuses them in core. Near-universal but not strictly baseline.
- The 5 hooks in the **Loop Hooks** graph view (`scripts/evolution-graph.html`) — SessionStart, UserPromptSubmit, PreToolUse, PostToolUse, Stop — are the **canonical subset**. Every agent except [[pi-mono]] has equivalents. The 25+ others are agent-specific extensions of the same architectural idea.

## Gaps and interesting omissions

- **`on-error` / exception hooks** — only loosely covered. [[openclaw]] has `PostToolUseFailure`-style. Most agents handle errors inline rather than via dedicated hooks.
- **`pre-respond` / `before-final-response`** — surprisingly absent across the wiki. The closest analogue is [[claude-code|Claude Code]]'s **Stop** hook (fires before exit when no follow-up tools).
- **Streaming-output hooks** — none of the 5 agents document a per-token or per-chunk output hook, even though all 5 stream output.

## Related queries

- [[common-features-of-modern-agents]] — the 8 universal features (hooks not baseline)
- [[from-transformer-to-modern-agents]] — historical track from Transformer → modern agents
- [[agent-three-layer-model]] — where hooks fit in the L1/L2/L3 architecture

## References

- [[claude-code-agent-doc-2026]] — primary source for Claude Code's 21 hook events
- [[opencode-agent-doc-2026]] — opencode's `opencode.json` plugin hook surface
- [[hermes-agent-doc-2026]] — Hermes's curator + plugin lifecycle
- [[openclaw-agent-doc-2026]] — broadest plugin hook taxonomy (24)
- [[pi-mono-agent-doc-2026]] — minimal-core stance; 2 in core
- [[cross-agent-comparison-doc-2026]] — cross-cutting comparison source
- [[cross-agent-comparison-2026]] — synthesized comparison page
