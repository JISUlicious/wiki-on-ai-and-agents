---
title: FlashAttention
type: concept
created: 2026-07-27
updated: 2026-07-27
sources:
  - flash-attention-dao-2022.md
  - flash-attention-2-dao-2023.md
  - flash-attention-3-shah-2024.md
builds-on:
  - [[attention-mechanism]]
status: complete
importance: high
---

# FlashAttention

**FlashAttention** is an **IO-aware exact attention** algorithm: it computes the same result as standard attention — no approximation, no sparsity — but reorganizes the computation to avoid ever materializing the N×N attention matrix in GPU high-bandwidth memory (HBM). It is the single most widely deployed attention optimization, and the reason long-context [[long-context-llm|inference]] is practical at all.

## The counterintuitive core

FlashAttention does **more arithmetic, and is dramatically faster.** On GPT-2-medium ([[flash-attention-dao-2022|Dao et al. 2022]], A100, seq 1024):

| | Standard | FlashAttention |
|---|---|---|
| FLOPs | 66.6 GFLOPs | **75.2 GFLOPs** (more!) |
| HBM traffic | 40.3 GB | **4.4 GB** (~9× less) |
| Runtime | 41.7 ms | **7.3 ms** (5.7× faster) |

The extra FLOPs come from **recomputing** attention in the backward pass rather than storing it. The win is entirely **memory traffic**: tiling the computation so blocks stay in fast on-chip SRAM, with online softmax so no full row must exist at once.

> [!important] This refines the prefill story
> At the *roofline/system* level prefill is compute-bound ([[llm-inference-roofline-yuan-2024]] measures arithmetic intensity 1,024–1,215 for prefill projections). But the **attention kernel specifically** is memory-bound, because standard attention materializes an N×N matrix. FlashAttention removes that bottleneck, which is why it speeds up prefill even though prefill is "compute-bound" overall. Both statements are true at different levels of the stack.

Memory also goes **quadratic → linear** in sequence length (up to 20× more memory-efficient), which is what makes long contexts fit. The paper proves an IO lower bound showing no exact algorithm can beat it asymptotically.

## The three generations

| | Contribution | Headline |
|---|---|---|
| **[[flash-attention-dao-2022\|FlashAttention]]** (2022) | tiling + recomputation; IO-aware | 7.6× on the attention kernel; 15% over the MLPerf 1.1 BERT-large record; first Transformer above chance on Path-X (16K) |
| **[[flash-attention-2-dao-2023\|FlashAttention-2]]** (2023) | better work partitioning | **1.7–3.0× over v1**; **230 TFLOPs/s = 73%** of A100 peak; GPT3-2.7B at 8k reaches 72% MFU |
| **[[flash-attention-3-shah-2024\|FlashAttention-3]]** (2024) | Hopper asynchrony (TMA/WGMMA) + FP8 | **1.5–2.0× over v2**, up to **740 TFLOPs/s (75%)**; FP8 near **1.2 PFLOPs/s** |

**v2's insight** is that non-matmul FLOPs are ~16× more expensive than matmul on an A100 (312 vs 19.5 TFLOPs/s), so minimizing them — plus parallelizing over *sequence length*, not just batch×heads — closes the gap to GEMM-level utilization. Causal masking lets it skip ~half the blocks for a further 1.7–1.8×.

**v3** exists because v2 achieved only **35% utilization on H100** — the Hopper architecture's asynchronous units sit idle unless explicitly targeted. It overlaps softmax with async block-wise GEMMs to break the sequential dependency. Its FP8 path uses block quantization plus **incoherent processing** (a Hadamard transform, the same [[quarot-ashkboos-2024|QuaRot]] trick) for **2.6× lower RMSE**, and the ablation shows incoherent processing carries nearly all of that gain.

> [!note] v3 explicitly does not optimize inference
> The FlashAttention-3 paper states LLM *inference* is not yet an optimization target. Its numbers are training/forward-pass benchmarks.

## Related

- [[attention-mechanism]] — the operation being optimized.
- [[grouped-query-attention]] — complementary: GQA shrinks the KV cache (a decode win), FlashAttention cuts attention memory traffic. FA2 implements GQA by index manipulation rather than materializing duplicated KV heads.
- [[linear-attention]] — the *structural* alternative: never incur the quadratic cost at all.
- [[minimax-sparse-attention-lai-2026]] — the *approximate* alternative: skip blocks entirely rather than compute all of them efficiently.
- [[quantization-performance]] — where attention cost sits in the prefill/decode picture.

## References

- [[flash-attention-dao-2022]] · [[flash-attention-2-dao-2023]] · [[flash-attention-3-shah-2024]]

## Training systems

- [[activation-recomputation]] — attacks the same quadratic activation term from the kernel side rather than by recomputing
