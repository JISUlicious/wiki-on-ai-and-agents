---
title: Titans
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - titans-behrouz-2025.md
status: complete
importance: high
tags:
  - 2025
---

# Titans

**Titans** is a family of architectures that adds a **neural long-term memory module whose weights are updated at test time** by a surprise-driven gradient signal. Rather than retrieving from an external store, Titans absorbs context by performing implicit [[online-learning]] during inference — surprising tokens generate large gradient updates to the memory module's parameters; predictable tokens are absorbed cheaply. Three variants (MAC / MAL / MAG) compose this memory with a short-window attention "core" and a bank of fixed persistent parameters.

## Core mechanism

- **Surprise = gradient magnitude** of an associative-memory loss `ℓ(M; x_t) = ||M(k_t) - v_t||²`. Tokens whose key→value mapping the memory currently mispredicts trigger larger weight updates.
- **Test-time weight updates**: the long-term memory's parameters are *not frozen after pretraining* — they evolve token-by-token via mini-batch SGD with momentum and weight decay. Equivalent to running a tiny inner optimisation loop while the rest of the network does forward passes.
- **Past + momentary surprise**: the update rule splits into a momentum-carried "past surprise" term and a "momentary surprise" term, both gated by data-dependent coefficients so context shifts can reset the running surprise signal.
- **Three memory subsystems**: (1) **Core** — short-window attention as working memory; (2) **Long-term Memory** — the test-time-updated deep MLP; (3) **Persistent Memory** — fixed-after-pretraining task-knowledge bank.
- **Three composition variants**: **MAC** (Memory as Context), **MAL** (Memory as a Layer), **MAG** (Memory as a Gated branch). A data-dependent forgetting gate (generalising Mamba/DeltaNet gates) prevents saturation on >2M-token streams.

## What it advances

Titans contributes the **parametric memory** branch to [[memory-management]]: *write*, *store*, and *evict* operations are realised as gradient updates and forgetting-gate decay on a neural submodule's weights, not as text insertions to a database. The novel write op is "gradient step on surprise"; the novel evict op is "weight decay scaled by current surprise". This is the first architecture to make long-term memory a first-class, online-learnable parametric component co-located with attention.

## Relation to other systems

- **Physical substrate — weights vs. external store**: [[memgpt]] and [[mem0]] write summarised text/facts to an external DB and retrieve via similarity search. Titans changes its own weights instead. Tradeoff: opaque parameter deltas (not user-inspectable rows) but no retrieval latency and tight integration with the forward pass.
- **Physical substrate — weights vs. KV-cache**: [[em-llm]] also keeps memory inside the model, but as preserved KV activations segmented by surprise; the base LLM stays frozen. Titans goes further — it *modifies the model itself* during inference.
- **Update mechanism — gradient vs. fact-extraction**: [[hipporag-2]] consolidates via OpenIE triple extraction into a [[knowledge-graph]]; Titans consolidates via mini-batch SGD on an associative-memory loss. Both treat surprise/novelty as the write signal, but only Titans makes the signal differentiable end-to-end.

## References

- [[titans-behrouz-2025]] — Behrouz, Zhong, Mirrokni, *Titans: Learning to Memorize at Test Time*, 2025 ([arXiv:2501.00663](https://arxiv.org/abs/2501.00663))
