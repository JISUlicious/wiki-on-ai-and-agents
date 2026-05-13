---
title: Tool Use
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - react-yao-2022.md
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

## References

- [[react-yao-2022]]
