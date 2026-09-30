---
title: "Normalized Low-Rank Adaptation"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.31036"
year: 2026
authors:
  - Jiale Kang
  - Ziyin Yue
  - Zheng Zhan
  - Yangyi Huang
  - Weiyang Liu
tags:
  - 2026
  - paper
  - peft
status: complete
importance: medium
---

# Normalized Low-Rank Adaptation (NoRA)

**Jiale Kang, Ziyin Yue, Zheng Zhan, Yangyi Huang, Weiyang Liu** (Yuanshi Intelligence; [[chinese-university-of-hong-kong]]; [[microsoft-research]]; Shenzhen Loop Area Institute) — [arXiv:2608.31036](https://arxiv.org/abs/2608.31036), cs.LG, 31 Aug 2026 (v1). Project page: spherelab.ai/NoRA. Preprint; no venue given.

## Summary

The whole paper turns on one under-noticed asymmetry in [[lora]]. Because the up-projection **B** is initialized to zero, `∂L/∂A|_{t=0} = 0` — the down-projection **A** receives *no gradient at all* at the start of training, while `∂L/∂B|_{t=0} = αG(A⁽⁰⁾)ᵀ`. So the entire early optimization of LoRA is determined by a matrix nobody tunes: the random **A**. The authors ask what happens if you regularize it.

Their answer comes from a **preconditioning** reading of LoRA. Take one gradient step on **B** at initialization and merge; the induced weight change is

> ΔW = αΔB·A = −ηG**P**, where **P** = α²AᵀA ∈ ℝ^{k×k}

Full finetuning is the same expression with **P** = I. So LoRA, for as long as **B** stays small, *is* full finetuning with the gradient right-multiplied by a rank-r positive semidefinite matrix acting on the **input coordinates** — exactly where curvature methods put an input-side preconditioner. Expanding column-wise makes the failure legible:

> ΔW_{:,j} = −η[ α²‖a_j‖₂² G_{:,j} + Σ_{i≠j} α²(a_iᵀa_j) G_{:,i} ]

The first term is coordinate *j*'s own update scaled by the **squared length of its latent direction**; the second is crosstalk from the rank bottleneck. **The column norms of A are per-input-coordinate learning rates.** With zero-mean i.i.d. entries of variance σ² ∝ 1/k, E‖a_j‖₂² = rσ² ∝ r/k ≪ 1 — so at any moderate α *every* per-coordinate learning rate is far too small, and "gradient flows through the adapter at a small fraction of the full-finetuning rate." Worse, ‖a_j‖₂² fluctuates with relative spread of order **1/√r**, so at small rank "each coordinate gets its learning rate randomly, unrelated to data or curvature." The authors' verdict: *"A preconditioner should remove distortion, not add its own."*

**NoRA** is the one-line fix: normalize each column of **A ∈ ℝ^{r×k}** to unit L2 norm **along the rank dimension**, i.e. Δy = αB·Norm(A)·x with Norm(A)_{:,j} = a_j / max(‖a_j‖₂, ε). At α = 1 (equivalently α = r under the α/r convention) this makes **Diag(P) = I deterministically** and E[P] = I over random directions, so the adapter's gradient norm matches full finetuning's in expectation, **independently of r**. Crucially the normalization depends only on the parameters, not the input, so unlike MLA-style latent normalization `B·Norm(Ax)` — which is nonlinear in **x** and cannot be merged — NoRA stays linear in **x** and **preserves exact weight mergeability**. No extra trainable parameters, no inference-time cost.

Because **A**'s gradient αBᵀG vanishes with **B**, fixing the learning rates *once* at initialization captures most of the benefit. Hence **NoRA-init**: normalize **A** at step 0, then train as vanilla LoRA. Keeping the constraint on throughout training adds two things — scale invariance of the loss in each column, so gradients are tangential and the effective step anneals automatically as in weight-normalized training; and a guarantee that Diag(P) = I for the whole run, so the latent scale can neither collapse nor explode.

## The genealogy: MLA → NoRA → MiSS

The idea is transplanted from [[multi-head-latent-attention|Multi-head Latent Attention]], whose normalized latent bottleneck `φ(x) = Norm(Ax)` stabilizes training. The paper's unifying frame is that every low-rank method just designs the latent feature presented to the up-projection: `Ax` (random projection, LoRA), `A_init·x` (structured projection, PiSSA/MiLoRA), `Norm(Ax)` (normalized projection, MLA). NoRA moves the normalization *from the feature to the projection matrix*.

A neat corollary falls out. **MiSS** — a matrix-shard-sharing PEFT method — expands to `αB[I_r, …, I_r]x`, i.e. a LoRA update with a **fixed** down-projection of repeated identity blocks, whose columns are unit-norm **by construction**. MiSS is therefore a special case of NoRA with **A** frozen. That in turn suggests **BIMI (Block Identity Matrix Initialization)**: use the block-identity matrix `[I_r, …, I_r, E_q]` only as an *initializer* and keep **A** fully trainable. BIMI satisfies the normalization requirement deterministically, with a completely different crosstalk pattern from random unit columns — and it performs the same, which the authors read as evidence that **the diagonal of P is the decisive quantity**, not the off-diagonal structure.

## Results

All comparisons in the headline SFT table are **at matched rank r = 32 and essentially matched trainable-parameter count** (48.6M for LoRA, PiSSA, RSLoRA, NoRA-init and NoRA; 49.4M for DoRA and its NoRA variants; 49.5M MiSS; 53.7M OFT). One caveat: α is *tuned per method* ("we tune the hyperparameters of each method individually"), so LoRA and DoRA run α = 64 while NoRA runs α = 32 — the α : r = 1 : 1 setting the theory prescribes. It is matched-rank, not matched-α.

### Supervised finetuning — Llama-3.2-3B, MetaMath + CodeFeedback

| Method | Trainable | GSM8K | Math | HumanEval | MBPP | **Avg** | Retention Avg Δ |
|---|---|---|---|---|---|---|---|
| Full finetuning | 3B | **65.12** | **17.96** | 36.15 | 49.05 | 42.07 | +0.02 |
| LoRA | 48.6M | 50.94 | 10.38 | 37.20 | 53.20 | 37.93 | −0.56 |
| **NoRA-init** | 48.6M | 59.89 | 14.24 | 39.60 | 55.80 | 42.38 | −0.19 |
| **NoRA** | 48.6M | 61.63 | 14.46 | **42.10** | 55.30 | **43.37** | **+0.02** |
| PiSSA | 48.6M | 54.66 | 12.08 | 39.00 | 55.30 | 40.26 | −0.28 |
| RSLoRA | 48.6M | 57.05 | 12.32 | 39.65 | **56.10** | 41.28 | −0.51 |
| OFT | 53.7M | 56.10 | 13.02 | 38.40 | 54.20 | 40.43 | **+0.52** |
| MiSS | 49.5M | 60.80 | 14.60 | 39.60 | 55.80 | 42.70 | −0.70 |
| DoRA | 49.4M | 51.63 | 11.36 | 37.80 | 52.40 | 38.30 | −0.64 |
| DoRA + NoRA-init | 49.4M | 59.59 | 14.10 | 38.70 | 53.20 | 41.40 | −0.40 |
| DoRA + NoRA | 49.4M | 61.33 | 14.44 | 42.10 | 55.60 | 43.34 | −0.43 |

**The measured gain at matched rank is +5.44 points of SFT average (37.93 → 43.37)**, of which **+4.45 comes from initialization alone** (NoRA-init at 42.38) and the remaining ~1 point from maintaining the constraint through training. Both numbers verify against the table. The single largest per-task movement is GSM8K, 50.94 → 61.63 (**+10.69**).

Two things the paper states less prominently:
- **NoRA's SFT average (43.37) exceeds full finetuning's (42.07)** with 1.6% of the trainable parameters — but the split matters: full FT wins decisively on math (GSM8K 65.12, Math 17.96) and loses on code (HumanEval 36.15, MBPP 49.05). This is a task-mix artifact as much as a method result.
- The principle **transfers to DoRA** (38.30 → 43.34), which is the strongest evidence that rank-dimension normalization is a general property of low-rank parameterizations rather than a LoRA-specific trick.
- On forgetting, the paper compares NoRA's +0.02 only against LoRA (−0.56) and MiSS (−0.70). **OFT retains better still at +0.52** and goes unmentioned.

### Ablation: which dimension you normalize is the whole result

SFT, average of GSM8K and Math accuracies. `Norm_k` normalizes each **row**; `Norm_r` normalizes each **column** (the rank dimension) — the latter is NoRA.

| Initialization of A | None | Norm_k (rows) | **Norm_r (rank dim)** |
|---|---|---|---|
| 𝒰(−1/√k, 1/√k) | 29.27 | 29.75 | **37.16** |
| 𝒩(0, 1/r²) | 31.05 | 30.54 | **36.95** |
| 𝒰(−1, 1) | 29.61 | 29.58 | **36.66** |
| BIMI (block identity) | — | — | **37.07** |

This is the cleanest table in the paper. **Row normalization does nothing** (±0.5, and it *hurts* under Gaussian init) — consistent with the theory, since Norm_k leaves Diag(P) random of order r/k. **Column normalization is worth +6 to +8 points** and, once applied, **erases the difference between initialization distributions entirely** (36.66–37.16 across three very different schemes, plus a deterministic block-identity construction at 37.07). The choice of random distribution stops mattering; only the column norms did.

### Pretraining — 340M models, 10B tokens, 0.5M-token batches

| Architecture | Parameterization | Params | LAMBADA ppl ↓ | LAMBADA acc ↑ | **Avg** |
|---|---|---|---|---|---|
| MLA | B·Norm(Ax) (latent norm, unmergeable) | 342.4M | **48.73** | **30.62** | **41.17** |
| MLA | B·A·x (standard low-rank) | 342.4M | 61.83 | 28.70 | 40.36 |
| MLA | B·A_NoRA-init·x | 342.4M | 50.77 | 29.87 | 40.64 |
| MHA | W·x (full, dense) | 348.7M | 47.95 | 30.55 | 41.06 |
| MHA | B·A·x (r = 64, all attn projections) | 289.3M | — *(collapse)* | **0.00** | 27.81 |
| MHA | B·A_NoRA-init·x (r = 64) | 289.3M | 63.45 | 28.29 | 39.69 |

Two findings. On MLA, **NoRA-init recovers most of the perplexity benefit of true latent normalization** (61.83 → 50.77 against 48.73 for the unmergeable version) while keeping the linear `BAx` form and therefore mergeability. On MHA the result is more dramatic: the standard low-rank parameterization **collapses outright** — LAMBADA accuracy 0.00, perplexity unreportable, average 27.81 (near-chance) — with gradient norms diminishing "to abnormally low levels during training." NoRA-init recovers it to 39.69, close to the 41.06 of the dense baseline. Since capacity is identical between the two low-rank rows, this is a clean demonstration that **the collapse is an optimization-geometry failure, not a rank-capacity failure**.

### RLVR — DeepSeek-R1-Distill-Qwen-1.5B, DAPO objective, DAPO-Math-17K, rank 32

| Method | AIME24@32 | AIME25@32 | AMC@32 | HMMT@32 | MATH500@4 | Minerva@4 | **Avg** |
|---|---|---|---|---|---|---|---|
| Base | 24.3 | 20.4 | 64.8 | 9.9 | 80.9 | 27.8 | 41.0 |
| LoRA | **28.0** | 23.1 | 68.0 | 10.3 | 81.3 | 30.1 | 42.8 |
| **NoRA** | 26.7 | **23.5** | **72.7** | 10.3 | **84.5** | **32.0** | **44.4** |
| MiLoRA | 4.2 | 0.0 | 19.6 | 0.0 | 44.5 | 11.7 | 18.0 |
| PiSSA | 0.0 | 0.0 | 0.0 | 0.0 | 0.6 | 0.1 | **0.2** |

NoRA is **+1.6 over LoRA and +3.4 over base** — a much smaller margin than in SFT, and it does *not* win everywhere (LoRA is ahead on [[aime|AIME]]24 average, and HMMT is a tie). The dramatic number in this table belongs to someone else: **the SVD-based spectral initializations detonate under RL** — PiSSA falls to 0.2, MiLoRA to 18.0, both far below the untrained base model. Since PiSSA is second-best-ish under SFT (40.26), this is a strong regime-dependence result and the practical case for NoRA: it needs no SVD of the pretrained weights, so it has nothing to be fragile about.

## Limitations and things that did not fully check out

- **Two contradictory statements of the pretraining corpus.** Table 2 and the Table 4 caption say **SlimPajama**; Appendix A says **FineWeb-10BT**. Both appear in the same v1. One of them is wrong and the paper does not reconcile them.
- **A typo inverts the ablation's own conclusion.** §4.1 reads "Norm_k provides little improvement, whereas Norm_k consistently improves Random Uniform, Gaussian, and Kaiming Uniform initializations." The second must be Norm_r; Table 3 makes the intent unambiguous.
- **α is not held fixed** across the compared methods, so the SFT comparison is matched-rank and matched-parameter-count but not matched-hyperparameter. The authors are transparent that they tuned each method individually, and their α = r recommendation is theoretically motivated (it is the setting that makes the gradient norm match full finetuning), but a reader wanting to attribute the +5.44 purely to normalization should note that LoRA ran at α/r = 2 and NoRA at α/r = 1.
- **The forgetting comparison is selectively reported** — OFT beats NoRA on retention and is not mentioned in that sentence.
- **No cost or wall-clock accounting.** The claim is that NoRA adds no trainable parameters and no inference-time computation, which is true by construction, but per-step training overhead from the forward-pass normalization is never measured.
- **Single seed.** Appendix A fixes seed 42 for pretraining "for controlled comparison"; no variance is reported anywhere, and several of the SFT deltas between NoRA-init, MiSS and NoRA are ~0.7 points.
- **Scale.** The largest model touched is 3B (SFT); pretraining is at 340M and RL at 1.5B. Whether the r/k-scaled learning-rate deficit behaves the same at 70B+ is untested.
- The authors note the effect is "related to" the capacity-limited view of LoRA but argue their MHA result shows **capacity alone does not determine optimization behavior** — same rank, opposite outcome, purely from initialization.

## Related

- [[lora]] — the object of study. The contribution is best filed as a correction to LoRA's *initialization*, not a new adapter family: NoRA-init is three lines of change and captures 4.45 of the 5.44 points.
- [[fine-tuning]] — the SFT result at 1.6% of trainable parameters exceeding full-finetuning average (on this task mix) is the headline PEFT claim.
- [[qlora-dettmers-2023]] — the other major "make LoRA practical" intervention, attacking memory rather than optimization geometry; the two are orthogonal and composable (NoRA touches only the initialization/forward form of **A**).
- [[understanding-lora-as-knowledge-memory-back-2026]] — a complementary account of what LoRA adapters *represent*; NoRA is about how they *train*.
- [[layer-normalization]] — the normalization principle being transplanted, and the reason the argument works: pre-normalized transformers already control ‖x‖, so residual scale imbalance sits in **A**'s columns rather than in the input.
- [[multi-head-attention]] — MLA's normalized latent bottleneck is the source of the idea; the paper's move is to keep MLA's numerical effect while giving up its input-dependence to regain mergeability.
- [[grpo]] / [[agentic-rl]] / [[reinforcement-learning]] — the RLVR experiments run the DAPO objective; the practically important finding for anyone doing RL with adapters is that **PiSSA and MiLoRA collapse under RL** while plain LoRA and NoRA do not.
- [[verifier]] / [[outcome-based-reward]] — RLVR is the reward setting used; the math benchmarks are the verifiable-reward tradition.
- [[deepseek-r1]] / [[deepseek-r1-2025]] — the RL base model is DeepSeek-R1-Distill-Qwen-1.5B.
- [[llama]] — Llama-3.2-3B is the SFT base.
- [[humaneval]] / [[mmlu]] — two of the evaluation suites; MMLU is used on the forgetting side, not the capability side.
- [[continual-learning]] / [[catastrophic-remembering]] — the retention half of Table 5 is a forgetting measurement (MMLU, AGIEval, ARC-C before vs. after adaptation).
- [[pre-training]] — the 340M MHA collapse is the paper's most striking single result and lives in the pretraining setting, not the adapter setting.
- [[chinese-university-of-hong-kong]] / [[microsoft-research]] — the affiliations with existing pages.

**Concept gaps noted, not created:** `multi-head-latent-attention` (MLA — cited throughout, no page, despite being the DeepSeek architecture the wiki covers elsewhere), `peft` (used as a tag here; the wiki has [[lora]] and [[qlora-dettmers-2023]] but no umbrella concept), `pissa` / `dora` / `miss` / `rslora` / `oft` (the LoRA-variant family this paper benchmarks against), `rlvr` (distinct from [[outcome-based-reward]] and used by several 2026 sources), `preconditioning`, `weight-normalization`, `gsm8k` and `aime` (benchmarks referenced across the wiki without pages).

## References

- [arXiv:2608.31036](https://arxiv.org/abs/2608.31036) — Kang, Yue, Zhan, Huang, Liu. *Normalized Low-Rank Adaptation.* cs.LG, 31 Aug 2026. Project page: spherelab.ai/NoRA.
- Hu et al., *LoRA: Low-Rank Adaptation of Large Language Models*, arXiv:2106.09685 — the baseline being corrected.
- Meng et al., *PiSSA*, 2024; Wang et al., *MiLoRA*, 2025 — the SVD-based initializations that win under SFT and collapse under RL.
- Kang and Yin, *MiSS*, 2026 — shown here to be a special case of NoRA with a frozen block-identity down-projection.
