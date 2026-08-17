---
title: "Scaling Laws Compared: Kaplan vs Chinchilla vs Skaling"
type: comparison
created: 2026-08-17
updated: 2026-08-17
sources:
  - scaling-laws-kaplan-2020.md
  - chinchilla-hoffmann-2022.md
  - skaling-chinchilla-exponents-kaplan-coupling-videau-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Scaling Laws Compared: Kaplan vs Chinchilla vs Skaling

Three functional forms for the same object — pretraining loss as a function of parameters `N` and tokens `D`. The differences are almost entirely about **one structural question: do `N` and `D` interact?**

## The three forms

| | Functional form | Inner exponents | Coupling | Irreducible term |
|---|---|---|---|---|
| **[[scaling-laws-kaplan-2020\|Kaplan]]** (2020) | `L = [(N_c/N)^(α_N/α_D) + D_c/D]^α_D` | **tied** through the ratio `α_N/α_D` | **yes** — outer exponent `α_D` | none |
| **[[chinchilla-hoffmann-2022\|Chinchilla]]** (2022) | `L = A/N^α + B/D^β + E` | **independent** (`α`, `β`) | **no** — purely additive | free `E` |
| **[[skaling-chinchilla-exponents-kaplan-coupling-videau-2026\|Skaling]]** (2026) | `L = (A/N^α + B/D^β)^k + E` | **independent** (`α`, `β`) | **yes** — outer exponent `k` | free `E` |

The title of the Skaling paper is literal rather than rhetorical: it takes **Chinchilla's independent exponents** and **Kaplan's outer-exponent coupling**, and keeps Chinchilla's free irreducible term. It is a strict one-parameter generalization — at `k = 1` it *is* Chinchilla, exactly. Kaplan also had a coupling, but paid for it by tying the per-axis decay rates together; Skaling gets the coupling without that constraint.

So this is not three competing theories. It is **one nested family**, and the field has been sampling corners of it.

## The structural argument

Any purely additive form forces the cross-derivative to vanish:

$$\frac{\partial^2 L}{\partial N \, \partial D} \equiv 0$$

That is not a fitted result — it is baked into the shape before any data is seen. Skaling's core empirical claim is that the measured mixed derivative is **non-zero everywhere and predominantly negative**, verified with two independent non-parametric surrogates (Gaussian process and moving least squares). Fitted `k` comes out at **0.31–0.45** across grids rather than collapsing to 1.

The consequence shows up as a **saddle-shaped residual**: Chinchilla is fine in the interior of a fitting grid and systematically wrong at the edges — under-predicting in the data-scarce corner, over-predicting under heavy overtraining. Both corners are where deployment now happens.

## Accuracy (MAPE, lower is better)

| Regime | Chinchilla | Skaling |
|---|---|---|
| Single-axis extrapolation, Farseer | 1.48 / 1.98 | **0.47 / 0.88** |
| Single-axis extrapolation, SK-Grid | 0.83 / 1.44 | **0.39 / 0.58** |
| Far extrapolation, SK-Grid full grid | 5.17 | **0.70** |
| Far extrapolation, SK-Grid L-shape | 14.63 | **1.15** |
| Largest-`N`, SK-Grid L-shape | 6.09 | **0.77** |

Headline: **1.5–3× lower error**, with the gap widening in the imbalanced corners.

## The operational result that matters most

Grid-corner cross-validation is a statistician's test. §4.3 runs the question a lab actually asks: *predict the most expensive runs from cheap ones.* 112 held-out high-compute runs, every law refit only on the inexpensive remainder, split by regime.

| Law | Pooled MAPE | Worst regime |
|---|---|---|
| **Skaling** | **0.60 ± 0.27%** | never above 0.9% |
| Chinchilla | ~2.3× higher | **3.47% — in the *optimal* band** |

Skaling is a **3.9× reduction** over Chinchilla and beats the far more heavily parameterized Farseer law.

The Chinchilla row is the striking one. Its pooled number hides *where* it fails, and where it fails is the **compute-optimal band** — precisely the regime it was derived for and is most often used to reason about. The only law that edges Skaling is a one-dimensional per-ratio power law near the optimum (0.77 vs 0.88%), and that one is fit separately per recipe, so it cannot inform joint `N`–`D` allocation at all.

## What changes in practice: the allocation rule

Skaling **inherits Chinchilla's closed-form optimum unchanged** — the algebra is identical, and `D*/N*` stays constant across scales when `α ≈ β`:

$$R_{opt} \;\propto\; C^{\frac{\alpha-\beta}{\alpha+\beta}}$$

What changes is the *fitted parameters* feeding it. On the Farseer data:

- Numerical loss gradients recover a token-to-parameter ratio that **decreases** with compute, exponents **−0.14 and −0.15**.
- Skaling's analytic exponent: **−0.11** — agreeing with the non-parametric estimates.
- Chinchilla's: **+0.03**, near-flat and **opposite in sign**.

Extrapolated one order of magnitude beyond the data, to **2×10²⁵ FLOPs**, the two prescriptions differ by **more than 10×**: Chinchilla approaches **~380 tokens per parameter**, while both the empirical fits and Skaling fall to **20–40**.

That is a live disagreement, not a rounding difference. Frontier labs typically lock a fixed token-to-parameter ratio for a training run, so which curve you believe sets the architecture.

## Cheaper profiling

Because the coupled form degrades gracefully when the fitting grid is concentrated at the low-compute edges, Skaling supports an **L-shaped sparse grid** — sweep data volume for small models, sweep model size at one small data budget — and still extrapolate the full grid.

At roughly **10× less fitting compute**, Skaling stays close to or better than *full-grid* Chinchilla. Chinchilla under the same restriction collapses: interpolation 0.77 → 2.51, far extrapolation 5.17 → 14.63.

## Three caveats that keep this a refinement, not a refutation

> [!warning] The direction of the allocation change is dataset-specific
> On Farseer the fitted exponents give `α < β`, so `D*/N*` **decreases** with compute. On SK-Grid they give `α > β` on both grids, so the same closed form would **increase** it. The paper's own conclusion, verbatim: *"coupling changes large-scale allocation, while the direction of that change depends on the fitted data and architecture."* Anyone quoting "Chinchilla over-recommends tokens" as a general result is over-reading this paper.

> [!warning] On Chinchilla's own measurements, the coupling is weak
> Appendix D refits the original Chinchilla data, and Farseer-code alongside it. In both, fitted `k ≈ 0.77–0.90` — much closer to additive — and Skaling performs at **roughly Chinchilla-level accuracy**. This is exactly what the nested form predicts: Skaling departs from the additive law only when the data support a coupled surface. It also means Skaling does **not** show that Hoffmann et al. mis-fit their own experiments.

> [!warning] The near-zero irreducible loss is not a claim about the loss floor
> Fitted `E` drops sharply under Skaling — 0.45 → 0.03 on the Farseer full grid, 0.59 → 0.05 on the L-shape — though it stays substantial on SK-Grid (1.75 → 1.14). The authors explicitly decline to read `E ≈ 0` as a vanishing floor: `k` and `E` **trade off**, because a concave outer map (`k < 1`) makes the reducible term decay more slowly and can absorb curvature the additive law can only express as a larger constant. No run reaches the scale where loss saturates, so the data fix the *total* loss but not the split between a decaying term and a floor.

## The empirical results are untouched

Worth stating plainly, because it is easy to lose: **none of this disturbs the Chinchilla-vs-Gopher experiment.** A 70B model on 1.4T tokens really did beat a 280B model at equal compute, and GPT-3-era models really were undertrained. What Skaling challenges is the *extrapolated allocation rule* derived from the additive form — a claim about curve shape far outside the fitting grid, not a claim about a training run anyone performed.

The honest summary: Kaplan had coupling but tied exponents; Chinchilla freed the exponents but dropped coupling; Skaling has both, and the evidence says the interaction term is real — but how much it matters depends on the corpus, and how it redirects allocation depends on which side of `α = β` your data lands.

## Related

- [[scaling-laws]] · [[chinchilla]] · [[pre-training]] · [[emergent-abilities]]
- [[training-memory-anatomy]] · [[llm-memory-and-compute-bottlenecks]] — what the allocation decision costs downstream.
- [[cracks-in-the-foundation-long-context-extension-bertsch-2026]] — a companion warning that loss-based model selection misses long-context damage entirely.

## References

- [[scaling-laws-kaplan-2020]] · [[chinchilla-hoffmann-2022]] · [[skaling-chinchilla-exponents-kaplan-coupling-videau-2026]]
