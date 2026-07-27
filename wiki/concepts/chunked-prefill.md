---
title: Chunked Prefill
type: concept
created: 2026-07-27
updated: 2026-07-27
sources:
  - sarathi-chunked-prefill-agrawal-2023.md
  - distserve-zhong-2024.md
  - splitwise-patel-2023.md
status: complete
importance: high
---

# Chunked Prefill

**Chunked prefill** splits a long prompt's prefill into fixed-size chunks and packs each chunk into a batch alongside other requests' decode tokens — "piggybacking decodes with chunked prefills." Introduced by **SARATHI** ([[sarathi-chunked-prefill-agrawal-2023|Agrawal et al. 2023]]), it is now standard in vLLM/TensorRT-LLM schedulers.

## Why it exists

Decode is catastrophically inefficient per token: measured at **200× / 100× / 16.7×** the per-token cost of prefill at batch 1 / 2 / 18 (LLaMA-13B, A6000). Prefill saturates the GPU; decode starves it. **Decode-maximal batching** fills each batch with one prefill chunk plus as many decodes as fit, concatenated into the same matmuls so weights are fetched once — converting decode from memory-bound to compute-bound.

Results: **decode throughput up to 10×** (2.8–10× range), 1.33× end-to-end on LLaMA-13B/A6000, and under pipeline parallelism on GPT-3 it cuts median pipeline-bubble time **6.29×** for **1.91× end-to-end**.

> [!warning] Chunked prefill makes *prefill slower*, deliberately
> This is a common misreading. Chunking lowers arithmetic intensity and forces KV reloads, costing **~5× total prefill time at chunk 64**, ~20% prefill loss at chunk 256, ~10% at chunk 512. The authors are explicit about the trade: *"even a 5x overhead in prefills is acceptable if the decodes can be optimized by 2x or more."*
>
> So chunked prefill is a **decode-throughput and pipeline-utilization** technique that *spends* prefill. It does help *other* requests' TTFT by reducing head-of-line blocking, but it does not make prompt processing faster. Chunk sizes should be multiples of 128 (tile quantization); 256 beat 320.

## The disaggregation counter-argument

> [!warning] Contradiction — chunked prefill vs. phase disaggregation
> **SARATHI's claim**: interleaving prefill chunks with decodes is the right way to reconcile the two phases' opposite bottlenecks.
> **[[distserve-zhong-2024|DistServe]]'s counter-claim** (OSDI 2024): chunking *"trades TTFT for TPOT"* and the trade cannot be tuned away — small chunks can't saturate the GPU alone, large chunks leave no room to piggyback, and chunking incurs **O(N²) KV-cache HBM loads for N chunks vs O(N)** unchunked, worsening as context grows. DistServe instead runs prefill and decode on **separate GPUs**, improving both SLOs at once: 13B on one A100 colocated = 1.6 rps vs **3.3 rps/GPU (2.1×)** for 2P+1D; headline **7.4× more requests or 12.6× tighter SLO**, and **5.7×** on TTFT-bound code completion. KV transfer costs **<0.1% of total latency**.
> **Status**: unresolved and workload-dependent. Chunking needs no extra hardware and suits single-node deployments; disaggregation needs a fast interconnect and more GPUs but decouples the two SLOs. [[splitwise-patel-2023|Splitwise]] reaches the same disaggregation conclusion from a provisioning angle.

## Related

- [[distserve-zhong-2024]] · [[splitwise-patel-2023]] — the disaggregation alternative.
- [[quantization-performance]] — the underlying prefill/decode asymmetry.
- [[paged-attention]] · [[kv-cache]] · [[vllm]]

## References

- [[sarathi-chunked-prefill-agrawal-2023]] · [[distserve-zhong-2024]] · [[splitwise-patel-2023]]
