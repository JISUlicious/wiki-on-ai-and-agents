---
title: ZeRO (Zero Redundancy Optimizer)
type: concept
created: 2026-07-29
updated: 2026-07-29
sources:
  - zero-memory-optimization-rajbhandari-2019.md
  - zero-offload-ren-2021.md
  - pytorch-fsdp-zhao-2023.md
status: complete
importance: high
---

# ZeRO (Zero Redundancy Optimizer)

**ZeRO** removes the redundancy in plain data parallelism. Standard DP replicates *the entire model state* — parameters, gradients, and optimizer states — on every device, so N devices hold N identical copies. ZeRO **shards** them instead, keeping DP's simple programming model while cutting per-device memory by up to N×.

## The three stages

Against the **16Ψ** per-parameter baseline (see [[training-memory-anatomy]]):

| Stage | Shards | Per-device model states | Reduction | Communication |
|---|---|---|---|---|
| **P_os** (ZeRO-1) | optimizer states | 4Ψ + 12Ψ/N_d ≈ 4Ψ | **4×** | 2Ψ — same as DP |
| **P_os+g** (ZeRO-2) | + gradients | 2Ψ + 14Ψ/N_d ≈ 2Ψ | **8×** | 2Ψ — same as DP |
| **P_os+g+p** (ZeRO-3) | + parameters | 16Ψ/N_d | **N_d×** | 1.5× DP |

Stages 1 and 2 are **free** — the same communication volume as ordinary data parallelism. Only stage 3 costs extra traffic, and only 1.5×.

A 7.5B model at N_d=64: **120 GB → 31.4 / 16.6 / 1.9 GB**. At N_d=1024, a 1T model goes 16,000 GB → **15.6 GB** per device.

## ZeRO-R: the residual half

Sharding model states leaves activations untouched, so ZeRO adds:

- **P_a** — partitioned activation checkpointing. A 100B model at batch 32, seq 1024, MP=16 drops from **33 GB → ~2 GB/GPU** (→ ~0 with CPU offload).
- **C_B** — constant-size buffers (a 3B model's fp32 fused buffer alone is **12 GB**).
- **M_D** — defragmentation.

Headline: **ZeRO-100B trains 170B parameters on 400 V100s** at 15 PetaFlops aggregate (38 TFlops/GPU, >30% of peak), with super-linear scaling from 64→400 GPUs, and trains **13B with no model parallelism at all** where DDP OOMs at 1.4B. It powered Turing-NLG 17B.

## ZeRO-Offload: trading PCIe for capacity

[[zero-offload-ren-2021|ZeRO-Offload]] moves the **fp16 gradients, all fp32 optimizer states, and the optimizer update computation** to CPU, keeping **fp16 parameters and the entire forward/backward** on GPU. That leaves 2M bytes on device — an **8× model-state reduction** — at the proven-minimum **4M** CPU↔GPU communication volume.

Result: **13B parameters on a single 32 GB V100 at 40 TFLOPS**, where PyTorch caps at 1.4B. One DGX-2 with model parallelism reaches **70B** (Megatron alone: 15B). Their CPU-Adam runs ~5–6× faster than PyTorch's.

## FSDP: the same idea, re-engineered

[[pytorch-fsdp-zhao-2023|PyTorch FSDP]] is ZeRO-3's design rebuilt for framework co-design. Its own account of the difference: ZeRO shards **per-parameter** with Broadcast/Gather, which can produce **uneven workload across GPUs**, and is implemented by modifying framework internals so it breaks as those internals change. FSDP instead flattens all parameters of a unit into one **FlatParameter** — concatenated, right-padded, split into equal chunks — communicated with **AllGather + ReduceScatter**, co-designed with PyTorch's dispatcher and caching allocator.

It also exposes a **sharding factor F** as an explicit knob: F=1 replicates, F=W fully shards, and intermediate values give *hybrid sharding* with cross-host traffic `2M(W/G−1)/W` versus `3M(W−1)/W` for full sharding. Peak parameter memory is `O(Σψᵢ/F + maxᵢ ψᵢ)`.

Measured: GPT-175B at **>173 / >186 TFLOPS per GPU** (batch 1/2) — roughly **55% / 60% MFU** against the A100's 312 TFLOPS BF16 peak — with linear scaling from 128→512 GPUs. Backward prefetch alone is worth ~18%.

## Where it sits against parallelism

ZeRO shards parameters and optimizer states but **not activations**; tensor and pipeline parallelism shard differently (see [[distributed-training]]). The two are complementary, and the historical record is easy to misread — see the callout on that page about ZeRO's "10× over Megatron" and PTD-P's "70% over ZeRO-3" being a year apart and measuring different configurations.

## Related

- [[training-memory-anatomy]] — the 16Ψ accounting ZeRO attacks.
- [[distributed-training]] · [[activation-recomputation]] · [[8-bit-optimizers-dettmers-2021]] (the orthogonal fix: shrink the states rather than shard them).

## References

- [[zero-memory-optimization-rajbhandari-2019]] · [[zero-offload-ren-2021]] · [[pytorch-fsdp-zhao-2023]]
