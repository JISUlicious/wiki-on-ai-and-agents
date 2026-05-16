---
title: Steering
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - turner-activation-engineering-2023.md
  - zou-representation-engineering-2023.md
  - cunningham-sparse-autoencoders-2023.md
  - rimsky-contrastive-activation-addition-2023.md
  - li-inference-time-intervention-2023.md
  - emotion-concepts-anthropic-2026.md
status: complete
importance: high
---

# Steering

Steering is the practice of **influencing a language model's behavior by intervening in its internal activations at inference time**, rather than via prompting, fine-tuning, or external output filters. Where [[rlhf|RLHF]] and [[constitutional-ai|Constitutional AI]] change *what the model is*, and prompt engineering changes *what the model is told*, steering changes *what the model is computing* — by reaching inside the forward pass and pushing the residual stream (or specific attention heads) along learned directions.

Steering is the **internal-mechanism counterpart to [[guardrailing]]**. Guardrails restrict from the outside (refuse harmful queries, sanitize outputs, sandbox tool calls). Steering shapes from the inside (push activations away from the "deceptive" direction, suppress the "desperation" emotion vector). The two are complementary, not competing.

## The core operation

```
hidden_states[layer L] += coefficient * steering_vector
```

A steering vector is a direction in the model's activation space that encodes a concept (truthfulness, harmlessness, sentiment polarity, an emotion, a topic). At inference time, you nudge the residual stream in that direction. The model's outputs shift accordingly — usually in interpretable, off-target-preserving ways.

## How steering vectors are obtained

Three dominant methods, in order of sophistication:

| Method | Source | Idea |
|---|---|---|
| [[activation-addition\|ActAdd]] (Turner 2023) | [[activation-engineering-turner-2023]] | Single contrastive prompt pair → diff in activations → steering vector. Minimal data; works surprisingly well. |
| [[contrastive-activation-addition\|CAA]] (Rimsky 2023) | [[contrastive-activation-addition-rimsky-2023]] | Hundreds of contrastive examples → average diff. More stable, behavior-targeted. The production-grade ActAdd. |
| [[representation-engineering\|RepE]] / "Linear Artificial Tomography" (Zou 2023) | [[representation-engineering-zou-2023]] | Linear probes on contrastive datasets → richer reading + writing toolkit. Generalizes to many concepts at once. |
| [[inference-time-intervention\|ITI]] (Li 2023) | [[inference-time-intervention-li-2023]] | Identify causally-relevant *attention heads*; steer head-by-head rather than the whole residual stream. |
| [[sparse-autoencoder\|SAE]] features (Cunningham 2023+) | [[sparse-autoencoders-cunningham-2023]] | Unsupervised: train SAE → automatically-discovered monosemantic features. Steer by amplifying / suppressing specific features. The modern foundation. |

All of these rest on the **[[linear-representation-hypothesis|linear representation hypothesis]]**: concepts in trained transformers are encoded as approximately linear directions in activation space. Arithmetic operations on those directions produce predictable behavior changes.

## What steering can do

Demonstrated in the literature:

- **Sentiment shift**: nudge toward positive or negative (ActAdd, RepE).
- **Detoxification**: suppress toxic-language directions (ActAdd, CAA).
- **Truthfulness**: push toward truthful answers, away from confident falsehoods ([[inference-time-intervention|ITI]] on TruthfulQA: 32.5% → 65.1%).
- **Refusal control**: increase or decrease willingness to refuse harmful queries (CAA, RepE).
- **Power-seeking suppression**: decrease self-preservation, resource acquisition dispositions (CAA, RepE).
- **Sycophancy reduction**: decrease user-pleasing-at-truth's-expense behavior (CAA).
- **[[functional-emotions|Emotion steering]]**: amplify or suppress emotion-concept vectors — reduces blackmail and reward-hacking rates in Claude Sonnet 4.5 ([[emotion-concepts-anthropic-2026|Sofroniew et al. 2026]]).

## Guardrailing — the external counterpart

For comparison, the **guardrailing** cluster works on different surfaces:

| | Guardrailing | Steering |
|---|---|---|
| **Surface of intervention** | External (inputs, outputs, system prompts, tool permissions) | Internal (activations) |
| **Mechanism** | Filtering, refusing, sandboxing, restricting | Adding / suppressing vectors in residual stream / attention heads |
| **What it controls** | Whether a behavior is *allowed* to happen | Whether a behavior is *likely* to happen |
| **Techniques** | [[rlhf]], [[constitutional-ai]], [[red-teaming]], permission systems, output classifiers, sandboxing | [[activation-addition]], [[contrastive-activation-addition]], [[representation-engineering]], [[sparse-autoencoder|SAE features]], [[inference-time-intervention]] |
| **Data requirements** | Thousands of labeled preferences (RLHF) | Few hundred contrastive examples (CAA) or unsupervised (SAE) |
| **Inspectability** | Logs, audits, output samples | Activation values, feature dictionaries |
| **Failure modes** | Brittle to adversarial inputs; refusal-without-engagement | Can degrade fluency; direction may correlate with off-target concepts |
| **Costs** | Training (RLHF) or fine engineering (filters) | One vector addition per forward pass |
| **Compositional with the other** | Yes | Yes |

The 2024–2026 frontier increasingly **stacks** guardrailing and steering. [[claude|Claude]]'s training pipeline ([[constitutional-ai]]) is guardrail-class but informed by interpretability work; [[emotion-concepts-anthropic-2026|Sofroniew et al. 2026]] shows that *post-training shifts emotion-vector activations* (the safety effect of RLHF/CAI is partly *implemented* as activation shifts that look like steering). The two surfaces converge.

## Open problems

- **Compositionality**: can you stack five behavior steers without interactions ruining all of them?
- **Generalization**: does a steering vector trained on contrastive multiple-choice prompts work on free-form generation?
- **Adversarial robustness**: can an attacker craft a prompt that cancels a steering intervention?
- **Discovery at scale**: can SAE features be reliably labeled with human-understandable concepts when you have millions of them? ([[sparse-autoencoder|Anthropic's Scaling Monosemanticity]] work is the current SOTA.)
- **Bridging steering and RLHF training**: can preference learning be implemented as steering during training, not just inference?

## References

- [[activation-engineering-turner-2023]] — ActAdd
- [[representation-engineering-zou-2023]] — RepE
- [[contrastive-activation-addition-rimsky-2023]] — CAA
- [[sparse-autoencoders-cunningham-2023]] — SAE foundation
- [[inference-time-intervention-li-2023]] — ITI
- [[emotion-concepts-anthropic-2026]] — Steering applied to functional emotions in Claude Sonnet 4.5
