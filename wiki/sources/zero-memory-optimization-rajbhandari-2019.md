---
title: "ZeRO: Memory Optimizations Toward Training Trillion Parameter Models"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - zero-memory-optimization-rajbhandari-2019.md
arxiv_id: "1910.02054"
year: 2019
authors:
  - Samyam Rajbhandari
  - Jeff Rasley
  - Olatunji Ruwase
  - Yuxiong He
tags:
  - 2019
  - paper
status: complete
importance: high
---

# ZeRO: Memory Optimizations Toward Training Trillion Parameter Models

**Rajbhandari, Rasley, Ruwase, He** (Microsoft) — [arXiv:1910.02054](https://arxiv.org/abs/1910.02054). No venue stated in the arXiv listing.

## Summary

ZeRO is the paper that gave the field its standard accounting of *where training memory actually goes*. It splits total training memory into **model states** (optimizer states, gradients, parameters) and **residual states** (activations, temporary buffers, fragmented memory), then shows that for large transformer models the model states dominate — and within model states, the **optimizer states dominate**. Under mixed-precision Adam the paper's per-parameter budget is **16 bytes per parameter**, of which **12 are optimizer states**. Plain data parallelism replicates all 16Ψ bytes on every device, which is pure redundancy.

The fix is ZeRO-DP: partition the model states across the Nd data-parallel ranks instead of replicating them, materializing what each rank needs on demand. This is delivered as three cumulative stages — **P_os** (optimizer-state partitioning, later "ZeRO-1"), **P_os+g** (adds gradient partitioning, "ZeRO-2"), and **P_os+g+p** (adds parameter partitioning, "ZeRO-3") — giving **4x, 8x, and Nd-fold** reduction in per-device model-state memory respectively. The first two stages cost *no* extra communication over baseline DP (2Ψ volume); the third costs a 1.5x increase.

A second, less-cited half of the paper is **ZeRO-R**, which attacks the residual states: partitioned activation checkpointing (P_a), constant-size fused buffers (C_B), and proactive memory defragmentation (M_D). The evaluated system, **ZeRO-100B** (P_os+g plus ZeRO-R), trains models up to **170B parameters on 400 V100 GPUs** at **15 PetaFlops aggregate** (38 TFlops/GPU, >30% of peak), an 8x model-size and 10x speed improvement over the Megatron-LM baseline, and powered **Turing-NLG (17B)**.

## Key points

### The per-parameter memory breakdown (mixed-precision Adam, Ψ parameters)

| Component | Precision | Bytes/param |
|---|---|---|
| Parameters | fp16 | 2Ψ |
| Gradients | fp16 | 2Ψ |
| Optimizer: fp32 parameter copy | fp32 | 4Ψ |
| Optimizer: Adam momentum | fp32 | 4Ψ |
| Optimizer: Adam variance | fp32 | 4Ψ |
| **Total** | | **16Ψ** |

The paper writes this as `2Ψ + 2Ψ + KΨ = 16Ψ` with **K = 12** the "memory multiplier of the optimizer states". Verbatim: *"the optimizer states usually consume the most memory, specially when mixed-precision training is applied."* Concretely, **GPT-2 at 1.5B params needs ≥24 GB of model states**, against only **3 GB** for the fp16 parameters alone.

### Residual states (the other half)

- GPT-2 1.5B, seq length 1K, batch 32: **~60 GB of activations**. Activation checkpointing cuts this to **~8 GB** (≈sqrt reduction) at **33% recompute overhead**.
- A 100B-param GPT-like model still needs **~60 GB** for activations at batch 32 *even with* checkpointing.
- Temporary buffers: a flattened fp32 all-reduce buffer for a 1.5B model is **6 GB**; for a 3B model, **12 GB**.
- Memory fragmentation can cause OOM even when free memory exists.

### The three ZeRO-DP stages

| Stage | What is partitioned | Per-device model states | Reduction | Comm. volume |
|---|---|---|---|---|
| Baseline DP | nothing | 16Ψ | 1x | 2Ψ |
| **P_os** (ZeRO-1) | optimizer states | 4Ψ + 12Ψ/Nd ≈ 4Ψ | **4x** | 2Ψ (same as DP) |
| **P_os+g** (ZeRO-2) | + gradients | 2Ψ + 14Ψ/Nd ≈ 2Ψ | **8x** | 2Ψ (same as DP) |
| **P_os+g+p** (ZeRO-3) | + parameters | 16Ψ/Nd | **Nd x** | 3Ψ (1.5x DP) |

Worked example from the paper (7.5B model, Nd = 64): **120 GB → 31.4 GB (P_os) → 16.6 GB (P_os+g) → 1.9 GB (P_os+g+p)**. Table 1 extends this: a 1T model at Nd=1024 needs 16,000 GB baseline, 4011 / 2013 / 15.6 GB under the three stages. Combined with model parallelism of degree Nm, the theoretical reduction is **Nd × Nm**.

Model-size implication at Nd = 64: up to **7.5B / 14B / 128B** parameters under P_os / P_os+g / P_os+g+p. At Nd = 1024 with all stages, **1 trillion parameters** (16 TB of model states ÷ 1024 = 16 GB/GPU). Without ZeRO, DP alone caps out below **1.5B** (measured: PyTorch DDP OOMs at 1.4B).

### ZeRO-R (residual states)

- **P_a — partitioned activation checkpointing**: removes the activation replication that model parallelism forces. 100B model, batch 32, seq 1024, MP=16: **~33 GB → ~2 GB per GPU**; with `P_a+cpu` offload, "nearly zero".
- **C_B — constant-size buffers**: caps fused-buffer size so it no longer scales with model size.
- **M_D — memory defragmentation**: proactively manages the interleaving of short- and long-lived tensors created by checkpointing.

### Measured results (ZeRO-100B, 400 V100s / 25 DGX-2 nodes)

| Result | Number |
|---|---|
| Max model size | **170B** params (vs ~40B for Megatron-LM alone) |
| Sustained throughput | **15 PetaFlops** aggregate, **38 TFlops/GPU**, >30% of peak, for 8B–100B models |
| Speedup vs SOTA | up to **10x** |
| Scaling | **super-linear** from 64 → 400 GPUs |
| No-model-parallelism ceiling | **13B** params on 128 GPUs at >40 TFlops/GPU (vs 1.4B for DDP) |
| Measured P_os model size | 6.2B (MP=1, 64 GPUs) up to 100B (MP=16, 1024 GPUs) — matches theory |
| Downstream artifact | **Turing-NLG, 17B** params |

## Concepts & entities

- [[training-memory-anatomy]] — this paper is the canonical source for the 16 bytes/param split and the model-states vs residual-states taxonomy.
- [[zero-redundancy-optimizer]] — the technique introduced here; P_os / P_os+g / P_os+g+p are ZeRO stages 1/2/3.
- [[distributed-training]] — ZeRO-DP recovers data parallelism's compute efficiency while matching model parallelism's memory efficiency.
- [[mixed-precision-training]] — the fp16/fp32 scheme that produces the 16-bytes-per-parameter figure (K=12).
- [[activation-recomputation]] — the ~sqrt memory reduction at 33% recompute overhead that ZeRO-R extends via activation partitioning.
- [[pre-training]] — the workload ZeRO targets: large transformer language-model pre-training.
- [[transformer-architecture]] — all evaluated models are GPT-2-like transformers.

## References

- [arXiv:1910.02054](https://arxiv.org/abs/1910.02054)
