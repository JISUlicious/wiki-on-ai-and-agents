---
title: "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness"
type: source
created: 2026-07-27
updated: 2026-07-27
sources:
  - flash-attention-dao-2022.md
arxiv_id: "2205.14135"
year: 2022
authors:
  - Tri Dao
  - Daniel Y. Fu
  - Stefano Ermon
  - Atri Rudra
  - Christopher Ré
tags:
  - 2022
  - paper
status: complete
importance: high
---

# FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness

**Tri Dao, Daniel Y. Fu, Stefano Ermon, Atri Rudra, Christopher Ré** (Stanford University; University at Buffalo, SUNY) — [arXiv:2205.14135](https://arxiv.org/abs/2205.14135), submitted 27 May 2022 (v2 23 Jun 2022). *No venue is stated in the arXiv comments field.*

## Summary

The paper's central claim is that attention on GPUs is **memory-bound, not compute-bound**, and that the decade of "approximate attention" work aimed at the wrong target. Prior sparse/low-rank methods cut FLOPs to linear or near-linear in sequence length but "do not display wall-clock speedup against standard attention," because they "focus on FLOP reduction (which may not correlate with wall-clock speed) and tend to ignore overheads from memory access (IO)." The bottleneck is traffic between GPU high-bandwidth memory (HBM: 40–80 GB at 1.5–2.0 TB/s on an A100) and on-chip SRAM (192 KB per SM at ~19 TB/s). A standard attention implementation materializes the N×N score matrix `S = QKᵀ` and the probability matrix `P = softmax(S)` in HBM, so it performs Θ(Nd + N²) HBM accesses.

FlashAttention computes **exact attention** — it is bit-for-bit the same mathematical function, not an approximation, and the paper confirms GPT-2 trained with it "achieves the same perplexity as the other two implementations, as we do not change the model definition." It gets its speed from two classic techniques applied to the attention kernel: **tiling** (split Q, K, V into blocks, load blocks to SRAM, and run the softmax reduction incrementally using online rescaling by the running max and sum, so the N×N matrix is never written to HBM) and **recomputation** (store only the per-row softmax normalization statistics from the forward pass, and recompute the attention block on-chip in the backward pass rather than reading it back from HBM). Everything is fused into a single CUDA kernel. Notably, FlashAttention does **more** FLOPs than the baseline (66.6 → 75.2 GFLOPs on the GPT-2-medium setting, from backward-pass recomputation) and still runs ~5.7× faster on that kernel, because HBM traffic falls from 40.3 GB to 4.4 GB.

The IO analysis is the formal core: FlashAttention needs Θ(N²d²M⁻¹) HBM accesses versus Θ(Nd + N²) for standard attention, where M is SRAM size. Since d is 64–128 and M ≈ 100 KB, d² ≪ M and the reduction is large in practice (up to ~9× fewer accesses). The paper also proves a matching lower bound: no exact attention algorithm can do o(N²d²M⁻¹) HBM accesses for all M in [d, Nd] — i.e. FlashAttention is IO-optimal up to constants over a range of SRAM sizes. Memory footprint drops from quadratic to **linear in sequence length** (up to 20× more memory-efficient than exact-attention baselines), which is what unlocks the long-context results. A block-sparse variant reuses the same kernel structure for approximate attention with IO complexity scaled by the nonzero-block fraction s.

## Key points

- **Exact, not approximate.** Tiling + online softmax rescaling + backward recomputation reproduce standard attention exactly; no quality is traded. Block-sparse FlashAttention is the separate, opt-in approximate variant.
- **The win is IO, not FLOPs.** Fig. 2 (left), GPT-2 medium, seq. 1024, head dim 64, 16 heads, batch 64, A100 — forward+backward:

| | GFLOPs | HBM read/write (GB) | Runtime (ms) |
|---|---|---|---|
| Standard attention | 66.6 | 40.3 | 41.7 |
| FlashAttention | **75.2** (higher!) | **4.4** (~9× less) | **7.3** |

- **IO complexity:** standard Θ(Nd + N²) HBM accesses vs. FlashAttention Θ(N²d²M⁻¹) (Theorem 2). Lower bound (Proposition 3): no exact algorithm achieves o(N²d²M⁻¹) across all M ∈ [d, Nd]. Block-sparse: Θ(Nd + N²d²M⁻¹·s) for nonzero-block fraction s.
- **Memory scaling: quadratic → linear in sequence length.** Up to **20× more memory-efficient** than exact-attention baselines, and still 2× more efficient than Linformer at 64K, where every baseline except Linformer OOMs on a 40 GB A100.
- **Kernel speedup:** up to **7.6×** on the GPT-2 attention computation (Fig. 1 right); up to **3×** faster than the PyTorch attention implementation across seq. 128–2K, scaling to 64K. Approximate-attention methods only start to cross over with FlashAttention between seq. 512 and 1024.
- **End-to-end training speedups:** BERT-large (seq. 512) 20.0 ± 1.5 → **17.4 ± 1.4 minutes** on 8×A100, i.e. **15% faster than the MLPerf 1.1 record**; GPT-2 up to **3×** vs. HuggingFace and **1.7×** vs. Megatron-LM; long-range arena (seq. 1K–4K) **2.4×**.
- **Longer context for free:** GPT-2 small at 4K context with FlashAttention is still **30% faster** than Megatron-LM at 1K context, at **0.7 better perplexity** (18.2 → 17.5). +6.4 points on long-document classification (MIMIC-III, ECtHR).
- **New capabilities:** first Transformers above chance on **Path-X (seq. 16K, 61.4%)** and, with block-sparse FlashAttention, **Path-256 (seq. 64K, 63.1%)**. All prior Transformers OOM'd or scored at chance.
- **Block size matters:** larger blocks mean fewer passes and fewer HBM accesses, but beyond block size 256 runtime is bottlenecked by arithmetic and larger blocks stop fitting in SRAM.
- **Stated limitation:** each new attention variant requires hand-written CUDA, and kernels are not portable across GPU architectures — motivating the successor papers.

## Concepts & entities

- [[flash-attention]] — this is the origin paper: the IO-aware, tiled, recomputing exact-attention kernel.
- [[attention-mechanism]] — the operation being reimplemented; the mathematical definition is untouched, only its memory schedule changes.
- [[transformer-architecture]] — the target; attention is identified as the most memory-intensive component of the stack.
- [[long-context-llm]] — linear memory scaling is what enables 4K–64K contexts on a single 40 GB A100 and the first non-random Path-X / Path-256 results.
- [[state-space-model]] — the alternative research direction the paper implicitly answers: rather than replacing quadratic attention with a sub-quadratic architecture, make exact attention fast enough that long context is affordable.

## References

- [arXiv:2205.14135](https://arxiv.org/abs/2205.14135)
