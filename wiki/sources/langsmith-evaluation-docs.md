---
title: "LangSmith Evaluation (documentation)"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
year: 2026
venue: "LangChain / LangSmith documentation"
tags:
  - 2026
status: complete
importance: high
---

# LangSmith Evaluation (documentation)

[docs.smith.langchain.com/evaluation](https://docs.smith.langchain.com/evaluation) (redirects to `docs.langchain.com/langsmith/evaluation-concepts`). One of the most-cited practitioner references for [[agent-evaluation]].

## Vocabulary

| Noun | Meaning |
|---|---|
| **Dataset** | collection of examples; has **splits** (an example may belong to several) and auto-**versions** |
| **Example** | a test input / reference-output pair. Reference outputs are *never* passed to your app — only to evaluators |
| **Experiment** | results of evaluating one app version on a dataset |
| **Run** | a single execution trace of the app, including all child runs |
| **Thread** | related runs forming a multi-turn conversation |
| **Evaluator** | a **workspace-level resource**; sampling rate, filters and spend limits are set *per attachment*, not per evaluator |
| **Feedback** | `key` + `score` (numeric) or `value` (categorical) + optional `comment` |
| **Annotation queue** | structured human review; single-run (rubrics, **assertions**) and pairwise (A/B) |

> [!note] Vocabulary collision
> LangSmith's **Run** = one app execution. MLflow's **Run** = the tracking container an evaluation writes into (its app execution is a **Trace**). The two products use the same word for different things — see [[mlflow-genai-evaluation-docs]].

## The three agent evaluation approaches

- **Final Response** — agent as black box. Simple; downsides are explicitly named: slow, no visibility inside, hard to define metrics.
- **Single step** — evaluate one decision in isolation, typically "was the right tool selected, with roughly right input". Dataset creation is easy for early steps, hard for late ones needing long prior history.
- **Trajectory** — evaluate all steps. *"Not mutually exclusive!"*

## Trajectory matching — the match-mode algebra

This is LangChain's most distinctive contribution, though it lives in the separate open-source **`agentevals`** package rather than in LangSmith itself. `create_trajectory_match_evaluator` takes:

| `trajectory_match_mode` | Meaning |
|---|---|
| `strict` | same messages/tool calls in the same order |
| `unordered` | same tool calls **in any order** — *"allow flexibility in how an agent obtains the proper information, but still care that all information was retrieved"* |
| `subset` / `superset` | actual trajectory is a subset (resp. superset) of reference |

Orthogonally, `tool_args_match_mode` (`exact` / `ignore` / `subset` / `superset`) controls argument equality, with per-tool `tool_args_match_overrides`.

Also available: `create_trajectory_llm_as_judge` (reference-free, via `TRAJECTORY_ACCURACY_PROMPT`, or reference-based), and LangGraph-specific **graph trajectory** evaluators operating on node steps across resumed/human-in-the-loop threads.

The docs are unusually candid about the limits: exact match *"is simple, but has some flaws. Sometimes there can be multiple correct paths... does not capture the difference between a trajectory being off by a single step versus being completely wrong."* And crucially: *"none of these approaches evaluate the input to the tools; they only focus on the tools selected."*

## Prebuilt evaluators

Named judge prompts ship in **`openevals`**, not the platform: `CORRECTNESS_PROMPT`, `HALLUCINATION_PROMPT`, `RAG_GROUNDEDNESS_PROMPT`, `RAG_RETRIEVAL_RELEVANCE_PROMPT`, `TASK_COMPLETION_PROMPT`, `TOOL_SELECTION_PROMPT`, `PLAN_ADHERENCE_PROMPT`, `TRAJECTORY_ACCURACY_PROMPT`, `KNOWLEDGE_RETENTION_PROMPT`, `USER_SATISFACTION_PROMPT`, `PROMPT_INJECTION_PROMPT`, `PII_LEAKAGE_PROMPT`, and more. Non-LLM: `exact_match`, `create_json_match_evaluator`, `create_regex_evaluator`, `create_embedding_similarity_evaluator`, `levenshtein_distance`, `create_pyright_evaluator`.

LangSmith's own UI ships evaluator **templates** grouped by category — Security, Safety, Quality, Conversation, **Trajectory**, Image, Voice — but the individual template names are not enumerated in the docs.

## Offline vs online

| | Offline | Online |
|---|---|---|
| Target | Dataset / Examples | Tracing project / Runs, Threads |
| Data | inputs, outputs, **reference outputs** | inputs, outputs only |
| Use | benchmarking, unit + regression tests, backtesting, pairwise | real-time monitoring, anomaly detection, feedback loop |

The consequence the docs draw out: **reference-based correctness is offline-only**, so reference-free evaluators are what *"provide consistency across both offline testing and online monitoring."* Online evaluators run server-side with run filters, a sampling rate, and support **backfill to past runs**.

## Judge alignment

**Align Evaluator**: push experiment runs to an annotation queue -> human labels -> "Add to Reference Dataset" -> Evaluator Playground -> **Start Alignment**. *"The alignment score is the percentage of examples where the evaluator's judgment matches that of the human expert."* Guidance: start with **>=20 labeled examples**, balanced; enable reasoning; group misaligned examples into failure modes and write them into the prompt. Separately, human corrections are auto-inserted as **few-shot examples**.

## Opinionated claims

- *"Evaluation measures performance according to metrics... Testing asserts correctness."* Regression tests are metrics converted into gates.
- Pairwise is preferred when absolute scoring is hard — *"choosing the more informative of two summaries is often easier than assigning an absolute score."*
- Human review first, citing Hamel Husain's "look at your traces."

## What it leaves to you

Trajectory evaluators are in a separate OSS package, not the platform. **State-change / side-effect verification is entirely absent** — which the [[agent-evaluation-readiness-checklist-langchain-2026|companion checklist]] argues is essential. No latency/cost quality metrics, no pass@k, no confidence intervals, no trial isolation.

## Concepts

- [[agent-evaluation]] · [[mlflow-genai-evaluation-docs]] · [[agent-evaluation-readiness-checklist-langchain-2026]] · [[llm-as-a-judge]]

## References

- [LangSmith evaluation concepts](https://docs.smith.langchain.com/evaluation)
