---
title: "FlashAttention-2: Faster Attention with Better Parallelism and Work Partitioning"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - flash-attention-2-dao-2023.md
arxiv_id: "2307.08691"
year: 2023
authors:
  - Tri Dao
tags:
  - 2023
  - paper
status: complete
importance: high
---

# FlashAttention-2: Faster Attention with Better Parallelism and Work Partitioning

**Tri Dao** (Princeton University; Stanford University — sole author) — [arXiv:2307.08691](https://arxiv.org/abs/2307.08691), submitted 17 Jul 2023. *No venue is stated in the arXiv comments field.*

## Summary

[[flash-attention]] made exact attention IO-aware and got 2–4× wall-clock speedup over optimized baselines with 10–20× memory saving, but it left a lot of silicon idle: its forward pass reached only 30–50% of an A100's theoretical max FLOPs/s and the backward pass only 25–35%, versus 80–90% for a well-optimized GEMM. FlashAttention-2 is a pure **GPU work-scheduling** paper — the algorithm still computes **exact attention** with tiling and recomputation, the output is unchanged, and no approximation is introduced. What changes is how the work is mapped onto thread blocks, warps, and Tensor Cores.

Three changes. (1) **Fewer non-matmul FLOPs.** An A100 does 312 TFLOPs/s of FP16/BF16 matmul but only 19.5 TFLOPs/s of non-matmul FP32, so each non-matmul FLOP costs ~16× a matmul FLOP. FlashAttention-2 keeps an *un-scaled* running output `Õ` and applies the `diag(ℓ)⁻¹` rescaling only once at the end of the loop instead of every iteration, and stores a single logsumexp statistic `L = m + log(ℓ)` for the backward pass instead of the max and sum separately. (2) **Parallelism over sequence length.** FlashAttention parallelized only over batch × heads, giving one thread block per head — fine when that product is ≥ ~80 (an A100 has 108 SMs), but long-context training means small batches and low occupancy. FlashAttention-2 additionally parallelizes over the sequence-length dimension; in the forward pass the outer loop over row blocks is embarrassingly parallel, and in the backward pass one thread block per column block with atomic adds to accumulate dQ. (3) **Better warp partitioning.** Within a thread block (typically 4 or 8 warps), FlashAttention-2 avoids the "split-K" layout, so warps no longer have to write partial results to shared memory and synchronize to combine them.

Two smaller but practically important items: for causal masking, blocks where every column index exceeds every row index (about half the blocks at long sequence length) are skipped entirely, which the paper measures at "around 1.7-1.8× speedup compared to attention without the causal mask"; and MQA/GQA are supported by index manipulation into the head dimension rather than physically duplicating K and V heads. The result is ~2× over FlashAttention, up to **73% of theoretical max throughput** forward and **63%** backward on A100, and **225 TFLOPs/s per A100 (72% MFU)** in end-to-end GPT training. Running the unmodified A100 kernel on an H100 (no TMA, no 4th-gen Tensor Core instructions) already reaches 335 TFLOPs/s, with the author predicting another 1.5–2× from Hopper-native instructions — which is exactly what FlashAttention-3 delivers.

## Key points

- **Still exact attention.** "with no approximation." The algorithmic tweaks explicitly do "not chang[e] the output"; this is a scheduling/partitioning paper, not a new attention formulation.
- **The gap being closed:** FlashAttention forward reaches 30–50% of theoretical max FLOPs/s, backward 25–35%; optimized GEMM reaches 80–90%.
- **Non-matmul FLOPs are ~16× more expensive:** A100 max is 312 TFLOPs/s FP16/BF16 matmul vs. 19.5 TFLOPs/s non-matmul FP32.
- **Kernel speedups (A100 80GB SXM4, seq. 512–16K, 16K total tokens, hidden dim 2048, head dim 64 or 128):**
  - **1.7–3.0×** faster than FlashAttention
  - **1.3–2.5×** faster than FlashAttention in Triton (≈1.3–1.5× forward, ≈2× backward)
  - **3–10×** faster than a standard PyTorch attention implementation
  - Peak **230 TFLOPs/s = 73%** of A100 theoretical max forward; **63%** backward
- **End-to-end GPT training (8×A100 80GB SXM), TFLOPs/s/GPU** — the gain is largest at long context, where FlashAttention-1's low occupancy hurt most:

| Model | Without FlashAttention | FlashAttention | FlashAttention-2 |
|---|---|---|---|
| GPT3-1.3B, 2k context | 142 | 189 | 196 |
| GPT3-1.3B, 8k context | 72 | 170 | **220** |
| GPT3-2.7B, 2k context | 149 | 189 | 205 |
| GPT3-2.7B, 8k context | 80 | 175 | **225** (72% MFU) |

  Overall: up to **1.3× over FlashAttention** and **2.8× over the no-FlashAttention baseline** end-to-end.
- **H100 without Hopper-specific instructions:** up to **335 TFLOPs/s**, with an expected further 1.5–2× available from TMA / 4th-gen Tensor Cores / FP8 (left to future work).
- **Causal masking optimization:** blocks whose column indices all exceed their row indices — "approximately half of the blocks for large sequence length" — are skipped outright, giving "around 1.7-1.8× speedup compared to attention without the causal mask." Only ~1 block per row actually needs the mask applied.
- **Memory:** still O(N) additional memory beyond inputs/output (just the logsumexp vector L ∈ ℝᴺ), and O(N²d) FLOPs, returning `softmax(QKᵀ)V` "with no approximation."
- **MQA/GQA supported natively** by implicitly indexing into the head dimension rather than materializing duplicated K/V heads; the backward pass sums dK/dV across the implicitly duplicated heads.
- **Block sizes** are typically {64, 128} × {64, 128}, hand-tuned per head dimension; larger blocks cut shared-memory traffic but risk register spilling or exceeding shared memory.
- **Framing of the payoff:** "2× faster than FlashAttention... means that we can train models with 16k longer context for the same price as previously training a 8k context model."

## Concepts & entities

- [[flash-attention]] — the direct predecessor; FlashAttention-2 keeps the tiling + recomputation algorithm and rewrites only the GPU work partitioning.
- [[attention-mechanism]] — computed exactly; the paper's contribution lives entirely below the mathematical definition.
- [[transformer-architecture]] — the attention layer is identified as "the main bottleneck in scaling to longer sequences."
- [[long-context-llm]] — motivated by GPT-4's 32k, MPT's 65k, and Claude's 100k contexts; the 8k-context training rows are where the speedup is largest.
- [[grouped-query-attention]] — GQA and MQA are supported by index manipulation instead of duplicating K/V heads, so the KV-cache savings do not cost extra kernel work.
- [[kv-cache]] — the stated purpose of the MQA/GQA variants the kernel supports is reducing KV-cache size at inference.

## References

- [arXiv:2307.08691](https://arxiv.org/abs/2307.08691)
