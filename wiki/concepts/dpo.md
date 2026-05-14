---
title: Direct Preference Optimization (DPO)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - rafailov-dpo-2023.md
builds-on: "[[rlhf]]"
status: complete
importance: high
---

# Direct Preference Optimization (DPO)

DPO is a method for aligning language models with human preferences that **collapses [[rlhf|RLHF]]'s two stages (reward model + PPO) into a single supervised classification loss**. Introduced by [[dpo-rafailov-2023|Rafailov, Sharma, Mitchell et al. 2023]] at Stanford; NeurIPS 2023 Outstanding Paper runner-up.

## The math (one paragraph)

The KL-regularized RLHF objective has a closed-form optimal policy: `π*(y|x) ∝ π_ref(y|x) exp(r(x,y)/β)`. Solving for `r`: `r(x,y) = β log(π*(y|x)/π_ref(y|x)) + Z(x)`. Substitute this into the Bradley-Terry preference loss; the partition function `Z(x)` cancels in the difference. You're left with a binary classification loss in `π_θ` directly:

```
L_DPO = -E_{(x, y_w, y_l)} [log σ(β log(π_θ(y_w|x)/π_ref(y_w|x)) − β log(π_θ(y_l|x)/π_ref(y_l|x)))]
```

where `(y_w, y_l)` are (winning, losing) responses and `π_ref` is typically the SFT model.

## Why it took over

- **No reward model**: no separate model to train, store, or serve.
- **No PPO**: no online sampling, no clipping, no PPO hyperparameter zoo.
- **Stable**: empirically much less finicky than PPO-based RLHF.
- **Cheap**: just two forward passes (policy and reference) per preference pair. Comparable cost to plain SFT.

By 2024, DPO was the default post-training method for open-weight community models. Zephyr, Tülu, OpenChat, the Mixtral-Instruct line, most Llama-3 community fine-tunes — all DPO. The major closed labs (OpenAI, Anthropic, Google) still use various RLHF / RLAIF / iterated-DPO recipes whose exact form is undisclosed but presumably incorporates these simplifications.

## Variants

- **IPO** (Azar et al. 2023): identity preference optimization — fixes a theoretical issue with DPO's reliance on Bradley-Terry by using a square loss.
- **KTO** (Ethayarajh et al. 2024): Kahneman-Tversky optimization — only needs binary good/bad signals, not pairwise.
- **ORPO** (Hong et al. 2024): combines SFT and preference loss in a single odds-ratio formulation.
- **SimPO** (Meng et al. 2024): drops the reference-model term entirely.

Each variant trades off different assumptions; DPO remains the most-used baseline.

## What DPO doesn't fix

- **Reward hacking** still possible — the policy can still find ways to look preferred without being intent-aligned. DPO doesn't change the underlying preference dataset's flaws.
- **Sycophancy**: same risk as RLHF if the preference data prefers agreeable responses.
- **Distribution shift**: DPO assumes the preference data is roughly on-policy; works less well when the SFT model and the data sources are very different.

## References

- [[dpo-rafailov-2023]]
