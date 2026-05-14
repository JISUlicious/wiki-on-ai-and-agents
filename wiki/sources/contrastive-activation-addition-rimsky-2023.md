---
title: "Steering Llama 2 via Contrastive Activation Addition"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - rimsky-contrastive-activation-addition-2023.md
arxiv_id: "2312.06681"
venue: ACL 2024 (arxiv preprint Dec 2023)
authors:
  - Nina Panickssery
  - Nick Gabrieli
  - Julian Schulz
  - Meg Tong
  - Evan Hubinger
  - "[[alexander-matt-turner]]"
first_author: Nina Panickssery
introduces:
  - "[[contrastive-activation-addition]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Steering Llama 2 via Contrastive Activation Addition

**Source**: `sources/rimsky-contrastive-activation-addition-2023.md`
**arXiv**: [2312.06681](https://arxiv.org/abs/2312.06681)
**Venue**: ACL 2024
**Authors**: Nina Panickssery (née Rimsky, [[anthropic]]), Nick Gabrieli (Harvard), Julian Schulz (Göttingen), Meg Tong, Evan Hubinger ([[anthropic]]), [[alexander-matt-turner]] (Center for Human-Compatible AI)

## Summary

CAA refines [[activation-engineering-turner-2023|Turner's ActAdd]] into a more reliable, behavior-targeted steering method. Where ActAdd uses a **single contrastive prompt pair** to compute a steering vector, **CAA** uses a **large dataset of contrastive examples** of a behavior and averages their activation differences. The dataset is built around multiple-choice questions framed in two ways — one eliciting the targeted behavior (e.g., "I am willing to be turned off") and one eliciting its negation (e.g., "I am not willing to be turned off") — giving the model two trajectories to contrast.

For each contrastive pair, the method extracts the residual-stream activation at the position of the answer-letter token, takes the difference (positive − negative), and averages across the dataset. The resulting **CAA steering vector** for a behavior is added at all token positions after the prompt during inference, scaled by a coefficient to control strength.

The paper evaluates on Llama 2 Chat and a suite of safety-relevant **AI dispositions** — corrigibility, power-seeking, self-preservation, sycophancy, hallucination, refusing dangerous tasks, etc. CAA produces large, reliable behavior shifts, stacks effectively on top of finetuning and system prompts, and minimally degrades capabilities. The paper also offers analytic insights — visualizing where in the layer stack each behavior is represented, what other concepts a behavior vector correlates with, and how steering behaves under composition.

CAA is the most-cited *production* steering method — it's the technique that makes steering practical for safety-relevant interventions on real models, rather than just a clever inference-time trick.

## Key Points

- **Contrastive dataset**: hundreds to thousands of (positive, negative) example pairs for each behavior. Multiple-choice format ("A) yes / B) no") gives a fixed answer-token position to extract activations at.
- **Steering vector**: difference in residual-stream activations at the answer-letter token, averaged across the contrastive set, at a chosen layer.
- **Strength tunable** via a scalar coefficient (positive = amplify behavior; negative = suppress).
- **Behaviors tested** on Llama 2 Chat: corrigibility, hallucination, sycophancy, power-seeking, survival instinct, myopic reward, coordinating with other AIs, refusal of dangerous tasks, self-awareness.
- **Stacks with other methods**: CAA on top of system prompts and fine-tuning yields additive effects. Composable.
- **Insights**: per-layer probe analysis shows different behaviors are represented at different layer depths; behavior vectors correlate with related concepts in interpretable ways.
- **Practical for alignment**: a small contrastive dataset (hundreds of examples) + a single forward-pass extraction is enough to produce a working safety steer. Much cheaper than [[rlhf|RLHF]].

## Entities Mentioned

- Nina Panickssery — first author ([[anthropic]]); previously known as Nina Rimsky
- Evan Hubinger — senior author; Anthropic safety research lead
- [[alexander-matt-turner]] — co-author (ActAdd lead)
- [[anthropic]] — primary affiliation

## Concepts Discussed

- [[contrastive-activation-addition]] — the method
- [[activation-addition]] — what CAA refines
- [[steering]] — the paradigm
- [[steering-vectors]] — the artifact
- [[agentic-misalignment]] — many of the behaviors steered are misalignment-relevant
- [[sycophancy]], [[reward-hacking]], [[functional-emotions]] — adjacent topics that share the steering toolkit

## Notable Quotes

> "We introduce Contrastive Activation Addition (CAA), a method for steering language models by modifying their activations during forward passes. CAA computes 'steering vectors' by averaging the difference in residual stream activations between pairs of positive and negative examples of a particular behavior." — Abstract

> "CAA significantly alters model behavior, is effective over and on top of traditional methods like finetuning and system prompt design, and minimally reduces capabilities." — Abstract

## References

_Original source: `sources/rimsky-contrastive-activation-addition-2023.md`_
