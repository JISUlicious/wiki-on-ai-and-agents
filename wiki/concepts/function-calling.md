---
title: Function Calling
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - function-calling-llms-promptingguide.md
  - openai-function-calling-2025.md
  - vllm-tool-calling-2025.md
status: complete
importance: high
tags:
  - 2024
---

# Function Calling

**Function calling** is the capability of an LLM to recognize when a user request requires an external tool and to emit a **structured JSON payload** specifying the function name and arguments to invoke. The model itself does not run the function; it produces a machine-readable instruction that the surrounding application parses, executes, and feeds back as context. Function calling is the dominant 2024-2026 mechanism for connecting LLMs to APIs, databases, code interpreters, and other side-effecting systems, and it is the substrate beneath nearly every production [[llm-agent|LLM agent]].

## What the LLM Does and Doesn't Do

The most important property of function calling is the **separation of concerns** between the model and the application.

**The model DOES:**

1. Analyze the user's natural-language query.
2. Semantically match the query against available tool descriptions.
3. Extract the relevant parameters from the query.
4. Format those parameters according to the declared JSON schema.
5. Emit a JSON tool-call payload (or a "no tool needed" plain response).

**The model does NOT:**

1. Execute the function or run any code.
2. Access external systems (HTTP, files, DBs, the network).
3. Produce the final user-facing answer directly from raw tool output — it only does so *after* the result is returned to it as context.
4. Handle errors, timeouts, or retries from the tool execution.
5. Persist state between calls — the application is responsible for conversation/tool-call history.

This split keeps the LLM stateless and side-effect-free, and concentrates all I/O, auth, and risk in the application layer where it can be audited and rate-limited.

## The JSON-Schema Agreement

Function calling rests on a **contract** between the model and the application. The application declares the tools it exposes by sending function definitions in the request — typically a name, a natural-language description, and a JSON-schema parameter object with types, enums, and `required` fields. Example:

```
{
  "name": "get_current_weather",
  "description": "Get the current weather in a given location",
  "parameters": {
    "type": "object",
    "properties": {
      "location": { "type": "string", "description": "The city and state" },
      "unit":     { "type": "string", "enum": ["celsius", "fahrenheit"] }
    },
    "required": ["location"]
  }
}
```

The model is expected to emit a response that conforms to this schema (e.g., `{"name": "get_current_weather", "arguments": {"location": "London", "unit": "celsius"}}`). The schema is the interface; everything downstream — parsers, executors, type-checked dispatch — assumes the model honored it. Validation libraries and constrained decoding (see [[structured-outputs]] and [[tool-call-parser]]) exist precisely to enforce this contract when the model drifts.

## Tool Selection and Routing

When multiple tools are available, the model selects one via **learned pattern recognition** — not via explicit search or rule-based routing. It matches the semantics of the user's query against each function's description and parameter docs and picks the best fit. This makes the *descriptions* a load-bearing piece of prompt engineering: under-specified or ambiguous descriptions are the most common cause of wrong-tool selection in practice.

## The Role of Fine-Tuning

Function calling is not an emergent capability of base LLMs — it is a **deliberately fine-tuned behavior**. Providers train models on examples where the correct response to a query is a JSON tool call rather than prose, teaching the model to (a) recognize "I need a tool" situations and (b) emit syntactically valid, schema-conforming JSON. This is why function-calling reliability differs sharply across model families and versions, and why provider-specific [[tool-call-parser|tool-call parsers]] exist to handle quirks in each model's output format.

## 7-Step Practical Workflow

The canonical end-to-end loop:

1. **Define tools** — the application specifies available functions as JSON schemas in the request.
2. **User query** — natural-language request enters the system.
3. **Model selection** — the LLM evaluates the query against all declared tool definitions and chooses one (or chooses to respond directly).
4. **JSON generation** — the model emits a structured tool-call payload with function name + parameters.
5. **Execution** — the application parses the JSON, executes the function, and captures the result.
6. **Result integration** — the tool's output is fed back into the model's context as a "tool result" message.
7. **Response generation** — the model synthesizes a natural-language reply, possibly issuing further tool calls.

This loop can iterate many times in a single user turn, which is what makes [[react|ReAct]]-style and other agentic patterns possible.

## Terminology Disambiguation

The space is muddled by overlapping vendor terms:

- **Function calling** — OpenAI's term, introduced June 2023. Originally referred to a single function-emit capability; now generalized.
- **Tool use** — Anthropic's broader term, used in Claude's API. Encompasses function calling but also frames it as a strategic capability (Claude can decide to use a tool, can issue parallel tool calls, etc.).
- **Tool calls** — the *outputs* themselves: individual JSON payloads requesting a specific invocation with specific parameters.

A useful mental model: **[[tool-use]] is the strategic objective; function calling is the mechanism; tool calls are the tactical artifacts.**

## Relation to MCP

The [[mcp|Model Context Protocol]] standardizes how tools, resources, and prompts are *exposed* to LLM clients across vendors. Function calling is the **underlying invocation mechanism** the model uses; MCP is the **protocol** by which those tools are discovered, described, and called over a stable interface. An MCP server publishes tool definitions; the host application surfaces them to the LLM via function calling; the model emits JSON; the host routes the call back through MCP. MCP did not replace function calling — it standardized the surrounding plumbing.

## Position in the Three-Layer Model

In the [[agent-three-layer-model]]:

- **L1** is the model itself (the LLM that emits tokens).
- **L2** is the **action surface** — the set of tools the agent can invoke.
- **L3** is the **invocation mechanism** that connects L1 to L2.

Function calling lives at **L3**: it is the JSON-payload protocol by which the model addresses the L2 action surface. Different L3 mechanisms exist (raw text parsing, XML tags, JSON function calls, structured outputs), but function calling has become the production default.

## Relation to Tool Use

[[tool-use]] is the strategic capability — *should* the model reach for an external tool, and which one? Function calling is one specific *mechanism* for realizing that capability. Earlier mechanisms (regex-parsed `Act:` tokens in [[react|ReAct]], Toolformer's self-supervised tags, custom XML schemas) are largely subsumed in production by JSON function calling because fine-tuned models emit it reliably and structured outputs/JSON-schema validators can enforce it.

## See Also

- [[tool-use]] — the broader concept and history
- [[mcp|Model Context Protocol]] — standardized tool exposure
- [[agent-three-layer-model]] — where function calling sits architecturally
- [[structured-outputs]] — constrained decoding to guarantee JSON validity
- [[json-schema]] — the schema language used in tool definitions
- [[tool-call-parser]] — provider-specific parsing of model tool-call outputs
- [[openai-api]] — the original function-calling API surface
- [[claude]] — Anthropic's tool-use implementation
- [[llm-agent]] — agentic systems built atop repeated function calling
- [[react]] — early reasoning + acting loop that prefigured function calling

## References

- [[function-calling-llms-promptingguide]]
- [[openai-function-calling-2025]]
- [[vllm-tool-calling-2025]]
