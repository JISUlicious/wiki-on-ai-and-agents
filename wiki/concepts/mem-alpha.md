---
title: Mem-α
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - mem-alpha-wang-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Mem-α

Mem-α (Wang et al., 2025) is a reinforcement-learning framework that trains LLM agents to **construct multi-component memory systems** from raw input streams, rather than to edit an existing flat memory. The agent ingests context chunks one at a time and emits structured tool calls against a memory store with **core (≤512-token persistent summary), semantic (atomic facts), and episodic (timestamped events)** components. Trained with [[grpo]] and a composite reward (QA correctness, tool-call format validity, compression, content quality) on 562 instances of up to 30k tokens, a Qwen3-4B agent generalizes to **474k-token inputs — over 13× the training length** — outperforming MemAgent, MEM1, and long-context/RAG baselines on MemoryAgentBench while using ~50% the memory footprint.

## Core mechanism

- **Three-component memory architecture**: core (single ≤512-token summary, `memory_update` only — holistic rewrite); semantic (atomic factual statements, insert/update/delete); episodic (timestamped events, insert/update/delete).
- **GRPO training over construction tool calls** (KL term dropped to encourage exploration; group-relative advantages over 8 rollouts per prompt). The backbone is Qwen3-4B; 32 H100s × 3 days × 205 steps.
- **Four-term reward**: `r = r1 + r2 + β·r3 + γ·r4` — r1 (BM25-RAG QA correctness over final memory), r2 (tool-call format validity), r3 (compression: 1 − memory/chunk length), r4 (memory-content quality judged by Qwen3-32B).
- **Decoupled RAG evaluator**: a frozen [[bm25]] retriever + frozen generator answers held-out questions over the constructed memory — only the *write policy* is learnable, sidestepping the need for memory ground truth.
- **Length extrapolation 30k → 400k+ tokens (13×)**: RL teaches *memory-management principles* rather than surface patterns; small models (4B) trained this way outperform GPT-4.1-mini with the same framework but no RL.

## What it advances

- [[memory-management]]: shifts the RL target from *editing* an existing memory ([[memory-r1]]) to *constructing* a structured one from scratch — including deciding which component (core/semantic/episodic) a new piece of information belongs in.
- Length generalization: demonstrates that RL-trained memory construction extrapolates far beyond the training distribution (13×), a property absent from heuristic and supervised baselines.
- Small-model viability: a 4B parameter agent with the right training can beat much larger heuristically-prompted agents on long-context memory tasks.

## Relation to other systems

- **vs. [[memory-r1]]**: both use RL (GRPO) over memory operations, but the *construction-vs-editing* axis separates them. Memory-R1 learns an editor over a flat fact memory inheriting [[mem0]]'s `ADD/UPDATE/DELETE/NOOP` operators; Mem-α learns a constructor over a multi-component (core/episodic/semantic) schema.
- **vs. MEM1 / MemAgent**: those works also use RL but train flat, single-paragraph rewritable memories (closer to compressed RNN-style state); Mem-α's structured multi-component memory scores 0.592 average on MemoryAgentBench vs. their 0.071 / 0.198.
- **vs. [[mirix]]**: architecturally convergent (multi-component memory with core/episodic/semantic), but MIRIX is a hand-designed multi-agent system using heuristic LLM prompting, while Mem-α uses a single agent with an RL-trained construction policy. The *learned-vs-heuristic* axis applies again.
- **vs. [[memgpt]] / [[mem0]]**: those provide elaborate tool sets and rely on the base model to figure out what to store and how; Mem-α treats the same problem as a sequential decision problem and trains the policy.

## References

- Source: [[mem-alpha-wang-2025]]
