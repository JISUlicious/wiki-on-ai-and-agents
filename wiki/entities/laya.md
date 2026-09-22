---
title: Laya
type: entity
created: 2026-09-22
updated: 2026-09-22
sources:
  - https://huggingface.co/convaiinnovations/laya
  - independent-jev-calibration-audits-2026.md
builds-on:
  - [[cloze-scoring]]
parameters: 421M (English) / 322M (multilingual)
status: complete
importance: medium
tags:
  - model
---

# Laya

Open [[system-one-model|System One]] model from Nandakishor Mukkunnoth at ConvAI Innovations, released September 2026 under Apache 2.0 with open weights. Two variants: **ModernBERT-large at 421M** parameters for English, and mmBERT-base at 322M covering 100+ languages.

It is the most important of the open [[jev|Jev]] alternatives, because it is the one that shows the whole diffusion detour was unnecessary.

## Mechanism

An **encoder**, not a decoder and not a diffusion model. Every option is scored at its own `[MASK]` token and softmaxed per question — the pattern-verbalizer readout of [[cloze-scoring]], applied at product scale. No canvas, no denoising, no seeded template, no engine patch. It claims RLCD-style training for calibrated probabilities, mirroring [[typesafe-ai|TypeSafe]]'s stated objective.

Because each option gets its own mask position, questions are naturally isolated — avoiding the slot-position asymmetry that afflicts multi-slot single-pass reads on diffusion models.

## Against Jev

| Measure | Laya | [[jev\|Jev]] 1.13.0 |
|---|---:|---:|
| p50 latency | **32.8 ms** | 236–276 ms |
| Typed-decisions | **0.766** | 0.727 |
| AG News | **0.950** | 0.910 |
| DAIR Emotion | **0.595** | 0.480 |
| Banking77 (77 classes) | 0.425 | **0.870** |

**7.8× faster** at roughly 1/60th the plausible parameter count, winning on three of four tasks — and then losing catastrophically on the fourth.

> [!important] The crossover is label cardinality
> Laya degrades past roughly 20 options; Jev handles 255. Banking77's 77 intents are where a small encoder with one mask per option runs out of room, and it loses by 44 points. This is the clearest statement available of what frontier scale actually buys in a typed-decision model: not accuracy on ordinary classification, but **capacity in the label space**.

Its vendor page also publishes a Jev ECE of 0.246 against Laya's 0.081 after temperature scaling. That is a competitor measuring a competitor and should be weighted accordingly — see [[independent-jev-calibration-audits-2026]] for the disinterested measurements, which put Jev's ECE lower but still overconfident.

## Why it matters

The first wave of open Jev clones ran on [[diffusiongemma|DiffusionGemma]], because a diffusion model's native masked-slot operation looked like the natural substrate. Laya shows the parallel-readout advantage was replicable far more cheaply with a bidirectional encoder — which has had exactly this capability since BERT. No one has ported the System One pattern to any other diffusion LM (LLaDA, Dream, Mercury, Gemini Diffusion), while encoder and frozen-autoregressive implementations proliferated.

## Related

- [[jev]] · [[system-one-model]] · [[cloze-scoring]] · [[logit-scoring]] · [[nimble]] · [[openjev]] · [[calibration]] · [[masked-language-modeling]] · [[bert]]

## References

- [laya.convaiinnovations.com](https://laya.convaiinnovations.com/) · [HuggingFace](https://huggingface.co/convaiinnovations/laya) · [receptron/laya](https://github.com/receptron/laya)
- [[independent-jev-calibration-audits-2026]]
