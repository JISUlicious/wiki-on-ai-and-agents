---
title: Jamba
type: entity
created: 2026-10-23
updated: 2026-10-23
sources:
  - jamba-hybrid-transformer-mamba-lieber-2024.md
tags:
  - model
status: draft
importance: medium
family: jamba
parameters: 52B total / 12B active
---

# Jamba

[[ai21-labs]]'s hybrid model (2024, Apache-2.0). It interleaves attention and Mamba ([[state-space-model|SSM]]) layers at **1 : 7** and adds [[mixture-of-experts|MoE]] (16 experts, top-2). It has 52B total / 12B active parameters and a 256K context. Paper: [[jamba-hybrid-transformer-mamba-lieber-2024]].

## Why it matters here

With only 4 of its 32 layers caching K/V, Jamba needs **4 GB of KV at 256K, against 32 GB for Mixtral** at similar active size. It is an early production demonstration that hybrid layer ratios, not parameter counts, set cache size. Later hybrids include [[kimi-linear-kimi-team-2025|Kimi Linear]] (3:1 KDA/MLA) and [[qwen3.8-flash-next|Qwen3.8-Flash-Next]] (3:1 Gated DeltaNet / sparse attention).

## Lineage

The authors' ablations found pure Mamba fails at in-context format-following, and attention layers restore it. That argument for keeping some attention layers recurs in later [[linear-attention]] hybrids.

## References

- [[jamba-hybrid-transformer-mamba-lieber-2024]]
