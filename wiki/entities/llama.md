---
title: LLaMA
type: entity
created: 2026-07-07
updated: 2026-07-07
sources: []
family: llama
release_date: 2023-02-24
status: draft
importance: high
tags:
  - model
---

# LLaMA

**LLaMA** (Large Language Model Meta AI) is [[meta|Meta]]'s family of **open-weights** [[decoder-only-transformer|decoder-only transformer]] language models — the single most important driver of the open-source LLM ecosystem, and the default base model across much of the research corpus in this wiki.

## Why it matters here

LLaMA's significance is less about any one checkpoint than about **availability**. Because the weights are open, LLaMA variants became the standard substrate for academic work that needs to fine-tune, ablate, or inspect a capable model. Across this wiki's sources it recurs as:

- the **base model for fine-tuning experiments** — e.g. [[probe-process-benchmark-hallucination-detection-2026|PROBE]], where an SFT'd Llama-3.1-8B **beats frontier models** at evidence finding
- the **open baseline** in agentic and memory work ([[codeact-wang-2024|CodeAct]], [[memfactory-guo-2026|MemFactory]], [[single-agent-vs-multi-agent-tran-2026]])
- the **serving target** for inference infrastructure ([[vllm-tool-calling-2025|vLLM]])
- the **student/teacher** in distillation studies ([[embarrassingly-simple-self-distillation-zhang-2026]])

This "open-weights base model" role is what makes the [[harness-vs-model-attribution|harness-vs-model]] question tractable at all — you can hold the model fixed and vary the scaffold only when you control the weights.

## Lineage

The family has iterated through several generations (LLaMA → Llama 2 → Llama 3 and successors), broadly scaling data and context length, adding instruct/chat-tuned variants alongside base checkpoints, and shifting licence terms toward permissive-but-conditional commercial use. Individual generation pages (`llama-2`, `llama-3`) can be split out from here as sources warrant.

> [!note] Draft
> Created during a lint pass as a gap-fill: `[[llama]]` was the most-referenced missing page in the wiki (4 inbound links across [[meta]] and three source pages) despite 279 ingested sources. Deserves proper treatment — per-generation parameter counts, training-token budgets, licence history, and the Llama-2 / Llama-3 technical reports as first-class sources.

## Related

- [[meta]] — the steward organization.
- [[decoder-only-transformer]] · [[transformer-architecture]] — the architecture.
- [[fine-tuning]] · [[lora]] — what open weights enable.

## References

- Referenced across the corpus; no dedicated LLaMA technical report ingested yet (see draft note).
