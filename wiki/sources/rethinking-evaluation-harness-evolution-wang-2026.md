---
title: "Rethinking the Evaluation of Harness Evolution for Agents"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - rethinking-evaluation-harness-evolution-wang-2026.md
arxiv_id: "2607.12227"
year: 2026
authors:
  - Yike Wang
  - Huaisheng Zhu
  - Teng Xiao
  - Hannaneh Hajishirzi
  - Pradeep Dasigi
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Rethinking the Evaluation of Harness Evolution for Agents

**Yike Wang\*, Huaisheng Zhu\*, Zhengyu Hu, Yige Yuan, Zhengyu Chen, Shakti Senthil, Hannaneh Hajishirzi, Yulia Tsvetkov, Pradeep Dasigi, Teng Xiao\*** (Allen Institute for AI; University of Washington; \*equal contribution) — [arXiv:2607.12227](https://arxiv.org/abs/2607.12227), v1, 14 Jul 2026 (cs.AI). Preprint; no venue. Code: `github.com/rethinking-harness-evolution`.

## Summary

This is a negative-result / methodology paper aimed squarely at the automatic harness evolution literature. Its central argument is structural: **harness evolution is itself an iterative search procedure** — it repeatedly samples rollouts, reads verifier feedback, and revises candidates. Evaluated the way the field currently evaluates it (search on benchmark tasks with unit-test feedback, then report final performance on that same public benchmark), any reported gain conflates two very different things: a genuinely better harness design, versus the ordinary benefit of spending more search compute on the evaluation tasks. The paper therefore insists harness evolution be compared against plain **test-time scaling baselines under matched feedback and inference budgets**, and be evaluated on **held-out tasks**.

The experimental setup instantiates harness evolution with **AHE** ([[agentic-harness-engineering]], Lin et al. arXiv:2604.25850) with its explore agent disabled, so improvements come from evolving the harness on feedback rather than retrieving benchmark-specific harnesses externally. All methods start from AHE's own initial harness `h₁`, use compute budget K = 5, m = 1 rollout per task per harness, 128k generation budget with high reasoning effort, averaged over two independent runs. Benchmark is [[terminal-bench]] 2.1 (a verified revision of 2.0 that repairs 28 of 89 tasks). Models: Claude Opus 4.6, GPT-5.4, GPT-5.4 mini. The paper also introduces **harness scaling** as a fourth arm — the harness-level analogue of test-time scaling, adapting the harness for a *single* evaluation instance rather than producing a reusable design.

The verdict across three settings: **harness evolution does not consistently beat simple test-time scaling, and its gains do not transfer to held-out tasks.** Without unit tests it lands *below* the direct-sampling baseline on average. With unit tests it improves, but never beats parallel sampling or sequential refinement on either pass@1 or pass@5. With disjoint search/eval splits it gains 0.6 points on average. The trajectory analysis explains why: the meta-agent's edits are rational and well-motivated (prompt rules → middleware enforcement → tool guidance), but "most edits memorize fixes rather than distilling strategies" — they save time on tasks the agent could already solve, rarely convert failures into successes, and accumulate prompt text that causes context bloat.

**This directly challenges self-evolving-harness claims** ([[self-improving-agent]], [[darwin-godel-machine]], and specifically the AHE results already recorded in this wiki). Important caveat the paper itself raises (§5.2): the negative result may be benchmark-specific. Terminal-Bench scores are already high, remaining failures may be model-limited rather than harness-limited, and a shell tool plus a basic prompt may already suffice for most solvable tasks — meaning Terminal-Bench may simply not be very harness-sensitive. The authors ask for future benchmarks that are (1) hard enough to leave headroom and (2) genuinely harness-dependent.

## Key points

**Setting 1 — no unit test feedback (Table 1, pass@1).** Averages are across all three models.

| Method | Claude Opus 4.6 | GPT-5.4 | GPT-5.4 mini | Average |
|---|---|---|---|---|
| Direct sampling, initial harness | 69.9 | 75.3 | 59.4 | **68.2** |
| Parallel Sampling | 74.7 | 79.2 | 62.9 | **72.3** |
| Sequential Refinement | 73.0 | 73.0 | 61.8 | **69.3** |
| Harness Evolution | 71.4 | 69.7 | 61.3 | **67.4** |
| Harness Scaling | 76.0 | 78.1 | 61.2 | **71.8** |

- Harness Evolution (67.4) falls **below** the static initial harness (68.2). The degradation is sharpest on the strongest model: GPT-5.4 drops 75.3 → 69.7, i.e. "iterative harness revision can actively hurt a strong model when the revision process is guided only by the agent's own judgment."
- Parallel Sampling is the most consistent method, gaining on all three models. Sequential Refinement adds only 1.1 points and slightly degrades GPT-5.4.
- Reading: without an external correctness signal, self-generated feedback is too noisy to ground harness revision, and sequential revision compounds early mistakes.

**Setting 2 — unit test feedback available (Table 2).** Two models only; unit tests serve as both refinement signal and oracle selector.

| Method | Opus 4.6 p@1 / p@5 | GPT-5.4 p@1 / p@5 | Avg p@1 / p@5 |
|---|---|---|---|
| Direct sampling, initial harness | 69.9 / — | 75.9 / — | 72.9 / — |
| Parallel Sampling | 84.8 / 84.8 | 87.1 / 87.1 | **86.0** / 86.0 |
| Sequential Refinement | 83.1 / 90.4 | 85.4 / 93.3 | 84.3 / **91.8** |
| Harness Evolution | 73.0 / 83.2 | 78.6 / 89.3 | 75.8 / 86.2 |
| Harness Scaling | 83.1 / 89.9 | 82.0 / 88.8 | 82.6 / 89.3 |

- Everything beats direct sampling once unit-test feedback exists — the feedback helps regardless of mechanism. But Harness Evolution barely improves pass@1 (72.9 → 75.8) while Parallel Sampling reaches 86.0.
- The diagnostic argument: if harness revision genuinely produced better harnesses, the gain would show in pass@1. Instead it only materializes under multi-trajectory selection, so "their gains largely stem from making multiple attempts." Conclusion: refining solutions is a more productive use of extra compute than revising the harness.

**Setting 3 — disjoint search and evaluation tasks (Table 3, pass@1 on test).** Terminal-Bench 2.1 split into 45 train / 10 validation / 34 held-out test tasks; best harness selected on validation.

| Method | Claude Opus 4.6 | GPT-5.4 | Average |
|---|---|---|---|
| Direct sampling, initial harness | 63.3 | 72.1 | 67.7 |
| Harness Evolution | 64.5 (**+1.2**) | 72.1 (**+0.0**) | 68.3 (**+0.6**) |

- "Severe overfitting to the training tasks": revisions "encode task-specific shortcuts rather than genuinely better harness design principles."

**Qualitative trajectory analysis (§5.1).** Harness Evolution starts at the prompt layer (behavioral rules about shipping deliverables early, copying fragile state before mutating, rechecking constraints), escalates to middleware when advisory text plateaus (turn-budget trackers, truncation of oversized tool outputs, finalization gates), then edits tool guidance. Harness Scaling instead memorizes per-task facts — known bugs, file paths, command sequences, batched install/verify plans, longer shell timeouts. Both patterns are rational; both mostly speed up already-solvable tasks. A "stable core of hard failures" stemming from deep domain reasoning or constraints outside harness control is untouched.

## Concepts & entities

- [[agentic-harness-engineering]] — the method under test; harness evolution is instantiated as AHE with its explore agent disabled, using AHE's own seed harness. The critique lands directly on this paper's protocol.
- [[self-improving-agent]] — the strongest challenge in the paper: self-evolution gains here are largely attributable to extra search, not to a better reusable artifact.
- [[darwin-godel-machine]] — the same budget-matching objection applies to any benchmark-feedback-driven self-evolution loop that reports on its search benchmark.
- [[terminal-bench]] — evaluation benchmark (v2.1, 89 tasks, 28 repaired vs 2.0); also the paper's chief confound, since §5.2 argues it may be insufficiently harness-sensitive.
- [[agent-evaluation]] — the paper's real contribution is an evaluation protocol: matched feedback and inference budgets, plus disjoint search/eval splits.
- [[harness-vs-model-attribution]] — sharpens the attribution question by adding a third candidate cause: neither model nor harness design, but search budget.
- [[verifier]] — unit-test feedback is the pivot; harness revision only becomes productive with a reliable external correctness signal, and even then loses to simpler uses of the same signal.
- [[code-as-harness]] — the object being searched over is harness code (prompts, tools, middleware, memory, control logic).
- [[fundamental-components-of-harness]] — the meta-agent's edits map onto the prompt / middleware / tool / long-term-memory component layers.
- [[learned-orchestration]] — harness scaling (instance-guided harness adaptation) is introduced here as a new point in this design space, and it outperforms dataset-guided harness evolution.

## References

- [arXiv:2607.12227](https://arxiv.org/abs/2607.12227)
- Method under test: Lin et al., "Agentic Harness Engineering" (arXiv:2604.25850) — see [[agentic-harness-engineering-lin-2026]].
- Other harness-evolution work critiqued: Meta-Harness (Lee et al., arXiv:2603.28052); AEVO (Zhang et al., 2026).
- Test-time scaling baselines drawn from: Snell et al. 2024; self-consistency (Wang et al. 2022); Large Language Monkeys (Brown et al. 2024); Self-Refine (Madaan et al. 2023); AlphaEvolve (Novikov et al. 2025).
- Benchmark: Terminal-Bench (Merrill et al., arXiv:2601.11868), v2.1.
