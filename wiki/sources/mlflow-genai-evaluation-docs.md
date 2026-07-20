---
title: "MLflow GenAI Evaluation (documentation)"
type: source
created: 2026-07-07
updated: 2026-07-07
sources: []
year: 2026
venue: "MLflow documentation"
tags:
  - 2026
status: complete
importance: medium
---

# MLflow GenAI Evaluation (documentation)

[mlflow.org/docs/latest/genai/eval-monitor](https://mlflow.org/docs/latest/genai/eval-monitor/). MLflow's [[agent-evaluation]] surface, organized around a philosophy it names **"Evaluation-Driven Development."**

## Vocabulary

*"Each evaluation is defined by three components: **Dataset**, **Scorer**, **Predict Function**."*

| Noun | Meaning |
|---|---|
| **Trace** | the unit of observation; everything flows from tracing |
| **Scorer** | umbrella API concept — both LLM judges and code-based scorers are scorers |
| **Judge** | an LLM-powered scorer. Four tiers: built-in -> Guidelines -> custom (`make_judge()`) -> code scorer |
| **Feedback** | judge output (`value` + `rationale`), attached to the trace as an assessment |
| **Expectation** | ground truth, in the dataset record or logged onto a trace |
| **Evaluation Dataset** | first-class server-side object (requires a SQL backend, not FileStore) |

## Built-in judges

**Response quality**: `RelevanceToQuery`, `Correctness`, `Completeness`, `Fluency`, `Safety`, `Equivalence`, `Summarization`, `Guidelines`, `ExpectationsGuidelines`.
**RAG** (trace required): `RetrievalRelevance`, `RetrievalGroundedness`, `RetrievalSufficiency`.
**Tool call** (experimental, no ground truth needed): **`ToolCallCorrectness`** — *"Are the tool calls and arguments correct for the user query?"* — and **`ToolCallEfficiency`** — *"Are the tool calls efficient without redundancy?"*
**Multi-turn** (experimental, session-scoped): `ConversationCompleteness`, `ConversationalGuidelines`, `ConversationalRoleAdherence`, `ConversationalSafety`, `ConversationalToolCallEfficiency`, `KnowledgeRetention`, `UserFrustration`.

Notably `ToolCallCorrectness` needs **no reference trajectory** — MLflow's approach to trajectory is LLM judgment over the trace, where [[langsmith-evaluation-docs|LangChain]]'s is reference-matching algebra. Exact-match trajectory is not shipped; the docs' own example hand-writes it over `trace.search_spans(span_type=SpanType.TOOL)`.

Trace-based custom judges are the interesting mechanism: they *"autonomously explore traces using Model Context Protocol ([[mcp|MCP]]) tools"* — a judge that navigates rather than reads.

## Judge alignment

MLflow's most distinctive feature. `judge.align(traces, optimizer)` requires each trace to carry **both** a judge assessment and human feedback **under the same assessment name**, minimum **10 traces** (recommended >=30% of each class). Optimizers: **MemAlign** (default, dual-memory few-shot), **SIMBA** and **GEPA** (both DSPy). Where LangSmith measures alignment and lets you tune the prompt, **MLflow optimizes the prompt for you**.

## Online vs offline

**"Automatic Evaluation"** is MLflow's online eval: register a judge, `.start(ScorerSamplingConfig(sample_rate=0.5))`, with trace-search filter strings. Runs server-side and asynchronously via AI Gateway. Sessions are evaluated after 5 minutes of inactivity and **re-evaluated if new traces arrive**. Judges only see traces <=1 hour old; failed evaluations are not retried.

> [!important] Code scorers cannot run online
> Automatic evaluation supports **LLM judges only** — code-based scorers are explicitly excluded. This is a real constraint: the cheap deterministic checks are exactly what you'd want on live traffic.

CI: `@mlflow.test` pytest decorator plus `assert result.passed, result.reason`.

## Opinionated claims

- *"Out-of-the-box judges such as 'Groundedness' or 'Safety' struggle to understand your domain-specific data and criteria."* The prescribed loop: collect traces -> gather human feedback -> error analysis -> translate failure modes into judges -> **align judges with human feedback**.
- A sharp split between **dataset evaluation** (a measurement question, aggregate metrics) and **regression test** (a gating question, binary pass/fail). *"The best regression suites are not written up front, they are grown from real failures."*
- Decompose judge instructions into "what to evaluate / what to examine / how to judge / what format to return."

## Gaps

No trajectory match-mode library; no built-in latency/cost/step-count scorers; no annotation queues; no pairwise evaluator; no synthetic dataset generation in OSS. `Safety` and `RetrievalRelevance` are documented as **Databricks-managed only**, "to be open-sourced soon" — so OSS MLflow's built-in safety story is thinner than the judge table implies.

## Concepts

- [[agent-evaluation]] · [[langsmith-evaluation-docs]] · [[llm-as-a-judge]] · [[mcp]]

## References

- [MLflow GenAI evaluation docs](https://mlflow.org/docs/latest/genai/eval-monitor/)
