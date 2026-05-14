---
title: "Direct Preference Optimization: Your Language Model is Secretly a Reward Model"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - rafailov-dpo-2023.md
arxiv_id: "2305.18290"
venue: NeurIPS 2023
authors:
  - "[[rafael-rafailov]]"
  - Archit Sharma
  - Eric Mitchell
  - Stefano Ermon
  - "[[christopher-manning]]"
  - "[[chelsea-finn]]"
first_author: "[[rafael-rafailov]]"
introduces:
  - "[[dpo]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Direct Preference Optimization: Your Language Model is Secretly a Reward Model

**Source**: `sources/rafailov-dpo-2023.md` (extracted from `rafailov-dpo-2023.pdf`)
**arXiv**: [2305.18290](https://arxiv.org/abs/2305.18290)
**Venue**: NeurIPS 2023 (Outstanding Paper Runner-Up)
**Authors**: [[rafael-rafailov]], Archit Sharma, Eric Mitchell (equal first; [[stanford-university|Stanford]]), Stefano Ermon, [[christopher-manning]], [[chelsea-finn]]

## Summary

This paper **collapses the [[rlhf|RLHF]] pipeline** from two stages (reward model + RL via PPO) into a **single supervised classification step**. The key insight is that for the KL-regularized RLHF objective, the optimal policy has a **closed-form** relationship to the reward function — meaning the reward function can be **re-parameterized in terms of the policy itself**. Doing so collapses the loss into a binary classification over preference pairs:

```
L_DPO = -E[log σ(β log(π_θ(y_w|x) / π_ref(y_w|x)) - β log(π_θ(y_l|x) / π_ref(y_l|x)))]
```

where `(y_w, y_l)` is a (winning, losing) pair from the preference dataset, `π_θ` is the policy being trained, `π_ref` is the reference policy (typically the SFT model), and `β` is a temperature that controls how strongly to push the policy.

The practical consequence: **no separate reward model**, **no PPO**, **no sampling from the model during training**, **no hyperparameter tuning hell**. Just a supervised loss on the same preference data RLHF would use. Stable, efficient, and as effective or better than PPO-based RLHF on summarization, sentiment control, and single-turn dialogue benchmarks.

DPO became the default post-training method for open-weight models almost immediately. By late 2023, virtually every fine-tuned Llama / Mistral / Mixtral derivative was DPO-trained. The variants — IPO, KTO, ORPO, SimPO — refine the loss formulation but all share DPO's "skip the reward model" insight.

## Key Points

- **The mathematical core**: under the KL-regularized RLHF objective `max_π E_x,y∼π[r(x,y)] − β·KL(π || π_ref)`, the optimal policy is `π*(y|x) ∝ π_ref(y|x) exp(r(x,y)/β)`. Rearranging: `r(x,y) = β log(π*(y|x)/π_ref(y|x)) + Z(x)`. The reward function is *expressible in terms of the policy*. Substituting this back into the Bradley-Terry preference loss yields a loss in `π_θ` directly.
- **The DPO loss**: binary classification over (winning, losing) preference pairs. Computationally cheap — just two forward passes per preference pair, no sampling.
- **Eliminates reward model**: no separate model to train, store, serve.
- **Eliminates PPO**: no on-policy sampling, no clipping schedule, no PPO hyperparameters.
- **Stability**: empirically much more stable than PPO; less hyperparameter tuning.
- **Results**: matches or beats PPO-based RLHF on summarization (Reddit TL;DR), sentiment control, and single-turn dialogue (Anthropic HH).
- **Adoption**: by 2024, the default post-training method for open-weight models (Zephyr, Mixtral-Instruct, Llama-2-chat variants, Tülu, etc.).

## Entities Mentioned

- [[rafael-rafailov]], Archit Sharma, Eric Mitchell — co-first authors (Stanford PhDs at the time)
- [[chelsea-finn]] — Stanford ML faculty; advisor on DPO
- [[christopher-manning]] — Stanford NLP director; advisor on DPO
- [[stanford-university]] — author affiliation

## Concepts Discussed

- [[dpo]] — the method introduced
- [[rlhf]] — what DPO simplifies
- [[preference-modeling]] — DPO uses the same Bradley-Terry preference data as RLHF
- [[reward-modeling]] — what DPO sidesteps

## Notable Quotes

> "In this paper we introduce a new parameterization of the reward model in RLHF that enables extraction of the corresponding optimal policy in closed form, allowing us to solve the standard RLHF problem with only a simple classification loss." — Abstract

> "The resulting algorithm, which we call Direct Preference Optimization (DPO), is stable, performant, and computationally lightweight, eliminating the need for sampling from the LM during fine-tuning or performing significant hyperparameter tuning." — Abstract

## References

_Original source: `sources/rafailov-dpo-2023.md`_
