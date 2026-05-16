---
title: Responses API
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - openai-function-calling-2025.md
status: complete
importance: high
tags:
  - api
  - 2025
---

# Responses API

The **Responses API** is [[openai]]'s unified inference endpoint, released in **March 2025**. It is part of the [[openai-api]] product and is the intended successor to both **Chat Completions** and the **Assistants API**, consolidating their capabilities into a single, more agent-shaped interface.

## What it unifies

Before March 2025, OpenAI developers chose between two surfaces:

- **Chat Completions** — stateless, message-list in / message out. Simple but increasingly awkward for agentic patterns: tools nested under a `"function"` wrapper, streaming via opaque delta chunks, no first-class slot for reasoning or built-in tool results.
- **Assistants API** — stateful threads with file search, code interpreter, and persistent context. More structured but heavier and harder to compose with custom application state.

The Responses API folds the strengths of both into one endpoint:

- Flat tool definitions (no `"function"` nesting).
- An `input` array (analogous to messages, but typed) and an `output` array of heterogeneous **items** (text, `function_call`, `function_call_output`, reasoning traces, built-in tool results).
- First-class support for [[function-calling]], [[structured-outputs]], built-in tools (web search, code execution, file search, [[mcp|MCP]] servers), and reasoning models.

## Event-based streaming

Where Chat Completions streams indexed delta chunks (clients must reassemble tool-call fragments by `index`), the Responses API streams **discrete, typed events**:

- `response.output_item.added` — a new output item (e.g., a function call) has begun.
- `response.function_call_arguments.delta` — incremental update to a function call's `arguments` string.
- `response.function_call_arguments.done` — the function call's arguments are complete.
- Analogous events fire for text deltas, reasoning, and built-in tool calls.

This makes UI patterns like "show the tool call as it's being constructed" or "stream reasoning in a collapsible panel" much cleaner than under Chat Completions.

## Function-call output structure

A function call in the Responses API output looks like:

```json
{
  "id": "fc_12345xyz",
  "call_id": "call_12345xyz",
  "type": "function_call",
  "name": "get_weather",
  "arguments": "{\"location\":\"Paris, France\"}"
}
```

Note `arguments` is a **JSON-encoded string**, not a parsed object (preserves exact bytes for replay/audit). The application parses, executes, and appends a `function_call_output` item to `input`:

```python
input_list.append({
    "type": "function_call_output",
    "call_id": tool_call.call_id,
    "output": result
})
```

The `call_id` round-trip is how the model correlates results with its original requests — critical for parallel tool calls where multiple calls are in flight.

## Relation to function calling

The Responses API is the modern delivery vehicle for [[function-calling]] on OpenAI's stack. All of the function-calling features — parallel tool calls, `tool_choice` constraints (`auto` / `required` / forced / `allowed_tools`), strict-mode [[structured-outputs]], custom tools with Lark/Regex grammars, and `tool_search` for large catalogs — are first-class here. Chat Completions still supports the same features but through its older, wrapper-based shape.

## See Also

- [[openai-api]] — the parent product
- [[function-calling]] — the core mechanism the API delivers
- [[structured-outputs]] — strict-mode schema enforcement
- [[mcp]] — Model Context Protocol, available as a built-in tool
- [[openai-function-calling-2025]] — source documentation
