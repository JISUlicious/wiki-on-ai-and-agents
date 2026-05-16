---
title: "Titans: Learning to Memorize at Test Time"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - titans-behrouz-2025.md
arxiv_id: "2501.00663"
authors:
  - Ali Behrouz
  - Peilin Zhong
  - Vahab Mirrokni
first_author: Ali Behrouz
year: 2025
introduces:
  - "[[titans]]"
  - "[[test-time-memory]]"
  - "[[neural-long-term-memory]]"
tags:
  - 2025
status: complete
importance: high
---

# Titans: Learning to Memorize at Test Time

- **Source**: `sources/titans-behrouz-2025.md`
- **arXiv**: [arXiv:2501.00663](https://arxiv.org/abs/2501.00663)
- **Authors**: Ali Behrouz, Peilin Zhong, Vahab Mirrokni (Google Research)
- **Year**: 2025

## Summary

Titans introduces a **neural long-term memory module** that learns *at test time* — its parameters are updated online, during inference, by a **surprise signal** defined as the gradient magnitude of an associative-memory loss (`||M(k_t) - v_t||²`). Tokens that the current memory fails to predict produce large gradients, which drive larger weight updates; predictable tokens are absorbed cheaply. The mechanism is formulated as mini-batch gradient descent with momentum and weight decay applied to the memory module's own parameters, yielding a fast, parallelizable training rule while still giving recurrent-style inference.

The architecture composes three distinct memory subsystems: (1) **Core** — short-window attention acting as working/short-term memory; (2) **Long-term Memory** — the test-time-updated neural module that stores compressed abstractions of past context; (3) **Persistent Memory** — a bank of learnable, data-independent parameters that encode task-level knowledge. Titans presents three variants for wiring these together: memory as context (MAC), memory as a layer (MAL), and memory as a gated branch (MAG). A data-dependent **forgetting gate** (generalizing the gates of Mamba/recent linear RNNs) governs decay, preventing memory saturation on very long streams.

This work stakes out a **distinct design axis** from external-store memory systems like [[memgpt]] and [[mem0]]: rather than retrieving from a vector DB or RAG-style sidecar, Titans pushes long-term memory into **parametric weights updated online**. The model "remembers" by changing its own neural memory's weights as it reads — a form of [[online-learning]] at inference. Experiments cover language modeling, common-sense reasoning, recall-intensive tasks, needle-in-a-haystack, time-series, and DNA modeling; Titans outperforms modern linear RNNs and matches/beats Transformers at equal context, while scaling to **>2M token contexts** with higher needle-in-haystack accuracy than baselines.

## Key Points

- **Surprise = gradient magnitude** of the associative-memory loss `ℓ(M; x_t) = ||M(k_t) - v_t||²`; tokens whose key→value mapping the memory mispredicts trigger larger weight updates.
- **Past surprise + momentary surprise**: the update rule splits surprise into a momentum-carried "past surprise" term and a current "momentary surprise" term, both gated by data-dependent coefficients (η_t, θ_t) so context shifts can reset the running surprise signal.
- **Online weight updates during inference**: the long-term memory's parameters are not frozen after pretraining — they evolve token-by-token at test time. Equivalent to running mini-batch SGD with momentum + weight decay on the memory module while the rest of the network runs forward passes.
- **Three memory types**: attention (Core / short-term), neural long-term memory (test-time-updated), and persistent memory (fixed-after-pretraining task knowledge).
- **Three Titan variants**: Memory as Context (MAC), Memory as a Layer (MAL), Memory as a Gated branch (MAG).
- **Adaptive forgetting gate**: generalizes the forget gates in Mamba/DeltaNet/Gated DeltaNet; decay is a function of memory size and current surprise.
- **Deep memory matters**: the long-term memory is itself a multi-layer MLP (not a single vector/matrix), answering the paper's Q5 — linear-state recurrent compression is insufficient for very long histories.
- **Scales beyond 2M tokens** with maintained needle-in-haystack accuracy, while remaining competitive on standard LM/reasoning benchmarks.

## Counterpoint to External-Store Memory

The dominant agent-memory paradigm (see [[memgpt]], [[mem0]], [[rag]]) treats memory as **external state**: text or embeddings written to a store and retrieved into the context window. Titans is the opposing pole — memory lives **inside the weights of a neural submodule** that is updated by gradient descent during inference. Tradeoffs:

| Axis | External store (MemGPT/Mem0) | Parametric online (Titans) |
|---|---|---|
| Substrate | Text / embeddings in a DB | Weights of a neural module |
| Update | Explicit write tool calls | Implicit gradient step on surprise |
| Retrieval | Similarity search + paging | Forward pass through memory module |
| Editability by user | Direct (inspectable rows) | Opaque (parameter deltas) |
| Capacity vs. context | Unbounded, paged | Fixed-capacity, decayed by forgetting gate |

## Entities Mentioned

- [[google-research]]

## Concepts Discussed

- [[titans]]
- [[test-time-memory]]
- [[neural-long-term-memory]]
- [[memory-management]]
- [[long-context-llm]]
- [[online-learning]]
- [[transformer-architecture]]
- [[linear-attention]]
- [[state-space-model]]

## Notable Quotes

> "We present a new neural long-term memory module that learns to memorize historical context and helps an attention to attend to the current context while utilizing long past information... attention due to its limited context but accurate dependency modeling performs as a short-term memory, while neural memory due to its ability to memorize the data, acts as a long-term, more persistent, memory."

> "Inspired by human long-term memory system, we design this memory module so an event that violates the expectations (being surprising) is more memorable. To this end, we measure the surprise of an input with the gradient of the neural network with respect to the input in associative memory loss."

## References

_Original source: `sources/titans-behrouz-2025.md`_
