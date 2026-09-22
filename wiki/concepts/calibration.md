---
title: Calibration
type: concept
created: 2026-09-21
updated: 2026-09-22
sources:
  - introducing-system-one-models-and-jev-typesafe-2026.md
  - llm-as-a-verifier-kwok-2026.md
  - rehearse-confidence-cliff-autoresearch-ji-2026.md
status: complete
importance: high
---

# Calibration

A model is **calibrated** when its stated probabilities match observed frequencies: among the cases it calls 70% likely, about 70% happen. It is the property that makes a probability safe to act on, and it is the training objective of the [[system-one-model|System One]] class.

## How it is measured

- **Reliability diagram**: plot accuracy against stated confidence. Perfect calibration is the diagonal; the usual deviation is above it, meaning overconfidence.
- **Expected calibration error (ECE)**: bin predictions by confidence, average the gap between confidence and accuracy. Convenient and bin-dependent.
- **Proper scoring rules**: Brier score and log loss. A scoring rule is *strictly proper* when its expected score is uniquely maximized by reporting your true belief, so honesty cannot be gamed (Gneiting and Raftery, 2007). Log loss is the only proper rule that is *local*, depending on the forecast only through the probability it gave the outcome that occurred.

> [!warning] Calibration alone is worthless without sharpness
> A model that always predicts the base rate is perfectly calibrated and tells you nothing. Calibration constrains the *average* of the probabilities, not their *informativeness*. Sharpness is the missing half, and strictly proper scoring rules are what force a model to have both. Any claim of "calibrated decisions" should be read with this in mind: ask what the sharpness is, not only whether the reliability diagram is near the diagonal.

Calibration is also a property *of a distribution*. A model calibrated on one traffic mix is not calibrated on another, which is why post-hoc methods such as temperature and Platt scaling are refit per deployment.

## A probability is still not a decision

Acting requires a loss function: choose the action minimizing expected loss under the predicted distribution. That threshold belongs to the operator, not the model vendor, because only the operator knows the cost of a false positive against a false negative. **Selective prediction** adds a third option, abstain, trading coverage for accuracy; conformal methods can give coverage guarantees without distributional assumptions.

## Why language models are bad at this

- **Post-training degrades it.** Supervised fine-tuning and RLHF reward answers that sound decisive, so preference-tuned models are more overconfident than their pre-trained selves.
- **Verbalized confidence is just tokens.** "I am 90% sure" is a generated string with no mechanical link to the model's internal probability.
- **The signal exists, and text decoding throws it away.** [[llm-as-a-verifier-kwok-2026|Kwok et al.]] read a *continuous* expectation over scoring-token logits instead of the argmax token and gained **16.6 points** over discrete judging on the same base model.
- **It degrades along a trajectory.** [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]] measured a judge falling from **82.8% to 56.9%** selective accuracy as an optimization run deepened, while its coverage *rose*: it committed more as it got worse, and emitted no signal that it had.
- **Self-confidence ranks poorly within a problem.** [[metacognition]] records judgment-of-learning spread of 0.048 within a problem against 0.270 across problems, so a model's own confidence can allocate compute but cannot pick the best attempt.

## Related

- [[logit-scoring]] · [[cloze-scoring]] · [[system-one-model]] · [[independent-jev-calibration-audits-2026]] · [[laya]] — the model class that makes calibration the training objective.
- [[llm-as-a-judge]] · [[verifier]] · [[agent-evaluation]] — where miscalibration does its damage.
- [[metacognition]] — the model's own sense of knowing, and its limits.
- [[constrained-decoding]] — validity by construction, which is a different guarantee from correctness.

## References

- [[llm-as-a-verifier-kwok-2026]] · [[rehearse-confidence-cliff-autoresearch-ji-2026]] · [[introducing-system-one-models-and-jev-typesafe-2026]]
- Gneiting and Raftery, *Strictly Proper Scoring Rules, Prediction, and Estimation*, JASA 2007.
