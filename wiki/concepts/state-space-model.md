---
title: State-Space Model (SSM)
type: concept
created: 2026-06-19
updated: 2026-10-23
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

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[a-hippocampus-for-linear-attention-cui-2026]] — pairs a fixed-size recurrent state with a bounded *exact* KV cache for what the state overwrites; −16.1% ppl
- [[linear-attention]] — the adjacent family; KDA's gated delta rule is the recurrent-state design taken in an attention-shaped direction

## Lineage sources

- [[jamba-hybrid-transformer-mamba-lieber-2024]] — [[jamba]]: 1:7 attention:Mamba with MoE. 4 GB of KV at 256K against Mixtral's 32 GB. Pure Mamba fails at in-context format-following; the hybrid fixes it.
