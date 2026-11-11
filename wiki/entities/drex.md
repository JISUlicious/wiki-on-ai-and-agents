---
title: Drex
type: entity
created: 2026-11-10
updated: 2026-11-10
sources:
  - towards-foundation-decision-models-drex-dhankhar-2026.md
family: drex
builds-on:
  - [[kev]]
  - [[qwen]]
  - [[logit-scoring]]
parameters: 8B / 9B / 35B-A3B / 27B
status: draft
importance: medium
tags:
  - model
---

# Drex

An open family of four [[system-one-model|System One]] decision models from Nace.AI, described in [[towards-foundation-decision-models-drex-dhankhar-2026]] (arXiv:2611.07854). Like [[jev]], a Drex model takes a state (a document, a record, an agent's context) plus questions with fixed options, and returns a probability for each option without generating text.

## Members

- **Pointer models on the [[kev|Kev]] code base:** Drex DLM 8B (a diffusion model), Drex v1.5 9B (a DeltaNet hybrid) and Drex v2.0 35B-A3B (a mixture-of-experts model).
- **Drex Max 27B:** built on Qwen3.8-27B, with non-causal attention and a readout head over the options instead of reading next-token logits ([[logit-scoring]]).

## Reported results

- Drex Max scores **67.52** on the public Decision Index 0.4.1 suite, against **60.70** for [[jev|Jev]] 1.14.0, and answers a typical request in about 183 ms.
- Calibration: ECE of 1.4–4.5% across about 230k decisions per model, 0.8–1.4% after temperature tuning, and worse when split by domain ([[calibration]]).
- Training adds reinforcement learning from adjusted feedback (RLAF): a teacher's option probabilities, adjusted by the verified answer, serve as targets (+2.0 points). A consistency loss cuts answers that flip when options are reordered from 6.3% to 4.2%.

These are the authors' own numbers; no independent replication is in the wiki yet.

## Lineage

Drex inherits the pointer-head design and code base of [[kev]] for its three smaller models, and moves to a dedicated readout head for Max, the same split the wiki records for pplx-decider and Clef on [[sglang]]. It scales the open decision-model family from Kev's 0.8B–27B dense Qwen fine-tunes to diffusion, hybrid and MoE backbones.

## References

- [[towards-foundation-decision-models-drex-dhankhar-2026]]
