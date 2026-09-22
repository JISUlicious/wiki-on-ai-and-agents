---
title: Jev
type: entity
created: 2026-09-21
updated: 2026-09-22
sources:
  - introducing-system-one-models-and-jev-typesafe-2026.md
  - independent-jev-calibration-audits-2026.md
status: complete
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

## Calibration, as measured

Seven independent audits between 2026-09-17 and 2026-09-21 found Jev's raw probabilities **overconfident**, with calibration dataset-dependent rather than a stable property: ECE 0.0204 on CLINC150 and 0.0936 on Banking77 in the same pre-registered study, `noul` ECE 0.169 and `choice` ECE 0.226 across 4,995 real agent decisions, and ECE 0.793 once an abstain option is removed. Type safety held perfectly — **zero type errors across 8,576 responses**. Full detail and caveats on [[independent-jev-calibration-audits-2026]].

TypeSafe's own jaggedness docs, reviewed 2026-09-17, concede that "jev-1.13's score levels are weak in numerical calibration" and that calibration is a **group-level** property, not a per-answer guarantee — which materially narrows what any single probability licenses you to do.

> [!note] What it is, by the founder's own account
> No architecture, parameter count or base model is public, and no paper exists — an arXiv search for "reinforcement learning for calibrated decisions" returns nothing. On Hacker News Diogo Almeida agreed that Jev is essentially a zero-shot classifier ("exactly right!") and that "Large Classification Model rather than a Large Language Model" was "very accurate!". He has said he will not publish ECE figures, considering public benchmarks gameable — which leaves the central claim unfalsifiable by the vendor and falsified-in-direction by everyone else.

Where frontier scale does show up is **label-space capacity**: Jev handles 255 options and takes Banking77's 77 intents at 0.870, where the 421M encoder [[laya]] collapses to 0.425.

## Related

- [[typesafe-ai]] · [[system-one-model]] · [[calibration]] · [[logit-scoring]] · [[cloze-scoring]] · [[nimble]] · [[openjev]] · [[laya]] · [[independent-jev-calibration-audits-2026]]

## References

- [[introducing-system-one-models-and-jev-typesafe-2026]]
