---
title: Generative Agents
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - generative-agents-park-2023.md
status: complete
importance: high
tags:
  - 2023
---

# Generative Agents

Generative agents are LLM-backed simulated humans that maintain a persistent natural-language memory and act believably over multi-day horizons. Park et al.'s architecture is the canonical reference for the **recency × importance × relevance** retrieval triad and for **reflection** as an LLM-driven consolidation primitive. The Smallville sandbox (25 agents) demonstrated emergent social behaviors — information diffusion, relationship formation, party coordination from a single seed prompt — and ablations show memory, reflection, and planning each contribute independently to believability.

## Core mechanism

- **[[memory-stream]]**: append-only, timestamped log of natural-language observations; every perception or action becomes an entry. The substrate for everything downstream.
- **Retrieval scoring**: `score = α_recency · recency + α_importance · importance + α_relevance · relevance` with min-max normalization to [0,1]; all αs = 1 in the paper.
  - *Recency*: exponential decay since last access.
  - *Importance*: LLM-rated 1–10 score assigned at write time.
  - *Relevance*: cosine similarity between memory embedding and query embedding.
- **[[reflection-mechanism]]**: triggered when summed importance of recent observations crosses a threshold. The agent generates salient questions, retrieves memories per question, and prompts the LLM for abstract inferences ("Klaus is passionate about research") that are written back into the stream.
- **Reflection trees**: reflections can themselves be reflected on, yielding a tree of progressively higher abstractions over raw observations.
- **Hierarchical planning**: top-down daily plan → hour blocks → 5–15-minute actions; plans live in the memory stream and re-plan on perceived disruption.

## What it advances

Generative agents contribute to [[memory-management]] across **store**, **consolidate**, and **retrieve**:

- **Store**: every observation lands in the memory stream with a timestamp and an LLM-scored importance rating — importance is computed at write time, not retrieval time, which is the key affordance enabling later weighted retrieval.
- **Consolidate**: reflection is an explicit, periodic LLM-driven consolidation pass that synthesizes higher-level memories from lower-level ones and writes them back into the same stream. Recursive reflection produces a multi-level abstraction hierarchy.
- **Retrieve**: the three-signal weighted scoring (recency × importance × relevance) is the paper's signature contribution and has become the standard baseline for LLM-agent memory retrieval.

The architecture has no compact, evict, or curate operations — the stream grows without bound, which is acceptable for the 2-day Smallville simulations but a known limitation for production-scale agents.

## Relation to other systems

- **vs. [[memgpt]]** — MemGPT adds OS-style paging and explicit eviction policies (compact + evict) over a similar NL memory substrate; generative agents have neither and rely on retrieval to surface what matters from an ever-growing log.
- **vs. [[voyager]]** — Voyager consolidates trial-and-error into [[procedural-memory]] (executable code); generative agents consolidate observations into declarative abstractions (reflections). Different memory subsystems, different consolidation primitives.
- **vs. naive RAG over chat history** — Plain vector retrieval uses relevance alone. The recency × importance × relevance blend is what makes long-horizon character coherence possible: it surfaces old-but-defining memories that pure cosine similarity would miss.

## References

- Source: [[generative-agents-park-2023]]
- arXiv: <https://arxiv.org/abs/2304.03442>
- Venue: UIST 2023
