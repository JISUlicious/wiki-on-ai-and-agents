---
title: GLM-5.2
type: entity
created: 2026-06-18
updated: 2026-06-18
sources:
  - glm-5-2-zai-2026.md
family: glm
parameters: "744B MoE (40B active)"
release_date: 2026-06-13
status: draft
importance: medium
tags:
  - model
---

# GLM-5.2

**GLM-5.2** is a coding-first frontier LLM from [[z-ai|Z.ai]] (Zhipu AI), released 2026-06-13 with open weights under the MIT license. It targets **long-horizon, agentic coding** over a **1M-token context**. Surfaced via the alphaXiv weekly digest ([[glm-5-2-zai-2026]]).

## Architecture

- **744B-parameter [[mixture-of-experts|Mixture-of-Experts]]** with **40B active parameters** — the same base as GLM-5.1, with post-training retargeted at extended-context coding.
- **IndexShare** sparse attention — every group of four transformer layers shares a single lightweight indexer rather than each layer computing its own, making the 1M-token context tractable (vendor-stated ~2.9× lower per-token FLOPs). See [[long-context-llm]].
- Open weights on Hugging Face (`zai-org`); available via Z.ai and Zhipu's Coding Plan.

## Lineage

Part of the **GLM** family; successor to GLM-5.1 (which it shares a base architecture with — the delta is post-training for long-horizon coding plus the IndexShare inference path). No technical report or benchmark table was published at release.

> [!note] Draft
> Specs are vendor-stated (no official report/benchmarks). Enrich when Z.ai publishes a GLM-5.2 report or independent evals land.

## References

- [[glm-5-2-zai-2026]] — the announcement / alphaXiv digest entry.
- [[z-ai]] — the lab.
