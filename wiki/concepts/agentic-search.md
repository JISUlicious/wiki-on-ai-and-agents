---
title: Agentic Search
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - grep-all-you-need-sen-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Agentic Search

**Agentic search** is the in-loop variant of retrieval-augmented generation. Rather than running a fixed RAG pipeline (embed → top-k → concatenate → prompt), an agent **decides** what to search, how many queries to issue, when to refine vs. re-query, and whether the retrieved evidence is sufficient. The retrieval calls are interleaved with the [[react|reason → act → observation]] loop and gated by the agent's harness.

The concept is named in [[grep-all-you-need-sen-2026]] (Sen et al., 2026) — the paper that systematically measures how retrieval strategy, harness choice, and tool-result delivery mode jointly determine end-to-end accuracy on long-horizon QA.

## Core mechanism

- **Retrieval as a tool call**, not a pipeline stage. The agent's `search` tool is one of many it can choose to invoke; the model decides whether/when to use it.
- **Iterative refinement.** Top-k results are inputs to the next reasoning step, not the final answer. The agent can re-query with refined terms, switch retrievers, or grep into a returned file.
- **Three coupled design choices** (per Sen et al. 2026):
  1. **Retriever**: lexical (grep / [[bm25]] / regex) vs. dense ([[dense-retrieval]]) vs. hybrid.
  2. **Harness**: custom SDK ([[chronos]], LangChain) vs. provider-native CLI ([[claude-code]], Codex CLI, Gemini CLI).
  3. **Tool-result delivery**: inline-stdout vs. file-based (agent must `cat`/`read` to consume).
- **Harness effect ≈ retriever effect.** Same backbone + same retriever can differ by ~20 accuracy points across harnesses (e.g., 93.1% Chronos vs. 76.7% Claude Code with grep-only Claude Opus 4.6).

## What it advances

- Reframes RAG ablation studies: fixing the harness obscures the joint design surface. A retriever that wins in isolation may lose in-loop, and vice versa.
- **Lexical search rehabilitated.** Inline grep beats inline vector for **every** harness-model pair tested in Sen et al. 2026 — challenging the default-to-embeddings stance of much agentic-RAG literature.
- Treats **provider-native CLI agents** ([[claude-code]], Codex, Gemini CLI) as a distinct architectural class because their shell-mediated tool surface changes which retrieval strategies are effective.

## Relation to other concepts

- [[retrieval-augmented-generation]] — agentic search is its in-loop generalization.
- [[memory-management]] — long-horizon agentic-search is one mechanism for the **retrieve** lifecycle operation in the memory-management taxonomy.
- [[agent-three-layer-model]] — agentic search sits across L2 [[memory-management]], L2 action surface (the search tool), and L2 harness (which mediates tool delivery).
- [[react]] — the loop pattern that contains the retrieval calls.
- [[bm25]] / [[dense-retrieval]] / [[hipporag-2]] — specific retrieval mechanisms that can plug in.
- [[longmemeval]] — the canonical evaluation benchmark for this class of system.

## References

- [[grep-all-you-need-sen-2026]] — the canonical paper naming and measuring agentic search.
- [[chronos]] — the authors' custom harness.
- [[longmemeval]] — benchmark for long-horizon agentic-memory QA.
