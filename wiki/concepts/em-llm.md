---
title: EM-LLM
type: concept
created: 2026-05-16
updated: 2026-05-16
sources:
  - em-llm-fountas-2024.md
status: complete
importance: high
tags:
  - 2024
---

# EM-LLM

**EM-LLM** is a training-free episodic-memory architecture that lets pre-trained [[long-context-llm|long-context LLMs]] handle effectively infinite streams by structuring the [[kv-cache]] itself into surprise-delimited events. It imports the algorithmic interpretation of **human episodic memory** — Bayesian-surprise segmentation, two-stage similarity+contiguity recall — directly into the transformer's attention substrate, with no fine-tuning of the base model.

## Core mechanism

- **Online Bayesian-surprise segmentation**: a token $x_t$ is a candidate event boundary when its negative log-likelihood exceeds a sliding-window mean plus $\gamma\sigma$. No extra forward passes needed — the model's own next-token loss is the segmentation signal.
- **Graph-theoretic boundary refinement**: per-head key-similarity treated as a weighted adjacency; boundaries are shifted to maximise modularity (best) or minimise conductance, so each event has cohesive intra-event keys and clean inter-event separation.
- **KV-cache-internal episodic memory**: events are *contiguous spans of past KV pairs* inside the cache — not text or embeddings in an external store. Episodic structure becomes a property of the context itself.
- **Two-stage retrieval**: (1) similarity buffer — k-NN over per-event representative tokens against the current query; (2) contiguity buffer — a queue that auto-enqueues temporally adjacent events of each retrieved hit, reproducing human free-recall contiguity/asymmetry effects.
- **Per-layer retrieval**: each transformer layer independently retrieves its own k events (contrast: RAG performs a single up-front retrieval). This is credited with most of the gain over [[retrieval-augmented-generation|RAG]].

## What it advances

EM-LLM contributes a new operation to the [[memory-management]] taxonomy: **event segmentation and retrieval performed inside the KV-cache**. Unlike systems that consolidate to an external store (write tool calls in [[memgpt]], extracted facts in [[mem0]], or graph triples in [[hipporag-2]]), EM-LLM's *consolidate* and *retrieve* steps act on raw KV pairs already produced by the model. There is no encode-to-text step and no separate retriever model — the cache *is* the memory.

## Relation to other systems

- **Physical substrate — KV-cache vs. external store**: EM-LLM keeps memory as native KV tensors. [[memgpt]] writes summarised text to a paged context; [[mem0]] writes extracted facts to a structured store; [[hipporag-2]] builds an OpenIE [[knowledge-graph]] index. EM-LLM avoids the lossy text-roundtrip those approaches incur.
- **Physical substrate — KV-cache vs. weights**: [[titans]] also keeps memory inside the model, but in *parametric weights* updated online via gradients on a surprise signal. EM-LLM keeps the original KV activations and uses surprise only for *segmentation*, not for weight updates — the base LLM stays frozen.
- **Retrieval primitive — temporal contiguity vs. similarity-only**: standard RAG ranks purely by embedding similarity. EM-LLM's contiguity buffer adds a temporal axis, mirroring induction-head attention asymmetry and human free-recall dynamics — a retrieval move that flat-vector retrieval cannot express.

## References

- [[em-llm-fountas-2024]] — Fountas et al., *Human-inspired Episodic Memory for Infinite Context LLMs*, ICLR 2025 ([arXiv:2407.09450](https://arxiv.org/abs/2407.09450))
