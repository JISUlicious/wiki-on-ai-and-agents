---
title: Quality-Diversity
type: concept
created: 2026-09-29
updated: 2026-09-30
sources:
  - darwin-godel-machine-zhang-2025.md
status: draft
importance: low
---

# Quality-Diversity

**Quality-diversity (QD)** is a family of search algorithms from evolutionary computation that returns not one best solution but an *archive* of many solutions that are each good and different from one another. The canonical instance, MAP-Elites (Mouret & [[jeff-clune|Clune]], 2015), builds a map of high-performing solutions across a space defined by dimensions of variation that the user chooses. Because it explores more of the space, it also tends to find a better overall solution than single-objective search. The underlying intuition is *stepping stones*: the path to the best solution often runs through candidates that a pure hill-climber would discard.

## In this wiki

QD enters the wiki as the algorithmic ancestor of the archive in the [[darwin-godel-machine]] ([[darwin-godel-machine-zhang-2025]]):

- **The archive is QD-style.** The DGM keeps every viable agent it has generated and picks parents with probability roughly proportional to score and inversely proportional to number of children, so every node keeps a non-zero chance of selection.
- **Stepping stones are measured, not assumed.** Iterations 4 and 56 of the SWE-bench run produced agents *worse* than their parents, yet both lie on the lineage of the best agent found.
- **The ablation isolates the archive.** DGM-without-archive (hill-climbing on the latest agent) plateaus, which is the QD argument in miniature.

The same concern with keeping diversity alive shows up on the training side. [[vector-policy-optimization-bahlous-boldi-2026|Vector Policy Optimization]] argues that scalar-reward RL collapses a policy to low-entropy outputs and hurts evolutionary and best-of-n search, and trains for diversity directly. That paper does not use the QD label.

## Related

- [[darwin-godel-machine]] · [[self-improving-agent]] · [[meta-learning]] · [[adas]]
- [[voyager]] — its [[skill-library]] is the wiki's other archive-as-stepping-stones example
- [[godel-machine]] · [[jeff-clune]] · [[sakana-ai]]

## References

- [[darwin-godel-machine-zhang-2025]]
- MAP-Elites: Mouret & Clune, "Illuminating search spaces by mapping elites" — [arXiv:1504.04909](https://arxiv.org/abs/1504.04909)

## Related 2026 sources

- [[agora-git-shared-memory-collective-autoresearch-zhang-2026]] — Agora (NVIDIA): a Git DAG as shared memory for 13 autoresearch agents; 98% of the gain came in the first 6 hours
