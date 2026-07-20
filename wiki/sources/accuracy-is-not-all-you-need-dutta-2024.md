---
title: "Accuracy is Not All You Need"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
arxiv_id: "2407.09141"
year: 2024
authors:
  - Abhinav Dutta
tags:
  - 2024
  - paper
status: complete
importance: high
---

# Accuracy is Not All You Need

**Dutta et al.** (Microsoft Research India) — [arXiv:2407.09141](https://arxiv.org/abs/2407.09141), 2024.

## The argument

Aggregate accuracy is the wrong metric for evaluating [[quantization]], because it can stay flat while the model's actual behaviour changes substantially. The paper introduces **%flips** — the percentage of answers that changed correct->incorrect *or* incorrect->correct between baseline and quantized model.

Verbatim: *"Accuracy is preserved within 1% for the majority of the quantization methods, tasks and models... all quantization schemes except GPTQ W8A16 have significant %flips"* — up to **13.6%**.

## Why accuracy hides it

The mechanism is the paper's best contribution. Define **top margin** = the probability gap between the model's best and second-best answer option. Quantization perturbs logits; an answer flips when its top margin is small.

*"Correct (incorrect) answers have higher (lower) top margin and are thus less (more) likely to change."*

So flips are **asymmetric** — more wrong answers change than right ones — and the two directions nearly cancel in the aggregate. The model looks unchanged and isn't.

**KL-divergence correlates with %flips at Spearman 0.981** on MMLU, making KLD a usable cheap proxy.

## Generative tasks amplify it

MT-Bench (GPT-4 judge) shows the cancellation failing: Llama-2-70B-chat drops **7.431 -> 6.801** at GPTQ W4A16 (~5%), and degradation is *"higher for the harder turn2 problem than for turn 1, with up to 10% loss for Llama2-70b."*

**Larger models flip less**: Llama-2-70b shows 3-8% flips at 4-bit vs 13-20% for 7b.

The authors are appropriately careful: flips are *"only a warning that the behaviour of a model and its compressed version is different - this may or may not materialize as visible degradation."*

## Concepts

- [[quantization-quality]] · [[quantization]]

## References

- [arXiv:2407.09141](https://arxiv.org/abs/2407.09141)

