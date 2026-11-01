---
title: "In-Parameter Memory: A Survey — Towards In-Parameter Memory Augmentation for Large Language Models"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - in-parameter-memory-survey-huang-2026.md
arxiv_id: "2610.08630"
authors:
  - Haoyu Huang
  - Zhongwei Xie
  - Jiaxin Bai
  - Yisen Gao
  - Hong Ting Tsang
  - Wuganjing Song
  - Huihao Jing
  - Yufei Li
  - Yangqiu Song
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# In-Parameter Memory: A Survey

**Haoyu Huang, Zhongwei Xie, Jiaxin Bai, Yisen Gao, Hong Ting Tsang, Wuganjing Song, Huihao Jing, Yufei Li, Yangqiu Song** (The Hong Kong University of Science and Technology; Hong Kong Baptist University) — [arXiv:2610.08630](https://arxiv.org/abs/2610.08630), v1, cs.CL, 6 Oct 2026. Survey, about 10 pages of main text.

## Summary

[[in-context-learning|ICL]] and [[retrieval-augmented-generation|RAG]] keep memory as text tokens: inspectable and easy to revise, but they consume the window and pay prefill cost on every use. **In-parameter memory (IPM)** instead represents reusable information as a parameter-like object ϕ (an adapter, soft prompt, encoded KV bank, memory table or online neural state) that is plugged into the forward pass at inference.

The survey formalizes IPM with three parts: a **memory object ϕ**, an **acquisition operator A** that produces it, and an **augmentation operator C** that composes it into the forward pass. A method counts as IPM only if ϕ is (1) parametric, (2) produced by an independent A rather than as a by-product of running the model, and (3) still a distinct object at deployment. By these tests:

- pluggable PEFT such as [[lora|LoRA]] adapters is IPM;
- full-parameter SFT is not (merged into θ before serving);
- an unmodified [[kv-cache]] is not (a by-product of prefill), but an encoder-produced KV bank is;
- [[continual-learning]] counts only when ϕ is still augmented at deployment.

## Key points

- **Two axes.** *Placement* (where C plugs in): embedding (soft prompts, latent tokens), attention (KV banks, attention-side adapters, online neural memory), FFN (adapters, memory tables), or hybrid. *Acquisition time*: offline (before serving) vs online (during serving).
- **Where things land.** [[titans|Titans]], Atlas, MesaNet, TTT layers and [[metis-memory-foundation-model-zhang-2026|Metis]] are attention-layer, online. KBLaM and AtlasKV are precomputed KV banks (offline). Doc-to-LoRA, Text2LoRA and SHINE generate adapters.
- **Cost pattern.** Online rows are per-request or per-token and ephemeral. Generated adapters pay a medium one-pass write; recurrent fast weights and neural memory pay a cheap per-token write. Offline rows pay a high one-time write and then read cheaply. The comparison uses qualitative bands, because papers rarely report comparable FLOPs.
- **Trade-offs.** Embedding-side objects are cheapest but act indirectly. Attention-side objects are content-addressable. FFN-side objects can overwrite specific behaviours but read position-wise and interfere under sequential writes. Hybrids are most expressive and hardest to attribute.
- **Text still wins** for exact quotation, provenance, targeted deletion and human revision.
- **Open directions.** Interference and forgetting under sequential writes; safety-aware acquisition (an untrusted write becomes trusted weights, and current systems keep no provenance); co-design with ICL; and **model-native recursive self-improvement**, where deployment experience is written into ϕ instead of into harness prompts, skills or knowledge bases.

The alphaXiv line ("towards in-parameter memory augmentation") is just the subtitle; nothing to verify.

## What it does not show

- No experiments. All comparisons are qualitative.
- Short for a survey, and the scope rule excludes some memory types readers might expect (plain KV caches, merged fine-tunes). A concurrent architecture-centric survey covers those.

## Concepts & entities

- [[test-time-memory]] · [[neural-long-term-memory]] · [[titans]] · [[linear-attention]] — online, attention-side IPM.
- [[lora]] · [[fine-tuning]] · [[continual-learning]] · [[catastrophic-remembering]] — adapters and the forgetting problem.
- [[retrieval-augmented-generation]] · [[in-context-learning]] · [[kv-cache]] — the token-memory side of the boundary.
- [[metis-memory-foundation-model-zhang-2026]] · [[memt-1-0-memory-native-evolution-li-2026]] — memory foundation models.
- [[self-improving-agent]] · [[long-term-memory]] · [[memory-consolidation]].

## References

- [arXiv:2610.08630](https://arxiv.org/abs/2610.08630)
