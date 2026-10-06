---
title: "Function calling and other API updates"
type: source
created: 2026-10-07
updated: 2026-10-07
sources:
  - function-calling-api-updates-openai-2023.md
authors:
  - "[[openai]]"
introduces:
  - "[[function-calling]]"
year: 2023
venue: OpenAI blog
url: https://openai.com/blog/function-calling-and-other-api-updates
tags:
  - 2023
  - blog
status: complete
importance: high
---

# Function calling and other API updates

**Source**: `sources/function-calling-api-updates-openai-2023.md`, from the Wayback snapshot of 2023-06-14
**Author**: [[openai]]
**Date**: **2023-06-13**

## Summary

OpenAI adds **[[function-calling|function calling]]** to the Chat Completions API:
- Developers describe functions to `gpt-4-0613` and `gpt-3.5-turbo-0613` using **JSON Schema**, through the new `functions` and `function_call` parameters.
- The model "intelligently choose[s] to output a JSON object containing arguments to call those functions".
- The models were **fine-tuned** both to detect *when* a function is needed and to emit JSON that matches the signature.

The same release:
- a **16K-context** `gpt-3.5-turbo`;
- `gpt-4-32k-0613`;
- price cuts;
- a deprecation timeline for the March models.

## Key points

- **Use cases named:**
  - chatbots that call external tools, "like ChatGPT Plugins";
  - converting natural language to API calls, such as `send_email(to, body)`;
  - **structured extraction**, e.g. `extract_people_data(...)`.
- **The loop:**
  1. Call the model with the functions.
  2. The model returns `finish_reason: function_call` with arguments.
  3. The developer runs the function.
  4. The result goes back in a `role: function` message, and the model writes the final answer.

  This is the [[react|ReAct]] act-observe loop, moved from text parsing into the API.
- **Safety note:**
  - "Since the alpha release of ChatGPT plugins, we have learned much about making tools and language models work together safely."
  - It cites a proof-of-concept in which "untrusted data from a tool's output can instruct the model to perform unintended actions". That is **indirect [[prompt-injection|prompt injection]]**.
  - Its advice: consume information only from trusted tools, and add "user confirmation steps before performing actions with real-world impact, such as sending an email, posting online, or making a purchase."

## Lineage

| | |
|---|---|
| Built on | [[chatgpt-plugins\|ChatGPT plugins]] (2023-03-23): tools described in a manifest and inserted into the prompt · [[toolformer\|Toolformer]] · [[react\|ReAct]] |
| Changed | Tool schemas move from a plugin manifest into each API request, and the model is fine-tuned to emit valid arguments. Any developer can define tools. |
| Followed by | `tools` / `tool_choice` and parallel calls (Nov 2023, general knowledge), structured outputs ([[openai-function-calling-2025]]) · Anthropic tool use · [[mcp\|MCP]] (2024-11-25), which standardizes tool *servers* |

[[llm-powered-autonomous-agents-weng-2023|Weng (Jun 2023)]] lists plugins and function calling as the two working examples of tool use: tools written by third parties versus tools defined by the developer.

## Concepts & entities

[[function-calling]] · [[tool-use]] · [[chatgpt-plugins]] · [[prompt-injection]] · [[indirect-prompt-injection]] · [[mcp]] · [[openai]]
