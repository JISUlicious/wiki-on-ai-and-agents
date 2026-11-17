---
title: "Can LLM Agents Organize Their Own Memory?"
type: source
created: 2026-11-17
updated: 2026-11-17
sources:
  - llm-agents-organize-own-memory-fu-2026.md
arxiv_id: "2611.11929"
venue: arXiv preprint (cs.AI)
authors:
  - Zichuan Fu
  - Junjia Qi
  - Wenlin Zhang
  - Yimin Deng
  - Shirong Wang
  - Jinyang Li
  - Xian Wu
  - Xiangyu Zhao
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Can LLM Agents Organize Their Own Memory?

**Source:** Zichuan Fu, Junjia Qi, Wenlin Zhang, Yimin Deng, Shirong Wang, Jinyang Li, Xian Wu, Xiangyu Zhao (City University of Hong Kong; Tencent Jarvis Lab) — [arXiv:2611.11929](https://arxiv.org/abs/2611.11929), cs.AI, v1 9 Nov 2026. Code: github.com/Qi202/SoMem.

## Summary

SoMem (self-organized memory) stores an agent's long-term memory as linked Markdown documents, close in spirit to an [[llm-wiki]]. New sessions are archived verbatim as immutable **Source** records. An LLM Writer then edits **Topic** documents: it decides what to summarize, how to group it, and when to merge, split or move topics. Every statement carries a footnote with its event time and the Source records that support it. A small, size-bounded **Core** file holds standing preferences and is placed in every query's context.

The split of labour is the key design choice. The model controls semantic organization; deterministic runtime code enforces the invariants. Edits are staged, validated (Source unchanged, references resolve, valid times, paths in the workspace) and installed atomically, or rejected. After each accepted edit the runtime rebuilds three access views from Topic: a **Graph** of paragraph links and backlinks, a **Timeline** of dated files, and a **Recent** view. At query time a Reader agent gets Core, Recent and a file inventory, then searches ([[bm25|BM25]] or dense) and reads files, going back to Source evidence when the summaries are not enough.

## Key points

- **Main results** (GPT-6 Luna as Writer and Reader for every method). [[locomo|LoCoMo]] 88.96% vs 86.30% for E-mem, the best baseline (+2.66), and 84.22% for full context. BEAM-100K nugget score 54.02 vs 50.92 for full context (+3.10). [[longmemeval|LongMemEval-S]] 76.50% vs 74.50% for LightMem (+2.00).
- Baselines include [[a-mem|A-Mem]], [[mem0|Mem0]], MemoryOS, Nemori, LightMem and E-mem.
- **Ablations on LoCoMo.** Removing Source drops accuracy most (88.96 → 80.32); removing Topic drops it to 86.04. Removing Core, Recent, Timeline or Graph changes overall accuracy by at most 1.30 points, but Timeline and Graph each cost temporal questions (88.16 → 86.92).
- **Memory is portable.** A workspace built by one Writer serves other Readers without rebuilding. Memory built by GPT-5.6 Luna gives 92.3–93.3% across four Readers.
- **Backbone choice matters a lot.** With DeepSeek V4 Flash as Writer and Reader, LongMemEval-S rises to 90.20%.
- **Reader effort.** More reasoning effort generally helps, more on BEAM's longer histories, but saturates or reverses at the top settings. On BEAM, raising GPT-6 Luna from High to xHigh improved scores while using fewer searches, reads and tokens.

## Caveats

Main comparisons use one shared Reader with no reasoning effort. The LLM judge is GPT-4o-mini. Gains over the best baseline are 2–3 points.

## Concepts & entities

- [[long-term-memory]] · [[memory-construction]] · [[memory-evolution]] · [[semantic-memory]] · [[graph-memory]] — agent-organized, evidence-linked memory.
- [[llm-wiki]] · [[zettelkasten-memory]] · [[a-mem]] · [[mem0]] · [[memgpt]] — related document- or note-based designs.
- [[locomo]] · [[longmemeval]] · [[bm25]] · [[reasoning-effort]].
- [[corpusmap-follow-the-entities-jeong-2026]] · [[memorytax-bitter-lesson-agent-memory-peng-2026]] — 2026 papers on linked-page memory vs raw-record search.

## References

- [arXiv:2611.11929](https://arxiv.org/abs/2611.11929)
