---
title: MIRIX
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - mirix-wang-2025.md
status: complete
importance: high
tags:
  - 2025
---

# MIRIX

MIRIX (Wang & Chen, 2025) is a modular **multi-agent memory system** for [[llm-agent]]s that operationalizes a six-type, cognitive-science-inspired memory taxonomy into a working architecture. Eight specialized agents — one Meta Memory Manager, six Memory Managers (one per memory type), and one Chat Agent — coordinate writes and reads against six structurally distinct stores. It also introduces multimodal memory ingestion from screenshots and a new benchmark, **ScreenshotVQA**, where it achieves 35% higher accuracy than a SigLIP RAG baseline with 99.9% less storage, plus 85.38% on [[locomo]] (SOTA, ~8 pts above LangMem/Zep).

## Core mechanism

- **Six memory types**, each with its own schema and dedicated Memory Manager agent: **Core Memory** (persistent persona blocks, MemGPT-style), **Episodic Memory** (time-stamped events), **Semantic Memory** (concepts/entities organized as a tree), **Procedural Memory** (how-to workflows with explicit step lists), **Resource Memory** (full/partial working documents), **Knowledge Vault** (sensitive credentials/keys with sensitivity labels).
- **Eight-agent coordination**: a Meta Memory Manager routes incoming inputs to relevant Memory Managers, which update in parallel; a separate Chat Agent handles user-facing dialogue and consults the memory base before responding.
- **Active Retrieval**: on every turn the Chat Agent generates a topic from the input, queries top-10 entries from each of the six memory components, tags retrieved content by source (e.g., `<episodic_memory>...</episodic_memory>`), and injects them into the system prompt — no explicit "search memory" trigger needed.
- **Multimodal ingestion**: screenshots are captured at ~1.5 s intervals, deduplicated by visual similarity, batched (groups of 20), and distilled into the six structured stores via Gemini async image upload; raw images are not retained.
- **Heuristic, not learned**: like [[mem0]] and [[a-mem]], MIRIX uses prompted-LLM control flow for memory operations — no RL signal tied to downstream task performance.

## What it advances

- [[memory-management]]: introduces typed, compositional memory (six-way split) versus a flat fact store, with each type managed by a dedicated agent.
- Multi-agent memory: Meta Memory Manager + six Memory Managers + Chat Agent as a coordination pattern.
- Multimodal memory abstraction: distilling continuous visual streams into structured textual memory rather than retaining raw frames.

## Relation to other systems

- **vs. [[mem0]] / [[a-mem]] / [[memgpt]]**: MIRIX's paper explicitly critiques these for flat-store architecture lacking compositional memory structure, poor multimodal support, and no abstraction layer above raw inputs.
- **vs. [[mem-alpha]]**: both adopt a multi-component memory (core/episodic/semantic appear in both); MIRIX builds it as a hand-designed multi-agent system with heuristic prompting, where Mem-α trains *construction* over a 3-component schema with RL — learned vs. heuristic, but converging architecturally.
- **vs. CoALA**: MIRIX operationalizes the cognitive-architecture memory typology that [[cognitive-architectures-for-language-agents|CoALA]] (Sumers et al., 2023) proposed conceptually.

## References

- Source: [[mirix-wang-2025]]
