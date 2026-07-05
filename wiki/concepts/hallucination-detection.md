---
title: Hallucination Detection
type: concept
created: 2026-07-05
updated: 2026-07-05
sources:
  - probe-process-benchmark-hallucination-detection-2026.md
  - reliability-without-validity-llm-as-a-judge-norman-2026.md
builds-on:
  - [[hallucination]]
status: draft
importance: medium
tags:
  - 2026
---

# Hallucination Detection

**Hallucination detection** is the task of deciding, for a given model output, whether (and *where*) it contains [[hallucination|hallucinated]] content — claims that are fluent but factually unsupported. It is distinct from [[hallucination]] (the phenomenon) and from *mitigation* ([[retrieval-augmented-generation|RAG]], self-critique): detection is the **measurement/verification** layer that a mitigation or guardrail can act on.

## Two framings

- **Single-step ("LLM-as-a-judge")** — prompt an LLM to self-assess factuality in one pass, optionally with ground-truth evidence. This is the dominant pattern in agentic apps, but it is **opaque and coarse**: even given ground truth it under-detects, and it can't say *where or why* it failed. [[reliability-without-validity-llm-as-a-judge-norman-2026|Reliability without Validity]] shows exact-match agreement systematically **overstates** LLM-judge quality (33–41pp gap vs chance-corrected agreement).
- **Process-based (decompose-and-localize)** — treat detection as a *pipeline* and evaluate each stage. [[probe-process-benchmark-hallucination-detection-2026|PROBE]] (Findings of ACL 2026; CUHK + NVIDIA) decomposes it into **claim decomposition → evidence finding → evidence evaluation → hallucination localization**, scoring each step so a failure can be attributed to the breaking stage. Its empirical payoff: **treating detection as multi-step lifts *all* models** over one-shot prompting, and it localizes the failure — **evidence finding is the primary bottleneck** (fixable by fine-tuning on step-wise supervision). This mirrors the reasoning-eval shift from outcome-level to **step-level / process supervision** (ProcessBench, PRMBench, FG-PRM).

## Why it's hard

- **Attribution vs. classification** — a binary "hallucinated?" verdict hides *which* claim and *which* evidence step went wrong; process benchmarks exist precisely to surface that.
- **Judge validity** — the evaluator is itself an LLM, so detection inherits the LLM-as-a-judge reliability problem.
- **Evidence dependence** — the "evidence finding" stage is a [[retrieval-augmented-generation|retrieval]] problem, so retrieval failures masquerade as detection failures.

## Related

- [[hallucination]] — the phenomenon being detected.
- [[probe-process-benchmark-hallucination-detection-2026]] — process-based, four-step detection benchmark.
- [[reliability-without-validity-llm-as-a-judge-norman-2026]] — why single-step LLM-judge evaluation overstates reliability.
- [[retrieval-augmented-generation]] — evidence retrieval underpins the evidence-finding step.
- [[agent-benchmark]] — the broader evaluation category.

## References

- [[probe-process-benchmark-hallucination-detection-2026]]
- [[reliability-without-validity-llm-as-a-judge-norman-2026]]
