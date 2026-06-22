---
title: "AtomMem: Building Simple and Effective Memory System for LLM Agents via Atomic Facts"
type: source
created: 2026-06-22
updated: 2026-06-22
sources:
  - atommem-memory-system-via-atomic-facts-yao-2026.pdf
arxiv_id: "2606.19847"
authors:
  - Yanyu Yao
  - et al.
first_author: Yanyu Yao
year: 2026
tags: [2026]
status: complete
importance: medium
---

# AtomMem: Building Simple and Effective Memory System for LLM Agents via Atomic Facts

*Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-14–06-21).*

## Summary

AtomMem is a long-term [[memory-management]] system for LLM agents built around **atomic facts** as the base unit of memory. It targets a core dilemma in memory-augmented agents: storing raw conversations preserves information but floods retrieval with noise, while condensed summaries are compact but discard fine-grained detail and accumulate LLM-generated noise. AtomMem aims for both high information density and contextual fidelity.

The pipeline centers on a **Fact Executor** (an SFT-tuned extractor) that selectively pulls high-value, self-contained atomic facts from long-form conversations, resolving coreference and temporal references. These facts are organized into **hierarchical event structures** (coherent episodic contexts) and **temporal user profiles** that incrementally track stable attributes and adapt to preference shifts. At retrieval time, an **associative memory graph** reconnects fragmented memories by linking facts through entity overlap and shared events. On the LoCoMo benchmark it reports state-of-the-art performance across reasoning tasks; a simplified flat-retrieval fact-level variant is competitive at minimal cost while the full modular design adds further gains.

## Key points

- Base representation is the **atomic fact**: self-contained, coreference-resolved, temporally anchored — denser than raw dialogue, more faithful than summaries.
- **Fact Executor** (supervised fine-tuned) extracts and verifies high-value atomic facts from noisy long-form conversations.
- Facts are consolidated into **hierarchical event memory** plus **temporal profile memory** that tracks evolving user state.
- Retrieval activates an **associative memory graph** connecting facts via entity overlap and shared events for hierarchical hybrid recall.
- Reports SOTA on **LoCoMo** (interactions averaging 600+ turns) and is also evaluated on LongMemEval; baselines include Mem0, A-MEM, MemoryBank, MemGPT.
- Token-efficient: a variant uses the lowest token count (~722k) among compared methods while delivering strong per-query performance.

## Concepts & entities

- [[memory-management]]
- [[locomo]]
- [[mirix]]
- [[mem0]]

## References

- [arXiv:2606.19847](https://arxiv.org/abs/2606.19847)
