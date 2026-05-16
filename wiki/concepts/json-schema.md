---
title: JSON Schema
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - openai-function-calling-2025.md
  - vllm-tool-calling-2025.md
  - function-calling-llms-promptingguide.md
status: draft
importance: medium
tags: []
---

JSON Schema is an IETF-standard specification language for describing the structure and validation rules of JSON data. A schema declaratively specifies required fields, primitive and composite data types (string, number, boolean, object, array), value constraints (min/max, regex patterns, length), nested objects, and enumerated value sets. In the LLM ecosystem, JSON Schema has become the dominant interchange format for [[function-calling|LLM function calling]]: every tool definition exposed to a model is a JSON Schema describing the tool's argument shape, and the model is fine-tuned (and at inference time, often constrained) to emit arguments that validate against that schema. Strict-mode enforcement — e.g. [[openai-api|OpenAI's `strict: true`]] flag or the [[structured-outputs]] response format — guarantees schema validity via [[constrained-decoding|constrained decoding]] at the token level, eliminating an entire class of parsing failures that plague free-form tool calling.

## Where it appears

- [[function-calling]] — every tool/function definition passed to a model is encoded as a JSON Schema describing its parameters.
- [[mcp]] — MCP tool, resource, and prompt definitions all use JSON Schema for their input/output contracts.
- [[structured-outputs]] and [[constrained-decoding]] — the schema is compiled into a grammar or FSM that constrains decoding to only emit valid JSON.
- [[tool-call-parser]] — runtime parsers validate model output against the declared JSON Schema before dispatching the call.

## References

- `openai-function-calling-2025.md`
- `vllm-tool-calling-2025.md`
- `function-calling-llms-promptingguide.md`
