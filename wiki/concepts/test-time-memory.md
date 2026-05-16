---
title: Test-Time Memory
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - titans-behrouz-2025.md
status: draft
importance: medium
tags:
  - 2025
---

Test-time memory refers to memory whose parameters are updated at *inference* time rather than only during training. In a conventional transformer, the model's weights are frozen at deployment and any per-conversation state lives in the KV-cache or in external retrieval stores. Test-time memory instead exposes a small memory module whose weights are mutated during the forward pass — typically driven by a *surprise* signal (a gradient on the prediction loss for the current input). This blurs the traditional train/inference boundary and gives the model a continual-learning substrate. Introduced by [[titans]] (Behrouz, Google 2025) as the "neural long-term memory" branch alongside short-term attention.

## Where it appears

- [[titans]] — Introduces a memory module whose weights are updated at test time by a surprise-driven loss.
- [[neural-long-term-memory]] — The substrate that test-time memory updates operate on.
- [[online-learning]] — Conceptually adjacent: both update parameters from a live data stream.
- [[memory-management]] — One pole (in-weights, mutable at test time) of the design space.

## References

- [[titans-behrouz-2025]] — Titans: Learning to Memorize at Test Time (Behrouz et al., Google 2025).
