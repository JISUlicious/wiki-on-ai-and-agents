---
title: "Human-inspired Episodic Memory for Infinite Context LLMs"
type: source
created: 2026-05-16
updated: 2026-05-16
sources:
  - em-llm-fountas-2024.md
arxiv_id: "2407.09450"
venue: ICLR 2025
authors:
  - Zafeirios Fountas
  - Martin A Benfeghoul
  - Adnan Oomerjee
  - Fenia Christopoulou
  - Gerasimos Lampouras
  - Haitham Bou-Ammar
  - Jun Wang
first_author: Zafeirios Fountas
year: 2024
introduces:
  - "[[em-llm]]"
  - "[[episodic-memory-llm]]"
  - "[[bayesian-surprise-segmentation]]"
tags:
  - 2024
status: complete
importance: high
---

# Human-inspired Episodic Memory for Infinite Context LLMs

**Source**: `em-llm-fountas-2024.md`
**arXiv**: [arXiv:2407.09450](https://arxiv.org/abs/2407.09450)
**Venue**: ICLR 2025
**Authors**: Zafeirios Fountas, Martin A Benfeghoul, Adnan Oomerjee, Fenia Christopoulou, Gerasimos Lampouras, Haitham Bou-Ammar, Jun Wang (Huawei Noah's Ark Lab; UCL AI Centre)

## Summary

EM-LLM imports the algorithmic interpretation of **human episodic memory** into pre-trained LLMs **without any fine-tuning**, letting them handle effectively infinite contexts while staying computationally tractable. As tokens stream in, EM-LLM performs **online Bayesian-surprise segmentation**: a token is a candidate event boundary when its negative log-likelihood under the model exceeds a moving-window mean-plus-γσ threshold. These initial boundaries are then **refined with graph-theoretic clustering metrics** (modularity or conductance) applied to the attention-key similarity matrix, so each event maximises intra-event key cohesion and inter-event separation.

Critically, the episodic memory is **internal to the KV-cache itself**, not an external vector store. Past tokens that fall outside the local context window are organised into surprise-delimited event units of contiguous KV pairs; initial-token attention sinks and a recent-token local-context window are preserved as in InfLLM. At inference time a **two-stage retrieval** pulls events back into the active context for each layer independently: (1) a **similarity buffer** populated by k-NN over per-event representative tokens against the current query, and (2) a **contiguity buffer** — a queue that automatically enqueues neighbouring events of any retrieved event, reproducing the temporal contiguity and asymmetry effects observed in human free recall and recently identified in LLM induction heads (Ji-An et al., 2024).

On LongBench and ∞-Bench, EM-LLM outperforms the prior SOTA group-based retrieval method **InfLLM** across 5 base LLMs (Mistral-7B, LLaMA-3, LLaMA-3.1, Phi-3, Phi-3.5), exceeds RAG with NV-Embed-v2 by 30.5% on LongBench and 11.5% on ∞-Bench, and even beats full-context baselines on most tasks. It achieves 100% passkey retrieval at **10.2M tokens** — a length infeasible for full-context models. Separately, the authors show that EM-LLM's surprise-based event boundaries correlate with human-annotated event boundaries on podcast transcripts, suggesting LLM next-token surprise is a viable proxy for the cognitive signal driving human event segmentation.

## Key Points

- **Bayesian surprise as event boundary**: token $x_t$ is a candidate boundary iff $-\log P(x_t \mid x_{<t};\theta) > \mu_{t-\tau:t} + \gamma\sigma_{t-\tau:t}$ — adaptive threshold over a sliding window, no extra forward passes needed.
- **Graph-theoretic boundary refinement**: treat per-head key-similarity as a weighted adjacency matrix, then shift each boundary to maximise modularity (best) or minimise conductance. Complexity $O(nm)$ with $m$ a small chunk size.
- **KV-cache-internal episodic memory**: events are *contiguous spans of past KV pairs*; episodic structure becomes a property of the context itself rather than an external store like RAG or kNN-LM.
- **Two-stage retrieval**: similarity buffer (k-NN over event representative tokens) + contiguity buffer (queue auto-enqueueing temporally adjacent events of retrieved hits) — directly engineered to mirror human free-recall dynamics and the contiguity/asymmetry attention patterns of induction heads.
- **Per-layer retrieval**: each transformer layer retrieves its own k events independently, in contrast to RAG's single up-front retrieval — credited with much of the gain over RAG.
- **Three-zone context layout**: initial-token sinks (128) + episodic-memory retrieval buffers + local context window with full softmax attention. Retrieved tokens get a fixed position embedding (à la T5/InfLLM) so they sit outside the pre-trained positional range.
- **Empirical**: beats InfLLM, RAG (NV-Embed-v2), and full-context baselines on LongBench and ∞-Bench; scales to 10M-token passkey retrieval; no fine-tuning of the base LLM.
- **Cognitive-science alignment**: surprise-based segments correlate with human-annotated event boundaries on podcast transcripts, with surprise + modularity refinement matching humans best (Wasserstein distance to human reports).

## Entities Mentioned

- Huawei Noah's Ark Lab (London)
- University College London — AI Centre

## Concepts Discussed

- [[em-llm]]
- [[episodic-memory-llm]]
- [[bayesian-surprise-segmentation]]
- [[memory-management]]
- [[long-context-llm]]
- [[kv-cache]]

## Notable Quotes

> "EM-LLM organises sequences of tokens into coherent episodic events using a combination of Bayesian surprise and graph-theoretic boundary refinement in an online fashion. When needed, these events are retrieved through a two-stage memory process, combining similarity-based and temporally contiguous retrieval for efficient, human-inspired access to relevant information."

> "EM-LLM's performance even surpasses full-context models in most tasks, while successfully performing retrieval across 10 million tokens — a scale computationally infeasible for such models."

## References

_Original source: `sources/em-llm-fountas-2024.md`_
