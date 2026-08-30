---
title: Qwen
type: entity
created: 2026-08-30
updated: 2026-08-30
sources:
  - qwen3.8-flash-next-model-card-2026.md
family: qwen
status: draft
importance: high
tags:
  - model
---

# Qwen

**Qwen** (通义千问) is [[alibaba|Alibaba]]'s family of open-weight language models, and — alongside [[llama|LLaMA]] — one of the two default substrates for open research in this wiki's corpus. Where LLaMA dominated the 2023–2024 literature, Qwen checkpoints (Qwen2.5, Qwen3, Qwen3.5) are the base model of choice across most 2026 sources here: agentic RL work, memory systems, harness studies, and distillation experiments overwhelmingly report Qwen backbones.

Its significance is availability at many sizes. The family ships dense models from ~0.5B to ~70B and MoE variants above that, all under permissive-ish community licences, which makes it the practical choice for ablations that need several capability tiers of the *same* model — the design that [[sample-more-reflect-less-mirzaei-2026|scale-sweep studies]] depend on.

## Lines in this wiki

- **Qwen2.5 / Qwen3** — the workhorse research backbones; appear across the corpus as fine-tuning and RL substrates.
- **[[qwen3.8-flash-next]]** — 2026 open-weight multimodal MoE on the new `qwen4_exp` architecture: hybrid Gated DeltaNet + Qwen Sparse Attention, 512-expert MoE, n-gram embeddings, 256K native context extensible to 1M via [[rope-scaling|YaRN]].
- **Qwen3.8-Flash** — the hosted Qwen Cloud productization of the above, with 1M context enabled by default.
- **Qwen3.8-27B** — the dense sibling used as the in-family comparator.

> [!note] Draft
> Created to close a long-standing gap: Qwen is referenced by over a hundred pages in this wiki with no page of its own. Earlier model generations, the Qwen2.5-VL / Qwen-CUA multimodal line, and release chronology are not yet recorded.

## Related

- [[alibaba]] · [[qwen3.8-flash-next]]
- [[llama]] · [[deepseek]] · [[moonshot-ai]] — the other open-weight families this corpus builds on.
- [[linear-attention]] · [[mixture-of-experts]] · [[rope-scaling]]

## References

- [[qwen3.8-flash-next-model-card-2026]]
