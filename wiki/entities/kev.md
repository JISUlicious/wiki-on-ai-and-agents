---
title: Kev
type: entity
created: 2026-09-28
updated: 2026-09-28
sources:
  - https://github.com/jaredpalmer/kev
  - system-one-ecosystem-sweep-2026-09-28.md
builds-on:
  - [[qwen]]
  - [[logit-scoring]]
parameters: 0.8B / 4B / 9B / 27B
status: complete
importance: medium
tags:
  - model
---

# Kev

A family of open [[system-one-model|System One]] decision models by Jared Palmer, Apache-2.0, and the most-starred project in the open [[jev|Jev]] ecosystem: 7,440 stars and 88 commits from 10 contributors in the week to 2026-09-28. Its design follows the black-box reverse-engineering in Archer Hume's "Jev's Architecture Unmasked."

## What it offers

- Four sizes: Kev-0.8B, 4B and 9B from Qwen3.5 base models with a shared recipe; Kev-27B from Qwen3.8-27B's post-trained release. See [[qwen]].
- `noul`, `choice` and `score` questions in one request. **Questions share the text but cannot read each other** — the isolation that [[logit-scoring]] records as the safe configuration.
- A temperature fitted on held-out data ships with each checkpoint, so probabilities are calibrated by default.
- Drop-in for Jev: TypeSafe's Python SDK works against a Kev server unchanged.
- Fine-tuning on your own labels. The bundled loop runs on Modal, a hosted service, so offline use means replacing that part.

## Against Jev, by its own numbers

Accuracy on "new sources" — datasets and policies never seen in training — development / test split:

| Model | Accuracy | Brier |
|---|---|---|
| Kev-4B | 0.817 / 0.838 | 0.269 / 0.242 |
| Kev-9B | 0.822 / 0.852 | 0.286 / 0.237 |
| Kev-27B | 0.848 / 0.896 | 0.236 / 0.164 |
| [[jev\|Jev]] | 0.857 / – | 0.211 / – |

The README is candid that this is not a controlled comparison, since Jev's training data is unknown.

> [!warning] Independent ranking disagrees
> On [[jevbench|JevBench]], Kev-4B ranks **31st of 91** (36.1), far below the fine-tuned 4B models at the top and below Jev. The two measures differ — JevBench is a composite that includes calibration on its hard tier, speed and cost — but the gap is large enough that Kev's own table should not be the only evidence. No independent figure exists for Kev-27B.

## Related

- [[jev]] · [[system-one-model]] · [[jevbench]] · [[nimble]] · [[laya]] · [[logit-scoring]] · [[calibration]] · [[lora]]

## References

- [jaredpalmer/kev](https://github.com/jaredpalmer/kev) · [[system-one-ecosystem-sweep-2026-09-28]]
