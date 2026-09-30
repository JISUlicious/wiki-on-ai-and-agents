---
title: DiffusionGemma
type: entity
created: 2026-09-21
updated: 2026-09-29
sources:
  - https://deepmind.google/models/gemma/diffusiongemma/
  - neither-parallel-nor-sequential-diffusiongemma-asaria-2026.md
  - how-transparent-is-diffusiongemma-engels-2026.md
  - structured-generation-mode-for-diffusiongemma-mastracci-2026.md
  - open-jev-diffusiongemma-harness-github-2026.md
family: gemma
parameters: 25.2B total / 3.8B active (marketed as 26B-A4B)
release_date: 2026-06-10
status: complete
importance: high
tags:
  - model
---

# DiffusionGemma

Open diffusion language model released by [[google|Google]] on 2026-06-10 under Apache 2.0. Instead of emitting one token at a time left to right, it denoises a fixed-size **canvas** of masked positions over repeated forward passes, committing the positions it is most confident about at each step.

Its significance turned out to exceed the speed pitch it launched with: in September 2026 it became the substrate for open reimplementations of [[jev|Jev]], because a model that fills masked slots can be asked to fill exactly one slot per question and then be *read* rather than run. See [[logit-scoring]].

## Architecture

- 25.2B total parameters, 3.8B active — a [[mixture-of-experts|mixture of experts]] routing to 8 of 128 experts plus 1 shared expert, over 30 layers. Distributed and marketed as **26B-A4B**, which is rounding.
- ~550M vision encoder; multimodal input, which carries over to image-based decisions.
- 256K context with a 1024-token sliding window.
- 256-token canvas. Serving stacks routinely reconfigure this — [[openjev]] defaults to 64.
- A 48-step denoising budget, temperature annealing 0.8 → 0.4, `entropy_bound` 0.1, adaptive stop at 0.005.
- ~18 GB in NVFP4, ~16 GB in 4-bit MLX — within reach of a 24GB consumer GPU or a 16GB+ Mac. See [[quantization-performance]], [[edge-inference]].

## How it actually decodes

The marketing and the model card both mislead, in opposite directions, and [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026|Asaria et al.]] instrumented the sampler to settle it. DiffusionGemma is **locally parallel with a partial global left-to-right drift** — neither the "256 tokens at once" of Google's page nor secretly autoregressive.

| Regime | Content tokens per accept-call | Accept-calls used |
|---|---:|---:|
| GSM8K (math) | 25.5 | — |
| HumanEval | 23.0 | — |
| MBPP | 19.8 | — |
| Open-ended | 12.6 | — |
| **JSON (structured)** | **8.5** | — |
| Factual recall | 5.8 | — |
| *Across regimes* | *5.8–25.5* | *3.3–17.1 of 48* |

Commits land at an entropy of 0.002–0.012 nats, one to two orders of magnitude inside the sampler's own 0.1 bound. Tie-aware Kendall τ_b runs 0.430–0.604 — far below strict autoregression (1.0) and below a block-sequential control (0.94–0.96), so there is real sub-block disorder. Block size is an artifact of the ruler: τ_b rises smoothly 0.59→0.91 as bins grow 4→64 with **no jump at 16**, so no privileged architectural block exists.

> [!success] Resolved contradiction: the published decoding figures are not measured
> Google's DeepMind page says the model generates "256 tokens in parallel with each forward pass" (>1,000 tok/s on one H100, 4× faster); the model card's commonly-cited "15–20 tokens per forward pass" and "adaptive stopping typically 12–16 steps" appear in **neither** arXiv paper. Direct instrumentation gives 5.8–25.5 tokens per accept-call over 3.3–17.1 calls — short regimes fall well below 12 steps. [[how-transparent-is-diffusiongemma-engels-2026|Engels et al.]] restate the 12–16 figure without measuring it. The 256 number is canvas size, not commit rate.
>
> **Resolved 2026-09-29:** in favour of direct measurement. [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026]] instruments the shipped sampler; the vendor figures are unmeasured restatements. 256 is the canvas width, not the commit rate.

Commits are also not frozen: 4,524 un-accept events across 600 generations, with only 220/600 fully monotone.

## Generation quality: the tradeoff

Against [[gemma|Gemma]] 4 26B A4B, its autoregressive sibling at the same size:

| Benchmark | DiffusionGemma | Gemma 4 26B A4B |
|---|---:|---:|
| MMLU Pro | 77.6 | 82.6 |
| AIME 2026 | 69.1 | 88.3 |
| GPQA Diamond | 73.2 | 82.3 |
| MMMU Pro | 54.3 | 73.8 |

The 19-point [[aime|AIME]] gap is the telling one: multi-step reasoning suffers most when tokens commit in parallel, as you would predict when later steps depend on earlier ones. Google concedes diminishing returns under high-QPS cloud workloads — batching already saturates those GPUs. The win is single-stream, latency-sensitive, local inference, where [[arithmetic-intensity|arithmetic intensity]] binds. Google also highlighted a Sudoku fine-tune, where revising earlier positions after seeing later ones is the whole point.

## As a System One model

A diffusion model that fills masked slots can be given a canvas in which **the only masked tokens are the answer slots**, one single token per question, every allowed answer a one-token label. One read-only pass returns a distribution over each slot — and that distribution *is* the answer. Nothing is generated, so nothing can go off-schema, and confidence comes from the model's own distribution rather than a number it reports about itself. See [[logit-scoring]].

There is a mechanistic reason this works especially well. Of every regime measured, **structured JSON is the one where commit order essentially does not matter**: τ_b = −0.044 [−0.086, −0.00], statistically indistinguishable from order-independent, with up to 72% of token pairs committing in the same call. The left-to-right drift that hurts chained reasoning is absent precisely where typed decisions live.

Three efforts landed within days of Jev's 2026-09-15 announcement:

- [[structured-generation-mode-for-diffusiongemma-mastracci-2026|vLLM PR #57250]] — seeded canvases, read-only steps, step caps, and a prototype `/v1/systemone` server.
- [[openjev]] — a Jev **wire-compatible** decision server, with NVIDIA and [[mlx|MLX]] backends.
- [[open-jev-diffusiongemma-harness-github-2026|open-jev]] — 88.4% against Jev's 90.8% on Jev's own published cases, with no fine-tuning.

Google's Gemma account promoted the pattern, framing "DiffusionGemma as Jev" as a demonstration of non-autoregressive architectures evaluating structured choices in a single parallel pass. [[rationale-behind-diffusiongemma-as-jev]] sets out the full argument — and why the route nevertheless lost to encoders like [[laya]]. By 2026-09-28 the DiffusionGemma servers ranked 11th and 30th of 91 on [[jevbench|JevBench]], and nobody had ported the pattern to any other diffusion model. The upstream vLLM support merged on 2026-09-22 all the same.

> [!warning] Confidence is informative on reasoning, null on recall
> Commit confidence predicts correctness on math (AUROC 0.749 [0.602, 0.879], with monotone tertiles 1.00/0.667/0.618) but is **uninformative on factual recall** (AUROC 0.471 [0.383, 0.544] — no better than chance). Since reading a decision off the canvas *is* reading commit confidence, this is a direct caveat for every DiffusionGemma-based System One server: the probabilities may be meaningful for judgment-type questions and meaningless for questions that turn on recalled facts. No one has yet measured [[calibration]] separately by question type.

## Related

- [[google]] · [[jev]] · [[openjev]] · [[nimble]] · [[laya]] · [[system-one-model]] · [[logit-scoring]] · [[cloze-scoring]] · [[diffusion-language-model]] · [[mixture-of-experts]] · [[edge-inference]] · [[arithmetic-intensity]] · [[latent-reasoning]] · [[mlx]] · [[vllm]] · [[calibration]]

## References

- [DiffusionGemma — Google DeepMind](https://deepmind.google/models/gemma/diffusiongemma/)
- [[neither-parallel-nor-sequential-diffusiongemma-asaria-2026]]
- [[how-transparent-is-diffusiongemma-engels-2026]]
- [[structured-generation-mode-for-diffusiongemma-mastracci-2026]]
- [[open-jev-diffusiongemma-harness-github-2026]]
