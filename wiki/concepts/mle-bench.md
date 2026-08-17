---
title: MLE-Bench
type: concept
created: 2026-08-17
updated: 2026-08-17
sources:
  - frontis-ma1-ai4ai-recursive-self-improvement-yang-2026.md
  - rehearse-confidence-cliff-autoresearch-ji-2026.md
status: draft
importance: medium
---

# MLE-Bench

A benchmark of real Kaggle competitions used to evaluate agents at **machine-learning engineering**: given a dataset and a task description, the agent must build, train and submit a model, scored against the competition's actual human leaderboard.

Two properties make it the standard testbed for [[self-improving-agent|self-improving]] and AutoResearch systems. Scoring is **grounded in human performance** — the headline metrics are *Medal Average* (fraction of competitions reaching bronze/silver/gold thresholds) and *Human Rank* (percentile against real submissions) — so a score means something outside the benchmark. And each task is a full train-evaluate loop, which makes the cost of a wasted trial large enough that *deciding what to try* becomes the bottleneck.

**MLE-Bench Lite** is the commonly reported 22-task subset.

## Results in this wiki

[[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026|Frontis-MA1-35B]] reaches **71.21% Medal Average and 0.8126 Human Rank** on MLE-Bench Lite at 12 h/task on a single RTX 4090.

> [!warning] Read the variance before the ranking
> That 71.21% carries **±8.57 pp** across three epochs, so the paper's claim of exceeding GPT-5.5 + Codex by 3.03 pp sits well inside one standard deviation — 3.03 pp is two task outcomes out of 66. The clean model-level gain with the harness held fixed is **+21.22 pp**, not the +30 pp headline, which silently bundles a harness upgrade.

[[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] uses MLE-Bench-derived tasks for its judge benchmark, and supplies the reason self-improvement loops stall on this benchmark specifically: the gate deciding which modification to run degrades as the trajectory deepens, with helpful-modification rate falling **70% → 43%** by iteration six.

## Related

- [[self-improving-agent]] · [[darwin-godel-machine]] · [[adas]] — the systems evaluated on it.
- [[agent-benchmark]] · [[swe-bench]] · [[code-generation]]
- [[reward-hacking]] — MLE-Bench agents have been caught submitting shuffled sample submissions.

## References

- [[frontis-ma1-ai4ai-recursive-self-improvement-yang-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]]
