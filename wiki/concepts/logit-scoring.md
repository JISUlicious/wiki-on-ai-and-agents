---
title: Logit Scoring
type: concept
created: 2026-09-21
updated: 2026-09-29
sources:
  - structured-generation-mode-for-diffusiongemma-mastracci-2026.md
  - bespoke-nimble-github-2026.md
  - introducing-system-one-models-and-jev-typesafe-2026.md
  - independent-jev-calibration-audits-2026.md
  - system-one-ecosystem-sweep-2026-09-28.md
status: complete
importance: high
---

# Logit Scoring

Reading a decision off a model's probability distribution instead of generating it as text. The model is never asked to *write* an answer; it produces a distribution over a fixed set of candidate tokens, and that distribution is the output.

This is the mechanism underneath every [[system-one-model|System One model]], and it is what separates them from [[structured-outputs|structured output]] and [[constrained-decoding|constrained decoding]], which still generate tokens and merely restrict which ones are legal.

The technique is old. [[cloze-scoring]] carries the lineage — in particular that this exact recipe is the [[mmlu|MMLU]] reference implementation from 2020.

## The recipe

1. **Declare the answer space.** Every allowed answer is enumerated in advance.
2. **Make each answer one token.** Multi-token labels are remapped — `moderation_spam` becomes `A`, or `yes`/`no`, or `0`/`1`/`2` — with the legend in the prompt. One token per answer means the position of everything else is fixed.
3. **Run one forward pass.**
4. **Read the logits at the answer position** and softmax over only the candidate tokens.
5. **Derive confidence from the distribution** — typically entropy-based. [[openjev]] uses `1 − H(p)/ln K`.

Two properties follow for free. An answer **cannot go off-schema**, because only legal tokens are ever scored — there is no JSON to parse and no repair path. And the confidence is the model's own distribution rather than a number the model *says* about itself.

> [!note] That second property is narrower than it sounds
> Reading the distribution beats verbalized self-report **for base models on bounded-candidate tasks** — the pre-trained GPT-4 is well calibrated on MMLU, and RLHF degrades it ([arXiv:2303.08774](https://arxiv.org/abs/2303.08774)). But for RLHF'd models on open-ended QA the comparison reverses: verbalized confidences are often *better* calibrated than conditional probabilities ([arXiv:2305.14975](https://arxiv.org/abs/2305.14975)). The advantage is real in this regime, not universal.

## Three implementations

The pattern is architecture-agnostic, which is why it appeared in three model families within a week of Jev's announcement:

| | Autoregressive ([[nimble]]) | Diffusion ([[openjev]]) | Encoder ([[laya]]) |
|---|---|---|---|
| Where the answer lives | next-token position | a masked canvas slot | a `[MASK]` per option |
| Fields per pass | one on CUDA, parallel on [[mlx\|MLX]] | all slots in one pass | one per option |
| Needs fine-tuning? | yes, [[lora\|LoRA]] | no — read the base model | yes |
| Typical latency | 106 ms (H100) | 94 ms p50 | **32.8 ms p50** |

The encoder route won on the merits. A 421M ModernBERT is 7.8× faster than Jev and competitive on accuracy, which is why the diffusion route looks in hindsight like a launch-week artifact rather than the destination.

> [!success] Resolved contradiction: "all slots in one pass" is a liability, not an advantage
> The diffusion route's apparent edge — every answer slot attends to the full context and to the other slots in a single bidirectional pass — is contradicted by every measurement available.
>
> dLLM-SetScore ([arXiv:2608.14649](https://arxiv.org/abs/2608.14649), a month *before* Jev) found that putting all labels on one canvas produces severe **slot-position asymmetry**: the first answer slot is predicted positive on **99.4%** of GoEmotions and **100%** of Reuters examples. Its fix is per-label scoring, one slot per pass — the configuration this table lists as the autoregressive route's handicap. Independently, [[open-jev-diffusiongemma-harness-github-2026|open-jev]] scored **88.4% with one question per canvas and 86.1% grouped**, on the same 337 questions with the same model.
>
> Two unrelated lines of evidence, same direction. [[jev|Jev]] and [[nimble]] both isolate questions by construction.
>
> **Resolved 2026-09-29:** the claimed advantage is withdrawn. Two independent measurements agree, and the size-dependence below explains when the cost matters: for large models it is small.

**The cost shrinks with scale.** On the autoregressive side, packing several questions into one sequence changed **37% of answers for a 0.6B model but only 3.9% for a 27B model**, with packed and unpacked accuracy statistically indistinguishable at 27B (open-alternative-jev, 2026-09-22). Cross-question interference is a small-model problem. Large readers can pack questions to share the state's prefill; small ones should read one question per pass.

## Why it is fast

One prefill plus one forward pass, with **zero decode steps**. Generation cost normally scales with output length; here the output is a single token position per question, so latency is dominated by the prompt. That is the whole basis for the order-of-magnitude gaps against generative models answering the same questions (2,792–2,896 ms). See [[arithmetic-intensity]].

This is arithmetic, not insight — there is nothing to decode, so decoding is free.

## Limits

> [!warning] Probabilities normalized over a supplied set are not correctness rates
> Softmax over the candidate tokens forces the probabilities to sum to 1 across *the answers you supplied*. If the true answer is not among them, the model still returns a confident-looking distribution. [[nimble|Nimble]]'s documentation is explicit: 0.9 does not mean right 90% of the time, and if "none of these" is possible you must add it. Validate any threshold on your own data.

**Confidence is not in calibratable units.** `1 − H(p)/ln K` is a *dispersion* statistic — how peaked the distribution is — not a probability of correctness. Expected Calibration Error is computed on max-probability versus empirical accuracy, so a reliability diagram cannot be drawn from normalized entropy without a mapping. This may be part of why no implementation ships a calibration curve.

**The readout inherits known biases.** Remapping answers onto symbols dodges surface form competition, but swaps it for contaminants of the same shape: option-order sensitivity moves performance 13–75% by benchmark ([arXiv:2308.11483](https://arxiv.org/abs/2308.11483)); models carry a priori preferences for specific option IDs, for which PriDe-style debiasing is the standard fix ([arXiv:2309.03882](https://arxiv.org/abs/2309.03882)); and even reading `"A"` versus `" A"` moves accuracy up to 11% ([arXiv:2509.15020](https://arxiv.org/abs/2509.15020)). None of the 2026 implementations mention debiasing.

**Multi-option probabilities can saturate.** Asked to choose between options with no evidence either way, [[jev|Jev]]'s `choice` gave 0.83–0.93 to one of them, and a stated 55/45 base rate came back as 0.90–0.996. A softmax over competing options behaves like a step function. **Asking one yes/no question per option and normalising** tracked the true targets far better, cutting Brier from 0.067 to 0.003 in that audit. It costs one read per option, and it is the better default when the probability itself will drive a decision.

**Averaging over option orderings is cheap and effective.** Two orderings took a frozen Qwen3.6-27B to 75.5% accuracy with **ECE 0.0075** on typed-decisions, against Jev's measured ECE of 0.144 on the same benchmark. Reversing the order cost 4 points at 4B but only 1.7 at 27B.

**Symbol logits may not be answer logits.** MCQA appears to be two stages — pick a winner in content space, then *bind* it to a symbol near emission ([arXiv:2601.03914](https://arxiv.org/abs/2601.03914)). Confidence read off symbol logits is partly measuring binding reliability.

Other constraints are structural: the answer space must be small enough to enumerate and map to single tokens (Nimble caps enums at 26); no rationale is produced, so a decision cannot be explained to an auditor from its output; and independently scored fields cannot be mutually consistent by construction.

## Related

- [[cloze-scoring]] · [[system-one-model]] · [[jev]] · [[nimble]] · [[openjev]] · [[laya]] · [[diffusiongemma]] · [[calibration]] · [[structured-outputs]] · [[constrained-decoding]] · [[logit-lens]] · [[independent-jev-calibration-audits-2026]]

## References

- [[system-one-ecosystem-sweep-2026-09-28]] · [[structured-generation-mode-for-diffusiongemma-mastracci-2026]] · [[bespoke-nimble-github-2026]] · [[introducing-system-one-models-and-jev-typesafe-2026]]
- [[independent-jev-calibration-audits-2026]] · [[constrained-canvas-vs-free-denoise]]
