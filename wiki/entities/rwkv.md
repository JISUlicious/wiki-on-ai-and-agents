---
title: RWKV
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - rwkv-7-goose-peng-2025.md
tags:
  - model
status: draft
importance: medium
family: rwkv
---

# RWKV

An open-source recurrent LM family ("Receptance Weighted Key Value"), now a Linux Foundation AI & Data project led by Bo Peng, with Apache-2.0 weights. It has constant memory and constant time per token.

**RWKV-7 "Goose"** (2025) uses a **generalized delta rule** with a vector-valued decay and in-context learning rate, and separate removal and replacement keys.
- **Results:** the 2.9B model matches Qwen2.5-3B's English average on about ⅓ of the tokens, and is state of the art on multilingual benchmarks at 3B.
- **Expressivity claim:** it recognizes all regular languages (NC¹-complete tasks).

See [[rwkv-7-goose-peng-2025]].

## Lineage

RWKV-4/5/6 (linear-attention-style RNNs with time decay) → **RWKV-7**, which adopts the delta rule alongside [[gated-deltanet|Gated DeltaNet]] (concurrent work). The World models were *continued* from RWKV-5/6 checkpoints rather than trained from scratch.

## References

- [[rwkv-7-goose-peng-2025]] · [[linear-attention]] · [[state-space-model]]
