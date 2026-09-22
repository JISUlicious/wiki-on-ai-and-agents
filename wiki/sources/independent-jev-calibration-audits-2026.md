---
title: "Independent calibration audits of Jev (September 2026)"
type: source
created: 2026-09-22
updated: 2026-09-22
sources:
  - https://github.com/jourdanlabs/assay-001
  - https://github.com/AnthusAI/Jev-Calibration
  - https://github.com/WanLanglin/jev-skills
  - https://github.com/jujumilk3/jev-calibration-audit
  - https://github.com/4esv/jev-eval
  - https://github.com/ickma2311/jev-baselines-eval
year: 2026
venue: "self-published GitHub repositories; none peer-reviewed"
status: complete
importance: high
tags:
  - 2026
---

# Independent calibration audits of Jev (September 2026)

An aggregation page, not a single source. Between 2026-09-17 and 2026-09-21, at least seven independent parties measured [[jev|Jev]]'s [[calibration]] — the thing [[typesafe-ai|TypeSafe AI]] sells and has never published a number for.

> [!important] The gap closed, and the answer is unflattering
> Jev's raw probabilities are **overconfident**, and calibration is **dataset-dependent** rather than a stable property. It degrades sharply on high-cardinality label spaces and collapses under format changes. Type safety, by contrast, holds up perfectly everywhere it was tested.

## The measurements

| Audit | Sample | Headline |
|---|---|---|
| `jourdanlabs/assay-001` | 8,576 API responses | CLINC150 **ECE 0.0204**; Banking77 **ECE 0.0936**, overconfident. **0 type errors** |
| `AnthusAI/Jev-Calibration` | 8,801 examples | Raw **ECE 0.117**, Brier 0.162, log-loss 0.467 → isotonic **ECE 0.008** |
| `WanLanglin/jev-skills` | 4,995 real agent decisions | `noul` **ECE 0.169**, `choice` **ECE 0.226** |
| `jujumilk3/jev-calibration-audit` | ~7,000 calls | Baseline ECE 0.023–0.076; **ECE 0.793** with abstain removed |
| `4esv/jev-eval` | 5 tasks × 300 | CLINC 0.039, Polarity 0.042, Intent 0.110, Sentiment5 **0.200** |
| `Luni/laya-jev-benchmark` | 2,000 + 2,000 | Phishing ECE 0.154; typed-decisions ECE 0.144 |
| `ickma2311/jev-baselines-eval` | 408 paired | Accuracy audit; both verdicts **AMBIGUOUS** vs pre-registered criteria |

**The most citable** is `jourdanlabs/assay-001`: pre-registered with its protocol frozen before any query was issued, dataset hashes recorded in advance, every request and response SHA256-sealed, and a deterministic re-score that matched every field. Its split verdict is the single most defensible claim available — calibration holds on CLINC150 and fails on Banking77, on the same model in the same week.

**The most naturalistic** is `WanLanglin/jev-skills`, measured on 4,995 real coding-agent decisions rather than a benchmark. Its reliability curve is well-behaved at the tails and broken in the middle: confidence 0.0–0.1 → 0.7% accuracy (n=843), 0.9–1.0 → 96.6% (n=29), but 0.4–0.5 → **11.4%** accuracy (n=271). The author's practical response was to discard everything below 0.90.

## Three findings that matter more than the numbers

**Miscalibration is plateau-shaped, not sigmoid-shaped.** `AnthusAI` reports isotonic regression vastly outperforming Platt scaling (ECE 0.008 vs 0.052). A temperature parameter cannot fix this shape — which matters because temperature scaling is the standard cheap remedy.

**It is fragile to format, not just to domain.** `jujumilk3` found that removing the abstain option collapses the model on unanswerable items: accuracy 0.950 → 0.000, ECE → **0.793**. Calibration that survives a domain shift but not a schema edit is not a robust property. Korean vs English made almost no difference (0.076 vs 0.075), so this is not a language effect.

**Type safety is real.** `jourdanlabs` recorded **zero type errors across all 8,576 responses**. Of TypeSafe's two claims, the one guaranteed by construction is the one that holds.

## What this does not establish

> [!warning] Seven unreviewed repos are not a literature
> Every measurement here is self-published on GitHub within seven days of launch, most with zero or one star, none peer-reviewed, none replicated by a second party. They use different datasets, different question types and different binning, so the ECE figures are **not comparable to each other** — a range of 0.02 to 0.25 across audits reflects methodology as much as model behaviour. The direction is consistent; the magnitude is not established.

Some are also self-interested: the ECE 0.246 figure comes from [[laya]]'s vendor page, a direct competitor. `Heman10x-NGU/openJev-verdict-2.0` catalogs its Jev row as a borrowed baseline rather than a measurement it performed. And `zhlei07/open-system-one` declined to publish a comparative ECE at all, on the honest grounds that its own temperatures were fitted on 1,000 labeled dev examples while "Jev gets none" — a fairness problem every other audit here ignores.

No audit separates **RLCD's contribution from the architecture's**; nobody has run that ablation, including TypeSafe.

## The vendor's own position

TypeSafe has published no reliability curve and, per a 2026-09-21 Latent Space interview, does not intend to: Diogo Almeida is "extremely anti-public benchmarks" and rejects publishing ECE figures as gameable. That makes the central claim **structurally unfalsifiable by design** — while these audits proceed anyway.

The docs do concede more than the launch post did. `docs.typesafe.ai/model-jaggedness/jev-1.13`, last reviewed 2026-09-17, lists nine failure modes and states that "jev-1.13's score levels are weak in numerical calibration," and that calibration is a **group-level** property, not a per-answer guarantee. The second point is easy to miss and materially narrows the product: a group-level guarantee does not license acting on any individual probability.

Separately, on Hacker News Almeida agreed that Jev is essentially a zero-shot classifier ("exactly right!") and that "Large Classification Model rather than a Large Language Model" was "very accurate!" — which concedes the framing critique the [[system-one-model]] page raises on independent grounds.

## Related

- [[jev]] · [[calibration]] · [[system-one-model]] · [[typesafe-ai]] · [[laya]] · [[introducing-system-one-models-and-jev-typesafe-2026]] · [[nimble]] · [[openjev]]

## References

- [jourdanlabs/assay-001](https://github.com/jourdanlabs/assay-001) · [AnthusAI/Jev-Calibration](https://github.com/AnthusAI/Jev-Calibration) · [WanLanglin/jev-skills](https://github.com/WanLanglin/jev-skills)
- [jujumilk3/jev-calibration-audit](https://github.com/jujumilk3/jev-calibration-audit) · [4esv/jev-eval](https://github.com/4esv/jev-eval) · [ickma2311/jev-baselines-eval](https://github.com/ickma2311/jev-baselines-eval)
- [TypeSafe model jaggedness docs](https://docs.typesafe.ai/model-jaggedness/jev-1.13) · [Latent Space interview, 2026-09-21](https://www.latent.space/p/jev)
