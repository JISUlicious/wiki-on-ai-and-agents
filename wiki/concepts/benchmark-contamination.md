---
title: Benchmark Contamination
type: concept
created: 2026-09-29
updated: 2026-09-29
sources:
  - swe-bench-live-zhang-2025.md
  - swe-bench-jimenez-2023.md
  - retro-borgeaud-2021.md
  - swe-refactor-bench-hong-2026.md
status: draft
importance: medium
---

# Benchmark Contamination

**Benchmark contamination** is when test items, or near-copies of them, end up in a model's training data. The model's score then measures memorization or implicit overfitting instead of the ability the benchmark is meant to test. For web-scale pretraining this is a standing risk, because static benchmarks are public and the training corpora cannot be fully inspected. The usual remedies are to filter overlaps from the training data, to evaluate only on items created after the training cutoff, and to keep refreshing the benchmark.

## In this wiki

- **Evidence from coding agents.** [[swe-bench-live-zhang-2025|SWE-bench-Live]] is the wiki's main case. The same OpenHands + Claude 3.7 Sonnet setup resolves **43.20%** on SWE-bench Verified but **19.25%** on fresh issues. Instances from repos that overlap with the original SWE-bench score higher (22.96%) than instances from new repos (18.89%). Its answer is the monthly refresh (see [[swe-bench-live]]).
- **Designed-in refresh.** [[swe-bench-jimenez-2023|SWE-bench]] already noted that its automatic pipeline could collect post-cutoff issues. [[livecodebench]] applies the same idea to contest problems.
- **Overlap filtering.** [[retro-borgeaud-2021|RETRO]] retrieves from its training set at evaluation time, so it introduces a 13-gram Jaccard filter and separates gains from copying neighbours from gains from general knowledge.
- **Design choices against leakage.** [[swe-refactor-bench-hong-2026|SWE-Refactor-Bench]] squashes repository history, citing SWE-bench-Live's evidence as precedent.
- **In self-authored leaderboards.** [[jevbench]] publishes half its hard tier, which can then be trained on. Several top entries are fine-tunes from the week the board appeared.

## Related

- [[benchmark]] — hub of every benchmark page in this wiki
- [[agent-benchmark]] · [[agent-evaluation]] · [[swe-bench-live]] · [[livecodebench]] · [[reward-hacking]]

## References

- [[swe-bench-live-zhang-2025]] · [[swe-bench-jimenez-2023]] · [[retro-borgeaud-2021]] · [[swe-refactor-bench-hong-2026]]
