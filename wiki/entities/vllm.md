---
title: vLLM
type: entity
created: 2026-05-16
updated: 2026-05-16
sources:
  - vllm-tool-calling-2025.md
status: complete
importance: high
tags:
  - product
---

# vLLM

**vLLM** is a high-throughput, memory-efficient inference and serving engine for large language models. It originated at [[uc-berkeley]] (Sky Computing Lab) and is now a widely deployed open-source project used as a default serving layer for self-hosted LLMs.

## What it is

vLLM accepts a model checkpoint (Hugging Face format or compatible) and exposes an OpenAI-compatible HTTP API (`/v1/chat/completions`, `/v1/completions`) so existing OpenAI-SDK clients can talk to it without code changes. It is designed for production serving — multi-tenant, high-QPS, GPU-resident.

## Key features

- **PagedAttention** — vLLM's signature contribution: manages the [[kv-cache]] in fixed-size blocks (analogous to OS virtual memory paging), eliminating internal fragmentation and enabling much higher batch sizes on the same hardware.
- **Continuous batching** — requests join and leave the running batch every decoding step, rather than waiting for a static batch to finish. Dramatically improves GPU utilization under variable-length workloads.
- **OpenAI-compatible API** — drop-in replacement for OpenAI endpoints, including streaming and tool calling.
- **Quantization support** — AWQ, GPTQ, FP8, INT4 paths.
- **Speculative decoding** — supports draft-model and Medusa-style speculation for latency reduction.
- **Distributed inference** — tensor parallelism and pipeline parallelism for multi-GPU / multi-node serving.

## Tool-calling support

vLLM ships [[function-calling]] via ~25 model-family-specific parsers (see [[tool-call-parser]]). Activated with `--enable-auto-tool-choice --tool-call-parser <name>` at server start. Parsers cover Llama 3.x/4, Mistral, Hermes (Nous Research) / Qwen, IBM Granite, DeepSeek v3/v3.1, Kimi K2, GLM 4.5/4.7, FunctionGemma, Salesforce xLAM, and more.

## Structured output backends

For schema-enforced generation, vLLM integrates three [[constrained-decoding]] backends:

- **xgrammar** — default as of 2025; <40 µs/token, up to 100× faster than naive FSM approaches.
- **outlines** — earlier mainstream backend; regex / JSON-schema / Lark grammars.
- **lm-format-enforcer** — alternative with similar feature set.

Selected via `--guided-decoding-backend`.

## Position in the inference-engine landscape

- **vs SGLang** — SGLang emphasizes RadixAttention (prefix caching) and a structured-programming front-end; vLLM emphasizes throughput and OpenAI compatibility. Both share xgrammar as a backend.
- **vs TensorRT-LLM** — NVIDIA's closed-source-leaning engine; lower-level, often faster on NVIDIA hardware but harder to operate. Also uses xgrammar as default in 2025.
- **vs llama.cpp** — CPU/GGUF-focused; targets single-user local inference, not multi-tenant serving.
- **vs Hugging Face TGI** — comparable scope, both OpenAI-compatible; TGI historically tighter integration with the Hugging Face stack.

## Cross-references

- API surface mirrors [[openai-api]] (chat completions, tools, tool_choice).
- Closely tied to [[function-calling]] and [[structured-outputs]] as runtime concepts.
- Origin: [[uc-berkeley]].
