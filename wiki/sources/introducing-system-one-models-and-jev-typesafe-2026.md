---
title: "Introducing System One Models and Jev"
type: source
created: 2026-09-21
updated: 2026-09-21
sources: []
authors:
  - TypeSafe AI
year: 2026
venue: "TypeSafe AI company blog and product docs"
status: complete
importance: medium
tags:
  - 2026
---

# Introducing System One Models and Jev

**Source**: [typesafe.ai/blog/introducing-system-one-models-and-jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev), with product facts from [docs.typesafe.ai/models](https://docs.typesafe.ai/models). Announced **2026-09-15**; early access behind a waitlist. A vendor announcement, not a paper.

## The claim

[[typesafe-ai|TypeSafe AI]] announces a model class it calls **System One models**, "built to make fast, structured decisions that software can use directly," and [[jev|Jev]] as the first one. The input is program state plus questions whose answer space is declared in advance; the output is typed values with probabilities and no text. The name follows Kahneman's fast-thinking system, and the model is named after the economist William Stanley Jevons.

## What the docs actually pin down

| | |
|---|---|
| Endpoint and model | `POST /v1/systemone`, `jev-1.13.0`, aliases `jev-latest` and `jev-preview` |
| Context | 64k tokens per request; 32k for the state plus the longest question |
| Input | text only: a string, a JSON object, or an array of text values |
| Price | $0.042 per million input tokens; output tokens free |
| Rate limits | 250,000 tokens per second, 1,200 requests per minute |

Third-party guides describe three question forms: a choice among declared options, a 0–100 score against a rubric, and a yes/no probability. The docs themselves do not enumerate them.

## Training

**RLCD**, "Reinforcement Learning for Calibrated Decisions," which the company frames against RLHF (trained on what humans prefer) and RLVR (trained on what a checker can verify). RLCD is described as optimizing for "epistemically honest probabilities." No objective, architecture, parameter count, base model or data description is published.

## The numbers, all vendor-reported

| Measure | Jev | Comparison |
|---|---|---|
| Latency | 70–500 ms | 3–329 s for frontier models |
| Speed claim | 40×–200× faster; 193.6× on its own workflow evals | |
| Cost | $0.0004 per case | $0.0304 GPT-5.6 Terra, $0.1761 Opus 5; 444.6× cheaper on workflow evals |
| Accuracy, 4 workflows | 67.8% | 67.9% Terra, 74.1% Sol, 73.1% Opus 5 |
| Type errors | 0% | 0.58% Terra, 5.73% Opus 5, 45.5% Haiku 4.5 |

> [!warning] Nothing here has been independently reproduced
> There is **no paper, model card or calibration curve**, which several write-ups note independently. The evaluation workflows "were made by individuals on our model capabilities team, so some bias could exist"; the comparison is "biased towards OpenAI and Anthropic's models"; the short input "paints our model in an advantageous light"; and the 0% type-error figure is guaranteed by the schema rather than measured, which the company states plainly: "our number is not empirical." On pricing: "We can't prove it isn't subsidized."
>
> One reviewer reported TypeSafe's own adapter and the hosted API returning **different confidences for the same input, 0.596 against 0.76**, with no published account of how either is computed. Treat the accuracy parity as promising rather than settled.

## Why it matters anyway

The interesting part is not the benchmark table. It is a commercial model whose *training objective* is calibration rather than preference, sold per decision rather than per token, with the answer space fixed by the caller. Whether RLCD contributes anything beyond the architecture and serving stack cannot be told from what is published. The ideas underneath are old and are set out on [[calibration]] and [[system-one-model]].

## Related

- [[jev]] · [[typesafe-ai]] · [[system-one-model]] · [[calibration]]
- [[constrained-decoding]] — the same validity-by-construction guarantee, at the decoding layer of an ordinary LLM.
- [[llm-as-a-verifier-kwok-2026]] — evidence that the probability signal is already in the logits and discrete decoding discards it.

## References

- [Introducing System One Models and Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev) · [TypeSafe docs](https://docs.typesafe.ai/models)
