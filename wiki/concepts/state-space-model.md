---
title: State-Space Model (SSM)
type: concept
created: 2026-06-19
updated: 2026-06-19
sources:
  - titans-behrouz-2025.md
status: draft
importance: medium
tags:
  - 2026
---

# State-Space Model (SSM)

State-space models (SSMs), such as Mamba, are sequence models that process tokens with a linear-time recurrence over a continuous hidden state, avoiding the quadratic cost of full attention. They scale efficiently to long contexts and are often hybridized with attention layers from the [[transformer-architecture]] and with [[mixture-of-experts]] routing. SSMs are a leading alternative for long-sequence and memory-centric architectures.

## References

- [[titans-behrouz-2025]]
