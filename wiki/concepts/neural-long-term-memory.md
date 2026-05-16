---
title: Neural Long-Term Memory
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

Neural long-term memory is a memory substrate implemented as a neural-network module whose representation lives in *parameters* (weights) rather than in an external store of tokens, embeddings, or notes. The module is trained to recall past context, and its weights are updated at test time via a surprise-driven loss so that information persists across the forward pass. This is orthogonal to external-store designs like [[memgpt]] and [[mem0]], which keep memory as serialized records in a database; neural long-term memory instead bakes memory directly into a small mutable subnetwork. Introduced by [[titans]] (Behrouz, Google 2025) as the long-horizon counterpart to short-term attention.

## Where it appears

- [[titans]] — Direct introduction; weights of the LTM module updated at test time.
- [[test-time-memory]] — The update mechanism that makes neural long-term memory work.
- [[memory-management]] — The "in-weights" pole of the memory substrate design space.

## References

- [[titans-behrouz-2025]] — Titans: Learning to Memorize at Test Time (Behrouz et al., Google 2025).
