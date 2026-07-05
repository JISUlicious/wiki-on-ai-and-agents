---
title: "PROBE: PROcess-Based BEnchmark for Hallucination Detection"
type: source
created: 2026-07-05
updated: 2026-07-05
sources:
  - probe-process-benchmark-hallucination-detection-2026.md
introduces:
  - [[hallucination-detection]]
year: 2026
venue: "OpenReview (id NuSq4FmzFb)"
tags:
  - 2026
  - benchmark
status: complete
importance: medium
---

# PROBE: PROcess-Based BEnchmark for Hallucination Detection

**OpenReview**: [NuSq4FmzFb](https://openreview.net/forum?id=NuSq4FmzFb) (submission ~2026-03-02)
**Authors**: not recovered (see provenance note)

_Requested by the user for ingest._

> [!note] Abstract-only ingest
> OpenReview is challenge/CDN-protected (HTTP 403 on the forum, the PDF, and the `api2` endpoints) in this sandbox, so the full paper could not be retrieved. This page is reconstructed from the title plus public search-result summaries of the abstract; **authors and venue were not recovered**. Treat specifics as abstract-level. (Same sandbox limitation as [[agent-harness-survey-meng-2026|preprints.org]].)

## Summary

**PROBE** reframes [[hallucination-detection]] as a **process** to be evaluated step-by-step rather than a single pass/fail judgment. It decomposes detection into **four stages — claim decomposition → evidence finding → evidence evaluation → hallucination localization — and scores each stage individually**, so a system's failure can be attributed to *where and why* it broke rather than only *whether* it caught the hallucination. The benchmark comprises **12,000 test cases across three task types: summarization, question answering, and style transfer**.

Its central critique targets the dominant **single-step "[[reliability-without-validity-llm-as-a-judge-norman-2026|LLM-as-a-judge]]"** pattern: agentic applications that ask an LLM to self-assess factuality in one shot fall short even when handed ground-truth evidence, and offer neither the **transparency** nor the **granularity** needed to diagnose failures. PROBE is the process-level, decompose-and-localize answer — the [[hallucination]]-detection analogue of process-supervision benchmarks for reasoning (ProcessBench / PRMBench / FG-PRM), which likewise score intermediate steps rather than only final answers.

## Key points

- **Four evaluated sub-steps** (each scored on its own): (1) **claim decomposition** — break the output into atomic checkable claims; (2) **evidence finding** — retrieve supporting/refuting evidence; (3) **evidence evaluation** — judge each claim against the evidence; (4) **hallucination localization** — pinpoint the hallucinated span.
- **Scale & scope**: 12,000 test cases spanning **summarization, QA, and style transfer**.
- **The gap it fills**: single-step LLM-as-a-judge factuality self-assessment is opaque and coarse; even with ground truth it under-detects. Process decomposition exposes *which* stage fails.
- **Diagnostic, not just a leaderboard**: the value is attribution — locating the breaking stage — over a single aggregate score.
- **Kinship with process supervision**: mirrors the reasoning-eval shift from outcome-level to step-level scoring (cf. process reward models).

## Concepts & entities

- [[hallucination-detection]] — the task PROBE benchmarks (a process, decomposed into four steps).
- [[hallucination]] — the phenomenon being detected.
- [[agent-benchmark]] — PROBE is an evaluation benchmark; [[retrieval-augmented-generation]] — evidence-finding is a retrieval step.
- [[reliability-without-validity-llm-as-a-judge-norman-2026]] — companion critique of single-step LLM-as-a-judge evaluation.

## References

- OpenReview: https://openreview.net/forum?id=NuSq4FmzFb
- _Original (provenance-only) source: `sources/probe-process-benchmark-hallucination-detection-2026.md`_
