---
title: "Efficient Memory Management for Large Language Model Serving with PagedAttention"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - paged-attention-kwon-2023.md
arxiv_id: "2309.06180"
year: 2023
venue: SOSP 2023
authors:
  - Woosuk Kwon
  - Zhuohan Li
  - Siyuan Zhuang
  - Joseph E. Gonzalez
  - Ion Stoica
tags:
  - 2023
  - paper
status: complete
importance: high
---

# Efficient Memory Management for Large Language Model Serving with PagedAttention

**Woosuk Kwon, Zhuohan Li, Siyuan Zhuang, Ying Sheng, Lianmin Zheng, Cody Hao Yu, Joseph E. Gonzalez, Hao Zhang, Ion Stoica** (UC Berkeley; Stanford; UC San Diego) — [arXiv:2309.06180](https://arxiv.org/abs/2309.06180), SOSP '23 (Koblenz, Germany, Oct 2023).

## Summary

The paper identifies GPU memory — specifically the [[kv-cache]] — as the binding constraint on LLM serving throughput. On an A100 40GB serving a 13B model, roughly 65% of memory is model weights and close to 30% is dynamic KV cache. Prior systems (FasterTransformer, Orca) store each request's KV cache as one contiguous tensor sized to the request's *maximum possible* sequence length, because standard attention kernels demand contiguity. That produces three kinds of waste: reserved-but-not-yet-used slots, internal fragmentation from over-provisioning, and external fragmentation from the allocator. Measured across their workloads, only **20.4%–38.2%** of KV cache memory in existing systems is actually effective.

**PagedAttention** borrows OS virtual memory: the KV cache of a sequence is split into fixed-size *blocks* of B tokens, each block holding keys/values for a contiguous span of positions, and blocks are stored **non-contiguously** in physical GPU memory with a per-sequence block table mapping logical to physical blocks. The attention kernel is rewritten to gather across blocks. Uniform block size eliminates external fragmentation entirely; on-demand allocation bounds internal fragmentation to at most one partially-filled block per sequence. In the same experiment where baselines achieve 20.4%–38.2% effective memory, **vLLM reaches 96.3%** — i.e. under 4% waste. More free memory means a much larger batch, and since decoding is memory-bandwidth-bound and under-utilizes the GPU, a larger batch translates fairly directly into throughput.

Crucially the win is **throughput via batching, not per-request prefill latency**. PagedAttention does not make a single prompt's prefill compute any cheaper — in fact the block-table indirection makes the attention kernel itself *slower*, by 20–26% versus FasterTransformer's highly-optimized implementation. What it buys is the ability to keep 2–4x more requests resident simultaneously. The second lever is **sharing**: because logical blocks are indirected through a block table, multiple sequences can point at the same physical block, with a reference count and **copy-on-write** at block granularity when one diverges. This is the direct ancestor of production prefix caching — the paper explicitly proposes that a provider pre-compute and pin the KV blocks of a known shared system prompt / few-shot prefix, so "the prompt phase computation only needs to execute on the user's task input." That *is* a prefill-latency win, but it comes from the sharing mechanism, not from paging per se.

## Key points

| Claim | Number |
|---|---|
| Effective KV cache memory, prior systems | 20.4% (Orca Max) / 26.8% (Orca Pow2) / 38.2% (Orca Oracle) |
| Effective KV cache memory, vLLM | **96.3%** |
| Memory split, 13B model on A100 40GB | ~65% weights, >30% KV cache, rest activations |
| End-to-end throughput vs. state of the art | **2–4x** at equal latency |
| Sustained request rate vs. Orca (Oracle), ShareGPT | 1.7x–2.7x |
| Sustained request rate vs. Orca (Max), ShareGPT | 2.7x–8x |
| Sustained request rate vs. FasterTransformer | up to 22x |
| Concurrent requests, OPT-13B | 2.2x more than Orca (Oracle), 4.3x more than Orca (Max) |
| Attention kernel overhead from paging | **+20–26% latency** vs. FasterTransformer kernel |

Sharing / prefix results:

- **Parallel sampling** (n samples from one prompt): prompt KV is ~12% of total KV memory in their setup and is shared across samples. Memory saving 6.1%–9.8% (Alpaca), 16.2%–30.5% (ShareGPT).
- **Beam search**: sharing extends past the prompt into the common beam prefix, so savings are much larger — 37.6%–55.2% (Alpaca), 44.3%–66.3% (ShareGPT). Speedup over Orca (Oracle) on OPT-13B/Alpaca rises from 1.3x for basic sampling to **2.3x for beam search with width 6**.
- **Shared prefix** (LLaMA-13B, WMT16 En→De, instruction + examples prefix): **1.67x** higher throughput than Orca (Oracle) with a one-shot prefix, **3.58x** with a five-shot prefix — the gain grows with prefix length, which is exactly the prefix-caching argument.
- Sharing is implemented via reference-counted physical blocks plus copy-on-write on the first block a sequence writes into after diverging; sequences in one request are gang-scheduled so they are preempted/restored together.
- Eviction under memory pressure is all-or-nothing per sequence, via swapping blocks to CPU RAM or recomputation.
- Block size is a tradeoff: too small under-utilizes GPU parallelism in the kernel, too large re-introduces internal fragmentation. vLLM defaults to 16 tokens per block.
- Evaluated on OPT-13B / 66B / 175B and LLaMA-13B with ShareGPT and Alpaca traces; ShareGPT sequences are ~8.4x longer than Alpaca's, and vLLM's advantage is larger there — gains grow with longer sequences, larger models, and more complex decoding.

## Concepts & entities

- [[kv-cache]] — the object being managed; the paper's whole thesis is that KV cache memory, not compute, caps serving batch size.
- [[vllm]] — the serving system built on PagedAttention; this paper is its origin.
- [[paged-attention]] — non-contiguous, block-paged KV storage with a per-sequence block table, introduced here.
- [[prefix-caching]] — the shared-prefix / copy-on-write mechanism here is the direct precursor: pre-computed prefix blocks let prefill skip the shared portion.
- [[attention-mechanism]] — the attention kernel is modified to read K/V block-by-block rather than from one contiguous tensor.
- [[transformer-architecture]] — autoregressive decoder inference is the workload being served.
- [[long-context-llm]] — benefits scale with sequence length, since fragmentation waste grows with reserved max-length allocations.
- [[flash-attention]] — the other major memory-aware attention kernel line of work; PagedAttention addresses cache *allocation* rather than on-chip tiling of the attention computation.

## References

- [arXiv:2309.06180](https://arxiv.org/abs/2309.06180)
