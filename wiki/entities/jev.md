---
title: Jev
type: entity
created: 2026-09-21
updated: 2026-09-21
sources:
  - introducing-system-one-models-and-jev-typesafe-2026.md
status: draft
importance: medium
tags:
  - model
---

# Jev

[[typesafe-ai|TypeSafe AI]]'s first [[system-one-model|System One model]], announced 2026-09-15 and named after the economist William Stanley Jevons. It answers questions whose answer space is declared in advance and returns typed values with probabilities. It generates no text.

- **Interface**: program state plus schema-defined questions in one request; all questions answered in a single parallel pass. `POST /v1/systemone`, currently `jev-1.13.0`.
- **Limits**: 64k tokens per request, 32k for the state plus the longest question; text input only.
- **Training**: RLCD, Reinforcement Learning for Calibrated Decisions. No details published.
- **Reported performance**: 70–500 ms per call and 67.8% accuracy on a four-workflow benchmark against 67.9% for GPT-5.6 Terra and 74.1% for Sol. All vendor-reported; see the source page for the caveats, which are substantial.

**Open reimplementations**: within four days of the announcement, [[nimble|Bespoke-Nimble-9B]] (a fine-tune) and [[openjev|OpenJev]] (reading [[diffusiongemma|DiffusionGemma]] untrained) reproduced the interface, the latter wire-compatible enough that TypeSafe's own SDK runs against it unchanged. Measured agreement landed within ~3 points of Jev on Jev's own public cases. See [[logit-scoring]].

**What it cannot do**: free text, code, open-ended reasoning, or any answer outside the declared space. It returns no rationale, so a decision it makes cannot be explained to an auditor from its output alone.

> [!note] Draft
> No architecture, parameter count or base model is public. Whether Jev is a distinct architecture or a small tuned model with a parallel classification head cannot be determined from published material.

## Related

- [[typesafe-ai]] · [[system-one-model]] · [[calibration]] · [[logit-scoring]] · [[nimble]] · [[openjev]] · [[introducing-system-one-models-and-jev-typesafe-2026]]

## References

- [[introducing-system-one-models-and-jev-typesafe-2026]]
