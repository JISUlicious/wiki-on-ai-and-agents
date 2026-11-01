---
title: "MemT-1.0: Towards Memory-Native Evolution"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - memt-1-0-memory-native-evolution-li-2026.md
arxiv_id: "2610.26825"
authors:
  - MemTensor Team
  - Zhiyu Li
  - Zeyu Zhang
  - Bo Tang
  - Feiyu Xiong
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# MemT-1.0: Towards Memory-Native Evolution

**MemTensor Team** (core contributors led by Zhiyu Li and Zeyu Zhang; about 60 core contributors and 12 co-advisors listed in Appendix A) — [arXiv:2610.26825](https://arxiv.org/abs/2610.26825), v2, cs.CL, 20 Oct 2026. Code `MemTensor/MemT-1.0`; models on Hugging Face.

## Summary

The direct successor to [[metis-memory-foundation-model-zhang-2026|Metis]], the same team's "memory foundation model". The thesis is unchanged: memory should be an intrinsic part of model computation rather than an external store. MemT-1.0 keeps Metis's split between **dynamic local memory** (per-trajectory state written during interaction) and **trainable hyper memory** (learned parameters for writing and reading it). It changes two things:

- **Token-level updates.** Metis's token selection is replaced by a stable token-level update based on **Gated DeltaNet-2** (a successor to [[gated-deltanet]]) within each turn. Metis's query-key normalization is removed.
- **Mixture-of-Storage (MoS).** Each native memory block is expanded into E parallel memory experts with routers, so memory capacity scales like [[mixture-of-experts]] scales FFNs.

The backbone stays frozen and also serves as the full-context distillation teacher. Training is continued pre-training then post-training on 34B tokens.

## Key points

- **Two models.** MemT-1.0-Nano: 36B total / 3B active / 10M dynamic memory, built on Qwen3.6-35B-A3B. MemT-1.0-Flash: 288B / 17B active / 336M memory, built on a DeepSeek-V4-Flash the team post-trained first.
- **Memory benchmarks (11).** Beats parametric-memory baselines on average and stays "competitive with" retrieval/textual-memory methods (MemOS, DenseRAG, Graphiti) — i.e. it does not beat them across the board.
- **Efficiency vs full-context KV (Nano).** Slower at short histories, about even at 32K, then 1.20× at 64K, **1.86× at 128K**, 3.99× at 256K end-to-end. Persistent storage is constant: 40 MiB vs 5.061 GiB of full-context KV at 256K, about 66× smaller at 128K.
- **Agentic use.** Procedural experience is written into native memory instead of the prompt. On [[swe-bench-verified|SWE-bench Verified]] (Before Execution), average success 66.07% → 69.33% and pass^3 54.00% → 61.60%, about the same as putting the experience in as text (68.87%, 61.60%). SWE-bench Multilingual: 60.00% → 62.89%.

The alphaXiv line ("extends Metis with token-level persistent state") is accurate; "persistent" refers to a fixed-size per-trajectory state, not to weight updates.

## What it does not show

- **Fixed capacity.** The state is O(1) in history length. The authors acknowledge it trades old detail for new and loses fidelity for precise recall.
- The speedup mostly comes from avoiding history prefill. At short histories it is slower than full context.
- The agentic gain matches textual experience rather than beating it; the claim is equivalence without the prompt tokens.
- Whether local memory can be consolidated into hyper-memory weights (true continual learning) is left open.

## Concepts & entities

- [[metis-memory-foundation-model-zhang-2026]] — predecessor.
- [[in-parameter-memory-survey-huang-2026]] — its taxonomy files Metis under attention-layer placement, online acquisition (alongside Titans and Atlas).
- [[test-time-memory]] · [[neural-long-term-memory]] · [[titans]] · [[linear-attention]] · [[gated-deltanet]] — learned recurrent memory states.
- [[mixture-of-experts]] · [[knowledge-distillation]] · [[procedural-memory]] · [[continual-learning]] · [[long-term-memory]].
- [[qwen]] · [[deepseek]] · [[locomo]].

## References

- [arXiv:2610.26825](https://arxiv.org/abs/2610.26825)
