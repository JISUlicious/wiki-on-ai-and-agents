---
title: "Residual Vector-based Reconstruction as Long-Context Recall Regardless of Context Window Size"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - residual-vector-reconstruction-long-context-recall-ryu-2026.md
arxiv_id: "2609.12686"
authors:
  - MyungHoon Ryu
  - XinYu Piao
  - Jong-Kook Kim
year: 2026
venue: preprint (cs.AI)
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Residual Vector-based Reconstruction as Long-Context Recall

[arXiv:2609.12686](https://arxiv.org/abs/2609.12686) — MyungHoon Ryu, XinYu Piao and Jong-Kook Kim, Korea University, 11 September 2026. A preprint with no stated venue.

## What it says

A model with a bounded context window is made to answer questions about documents far longer than that window, at nearly constant GPU memory and without training. The document is read once in bounded windows. Individual factual sentences are extracted **before any question is known**, and each is encoded as a small vector outside the frozen model. The document and its [[kv-cache|KV cache]] are then discarded. At question time an external index picks the relevant facts, each fact's vector is injected into one feed-forward layer to make the model regenerate the stored sentence verbatim, and a normal generation pass answers from the regenerated sentences.

The paper's framing: every training-free alternative loses detail it cannot know it will need. Truncation, streaming, cache eviction and [[prompt-compression|prompt compression]] discard or condense earlier text, and [[retrieval-augmented-generation|retrieval]] can fail to put the needed passage into the answer context. This method keeps sentence-level facts instead, at a storage cost set by a fixed fact budget rather than by document length.

## How it works

**Memorization (once per document).**

1. A one-pass filter keeps candidate sentences containing entities, numerals or identifier-like tokens. The frozen model proposes a short verbatim **anchor** for each, such as a character's name.
2. A budget `B = min(m̂ · n_anchors, K · C)` caps how many facts are kept. When there are more anchors than budget, each anchor keeps only its **latest** fact, which suits the state-tracking benchmark used.
3. Facts are spread round-robin over `K` feed-forward layers (12 by default), chosen by a held-out probe of which layers reconstruct best.
4. For each fact, a fixed prompt — `Recall the fact about {key}:` — produces an activation at the input of that layer's down-projection. That activation is the fact's **address**. A residual vector `δ` is then optimized, with the model frozen, until adding it at that layer makes the model reproduce the sentence: 40 teacher-forced steps, then 20 free-running refinement steps if needed.
5. The normalized address and the residual are stored outside the model, alongside one routing document per anchor containing its facts and generated paraphrase questions.

**Response (per question).**

1. The question is matched against the routing documents with dense embeddings (BGE-M3) and inverse-document-frequency lexical ranking, fused by reciprocal rank. At most five anchors and 24 facts pass.
2. For each selected fact, its recall prompt is replayed. A hook at the assigned layer compares the live activation with every stored address at that layer; if the best cosine similarity exceeds 0.6, it adds a sharply softmax-weighted (β = 50, effectively top-1) mix of the stored residuals. Greedy decoding regenerates the stored sentence, the same way every time.
3. The regenerated sentences go into the answer prompt in source order, and the frozen model answers.

The idea builds on the finding that transformer feed-forward layers behave like key–value memories. Unlike model editing (ROME, MEMIT), nothing is written into the weights: removing the external store restores the original model exactly.

## Results

Three frozen models: Phi-3.5-mini (3.8B, 128K window), Qwen3-4B (40,960-token window) and Llama-3.1-8B (128K), on one 80 GB A100.

| Measurement | Result |
|---|---|
| BABILong single-fact recall at 2M tokens | 50% Phi-3.5-mini, 67% Qwen3-4B, 80% Llama-3.1-8B, from **4, 3 and 5 questions**; every compared training-free method at or near zero |
| BABILong two-fact composition | nonzero on Phi-3.5-mini to 1M and Qwen3-4B to 512K; Llama-3.1-8B falls to zero after 128K |
| RULER QA, encyclopedic prose | leads on Phi-3.5-mini from 512K and on Llama-3.1-8B from 128K to 1M; ties retrieval on Qwen3-4B at 2M; Llama-3.1-8B fails at 2M |
| Response-time GPU memory, 4K → 2M tokens | 9.3–10.3 GB for the small models, 17.1–17.9 GB for Llama-3.1-8B; the fact store grows by under 0.8 GB |
| Evidence position | 100% at all five tested depths and three lengths, including 2M (Phi-3.5-mini) |
| Deletion | removing 8 of 64 stored facts drops their recall to zero and leaves all 56 other outputs unchanged, on all three models |

On the BABILong task grid, single-fact recall (QA1) reaches 100% at 128K and 256K, while the three-fact and argument-relation tasks (QA3, QA4) stay at or near zero from 16K.

## What it does not show

- **The long-context scores rest on very few questions.** The 1M and 2M cells hold one to five questions each; the authors call them "feasibility rather than precise accuracy estimates." Phi-3.5-mini's 100% on RULER at 1M and 2M is one correct question each.
- **Answer accuracy is partly pretraining.** Closed-book scores are 13–38% on the RULER prose, and Qwen3-4B's 33% at 1M is below its own 38% closed-book score.
- **Routing is the weak point.** On a probe built so questions share few words with the stored facts, the method answers 1 of 8 where chunk retrieval answers 7 of 8, and it fails all four one-hop questions. A fact can be stored perfectly and still be unreachable from a differently worded question.
- **Composition mostly fails.** Tasks needing three facts or relations without explicit names score near zero; extraction misses evidence that lacks entity or number cues.
- **The baselines are weak at long length by design.** Truncation, the streaming proxy (four sink tokens plus the latest 4,092, not the official StreamingLLM) and compression all run on a 4,096-token budget, and the full-context reference stops at 40,000 tokens. Chunk retrieval, keeping the top three of twenty 384-token chunks, is the only competitor that can reach far into a long document.
- **Memorization is expensive.** Every kept fact needs its own gradient-optimization loop, far more model passes than embedding a document for retrieval, and every recalled fact costs its own decoding pass before the answer.
- **The most informative baseline is missing** *(this wiki's inference, not the paper's)*. Stage 1 already extracts each fact as a verbatim sentence, and routing already uses an external text index. Storing those same sentences as plain text behind the same index would also be deterministic and deletable, and far smaller: each stored entry here is `d_ff + d_model` scalars, 18,432 for Llama-3.1-8B, against a sentence of a few hundred bytes. Without that comparison, the gain over chunk retrieval could come from query-agnostic fact extraction and anchor routing rather than from the residual vectors themselves.

The paper's generative-AI disclosure states that Claude assisted in implementing the code and that Claude and OpenAI Codex assisted in revising the manuscript.

## Related

- [[long-context-llm]] · [[kv-cache]] · [[rope-scaling]] — the problem, and the training-based way around it.
- [[retrieval-augmented-generation]] · [[prompt-compression]] — the training-free alternatives it compares against.
- [[memory-management]] · [[memgpt]] · [[virtual-context]] — agent memory, where the authors aim this.
- [[residual-stream]] · [[mechanistic-interpretability]] · [[activation-addition]] — injecting vectors into the forward pass, used here for storage rather than steering.
- [[catastrophic-remembering]] — a store where deleting one entry is local and verified, the property that page finds missing from appending memory.

## References

- [arXiv:2609.12686](https://arxiv.org/abs/2609.12686)
