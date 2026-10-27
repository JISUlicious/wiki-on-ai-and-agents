---
title: Tri Dao
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - mamba-selective-state-spaces-gu-2023.md
  - mamba-2-transformers-are-ssms-dao-2024.md
tags:
  - person
status: draft
importance: medium
affiliations:
  - org: [[princeton-university]]
    role: CS faculty (Mamba, Mamba-2)
---

# Tri Dao

Author of **[[flash-attention|FlashAttention]]** ([[flash-attention-dao-2022]]) and co-author, with [[albert-gu]], of **[[mamba|Mamba]]** and **Mamba-2**. Both lines rest on the same lesson: make the algorithm fit the GPU memory hierarchy.
- Mamba's fused scan keeps the state in SRAM.
- Mamba-2's SSD turns the recurrence into tensor-core matmuls.

## References

- [[mamba-selective-state-spaces-gu-2023]] · [[mamba-2-transformers-are-ssms-dao-2024]] · [[flash-attention-dao-2022]]
