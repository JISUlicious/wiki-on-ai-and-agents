---
title: "MIRIX: Multi-Agent Memory System for LLM-Based Agents"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - mirix-wang-2025.md
arxiv_id: "2507.07957"
authors:
  - Yu Wang
  - Xi Chen
first_author: Yu Wang
year: 2025
introduces:
  - "[[mirix]]"
  - "[[multi-agent-memory]]"
tags:
  - 2025
status: complete
importance: high
---

# MIRIX: Multi-Agent Memory System for LLM-Based Agents

**Source**: `sources/mirix-wang-2025.md`
**arXiv**: [arXiv:2507.07957](https://arxiv.org/abs/2507.07957)
**Authors**: Yu Wang, Xi Chen (MIRIX AI; affiliations listed via UCSD and NYU Stern emails)
**Year**: 2025 (July)
**Project**: https://mirix.io/

> Note: this Yu Wang (yuw164@ucsd.edu, MIRIX AI) is distinct from the Wang who authored the Mem-α paper — both work on agent memory but they are separate researchers.

## Summary

MIRIX is a modular, multi-agent memory system that operationalizes a **six-type memory taxonomy** for LLM-based agents: **Core Memory**, **Episodic Memory**, **Semantic Memory**, **Procedural Memory**, **Resource Memory**, and **Knowledge Vault**. Each memory type is structurally distinct (different schemas, fields, retention policies) and is managed by its own dedicated *Memory Manager* agent. A central **Meta Memory Manager** routes incoming inputs to the relevant Memory Managers, which then update their respective stores in parallel. A separate **Chat Agent** handles user-facing dialogue and consults the memory base before responding. In total the system uses eight specialized agents, and is among the first works to translate the cognitive-science-inspired memory typology (popularized by [[cognitive-architectures-for-language-agents]] / CoALA and related works) into a working agent stack rather than a single flat memory store.

A second contribution is **multi-modal memory**: MIRIX is designed to ingest screenshots (a stream of ~one image every 1.5 seconds, deduplicated) and distill them into structured memory entries rather than retaining raw images. The authors introduce **ScreenshotVQA**, a new benchmark built from 5,000–20,000 high-resolution screenshots collected over one month from three PhD students, with manually authored questions. MIRIX achieves a **35% accuracy improvement over a SigLIP RAG baseline while cutting storage by 99.9%** (SigLIP@50: 15.07 GB; MIRIX SQLite DB: 15.89 MB overall), and a **410% improvement over a long-context Gemini baseline with 93.3% storage reduction**. On the textual long-form conversation benchmark **LOCOMO**, MIRIX reaches **85.38% overall LLM-as-a-Judge accuracy** — state-of-the-art, ~8 points above the strongest prior memory system (LangMem at ~78%, Zep at 75.14%), and approaching the Full-Context upper bound of 87.52%.

The paper also proposes an **Active Retrieval** mechanism: rather than waiting for an explicit "search your memory" instruction, the Chat Agent first generates a *topic* from the input, then queries the top-K entries from each of the six memory components, tags retrieved content by source (e.g. `<episodic_memory>...</episodic_memory>`), and injects them into the system prompt. The team additionally ships a desktop application (React + Electron + Uvicorn) that builds a personal memory from local screen monitoring with local SQLite storage.

## Key Points

- **Six memory types, each with its own agent**:
  - **Core Memory** — persistent persona/human blocks (MemGPT-style); rewritten when capacity exceeds 90%.
  - **Episodic Memory** — time-stamped events with fields `event_type / summary / details / actor / timestamp`.
  - **Semantic Memory** — abstract concepts and entities (`name / summary / details / source`); organized as a tree (e.g. Social Network → Favorites → Sports/Pets/Music).
  - **Procedural Memory** — how-to workflows / guides / scripts with explicit `steps` lists.
  - **Resource Memory** — full or partial documents the user is working with (`title / summary / resource_type / content`).
  - **Knowledge Vault** — verbatim sensitive data (credentials, addresses, API keys) with explicit `sensitivity` levels and access control.
- **Eight agents total**: one Meta Memory Manager + six Memory Managers + one Chat Agent. The Meta Memory Manager routes input; Memory Managers update in parallel.
- **Active Retrieval**: agent generates a topic for *every* turn, then retrieves top-10 entries per memory component automatically — no explicit user trigger required. Multiple retrieval functions supported (`embedding_match`, `bm25_match`, `string_match`).
- **Multi-modal**: screenshots are captured at ~1.5 s intervals, deduplicated by visual similarity (>0.99 threshold skipped), batched into groups of 20, then distilled into the six memory stores via Gemini's async image upload. Raw images are *not* retained.
- **ScreenshotVQA results** (LLM-as-a-Judge accuracy, overall across three students):
  - Gemini long-context (256×256 resize): 11.66%, 236.70 MB
  - SigLIP@50 RAG: 44.10%, 15.07 GB
  - **MIRIX: 59.50%, 15.89 MB** (smallest storage by 3 orders of magnitude vs SigLIP).
- **LOCOMO results** (LLM-as-a-Judge, gpt-4.1-mini backbone): MIRIX 85.38% overall — Single-Hop 85.11, Multi-Hop **83.70 (+24 points over next-best)**, Open-Domain 65.62, Temporal 88.39. Full-Context upper bound: 87.52%.
- **Storage backend**: SQLite for local persistence. Backbone models: `gemini-2.5-flash-preview-04-17` for screenshots, `gpt-4.1-mini` for LOCOMO (chosen for stronger function-calling than `gpt-4o-mini`, per Berkeley Function Calling Benchmark).
- **Motivation against flat memory**: paper criticizes [[mem0]], [[memgpt]], LangMem, [[zep]], and Cognee for lacking compositional memory structure, poor multi-modal support, and no abstraction layer above raw inputs.
- **Forward-looking framing**: paper proposes an "Agent Memory Marketplace" vision with privacy-preserving, decentralized memory sharing — speculative, but signals product/business ambitions of the MIRIX AI startup.

## Entities Mentioned

- **MIRIX AI** — the startup behind the system (Yu Wang, Xi Chen).
- **UCSD**, **NYU Stern** — author affiliations via email domains.
- **Gemini** ([[gemini]]) — used as multimodal backbone for screenshot processing.
- **GPT-4.1-mini** — used as backbone for LOCOMO experiments.
- **SigLIP** — vision encoder used as the RAG retrieval baseline.
- Baseline systems compared: [[mem0]], [[memgpt]] (MemGPT/Letta), **LangMem**, [[zep]], **Memobase**, **A-Mem** ([[a-mem-xu-2025]]).

## Concepts Discussed

- [[mirix]] — the six-component, eight-agent memory architecture introduced here.
- [[multi-agent-memory]] — Meta Memory Manager + six Memory Managers + Chat Agent coordination pattern.
- [[memory-typology]] — six-way decomposition: Core / Episodic / Semantic / Procedural / Resource / Knowledge Vault.
- [[episodic-memory-llm]] — time-stamped events with structured fields.
- [[semantic-memory]] — abstract concepts and entity knowledge.
- [[procedural-memory]] — how-to workflows and scripts.
- [[memory-management]] — controlled rewrites, capacity thresholds, deduplication.
- [[cognitive-architectures-for-language-agents]] (CoALA) — referenced as the source of structured memory taxonomy that MIRIX operationalizes.
- [[active-retrieval]] — topic-generation-then-retrieve mechanism (versus reactive "search your memory" calls).
- [[multimodal-memory]] — screenshot ingestion with abstraction (no raw image retention).
- [[locomo]] — long-form conversation benchmark; MIRIX SOTA.
- [[screenshotvqa]] — new benchmark introduced by the paper.

## Notable Quotes

> "MIRIX consists of six distinct, carefully structured memory types: Core, Episodic, Semantic, Procedural, Resource Memory, and Knowledge Vault, coupled with a multi-agent framework that dynamically controls and coordinates updates and retrieval."

> "MIRIX achieves 35% higher accuracy than the RAG baseline while reducing storage requirements by 99.9% [...] On LOCOMO, MIRIX attains state-of-the-art performance of 85.4%, far surpassing existing baselines."

> "Most approaches store all historical data in a single flat store without routing into specialized memory types (e.g., procedural, episodic, semantic), making retrieval inefficient and less accurate."

## References

_Original source: `sources/mirix-wang-2025.md`_
