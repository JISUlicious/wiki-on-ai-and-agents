---
title: "Chronos: Temporal-Aware Conversational Agents with Structured Event Retrieval for Long-Term Memory"
type: source
created: 2026-05-21
updated: 2026-05-21
sources:
  - chronos-sen-2026.md
arxiv_id: "2603.16862"
authors:
  - Sahil Sen
  - Elias Lumer
  - Anmol Gulati
  - Vamse Kumar Subbiah
first_author: Sahil Sen
year: 2026
introduces:
  - "[[chronos]]"
tags:
  - 2026
status: complete
importance: high
---

# Chronos: Temporal-Aware Conversational Agents with Structured Event Retrieval for Long-Term Memory

[arXiv:2603.16862](https://arxiv.org/abs/2603.16862) — Sen, Lumer, Gulati, Subbiah ([[pwc|PricewaterhouseCoopers]] Commercial Technology and Innovation Office), 2026.

## Summary

Chronos is a custom long-horizon conversational-memory harness built around the insight that pure turn-level retrieval is **temporally blind** while comprehensive knowledge-graph extraction introduces **context entropy** through over-structuring. Rather than treating long-term memory as a flat chat history or a global entity graph, Chronos performs **query-conditioned selective extraction**: it decomposes raw dialogue into `<subject, verb, object>` event tuples with resolved ISO 8601 datetime ranges and entity aliases, indexed in a structured **event calendar**, while preserving the raw dialogue in a parallel **turn calendar** for semantic retrieval. The result is the minimal sufficient abstraction for conversational memory — structure exactly what LLMs struggle with (temporal deltas, event sequences, date arithmetic) and leave the rest as natural language turns.

At query time, Chronos runs a **dynamic prompting** stage (an LLM meta-prompt that analyzes the question and emits 1–5 bullets of retrieval guidance: which entities, attributes, temporal constraints, and operations to focus on), pre-fills the agent's context with the **top-15 reranked turns** from a vector + cross-encoder pipeline (Cohere Rerank v3 over a top-100 candidate pool, each turn expanded with one neighbor before and after for conversational continuity), and then hands control to a [[react|ReAct]]-style tool-calling loop. The agent has four tools — `search_turns`, `search_events`, `grep_turns`, `grep_events` — letting it iteratively refine retrieval by datetime range, cross-reference events against source dialogue, and fall back to grep when vector similarity misses exact entities.

Chronos serves as the **custom harness baseline** in the companion paper [[grep-all-you-need-sen-2026|"grep is all you need"]] study by the same PwC team: it is the in-house long-horizon agent that outperformed provider-native CLI harnesses (Claude Code, Codex CLI, Gemini CLI) on [[longmemeval|LongMemEval]]-S when evaluated under matched conditions. On the full LongMemEval-S benchmark (500 questions, six categories), **Chronos Low** (GPT-4o) reaches **92.60%** accuracy — a 7.67% relative gain over the previous best (EmergenceMem Internal at 86%) — and **Chronos High** (Claude Opus 4.6) reaches **95.60%**, a 3.02% relative gain over the prior record (Mastra's OM). Ablations isolate the events calendar as the single largest contributor: removing it nearly halves accuracy for Chronos Low (34.5-point drop).

## Key Points

- **Dual calendars, not flat history.** Two parallel indexes: an **event calendar** of `<subj, verb, obj>` tuples with ISO 8601 datetime ranges, and a **turn calendar** of raw exchanges. Both are embedded with `text-embedding-3-large` and queried independently.
- **Event-based memory representation.** Events are extracted in batches of 25 turns with 5-turn overlap. Each event gets 2–4 lexical aliases (e.g., "bought Fitbit" → "picked up a fitness tracker," "got a step counter") to robustify keyword search across paraphrases.
- **Multi-resolution temporal normalization.** Natural-language time references ("recently," "last month," "the week after my vacation") are resolved into start/end datetime windows centered on the conversation timestamp, converting fuzzy temporal language into executable date-range filters.
- **Dynamic prompting (query-conditioned guidance).** A small model (Gemini 3 Flash) generates a per-question preamble — *not* a rewritten query — telling the agent which information dimensions and retrieval strategies matter. Extends [[retrieval-augmented-generation|RAG]]-style query rewriting from document retrieval to long-term memory.
- **Pre-filled vector top-15 before tool-calling.** Initial retrieval runs vector search (top-100) → Cohere Rerank v3 → top-15 → ±1-turn context expansion, organized by session date. This seeds the agent before its [[react|ReAct]] loop begins, cutting redundant in-loop searches.
- **Four agent tools.** `search_turns`, `search_events` (vector); `grep_turns`, `grep_events` (exact match). Reranking on dense retrieval is applied to the **original question**, not the agent's reformulated query.
- **Matched LongMemEval-S subset scoring.** Same 500-question protocol and LLM-judge as the canonical benchmark; the paper also flags benchmark-side issues (ambiguous ground truth, LLM-as-judge variance) that the team raised against the LongMemEval repo.
- **Ablation: events calendar dominates.** Removing events drops Chronos Low by 34.5 points; removing dynamic prompting + initial retrieval drops by 14–16 points. Under Chronos High, dynamic prompting contributes ~0 — stronger backbones absorb the routing job.
- **State-of-the-art on LongMemEval-S.** 92.60% (GPT-4o) / 95.60% (Claude Opus 4.6); biggest gains on multi-session aggregation (91.73%) and temporal reasoning (90.23%).

## Notable Quotes

> "By structuring exactly what LLMs struggle with (temporal deltas, event sequences, date calculations) and leaving the rest as natural language turns, Chronos achieves the minimal sufficient abstraction for conversational memory."

> "Rather than reformulating the search query, Chronos analyzes each question and generates tailored retrieval guidance for the agent."

> "Retrieval failures remain the most common error category for both models, showing that even with specific guidance, LLMs still fail to retrieve over large amounts of data."

> "Persistent conversational agents do not require comprehensive knowledge graph construction to achieve high-fidelity long-horizon memory."

## Entities

- [[pwc]] — Commercial Technology and Innovation Office; institutional author of both Chronos and the companion grep-vs-vector study.

## Concepts

- [[memory-management]] — Chronos is a concrete memory-management architecture for multi-month dialogue.
- [[agentic-search]] — Iterative tool-calling over dual indexes with vector + grep tools.
- [[longmemeval|LongMemEval]] — Primary evaluation benchmark; Chronos sets SOTA on the -S subset.
- [[react|ReAct]] — Underlying reasoning pattern for the Chronos Agent's tool-calling loop.
- [[retrieval-augmented-generation|RAG]] — Dynamic prompting extends RAG-era query rewriting to conversational memory.
- [[dense-retrieval]] — Initial retrieval stage uses dense vector search with cross-encoder reranking.

## References

- [[grep-all-you-need-sen-2026]] — Companion paper from the same PwC team; uses Chronos as the "custom harness" baseline that beat provider-native CLI agents on LongMemEval.
- [[longmemeval]] — Benchmark definition and prior-state-of-the-art comparison points.
- [[pwc]] — Institutional context for both papers.
