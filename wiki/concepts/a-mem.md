---
title: A-MEM
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - a-mem-xu-2025.md
status: complete
importance: high
tags:
  - 2025
---

# A-MEM

A-MEM ("Agentic Memory") is a self-organizing memory system for [[llm-agent|LLM agents]] proposed by Xu et al. (2025, NeurIPS 2025, Rutgers). It applies the **Zettelkasten** note-taking philosophy to agent memory: each new interaction becomes a structured note with LLM-generated attributes, and an LLM-driven analysis step decides how the note links into — and *rewrites* — the surrounding memory network.

## Core mechanism

- **Structured note schema**: each memory `m_i = {c_i, t_i, K_i, G_i, X_i, e_i, L_i}` — original content, timestamp, LLM-generated keywords, tags, contextual description, dense embedding, and link set. Atomic and self-contained.
- **Link generation**: a new note's embedding retrieves top-k semantic neighbors via [[embedding-retrieval]]; the LLM then judges which connections to actually establish based on shared attributes, not raw similarity alone.
- **Memory evolution** (the novel contribution): for each retrieved neighbor `m_j`, the LLM is prompted with the new memory and the neighborhood and decides whether to rewrite `m_j`'s context, keywords, and tags. The evolved `m_j*` replaces the original. This breaks the append-only assumption of nearly all prior agent-memory systems.
- **Multi-box membership**: linked notes form Zettelkasten-style "boxes" of related ideas, but a single memory can belong to multiple boxes simultaneously — no predefined schema.
- **Implementation**: `all-MiniLM-L6-v2` encoder, default top-k = 10; structured-output prompting via LiteLLM/native APIs.

## What it advances

Primarily **curate** in the [[memory-management]] vocabulary: A-MEM is the first agent memory system in which **stored memories are mutated by subsequent experience**. Insertion is no longer purely additive — neighbor notes' contexts, keywords, and tags are rewritten in light of the new memory. It also contributes a structured **write** operation (notes-with-attributes rather than flat dialog spans) and a learned-graph **store** layer that emerges from content rather than from a predefined schema.

## Relation to other systems

- **vs. [[memgpt]]** and [[memorybank]]: both are append-only — once stored, items are only retrieved, summarized, or evicted, never rewritten. A-MEM's evolution step is what they lack.
- **vs. [[mem0]]**: Mem0 uses a graph database with a fixed schema; A-MEM lets structure emerge from content, allowing notes to belong to multiple "boxes" without rigid typing.
- **vs. agentic RAG**: agentic RAG exercises agency at *retrieval* time over a static knowledge base; A-MEM exercises agency at *storage and evolution* time over a self-modifying memory.

## References

- [[a-mem-xu-2025]]
