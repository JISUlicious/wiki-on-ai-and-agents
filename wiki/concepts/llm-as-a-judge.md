---
title: LLM-as-a-Judge
type: concept
created: 2026-07-07
updated: 2026-07-07
sources:
  - demystifying-evals-for-ai-agents-anthropic-2026.md
  - langsmith-evaluation-docs.md
  - mlflow-genai-evaluation-docs.md
  - deepeval-agent-evaluation-docs.md
status: complete
importance: high
---

# LLM-as-a-Judge

Using a language model to grade another model's output. It is the default mechanism wherever criteria are subjective, open-ended, or too varied for deterministic checks — and the default failure point of naive [[agent-evaluation]].

## Where it fits

The standard grader taxonomy is **code-based / model-based / human**, with consistent guidance: *deterministic where possible, LLM where necessary, human for validation*. LLM judging buys flexibility and costs calibration.

Common forms: **rubric-based scoring**, **natural-language assertions**, **pairwise comparison**, **reference-based** evaluation, and **multi-judge consensus**.

**Reference-free vs reference-based** is the design constraint that matters most operationally: reference-based judges need ground truth and are therefore **offline-only**, while reference-free judges *"provide consistency across both offline testing and online monitoring."* No labeled data exists on live traffic.

## Judge alignment — the practice that separates working judges from decorative ones

Both major platforms ship an alignment feature measuring the judge against human labels, and they converge on a similar threshold while differing in philosophy:

| | Mechanism | Minimum labels |
|---|---|---|
| **LangSmith** *Align Evaluator* | human-in-the-loop prompt iteration in a playground; **alignment score = % agreement with human experts**; corrections auto-inserted as few-shot examples | **>=20**, balanced; grow to ~100 for production |
| **MLflow** `judge.align()` | **automated prompt optimization** — MemAlign (default), SIMBA, GEPA (DSPy-backed) | **>=10** traces, >=30% of each class |

LangSmith measures and lets you tune; MLflow optimizes for you.

The consensus reason this is necessary, from MLflow: *"Out-of-the-box judges such as 'Groundedness' or 'Safety' struggle to understand your domain-specific data and criteria."* And: *"LLM-as-a-Judge has natural biases. Relying on biased evaluation will lead to incorrect decision making."*

## Design rules that recur across sources

- **Decompose.** Use *one isolated judge per rubric dimension* rather than one judge grading everything. A cited team replaced a monolithic correctness evaluator with five specialized ones.
- **Give the judge an out.** Instruct it to return "Unknown" when it lacks information, to suppress hallucinated verdicts.
- **Ask for reasoning**, and read it — it is how you diagnose misalignment.
- **Prefer binary pass/fail** over 1-5 scales for human-reviewed criteria (adjacent scores are subjective and need larger samples), though some evidence suggests short scales align better for LLM judges specifically.
- **Prefer deterministic graders for objective tasks.** *"LLM-as-judge grading for objective tasks can be unreliable... switching to deterministic comparison can often eliminate inconsistency."*
- **Recalibrate periodically** — judges drift, and no single judge is uniformly reliable across benchmarks.
- **Reserve the expense.** *"Use cheap code-based graders in CI for every commit. Reserve expensive LLM-as-judge evaluations for preview/production evaluation."*

## Custom-criteria frameworks

**G-Eval** (DeepEval) is the most-specified: *"a framework that uses LLM-as-a-judge with chain-of-thoughts (CoT) to evaluate LLM outputs based on ANY custom criteria."* Supply `criteria` and it generates its own `evaluation_steps`; supply a `rubric` to confine score ranges. Its deterministic sibling **DAG** builds LLM-powered decision trees instead.

MLflow's `make_judge()` takes natural-language instructions with reserved variables (`{{ inputs }}`, `{{ outputs }}`, `{{ expectations }}`, `{{ trace }}`), and **trace-based judges can autonomously explore traces using [[mcp|MCP]] tools** — a judge that navigates rather than reads.

## The known limits

A plain LLM judge degrades when given too much context. [[agent-as-a-judge-zhuge-2024|Agent-as-a-Judge]] measured it directly: adding the agent's trajectory **dropped** LLM-as-a-Judge alignment from 84.15% to 68.86%, while an agentic judge *improved* on the same input. Judging long multi-step trajectories is where the single-pass form runs out.

**A second, independent degradation axis: trajectory depth.** [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse 2026]] holds per-call input fixed and varies only how deep into an optimization loop the judge sits. Selective accuracy falls from **82.8% to 56.9%** once successful modifications accumulate — the "confidence cliff." This is not the context-length effect above; every configuration receives the same candidate rationale.

Two operational consequences this page did not previously carry:

- **The judge gives no signal that it has degraded.** Coverage moves *up* from 76% to 85% as accuracy collapses — it commits more readily as it gets worse. Plain accuracy averages hide this entirely (77.6% overall versus 56.9% in the deep bucket). Seeing it at all requires **selective accuracy conditioned on an abstention protocol**.
- **"Give the judge an out" is validated but insufficient.** Abstention alone does not explain the recovery: on deep calls that *both* judges commit to, accuracy still rises 63.4% → 82.5% once the judge is given a focused memory of similar past attempts and their outcomes. What fixes the cliff is relevant history, not permission to decline.

Caveat on that result: the depth variable is **observational**, so deep decisions may genuinely differ in headroom and difficulty. The authors call it a controlled proxy, not a randomized causal estimate.

Worth keeping in view: human judges are not an oracle either. Inter-evaluator disagreement in the same study ran **~10-30%**, with one expert at 23.77% error dropping to 6.01% under majority vote.

## Related

- [[agent-as-a-judge]] — the agentic extension.
- [[agent-evaluation]] · [[verifier]] · [[hallucination-detection]]

## References

- [[demystifying-evals-for-ai-agents-anthropic-2026]] · [[langsmith-evaluation-docs]] · [[mlflow-genai-evaluation-docs]] · [[deepeval-agent-evaluation-docs]] · [[agent-as-a-judge-zhuge-2024]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[metacognition-in-llms-liu-2026]] — survey of LLM self-monitoring: verdict *not yet robust*, and better reasoning does not buy better metacognition
- [[llm-as-a-verifier-kwok-2026]] — a continuous scoring-token expectation beats discrete judging by 16.6 pts on the identical base model
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] — the confidence cliff: 82.8% → 56.9% selective accuracy as a self-improvement trajectory deepens, while coverage *rises*
- [[metacognition]] — JOL is a poor *ranker* — within-problem SD 0.048 vs 0.270 across problems, so self-confidence can allocate compute but cannot pick the best attempt
- [[lifecycle-of-llm-as-a-judge-recommendation-explanations-kong-2026]] — Netflix, in production: a judge as a four-phase lifecycle rather than a one-time artifact, with drift monitoring behind a review gate. Supplies the *mechanism* this page's "recalibrate periodically" rule lacked — a meta-judge over the judge's own reasoning traces mismatches back to specific rubric language
