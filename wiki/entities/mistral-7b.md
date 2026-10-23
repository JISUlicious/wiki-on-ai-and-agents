---
title: Mistral 7B
type: entity
created: 2026-10-23
updated: 2026-10-23
sources:
  - mistral-7b-jiang-2023.md
tags:
  - model
status: draft
importance: medium
family: mistral
parameters: 7.2B
release_date: 2023-10-10
---

# Mistral 7B

[[mistral-ai]]'s first model (October 2023, Apache-2.0). It is a dense 7B model that beat Llama 2 13B on every benchmark the paper reports. Paper: [[mistral-7b-jiang-2023]].

## Architecture

- 32 layers, dim 4096.
- **[[grouped-query-attention|GQA]]:** 32 query heads / 8 KV heads, giving 128 KiB of KV per token at fp16.
- **[[sliding-window-attention|Sliding-window attention]]:** W = 4096, with a rolling-buffer cache.
- Context 8192; theoretical span about 131K.

## Lineage

Its successor Mixtral is an MoE with 46.7B total / 12.9B active parameters but the **same 32 GB KV cache at 256K**. That shows the cache follows the attention configuration, not the experts (see [[jamba-hybrid-transformer-mamba-lieber-2024]] and [[kv-cache-size-vs-active-parameters]]).

## References

- [[mistral-7b-jiang-2023]]
