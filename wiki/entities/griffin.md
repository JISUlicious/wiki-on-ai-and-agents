---
title: Griffin (and Hawk)
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - griffin-gated-linear-recurrences-de-2024.md
tags:
  - model
status: draft
importance: medium
family: griffin
---

# Griffin (and Hawk)

[[deepmind|Google DeepMind]]'s 2024 recurrent LMs built on the **RG-LRU**, a real-gated diagonal linear recurrence.
- **Hawk** is pure recurrence.
- **Griffin** mixes recurrent blocks with **local [[multi-query-attention|MQA]] (1,024-token window)** at 2 : 1.
- **Data efficiency:** Griffin-7B/14B roughly match Llama-2-7B/13B on **300B vs 2T tokens**.

Paper: [[griffin-gated-linear-recurrences-de-2024]]. RecurrentGemma is the product released from this line (not covered by the paper).

## Lineage

It takes the diagonal recurrence from S4/LRU but drops HiPPO, continuous-time discretization and complex numbers. Its gate is GRU-like rather than Mamba's selection. Griffin is an early **recurrence + [[sliding-window-attention|local attention]]** hybrid, the pattern later used by [[gated-delta-networks-yang-2024|Gated DeltaNet-H1]] and Samba.

## References

- [[griffin-gated-linear-recurrences-de-2024]] · [[state-space-model]]
