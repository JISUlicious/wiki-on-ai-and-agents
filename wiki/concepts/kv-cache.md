---
title: KV-Cache
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - em-llm-fountas-2024.md
status: draft
importance: medium
tags:
  - 2024
---

# KV-Cache

The **key/value cache** in [[transformer-architecture|Transformer]] attention: when generating token *t+1*, the model needs the keys and values of all previous tokens *1..t*. Rather than recompute them every step, those projections are cached. The KV-cache **is** the model's working memory during autoregressive generation — its size grows linearly with context length and dominates the memory footprint of long-context inference.

[[em-llm]] ([[em-llm-fountas-2024|Fountas et al. 2024]]) repurposes the KV-cache as the *physical substrate* for episodic memory: coherent spans of KV pairs are bounded into "events" via Bayesian-surprise segmentation, then retrieved by similarity plus temporal contiguity. This is a fundamentally different substrate from **external-store** memory systems like [[memgpt]] or [[mem0]], which keep memory as text/JSON outside the model and reload it via prompts.

## Where it appears

- [[em-llm]] — segments the KV-cache itself into events and retrieves spans of cached KV pairs.
- [[transformer-architecture]] — the original locus of the cache.
- Contrast with external-store memory: [[memgpt]], [[mem0]] keep memory as serialized text reloaded into context.

## References

- [[em-llm-fountas-2024]]

## KV-cache quantization

The cache is a memory consumer independent of parameter count, scaling with context length — for Llama-2-7B at 32k context it **exceeds the FP16 weights**. Quantizing it (`q8_0`/`q4_0` in [[llama-cpp]], or KVQuant/KIVI in research) is a distinct lever from weight quantization, with its own quality and safety profile. See [[quantization-performance]].

## Inference efficiency

- [[prefix-caching]] — reusing cached KV so repeated prompt content is never prefilled twice — the highest-leverage prompt-processing optimization
- [[paged-attention]] — OS-style paging of the cache; lifts effective utilization from 20–38% to 96.3% and enables ref-counted prefix sharing
- [[grouped-query-attention]] — the architectural lever that shrinks the cache itself
