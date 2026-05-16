---
title: Mem0
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - mem0-chhikara-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Mem0

Mem0 (Chhikara et al., 2025; ECAI 2025) is a production-deployment-focused long-term memory architecture for [[llm-agent]]s that addresses fixed-context-window limits in multi-session dialogue via an incremental **extract → consolidate → retrieve** pipeline. Its core contribution is letting the LLM itself act as the judge for memory edits (no separate classifier), and demonstrating that a small set of consolidated facts yields **91% lower p95 latency and >90% token-cost savings vs. full-context** baselines on [[locomo]], with a 26% relative LLM-as-Judge improvement over OpenAI ChatGPT memory. A graph variant, **Mem0^g**, layers a Neo4j-backed knowledge graph for relational and temporal queries.

## Core mechanism

- **Extract**: from each new `(user, assistant)` message pair, an LLM extractor consumes a sliding window of recent messages plus an async-refreshed conversation summary and emits candidate facts.
- **Consolidate**: for each candidate, retrieve top-10 semantically similar memories, then prompt an LLM "tool call" to pick one of `ADD` / `UPDATE` / `DELETE` / `NOOP` — heuristic conflict-resolution and dedup come free from LLM reasoning.
- **Retrieve**: at query time, fetch top-k relevant facts and inject into the prompt; facts are concise compared to RAG chunks, so context-assembly cost is small.
- **Graph variant (Mem0^g)**: entity extractor + relationship generator produce `(entity, relation, entity)` triplets stored in Neo4j; obsolete relationships are marked invalid (not deleted) to preserve temporal reasoning. Retrieval combines entity-centric subgraph walks with semantic triplet matching.
- **Heuristic, not learned**: the operator selection is in-context prompting of a vanilla LLM — there is no RL signal tied to downstream answer correctness.

## What it advances

- [[memory-management]]: defines the canonical `ADD/UPDATE/DELETE/NOOP` operator set that later RL-trained systems ([[memory-r1]]) inherit.
- Production cross-session memory: extract/consolidate/retrieve pipeline with massive latency and token-cost reductions over full-context baselines.
- Graph memory for agents: a working pattern for combining vector similarity with relational subgraph retrieval in a single memory layer.

## Relation to other systems

- **vs. [[memgpt]] / [[memorybank]]**: Mem0 abandons hierarchical paging in favor of a flat fact store with LLM-judged consolidation; simpler, cheaper, and the SOTA reference design through 2025–2026.
- **vs. [[a-mem]]**: both are heuristic (LLM-prompted, no learning signal); Mem0 is more production-pragmatic, A-MEM emphasizes Zettelkasten-style note linking.
- **vs. [[memory-r1]]**: same operator set, but Memory-R1 *learns* the operation policy with PPO/GRPO and downstream-QA reward, where Mem0 relies on in-context instructions. The *learned-vs-heuristic* axis runs directly through this pair.
- **vs. [[mirix]]**: Mem0 stores a flat list of consolidated facts; MIRIX decomposes memory into six cognitive-science-inspired types managed by separate agents.

## References

- Source: [[mem0-chhikara-2025]]
