---
title: "FlashAttention-3: Fast and Accurate Attention with Asynchrony and Low-precision"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - flash-attention-3-shah-2024.md
arxiv_id: "2407.08608"
year: 2024
authors:
  - Jay Shah
  - Ganesh Bikshandi
  - Ying Zhang
  - Vijay Thakkar
  - Tri Dao
tags:
  - 2024
  - paper
status: complete
importance: high
---

# FlashAttention-3: Fast and Accurate Attention with Asynchrony and Low-precision

**Jay Shah, Ganesh Bikshandi (equal contribution), Ying Zhang, Vijay Thakkar, Pradeep Ramani, Tri Dao** (Colfax Research; Meta; NVIDIA; Georgia Tech; Princeton University; Together AI) — [arXiv:2407.08608](https://arxiv.org/abs/2407.08608), submitted 11 Jul 2024 (v2 12 Jul 2024). *No venue is stated in the arXiv comments field.*

## Summary

[[flash-attention]]-2 was written against "a simplified synchronous model" of the GPU and makes "no explicit use of asynchrony and low-precision in its design." On Hopper that costs dearly: FlashAttention-2 reaches only **35% utilization on an H100**, against 80–90% for an optimized GEMM. FlashAttention-3 keeps the same exact-attention algorithm — tiling, online softmax, recomputation, no approximation — and rebuilds the kernel around two Hopper-specific hardware realities: dedicated asynchronous units (Tensor Cores via WGMMA, and the Tensor Memory Accelerator for memory movement, both running independently of the CUDA cores), and hardware FP8 support.

Three techniques. (1) **Producer–consumer asynchrony via warp specialization:** producers (TMA loads) and consumers (WGMMA + softmax) live in separate warps in a software-pipelined scheme, so memory and instruction-issue latency is hidden rather than stalled on. (2) **Hiding softmax under asynchronous block-wise GEMMs:** softmax is the low-throughput non-GEMM work (exponentials, FP multiply-add) and normally sits on the critical path between the two matmuls. The algorithm is reworked to break that sequential dependency — in the 2-stage version, while softmax runs on one block of the scores matrix, WGMMA is already computing the next block in the asynchronous proxy. (3) **FP8 Tensor Cores**, which roughly double measured TFLOPs/s but need care about layout conformance (FP32 accumulator vs. FP8 operand layouts, plus an in-kernel transpose of V) and about accuracy.

The FP8 accuracy story is the second half of the contribution and is quantization work in its own right. Instead of one scale per tensor, FlashAttention-3 uses **block quantization** (one scalar per B_r×d or B_c×d block — free, because the algorithm already operates blockwise, and fusable with the preceding rotary embedding since that op is bandwidth-bound) plus **incoherent processing**: multiply Q and K by a random orthogonal matrix M before quantizing. Since MMᵀ = I, `(QM)(KM)ᵀ = QKᵀ` leaves the attention output mathematically unchanged, but each entry of QM becomes a random sum of entries of Q, spreading outlier features out so FP8 quantizes them better. M is chosen as random ±1 diagonals times a Hadamard matrix, so it costs O(d log d) rather than O(d²) and fuses into the rotary embedding for free. Results on H100 SXM5: **1.5–2.0× over FlashAttention-2 forward (up to 740 TFLOPs/s, 75% utilization), 1.5–1.75× backward, FP8 close to 1.2 PFLOPs/s**, and FP8 RMSE **2.6× lower** than a per-tensor-quantized FP8 baseline.

## Key points

- **Still exact attention in FP16/BF16.** The algorithm is unchanged in what it computes; asynchrony is a scheduling change. FP8 is the one place precision is genuinely traded, and the paper measures that trade explicitly.
- **The gap being closed:** FlashAttention-2 hits **35% utilization on H100** vs. 80–90% for optimized GEMM — largely from using Ampere-era instructions on Hopper Tensor Cores and from a synchronous execution model.
- **Hopper memory hierarchy (Table 1, H100 SXM5):** GMEM/HBM 80 GiB @ 3.35 TB/s; L2 50 MiB @ 12 TB/s; SMEM 228 KiB per SM @ 31 TB/s per GPU; RMEM 256 KiB per SM.
- **Speedups (H100 80GB SXM5, FP16, seq. 512–16k at 16k total tokens, hidden dim 2048, head dim 64/128/256):**

| Comparison | Speedup |
|---|---|
| vs. FlashAttention-2, forward | **1.5–2.0×** |
| vs. FlashAttention-2, backward | **1.5–1.75×** |
| vs. FlashAttention-2 in Triton (H100-specific instrs.) | up to 1.5× |
| vs. standard PyTorch attention | **3–16×** |
| vs. cuDNN (vendor, closed-source, H100-optimized) | FP16 surpasses it for seq. ≥ 1k |

- **Absolute throughput:** FP16 up to **740 TFLOPs/s = 75%** of H100 theoretical max; FP8 **close to 1.2 PFLOPs/s**. For FP8, head dim 64 beats cuDNN, head dims 128/256 are at par without causal masking and behind with it.
- **Ablation (non-causal FP16, batch 4, seqlen 8448, 16 heads, head dim 128)** — both asynchrony techniques matter independently:

| Configuration | Time | TFLOPs/s |
|---|---|---|
| FlashAttention-3 (full) | 3.538 ms | **661** |
| No GEMM-softmax pipelining, with warp-specialization | 4.021 ms | 582 |
| GEMM-softmax pipelining, no warp-specialization | 4.105 ms | 570 |

- **Numerical error (RMSE vs. an FP64 reference, with synthetic outliers: N(0,1) + N(0,100)·Bernoulli(0.001)):**

| FP16 | Baseline | FlashAttention-2 | FlashAttention-3 |
|---|---|---|---|
| RMSE | 3.2e-4 | 1.9e-4 | 1.9e-4 |

| FP8 (e4m3) | Baseline (per-tensor) | FlashAttention-3 | No block quant | No incoherent processing |
|---|---|---|---|---|
| RMSE | 2.4e-2 | **9.1e-3** | 9.3e-3 | 2.4e-2 |

  FP16 FlashAttention-2/-3 are **1.7×** more accurate than a standard implementation because softmax intermediates are kept in FP32. In FP8 the **2.6×** gain comes almost entirely from **incoherent processing** — removing it returns RMSE to the 2.4e-2 baseline, while removing block quantization alone costs almost nothing (9.1e-3 → 9.3e-3).
- **Incoherent processing is output-preserving in exact arithmetic:** MMᵀ = I ⟹ (QM)(KM)ᵀ = QKᵀ. It only changes what the FP8 grid has to represent.
- **Stated limitations:** LLM *inference* is explicitly not yet optimized for; the FP8 kernel does not yet use a persistent-kernel design; effects of low-precision attention on large-scale training are not characterized.

## Concepts & entities

- [[flash-attention]] — the third generation of the kernel; retains tiling, online softmax, and recomputation, and rewrites the schedule for Hopper's asynchronous units.
- [[attention-mechanism]] — identified as "the primary computational bottleneck" of the Transformer, quadratic in sequence length.
- [[transformer-architecture]] — the layer being accelerated; the paper's stated motivation is unlocking long-context, multi-modal, and long-horizon agent workloads.
- [[long-context-llm]] — the explicit target: multi-document reasoning, large codebases, high-resolution images, audio, video, long-history user interaction.
- [[quantization]] — FP8 (e4m3) attention with block-level scales instead of per-tensor scales, plus Hadamard-based incoherent processing to tame outlier features.
- [[quantization-performance]] — the FP8 path nearly doubles measured TFLOPs/s (740 TFLOPs/s FP16 → ~1.2 PFLOPs/s FP8) while cutting RMSE 2.6× versus naive per-tensor FP8.
- [[state-space-model]] — cited in the paper's framing as one of the "alternative architectures" that FlashAttention's line of work is an alternative to: make exact attention fast rather than replace it.

## References

- [arXiv:2407.08608](https://arxiv.org/abs/2407.08608)
