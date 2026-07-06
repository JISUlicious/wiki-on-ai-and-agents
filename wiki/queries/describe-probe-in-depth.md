---
title: "Describe PROBE in depth"
type: query
created: 2026-07-06
updated: 2026-07-06
sources:
  - probe-process-benchmark-hallucination-detection-2026.md
  - reliability-without-validity-llm-as-a-judge-norman-2026.md
status: complete
importance: medium
tags:
  - 2026
---

# Describe PROBE in depth

An in-depth account of **[[probe-process-benchmark-hallucination-detection-2026|PROBE]]** (PROcess-Based BEnchmark for Hallucination Detection; Findings of ACL 2026, pp. 42303–42320) — Yu Zhang & Bei Yu ([[chinese-university-of-hong-kong|CUHK]]) with an [[nvidia|NVIDIA]] team.

## The core reframing

Existing [[hallucination-detection]] is **outcome-based**: a single "[[reliability-without-validity-llm-as-a-judge-norman-2026|LLM-as-a-judge]]" pass emits a binary "hallucinated?" verdict. PROBE argues this is opaque and coarse — even *given ground-truth evidence*, one-shot judging under-detects, and it can't say *where or why* it failed. So PROBE treats detection as a **four-stage process and scores each stage individually**, mirroring the reasoning field's shift from outcome-level to **process/step-level supervision** (ProcessBench, PRMBench, FG-PRM). It generalizes prior detection benchmarks — RAGTruth (word-level) and HaluBench (sample-level binary) — into a diagnostic, per-step framing.

## The four steps

| Step | Tests | Method | Finding |
|---|---|---|---|
| **1. Claim decomposition** | break an output into atomic checkable claims | prompt → match vs ground-truth claims | **easy** — recall **>95%** across frontier LLMs |
| **2. Evidence finding (EF)** | locate supporting evidence per claim | Partial (≥1) vs Complete (all) match | **the bottleneck** — Partial ~80% but Complete ~63–70% |
| **3. Evidence evaluation (EE)** | judge whether evidence supports a claim | binary vote, reuse EF evidence to isolate judgment | sub-optimal even given evidence — Claude-Sonnet-4.5 only **69.8%** |
| **4. Hallucination localization** | pinpoint the hallucinated span | claim-wise, **character-level P/R/F1** | fine-grained (vs response/word-level) |

Localization labels: **Factual-Supported**, **Hallucinated-Unsupported**, **Hallucinated-Unverifiable**.

## The dataset

- **12,000 test cases / 118,628 annotated claims** (claim + evidence level) — considerably larger than prior hallucination-detection datasets.
- **Three RAG-style tasks over Wikipedia**: summarization, QA, style transfer.
- **Construction**: baseline generation → **hallucination injection** → claim-evidence annotation (Llama-3.1-70B as the annotation engine).
- **Statistics**: ~21.7% claims hallucinated (25,703) vs 78.3% grounded (92,925). Task asymmetry: long summarization/style-transfer answers have low response-level hallucination (15.13% / 16.93%), but **short QA answers are far more hallucination-dense (47.42%)** — concise answers are brittle. Hallucinated spans avg ~230 chars, in ~29.96% of responses — motivating claim-level not response-level scoring.
- **Split**: 100 instances/task held out; the rest **released as fine-tuning data** for EF/EE.

## Headline results

- **Decomposition helps everyone** — treating detection as multi-step beats one-shot outcome-based prompting for *all* models.
- **The failure is localizable** — the weakness is **evidence finding** (then evidence evaluation), not decomposition/localization.
- **Step-wise supervision fixes the weak step** — a fine-tuned **Llama-3.1-8B (SFT) beats the far larger frontier models** on EF/EE (Partial-EF 83.2/87.4/84.1 vs Claude-Sonnet-4.5's 81.6/86.4/81.6). Targeted training on the bottleneck beats scale.

## Positioning

- The [[hallucination]]-detection analogue of **process reward models** for reasoning — score the *process*, not just the outcome.
- The constructive complement to [[reliability-without-validity-llm-as-a-judge-norman-2026]]: that shows single-step LLM-judges *overstate* reliability (33–41pp kappa gap); PROBE shows *what to do instead* — decompose and localize.
- Step 2 being the bottleneck ties detection back to **[[retrieval-augmented-generation|retrieval]]**: "evidence finding" is fundamentally a retrieval problem, so retrieval failures masquerade as detection failures.

## References

- [[probe-process-benchmark-hallucination-detection-2026]] — the source (full ACL Anthology text).
- [[hallucination-detection]] · [[reliability-without-validity-llm-as-a-judge-norman-2026]] · [[hallucination]] · [[retrieval-augmented-generation]]
