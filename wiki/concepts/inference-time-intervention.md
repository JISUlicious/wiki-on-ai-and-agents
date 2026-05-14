---
title: Inference-Time Intervention (ITI)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - li-inference-time-intervention-2023.md
builds-on: "[[steering]]"
status: complete
importance: medium
---

# Inference-Time Intervention (ITI)

ITI is a **head-level [[steering]]** technique introduced by [[inference-time-intervention-li-2023|Li et al. (NeurIPS 2023)]] at Harvard. Unlike residual-stream methods ([[activation-addition|ActAdd]], [[contrastive-activation-addition|CAA]]), ITI targets **specific attention heads** identified as carrying truthfulness signal, intervening only on those.

## Procedure

1. **Probe**: train linear classifiers on each head's activations to predict whether the head is encoding truthful content at a given position. Train on TruthfulQA-style true/false pairs.
2. **Identify causally-relevant heads**: heads where the probe is accurate *and* a perturbation changes behavior. Result: ~10–50 heads out of ~1000.
3. **Steer at inference**: at every token position, add a steering vector along the "truthful" direction *only on the identified heads*, scaled by a tunable strength.

## Result

On Alpaca (instruction-finetuned LLaMA), ITI lifts TruthfulQA from **32.5% → 65.1%** — nearly doubling truthfulness. Minimal degradation on unrelated benchmarks. Achieved with just a few hundred labeled examples (compared to thousands needed for [[rlhf|RLHF]]).

## What makes ITI distinctive

- **Head-level surgical surface**: more fine-grained than residual-stream methods. Lets you reason about *where* in the architecture a concept is being computed.
- **Data efficiency**: a few hundred examples per concept is enough.
- **Truthfulness ↔ helpfulness trade-off**: stronger intervention increases refusals ("I have no comment") — adjustable via strength coefficient.

## The philosophical claim

> "LLMs may have an internal representation of the likelihood of something being true, even as they produce falsehoods on the surface."

This is the strongest version of the **"the model already knows" thesis** — the truthful answer exists somewhere in the activations, and default decoding can fail to surface it. ITI's contribution is finding which heads carry that knowledge and forcing them to express it.

The same thesis runs through [[representation-engineering|RepE]] (concepts as readable directions), [[functional-emotions|emotion vectors]] (emotions causally drive outputs), and [[sparse-autoencoder|SAE]] features (clean monosemantic features lurking in superposition). ITI is one of the first crisp empirical demonstrations.

## References

- [[inference-time-intervention-li-2023]]
- See [[steering]] for the broader paradigm.
