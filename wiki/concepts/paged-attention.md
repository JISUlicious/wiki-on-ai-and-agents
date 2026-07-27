---
title: PagedAttention
type: concept
created: 2026-07-27
updated: 2026-07-27
sources:
  - paged-attention-kwon-2023.md
builds-on:
  - [[kv-cache]]
status: complete
importance: high
---

# PagedAttention

**PagedAttention** applies OS virtual-memory paging to the [[kv-cache|KV cache]]: instead of one contiguous reservation per sequence, the cache is split into fixed-size blocks that can live anywhere in GPU memory, indexed by a per-sequence **block table**. Introduced by [[paged-attention-kwon-2023|Kwon et al. (SOSP 2023)]], it is the foundation of [[vllm|vLLM]] and now standard across serving stacks.

## The problem it solves

Contiguous allocation forces you to reserve for the *maximum* possible length. Measured effective KV utilization in prior systems was **20.4%–38.2%** — most of the cache was reserved-but-unused slots plus internal and external fragmentation. vLLM reaches **96.3%**.

Recovered memory becomes batch size, and batch size becomes throughput: **2–4× end-to-end** at equal latency (2.2× more concurrent requests on OPT-13B).

> [!warning] This is throughput via batching — not faster prefill
> The block-table indirection makes the attention kernel itself **20–26% slower** than FasterTransformer's. PagedAttention buys aggregate throughput by fitting more sequences, not lower per-request latency. Do not reach for it to reduce TTFT on a single request.

## The prefix-sharing mechanism (the prefill-relevant part)

Because blocks are physical and ref-counted, they can be **shared across sequences** with copy-on-write at block granularity. This is the direct ancestor of [[prefix-caching]]:

| Sharing scenario | Saving |
|---|---|
| Parallel sampling | 6.1–9.8% memory |
| Beam search | 37.6–55.2% (up to 66.3% on ShareGPT) |
| **Shared prefix, 1-shot** | **1.67× throughput** |
| **Shared prefix, 5-shot** | **3.58× throughput** |

For a pre-computed shared prefix the paper notes the prompt phase *"only needs to execute on the user's task input"* — i.e. the shared portion of prefill is **skipped entirely**, which is the largest prefill win available.

## Related

- [[prefix-caching]] — generalizes the sharing idea to automatic cross-request reuse.
- [[kv-cache]] · [[vllm]] · [[chunked-prefill]] · [[quantization-performance]]

## References

- [[paged-attention-kwon-2023]]
