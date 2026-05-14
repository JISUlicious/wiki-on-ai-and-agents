---
title: "Inference-Time Intervention: Eliciting Truthful Answers from a Language Model"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - li-inference-time-intervention-2023.md
arxiv_id: "2306.03341"
venue: NeurIPS 2023
authors:
  - Kenneth Li
  - Oam Patel
  - "[[fernanda-viegas]]"
  - Hanspeter Pfister
  - "[[martin-wattenberg]]"
first_author: Kenneth Li
introduces:
  - "[[inference-time-intervention]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Inference-Time Intervention: Eliciting Truthful Answers from a Language Model

**Source**: `sources/li-inference-time-intervention-2023.md`
**arXiv**: [2306.03341](https://arxiv.org/abs/2306.03341)
**Venue**: NeurIPS 2023
**Authors**: Kenneth Li (first), Oam Patel, [[fernanda-viegas]], Hanspeter Pfister, [[martin-wattenberg]] (Harvard)

## Summary

Inference-Time Intervention (ITI) is a head-level steering method that increases LLM truthfulness by intervening on **specific attention heads** identified as carrying truthfulness signal. The procedure:

1. **Probe**: train a small linear classifier (probe) on each attention head's per-token activations to predict whether the head is encoding truthful information at that position. Use TruthfulQA-style true/false statement pairs as training data.
2. **Identify causally-relevant heads**: probes with high classification accuracy + intervention validation (perturbing the head changes behavior) identify a small set (~10–50 heads) that matter for truthfulness.
3. **Intervene at inference**: at each token position, add a steering vector along the "truthful" direction *only on the identified heads*, at every layer the head appears, with a tuned strength.

Results: on Alpaca (instruction-finetuned LLaMA), ITI improves TruthfulQA scores from **32.5% → 65.1%** — nearly doubling truthfulness — with minimal capability degradation on other benchmarks. The paper also highlights a **truthfulness ↔ helpfulness trade-off**: stronger truthfulness interventions can make the model refuse to answer ("I have no comment") more often, reducing helpfulness.

ITI is an important data point in the **[[steering|steering]]** literature for two reasons:
1. It's **head-level**, not residual-stream-level — a more fine-grained intervention surface than [[activation-addition|ActAdd]] or [[contrastive-activation-addition|CAA]].
2. It's **highly data-efficient**: only a few hundred labeled examples needed to locate the directions, compared to the tens of thousands of preference labels for [[rlhf|RLHF]].

The paper's broader claim: **"LLMs may have an internal representation of the likelihood of something being true, even as they produce falsehoods on the surface"** — i.e., the truthfulness direction *exists* inside the model regardless of whether the default decoding surfaces it. This anticipates the broader "representations encode the answer; sampling can fail to elicit it" theme that runs through [[functional-emotions|emotion vectors]], [[representation-engineering|RepE]], and SAE-based steering.

## Key Points

- **Head-level intervention**: target ~10–50 specific attention heads rather than the full residual stream. More surgical than residual-stream methods.
- **Two-stage**: linear probe identifies candidate heads → causal validation filters to truly relevant ones.
- **Result**: Alpaca TruthfulQA 32.5% → 65.1%. Nearly doubles truthfulness without breaking other capabilities.
- **Data efficiency**: ~few hundred labeled true/false pairs. RLHF would need thousands.
- **Truthfulness–helpfulness trade-off**: tunable via intervention strength. Stronger intervention → more refusals.
- **Implicit philosophical claim**: the model already "knows" the truth in some internal sense; default decoding may not surface that knowledge. Steering reveals latent capabilities.

## Entities Mentioned

- Kenneth Li — first author (Harvard PhD)
- [[fernanda-viegas]], [[martin-wattenberg]] — senior authors; founding figures in visual analytics + AI interpretability at Harvard / Google

## Concepts Discussed

- [[inference-time-intervention]] — the method
- [[steering]] — the broader paradigm
- [[hallucination]] — what ITI mitigates
- [[mechanistic-interpretability]] — head-level analysis is in this style

## Notable Quotes

> "We introduce Inference-Time Intervention (ITI), a technique designed to enhance the 'truthfulness' of large language models (LLMs). ITI operates by shifting model activations during inference, following a set of directions across a limited number of attention heads." — Abstract

> "Our findings suggest that LLMs may have an internal representation of the likelihood of something being true, even as they produce falsehoods on the surface." — Abstract

## References

_Original source: `sources/li-inference-time-intervention-2023.md`_
_Code: https://github.com/likenneth/honest_llama_
