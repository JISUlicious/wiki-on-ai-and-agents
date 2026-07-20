---
title: "DeepEval Agent Evaluation (documentation)"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
year: 2026
venue: "DeepEval documentation"
tags:
  - 2026
status: complete
importance: medium
---

# DeepEval Agent Evaluation (documentation)

[deepeval.com/docs/getting-started-agents](https://deepeval.com/docs/getting-started-agents) (DeepEval 4.x). The open-source eval framework with the **most fully specified agentic metric set** of the tools surveyed.

## Vocabulary

- **Golden** — *"a precursor to a test case"*; holds `input`, `expected_output`, `context`, **`expected_tools`**. Becomes a test case once the app produces `actual_output` / `tools_called`.
- **EvaluationDataset** — a collection of goldens; single-turn XOR multi-turn.
- **Metric** — *"while a test case represents the thing you're trying to measure, the metric acts as the ruler."* All output **0-1 plus reasoning**; pass iff `score >= threshold` (**default 0.5**).
- **trace / span** — agent evals are *"powered by tracing"*; attach metrics to the **trace** for end-to-end or to **spans** for component-level. Sub-agent delegations each emit an **agent span**.

## The six agentic metrics

| Metric | Judge? | Reference? | What it measures |
|---|---|---|---|
| **Task Completion** | LLM | referenceless | `AlignmentScore(Task, Outcome)`, both extracted from the trace; task can be inferred or pinned |
| **Tool Correctness** | **mostly deterministic** | **reference-based** (`expected_tools`) | deterministic match of called vs expected tools; if `available_tools` is supplied an LLM also judges optimality and the **final score is the minimum of both** |
| **Argument Correctness** | LLM | referenceless | correct input parameters / total tool calls |
| **Step Efficiency** | LLM | referenceless | penalizes actions *"not strictly required to finish the task"* — duplicates, retries, detours |
| **Plan Adherence** | LLM | referenceless | did execution follow the plan extracted from the agent's reasoning |
| **Plan Quality** | LLM | referenceless | was the plan itself good, ignoring execution |

Tool Correctness is notable for being the one metric that *"uses both deterministic and non-deterministic evaluation."* There is **no single trajectory metric** — trajectory concerns are decomposed across which tools (Tool Correctness, with optional `should_consider_ordering`), which arguments (Argument Correctness), how many steps (Step Efficiency), and plan conformance (Plan Adherence).

The **Plan Quality / Plan Adherence** pair is the closest any surveyed tool comes to the plan-validity vs plan-conformance distinction in [[verifier]].

> [!warning] False-pass footgun
> **Plan Adherence and Plan Quality pass by default with score 1 when no plan can be extracted** from the agent's reasoning. An agent that never plans scores perfectly on both. The docs state this plainly; it is easy to miss.

## Custom criteria: G-Eval

*"G-Eval is a framework that uses LLM-as-a-judge with chain-of-thoughts (CoT) to evaluate LLM outputs based on ANY custom criteria."* Takes `name`, `criteria`, `evaluation_params`; optional `evaluation_steps` (**mutually exclusive with `criteria`** — omit it and G-Eval generates steps itself) and `rubric`. The docs recommend pairing a G-Eval metric with `TaskCompletionMetric` for agents. The deterministic alternative is **DAG** ("LLM-powered decision trees").

## Offline vs online

Offline/CI is first-class: `deepeval test run` wrapping pytest, `assert_test(golden=..., metrics=[...])`, a full GitHub Actions recipe — *"Failing metrics now fail the build."* Online (production) evaluation exists but is **Confident AI enterprise territory**, not the OSS CLI.

A constraint worth carrying over to any tool: *"Which metrics can I use in production? Only referenceless ones, since no labeled data exists live."* So **Tool Correctness is offline-only in practice**, while Task Completion / Argument Correctness / Step Efficiency / Plan* are production-viable.

## Concepts

- [[agent-evaluation]] · [[llm-as-a-judge]] · [[verifier]]

## References

- [DeepEval agents quickstart](https://deepeval.com/docs/getting-started-agents) · [metrics introduction](https://deepeval.com/docs/metrics-introduction) · [G-Eval](https://deepeval.com/docs/metrics-llm-evals)
