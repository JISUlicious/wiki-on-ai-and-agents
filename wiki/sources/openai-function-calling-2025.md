---
title: "OpenAI Function Calling Guide"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - openai-function-calling-2025.md
authors:
  - "[[openai]]"
year: 2025
introduces:
  - "[[openai-api]]"
  - "[[function-calling]]"
tags:
  - 2025
status: complete
importance: high
---

# OpenAI Function Calling Guide

**Source URL**: https://developers.openai.com/api/docs/guides/function-calling

## Summary

[[openai]]'s official guide to function calling (a.k.a. tool calling) — the mechanism by which OpenAI models interface with external systems beyond their training data. The guide covers both API surfaces: the older **[[chat-completions-api|Chat Completions]]** endpoint and the newer **[[responses-api|Responses API]]** (released March 2025), which unifies capabilities previously split across Chat Completions and the Assistants API.

The doc walks through the canonical 5-step workflow (provide tools → receive tool calls → execute → return results → final response), the JSON-schema tool definition format, strict mode for guaranteed schema compliance, parallel tool calls, streaming events, and advanced features like custom tools with Lark/Regex grammars and `tool_search` for large tool catalogs. It also distinguishes custom user-defined functions from OpenAI's growing set of built-in tools (web search, code execution, file search, [[mcp|MCP]] server integration).

## Key Points

- **Five-step workflow** — (1) provide model with tools, (2) receive tool calls, (3) execute application logic, (4) return results with `call_id`, (5) obtain final text response.
- **Tool definition as [[json-schema|JSON Schema]]** — required fields are `type` ("function"), `name`, `description`, `parameters`. Optional `strict: true` triggers constrained decoding for schema compliance.
- **Namespaces** group related tools (e.g., `crm` namespace with customer-lookup and order-management tools) — keeps large tool surfaces organized.
- **Two API surfaces**:
  - *Chat Completions* — tools nested under a `"function"` wrapper; responses contain a `tool_calls` array.
  - *Responses API (March 2025)* — flat tool definitions; output is an array of items including `function_call` items with `call_id`. Application returns `function_call_output` items referencing that `call_id`.
- **Parallel tool calls** — models can invoke multiple tools per response; can be disabled with `parallel_tool_calls=false`. Doesn't work with built-in tools, and is disabled in strict mode for fine-tuned models.
- **Tool choice constraints** — `auto` (default), `required` (must call at least one), forced (specific function), or `allowed_tools` (subset without removing others from schema — useful for prompt caching).
- **Strict mode** ([[structured-outputs|Structured Outputs]]) requires `additionalProperties: false` on every object and that all fields are listed as `required` (optional fields modeled as `["type", "null"]` union).
- **Streaming**:
  - Chat Completions emits delta chunks with `index` identifying which tool call is being built.
  - Responses API emits discrete events: `response.output_item.added`, `response.function_call_arguments.delta`, `response.function_call_arguments.done`.
- **Custom tools with grammars** — beyond JSON schema, tools can accept arbitrary text constrained by a context-free grammar (Lark or Regex syntax). Useful for DSLs (e.g., math expressions).
- **Tool search** (gpt-5.4+) — defers tool loading for ecosystems with many infrequently-used tools. Model first searches the catalog, then loads the relevant subset, avoiding token bloat in the system prompt.
- **Built-in tools vs custom functions** — built-in tools (web search, code execution, file search, [[mcp|MCP]] servers) access OpenAI-hosted capabilities; custom functions move data between the application and the model.
- **Best practices** — clear descriptions; "intern test" (could someone use the function from docs alone?); under ~20 tools for accuracy; combine functions that are always called sequentially.

## Entities / Concepts Mentioned

- [[openai]] — publisher of the guide and the underlying API
- [[openai-api]] — the API surface itself (Chat Completions, Responses, Assistants)
- [[gpt-4]] — example model family (the guide uses `gpt-4.1` and `gpt-5` in code samples)
- [[function-calling]] — the core concept being documented
- [[responses-api]] — the March 2025 unified API
- [[json-schema]] — the tool definition format
- [[structured-outputs]] — the constrained-decoding mechanism behind strict mode
- [[mcp]] — Model Context Protocol, available as a built-in tool integration
- [[agent-three-layer-model]] — function calling is the substrate of the model→tool→world layered pattern

## Notable Quotes

> "Function calling (a.k.a. tool calling) enables OpenAI models to interface with external systems beyond their training data."

> "The Responses API unifies capabilities previously split across Chat Completions and the Assistants API."

> "Function definitions inject as input tokens — use tool search to defer large/infrequent functions."

> "Intern test — could someone use the function with only documented information?"

## References

- Source URL: https://developers.openai.com/api/docs/guides/function-calling
- Local copy: `sources/openai-function-calling-2025.md`
- Related: [[function-calling]], [[responses-api]], [[openai-api]], [[mcp]], [[structured-outputs]]
