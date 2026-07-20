---
title: llama.cpp
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - which-quantization-kurt-2026.md
status: draft
importance: medium
---

# llama.cpp

**llama.cpp** is a C/C++ LLM inference engine built on the `ggml` tensor library, created by Georgi Gerganov. It is the backbone of local LLM inference — running on CPU, Apple Silicon (Metal), CUDA, and more — and the engine behind ollama, LM Studio, and much of the local-model ecosystem.

Its significance for this wiki is threefold:

1. **It defines the [[gguf|GGUF]] format and k-quant/i-quant schemes** that most locally-run quantized models actually use — a body of practice largely disconnected from the academic [[quantization]] literature.
2. **Its `llama-bench` tool established the `pp` / `tg` vocabulary** (prompt processing vs token generation) that makes the prefill/decode asymmetry legible. See [[quantization-performance]].
3. **It is an attack surface**: GGUF is the format targeted by [[mind-the-gap-egashira-2025]], which extends [[quantization-conditioned-backdoor|quantization-conditioned backdoors]] to nine GGUF data types.

Useful flags: `-ctk` / `-ctv` set [[kv-cache]] precision (`f16` / `q8_0` / `q4_0`); q8_0 cache requires Flash Attention.

> [!note] Draft
> Created as a supporting page during the quantization ingest. Deserves fuller treatment of the ggml project, backend coverage, and its role in the local-inference ecosystem.

## Related

- [[gguf]] · [[quantization]] · [[quantization-performance]] · [[vllm]]

## References

- [[which-quantization-kurt-2026]]
