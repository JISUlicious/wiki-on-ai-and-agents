---
title: SGLang
type: entity
created: 2026-09-28
updated: 2026-10-05
sources:
  - system-one-ecosystem-sweep-2026-09-28.md
status: draft
importance: medium
tags:
  - product
---

# SGLang

Open-source LLM serving engine from the LMSYS / sgl-project community, a peer of [[vllm|vLLM]]. It is referenced across the wiki for structured-output serving via xgrammar (see [[constrained-decoding]]) and for RadixAttention [[prefix-caching|prefix caching]] (see [[kv-cache]]).

## First engine with a native System One endpoint

PR [#41208](https://github.com/sgl-project/sglang/pull/41208), merged **2026-09-25** (+2,803 lines), added a `/v1/systemone` route compatible with [[jev|Jev]]'s API. It serves **any chat model**, supports up to 255 choice options by switching to two-letter labels past 26, and was validated with TypeSafe's official Python and JS SDKs on Qwen3.8-27B and Qwen3.5-35B-A3B. The 27B passed all 32 required checks of a community conformance tool.

This makes SGLang the simplest way to run a Jev-compatible service on your own hardware: a stock engine and an off-the-shelf model, with no fine-tuning and no patched fork. The answers are read off the model's logits — see [[logit-scoring]] — so calibration is still the operator's job.

At the time of writing, vLLM's equivalent server remains example code rather than a built-in endpoint; a native-endpoint request was opened on 2026-09-28.

## Related

- [[vllm]] · [[system-one-model]] · [[logit-scoring]] · [[jev]] · [[structured-outputs]] · [[constrained-decoding]]

## References

- [[system-one-ecosystem-sweep-2026-09-28]]

## Related 2026 sources

- [[ollama]] — Ollama: local model runner on llama.cpp/GGUF; native /v1/systemone decision models since v0.35.0
