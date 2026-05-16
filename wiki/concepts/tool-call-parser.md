---
title: Tool Call Parser
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

# Tool Call Parser

A **tool call parser** is the component in an inference engine that converts a model's raw decoded text into a structured tool-call object (function name + JSON arguments) consumable by an OpenAI-compatible API. It sits between the model's token stream and the client-facing `tool_calls` field in the response.

## The model-format problem

There is **no industry standard** at the model layer for how a tool invocation is emitted. Each model family encodes calls differently — and the parser must speak each dialect:

| Family | Wire format (example) |
|---|---|
| Llama 3.x | JSON object: `{"name": "get_weather", "parameters": {...}}` |
| Hermes / Qwen 2.5 | XML-style tags: `<tool_call>{"name": ...}</tool_call>` |
| DeepSeek v3 / v3.1 | XML-wrapped JSON with custom delimiters |
| "Pythonic" models (Llama 3.2 small, ToolACE, xLAM) | Python list: `[get_weather(city='SF'), get_weather(city='Seattle')]` |
| FunctionGemma (Google, 270M) | `<start_function_call> ... <end_function_call>` |
| Kimi K2 (Moonshot) | Moonshot-specific delimiters |
| Salesforce xLAM | Multiple alternatives: bare JSON, `[TOOL_CALLS]` markers, thinking tags, code blocks |

This heterogeneity is why [[vllm]] ships ~25 different parser implementations — `llama3_json`, `mistral`, `hermes`, `granite` / `granite4` / `granite-20b-fc`, `pythonic`, `llama4_pythonic`, `olmo3`, `deepseek_v3` / `deepseek_v31`, `cohere_command3`, `functiongemma`, `kimi_k2`, `glm45` / `glm47`, `xlam`, `internlm`, etc.

## Parser interface (vLLM)

Each parser implements two methods:

### `extract_tool_calls` (non-streaming)
- Input: full model output text.
- Output: `ExtractedToolCallInformation` with a `tools_called` flag, a list of `ToolCall` objects, and any remaining natural-language content.

### `extract_tool_calls_streaming` (streaming)
- Called incrementally as tokens arrive.
- Input: previous full text, current full text, delta text, token sequences.
- Output: `DeltaMessage` representing the incremental tool-call delta.
- Enables low-latency execution patterns: e.g., a downstream router can dispatch on the function name as soon as it has been emitted, in parallel with the model still generating arguments.

## Relationship to structured outputs

A tool-call parser is **descriptive**, not prescriptive — it interprets whatever the model emits. It does **not** guarantee that the extracted arguments validate against the function's JSON schema. For schema guarantees, [[vllm]] (and similar engines) pair the parser with a [[structured-outputs]] backend that uses [[constrained-decoding]]:

- **Auto mode** (`tool_choice="auto"`): parser alone — argument correctness is a function of model quality.
- **Named / required mode** (`tool_choice="function_name"` or `"required"`): constrained decoding forces the model to emit schema-valid JSON; the parser then extracts it.

## Custom parsers

vLLM exposes a plugin system: implement a `ToolParser` subclass and register it with `ToolParserManager.register_lazy_module(...)`. This lets new model families add tool-calling support without core-engine changes.

## Cross-references

- Surface concept: [[function-calling]].
- Schema enforcement: [[structured-outputs]], [[constrained-decoding]].
- Host engine: [[vllm]].
