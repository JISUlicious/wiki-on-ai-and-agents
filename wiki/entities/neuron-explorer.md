---
title: neuron-explorer
type: entity
created: 2026-10-27
updated: 2026-10-27
sources:
  - neuron-explorer-lyramakesmusic-2026.md
  - neuron-explorer-reactions-2026-10.md
tags:
  - product
founders:
  - [[lyra-c]]
builds-on:
  - [[steering]]
  - [[logit-lens]]
  - [[vllm]]
  - [[mcp]]
status: complete
importance: medium
---

# neuron-explorer

neuron-explorer is an open-source, **steering-first** neuron browser for Qwen3.5-4B ([[qwen]]), released by [[lyra-c|Lyra C]] on GitHub on 2026-10-21. Within a week it had about 1.3k stars and 148 forks. The full write-up is on [[neuron-explorer-lyramakesmusic-2026]].

## What it does

- **Sweep (offline, on a GPU).** The sweep runs on a steering fork of [[vllm]] from [[goodfire]]. For each MLP neuron of a layer it adds `dose · W_down[:, n]`, sized relative to the [[residual-stream]] norm. It runs three prompts at −0.7, +0.5 and +0.8, plus a free prompt at +3, and keeps every rollout. Each layer also gets 256 random directions of equal norm as a null.
- **Ranking.** Neurons are ranked by **word-cluster mass**: how strongly words over-represented in a neuron's rollouts stand out against the random controls.
- **Lenses.** Each neuron has a [[logit-lens]] reading of its output weights, plus a **steer lens**: the next-token change in log-probability and the KL divergence when the neuron is pushed ±0.8.
- **Local explorer** (port 8791; runs on MPS, CUDA or CPU):
  - a 3D map of the real architecture with one point per neuron;
  - a ranked list;
  - a neuron panel with rollouts and a live −/0/+ steered chat;
  - "fires" (your own text → which neurons activate);
  - links to other neurons, precomputed from the weights;
  - UMAP of rollout embeddings.
- **Agent tab.** A [[claude-code]] CLI session that uses the explorer as an [[mcp]] server. It can inspect neurons, run steering and ablations on the live model, and drive the user interface. It also has file and shell tools in the user's home folder, which is worth knowing before running it.

## Why it matters

- **Labels neurons by effect.** It labels every neuron by its **causal effect on generation**, not only by its top activations. That makes it a practical test bed for the claim that individual neurons are readable, Cammarata's "already clean" thesis (see [[monosemanticity]]).
- **Random controls guard against false labels.** Without them, cheap name flips ("I am Grok", "trained by Google") would pass as persona neurons.
- **The explorer produced [[evil-qwen]].** That two-neuron recipe, which flips Qwen3.5-4B's safety behaviour, went viral on 2026-10-24.

## Derivatives

All appeared within days of the evil-qwen post:
- a Gemma 4 E4B / Gemma 3 4B port, only tested on random-weight models;
- a from-scratch GGUF/llama.cpp reimplementation that exports control vectors and runs its own MCP server;
- a Qwen3-0.6B live neuron map on a Coral NPU board;
- an Apple Silicon fork.

Details are in [[neuron-explorer-lyramakesmusic-2026]].

## References

- [[neuron-explorer-lyramakesmusic-2026]]
- [[layer-depth-hierarchy-cnn-vs-llm]]
