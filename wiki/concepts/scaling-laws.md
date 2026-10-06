---
title: Scaling Laws
type: concept
created: 2026-05-13
updated: 2026-10-07
sources:
  - gpt-3-brown-2020.md
status: complete
importance: high
---

# Scaling Laws

Scaling laws are empirical **power-law relationships** between a language model's loss (typically cross-entropy on held-out data) and the resources spent training it — parameters `N`, training tokens `D`, and compute `C`. The first influential statement was Kaplan et al. 2020 ("Scaling Laws for Neural Language Models", arXiv 2001.08361), and the relationships were re-derived with corrections by Hoffmann et al. 2022 ("Chinchilla").

## Kaplan et al. 2020 (the "GPT-3 era" laws)

Across many orders of magnitude:

- Loss scales as a power law in `N` (parameters), `D` (tokens), and `C` (compute), each holding the others non-bottlenecking.
- For a fixed compute budget, **bigger models trained on relatively less data** are optimal — implying that compute should be spent on parameters rather than tokens.

This directly motivated [[gpt-3-brown-2020|GPT-3]]'s 175B-parameter scale (Kaplan was a GPT-3 co-author).

## Hoffmann et al. 2022 ("Chinchilla")

Re-derived the laws with better methodology and concluded:

- The Kaplan recipe **under-trained** models on data.
- For a fixed compute budget, **parameters and tokens should scale roughly equally** (model size and data size both ~ √C).
- Most GPT-3-era models (GPT-3, MT-NLG, Gopher) were significantly larger than compute-optimal — a 70B model trained on more data ("Chinchilla") matched or beat a 280B model.

The Chinchilla picture shifted the field toward training smaller models on more data — visible in LLaMA, LLaMA-2, Mistral, etc.

> [!note] Refinement — the additive form assumes N and D don't interact
> [[skaling-chinchilla-exponents-kaplan-coupling-videau-2026|Skaling]] (FAIR, 2026) is a strict **one-parameter generalization**, `L = (A/N^α + B/D^β)^k + E`, recovering Chinchilla exactly at `k = 1`. Its argument is that any additive form forces the cross-derivative `∂²L/∂N∂D ≡ 0`, while the measured mixed derivative is non-zero everywhere and predominantly negative; fitted `k` is 0.41 and 0.31 on two grids. Error falls 1.5–3× (interpolation MAPE 0.41 vs 0.77; far extrapolation 0.70 vs 5.17 on one grid), with the largest corrections away from the compute-optimal ridge — which is where deployment now happens.
>
> **What this touches is the bullet above about equal scaling.** "Parameters and tokens should scale roughly equally" is a *consequence of the additive form*, not an independent measurement. Under coupling the optimal ratio `D*/N* ∝ C^((α−β)/(α+β))` becomes compute-dependent: on one grid the exponent is −0.11 against Chinchilla's ≈+0.03, implying **20–40 tokens/parameter at 10²⁵ FLOPs where Chinchilla extrapolates to 380**.
>
> Treat this as a refinement with an open frontier rather than a refutation. The Chinchilla-vs-Gopher empirical result is untouched, and — importantly — **the sign of the correction flips between the paper's two fitted grids**, so the authors' own conclusion is that coupling changes allocation but the *direction* depends on the data and architecture fitted. The practical takeaway is cheaper profiling: an L-shaped sparse grid of low-compute runs extrapolates the full grid at roughly 5–10× less compute.

## Why scaling laws matter

- **Predictability**: you can extrapolate the loss of a model 100× larger from smaller-scale runs, before spending the compute.
- **Compute-optimal allocation**: scaling laws tell you the right N/D ratio for a given budget.
- **Capability emergence**: while loss scales smoothly, *downstream capabilities* often emerge non-smoothly with scale (see [[gpt-3-brown-2020|GPT-3's 3-digit arithmetic]] at 13B). Whether emergence is a real phenomenon or a metric artifact (Schaeffer et al. 2023) is contested.

## Beyond pre-training

Scaling laws have also been derived for:

- **Fine-tuning** (Hernandez et al. 2021): transfer benefits scale with both base-model size and target-task data.
- **Inference compute** (test-time scaling, 2024+): test-time chain-of-thought / search expenditure can substitute for parameter scale in reasoning tasks.
- **Data quality** (Penedo et al. 2023, FineWeb): the constant in the power law depends on data quality, not just quantity.

## References

- [[gpt-3-brown-2020]]

## Precision as a scaling dimension

[[scaling-laws-for-precision-kumar-2024|Kumar et al.]] extend scaling laws to numeric precision, finding post-training-quantization damage grows with training tokens and shrinks with parameters — so **overtrained models quantize worse**. Compute-optimal pretraining precision comes out at ~7–8 bits. Corroborated by [[low-bit-favors-undertrained-ouyang-2024]]. See [[quantization-quality]].

## Training systems

- [[distributed-training]] — the systems side: PTD gives training FLOPs as F = 96Bslh²(1 + s/6h + V/16lh)

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[understanding-reasoning-pretraining-posttraining-shen-2026]] — explicit **joint pretraining–RL scaling law**; adds the outer split (pretraining vs RL) while leaving Chinchilla's inner split unchanged
- [[skaling-chinchilla-exponents-kaplan-coupling-videau-2026]] — one coupling exponent restores the N–D interaction the additive form discards; 1.5–3× lower error and a 5–10× cheaper profiling grid
- [[kaplan-vs-chinchilla-vs-skaling]] — side-by-side of all three functional forms, the accuracy tables, and the 10× allocation disagreement at 2×10²⁵ FLOPs
- [[breaking-the-token-ceiling-byte-model-distillation-marathe-2026]] — Meta: token→byte logit distillation; ~1B byte students overtake token students at high compute (extrapolated)
- [[self-play-pretraining-zero-data-cowsik-2026]] — Self-play pretraining: a generator writes programs, a byte learner predicts them; zero-shot power laws on natural data

## Lineage sources

- [[scaling-laws-carefully-weng-2026]] — Weng (2026): reconciling Kaplan and Chinchilla; data-constrained laws; how fragile the fits are
