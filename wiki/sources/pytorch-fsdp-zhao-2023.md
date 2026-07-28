---
title: "PyTorch FSDP: Experiences on Scaling Fully Sharded Data Parallel"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - pytorch-fsdp-zhao-2023.md
arxiv_id: "2304.11277"
year: 2023
venue: PVLDB 16(12) 2023
authors:
  - Yanli Zhao
  - Andrew Gu
  - Rohan Varma
  - Liang Luo
  - Shen Li
tags:
  - 2023
  - paper
status: complete
importance: high
---

# PyTorch FSDP: Experiences on Scaling Fully Sharded Data Parallel

**Zhao, Gu, Varma, Luo, … Li** (Meta AI) — [arXiv:2304.11277](https://arxiv.org/abs/2304.11277), **PVLDB 16(12):3848-3860, 2023** (venue stated on the paper's first page, not in the arXiv comments).

## Summary

FSDP is PyTorch's native answer to the same problem [[zero-redundancy-optimizer]] solved: shard parameters, gradients, and optimizer states across data-parallel ranks, all-gather each shard on demand for compute, then discard it. The paper is explicit that ZeRO inspired the design but insists FSDP is "intrinsically different" — the difference is engineering, not algorithm, and the paper is essentially an experience report about that engineering.

The central design object is the **FlatParameter**: within each FSDP unit, all parameters are flattened and concatenated into one 1D tensor, right-padded, then split into equal chunks across ranks. This coalesces many small collectives into one large `AllGather` (and `ReduceScatter` for gradients) and guarantees even shard sizes. Contrast with ZeRO/cross-replica sharding, which the paper says shard per-parameter and rely on `Broadcast`/`Gather`, risking uneven workload across devices — and, more importantly, are implemented by reaching into framework internals (tensor storage, memory management), so they break as those internals evolve. FSDP is instead co-designed with PyTorch's tensor implementation, dispatcher, and CUDA caching allocator.

The other practical differences are a **configurable sharding factor F** (F=1 is pure replication/DDP, F=W is full sharding, 1<F<W is **hybrid sharding** matched to the interconnect topology), **deferred initialization** so models too large for one GPU can still be constructed unit by unit, backward prefetching, and a **rate limiter** that throttles `AllGather` issuance to prevent the CPU thread from over-allocating CUDA blocks and triggering defragmentation. Peak parameter memory is O(Σψᵢ/F + maxᵢ ψᵢ): the sharded total plus the single largest unsharded unit — which makes explicit the memory-vs-throughput knob that unit granularity controls (finer units = lower peak memory, more collectives).

## Key points

### How it differs from ZeRO-3 in practice

| | ZeRO-3 / cross-replica sharding | PyTorch FSDP |
|---|---|---|
| Sharding unit | per-parameter tensors | **FlatParameter** — one flattened, padded 1D tensor per FSDP unit |
| Collectives | Broadcast / Gather | **AllGather** (params) + **ReduceScatter** (grads) |
| Shard balance | can be uneven → straggler cost | equal-sized chunks by construction |
| Framework coupling | modifies ML-framework internals; fragile across versions | co-designed with PyTorch tensor impl, dispatcher, CUDA caching allocator |
| Sharding degree | full shard across the world | **sharding factor F**: 1 (replicate) … W (full shard), plus hybrid |
| Also contrasted: MiCS | global AllReduce then shard within group → every rank holds full gradients | AllGather + ReduceScatter → gradients sharded per layer, lower memory |

**Hybrid sharding math**: with world size W and G accelerators per host, setting F = W/G confines AllGather/ReduceScatter to a host. Cross-host traffic per GPU becomes **2M(W/G − 1)/W**, versus **2M(W−1)/W** for full replication and **3M(W−1)/W** for full sharding.

**Peak parameter memory**: for N FlatParameters with sizes ψ₁…ψ_N summing to Ψ and sharding factor F, memory is in **O(Σψᵢ/F + maxᵢ ψᵢ)** with **O(N)** collectives per iteration — the explicit memory/throughput trade-off users control by choosing wrapping granularity.

### Measured results (up to 512× A100 80GB, 2 Tb/s RoCE)

| Result | Number |
|---|---|
| GPT-175B (minGPT), batch 1 / 2 | **>173** and **>186 TFLOPS per GPU** |
| …as hardware utilization | **~55%** and **~60%** of the A100's 312 TFLOPS BF16 peak |
| GPT-175B scaling | **linear in TFLOPS from 128 → 512 GPUs** |
| Backward prefetching | **~18% speedup** on GPT-175B, consistent across cluster sizes |
| T5: FSDP vs DDP | comparable at 611M and 2.28B; **DDP OOMs above 2.28B**, FSDP handles **11B** and gains further with BF16 |
| T5-11B, 8 → 512 GPUs | **7% regression** in per-GPU TFLOPS (communication starts to outrun computation) |
| Models evaluated | T5-611M/2B/11B, minGPT-175B, DHEN recommender (768B sparse + 550M dense), RegNet-9B, DeepViT-8B |

Notes from the large-model runs: Adam was used deliberately "to incur the costly two optimizer states per parameter," with **activation checkpointing and BF16 mixed precision** on. Peak memory falls consistently as GPUs are added, since each rank's shard shrinks. The 128-GPU / batch-2 anomaly (lower TFLOPS) was traced to CUDA memory defragmentation — the backward pass took **85.56%** of iteration latency versus a normal **~67%** — because at 128 GPUs each shard is large enough to exhaust the 80 GB allocator.

Sharding strategy trade-off on DHEN: **RAF** (reshard-after-forward) gives the smallest memory footprint at lower QPS; **NRAF** (no-reshard-after-forward) keeps unsharded parameters resident through backward for lower communication at higher memory.

## Concepts & entities

- [[distributed-training]] — FSDP is the PyTorch-native sharded data-parallel implementation; beta as of PyTorch 2.0.
- [[zero-redundancy-optimizer]] — the acknowledged inspiration; this paper's related-work section is the clearest published statement of FSDP-vs-ZeRO differences.
- [[training-memory-anatomy]] — quantifies peak memory as sharded-total plus one unsharded unit, and notes Adam's "two optimizer states per parameter" as the cost being sharded.
- [[mixed-precision-training]] — BF16 is what unlocks both the larger models and the higher TFLOPS in these experiments.
- [[activation-recomputation]] — enabled throughout the large-model runs alongside sharding.
- [[transformer-architecture]] — T5 and minGPT-175B are the primary benchmarks.

## References

- [arXiv:2304.11277](https://arxiv.org/abs/2304.11277)
