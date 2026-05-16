---
title: "vLLM Tool Calling Documentation"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - vllm-tool-calling-2025.md
year: 2025
introduces:
  - "[[vllm]]"
  - "[[tool-call-parser]]"
  - "[[constrained-decoding]]"
tags:
  - 2025
status: complete
importance: high
---

# vLLM Tool Calling Documentation

**Source URL**: <https://docs.vllm.ai/en/latest/features/tool_calling/>

## Summary

Reference documentation for [[vllm]]'s tool-calling subsystem: how the inference engine bridges between heterogeneous model output formats (JSON, XML-wrapped JSON, pythonic syntax, custom delimiters) and a standardized, OpenAI-compatible `tool_calls` API. The doc enumerates ~25 parser implementations (one per model family), describes the streaming vs non-streaming extraction interface, and explains how parsers integrate with structured-output backends ([[constrained-decoding]] via xgrammar / outlines / lm-format-enforcer) for schema-enforced generation.

## Key Points

- **Three components** combine to deliver tool calling: a [[tool-call-parser]] (converts raw model text to `ToolCall` objects), a chat template (handles the `tool` role and reconstructs assistant tool calls), and a structured-output backend (constrains decoding to enforce schema).
- **~25 parsers** ship with vLLM, each tailored to a model family — there is no industry standard for how a model emits a tool call at the token level, so each family (Llama, Mistral, Hermes/Qwen, Granite, DeepSeek, Kimi, GLM, FunctionGemma, xLAM, ...) requires bespoke extraction logic.
- **Auto mode vs constrained mode**: with `tool_choice="auto"` vLLM parses raw model output without enforcing the schema — argument correctness depends on the model. With `tool_choice="function_name"` or `"required"` vLLM uses [[constrained-decoding]] to guarantee schema-valid JSON. Notably, vLLM does **not** implement OpenAI's `strict: true` mode (which would constrain even auto mode).
- **Backends**: xgrammar (default, 2025+) achieves <40 microseconds per token and up to 100× speedup over traditional grammar-constrained methods by splitting tokens into context-independent vs context-dependent sets. Alternatives: outlines, lm-format-enforcer.
- **Streaming interface**: parsers implement both `extract_tool_calls` (full output) and `extract_tool_calls_streaming` (incremental) — the latter enables function name completion to trigger execution before all arguments arrive.
- **OpenAI-compatible API**: clients submit a standard `tools` array (JSON Schema) plus optional `tool_choice`; responses carry `tool_calls` with `function.name` + `function.arguments` JSON-string, identical in shape to OpenAI's chat completions.
- **Plugin system**: custom parsers can be registered via `ToolParserManager.register_lazy_module` without modifying core vLLM.

## Entities Mentioned

- [[vllm]]
- [[openai]] (API compatibility surface)
- [[uc-berkeley]] (vLLM origin)

## Concepts Mentioned

- [[tool-call-parser]]
- [[constrained-decoding]]
- [[structured-outputs]]
- [[function-calling]]

## References

- Source URL: <https://docs.vllm.ai/en/latest/features/tool_calling/>
- xgrammar paper / backend
- OpenAI chat completions API (the surface vLLM mirrors)
