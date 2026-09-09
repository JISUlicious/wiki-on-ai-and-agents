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

- [[cross-model-kv-cache-transfer]] — reusing one model's KV in a *different* model so the receiver skips prefill — the cross-model counterpart to prefix caching

## Related

- [[linear-attention]] — the architectural way to not have one — Kimi Linear cuts KV cache 75% via a 3:1 hybrid

## Related 2026 sources

- [[toktier-exact-stateful-tokenization-zhang-2026]] — once prefix caching works the bottleneck moves upstream: tokenization grows to 64% of TTFT, because caching cannot start until token IDs exist
- [[sparse-prefill]] — every token-dropping method leaves dropped tokens with **no KV entry** — gone for the generation and later turns; only LazyLLM and SlimInfer can revive them
- [[token-pruning]] — distinct from KV *eviction* (SnapKV, H2O): pruning skips the tokens' attention and FFN during prefill, eviction prefills everything then forgets
- [[weight-streaming]] — the KV cache competes with the streamed-expert cache for one RAM budget, and KV wins: a 126 GB MoE streamed on a 64 GB Mac falls from 4.65 tok/s at 512 context to **0.32 at 40K**, and KV quantization restores it to 4.16. At long context, quantizing the KV cache is the precondition for streaming, not an optimization of it
