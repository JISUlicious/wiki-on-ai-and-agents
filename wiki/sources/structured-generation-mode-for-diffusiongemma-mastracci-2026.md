---
title: "Structured generation mode for DiffusionGemma (Jev-like) — vLLM PR #57250"
type: source
created: 2026-09-21
updated: 2026-09-21
sources:
  - https://github.com/vllm-project/vllm/pull/57250
authors:
  - Matt Mastracci
year: 2026
venue: "vLLM pull request (open, unmerged)"
introduces:
  - [[logit-scoring]]
status: complete
importance: high
tags:
  - 2026
---

# Structured generation mode for DiffusionGemma (Jev-like) — vLLM PR #57250

An open pull request against [[vllm|vLLM]] by Matt Mastracci adding the machinery to serve [[diffusiongemma|DiffusionGemma]] as a typed-decision model. This is the primary technical source for the claim that [[jev|Jev]]'s capability is reproducible on open weights.

## The claim

> DiffusionGemma (and likely other models in vLLM) contains all of the necessary components to build a Jev-like model that provides structured answers with bounded choices.

The method: fix the canvas positions dictated by the structured output format, leaving only answer slots unfixed. Ask the model for a token plus logprobs at those slots, from which the client derives entropy. If entropy fails a threshold, sample again and measure agreement across runs to get the model's actual confidence. See [[logit-scoring]].

The PR ships a prototype server for the `/v1/systemone` endpoint — the same path [[typesafe-ai|TypeSafe]] uses — rather than a standard vLLM endpoint.

## The load-bearing constraint

The approach works for *any* structured output subject to one limitation: **answer choices must each be a single token**, so the canvas does not shift. The author notes clients can always remap multi-token options — `moderation_spam` → `A`. Choices are laid out in a system prompt, and the model "correctly diffuses single-token-slot answers (or makes a guess and flags it with uncertainty)." Supported question shapes are yes/no — "noul in Jev parlance" — scale, and multiple choice, each returned with certainty and error bars.

## What vLLM was missing

Four gaps, which is the substance of the patch:

| Gap | Why it is needed |
|---|---|
| Pre-seed the canvas with fixed tokens | so only answer slots are masked |
| Read-only requests emitting the argmax canvas on the converging step, skipping the commit forward | logprobs come from that step at temperature 1 |
| Cap the number of steps | only the first step is wanted |
| Bugfix: two concurrent DiffusionGemma generations in lockstep | one converging while another commits loses its stash, failing with HTTP 500 "list index out of range" |

Five prerequisite PRs were split out on their own merits (#57414, #57416, #57417, #57462, #57589), covering stashed-logprob handling, prefill-only logit rows, `logprob_token_ids` parity with autoregressive decoding, a dynamo eager-fallback type cast, and multimodal support.

## What it does not show

The PR demonstrates **mechanism, not quality**. It contains no accuracy benchmark, no comparison against Jev, and no [[calibration]] measurement — no reliability diagram, no ECE. The author's confidence procedure (resample on high entropy, measure agreement) is a reasonable proxy but is asserted rather than validated here.

It is also unmerged as of 2026-09-21, and its request fields (`vllm_xargs`) are explicitly provisional. Downstream users such as [[openjev]] pin a rebased fork carrying additional crash fixes, which indicates the path is not yet stable. Whether "likely other models in vLLM" can do the same is a conjecture in the PR body, untested.

## Related

- [[diffusiongemma]] · [[jev]] · [[openjev]] · [[logit-scoring]] · [[system-one-model]] · [[vllm]] · [[structured-outputs]] · [[constrained-decoding]]

## References

- [vllm-project/vllm#57250](https://github.com/vllm-project/vllm/pull/57250)
