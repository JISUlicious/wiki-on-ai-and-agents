---
title: "ZeRO-Offload: Democratizing Billion-Scale Model Training"
type: source
created: 2026-07-29
updated: 2026-07-29
sources:
  - zero-offload-ren-2021.md
arxiv_id: "2101.06840"
year: 2021
authors:
  - Jie Ren
  - Samyam Rajbhandari
  - Reza Yazdani Aminabadi
  - Olatunji Ruwase
  - Yuxiong He
tags:
  - 2021
  - paper
status: complete
importance: high
---

# ZeRO-Offload: Democratizing Billion-Scale Model Training

**Ren, Rajbhandari, Yazdani Aminabadi, Ruwase, He** (UC Merced + Microsoft) — [arXiv:2101.06840](https://arxiv.org/abs/2101.06840). No venue stated in the arXiv listing.

## Summary

ZeRO-Offload takes the same memory accounting as [[zero-redundancy-optimizer]] — **16 bytes per parameter** for mixed-precision Adam — and asks a different question: instead of sharding those bytes across many GPUs, can you push most of them off the GPU entirely, onto CPU DRAM, and still train efficiently? The answer is yes, and the paper derives the offload split from first principles rather than picking one heuristically.

The analysis models training as a data-flow graph whose nodes are fp16 parameters (p16), fp16 gradients (g16), and the fp32 "update super-node" (fp32 parameter copy + Adam momentum + variance), with edge weights of 2M or 4M bytes. The theoretical minimum CPU↔GPU communication volume for *any* offload strategy is **4M**. Among the strategies achieving that minimum, exactly one maximizes GPU memory savings: **offload the fp16 gradients, all fp32 optimizer states, and the optimizer update computation to CPU; keep the fp16 parameters and the whole forward/backward pass on GPU**. That leaves only the **2M bytes of fp16 parameters** resident on the GPU — an **8x** reduction in model-state memory.

The catch is that the CPU now runs the Adam step. Because the optimizer update is O(M) work while forward/backward is O(M·B), CPU compute is usually not the bottleneck; for small batches the paper adds a hand-optimized **CPU-Adam** (up to 6x faster than PyTorch's CPU Adam) and an optional **one-step delayed parameter update (DPU)** that overlaps the CPU step with GPU compute. Result: **13B parameters trainable on a single 32 GB V100** at **40 TFLOPS**, versus 1.4B / 30 TFLOPS for stock PyTorch — roughly a **10x** model-scale increase on one GPU, with near-linear scaling out to 128 GPUs when more are available.

## Key points

### Memory accounting (identical to ZeRO's)

Mixed-precision Adam needs 2 bytes each for the fp16 parameter and fp16 gradient, plus 4 bytes each for the fp32 parameter copy, momentum, and variance — **16 × M bytes total**. The paper's worked examples: model states for **Megatron-LM (8B) = 128 GB, T5 (11B) = 176 GB, Turing-NLG (17.2B) = 284 GB**.

### Offload strategies analyzed (Table 1 — all achieve the minimum 4M communication)

| FWD-BWD | p16 | g16 | Update | GPU memory | Reduction |
|---|---|---|---|---|---|
| GPU | GPU | GPU | GPU | 16M | 1x (baseline) |
| GPU | GPU | **CPU** | GPU | 14M | 1.14x |
| GPU | GPU | GPU | **CPU** | 4M | 4x |
| GPU | GPU | **CPU** | **CPU** | (2M) | **8x** ← chosen |

> [!note] Typo in the source table
> The paper's Table 1 prints "4M" in the memory column for the chosen last row while reporting an 8x reduction. The prose is unambiguous that only the fp16 parameters stay on GPU, i.e. **2M**, which is what 8x implies.

**What lives where**: GPU holds fp16 parameters and runs forward + backward. CPU holds fp16 gradients, the fp32 parameter copy, fp32 momentum and variance, and runs the parameter update. Communication per step is 4M (gradients down, updated parameters up) — **7x less than L2L's 28M**.

### Scale and throughput

| Result | Number |
|---|---|
| Largest model, single 32 GB V100 | **13B** (PyTorch DDP / ZeRO-2 / Megatron all cap at **1.4B**) — >9x |
| Throughput, single V100 | **40 TFLOPS** for a 10B model, vs **30 TFLOPS** for PyTorch's largest-fitting 1.4B model |
| Largest model, single DGX-2 (16 GPUs) + model parallelism | **70B** (Megatron alone: 15B) — 4.5x |
| Model-scale gain on one DGX-2 | **50x / 4.5x / 7.8x / 4.2x** vs PyTorch / Megatron / ZeRO-2 / L2L |
| Throughput, 1–15B models on one DGX-2 | up to **1.33x / 1.11x / 1.64x** faster than PyTorch / ZeRO-2 / Megatron |
| Throughput vs L2L, single GPU | **+14% average, up to +22%** |
| Multi-GPU scaling | **near-perfect linear** aggregate speedup to **128 GPUs** at **>30 TFlops/GPU** |
| 70B with model parallelism | **>30 TFLOPS/GPU** |

### CPU-Adam latency (seconds, Table 4)

| Params | CPU-Adam | PyTorch CPU | PyTorch GPU |
|---|---|---|---|
| 1B | 0.22 | 1.39 | 0.10 |
| 4B | 1.03 | 5.71 | 0.64 |
| 10B | 2.57 | 14.76 | 1.00 |

So the optimized CPU step is ~5-6x faster than PyTorch on CPU and within ~2.5x of a GPU Adam step — small enough to hide behind GPU compute at realistic batch sizes.

## Concepts & entities

- [[training-memory-anatomy]] — reuses ZeRO's 16 bytes/param model; the key claim is that 14 of those 16 bytes can leave the GPU.
- [[zero-redundancy-optimizer]] — ZeRO-Offload composes with ZeRO-powered data parallelism to scale past one GPU.
- [[distributed-training]] — heterogeneous CPU+GPU training, presented as an alternative to needing more GPUs.
- [[mixed-precision-training]] — the fp16-compute / fp32-state split is exactly what makes the offload partition tractable.
- [[pre-training]] — target workload is billion-scale transformer language-model training.

## References

- [arXiv:2101.06840](https://arxiv.org/abs/2101.06840)
