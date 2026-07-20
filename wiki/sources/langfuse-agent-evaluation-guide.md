---
title: "Langfuse Agent Evaluation Guide"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
year: 2026
venue: "Langfuse cookbook"
tags:
  - 2026
status: complete
importance: medium
---

# Langfuse Agent Evaluation Guide

[langfuse.com/guides/cookbook/example_pydantic_ai_mcp_agent_evaluation](https://langfuse.com/guides/cookbook/example_pydantic_ai_mcp_agent_evaluation) — a worked notebook, not reference docs: it evaluates a **Pydantic AI + [[mcp|MCP]]** agent end to end.

## The three-strategy taxonomy (its most-quoted contribution)

| Strategy | Alias | What it sees |
|---|---|---|
| **Final Response Evaluation** | Black-Box | input + final answer only |
| **Trajectory Evaluation** | Glass-Box | actual tool-call sequence vs expected |
| **Single Step Evaluation** | White-Box | one decision in isolation — *"like a unit test for agent reasoning"* |

The formulation people cite:

> *"Final response evaluation tells you **what** went wrong. Trajectory evaluation tells you **where** it went wrong. Single step evaluation tells you **why** it went wrong."*

It also stages adoption in **three phases**: early development (manual tracing) → first users (online evaluation: thumbs up/down + automated judges on production traces) → scaling (offline pipeline, "gold standard" dataset, regression prevention).

## What it actually implements

> [!note] Langfuse ships no named agent metrics
> Unlike [[deepeval-agent-evaluation-docs|DeepEval]] or [[nvidia-nemo-agentic-metrics|NeMo]], every evaluator here is a **hand-written LLM-as-a-judge prompt** pasted into the Langfuse UI (Prompts → Create Evaluator), with `{{variable}}` placeholders and few-shot Reasoning+Score examples. There is no built-in agentic metric library in this guide.

The **trajectory is recovered by manual instrumentation, not by the tracer** — the agent wraps Pydantic AI's MCP `process_tool_call` hook to append to a `tool_call_history`, then stashes it via `update_current_observation(metadata=...)`.

The trajectory judge is a **string-list set comparison, deliberately order-insensitive**: *"Check whether the lists contain exactly the same items. Order does not matter."* Binary 0/1, scoring 0 for either a missing or an extra tool.

Dataset items carry three expectation keys: **`response_facts`** (facts the answer must contain), **`trajectory`** (expected tool sequence), **`search_term`**. The demo closes with a **2x2 grid sweep** (two system prompts x two models) compared in the experiment view.

## Online and offline

Both, explicitly. Offline via `dataset.run_experiment(...)` in CI — *"Block deployments that cause score regressions on your benchmark dataset."* Online via automated judges scoring production traces in real time plus user thumbs up/down. Recommended dataset practice: *"Build your dataset from real failures... Use **annotation queues** to systematically review and label problematic traces, then add them to your evaluation dataset."*

## Concepts

- [[agent-evaluation]] · [[llm-as-a-judge]] · [[mcp]]

## References

- [Langfuse agent evaluation guide](https://langfuse.com/guides/cookbook/example_pydantic_ai_mcp_agent_evaluation)
