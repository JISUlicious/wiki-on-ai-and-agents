---
title: Reward Modeling
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - christiano-rlhf-2017.md
  - ouyang-instructgpt-2022.md
builds-on: "[[preference-modeling]]"
status: draft
importance: medium
---

# Reward Modeling

Reward modeling is the practice of training a separate neural network to **predict a reward signal** that can then be optimized against — typically because the true reward is unavailable, expensive, or hard to specify (cf. [[rlhf]] for LLMs).

In the LLM context, the canonical reward model is a fine-tuned LM with a scalar head, trained on pairwise human (or AI) [[preference-modeling|preference]] data via a Bradley-Terry loss:

```
P(y_w ≻ y_l | x) = σ(r̂(x, y_w) − r̂(x, y_l))
```

The trained `r̂` then provides a scalar signal for policy optimization (PPO, RLOO, etc.) or can be used at inference time for best-of-N sampling.

## Where it appears

- **[[rlhf-christiano-2017|Christiano et al. 2017]]** — the foundational RL-from-preferences paper. Reward model trained on pairwise trajectory comparisons.
- **[[instructgpt-ouyang-2022|InstructGPT (Ouyang 2022)]]** — Reward Model is stage 2 of the SFT → RM → PPO pipeline.
- **[[constitutional-ai-bai-2022|Constitutional AI]]** — preference model trained on AI-judged comparisons ([[rlaif]]).
- Process Reward Models (Lightman et al. 2023) — score intermediate reasoning steps, not just final answers.

## Avoided in DPO

[[dpo]] (Rafailov 2023) **eliminates the explicit reward model**: the policy itself implicitly parametrizes the reward, and the preference loss is optimized directly. This is one of DPO's major practical wins — no separate model to train, store, or serve.

## Caveats

- **Reward hacking** ([[reward-hacking]]): the policy can find ways to score high on `r̂` without satisfying the underlying intent.
- **Out-of-distribution behavior**: `r̂` is reliable on data near the training distribution; it can score wildly on unfamiliar inputs.
- **KL regularization** vs. a reference model is the standard mitigation — penalize the policy for drifting too far from a known-good starting point.

## References

- [[rlhf-christiano-2017]]
- [[instructgpt-ouyang-2022]]
