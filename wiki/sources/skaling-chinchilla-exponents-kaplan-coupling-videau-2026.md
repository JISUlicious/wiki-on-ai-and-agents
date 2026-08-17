---
title: "Skaling: Chinchilla's Exponents Meet Kaplan's Coupling"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.07222"
authors:
  - Mathurin Videau
  - Badr Youbi-Idrissi
  - David Lopez-Paz
  - Kartik Ahuja
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Skaling: Chinchilla's Exponents Meet Kaplan's Coupling

**Mathurin Videau, Badr Youbi-Idrissi, David Lopez-Paz, Kartik Ahuja** (FAIR at [[meta|Meta]]) — [arXiv:2608.07222](https://arxiv.org/abs/2608.07222), cs.CL, 7 Aug 2026. No venue listed.

## The problem

The [[chinchilla|Chinchilla]] parametric fit — `L(N,D) = A/N^α + B/D^β + E` — is the workhorse functional form of modern [[scaling-laws|scaling laws]]. Its two capacity/data terms are **added**, and the paper's central observation is that addition is not a neutral choice: an additive law forces the mixed partial derivative to vanish identically,

```
∂²L / ∂N∂D ≡ 0
```

i.e. it *assumes by construction* that model size and training data affect the loss independently. The authors estimate the mixed derivative numerically across their grids and find it **non-zero everywhere, and predominantly negative** — real synergy between N and D that no additive law can express. The empirical symptom is that fitted Chinchilla residuals form a **saddle over the (N,D) plane**: near-zero in the interior, growing in both directions toward the corners — overestimating loss where data is scarce relative to capacity, underestimating it in the heavily overtrained corner. Those corners are exactly where frontier decisions live (extreme overtraining) and where cheap pilot runs live (data-scarce).

## The method

**Skaling** keeps Chinchilla's interpretable base terms and its two *independent* inner exponents, but raises their sum to a single free outer exponent `k`, reinstating the coupling that [[jared-kaplan|Kaplan]] et al. 2020 had (via a tied exponent ratio) and Chinchilla dropped:

| Law | Form | Free params |
|---|---|---|
| Kaplan 2020 | `L = [ (N_c/N)^(α_N/α_D) + (D_c/D) ]^α_D` | inner exponents **tied** |
| Chinchilla | `L = A/N^α + B/D^β + E` | 5 |
| Farseer | data exponent & amplitude made N-dependent | 9 |
| **Skaling** | **`L = (A/N^α + B/D^β)^k + E`** | **6** |

`k = 1` recovers Chinchilla exactly, so Skaling is a strict **one-parameter generalization**, not a competing form. Fitted `k` comes out well below 1 on both grids (0.41 on Farseer, 0.31 on SK-Grid).

**Evaluation grids.** *Farseer* (Li et al. 2025a, external): 404 (N,D) configurations, 25 model sizes 100M–6.4B, 55 data budgets 1B–512B tokens, 1.6×10¹⁸–4.1×10²¹ FLOPs, seq len 2048. *SK-Grid* (the authors' own, internal, trained with Lingua): 134 configurations, 134M–4.9B params, 316M–316B tokens, 9.0×10¹⁶–9.9×10²⁰ FLOPs, Llama-3-style transformer (d_model 672–3264, 7–34 layers, 128,256 vocab, RoPE θ=10⁴), data mix 60% DCLM-Edu / 30% code / 10% math, StepLaw-tuned batch size and LR.

## Results

MAPE (lower is better), Skaling vs. Chinchilla vs. Farseer, on four regimes — interpolation, extrapolation in N, extrapolation in D, and "far" (both):

**Farseer grid, fit on the full grid (5.0×10²² FLOPs of fitting compute)**

| Regime | Chinchilla | Farseer | **Skaling** |
|---|---|---|---|
| Interpolation | 0.77 ± 0.04 | 1.73 ± 0.45 | **0.41 ± 0.05** |
| Ext. N | 1.48 ± 0.03 | 2.37 ± 0.08 | **0.47 ± 0.03** |
| Ext. D | 1.98 ± 0.08 | 4.13 ± 1.36 | **0.88 ± 0.06** |
| Far | 2.46 ± 0.19 | 2.43 ± 1.93 | **2.31 ± 0.18** |

**SK-Grid, full grid (3.1×10²¹ FLOPs)**

| Regime | Chinchilla | Farseer | **Skaling** |
|---|---|---|---|
| Interpolation | 0.81 ± 0.14 | 1.66 ± 0.32 | **0.33 ± 0.11** |
| Ext. N | 0.83 ± 0.11 | 0.90 ± 0.49 | **0.39 ± 0.05** |
| Ext. D | 1.44 ± 0.03 | 4.45 ± 0.21 | **0.58 ± 0.07** |
| Far | 5.17 ± 0.28 | 3.98 ± 1.25 | **0.70 ± 0.39** |

Aggregated over configurations, **Skaling wins at 76% of configurations, median improvement 2.2×, and ≥4× at a third of them**, "with the largest gains at the cheaper edges." The abstract's headline is a **1.5–3× MAPE reduction across interpolation and extrapolation** — the tables support that for every regime except Farseer-grid *far* extrapolation, where the three laws are within noise of each other (2.46 / 2.43 / 2.31).

### The L-shape sparse grid

Because the two decay rates are asymptotically separable (as D→∞ the loss becomes size-limited; as N→∞ it becomes data-limited), the authors propose fitting on an **L-shaped** subgrid: a D-band swept at the *smallest* models plus an N-band swept at the *shortest* training horizons. Both bands are cheap by construction.

| Fit grid | Fitting compute | Chinchilla far-MAPE | Skaling far-MAPE |
|---|---|---|---|
| Farseer full | 5.0×10²² FLOPs | 2.46 | 2.31 |
| Farseer L-shape | 5.1×10²¹ FLOPs (**~10×** less) | 9.82 | **1.51** |
| SK-Grid full | 3.1×10²¹ FLOPs | 5.17 | 0.70 |
| SK-Grid L-shape | 6.5×10²⁰ FLOPs (**~5×** less) | 14.63 | **1.15** |

The pattern is the real result: on the L-shape, **Chinchilla degrades catastrophically (2.46 → 9.82 and 5.17 → 14.63) while Skaling barely moves and stays at or below the *full-grid* Chinchilla baseline.** So the ~10× compute saving is a claim about *Skaling's* robustness to sparse, low-compute fitting data — not a property of scaling-law fitting in general.

> [!note] The "~10×" is grid-dependent
> The paper's Farseer L-shape is ~10× cheaper than its full grid; the SK-Grid L-shape is ~5×. The abstract rounds to "approximately 10x less compute than uniform sweeps."

### What coupling does to compute-optimal allocation

Under Skaling the optimal token-to-parameter ratio has a closed form:

```
D*/N*  ∝  C^((α − β)/(α + β))
```

Chinchilla's fits give a near-flat exponent (≈ +0.03) — the familiar "≈20 tokens/param, roughly constant with compute." On Farseer, Skaling fits α < β and yields an exponent of **≈ −0.11** (numerical gradients confirm −0.14 to −0.15): the compute-optimal ratio *shrinks* with scale, favoring parameters over tokens at the frontier. Extrapolated to 10²⁵ FLOPs the two prescriptions differ by more than 10× — **Chinchilla ≈ 380 tokens/param vs. Skaling 20–40**.

Crucially, **the sign is not universal**: on SK-Grid the fit gives α > β, i.e. the opposite direction. The authors' own summary is that "coupling changes large-scale allocation, while the direction of that change depends on the fitted data and architecture."

Fitted parameters, for reference:

| Grid | Law | A | B | α | β | k | E |
|---|---|---|---|---|---|---|---|
| Farseer | Chinchilla | 48 ± 12 | 110 ± 10 | 0.27 | 0.24 | 1 (fixed) | 0.45 |
| Farseer | Skaling | 290 ± 20 | 6000 ± 300 | 0.32 | 0.39 | **0.41 ± 0.01** | 0.03 ± 0.02 |
| SK-Grid | Chinchilla | 500 ± 120 | 710 ± 80 | 0.34 | 0.31 | 1 (fixed) | 1.75 |
| SK-Grid | Skaling | 5.3×10⁶ | 7.1×10⁶ | 0.73 | 0.63 | **0.31 ± 0.02** | 1.14 ± 0.06 |

Note how differently the two laws distribute the same fit: Skaling's inner exponents are much larger and its irreducible-loss floor `E` much smaller, because the outer `k` absorbs the flattening.

By D/N regime (Table 3 slices): undertrained D/N = 1.8–7, optimal D/N = 10–40, overtrained D/N = 56–158. Skaling stays under 0.9% MAPE in all three; **Chinchilla's worst band is actually the *optimal* one at 3.47% MAPE**, which cuts against the naive reading that the additive form only fails at the extremes.

## Where it breaks

There is no explicit limitations section; these are stated in-line.

- **Direction of the allocation shift is dataset-dependent.** Farseer and SK-Grid disagree on the sign of α−β. The robust claim is "coupling matters," not "train bigger models."
- **Hyperparameter parameterization confounds the coupling.** "Different parameterizations of hyperparameters in the training recipe can alter the apparent N–D interaction and shift the inferred compute-optimal ratio" — Chinchilla's original recipe and StepLaw tuning give different `k`.
- **`k` and `E` trade off.** The coupling exponent and the irreducible-loss floor both control how the surface flattens, and interior points constrain `E` only weakly (visible in the huge swing in fitted `E`: 0.45 → 0.03 on Farseer).
- **Global fit ≠ best per-ratio fit.** In far extrapolation Skaling beats per-ratio power laws (0.60% vs 0.99%) but cannot match a dedicated single-ratio fit (0.77%), because it optimizes across all ratios at once.
- **Far extrapolation on Farseer is a wash** (2.31 vs Chinchilla 2.46) — the win there is small.
- One grid is internal and unreleased (SK-Grid); no code/data release statement appears.

## Relationship to existing wiki pages

- **[[scaling-laws]] — generalization, not contradiction.** Skaling does not overturn the Chinchilla *result*; it nests it. `k = 1` is Chinchilla, and the paper argues the data prefer `k < 1`. The page's claim that "parameters and tokens should scale roughly equally" is a *consequence of the additive form*, and Skaling shows that consequence is an artifact of the assumed independence, not something measured. The concrete revision: the token/param optimum is compute-*dependent* under coupling, and one of the two grids says it decreases toward the frontier.
- **[[chinchilla]] — refinement at the edges.** The Chinchilla-vs-Gopher empirical comparison stands untouched. What is challenged is the extrapolated *prescription* far from the fitted interior, where the additive residual is saddle-shaped.
- **[[jared-kaplan|Kaplan]] is partly vindicated on form, not on prescription** — the coupling he had (and Hoffmann removed) is the thing being restored; his tied-exponent constraint is still relaxed.

## Related

- [[scaling-laws]] · [[chinchilla]] · [[pre-training]] · [[jared-kaplan]] · [[jordan-hoffmann]] · [[deepmind]] · [[meta]]
- [[transformer-architecture]] · [[llama]] — the SK-Grid runs are Llama-3-style dense transformers
- [[positional-encoding]] — RoPE θ=10⁴ in the SK-Grid recipe
- [[emergent-abilities]] — the corner regimes where the loss surface is worst-fit are also where downstream capability claims are most extrapolated

## References

- [arXiv:2608.07222](https://arxiv.org/abs/2608.07222)

## Related 2026 sources

- [[kaplan-vs-chinchilla-vs-skaling]] — the three-way comparison with Kaplan and Chinchilla
