---
title: Tool Use
type: concept
created: 2026-05-13
updated: 2026-05-16
sources:
  - react-yao-2022.md
  - function-calling-llms-promptingguide.md
status: draft
importance: high
---

# Tool Use

Tool use (sometimes "tool-augmented LLMs" or "function calling") is the practice of letting an LLM **invoke external tools** as part of its inference — search engines, code interpreters, calculators, databases, APIs, other models. The LLM emits a structured action (`Act:` token, JSON function call, etc.); the tool runs externally; the result is fed back into the model's context for further reasoning.

## Historical line

- **Calculator + LM** (Cobbe et al. 2021, "Training Verifiers"): early demonstration of tool augmentation for math.
- **[[react|ReAct]]** ([[react-yao-2022|Yao et al. 2022]]): unified reasoning + acting in a single LLM trajectory, the prototypical agent loop.
- **Toolformer** (Schick et al. 2023): LLM learns when to call which tool via self-supervised data generation.
- **OpenAI function calling** (June 2023): structured JSON arguments instead of text parsing; mainstream production interface.
- **Anthropic tool use** (mid-2024): similar structured tool interface for Claude.
- **MCP (Model Context Protocol)** (Anthropic, 2024): standardizes tool/resource interfaces across LLM clients.

## Why it matters

- **Mitigates [[hallucination|hallucination]]**: facts can be looked up rather than fabricated.
- **Extends capability**: LLMs are bad at arithmetic, perfect-recall, and current-events — tools fix these.
- **Connects to the real world**: agents can take actions (send emails, make purchases, run code), not just answer questions.
- **Inspectable**: the tool call log is a clearer audit trail than an opaque model rollout.

## Trade-offs

- **Latency**: every tool call adds round-trip time.
- **Reliability**: errors (bad parameters, tool timeouts) require careful handling.
- **Security**: a tool-using LLM is an attack surface — prompt injection, data exfiltration, unauthorized actions.
- **Agency limits**: how much autonomy to grant (read-only tools vs. arbitrary actuation) is a design and safety question.

## Function calling as the dominant mechanism (2024-2026)

By 2024, **[[function-calling]]** had consolidated as the production mechanism for tool use across major model providers. Where earlier approaches parsed free-form `Act:` tokens (see [[react|ReAct]]) or relied on self-supervised tagging (Toolformer), modern models are fine-tuned to emit **structured JSON** describing the function name and arguments to invoke. The application — never the model — executes the call and feeds the result back into context.

Function calling is *one* mechanism for tool use; tool use is the broader strategic concept. Concretely:

- **Tool use** = the strategic capability (should the model reach for an external system? which one?).
- **Function calling** = the mechanism (JSON-schema contract, fine-tuned JSON emission, separation of execution into the application layer).
- **Tool calls** = the tactical artifacts (individual JSON payloads).

See [[function-calling]] for the canonical treatment — the model/application contract, the 7-step workflow, the role of fine-tuning, the JSON-schema agreement, and how function calling relates to [[mcp|MCP]] and the [[agent-three-layer-model]].

## References

- [[react-yao-2022]]
- [[function-calling-llms-promptingguide]]
