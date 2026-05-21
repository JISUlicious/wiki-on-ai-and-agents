---
title: Chronos
type: concept
created: 2026-05-21
updated: 2026-05-21
sources:
  - chronos-sen-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Chronos

**Chronos** is a temporal-aware conversational agent for long-term memory, introduced by Sen et al. ([[pwc|PwC]]) in March 2026 ([[chronos-sen-2026]]). It is also the **custom-harness baseline** in their later [[grep-all-you-need-sen-2026|Is Grep All You Need?]] study — where the same Claude Opus 4.6 backbone hits **93.1% on Chronos vs. 76.7% on [[claude-code]]** under matched conditions.

## Core mechanism

- **Dual calendars**: events calendar + turns calendar; events stored as `<subj, verb, obj>` tuples with ISO 8601 ranges + 2–4 lexical aliases.
- **Dynamic prompting**: a Gemini-3-Flash meta-prompt generates per-question search hints + tool-use guidance before the [[react|ReAct]] loop begins.
- **Hybrid retrieval pipeline**: top-100 vector → Cohere Rerank v3 → top-15 → ±1 turn-neighbor expansion. Pre-fills retrieval *before* the tool-calling loop.
- **Four tools** in the loop: `search_turns`, `search_events`, `grep_turns`, `grep_events`. Agent decides which to invoke.

## What it advances

- **State of the art on [[longmemeval|LongMemEval-S]]**: 92.60% (Chronos-Low, GPT-4o) / 95.60% (Chronos-High, Claude Opus 4.6).
- The events calendar is the dominant ablation contributor (34.5-point drop on Low when removed) — empirical evidence that **structured temporal-event representation beats flat chat history** for long-horizon QA.
- Serves as the canonical "custom harness" baseline in the harness-vs-retriever joint study, demonstrating that custom SDK harnesses can outperform provider-native CLI harnesses by ~16 points on the same backbone.

## Relation to other concepts

- [[memory-management]] — Chronos is an L3 implementation of the L2 memory-management abstraction in [[agent-three-layer-model]].
- [[agentic-search]] — the umbrella concept this work helped name.
- [[longmemeval]] — the benchmark used.
- [[grep-all-you-need-sen-2026]] — the companion study that uses Chronos as a baseline.
- [[react]] — the underlying loop.
- [[pwc]] — the authors' org.

## References

- [[chronos-sen-2026]] — primary source
- [[grep-all-you-need-sen-2026]] — companion paper using Chronos as a harness baseline
