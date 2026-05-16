---
title: Structured Outputs
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - vllm-tool-calling-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Structured Outputs

**Structured outputs** is the practice of forcing an LLM's generation to conform to a machine-readable schema — most commonly JSON matching a given [[json-schema]], but also regex, context-free grammars, or Lark grammars. It replaces the brittle pattern of "ask the model to produce JSON in the prompt, then parse and hope" with a generation-time guarantee.

## Implementation: constrained decoding

Structured outputs are typically implemented via [[constrained-decoding]]: the engine compiles the target grammar (JSON schema, regex, CFG) into a finite-state machine, and at each decoding step masks logits for any token that would lead the FSM into an invalid state before sampling. The model literally cannot emit a non-conforming token.

Major backends in 2025:

- **xgrammar** — default in [[vllm]], SGLang, and TensorRT-LLM. <40 µs/token; up to ~100× faster than naive grammar-constrained methods. Splits the vocabulary into context-independent tokens (precomputable) and context-dependent tokens (resolved at runtime).
- **outlines** — earlier widely adopted library; regex / JSON-schema / Lark.
- **lm-format-enforcer** — alternative with similar coverage.
- **llguidance** — Microsoft's guidance-language runtime.

## OpenAI's `strict: true` mode

OpenAI exposes structured outputs through:
- `response_format: { type: "json_schema", json_schema: { ..., strict: true } }` for free-form responses.
- `tools: [{ ..., function: { ..., strict: true } }]` for tool-calling.

With `strict: true`, OpenAI applies constrained decoding so the returned JSON is guaranteed schema-valid. Notably, [[vllm]] does **not** implement `strict: true` in auto tool-choice mode (a documented limitation) — it only applies constrained decoding when `tool_choice` names a function or is `"required"`.

## What it guarantees — and what it doesn't

- **Guaranteed**: syntactic validity — the output parses, all required fields are present, types match, enums are respected.
- **NOT guaranteed**: semantic correctness — the model can still return `{"city": "Mars", "temperature_f": 9999}` for a weather query. Schema enforcement is a parser-friendliness contract, not a fact-checking one.

## Performance implications

- **FSM compilation overhead**: first call with a new schema pays a one-time cost (often several seconds for complex schemas). Subsequent calls hit a cache.
- **Throughput cost**: constrained decoding adds per-token mask computation. xgrammar's design makes this nearly free for context-independent tokens, hence the ~100× speedup vs older approaches.
- **Streaming**: structured outputs are compatible with token-by-token streaming — the FSM state advances per token just like an unconstrained decoder.

## Use cases

- **[[function-calling]] / tool calling** — the canonical use; arguments must validate against the function's parameter schema. See [[tool-call-parser]] for the extraction half of the loop.
- **API extraction** — pulling structured records out of unstructured text.
- **Code generation** — constraining to a target language grammar.
- **Agent state** — forcing the model's "next action" emission into a finite vocabulary of action tags.

## Cross-references

- Underlying mechanism: [[constrained-decoding]].
- Schema language: [[json-schema]].
- Extraction counterpart: [[tool-call-parser]].
- Use case: [[function-calling]].
