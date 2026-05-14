---
title: Preference Modeling
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - christiano-rlhf-2017.md
status: draft
importance: medium
---

# Preference Modeling

The practice of learning a **reward function** (or policy directly) from **pairwise comparisons** between options, rather than from a dense scalar reward signal. Introduced in the LLM/RL context by [[rlhf-christiano-2017|Christiano et al. 2017]] as the core of [[rlhf|RLHF]].

## The Bradley-Terry model

Given two trajectory segments (or LLM responses) `σ¹` and `σ²`, and a human label of which is preferred, model the probability of the human's choice as:

```
P(σ¹ ≻ σ²) = exp(r̂(σ¹)) / (exp(r̂(σ¹)) + exp(r̂(σ²)))
             = σ(r̂(σ¹) − r̂(σ²))
```

where `r̂` is a learned reward function (typically a fine-tuned LLM with a scalar output head). Train by cross-entropy against the observed preferences.

The Bradley-Terry parametrization assumes underlying scalar utilities exist that explain pairwise choices — convenient and empirically works. The same form underlies the Elo rating system used in chess.

## Where it shows up

- [[rlhf]] — the canonical use case; reward model is trained from preferences, then policy is optimized against the reward via PPO.
- **DPO** ([[dpo-rafailov-2023|Rafailov et al. 2023]]): reformulates the same preference objective so it can be optimized directly on the policy without an explicit reward model. Same preference data, same Bradley-Terry assumption.
- **Constitutional AI** ([[constitutional-ai-bai-2022|Bai et al. 2022]]): AI-generated preferences (RLAIF) substitute for human ones.
- **AlphaGo Zero** and its descendants use a related "value head" trained on game outcomes — a degenerate preference (game win/loss) signal.

## Limitations

- **Transitive assumption**: Bradley-Terry assumes a consistent underlying utility, but real human preferences are often intransitive (A>B>C>A on different criteria).
- **Label quality**: preferences are noisy and rater-dependent. The "average human preference" is a fuzzy target.
- **Reward hacking**: the policy can find ways to satisfy the reward model without satisfying the underlying intent — see [[reward-hacking]].

## References

- [[rlhf-christiano-2017]]
