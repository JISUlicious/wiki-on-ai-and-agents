---
title: Ollama
type: entity
created: 2026-10-05
updated: 2026-10-05
sources:
  - edgextract-markdown-to-knowledge-graph-mansuy-2026.md
builds-on:
  - [[llama-cpp]]
  - [[gguf]]
status: draft
importance: medium
tags:
  - product
---

# Ollama

A local model runner: it pulls, serves and runs open-weight models on a laptop or server, with an HTTP API on `localhost:11434`. It runs on the [[llama-cpp|llama.cpp]] engine and the [[gguf|GGUF]] model format. Along with LM Studio, it is one of the main front ends to the local-model ecosystem, including on [[apple-silicon|Apple Silicon]] through [[metal|Metal]].

## Decision models (v0.35.0, 2026-09-28)

Ollama v0.35.0 added **decision models** through a native `POST /v1/systemone` endpoint, "based on TypeSafe's Jev API". These models return choices, probabilities and scores instead of text, for tasks like ticket triage, model routing and classification. The models available at launch:
- [[nimble|Nimble]] (Bespoke Labs)
- Tev1 (Together AI)
- Clef and Clef Flash, which can also judge images (v0.35.1+)

Local requests need no API key.

This makes Ollama one of the first general-purpose serving engines with a built-in [[system-one-model|System One]] route, after [[sglang|SGLang]] (2026-09-25). The [[system-one-ecosystem-sweep-2026-09-28|ecosystem sweep]] recorded "nothing found" for Ollama, on the same day the release shipped. [[edgextract-markdown-to-knowledge-graph-mansuy-2026|edgextract]] builds on this route to extract knowledge graphs with closed yes/no decisions.

## How it figures elsewhere in this wiki

- **As a backend.** [[a-mem-xu-2025|A-Mem]] serves Qwen and Llama through Ollama; [[selective-forgetting-graph-based-memory-rusu-2026|Rusu et al.]] use Ollama-served embeddings; [[hermes-agent]] lists it among its providers.
- **As a baseline.** [[neuroprefetcher-delta-prefetching-dhar-2026|NeuroPrefetcher]] measures Ollama at 0.07 tok/s when model weights must stream from SSD, against 3.22 tok/s for its own system. See [[weight-streaming]].
- **As an attack surface.** Quantization-conditioned backdoors were extended to GGUF, "the format llama.cpp and ollama actually run" ([[mind-the-gap-egashira-2025]], [[quantization-conditioned-backdoor]]). A model that behaves well in full precision can misbehave once quantized for local use.

## Related

- [[llama-cpp]] · [[gguf]] · [[mlx]] · [[apple-silicon]] · [[edge-inference]] · [[system-one-model]] · [[nimble]] · [[sglang]]

## References

- [Ollama v0.35.0 release notes](https://github.com/ollama/ollama/releases/tag/v0.35.0) · [Decision capability docs](https://github.com/ollama/ollama/blob/main/docs/capabilities/decision.mdx)
- [[edgextract-markdown-to-knowledge-graph-mansuy-2026]]
