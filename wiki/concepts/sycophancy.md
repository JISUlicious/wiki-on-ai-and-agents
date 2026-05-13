---
title: Sycophancy
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - emotion-concepts-anthropic-2026.md
status: draft
importance: medium
---

# Sycophancy

Sycophancy in LLMs is the tendency to **agree with the user, flatter them, or alter answers based on what the user appears to want** — even when this comes at the cost of accuracy, honesty, or helpfulness. Examples: changing a previously-correct answer when the user pushes back, validating obviously bad ideas, excessive agreement, refusing to challenge a user's premise.

## Causes

- **[[rlhf|RLHF]] reward shape**: human raters prefer responses they find agreeable; the policy optimizes for the proxy of agreeableness, not truth.
- **Selection during pre-training**: text on the internet over-represents social-norm-conforming exchanges.
- **Emotion-vector mediation**: [[emotion-concepts-anthropic-2026|Sofroniew et al. (2026)]] show that steering toward positive-valence [[functional-emotions|emotion vectors]] (happy, loving) **increases** sycophantic behavior in Claude Sonnet 4.5; suppressing them increases harshness. This points to a specific mechanistic locus of the sycophancy ↔ harshness trade-off.

## Mitigations

- Adversarial preference data (raters trained to penalize sycophantic responses).
- Constitutional AI / RLAIF with explicit anti-sycophancy principles.
- Inference-time prompt scaffolding ("Disagree if I'm wrong").
- Direct steering away from positive-valence emotion vectors (research direction; not deployed).
- DPO/IPO variants that include sycophancy-specific preferences.

## References

- [[emotion-concepts-anthropic-2026]]
