---
title: "Function Calling with LLMs (Prompting Guide)"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - function-calling-llms-promptingguide.md
year: 2024
introduces:
  - "[[function-calling]]"
tags:
  - 2024
status: complete
importance: medium
---

# Function Calling with LLMs (Prompting Guide)

Source URL: [https://www.promptingguide.ai/applications/function_calling](https://www.promptingguide.ai/applications/function_calling)

## Summary

A practitioner-oriented overview of [[function-calling]] as the mechanism by which modern LLMs (GPT-4, GPT-3.5, and peers) reliably bridge natural-language queries to external systems. The piece frames function calling as an *architectural* innovation: rather than fine-tuning models only to produce free-form prose, providers fine-tune them to **detect when a tool is needed** and **emit structured JSON** specifying which function should run and with which arguments. The application — not the model — then executes the call and feeds the result back. The guide walks through tool selection, the JSON-schema contract between model and app, the 7-step practical workflow, and disambiguates the overloaded terminology ("function calling" vs. "tool use" vs. "tool calls").

## Key Points

- **Function calling is a fine-tuned capability**, not an emergent one. Models are explicitly trained to recognize when the user's query exceeds internal knowledge (real-time weather, DB lookups, current events) and to respond with a well-formed JSON tool-call payload instead of prose.
- **The LLM does NOT execute tools.** It only outputs JSON that describes a function invocation. Execution, error handling, retries, and state management all belong to the application layer. This separation of concerns is the load-bearing design choice.
- **The model→app agreement is the contract.** Developers pass function definitions (name, description, parameter schema with types and `required` fields) into the request; the model is expected to emit responses matching that schema. The schema is the interface; everything else depends on both sides honoring it.
- **Tool selection is learned pattern recognition.** When given a query like *"What is the weather in London?"* alongside a `get_current_weather` definition, the model matches query semantics to function descriptions and routes appropriately. Good descriptions and parameter docs are therefore prompt engineering, not decoration.
- **Five things the model DOES**: analyze the query, semantically match it to tool descriptions, extract parameters, format them to the schema, emit JSON.
- **Five things the model DOES NOT**: execute the function, hit external systems, deliver the final user-facing answer from raw tool output, handle tool errors, manage state.
- **Terminology**:
  - *Function calling* — OpenAI-originated term for the specific JSON-emit capability; now generalized.
  - *Tool use* — the broader strategic objective (Anthropic's preferred term).
  - *Tool calls* — individual JSON-payload instances of an invocation.
- **The 7-step workflow**: (1) Define tools as JSON schemas → (2) User issues query → (3) Model selects appropriate function → (4) Model generates JSON with name + parameters → (5) Application parses and executes → (6) Result returned to model as context → (7) Model synthesizes natural-language response. The cycle can iterate.
- **Why it matters for agents**: function calling is what makes [[llm-agent|LLM agents]] possible — conversational agents, retrieve-and-synthesize patterns, and write/action systems all reduce to repeated tool-call cycles.

## Concepts Mentioned

- [[function-calling]]
- [[tool-use]]
- [[llm-agent]]
- [[json-schema]]
- [[structured-outputs]]
- [[fine-tuning]]
- [[hallucination]]
- [[mcp|Model Context Protocol]]

## References

- [Prompting Guide — Function Calling with LLMs](https://www.promptingguide.ai/applications/function_calling)
- OpenAI Function Calling docs (June 2023 launch)
- Anthropic Tool Use docs (2024)
