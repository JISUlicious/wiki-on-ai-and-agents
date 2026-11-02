---
title: "Does Harness Engineering Split into Extension and Goal-Loop Axes?"
type: query
created: 2026-11-02
updated: 2026-11-02
sources:
  - harness-engineering-for-coding-agent-users-bockeler-2026.html
  - claude-code-goal-docs-2026-11-02.md
status: complete
importance: medium
---

# Does Harness Engineering Split into Extension and Goal-Loop Axes?

> [!note] Hypothesis
> "Extension guardrails + goal loop" is the user's hypothesis, not a claim from the literature. No source found states it as two axes. The search covered this wiki and the documents in [[agent-goal-loops-extension-docs-2026]], not the wider literature.

**Short answer: useful as a split by purpose, but not as two independent mechanisms.** One asks what the harness allows and how it can be changed; the other asks when the agent may stop. The evidence supports that distinction, but the two share machinery, and proactive triggers fit neither.

## Evidence for the split

- **Products ship them separately.** Hermes and OpenClaw have both a plugin surface and a dedicated goal command.
- **Pi has one and not the other, first-party.** It has a large extension surface and no built-in goal command.
- **Goal commands appeared later and close together.** Codex CLI on 2026-04-30, Hermes on 2026-05-01, Claude Code on 2026-05-11 and OpenClaw on 2026-05-29. Hermes explicitly credits Codex; the other dates show order, not influence.

## Evidence against independence

- **Claude Code builds its goal on a hook.** Its docs call `/goal` a wrapper around a prompt-based Stop hook, so here the goal loop is an extension-point use.
- **Hermes has both routes.** Its `pre_verify` plugin hook can also keep the agent going, separately from `/goal`.
- **Pi has a real third-party case.** `@tian.zuo/pi-goal` says it follows Codex's goal architecture using Pi's extension primitives. It queues continuations from `agent_settled`, not the `agent_before_settle` event this page first pointed to.
- **Four implementations sit on such a surface, of different kinds.** Claude Code uses a Stop hook, Codex an in-tree extension crate, OpenCode an in-tree plugin and pi-goal a third-party extension. Hermes and OpenClaw implement goals in core, and Codex's crate dates from after its `/goal` shipped, so this is not a universal or original design.
- **One existing model covers both.** In [[guides-and-sensors|Böckeler's terms]], a shell gate is a computational sensor and a goal judge is an inferential one. That mapping is this wiki's. Böckeler does not discuss goal commands, and her "steering loop" is a human improving the harness.

## What the split hides

- **Who decides completion varies, and this may matter more than the axes.**

| Agent | Decides completion | Independent of the working model? |
|---|---|---|
| Hermes | Shell gates, then an auxiliary judge | Yes |
| OpenCode | A reviewer step on the same session, no tools | Separate step; whether it uses another model was not established |
| Claude Code | Small fast model with no tools | Yes, but it sees only the conversation |
| OpenClaw | Working model or operator | No separate judge found; code path not fully traced |
| Codex CLI | Working model after a completion audit | No separate judge found in `ext/goal` |
| pi-goal (third party) | Working model or user | No judge found in its main files |

- **That suggests two completion schools.** Codex, OpenClaw and pi-goal rely on the working model reporting completion, with audit instructions in at least Codex and OpenClaw. Hermes, Claude Code and OpenCode add a separate check. This grouping is a synthesis, and "separate" does not always mean a different model.
- **Failure behaviour differs too.** Hermes's judge fails open, so a broken judge means more work until the turn budget stops it.
- **Proactive triggers look like a third thing.** Cron, heartbeats, `/loop` and standing intents decide when work *starts*, not when it may stop. Hermes's docs contrast timer-driven `/loop` with judge-driven `/goal`, and OpenClaw's say a goal is not a cron job or standing order.
- **"Guardrail" overstates extensions.** Pi's security doc locates safety in limiting access, such as isolation, and its permission extensions are only examples. An extension point is not itself a guardrail.

## Grading

| Claim | Grade |
|---|---|
| The commands, hooks and dates above | Confirmed in docs or commits at the pinned revisions |
| OpenClaw has no separate judge | Not found in goal-named files; not proven absent |
| Claude Code has a `/proactive` alias | Confirmed by the changelog (2.1.105), though the current docs pages checked omit it |
| Pi hosts a goal loop as an extension, via pi-goal | Package and source read; not run |
| Two completion schools: self-report with audit vs separate reviewer | This wiki's synthesis, from the requesting session; no source states it |
| Harness engineering has two axes | Hypothesis, unsupported by any source found |

## References

- [[agent-goal-loops-extension-docs-2026]] · [[goal-command-lineage-repos-2026]]
- [[harness-engineering-for-coding-agent-users-bockeler-2026]]
- [[claude-code-changelog-2026]] · [[ralph-loop]] · [[hooks-in-modern-agents]] · [[plugins-hooks-harness-influence-evidence]]
