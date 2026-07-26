---
title: Agent Evaluation
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - demystifying-evals-for-ai-agents-anthropic-2026.md
  - agent-evaluation-readiness-checklist-langchain-2026.md
  - langsmith-evaluation-docs.md
  - mlflow-genai-evaluation-docs.md
  - langfuse-agent-evaluation-guide.md
  - deepeval-agent-evaluation-docs.md
  - nvidia-nemo-agentic-metrics.md
  - agentbench-liu-2023.md
  - agent-as-a-judge-zhuge-2024.md
status: complete
importance: high
---

# Agent Evaluation

How do you tell whether an agent is working? Agents call tools across many turns, modify state, and adapt — so mistakes compound, and there is rarely one right answer to compare against. This page synthesizes the practitioner consensus from **[[anthropic|Anthropic]], LangChain/LangSmith, MLflow, Langfuse, DeepEval, NVIDIA NeMo, and IBM**, plus the benchmark literature.

For *what* to verify and against which reference, see [[verifier]]. This page is about *how the industry actually measures it*.

## The universal three-level split

Every source converges on the same decomposition, with different names:

| Level | Anthropic | LangSmith | Langfuse | LangChain checklist |
|---|---|---|---|---|
| Whole task, black box | **outcome** | Final Response | **Final Response (Black-Box)** | full-turn (**trace**) |
| Path taken | **transcript** | Trajectory | **Trajectory (Glass-Box)** | full-turn (**trace**) |
| One decision | — | Single step | **Single Step (White-Box)** | single-step (**run**) |
| Conversation | — | Thread | — | multi-turn (**thread**) |

Langfuse's formulation is the one worth memorizing:

> *"Final response evaluation tells you **what** went wrong. Trajectory evaluation tells you **where** it went wrong. Single step evaluation tells you **why** it went wrong."*

**Recommended entry point** (LangChain checklist): start at **trace level**, then layer in run- and thread-level as needed.

## Anthropic's vocabulary

The [[demystifying-evals-for-ai-agents-anthropic-2026|"Demystifying evals"]] terms are becoming the shared language: **task** (one test) → **trial** (one attempt; run several) → **grader** (scoring logic, containing **assertions**) → **transcript** (the full record) → **outcome** (the final environment state) → **eval harness** vs **agent harness** → **eval suite**.

The **outcome vs transcript** split is load-bearing: an agent might say *"Your flight has been booked"* while the outcome is whether a reservation exists in the database.

> [!warning] Terminology collisions to watch
> - **"Run"** — in LangSmith, one execution of your app. In MLflow, the *tracking container an evaluation writes into* (its app execution is a **Trace**).
> - **"Online"** — for Langfuse/LangSmith it means *scoring live production traffic*. For **NVIDIA NeMo it means the platform generates the responses for you** from a prompt template; neither NeMo mode is production monitoring.

## The trajectory question — the field's live disagreement

The sharpest methodological tension in this literature.

**Against path-grading**, Anthropic is blunt: *"There is a common instinct to check that agents followed very specific steps like a sequence of tool calls in the right order. We've found this approach too rigid... agents regularly find valid approaches that eval designers didn't anticipate."* Their illustration: Opus 4.5 solved a τ²-bench booking task **by finding a loophole in the policy** — it failed the eval as written while producing a better outcome. The LangChain checklist adopts the rule verbatim: *"Grade the outcome, not the exact path."*

**For path-grading**, [[agent-as-a-judge-zhuge-2024|Agent-as-a-Judge]] argues final-outcome evaluation is inadequate — *"evaluating an agentic system in the traditional way is like evaluating a student using multiple-choice testing"* — and demonstrates why: on its DevAI benchmark, full-task solve rates are **0–1.81%**, so binary end-to-end success carries almost no signal.

**How the field actually resolves it**, in three moves:

1. **Order-insensitivity by default.** Langfuse's trajectory judge says *"Order does not matter."* NVIDIA's tool-calling metrics explicitly ignore order (to permit parallel calls). LangChain ships `unordered` alongside `strict`, plus `subset`/`superset`. Strict ordering is opt-in, not the default.
2. **Requirement-level, not step-level.** Agent-as-a-Judge doesn't match tool sequences — it judges **365 hierarchical requirements** arranged in a dependency DAG. That is a different thing from trajectory matching, and it's what makes intermediate feedback tractable.
3. **Trajectory as an efficiency signal, not a correctness signal.** The LangChain checklist's reconciliation, and the cleanest resolution available: *"ideal trajectories measure efficiency, not correctness. You still pass an agent that found a creative route, but you can see if it took longer to get there."* Track observed/ideal ratios for steps, tool calls, and latency.

A caveat LangSmith states that applies to every match-based approach: *"none of these approaches evaluate the input to the tools; they only focus on the tools selected."*

## The dimension almost everyone forgets: state changes

The LangChain checklist names three trace-level dimensions — final response, trajectory, and **state changes**:

> *"Often overlooked but critical for agents that do things, not just say things... don't just check that it said 'Meeting scheduled!' Verify the calendar event actually exists. If it writes code, run the code. If it updates a database, query the rows."*

This is exactly Anthropic's **outcome** definition. **Neither LangSmith nor MLflow ships anything for it** — it is left entirely to custom code. Given it's the only dimension that verifies the agent changed the world rather than described changing it, that's the most consequential gap in current tooling.

## Grader types

Universal three-way split — **code-based, model-based (LLM-as-judge), human** — with universal guidance: *deterministic where possible, LLM where necessary, human for validation*.

| | Strengths | Weaknesses |
|---|---|---|
| **Code** | fast, cheap, objective, reproducible | brittle to valid variations |
| **LLM judge** | flexible, handles subjective criteria | needs calibration; biased |
| **Human** | ground truth | expensive, unscalable |

Two rules recur across sources: **decompose the grader** (one isolated judge per rubric dimension, not one monolithic correctness judge — a cited team built five), and **build in partial credit** (an agent that identifies the problem but fails the last step is better than one that fails immediately).

The checklist adds a distinction the tools blur: **guardrails ≠ evaluators.** Guardrails run *during* execution, in milliseconds, to *block*. Evaluators run *after*, asynchronously, in seconds-to-minutes, to *measure*. *"Don't confuse the two."*

## What the tools actually ship

| | Trajectory approach | Judge alignment | Online eval |
|---|---|---|---|
| **LangChain** (`agentevals`) | **reference-matching algebra** — `strict`/`unordered`/`subset`/`superset` × tool-arg modes | Align Evaluator, ≥20 labeled examples, % agreement | yes, + backfill |
| **MLflow** | **LLM judgment over the trace** — `ToolCallCorrectness`, `ToolCallEfficiency` (no reference needed) | `judge.align()`, ≥10 traces, auto-optimizes via MemAlign/SIMBA/GEPA | **LLM judges only** — code scorers excluded |
| **DeepEval** | decomposed into 6 metrics: Task Completion, Tool Correctness, Argument Correctness, Step Efficiency, **Plan Adherence, Plan Quality** | — | referenceless metrics only |
| **NVIDIA NeMo** | Tool Call Accuracy (order-ignored) + **Trajectory Evaluation** (order-checked, offline only) | — | "online" = live *generation*, not traffic |
| **Langfuse** | **no built-in metrics** — hand-written judge prompts; order-insensitive set comparison | — | yes |

Two things worth flagging from this table:

- **DeepEval's Plan Quality vs Plan Adherence** is the only tool-level instance of the plan-validity vs plan-conformance distinction in [[verifier]] — *was the plan good* versus *did execution follow it*. But both **pass by default with score 1 when no plan can be extracted**, so an agent that never plans scores perfectly on both.
- **Reference-based metrics cannot run in production**, because no labeled data exists live. DeepEval states it explicitly; it's why reference-free evaluators are what give *"consistency across both offline testing and online monitoring."*

## Capability vs regression evals

A distinction every practitioner source draws:

- **Capability evals** — *"what can it do?"* Should **start at a low pass rate**; they give you a hill to climb.
- **Regression evals** — *"does it still work?"* Should sit near **100%**.
- **Graduation**: a saturated capability eval becomes a regression eval. MLflow phrases the same split as *measurement question* vs *gating question*, and adds: *"the best regression suites are not written up front, they are grown from real failures."*

## Non-determinism

Agent behavior varies between runs, so single-run scores mislead. Two metrics, from Anthropic:

- **pass@k** — probability of ≥1 success in k attempts. *Rises* with k.
- **pass^k** — probability **all** k succeed. *Falls* with k. At 75% per-trial and k=3, ≈**42%**.

*"At k=1 they're identical. By k=10 they tell opposite stories."* **pass^k is the honest metric for customer-facing agents.** The checklist adds: compute confidence intervals before declaring an improvement.

## Practical rules that recur

- **20–50 tasks from real failures is enough to start.** *"20-50 hand-reviewed examples you're confident in will outperform hundreds of synthetic examples you haven't verified."*
- **Spend 60–80% of eval effort on error analysis**, not infrastructure.
- **A good task is one where two domain experts independently reach the same verdict.**
- **0% pass@100 with a frontier model means a broken task**, not an incapable agent.
- **Test both directions** — *"one-sided evals create one-sided optimization."*
- **Isolate every trial.** Anthropic found Claude gaining advantage by reading **git history from previous trials**.
- **Read the transcripts.** *"We do not take eval scores at face value until someone digs into the details."*
- **Distinguish task failures from evaluation failures** — a grader marking a timeout as "incorrect reasoning" pollutes the signal.
- **Check infrastructure first**: one cited team found **a single extraction bug moved their benchmark from 50% to 73%**.
- **More evals ≠ better agents.** Prune ones that stop giving signal.

> [!important] The eval is often the thing that's broken
> Opus 4.5 initially scored **42% on CORE-Bench**; after fixing rigid grading, ambiguous specs and irreproducible tasks, it scored **95%**. METR found tasks that asked agents to reach a threshold while grading required exceeding it. Benchmark scores are claims about the benchmark as much as the agent.

## What the benchmarks contribute

- **[[agentbench-liu-2023|AgentBench]]** — 8 environments, 29 models. A ~4× commercial/open gap, and a failure taxonomy that maps to distinct deficits: **Invalid Format** (instruction-following), **Invalid Action** (grounding), **Task Limit Exceeded** (long-horizon reasoning).
- **[[mlagentbench-huang-2023|MLAgentBench]]** — agents doing ML experimentation, judged by whether their code actually improves a metric ≥10%. Introduces **cost-per-success** reporting ($231 expected cost per completed task).
- **[[agent-as-a-judge-zhuge-2024|Agent-as-a-Judge]]** — agentic systems judging agentic systems. Its most useful empirical result: giving the trajectory to a **plain LLM judge made it worse** (84.15% → 68.86% alignment) while the agentic judge **improved** (88.52% → 92.07%). More evidence only helps a judge equipped to navigate it. It also calibrates against **human-vs-human disagreement (~10–30%)** rather than assuming humans are an oracle.

## Worked example

- [[in-house-data-agent-eval-test-set]] — a 35-task suite instantiating this methodology for an internal data-search-and-analysis agent (dataset schema, decomposed grader stack, build order).

## Related

- [[verifier]] — verification targets by reference type; the conceptual layer beneath these practices.
- [[llm-as-a-judge]] · [[agent-as-a-judge]] — the grading mechanisms.
- [[terminal-bench]] · [[swe-bench]] — benchmarks used as eval substrates.
- [[dynamic-workflows]] · [[agentic-harness-engineering]] — what gets evaluated.

## References

- [[demystifying-evals-for-ai-agents-anthropic-2026]] · [[agent-evaluation-readiness-checklist-langchain-2026]] · [[langsmith-evaluation-docs]] · [[mlflow-genai-evaluation-docs]] · [[langfuse-agent-evaluation-guide]] · [[deepeval-agent-evaluation-docs]] · [[nvidia-nemo-agentic-metrics]] · [[ibm-ai-agent-evaluation-tutorial]] · [[agentbench-liu-2023]] · [[mlagentbench-huang-2023]] · [[agent-as-a-judge-zhuge-2024]]
- Anthropic webinar *"Evals for AI Agents: How Product Builders Get the Most Out of Every New Model"* — [registration page](https://www.anthropic.com/webinars/evals-for-ai-agents-how-product-builders-get-the-most-out-of-every-new-model); **no recording available as of 2026-07-07**, so not ingested. Stated agenda: why single-turn evals miss most agent failures, building a first eval set from production failures, and deciding whether a new model release is worth adopting.

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[beyond-the-leaderboard-albayaydh-2026]] — Oxford synthesis of 27 papers / 19 benchmarks; argues part of year-over-year gains is *correction of measurement error*, not capability
- [[failure-as-a-process-cli-agent-trajectories-zhao-2026]] — 63k annotated steps; failure has three timestamps (error / lock-in / observability), and 71% of *successful* runs also err
- [[tracing-agentic-failure-flow-of-success-yeh-2026]] — failure attribution as anomaly detection against successful-trajectory flow — 7 ms and 0 output tokens per trajectory
