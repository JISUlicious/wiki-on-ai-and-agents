---
title: "SARATHI: Efficient LLM Inference by Piggybacking Decodes with Chunked Prefills"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - sarathi-chunked-prefill-agrawal-2023.md
arxiv_id: "2308.16369"
year: 2023
authors:
  - Amey Agrawal
  - Ashish Panwar
  - Jayashree Mohan
  - Nipun Kwatra
  - Ramachandran Ramjee
tags:
  - 2023
  - paper
status: complete
importance: high
---

# SARATHI: Efficient LLM Inference by Piggybacking Decodes with Chunked Prefills

**Amey Agrawal, Ashish Panwar, Jayashree Mohan, Nipun Kwatra, Bhargav S. Gulavani, Ramachandran Ramjee** (Microsoft Research India; Georgia Institute of Technology) — [arXiv:2308.16369](https://arxiv.org/abs/2308.16369). No venue in the arXiv comments (preprint, Aug 2023).

## Summary

This is *the* chunked-prefill paper. Its starting observation is the asymmetry between the two phases of LLM inference: **prefill** processes the whole prompt in parallel and is compute-bound — it saturates the GPU at batch size 1 once B x L >= 512 — while **decode** emits one token per request per step and is memory-bandwidth-bound, spending most of its time re-fetching model weights. Profiled on LLaMA-13B/A6000 at sequence length 1024, the decode cost *per token* is **200x, 100x, and 16.7x** the prefill cost per token at batch sizes 1, 2, and 18 respectively. Decode throughput only saturates at batch sizes around 256, which don't fit alongside the weights and KV cache.

SARATHI's fix has two parts. **Chunked-prefills** splits a prefill request into equal-sized chunks (e.g. 256 or 512 tokens), each chunk being large enough to saturate GPU compute on its own. **Decode-maximal batching** then builds each batch out of exactly one prefill chunk plus as many decode tokens as will fit. Because prefill and decode share the same weight tensors and the same computation path, the decode tokens are concatenated into the prefill chunk's matrices and ride along in the *same* matmuls — the weights are fetched once and reused for both. This converts decoding from memory-bound to compute-bound, so piggybacked decodes "cost up to an order of magnitude less" than a decode-only batch. Splitting one prefill into many chunks means many decode-maximal batches can be formed per prefill request, maximizing how many decodes get a free ride. A secondary benefit: every batch now has near-identical compute cost, which removes the micro-batch imbalance that creates pipeline bubbles under pipeline parallelism.

The tradeoff is explicit and is about **prefill efficiency**, not aggregate throughput. Chunking hurts prefill two ways: smaller chunks have lower arithmetic intensity (less efficient matmuls), and each chunk must re-read the KV cache of all preceding chunks for attention, so a prefill split into N chunks reloads earlier KV up to N-1 extra times. Measured: chunk size 64 costs **3x** in attention time and about **5x** in total prefill time; chunk 256 keeps end-to-end prefill loss within **20%**; chunk 512 within **10%**. So a request's own prefill takes longer in wall-clock terms — its time-to-first-token gets worse — and SARATHI pays that to win on decode and on aggregate throughput. The authors accept this openly: "even a 5x overhead in prefills is acceptable if the decodes can be optimized by 2x or more." (Note the paper does argue one *latency* benefit: because a long prompt now enters the batch as small chunks rather than one giant prefill, it no longer stalls the decodes of already-running requests the way Orca's scheduling does — a head-of-line-blocking win for other requests, not a TTFT win for the chunked request itself.)

## Key points

| Result | Number |
|---|---|
| Decode cost per token vs. prefill (LLaMA-13B, A6000, seq 1024) | 200x at B=1, 100x at B=2, 16.7x at B=18 |
| Prefill throughput saturation | ~180 tokens/ms once B x L >= 512 |
| Decode throughput saturation batch size | ~256 (seq 1024) — infeasible with full model resident |
| LLaMA-13B / A6000: decode throughput | **up to 10x** (range 2.8x–10x across batch sizes and lengths) |
| LLaMA-13B / A6000: end-to-end throughput | **up to 1.33x** |
| LLaMA-33B / A100: decode throughput | **up to 4.25x** |
| LLaMA-33B / A100: end-to-end throughput | **up to 1.25x** (consistently 1.14x–1.25x) |
| GPT-3 with pipeline parallelism (simulated): median pipeline bubble time per request | **reduced 6.29x** |
| GPT-3 with pipeline parallelism: end-to-end throughput | **1.91x** vs. baseline TP-PP, **1.48x** vs. TP-only |

Chunk-size tradeoff, measured:

- Chunk 64: 3x attention overhead, ~5x total prefill overhead — yet still roughly matches baseline end-to-end because of the decodes it piggybacks.
- Chunk 128: prefill >2x slower than baseline, still **1.16x** higher end-to-end throughput.
- Chunk 256: prefill loss within 20%; best in low P:D regimes, peak gain **1.27x**.
- Chunk 512: prefill loss within 10%; best in high P:D regimes, peak gain **1.23x**.
- Baseline Orca best-case peaks at only 1.11x, and is a special case of SARATHI with chunk size = max sequence length.
- **Tile quantization**: chunk sizes that are multiples of 128 align with GPU matmul tiling and do measurably better — chunk 256 beats chunk 320.
- Ideal chunk size depends on the deployment's expected prefill:decode token ratio (P:D); the paper's sweep covers P:D of 1–200 and sequence lengths up to 3K, and flags very long (10s–100s of thousands) sequences as unaddressed since attention cost grows quadratically and SARATHI's gains come from the linear operators.

## Concepts & entities

- [[chunked-prefill]] — introduced here: split a prompt into equal fixed-size chunks so each chunk still saturates compute while leaving room in the batch for decodes.
- [[kv-cache]] — chunking forces each chunk to re-read the KV of all prior chunks, the second source of the prefill overhead.
- [[attention-mechanism]] — the only operator that does *not* get cheaper from piggybacking (its cost is per-sequence, not shared across the batch), which is why gains shrink at long sequence lengths.
- [[transformer-architecture]] — the prefill/decode split and the six-operation cost breakdown are properties of decoder-only transformer inference.
- [[vllm]] — the ideas here were folded into production serving stacks; vLLM ships chunked prefill as a scheduler option.
- [[paged-attention]] — complementary: PagedAttention makes room for a bigger batch, SARATHI decides what goes *into* each batch.
- [[long-context-llm]] — explicitly listed as future work; quadratic attention erodes the technique's headroom.
- [[mixture-of-experts]] — cited in related work as an orthogonal architectural route to cutting inference cost.
- [[quantization-performance]] — likewise cited as an orthogonal compression-based route.

## References

- [arXiv:2308.16369](https://arxiv.org/abs/2308.16369)
