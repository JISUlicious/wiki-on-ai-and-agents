---
title: "SOAP, Muon, and Beyond: Pushing LLM Pretraining Scales"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - soap-muon-and-beyond-khona-2026.md
arxiv_id: "2607.20548"
year: 2026
authors:
  - Mikail Khona
  - Aditya Vavre
  - Boxiang Wang
  - Bryan Catanzaro
  - Tijmen Blankevoort
tags:
  - 2026
  - paper
status: complete
importance: high
---

# SOAP, Muon, and Beyond: Pushing LLM Pretraining Scales

**Mikail Khona, Aditya Vavre, Boxiang Wang, Deyu Fu, Hao Wu, Mike Chrzanowski, Bryan Catanzaro, Dheevatsa Mudigere, Jeff Pool, Michael Lightstone, Mohammad Shoeybi, Mostofa Patwary, Nima Tajbakhsh, Tijmen Blankevoort** (NVIDIA) — [arXiv:2607.20548](https://arxiv.org/abs/2607.20548).

## Summary

An NVIDIA engineering report on taking two higher-order (preconditioned) optimizers — **SOAP** (Shampoo-with-Adam-in-the-eigenbasis) and **Muon** (momentum orthogonalized by Newton–Schulz) — from research curiosities to production-scale LLM pretraining. The framing is that the optimizer sits at the junction of algorithms and systems: it dictates memory and sharding strategy, and it determines how far batch size can scale before communication bottlenecks or instability bite. AdamW is trivially shardable because its state is element-wise, but it ignores the matrix/operator structure of weights and, empirically, hits a critical batch size beyond which it degrades.

The central experimental claim is a **large-batch** one. Across 8B dense GPT models, 30B-A3B and Qwen3-30B-A3B transformer MoEs, and a 72B-A8B hybrid Mamba–Transformer MoE, trained on 1T- and 3T-token Nemotron-3 subsets, both Muon and SOAP keep improving as global batch size scales to **100M tokens per batch**, while AdamW degrades. Comparisons use Kimi/Moonshot-style **update-RMS matching** so learning rates transfer fairly between optimizers rather than being confounded by tuning. MoE architectures motivate the large-batch push directly: with top-k routing over N experts, each expert sees only `B_global × k/N` tokens (0.03125 for top-8 of 256), so scaling global batch size mainly stresses the *dense* parameters' large-batch tolerance.

The paper's most transferable engineering finding is a diagnosed and fixed **SOAP instability**. The reference SOAP implementation refreshes the eigenbasis only every ~10 steps and excludes the current step's gradient from that recomputation. At large batch size this produces a "slingshot": oscillating gradient norms followed by loss spikes — recoverable at 621M parameters, **catastrophically divergent at 8B**. The fix is two-part: per-step QR eigenbasis updates *including* the current gradient (increasing frequency alone is insufficient), plus KL-divergence-based covariance estimation for the Kronecker factors, which provably square-roots the factors' condition number. Together these eliminate all loss spikes. Systems work rounds this out: standard ZeRO/FSDP sharding fragments the 2D matrices these optimizers need, so the authors built a **layer-wise distributed optimizer** in Megatron-LM that assigns whole layers to DP ranks, plus tensor-parallel "duplicated" and "distributed" Newton–Schulz modes that stay mathematically equivalent to the unsharded computation.

## Key points

**Optimizer state cost vs AdamW (the memory question)**

> [!warning] The paper gives **no explicit bytes-per-parameter or GB memory numbers.** Its memory claims are qualitative. The state counts below are read directly off the algorithm boxes (Alg. 1 for Muon, Alg. 2 for KL-SOAP) and the primer in Sec. 2, not from a memory table the paper provides.

For a weight matrix `W ∈ R^(in×out)`:

| Optimizer | Optimizer states held | Per-parameter states | Paper's own wording |
|---|---|---|---|
| AdamW | `m_t`, `v_t` (both element-wise, in×out) | 2 | "memory-regular… infinitely shardable"; "easiest optimizer to scale" |
| Muon | `M_t` (momentum buffer only, in×out) | 1 (no second moment) | "Muon has a **lower footprint than AdamW** since it does not maintain second moments" |
| SOAP / KL-SOAP | `m_t`, `v_t` (in×out) **plus** `L_t`, `Q_L` (in×in) and `R_t`, `Q_R` (out×out) | 2 + four square matrices (2·in² + 2·out² extra) | "must maintain and update Kronecker factors and eigenbases… **has a much larger memory footprint**"; Shampoo/SOAP "maintain full precision preconditioner and kronecker factor matrices requiring significantly more memory than AdamW" |

So the direction is: **Muon roughly halves AdamW's optimizer state** (one moment instead of two), while **SOAP adds AdamW's full state plus four square preconditioner/eigenbasis matrices on top**. The conclusion is explicit that this is the deciding tradeoff: *"in scenarios where memory footprint is not a limiting factor, we recommend KL-SOAP over Muon."*

Neither optimizer is free on the systems side, and the paper is clear that Muon's saving is not a pure win: it "still requires full 2D matrix updates which faces the same matrix-based sharding issues as SOAP", and it "uses more computation in the optimizer step", scaling with the number of Newton–Schulz iterations (16 here, PolarExpress coefficients).

**Instabilities found in SOAP, and their fixes**
- **Root cause**: reference SOAP refreshes the eigenbasis infrequently (~every 10 steps) *and* omits the current step's gradient from the recomputation. Under rapidly-changing early large-batch loss landscapes, the stale preconditioner "slingshots" the trajectory.
- **Symptom**: oscillating weight-gradient norms followed by LM loss spikes. Demonstrated on a **dense 621M transformer at 12.5M-token global batch** (oscillations eventually damped, model converges) and at **8B dense**, where the same lag causes **outright divergence** and much worse quality than AdamW.
- **Fix 1 — real-time eigenbasis**: per-step QR updates *and* including the current gradient. The paper stresses that raising update frequency alone is insufficient; it's the combination that resolves it. Result: loss spikes eliminated, LM loss on par with Muon.
- **Fix 2 — KL-Shampoo covariance**: replacing standard `GGᵀ`/`GᵀG` accumulation with KL-divergence-regularized covariance estimation. Suppresses residual spikes and lowers final training loss on Qwen3-30B-A3B. Mechanism: `κ(S_KL-Shampoo) = sqrt(κ(S_Shampoo))` — the condition number of the Kronecker factors is square-rooted, protecting eigenbasis recomputation from floating-point noise.
- **QR vs eigh**: negligible loss difference once QR runs every step, so QR is the default (cheaper).

**Convergence and quality results at scale**
- Models: 8B dense GPT; Nemotron-3-Nano-30B-A3B (hybrid attention–Mamba MoE, 128 experts, top-6); Nemotron-3-72B-A8B (512 experts, top-6, shared expert size 5376); Qwen3-30B-A3B (128 experts, top-8). Data: 1T- and 3T-token Nemotron-3 subsets. Baseline global batch 3072 × 8192 = **25M tokens**; WSD schedule with minus-sqrt decay, weight decay 0.1.
- Muon large-batch results transfer to MoE architectures up to **≈5×10²² pretraining FLOPs**.
- **KL-SOAP > Muon > AdamW** on loss: KL-SOAP holds a "consistent, albeit slight" CE-loss edge over Muon through most of the run (Qwen3-30B-A3B, 24M-token batch). MOP (Muon with exact polar decomposition via SVD instead of Newton–Schulz) also lands slightly below Muon.
- Downstream (Table 5, Nemotron-Nano-V3, 3T tokens): Muon 2× batch vs AdamW 1× — MMLU **74.80 vs 73.38**, MMLU PRO CoT **56.77 vs 54.45**, Coding avg **63.70 vs 62.22**, Commonsense avg **82.28 vs 81.89**, Math avg **81.57 vs 79.65**. Muon 3× also beats AdamW 1× on MMLU (74.00) and Math avg (81.47).
- Downstream (8B Hybrid MTP, 1T tokens): Muon 2× vs AdamW 1× — MMLU **74.89 vs 74.59**, MMLU PRO CoT **59.60 vs 57.52**, Coding avg **63.48 vs 58.97** (HumanEval 65.79 vs 57.62), Commonsense avg **83.31 vs 82.66**. Math is the one regression: **80.79 vs 84.04**. The authors state gains are most pronounced in coding and commonsense reasoning.
- Batch ramp-up matters: starting at 256 samples (2M tokens) and doubling every S steps over a fixed horizon, with LR warmup tied to the ramp so `η ∝ sqrt(B)` (square-root scaling rule, chosen over linear scaling as safer for very large batches). Applied only to the large-batch runs. AdamW does *not* benefit from the same techniques (Appendix C).
- Update-RMS matching: SOAP and AdamW share identical update RMS by construction (rotations preserve Frobenius norm); Muon needs a correction factor `sqrt((1−β₁)/(1+β₁)) ≈ 0.2`. Nesterov momentum did not help and was dropped for both.

**Systems / implementation**
- ZeRO-1-style sharding is "not directly appropriate" for matrix-based optimizers: if weights and states are split evenly across DP ranks, no rank can compute a full update without extra communication.
- **Layer-wise distributed optimizer** (in Megatron-LM): whole parameter matrices sorted by size and round-robined across GPUs; each GPU updates only its own layers; parameter sync uses bucketed asynchronous **allgather-V** (variable-sized) aligned to model execution order, hiding latency behind the forward pass and avoiding padding. No approximations to the optimizer math, so convergence benefits are retained.
- Tensor-parallel Newton–Schulz has two exact modes: **duplicated** (all-gather weights inside the TP domain, each GPU runs NS locally — better when communication-bound) and **distributed** (all-reduce the intermediate of the first matmul each NS iteration — better when compute-bound). Both normalize using whole-layer statistics for mathematical equivalence to the non-TP case.
- Open source: `github.com/NVIDIA-NeMo/Emerging-Optimizers` and `megatron/core/optimizer/layer_wise_optimizer.py`.

**Precision and architecture interactions**
- MXFP8 pretraining (Blackwell recipe) with Muon on Nemotron-NanoV3 and the 8B hybrid, 1T tokens: small loss gap vs BF16 but *most downstream evals are higher* with MXFP8. Swapping AdamW for Lion (β = 0.95, 0.98) on the non-Muon parameters improved both loss and evals.
- Numerical noise floor: the optimizer step runs in FP32 (2⁻²³ ≈ 1.19×10⁻⁷) but Muon's Newton–Schulz iterations run in **BF16** (2⁻⁷ = 0.0078125), so the heavy tail of small singular values is largely noise — flagged as an open problem as matrices grow.
- **Architectural mismatch**: applying Muon to Mamba2's Conv1D filters degrades accuracy and sometimes produces NaNs; falling back to AdamW for Conv1D gives ≈0.1% lower loss. Orthogonality is "geometrically ill-motivated" for shared temporal filters. Muon should be applied selectively to dense linear projections.
- Splitting the fused QKV tensor before orthogonalization helps early in training but the gap vanishes by the end of a 1T-token horizon.

**Stated limitations**
- ε was not tuned systematically (10⁻⁸ for SOAP/AdamW, 10⁻⁷ for Muon), and ε plays structurally different roles in the two optimizers (second-moment floor vs Frobenius-norm floor).
- Batch-size ramp-up was not tuned as a function of model scale.
- No batch-size scaling law is derived; batch size is treated as a systems-level constraint (maximize GPU utilization by minimizing DP communication).
- SOAP still lacks native tensor-parallel support and fused-tensor (QKV-split) handling in the layer-wise optimizer.

## Relation to Adam's memory footprint

[[training-memory-anatomy]] records the standard mixed-precision accounting where Adam's optimizer states are 12 of 16 bytes per parameter — 75% of model-state memory. This paper is the scaled-up empirical case that those 12 bytes are not fixed: **Muon drops the second moment entirely**, removing one of the two full-size EMA buffers, and does so while *improving* loss and large-batch tolerance rather than trading quality for memory (contrast [[8-bit-optimizers-dettmers-2021]], which shrinks the same states by quantizing them instead of eliminating one). SOAP moves the other way, spending strictly more memory than AdamW for the best measured loss. But the paper is equally clear that the memory ledger is not the whole systems story: both optimizers need *unfragmented 2D matrices*, which breaks the ZeRO/FSDP assumption that optimizer state can be sliced arbitrarily — see [[zero-redundancy-optimizer]] and [[distributed-training]]. The layer-wise optimizer is the price paid for that.

## Concepts & entities
- [[training-memory-anatomy]] — the optimizer-state share of training memory is exactly what Muon (1 state) and SOAP (2 states + 4 square matrices) move in opposite directions.
- [[zero-redundancy-optimizer]] — ZeRO-1-style even sharding is stated to be unsuitable here; the layer-wise optimizer replaces it for matrix-based methods.
- [[distributed-training]] — tensor-parallel Newton–Schulz modes, bucketed allgather-V, and Megatron-LM integration.
- [[mixed-precision-training]] — MXFP8 pretraining results and the FP32-step / BF16-Newton–Schulz numerical noise floor.
- [[mixture-of-experts]] — MoE routing makes each expert see `B_global × k/N` tokens, which is the specific reason large-batch tolerance of the *dense* parameters is the binding constraint.
- [[pre-training]] — the entire study is pretraining-stage, 1T–3T token horizons.
- [[backpropagation]] — the preconditioners operate on the gradients backprop produces; the distinction is element-wise vs matrix-structured use of them.
- [[scaling-laws-for-precision-kumar-2024]] — adjacent precision-aware training work; here MXFP8 is evaluated empirically rather than modeled.
- [[scaling-laws]] — the authors explicitly decline to derive a batch-size scaling law, treating batch size as a systems constraint instead, and flag combining the two views as future work.

## References
- [arXiv:2607.20548](https://arxiv.org/abs/2607.20548)
- Code: github.com/NVIDIA-NeMo/Emerging-Optimizers; Megatron-LM `layer_wise_optimizer.py`
- Vyas et al. (2024) — SOAP: Improving and stabilizing Shampoo using Adam ([arXiv:2409.11321](https://arxiv.org/abs/2409.11321))
- Gupta et al. — Shampoo; Eschenhagen et al. (2025) — Purifying Shampoo ([arXiv:2506.03595](https://arxiv.org/abs/2506.03595)); KL-Shampoo — the covariance estimator adopted here
- Jordan et al. — Muon; PolarExpress — the Newton–Schulz coefficient recipe used (16 iterations)
- Kimi/Moonshot — the update-RMS matching framework used for fair LR transfer
