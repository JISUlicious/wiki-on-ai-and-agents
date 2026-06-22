---
title: "PreAct: Computer-Using Agents that Get Faster on Repeated Tasks"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - preact-computer-using-agents-li-2026.md
arxiv_id: "2606.17929"
authors:
  - Bojie Li
first_author: Bojie Li
year: 2026
venue: arXiv preprint
tags:
  - 2026
status: complete
importance: medium
---

# PreAct: Computer-Using Agents that Get Faster on Repeated Tasks

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-14–06-21).

## Summary

PreAct lets a [[computer-use-agent]] get faster on tasks it has done before. A standard computer-using agent drives software through the screen — clicking and typing — but re-reads the screen and re-reasons every step each time, paying full inference cost on repeats. The first time PreAct succeeds at a task, it **compiles the run into a small state-machine program**: states check the screen, transitions act. On later runs it replays the program directly with **no per-step LLM calls, running 8.5–13× faster**. Replay is safe-by-construction: at each step PreAct verifies the screen matches what the program expects before acting, and hands control back to the agent the moment something is off.

PreAct applies the same discipline at store time: a compiled program is kept only if, re-run from a clean state, an independent evaluator confirms it actually solved the task — catching programs that replay to their last step yet leave the task undone. Across a mobile, a desktop, and a web benchmark, this store-time check separates repeated runs that improve from ones that degrade as faulty programs accumulate (worth 1.75–2.6 tasks per benchmark), and a fallback that explores afresh when no program fits brings PreAct level with a strong record-and-replay baseline.

## Key points

- **Compile-once, replay-many:** the first successful run is compiled into a compact state-machine program (screen-checking states, acting transitions).
- **8.5–13× faster replay** with **zero per-step language-model calls** on repeated tasks.
- **Safe-by-construction replay:** each step validates the screen against the program's expectation and hands back to the agent on any mismatch.
- **Store-time verification:** an independent evaluator re-runs a fresh program from a clean state before admitting it, catching "replayed but task undone" failures — worth **1.75–2.6 tasks/benchmark**.
- **Evaluated on mobile, desktop, and web benchmarks** ([[agent-benchmark]]); a fresh-exploration fallback matches a strong record-and-replay baseline when no program fits.
- **Negative results reported:** prompt wording, runtime guardrails, and LLM-vs-embedding program retrieval did not matter much.

## Concepts & entities

- [[computer-use-agent]] — agents that operate software via screen reading and clicks/keystrokes.
- [[agent-benchmark]] — evaluated across mobile (AndroidWorld-style), desktop, and web suites.

## References

- [arXiv:2606.17929](https://arxiv.org/abs/2606.17929)
- Source PDF: `preact-computer-using-agents-li-2026.pdf`
