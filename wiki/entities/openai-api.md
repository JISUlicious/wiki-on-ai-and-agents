---
title: OpenAI API
type: entity
created: 2026-05-16
updated: 2026-05-16
sources:
  - openai-function-calling-2025.md
status: complete
importance: high
tags:
  - product
---

# OpenAI API

The **OpenAI API** is the developer-facing product offered by [[openai]] (the company) for programmatic access to its models (GPT-3, [[gpt-4]], GPT-5, and successors) and built-in capabilities. It is distinct from consumer products like ChatGPT: the API is a paid HTTPS interface intended for application developers, not end users.

This page covers the API as a *product*. For the company, see [[openai]]. For the unified March 2025 endpoint, see [[responses-api]].

## Key Endpoints

- **Chat Completions** (`/v1/chat/completions`) — the original conversational endpoint; takes a list of `messages` and returns an assistant message. Tools are nested under a `"function"` wrapper. Still widely used and supported.
- **Responses API** (`/v1/responses`, March 2025) — the unified successor to Chat Completions + Assistants. Takes `input` items and returns an `output` array of structured items (text, function calls, tool results, reasoning traces). Flat tool definitions (no `"function"` wrapper). See [[responses-api]].
- **Assistants API** — earlier abstraction for stateful, tool-using assistants with persistent threads. Effectively subsumed by the Responses API.
- Other endpoints: embeddings, fine-tuning, batch, files, audio (Whisper, TTS), image (DALL·E / `gpt-image-1`), realtime (voice).

## Function Calling

The OpenAI API supports **[[function-calling]]** across both Chat Completions and the Responses API. Developers register tools as [[json-schema|JSON Schema]] objects; the model emits structured tool calls that the application executes and returns to the model with a matching `call_id`. The canonical 5-step loop is documented in [[openai-function-calling-2025]].

Features layered on function calling:

- **Parallel tool calls** — models can invoke multiple tools in one response.
- **Tool choice constraints** — `auto`, `required`, forced, or `allowed_tools` (subset without removing others, for prompt-cache friendliness).
- **Custom tools with grammars** — Lark or Regex context-free grammars constrain free-form text outputs.
- **Tool search** (gpt-5.4+) — defers tool loading for large tool catalogs.

## Structured Outputs

Strict-mode JSON schemas (`strict: true`) trigger **[[structured-outputs]]** — constrained decoding that guarantees the model's output matches the schema. Requires `additionalProperties: false` and that all fields are `required` (optional fields modeled as `["type", "null"]` unions).

## Built-in Tools

Beyond user-defined functions, the API exposes OpenAI-hosted tools that can be invoked the same way:

- **Web search** — live search-grounded answers.
- **Code execution** — sandboxed Python interpreter.
- **File search** — retrieval against developer-uploaded files (RAG primitive).
- **[[mcp|MCP]] server integration** — call out to Model Context Protocol servers for external tools/data.

Parallel tool calls do not work with built-in tools.

## Historical Evolution

1. **Completions API** (2020) — single-shot text completion (`/v1/completions`); now legacy.
2. **Chat Completions** (2023) — message-based, the workhorse during the GPT-3.5/4 era. Function calling added mid-2023.
3. **Assistants API** (2023) — stateful threads, file search, code interpreter.
4. **Responses API** (March 2025) — unification of Chat Completions and Assistants; event-based streaming model; first-class function calls and reasoning items in the output array.

The trajectory has been toward richer, more agent-shaped primitives: from "complete this text" to "here are tools and an input, return a stream of structured items including tool calls, reasoning, and text."

## Relation to OpenAI (the company)

[[openai]] is the legal and research entity; the OpenAI API is one of its commercial products (alongside ChatGPT and enterprise offerings). API revenue funds and is funded by the same model training that produces ChatGPT — the API typically exposes models shortly after or alongside their consumer launch.

## See Also

- [[responses-api]] — the unified March 2025 endpoint
- [[function-calling]] — the core tool-use mechanism
- [[structured-outputs]] — schema-guaranteed outputs
- [[mcp]] — external tool protocol available as a built-in integration
- [[openai-function-calling-2025]] — source document for this page
