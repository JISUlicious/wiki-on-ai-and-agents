---
title: "Goal loops, proactive triggers and extension surfaces: repository and docs check"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - claude-code-goal-docs-2026-11-02.md
year: 2026
url: https://code.claude.com/docs/en/goal
status: complete
importance: medium
tags:
  - 2026
---

# Goal loops, proactive triggers and extension surfaces: repository and docs check

Read on 2026-11-02 at [earendil-works/pi](https://github.com/earendil-works/pi) `c9f4c7b9fc`, [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) `5aa8df742c` and [openclaw/openclaw](https://github.com/openclaw/openclaw) `28a93a00ac`, plus the live Claude Code docs and Codex release notes. Docs and source were read; nothing was run.

## Goal commands

| Agent | First seen | Who decides the goal is done | Location |
|---|---|---|---|
| Codex CLI | 0.128.0, 2026-04-30 | Not checked. The notes list "model tools" and "runtime continuation". | [release notes](https://github.com/openai/codex/releases/tag/rust-v0.128.0) |
| [[hermes-agent]] | 2026-05-01 | Shell quality gates run first; an auxiliary judge model then decides. Judge failures count as `continue`, with the turn budget as backstop. | commit `265bd59c1d`; `website/docs/user-guide/features/goals.md`; `hermes_cli/goals.py` |
| [[claude-code]] | 2.1.139, 2026-05-11 | The small fast model, as evaluator. It calls no tools and judges only what the conversation shows. | [goal docs](https://code.claude.com/docs/en/goal); [[claude-code-changelog-2026]] |
| [[openclaw]] | 2026-05-29 | The working model, through `update_goal`, or the operator through `/goal complete`. | commit `a509c48f0e`; `docs/tools/goal.md`; `src/agents/tools/goal-tools.ts` |
| [[pi-mono]] | None found | Not applicable | Keyword search of `packages/coding-agent` `src`, `docs` and `examples` |

- Hermes's doc says its goal is "directly inspired by" Codex CLI 0.128.0's `/goal`, with an independent implementation. It credits its quality gates to Prime-Agent's bounded autonomous mode.
- Claude Code's doc calls `/goal` a wrapper around a session-scoped prompt-based Stop hook.
- OpenClaw's doc says a goal is not a reminder, cron job or standing order. It names no inspiration. "Judge", "evaluator", "verifier" and "verdict" do not appear in `goal.md` or in 36 goal-named TypeScript files. Other files may still add a check, so the code path is not fully traced.
- Pi's `agent_before_settle` handler can return `continue: true` to request one continuation for that boundary (`docs/extensions.md`). No Pi example uses it to pursue a goal.

## Proactive triggers

| Agent | Feature | Date | Note |
|---|---|---|---|
| Hermes | cron | 2026-02-02 | Commit `a3ba41fce2`; the title compares it to OpenAI's Pulse |
| Hermes | `/heartbeat` | authored 2026-08-05, committed 2026-08-06 | `6518aa184e`; the doc says inspired by Prime-Agent's `/heartbeat` |
| Hermes | `/loop` | authored 2026-07-27, committed 2026-08-14 | `f79440e0f4`; the title says Claude Code parity |
| OpenClaw | Standing orders | doc first committed 2026-03-20 | `99e53612cb`; this dates the doc, not necessarily the feature |
| OpenClaw | Standing intents | doc first committed 2026-07-28 | `28630a9a65`; event-conditioned, not clock-based |
| OpenClaw | Proactive check job | undated here | `docs/gateway/heartbeat.md`; every 30 minutes by default after explicit setup |

Hermes's `loops.md` mentions a Claude Code `/proactive` alias. The Claude Code scheduled-tasks and commands pages do not mention one, so that detail is unconfirmed. Claude Code's own `/loop` dates from 2026-03-06.

## Extension surfaces

- **Pi:** the README calls it a "minimal, extensible agent harness" and says it skips sub-agents and plan mode. Commit `c6fc084534` (2026-01-05) merged hooks and custom tools into one extension system.
  - `packages/agent/src/types.ts` has two tool hooks, `beforeToolCall` and `afterToolCall`.
  - `docs/extensions.md` documents 40 distinct `pi.on` event names in the shipped coding agent.
  - `docs/security.md` says watching the transcript, project trust and reviewing changes do not create a security boundary. `permission-gate` and `protected-paths` are examples, not defaults.
- **Hermes:** `VALID_HOOKS` in `hermes_cli/plugins.py` has 46 names. `pre_verify` can keep the agent going and accepts a Claude-Code-style Stop decision, bounded by `agent.max_verify_nudges`.
- **OpenClaw:** `VISION.md` says core stays lean and optional capabilities should usually ship as plugins.

## Concepts & entities

- [[harness-extension-and-goal-loop-axes]] · [[ralph-loop]] · [[hooks-in-modern-agents]]
- [[pi-mono]] · [[hermes-agent]] · [[openclaw]] · [[claude-code]]
- [[prime-agent-self-improving-rlm-harness-karten-2026]]

## References

- [Claude Code `/goal` docs](https://code.claude.com/docs/en/goal)
- [Codex 0.128.0 release](https://github.com/openai/codex/releases/tag/rust-v0.128.0)
