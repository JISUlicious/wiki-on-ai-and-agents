---
title: TypeSafe AI
type: entity
created: 2026-09-21
updated: 2026-10-11
sources:
  - introducing-system-one-models-and-jev-typesafe-2026.md
status: draft
importance: medium
tags:
  - org
---

# TypeSafe AI

AI company that ships [[jev|Jev]] and coined the term **System One model** for it. Founded by Diogo Almeida, reported as a former [[openai|OpenAI]] researcher; press coverage describes roughly two years of development before the 2026-09-15 announcement. Early access is behind a waitlist.

Its commercial claim is a model priced per decision rather than per generated token: input at $0.042 per million tokens, output free. Its technical claim is a training method, RLCD, that optimizes for calibrated probabilities instead of human preference.

> [!note] Draft, and entirely self-reported
> Every figure in this wiki about TypeSafe comes from the company or from write-ups of the company's material. No paper, model card or calibration curve has been published, and no independent reproduction has appeared. See [[introducing-system-one-models-and-jev-typesafe-2026]].

## Since launch (to 2026-09-28)

Signups paused on 2026-09-22; no new model or paper; a second product, **Jev Router**, launched on OpenRouter on 2026-09-25. The $40M seed led by DCVC remains the only disclosed round. About a dozen third-party arXiv papers now study Jev, none from TypeSafe — see [[jev-research-papers-september-2026]].

> [!warning] The `TypeSafeAI` GitHub organisation is not TypeSafe
> `github.com/TypeSafeAI` describes itself as "an UNOFFICIAL organization made for community repos," created 2026-09-18. Issues filed there — including a calibration-verification offer — were never addressed to the company. TypeSafe's own organisation is `typesafe-ai`.

## Related

- [[jev]] · [[system-one-model]] · [[calibration]]

## References

- [[introducing-system-one-models-and-jev-typesafe-2026]]

## Related 2026 sources

- [[jev-as-a-judge-accept-when-confident-li-2026]] — CMU pre-registered study of Jev as a cheap judge: well calibrated when a reference or pair anchors the verdict, near chance without one

## Lineage sources

- [[system-one-ecosystem-sweep-2026-10-10]] — the category after OpenAI and Microsoft joined
- [[system-one-models-vs-system-one-thinking]] — name origin of "System One models" and "Jev"
