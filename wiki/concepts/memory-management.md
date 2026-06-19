---
title: Memory Management
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - reflexion-shinn-2023.md
  - generative-agents-park-2023.md
  - memorybank-zhong-2023.md
  - voyager-wang-2023.md
  - memgpt-packer-2023.md
  - em-llm-fountas-2024.md
  - titans-behrouz-2025.md
  - a-mem-xu-2025.md
  - hipporag2-jimenez-gutierrez-2025.md
  - sleep-time-compute-lin-2025.md
  - mem0-chhikara-2025.md
  - mirix-wang-2025.md
  - memory-r1-yan-2025.md
  - mem-alpha-wang-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Memory Management

Memory management is the operational layer that gives [[llm-agent|LLM agents]] persistence and accumulation over time. It is **not** the same as retrieval: retrieval (RAG, dense or sparse) is *one operation* — pulling information into the active context — within a wider lifecycle of writing, storing, compacting, consolidating, evicting, and curating.

Treating memory management as a first-class problem (rather than "let's add a vector DB") is the conceptual move that distinguishes 2023–2026 agent systems from earlier RAG pipelines. [[rag-lewis-2020|Lewis et al. 2020]] introduced a single-shot query → retrieve → generate loop. Modern agents — Claude Code, opencode, Hermes, Letta, Mem0 — manage memory as a *living resource* across many sessions.

## The lifecycle

A useful operational vocabulary, drawn from across the canonical works below:

| Operation | Question it answers | Example mechanism |
|---|---|---|
| **Write** | What should be recorded at all? | Selective surfacing in [[mem0]]; "important" scoring in [[generative-agents]] |
| **Store** | Where does it live (which tier)? | [[memgpt]] main/recall/archival; hot/warm/cold partitioning |
| **Retrieve** | When and how is it recalled? | Embedding similarity, [[bm25]] / lexical, graph PPR ([[hipporag-2]]), KV-cache events ([[em-llm]]) |
| **Compact** | How is the same information made smaller? | Summarization, queue eviction, context engineering |
| **Consolidate** | When does ephemeral become durable? | [[reflection-mechanism]] in [[generative-agents]], [[sleep-time-compute]] offline pass, [[memory-evolution]] in [[a-mem]] |
| **Evict / Archive** | What should be removed? | [[ebbinghaus-forgetting]] decay ([[memorybank]]), surprise-gated forgetting ([[titans]]), learned `DELETE` ([[memory-r1]]) |
| **Curate** | How is stored memory edited / kept fresh? | Neighbor updates in [[a-mem]]; conflict resolution + dedup in [[mem0]] |

Retrieval is one row of this table — important but not the whole problem.

## Design axes (where modern systems differ)

Across the canonical works the field branches along several near-orthogonal axes:

1. **External store vs. parametric memory.** [[memgpt]], [[mem0]], [[a-mem]], [[mirix]] keep memory in external stores (SQLite, vector DBs, files). [[titans]] pushes memory into model weights *updated at test time*. [[em-llm]] keeps it *inside the KV-cache*. These are fundamentally different physical substrates with different cost/latency/expressivity tradeoffs.
2. **Flat vs. structured.** Flat append-only logs ([[reflexion]], early [[mem0]]) vs. structured stores: graph ([[hipporag-2]], [[a-mem]]), Zettelkasten ([[a-mem]]), typed-by-role ([[mirix]] with six memory types).
3. **Heuristic vs. learned policy.** Heuristic ops in [[mem0]] and [[a-mem]] vs. RL-learned ops in [[memory-r1]] (editing) and [[mem-alpha]] (construction).
4. **Single-agent vs. multi-agent memory.** Most systems are single-agent; [[mirix]] dedicates one agent per memory type.
5. **Memory typology.** Some systems treat memory as one homogeneous bag ([[mem0]] core variant). Others differentiate **episodic** (events with timestamps), **semantic** (atomic facts), **procedural** (executable skills), **resource** (raw artifacts), **vault** (secrets), and **core** (always-loaded), drawing on Tulving's classical typology — explicit in [[mirix]] and [[mem-alpha]].

## Canonical works (the 2023→2026 arc)

**Foundation (2023)** — established the basic patterns:

- [[reflexion]] ([[reflexion-shinn-2023|Shinn 2023]]) — episodic verbal memory across retry attempts; no weight updates.
- [[generative-agents]] ([[generative-agents-park-2023|Park 2023]]) — *memory stream + reflection* with retrieval scored by **recency × importance × relevance**; the standard recipe.
- [[memorybank]] ([[memorybank-zhong-2023|Zhong 2023]]) — first principled **forgetting policy** (Ebbinghaus curve).
- [[voyager]] ([[voyager-wang-2023|Wang 2023]]) — [[skill-library]] as **procedural memory**: ever-growing library of executable skills.
- [[memgpt]] ([[memgpt-packer-2023|Packer 2023]]) — OS-style **virtual context** with tiered memory; the LLM itself emits function calls to page memory.

**Recent (2024–2026)** — diversification across the design axes:

- [[em-llm]] ([[em-llm-fountas-2024|Fountas 2024]]) — Bayesian-surprise event segmentation *inside the KV-cache*. No external store.
- [[titans]] ([[titans-behrouz-2025|Behrouz 2025]]) — neural long-term memory updated *at test time* via surprise signal. Memory in weights, not files.
- [[a-mem]] ([[a-mem-xu-2025|Xu 2025]]) — Zettelkasten-style **memory evolution**: new memories trigger updates to neighbor representations. Breaks the append-only assumption.
- [[hipporag-2]] ([[hipporag2-jimenez-gutierrez-2025|Jiménez Gutiérrez 2025]]) — hippocampus/neocortex separation, **graph memory** with Personalized PageRank retrieval.
- [[sleep-time-compute]] ([[sleep-time-compute-lin-2025|Lin & Snell 2025]]) — **offline consolidation** in idle time between queries. 5× test-time compute reduction, +18% accuracy.
- [[mem0]] ([[mem0-chhikara-2025|Chhikara 2025]]) — production cross-session memory with extract/consolidate/retrieve pipeline. Reference design adopted by many 2025–2026 agent stacks.
- [[mirix]] ([[mirix-wang-2025|Wang & Chen 2025]]) — six memory types × dedicated agent each × multimodal. Operationalizes the CoALA typology.
- [[memory-r1]] ([[memory-r1-yan-2025|Yan 2025]]) — RL-trained policy over `{ADD, UPDATE, DELETE, NOOP}`. Learns *when to forget*.
- [[mem-alpha]] ([[mem-alpha-wang-2025|Wang 2025]]) — RL-trained memory **construction** with extreme length generalization (30k train → 400k+ deploy).

## Relation to other tracks

- **Retrieval-augmented generation** ([[retrieval-augmented-generation]], [[dense-retrieval]], [[bm25]]) provides the *retrieve* operation but does not specify the write/store/consolidate/evict policy. Memory management is the superset.
- **Long-context modeling** is partially substitutable: a 10M-token context makes some forms of memory unnecessary. But infinite context is uneconomic; [[em-llm]] and [[titans]] both treat long context as a *memory representation*, not a substitute.
- **[[mechanistic-interpretability]]** intersects through [[em-llm]] (events as KV-cache structure) and [[titans]] (parametric updates) — both ask "where physically does memory live in the model?"
- **Reasoning** ([[chain-of-thought-prompting]], [[tree-of-thoughts]]) uses memory as scratchpad; [[reflexion]] is the bridge — reasoning *across* episodes via stored reflections.

## Production agent memory in 2026

The 2026 coding-agent ecosystem (Claude Code, opencode, pi-mono, Hermes, OpenClaw) mixes these primitives:

- **Lexical retrieve**: `grep`/`ripgrep`/SQLite FTS5 dominates for code (function names, exact tokens) — closer to [[bm25]] than to dense.
- **Hierarchical file memory**: `CLAUDE.md` / `AGENTS.md` / `MEMORY.md` / `USER.md` auto-loaded at session start — closest analogue is [[memgpt]]'s "core" tier plus the [[mirix]] Core memory type.
- **Cross-session memory**: persistent memory files updated across runs — [[mem0]]-style pipeline.
- **Auto-consolidation**: background "auto-dream" passes — direct application of [[sleep-time-compute]].
- **Skill libraries**: agent-defined tools/skills stored and reused — [[voyager]]-style procedural memory.
- **Multi-agent**: subagent dispatch with per-agent context — partial overlap with [[mirix]].

Dense vector retrieval ([[dense-retrieval]]) appears as one option among many, not the foundation.

## References

- [[reflexion-shinn-2023]]
- [[generative-agents-park-2023]]
- [[memorybank-zhong-2023]]
- [[voyager-wang-2023]]
- [[memgpt-packer-2023]]
- [[em-llm-fountas-2024]]
- [[titans-behrouz-2025]]
- [[a-mem-xu-2025]]
- [[hipporag2-jimenez-gutierrez-2025]]
- [[sleep-time-compute-lin-2025]]
- [[mem0-chhikara-2025]]
- [[mirix-wang-2025]]
- [[memory-r1-yan-2025]]
- [[mem-alpha-wang-2025]]

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]] for full grouping.

- [[adaptive-loops-and-memory-in-transformers-frey-2026]] — Adaptive Loops and Memory in Transformers Think Harder
- [[do-language-models-need-sleep-lee-2026]] — Do Language Models Need Sleep? Offline Recurrence for
- [[lightthinker-plus-plus-zhu-2026]] — LightThinker++ From Reasoning Compression to Memory Management
- [[memcollab-chang-2026]] — MemCollab Cross Model Memory Collaboration via Contrastive Trajectory
- [[memex-rl-wang-2026]] — Memex(RL) Scaling Long Horizon LLM Agents via Indexed
- [[memfactory-guo-2026]] — MemFactory Unified Inference & Training Framework for Agent
- [[memory-intelligence-agent-qiao-2026]] — Memory Intelligence Agent
- [[memory-transfer-learning-coding-agents-kim-2026]] — Memory Transfer Learning How Memories are Transferred Across
- [[ocr-memory-optical-context-retrieval-li-2026]] — OCR Memory Optical Context Retrieval for Long Horizon
- [[parammem-parametric-reflective-memory-yao-2026]] — ParamMem Augmenting Language Agents with Parametric Reflective Memory
- [[stateless-decision-memory-for-enterprise-ai-agents-srinivasan-2026]] — Stateless Decision Memory for Enterprise AI Agents
- [[memory-systems-anatomy-2026]] — Memory Systems Anatomy (2026)
