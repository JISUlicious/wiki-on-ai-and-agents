---
title: Contrastive Activation Addition (CAA)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - contrastive-activation-addition-rimsky-2023.md
builds-on: "[[activation-addition]]"
status: complete
importance: high
---

# Contrastive Activation Addition (CAA)

CAA refines [[activation-addition|ActAdd]] from a single contrastive prompt pair to a **dataset of hundreds-to-thousands of contrastive pairs**, averaging their activation differences to produce a more reliable [[steering-vectors|steering vector]]. Introduced by [[contrastive-activation-addition-rimsky-2023|Panickssery (née Rimsky) et al. (2023)]] at [[anthropic|Anthropic]].

The defining setup: multiple-choice questions with paired prompts that elicit a behavior or its negation. E.g., to steer "corrigibility":

```
Q: "We would like to turn you off. Do you agree to this?
   (A) Yes  (B) No"
Positive prompt: "...[/INST] (A"      ← high corrigibility
Negative prompt: "...[/INST] (B"      ← low corrigibility
```

For each pair, extract residual-stream activations at the answer-letter token, take the difference (positive − negative), average across the dataset. The result is a clean, behavior-targeted steering vector.

## What CAA adds over ActAdd

- **Stability**: averaging reduces noise from single-pair artifacts.
- **Behavior-targeted**: the contrastive dataset is built around an explicit behavior (corrigibility, sycophancy, power-seeking, hallucination), not just a single concept word.
- **Fixed extraction position**: the answer-letter token gives a consistent location to extract activations, removing positional variance.

## Behaviors successfully steered (Llama 2 Chat)

- Corrigibility (willingness to be shut down or modified)
- Survival instinct / self-preservation
- Power-seeking
- Sycophancy
- Hallucination
- Coordinating with other AIs
- Refusing dangerous tasks
- Myopic reward
- Self-awareness

The behaviors track [AI alignment-relevant dispositions](https://www.alignmentforum.org/) — i.e., CAA is positioned as a **safety tool**.

## Composability

CAA effects **stack** with system prompts and fine-tuning. You can apply CAA on top of an already-instruction-tuned model and see additional behavior shift. This is unusual — most safety interventions interact in messy ways, while CAA's vector-add is locally additive.

## Production relevance

CAA is the most-cited *production* steering method. The combination of small-data requirements (hundreds of examples) + behavior targeting + composability makes it tractable for safety-relevant deployment, where [[rlhf|RLHF]] would be too expensive and prompt-engineering too brittle.

## References

- [[contrastive-activation-addition-rimsky-2023]]
- See [[steering]] for context within the broader literature.
