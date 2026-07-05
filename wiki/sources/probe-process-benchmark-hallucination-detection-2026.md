---
title: "PROBE: PROcess-Based BEnchmark for Hallucination Detection"
type: source
created: 2026-07-05
updated: 2026-07-06
sources:
  - probe-process-benchmark-hallucination-detection-2026.md
authors:
  - Yu Zhang
  - Peter Belcak
  - Shizhe Diao
  - Yonggan Fu
  - Shaona Ghosh
  - Morteza Mardani
  - Eileen Long
  - Bei Yu
  - Pavlo Molchanov
first_author: Yu Zhang
introduces:
  - [[hallucination-detection]]
year: 2026
venue: "Findings of ACL 2026 (2026.findings-acl.2099, pp. 42303–42320)"
tags:
  - 2026
  - benchmark
status: complete
importance: medium
---

# PROBE: PROcess-Based BEnchmark for Hallucination Detection

**Venue**: Findings of the ACL 2026 (pp. 42303–42320) — [PDF](https://aclanthology.org/2026.findings-acl.2099.pdf) · [OpenReview NuSq4FmzFb](https://openreview.net/forum?id=NuSq4FmzFb)
**Authors**: Yu Zhang & Bei Yu ([[chinese-university-of-hong-kong|CUHK]]); Peter Belcak, Shizhe Diao, Yonggan Fu, Shaona Ghosh, Morteza Mardani, Eileen Long, Pavlo Molchanov ([[nvidia|NVIDIA]]). *(Yu Zhang did this during an NVIDIA internship.)*

_Requested by the user for ingest; full text from the ACL Anthology PDF._

## Summary

**PROBE** reframes [[hallucination-detection]] as a **process to be evaluated step-by-step** rather than a single pass/fail judgment. It decomposes detection into **four critical steps — claim decomposition → evidence finding → evidence evaluation → hallucination localization — and scores each individually**, so a system's failure can be attributed to *where and why* it broke. The benchmark has **12,000 test cases with 118,628 annotated claims** (annotated at both claim and evidence level) across **three RAG-style task types over Wikipedia articles: summarization, question answering, and style transfer**.

Its target is the dominant single-step **"[[reliability-without-validity-llm-as-a-judge-norman-2026|LLM-as-a-judge]]"** pattern: even with ground-truth evidence, one-shot self-assessment under-detects hallucinations and offers no transparency into failure. PROBE's headline empirical result is that **when detection is treated as a multi-step process, *all* models do considerably better** than one-shot outcome-based prompting — and that **evidence finding is the primary bottleneck**, which **fine-tuning on PROBE's released training data substantially improves**. It is the [[hallucination]]-detection analogue of the reasoning field's shift to step-level / process supervision (ProcessBench, PRMBench, FG-PRM), and it advances prior detection benchmarks RAGTruth (word-level) and HaluBench (sample-level binary) to a diagnostic, per-step framing.

## Key points

- **Four evaluated sub-steps** (each scored on its own): (1) **claim decomposition** — break output into verifiable atomic claims; (2) **evidence finding** — locate truthful, relevant evidence; (3) **evidence evaluation** — distinguish relevant from irrelevant evidence; (4) **hallucination localization** — pinpoint the exact hallucinated span (labels: *Factual-Supported*, *Hallucinated-Unsupported*, *Hallucinated-Unverifiable*).
- **Dataset**: 12,000 cases / 118,628 claims across summarization, QA, style transfer (Wikipedia, [[retrieval-augmented-generation|RAG]] setting). Built via baseline generation → hallucination *injection* → claim-evidence annotation. ~21.7% of claims hallucinated (78.3% grounded); QA answers are shorter but far more hallucination-dense (47.42%); hallucinated spans average ~230 chars.
- **Central finding — decompose and all models improve**: process-based evaluation yields substantially higher detection accuracy than one-shot outcome-based prompting, and gives *actionable* per-step diagnosis.
- **The bottleneck is evidence finding**, not claim decomposition or localization — the retrieval-flavored step is where current LLMs fail; step-wise supervision (fine-tuning on the released training data) targets exactly it.
- **Models evaluated**: Llama-3.1-70B, GPT-4o-mini, Mixtral-8×22B, Claude-Sonnet-4.5 (Llama-3.1-70B also used as the annotation/evidence engine).
- **Transparency as the goal**: attribution of *which* stage failed, not just a binary verdict — a step toward diagnosable, robust detection.

## Concepts & entities

- [[hallucination-detection]] — the task PROBE benchmarks (a process, decomposed into four steps).
- [[hallucination]] — the phenomenon; [[retrieval-augmented-generation]] — evidence finding is a retrieval step (and the tasks are RAG-style).
- [[agent-benchmark]] — PROBE is an evaluation benchmark.
- [[reliability-without-validity-llm-as-a-judge-norman-2026]] — companion critique of single-step LLM-as-a-judge evaluation.
- [[nvidia]], [[chinese-university-of-hong-kong]] — authoring institutions.

## References

- ACL Anthology: https://aclanthology.org/2026.findings-acl.2099.pdf
- _Original source: `sources/probe-process-benchmark-hallucination-detection-2026.md` (extracted full text)_
