---
title: Recursive Agent Harnessing
type: concept
created: 2026-10-21
updated: 2026-10-21
sources:
  - harness2-recursive-agent-harnessing-xu-2026.md
status: draft
importance: medium
builds-on:
  - [[agentic-harness-engineering]]
  - [[self-improving-agent]]
---

# Recursive Agent Harnessing

**Recursive agent harnessing** is the term [[harness2-recursive-agent-harnessing-xu-2026|Harness²]] (Google Research, 2026) uses for [[self-improving-agent|recursive self-improvement]] in which the object being improved is the **agent harness**, not the model weights. It improves two things:
1. the **execution harness**: what the agent runs with on the current task;
2. the **improvement harness**: the process, plus a library of past harnesses and lessons, that decides *how* execution harnesses get improved.

The central question, in the paper's words: "Can recursive self-improvement improve both the current harness and how future harnesses are improved?"

## Two levels

| | Task-level recursion | Harnessing-level recursion |
|---|---|---|
| Improves | The execution harness for *this* task | The improvement harness (library + experience) |
| Signal | Behavioral contrasts between runs of the same task | Lessons supported by those contrasts |
| Persistence | The adopted harness is used for this task's deliverable | Carried across all later tasks |
| Next task | Starts again from the base harness | Starts with the updated library guiding edits |

## Propose–Probe–Compose

This is the core step that works **without ground-truth feedback**:
1. **Propose** scoped edits: one *invariant* (general agent behavior), one *domain* (field knowledge and procedure).
2. **Probe** them by re-running the *same task* under matched conditions.
3. **Compose** a successor harness from edits the behavioral evidence supports. Keeping the current harness is allowed.

The step can be widened (parallel: k candidates per scope, k compositions) or deepened (sequential: k chained steps).

## Why it differs from earlier harness optimization

- **Offline search** ([[meta-harness-lee-2026|Meta-Harness]], [[agentic-harness-engineering|AHE]]-style) tunes on training tasks with verifiers, then freezes the harness. The paper's argument: a static harness chosen in advance generalizes poorly to open-world tasks.
- **Per-task test-time revision** (Harness Scaling) adapts each task from scratch and learns nothing across tasks.
- **Carrying the last harness forward** risks noisy drift when the task distribution shifts.
- **Recursive harnessing** instead carries the *improvement experience* forward, and reuses components only selectively.

## Evidence and open questions

- **Evidence (Harness², self-reported):** gains in all 12 benchmark × configuration pairs.
  - Each part (library, scoping, composition) contributes 1.6 to 3.7 points.
  - A library built with more exploration helps later tasks even without probing, and transfers across benchmarks.
- **Open questions:**
  - **Proxy signal.** The signal is model judgment over behavioral contrasts, so shared biases between editor, selector and judge are a risk.
  - **Cost.** Compute grows with the recursion budget.
  - **Library upkeep.** The library has no consolidation yet.
  - **How much do executors benefit?** That depends on the executor ([[harness-updating-is-not-harness-benefit-lin-2026]]).

## Related

- [[agentic-harness-engineering]] · [[self-improving-agent]] · [[harness-continual-learning-kang-2026]] · [[rrsi-regularized-recursive-self-improvement-agent-harnesses-xia-2026]] · [[last-ai-built-by-humans-recursive-self-improvement-duan-2026]]

## References

- [[harness2-recursive-agent-harnessing-xu-2026]]
