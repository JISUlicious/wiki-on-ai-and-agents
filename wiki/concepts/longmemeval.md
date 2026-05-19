---
title: LongMemEval
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - grep-all-you-need-sen-2026.md
status: draft
importance: medium
tags:
  - 2025
---

# LongMemEval

**LongMemEval** is a benchmark for long-term interactive memory in chat assistants, introduced by Wu et al. at ICLR 2025. It measures whether an agent can correctly recall, integrate, and reason over information that surfaced **across multiple prior sessions** — i.e., whether memory survives compaction, eviction, and retrieval over long horizons.

This stub is a placeholder; the canonical benchmark paper has not yet been ingested. The reference came in via [[grep-all-you-need-sen-2026]], which uses a 116-question LongMemEval-S subset to evaluate the joint effect of retriever, harness, and tool-result delivery on agentic-search accuracy.

## Core mechanism

- **Multi-session chat corpus** as the haystack; each question targets evidence buried in earlier sessions.
- **Six task categories** spanning factual recall, multi-hop reasoning, temporal reasoning, and integration across sessions.
- **LongMemEval-S** is a smaller 116-question slice used in efficiency-conscious evaluations (e.g., Sen et al. 2026).

## What it advances

The benchmark formalizes what every memory-system paper since 2023 has informally tried to test — whether the agent's retrieval / consolidation / curation pipeline actually surfaces the right facts when the corpus grows. It is now the de-facto eval for systems like [[memgpt]], [[memorybank]], [[hipporag-2]], [[a-mem]], [[mem0]], [[mirix]], [[memory-r1]], [[mem-alpha]], and [[chronos]].

## Relation to other concepts

- [[memory-management]] — the umbrella; LongMemEval is the eval target.
- [[agentic-search]] — Sen et al. 2026 reuses LongMemEval as the testbed for joint retriever × harness measurement.
- [[gaia]] — adjacent general-assistant benchmark; LongMemEval is memory-specific, GAIA is tool-use-general.

## References

- ICLR 2025 paper (not yet ingested in this wiki).
- Cited by [[grep-all-you-need-sen-2026]] as the primary benchmark.
