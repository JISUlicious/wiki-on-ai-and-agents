---
title: JevBench
type: concept
created: 2026-09-28
updated: 2026-09-28
sources:
  - system-one-ecosystem-sweep-2026-09-28.md
status: complete
importance: medium
---

# JevBench

The de facto public leaderboard for [[system-one-model|System One]] decision models, maintained by one person at [fstandhartinger/jevbench](https://github.com/fstandhartinger/jevbench) with a live board at benchmarkheaven.com. Version 1.4.2.2 (2026-09-27) lists 95 systems, 91 ranked.

## How it scores

An **equal-weight composite of four axes**: Intelligence, Calibration, Speed and Cost.

- **Intelligence** is chance-corrected accuracy, `(accuracy − chance) / (1 − chance)`, per tier: hard 30%, standard 28%, judge 28%, easy 14%. The official figure blends 20% from 308 fresh sealed decisions with 80% of the v1.3 axis.
- **Calibration** is measured on the hard tier: ECE plus fidelity to exact gold distributions. Systems that return labels without probabilities score zero here.
- The hard tier is **220 decisions, 111 of them public** and 109 held out, written by Claude Opus 5 and GPT-5.6 Sol.

## The top of the board (v1.4.2.2)

| Rank | System | Composite |
|---:|---|---:|
| 1 | Imajev-4B | 67.37 |
| 2 | Plumb-4B (JevK5 v0.2 + LoRA) | 65.84 |
| 3 | decider-4b v2 | 64.13 |
| 4 | [[jev\|Jev]] 1.13.0 | 63.29 |
| 5 | JevK5 v0.2.0 | 62.04 |

Diffusion-based servers rank far lower: djev 11th (52.2), [[openjev|OpenJev]] on [[diffusiongemma|DiffusionGemma]] 30th (36.9). [[kev|Kev-4B]] is 31st (36.1).

> [!warning] "Open 4B models beat Jev" is mostly a speed-and-cost result
> The board's own notes say Jev **out-reasons** decider-4b v2 (Intelligence 53.1 vs 49.4) and is **better calibrated**; the 4B model leads on speed and cost. A self-hosted 4B model will always win those two axes against a hosted API, and they are half the composite. Read the per-axis scores, not the rank, when choosing a model for decision quality.

Two further caveats. Half the hard tier is public and can be trained on — several top entries are LoRA fine-tunes built during the same week the board existed. And cost for self-hosted entries is estimated from a public reference price for the base model, not from a GPU bill.

## Related

- [[system-one-model]] · [[jev]] · [[kev]] · [[laya]] · [[openjev]] · [[calibration]] · [[independent-jev-calibration-audits-2026]]

## References

- [[system-one-ecosystem-sweep-2026-09-28]] · [JevBench repository](https://github.com/fstandhartinger/jevbench)
