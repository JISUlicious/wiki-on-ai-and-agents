---
title: "NVIDIA NeMo Agentic Evaluation Metrics"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
year: 2026
venue: "NVIDIA NeMo Platform documentation"
tags:
  - 2026
status: complete
importance: medium
---

# NVIDIA NeMo Agentic Evaluation Metrics

> [!note] The documented URL moved
> `docs.nvidia.com/nemo/evaluator/latest/evaluation/agentic.html` now **404s**. The content was relocated out of the NeMo Evaluator SDK docs into the **NeMo Platform / microservices** docs: [docs.nvidia.com/nemo/microservices/latest/evaluator/metrics/agentic.html](https://docs.nvidia.com/nemo/microservices/latest/evaluator/metrics/agentic.html), with a companion [Agentic Benchmarks](https://docs.nvidia.com/nemo/microservices/latest/evaluator/benchmarks/agentic.html) page.

*"Evaluate agent-based and multi-step reasoning models using metrics powered by **RAGAS**."*

## The metrics

| Metric | Judge required | Notes |
|---|---|---|
| **Tool Call Accuracy** | **No** | correct tools with correct arguments, against `reference_tool_calls` |
| **Tool Calling** (template) | **No** | emits two scores: **`function_name_accuracy`** and **`function_name_and_args_accuracy`**. **Order of tool calls is ignored** (supports parallel calling); comparison is case-sensitive |
| **Topic Adherence** | Yes | topic focus in multi-turn; `metric_mode` = f1 / precision / recall |
| **Agent Goal Accuracy** | Yes | **binary 0 or 1**; works with or without a reference outcome |
| **Answer Accuracy** | Yes | **two LLM judges rate independently and scores are averaged**; 0 / 0.5 / 1 |
| **Trajectory Evaluation** | Yes | the real trajectory metric — *"whether the agent chose appropriate tools **in the correct order**"*. **Job-only, offline** |

Note the deliberate split: the tool-calling metrics **ignore order** (to support parallel tool calls), while Trajectory Evaluation explicitly checks it. Trajectory Evaluation also requires a genuine execution trace in NeMo Agent Toolkit format, consuming `intermediate_steps` events.

NVIDIA organizes these into a three-stage workflow: **Intermediate Steps Evaluation** (tool use, retrieval) → **Final-Step Evaluation** (goal accuracy, topic adherence, answer accuracy) → **Trajectory Evaluation** (whole action sequence). The companion benchmarks page runs **BFCL** (Berkeley Function Calling Leaderboard) tasks.

> [!warning] "Online" here does not mean production traffic
> NVIDIA's `MetricOnlineJob` means *the platform generates the responses for you* from a prompt template; `MetricOfflineJob` means the dataset already contains responses. Neither describes scoring live production traffic the way [[langfuse-agent-evaluation-guide|Langfuse]] or Confident AI do. A separate axis distinguishes **Live** evaluation (immediate, <=10 rows) from **Job** evaluation (async, large datasets).

## Concepts

- [[agent-evaluation]] · [[llm-as-a-judge]]

## References

- [Agentic Evaluation Metrics](https://docs.nvidia.com/nemo/microservices/latest/evaluator/metrics/agentic.html) · [Agentic Benchmarks](https://docs.nvidia.com/nemo/microservices/latest/evaluator/benchmarks/agentic.html)
