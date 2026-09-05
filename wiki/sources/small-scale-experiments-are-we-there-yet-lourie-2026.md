---
title: "Small-Scale Experiments: Are We There Yet?"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.11859"
year: 2026
authors:
  - Nicholas Lourie
  - Kyunghyun Cho
  - Karen Ullrich
  - Sanae Lotfi
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Small-Scale Experiments: Are We There Yet?

**Nicholas Lourie, Kyunghyun Cho, Karen Ullrich, Sanae Lotfi** ([[fair|FAIR]] at MSL [[meta|Meta]] / New York University) — [arXiv:2608.11859](https://arxiv.org/abs/2608.11859), cs.LG, 12 Aug 2026 (v1). 29 pages, 17 figures. No venue listed.

## Summary

[[scaling-laws|Scaling laws]] were supposed to make pretraining research cheap — study an idea at small scale, extrapolate to large. Six years after Kaplan, the paper argues, that promise is unfulfilled: experiments (not final runs) dominate model-development cost, and the field has concluded that laws fitted below ~100M parameters are unreliable (Li et al. 2025c) and that the [[scaling-laws-kaplan-2020|Kaplan]]/[[chinchilla-hoffmann-2022|Chinchilla]] discrepancy itself traces to methodological choices that disproportionately affect small models (Porian et al. 2024; Pearce & Song 2024). The authors' claim is that the confound is **hyperparameters, not scale**: laws are present down to **4M effective parameters** — where a run takes under an hour on one GPU — but they only appear on the **fully tuned frontier**, and small models are so hyperparameter-sensitive that reaching that frontier takes **hundreds of random-search configurations per scale**, far more than the small grids most labs sweep.

The mechanism is geometric. Fitting the noisy-quadratic limit (Lourie et al. 2025a) to random-search results shows that as parameters grow, the **intrinsic dimension of the hyperparameter loss surface (γ) falls to 1** and near-optimal configurations fill a growing fraction of the search space. Parameters, not data, drive the drop. So the small scale is high-dimensional and unforgiving (search exhaustively), while the large scale is low-dimensional and forgiving (carry hyperparameters up with simple rules). The paper closes with a four-diagnostic methodology for *model-centric* research and demonstrates it on pre-norm vs. post-norm, recovering from 4M–134M models the result that pre-norm scales better — a question that historically took ~3 years and a theoretical analysis (Xiong et al. 2020) to settle.

The honest caveat, which the authors state repeatedly: **laws exist at small scale, but extrapolating them far from the data does not**. Independent 128-run estimates agree near the data and diverge wildly in their irreducible-error term; comparing two architectures by extrapolated final loss is comparing two noisy estimates of ε.

## Setup

- **Models.** Decoder-only Llama variant in Meta Lingua (RMSNorm, SwiGLU, RoPE), context 1,024, `p50k_base` tokenizer padded to a 50,536 vocabulary. Scale is measured in *effective* parameters `N_eff = (3d_ff + 4d)·d·l + d·v + n·d·l` (unembedding and attention FLOPs counted, embedding excluded) so that `c = 6pd` is exact. Targets are powers of two from `2^22 ≈ 4M` to `2^28 ≈ 268M`.
- **Data.** 100B-token subset of FineWeb-Edu; every run sees the same data order (fixed seeds), so runs differ only in hyperparameters.
- **Training.** AdamW with a warmup-stable-decay schedule; each configuration trains to **32 tokens per effective parameter** (~50% past the Chinchilla-optimal ~20), with checkpoints at 1/8…8/8 of training; each checkpoint is branched into a linear cooldown that extends training by 25% (i.e. decays over the last 20%). One sweep therefore yields stable *and* decayed losses at eight token budgets. Metric: bits-per-character on held-out FineWeb-Edu.
- **Search.** Random search, not grid, over seven non-architectural hyperparameters — batch size (64–4,096), learning rate (log-uniform 1e-5–1e-1), β₁, β₂, warmup fraction (up to 1/8), weight decay (1e-4–1), RoPE θ (2¹⁰–2²⁰) — plus depth and head count in the random-architecture experiment (9 nominal hyperparameters). No μP; standard parametrization throughout.
- **Budget.** Ladder experiment: 867 / 826 / 867 / 934 configurations at 4M / 8M / 17M / 34M, 128 at 67M and 134M, 64 at 268M. Random-architecture experiment: ~128 per scale from 4M to 134M. Scales are partitioned into **train (4M–34M) / validation (67M–134M) / test (268M)**, so every reported fit is an extrapolation to held-out larger models. The two shortest token budgets are dropped before fitting (they fit poorly, as in Hilton 2023 / Li 2025c).
- **Hardware.** 80GB A100s; 1 GPU for 4M–134M, 8 GPUs data-parallel for 268M.
- **Functional form.** Additive Chinchilla, `L(p,d) = ε + ζ/p^ι + η/d^κ`, fit to the best loss at each (parameter, data) budget by differential evolution on MSE.

## Key results

**Methodology refinements help; they do not make or break the law (§3.1, Fig. 3–4).** Six ways of counting parameters (attention ± embedding ± unembedding) all give a reasonable law. Starting from "fit the best run at each scale, no LR decay":

| Refinement | Effect on test MSE |
|---|---|
| Tune hyperparameters per (parameter, data) budget rather than per scale | **−50%** |
| Decay the learning rate | **−98%** |
| Tie the exponents (ι = κ) | inconsistent — slightly better test, worse validation |

**Tuning effort is the ingredient that makes or breaks it (§3.2, Fig. 5).** Refitting with sub-sampled sweeps, evaluated on the best configurations from the full sweep:

| Configurations per scale | Law |
|---|---|
| 4 | completely absent |
| 16 | not yet visible; suboptimal configs inject too much noise |
| 64 | clearly visible, extrapolation weak |
| **256** | **accurate** |

**Sensitivity fades with scale (§4.1, Fig. 6–7).** The gap between the 25th-percentile and minimum validation loss under random search — a direct measure of how much of the space is near-optimal — barely moves when only parameters *or* only data are scaled, and **collapses when both grow together**.

**The surface gets lower-dimensional (§4.2, Fig. 8–9).** The effective hyperparameter count γ from the noisy-quadratic fit **drops to 1 as models scale**; the trend holds with and without LR decay; checkpoints across training share roughly the same γ, so parameters rather than data are the driver. Restricted γ ranges were 1–7 (ladder) and 1–9 (random). The authors flag that γ was previously found *architecture*-independent for a fixed search space, and that μP-style parametrizations might change the scale dependence — unresolved.

**Interpretable surface models fall short (App. B, Table 2).** Combining the law with a low-rank (γ ≤ 3) quadratic in the hyperparameters and extrapolating to held-out scales, MSE on all configurations in the asymptotic regime:

| Model | 67M | 134M | 268M |
|---|---|---|---|
| Scaling law (optimum only, reference) | 0.000011 | 0.000025 | 0.000020 |
| Parameters & tokens only | 0.011471 | 0.014014 | 0.019795 |
| Static quadratic (scale-independent optimum & curvature) | 0.004820 | 0.005900 | 0.010027 |
| Dynamic optima (optimum drifts linearly in log-scale) | **0.002824** | **0.004401** | **0.006630** |
| Dynamic sensitivity (curvature varies) | 0.005413 | 0.007185 | 0.010813 |
| MLP on quadratic features | 0.008866 | 0.012003 | 0.015464 |

The scale-agnostic static model is already a strong baseline; letting the *optimum* move helps, letting *curvature* move hurts. Reading: at larger scale there is little hyperparameter variation left to model.

**Extrapolation hits statistical limits (§5.2, Fig. 10).** Independent laws, each fit to 128 runs at 4M/8M/17M/34M, agree near the data and diverge far from it because the fitted irreducible error ε "varies wildly across estimates." The law is reliable *near the data*; ε is not reliable until the law begins to saturate.

**Case study: pre-norm vs. post-norm (§5.3).** 511 / 512 / 128 post-norm configurations at 4M / 34M / 134M vs. 128 each for pre-norm; fit on the two smaller scales, validate on 134M. Cost framing: one 1B run ≈ 64 runs at 134M ≈ 1,024 at 34M ≈ 65,536 at 4M, so the whole study costs "a handful of billion-parameter runs." Four diagnostics: (1) the noisy-quadratic limit emerges for both, but for post-norm only deeper into the tail (smaller asymptotic regime → harder to tune); (2) sensitivity falls with scale for both, but post-norm keeps a second peak at a suboptimal loss even at 134M; (3) loss tracks accuracy on ARC-Easy/Challenge, BoolQ, [[mmlu|MMLU]], OpenBookQA, PIQA on a *common* curve across both architectures and all scales; (4) pre-norm gives an "excellent" law, post-norm a "reasonable" one. Pooling all sizes, **pre-norm scales better across the tested range**; with a *shared* irreducible error the advantage persists and grows with compute, whereas with *free* per-architecture ε post-norm would appear to overtake beyond the tested scales — which the authors read as the ε-noise problem of §5.2, not a real crossover.

## How this bears on the wiki's scaling-law comparison

[[kaplan-vs-chinchilla-vs-skaling]] rests on fits to grids of small runs and highlights [[skaling-chinchilla-exponents-kaplan-coupling-videau-2026|Skaling]]'s L-shaped sparse grid (sweep data at the smallest models, sweep size at the shortest horizons) extrapolating the full grid at ~5–10× less compute. This paper **supports the premise and complicates the accounting**:

- *Support.* The additive Chinchilla form fits cleanly down to 4M with 268M held out — well below the 100M–134M floors of Skaling's Farseer and SK-Grid — so there is nothing in principle wrong with fitting laws from cheap runs. The fitting form here is exactly Chinchilla's (`k = 1`); the paper never tests coupling, so it neither confirms nor refutes Skaling's `k < 1`.
- *Complication 1 — the cheap band is the sensitive band.* The L-shape concentrates fitting data at the smallest models and shortest horizons, precisely where this paper finds γ highest and the 25th-percentile gap widest. Skaling's grids used StepLaw-*rule* hyperparameters, not per-budget random search; this paper's Fig. 5 says that with a few configurations per budget "there is nothing to fit," and its Fig. 4 says per-budget tuning alone halves test MSE. So the "5–10× cheaper" figure is a compute count for the *runs*, not for the search needed to put those runs on the fully tuned frontier.
- *Complication 2 — a saddle is what under-tuning looks like.* Skaling's evidence for coupling is a saddle-shaped Chinchilla residual with the largest errors at the data-scarce and overtrained corners. Under-tuned small runs sit *above* the frontier by an amount that shrinks with scale (this paper's Fig. 6–7), which would also produce corner-heavy residuals. Skaling's own text concedes that hyperparameter parameterization "can alter the apparent N–D interaction"; this paper supplies the mechanism. It does not show Skaling's `k` is an artifact — SK-Grid's smallest model (134M) is already near the scale where γ→1 here — but it makes rigorous tuning a precondition for reading `k` off a sparse grid.
- *Complication 3 — ε is the least trustworthy number.* The comparison page already warns that Skaling's near-zero fitted `E` should not be read as a loss floor because `k` and `E` trade off. This paper shows that even within a single functional form, ε varies wildly across independent 128-run samples. Both papers converge on the same advice: trust the law near the data, distrust asymptotics.

Net: **complicates, does not contradict.** The comparison's claim that "the three laws are one nested family" is untouched; what changes is the cost model for the small-run strategy and the bar of evidence for corner residuals.

## Limitations (stated)

1. **Model-centric only.** Perplexity–capability correspondence (Mayilvahanan et al. 2025) breaks when pretraining data changes, so the methodology cannot rank *data* interventions; the authors leave data-centric small-scale research to future work.
2. **Extrapolation compounds error.** Laws hold near the data; the irreducible term is unidentifiable until saturation.
3. **Parametrization.** All results are under standard parametrization; μP or its successors may change how γ depends on scale.
4. **No quantitative surface model.** Coarse structure is captured; fine structure is not.
5. Scope: one recipe (Llama/Lingua), one corpus (FineWeb-Edu), scales ≤ 268M, no MoE, no long context.

## Concepts & entities

- [[scaling-laws]] — the object under study; the paper's contribution is *where* laws are observable (4M+) and *what it costs* to observe them.
- [[kaplan-vs-chinchilla-vs-skaling]] · [[skaling-chinchilla-exponents-kaplan-coupling-videau-2026]] — see the section above; this paper conditions the L-shaped sparse-grid strategy on per-budget tuning.
- [[chinchilla-hoffmann-2022]] · [[chinchilla]] — the additive functional form used here; runs go to 32 tokens/param to bracket the ~20 optimum.
- [[scaling-laws-kaplan-2020]] · [[jared-kaplan]] — the Kaplan/Hoffmann discrepancy is the paper's opening example of small-scale unreliability.
- [[pre-training]] — every claim is about pretraining loss; downstream capability enters only through the fixed-data correspondence.
- [[emergent-abilities]] — cited as one of the behaviors (with inverse and U-shaped scaling) that make *capability* laws unpredictable, motivating loss-as-proxy instead.
- [[layer-normalization]] · [[residual-connection]] · [[transformer-architecture]] — the pre-norm vs. post-norm case study.
- [[llama]] — the architecture family of every model trained.
- [[kyunghyun-cho]] · [[fair]] · [[meta]] — authors and affiliations.
- [[parcae-scaling-laws-looped-2026]] — another 2026 scaling-law recipe paper in the wiki whose loop-axis fits would inherit the same tuning caveat.

**Concept gaps (not created):** `noisy-quadratic-limit`, `hyperparameter-tuning` / `random-search`, `perplexity-capability-correspondence`, `mup` (maximal update parametrization), `warmup-stable-decay`, `pre-norm-vs-post-norm`, `fineweb-edu`, `lingua`.

## References

- [arXiv:2608.11859](https://arxiv.org/abs/2608.11859)
- [[scaling-laws-kaplan-2020]] · [[chinchilla-hoffmann-2022]] · [[skaling-chinchilla-exponents-kaplan-coupling-videau-2026]] · [[kaplan-vs-chinchilla-vs-skaling]]
