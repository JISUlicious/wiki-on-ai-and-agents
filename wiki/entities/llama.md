---
title: LLaMA
type: entity
created: 2026-07-07
updated: 2026-11-02
sources:
  - llama-touvron-2023.pdf
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

The first release ([[llama-touvron-2023|Touvron et al., 2023]]) ranged from 7B to 65B and trained on about 1.4T tokens of public data. LLaMA-13B beat [[gpt-3|GPT-3]] on most benchmarks at a tenth of the size, and the models went to the research community, rather than being openly licensed from the start.

The family has iterated through several generations (LLaMA → Llama 2 → Llama 3 and successors), broadly scaling data and context length, adding instruct/chat-tuned variants alongside base checkpoints, and shifting licence terms toward permissive-but-conditional commercial use. Individual generation pages (`llama-2`, `llama-3`) can be split out from here as sources warrant.

## Related

- [[meta]] — the steward organization.
- [[decoder-only-transformer]] · [[transformer-architecture]] — the architecture.
- [[fine-tuning]] · [[lora]] — what open weights enable.

## References

- [[llama-touvron-2023]]
