---
title: "Weight-Space Programming Doesn't Need Transformers"
type: source
created: 2026-11-19
updated: 2026-11-19
sources:
  - weight-space-programming-doesnt-need-transformers-ok-2026.md
authors:
  - Hyunjong Ok
  - Jaeho Lee
year: 2026
venue: "alphaXiv (2611.weight-space-programming-associative-memory); preprint"
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Weight-Space Programming Doesn't Need Transformers

**Hyunjong Ok and Jaeho Lee** (POSTECH). Preprint, Nov 2026. A follow-up to [[weight-space-programming-takehi-2026|Takehi et al.'s blog post]], which it cites as the origin of [[weight-space-programming]] (WSP).

> [!note] No arXiv ID
> This paper is **alphaXiv-only and has no arXiv ID**. It was read from the full PDF at https://www.alphaxiv.org/abs/2611.weight-space-programming-associative-memory.pdf, which downloaded normally. The alphaXiv page also shows arXiv 2611.10866 ([[the-surprising-effectiveness-of-deleting-weights-lu-2026]]), but only in its related-papers sidebar. The PDF does not cite it.

## Summary

Transformers were picked because they train well with gradients and run fast on GPUs. In WSP an LLM writes the weights by hand, so neither reason applies. The paper asks whether WSP should keep the Transformer.

Three writer LLMs ([[claude-opus-5-5|Claude Opus 5.5]], GPT-6.1 Sol, [[gpt-6-astra|GPT-6 Astra]]) each get **ten labeled ARC questions** and a sandbox. They either write weights into a fixed architecture (15 released architectures, including Transformers, [[state-space-model|SSMs]] and RNNs, from random init; or a given two-layer Transformer), or design the model themselves. Test set: 3,530 ARC-Easy + ARC-Challenge questions.

When free to design, **all three writers build the same thing**: plain-text facts compiled into a sparse table, read by a fixed lookup procedure. No attention, no MLPs, no dense embeddings. Claude Opus 5.5's first-generation model maps 361,150 words to 3,368 concepts and stores 4,288 fact rows (0.94M parameters). The authors then fix a faster forward pass themselves (an "associative-memory-based" model with hashed pair-table lookups) and let the LLM write only its tables.

## Key points

- **Designed beats fixed.** Claude Opus 5.5: released architectures 43.1%, given Transformer 36.1%, LLM-designed gen 1 50.8%, gen 5 **54.4%**. The best designed model beats the most accurate released Transformer the same LLM wrote with about **1,800× fewer parameters** (abstract).
- **Beats gradient descent without pretraining.** Trained on all 3,358 labeled ARC questions, Mamba-2 130M reaches 24.6% and a same-sized two-layer Transformer 29.8%.
- **Pretrained models still win by a lot.** Fine-tuned Qwen3.5-0.8B: 82.1%. ARC-Challenge stays near 38% for every written model.
- **Why tables.** Answers trace to one row: keeping only the best fact per option gives 49.1% vs 50.8% for the full model. Edits stay local: removing one line changes 0.5–1.8 test answers in table models, versus 4.8 (and in one run 38% of answers) in written Transformers.
- **Structure, not just content.** All six written two-layer Transformers by Claude Opus 5.5 score below [[bm25|BM25]] over their own written text (−8.0 points on average). Table models score above it (+1.5 to +5.7). The authors read this as interference between dense keys.
- **Speed.** LLM-designed models score every fact per question; the median of 46 variants is 9.4× slower than a same-sized Transformer on GPU. The associative-memory model reads a fixed 120 pair-table rows per question: 18 ms for all 3,530 questions on one RTX 3090, **7–17× faster** than a same-sized Transformer, within about 3 points of the designed models' accuracy.
- **Other tasks.** Same table structure on SQuAD (F1 42.3 vs 33.2 for the written Transformer) and SciQ. Near chance on MMLU. CIFAR-10 raw pixels: 21–28%. With DINOv2 tokens a written table reaches 96.1%, which the authors credit to the encoder.
- **Pretrained checkpoints.** In 48 of 87 runs the LLM changes under 1% of weights; checkpoints above 72% are never overwritten.

## Relation to the Takehi blog post

- Takehi et al. had one LLM (GPT-6 Astra) read and write weights of **fixed** networks: PPO policies, a Go net, Qwen3.5-0.8B. This paper lets the writer **choose the architecture**, uses three LLMs, and adds SSMs and RNNs among the fixed targets. It confirms the core claim that LLMs can write useful weights without gradients.
- It adds a constraint: given the choice, writers do not build Transformers. Dense weights are hard to write and debug by hand.
- It re-evaluates the **edited Qwen3.5-0.8B released by Takehi et al.**: within half a point of the unedited instruct base on ARC, MMLU and MathBench, slightly below on SQuAD (73.9 vs 73.5 ARC; 47.0 vs 46.7 MMLU). This does not contradict the blog's MATH-500/GSM8K gains, which came from stopping thinking loops on different tasks (inference).

## What it does not show

- Written models only **recall facts the writer states**. None chains facts across steps or learns representations of its own.
- The authors cannot separate "tables suit writing" from "LLMs know retrieval is the classic ARC baseline".
- Speed was measured on one GPU type. The speed advantage holds only up to about 15,000–32,000 concepts (dense concept table) or about 100,000 (row lists).
- Mainly one benchmark (ARC), ten labeled examples, and lineage runs that cost up to $490.

## Concepts & entities

- [[weight-space-programming]] · [[weight-space-programming-takehi-2026]]
- [[claude-opus-5-5]] · [[gpt-6-astra]] · [[qwen]] · [[mamba]] · [[dinov2]]
- [[transformer-architecture]] · [[state-space-model]] · [[bm25]] · [[squad]] · [[mmlu]]
- [[mechanistic-interpretability]] — every answer traces to a written row, so the models are interpretable by construction.

## References

- alphaXiv: https://www.alphaxiv.org/abs/2611.weight-space-programming-associative-memory.pdf (no arXiv ID)
- Takehi et al. 2026 (Sparks of WSP). Associative memory view of Transformers: Zhong et al. 2025; Wang 2026.
- Model editing: Meng et al. 2022, 2023. Tracr: Lindner et al. 2023. Fast weight programmers: Schmidhuber 1992; Schlag et al. 2021.
