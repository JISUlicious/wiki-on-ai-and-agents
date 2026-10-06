---
title: Ralph Loop
type: concept
created: 2026-09-29
updated: 2026-10-06
sources:
  - oh-my-claude-ultrawork-2026.md
status: draft
importance: low
---

# Ralph Loop

A **Ralph loop** (after Ralph Wiggum) is the practice of running a coding agent inside a plain shell loop, feeding it the same prompt file every iteration until the work is done. Geoffrey Huntley's July 2025 post "Ralph Wiggum as a 'software engineer'" popularized it, and its purest form is a single line: `while :; do cat PROMPT.md | claude-code ; done`. No state carries over between iterations inside the model. Progress persists in the repository and in plan and notes files (the post uses `fix_plan.md` and `AGENT.md`) that the agent reads and updates on each pass. Huntley's own emphasis is on "loop back": building ways for the agent's output to be fed back to it for evaluation, such as tests.

## In this wiki

- **The ancestor of ultrawork.** The [[ultrawork]] page, built on the [[oh-my-claude-ultrawork-2026|oh-my-claude README]], traces that community trend back to "Ralph loops" and parallel-agent patterns. It notes that other tools in the space ship modes named "Ralph" alongside Team, Autopilot and Ultrawork.
- **The same pressure as the official features.** A Ralph loop, ultrawork's completion gates ("stops only when all todos complete") and Anthropic's official [[dynamic-workflows]] and [[ultracode]] all push against the same failure: a single agent in one context window stalls on large, multi-part work.
- **A harness pattern in its simplest form.** Keeping state in the filesystem (plans, notes, git) rather than in the context window is the "PLAN.md as control object" pattern that [[code-as-agent-harness-ning-2026]] describes under planning. See [[code-as-harness]] and [[plan-execute-verify-loop]].

## Related

- [[ultrawork]] · [[oh-my-claude]] · [[claude-code]] · [[dynamic-workflows]] · [[ultracode]]
- [[multi-agent]] · [[code-as-harness]] · [[agentic-harness-engineering]] · [[plan-execute-verify-loop]]

## References

- [[oh-my-claude-ultrawork-2026]] · [[ultrawork]]
- Geoffrey Huntley, "Ralph Wiggum as a 'software engineer'" (2025-07-14) — <https://ghuntley.com/ralph/>

## Lineage sources (added 2026-10-06)

- [[claude-code-changelog-2026]] — native /loop arrived 2026-03-06 and /goal 2026-05-11
