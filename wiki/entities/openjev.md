---
title: OpenJev
type: entity
created: 2026-09-21
updated: 2026-09-21
sources:
  - https://github.com/razorback16/openjev
builds-on:
  - [[diffusiongemma]]
  - [[vllm]]
  - [[mlx]]
status: complete
importance: medium
tags:
  - product
---

# OpenJev

Open-source **System One decision server** built on [[diffusiongemma|DiffusionGemma]] 26B-A4B, created 2026-09-18 — three days after [[jev|Jev]]'s announcement — and under active development (253 stars as of 2026-09-21). Apache 2.0, including the weights. It is independent and not affiliated with [[typesafe-ai|TypeSafe AI]].

Its distinguishing move is **wire compatibility**: it serves `POST /v1/systemone` with Jev's request and response shapes, accepts the model names `jev-latest` and `jev-preview` so TypeSafe SDK defaults resolve, and mirrors Jev's error shapes "checked against the live API." TypeSafe's own `typesafe-sdk` runs against it unchanged, with only a base-URL swap. It adopts Jev's question vocabulary wholesale — `noul` (yes/no), `choice`, and `score`.

## Mechanism

It builds a canvas in which the only masked tokens are the answer slots, one token per question, and takes a single **read-only** pass. The returned distribution over each slot is the answer — see [[logit-scoring]]. Labels are single tokens (`yes`/`no`, `A`/`B`/`C`, `0`/`1`/`2`); the model never writes into the slots. Confidence is `1 − H(p)/ln K`. If any slot has entropy above 0.1, it re-reads with fresh noise up to four times and averages. Question ids are hidden from the model, which sees only `q1`, `q2`, `q3`.

It also serves ordinary text generation on an OpenAI-compatible `/v1/chat/completions` at no extra GPU memory, since it is the same model.

## Backends

| | vLLM | [[mlx\|MLX]] |
|---|---|---|
| Hardware | NVIDIA GPU, 24 GB+ | Apple silicon, ~16 GB free |
| Weights | `nvidia/diffusiongemma-26B-A4B-it-NVFP4`, ~18 GB | `mlx-community/diffusiongemma-26B-A4B-it-4bit` |
| Concurrency | up to 64 reads in flight | one at a time |

The [[mlx|MLX]] path runs the model in-process, needs neither Docker nor vLLM, and answers with the same prompts, canvases and seeds as the GPU path. On [[apple|Apple silicon]] a 3-question request takes about **0.2–0.4 s on an M3 Ultra** and **~0.39 s on an M4 Max** with 4-bit weights; 16 concurrent requests finish at roughly 4 req/s. Reads run one at a time, so it suits local use rather than serving.

On an RTX PRO 6000 Blackwell using 38% of the GPU, 3 questions per request, cache-busted states:

| Concurrency | req/s | p50 | p95 |
|---:|---:|---:|---:|
| 1 | 10.7 | 94 ms | 94 ms |
| 16 | 43.3 | 367 ms | 369 ms |
| 32 | 51.7 | 545 ms | 618 ms |
| 64 | 57.4 | 760 ms | 1109 ms |

A 94 ms p50 at concurrency 1 undercuts Jev's reported 246.7 ms median through its own API, though on different hardware and question mixes.

> [!warning] It depends on an unmerged patch
> The [[structured-generation-mode-for-diffusiongemma-mastracci-2026|vLLM PR #57250]] it needs has not merged. The request fields are provisional (`vllm_xargs`), so the project pins a rebased fork carrying the PR head plus three crash fixes: an image-request fix, a dtype mismatch when torch.compile hits its recompile limit and falls back to eager, and logprobs stashed at different steps failing to join when reads and generations share a batch. This is pre-release infrastructure.

The README also states plainly that "answer quality is the quality of DiffusionGemma 26B-A4B used in this mode" and tells users to evaluate on their own tasks. It publishes no accuracy or [[calibration]] figures of its own — for those, see [[open-jev-diffusiongemma-harness-github-2026]].

## Related

- [[jev]] · [[diffusiongemma]] · [[system-one-model]] · [[logit-scoring]] · [[nimble]] · [[mlx]] · [[vllm]] · [[edge-inference]] · [[calibration]]

## References

- [https://github.com/razorback16/openjev](https://github.com/razorback16/openjev)
- [[structured-generation-mode-for-diffusiongemma-mastracci-2026]]
