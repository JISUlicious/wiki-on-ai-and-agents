---
title: Logit Scoring
type: concept
created: 2026-09-21
updated: 2026-09-21
sources:
  - structured-generation-mode-for-diffusiongemma-mastracci-2026.md
  - bespoke-nimble-github-2026.md
  - introducing-system-one-models-and-jev-typesafe-2026.md
status: complete
importance: high
---

# Logit Scoring

Reading a decision off a model's probability distribution instead of generating it as text. The model is never asked to *write* an answer; it is asked to produce a distribution over a fixed set of candidate tokens, and that distribution is the output.

This is the mechanism underneath every [[system-one-model|System One model]], and it is what separates them from [[structured-outputs|structured output]] and [[constrained-decoding|constrained decoding]], which still generate tokens and merely restrict which ones are legal.

## The recipe

1. **Declare the answer space.** Every allowed answer is enumerated in advance.
2. **Make each answer one token.** This is the load-bearing constraint. Multi-token labels are remapped to single tokens — `moderation_spam` becomes `A`, or `yes`/`no`, or `0`/`1`/`2` — with the legend supplied in the prompt. One token per answer means the position of everything else is fixed.
3. **Run one forward pass.**
4. **Read the logits at the answer position** and softmax over only the candidate tokens.
5. **Derive confidence from the distribution itself** — typically entropy-based. [[openjev]] uses `1 − H(p)/ln K`, which is 1 when the model is certain and 0 when the distribution is uniform.

Two properties follow for free. An answer **cannot go off-schema**, because only legal tokens are ever scored — there is no JSON to parse and no repair path. And the confidence is the model's own distribution rather than a number the model *says* about itself, which matters because verbalized self-confidence is poorly calibrated (see [[calibration]]).

## Two implementations

The pattern is architecture-agnostic, which is why it appeared independently in both model families within a week of Jev's announcement:

| | Autoregressive ([[nimble]]) | Diffusion ([[openjev]], [[diffusiongemma]]) |
|---|---|---|
| Where the answer lives | next-token position after the prompt | a masked slot in the canvas |
| Fields per pass | one field per pass on CUDA; parallel on [[mlx\|MLX]] | all slots in one pass, natively |
| Cross-field attention | none — fields scored independently | every slot attends to the full context and to the other slots |
| Needs fine-tuning? | yes, [[lora\|LoRA]] on curated pairs | no — read the base model |

The diffusion route has a structural advantage here: filling masked slots *is* the model's native operation, so reading several answer slots in a single bidirectional pass is not a repurposing. The autoregressive route needs either one pass per field or a fine-tune that teaches the model to answer in a fixed position.

## Why it is fast

The cost is one prefill plus one (or a handful of) forward passes, with **zero decode steps**. Generation cost normally scales with output length; here the output is a single token position per question, so latency is dominated by the prompt. That is the entire basis for the order-of-magnitude latency gaps reported across [[jev]] (246.7 ms), [[nimble]] (106 ms on H100), [[openjev]] (94 ms p50) and generative frontier models answering the same questions (2,792–2,896 ms). See [[arithmetic-intensity]].

Multi-step refinement on the diffusion side reuses the single prefill and rewrites only the answer slots, so extra steps cost GPU time but not prompt tokens.

## Limits

> [!warning] Probabilities normalized over a supplied set are not correctness rates
> Softmax over the candidate tokens forces the probabilities to sum to 1 across *the answers you supplied*. If the true answer is not among them, the model still returns a confident-looking distribution. Nimble's documentation is explicit: 0.9 does not mean right 90% of the time, and if "none of these" is possible you must add it as an option. Any threshold has to be validated on your own data.

Other constraints are inherent: the answer space must be small enough to enumerate and map to single tokens (Nimble caps enums at 26); no rationale is produced, so a decision cannot be explained to an auditor from its output alone; and fields scored independently cannot be mutually consistent by construction — the caller must check.

## Related

- [[system-one-model]] · [[jev]] · [[nimble]] · [[openjev]] · [[diffusiongemma]] · [[calibration]] · [[structured-outputs]] · [[constrained-decoding]] · [[logit-lens]]

## References

- [[structured-generation-mode-for-diffusiongemma-mastracci-2026]]
- [[bespoke-nimble-github-2026]]
- [[introducing-system-one-models-and-jev-typesafe-2026]]
