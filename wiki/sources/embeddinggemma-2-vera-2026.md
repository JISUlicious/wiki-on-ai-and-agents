---
title: "EmbeddingGemma 2: Unified and Efficient Multimodal Representations"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - embeddinggemma-2-vera-2026.md
arxiv_id: "2610.37364"
venue: arXiv preprint (cs.LG)
authors:
  - Henrique Schechter Vera
  - Sahil Dua
  - EmbeddingGemma Team (Google; 80 authors total)
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# EmbeddingGemma 2: Unified and Efficient Multimodal Representations

**Source:** Henrique Schechter Vera, Sahil Dua (co-first) and the EmbeddingGemma Team, [[google|Google]] / [[deepmind|Google DeepMind]] — [arXiv:2610.37364](https://arxiv.org/abs/2610.37364), cs.LG, v1 27 Oct 2026, v2 9 Nov 2026. It is a technical report for an open-weight release.

## Summary

EmbeddingGemma 2 is an open **744M-parameter** embedding model. It maps text, images, audio, video and any interleaved mix of them into one 768-dim space. Earlier multimodal embedders are either two-tower models ([[clip|CLIP]]-style, which cannot handle mixed input) or multimodal LLMs adapted for embeddings (several billion parameters). This model is small enough for on-device use.

It has three modular parts:

- a 271M bidirectional [[gemma|Gemma 4]]-derived backbone, with 24 layers, d=512 and mean pooling;
- a 167M [[vit|ViT]] vision encoder;
- a 305M USM-style conformer audio encoder.

Because the parts are separate, it can run text-only (271M), as vision+text (439M) or as audio+text (577M). It is distilled from Gemini Embedding 2.

## Key points

- **Simplified per-layer embeddings (PLE).** It keeps Gemma 3n's per-layer residual inputs but drops the separate PLE lookup table, projecting from the main input embeddings instead. That is cheaper *and* scores better in ablation.
- **Three-stage curriculum.** The order is text pre-finetuning (mostly distillation), then multimodal distillation, then multimodal *contrastive* finetuning, with a spread-out regulariser and Matryoshka losses. Swapping the last stage for continued distillation does worse, because the teacher's space is not fitted to the student.
- **Deployment knobs.** Matryoshka truncation to 512, 256 or 128 dims. A vision-token budget of 70–1120 per image, which matters mostly for document retrieval. QAT int4 checkpoints, made in a separate post-merge tail stage distilled from the float model, lose little on MTEB.
- **Results.** It sets a new Pareto frontier across MTEB (Multilingual, English, Code), MMEB-v2, MIEB(lite), MAEB and MSEB. It beats specialists and models 2–12× larger.
  - MTEB(Code) jumps from 68.8 to 78.7 over EmbeddingGemma 1 (308M).
  - Multilingual text is roughly flat, and English regresses slightly.
- **Native audio beats ASR.** MSEB retrieval scores 69.5, against 51.0 for transcribing the audio and embedding it with the text-only predecessor.

## Lineage

It inherits the backbone and encoders from Gemma 4 and PLE from Gemma 3n, and learns from Gemini Embedding 2 as teacher. It succeeds EmbeddingGemma 1, which was text-only with 308M parameters. It gives up some English text quality to add every other modality.

## Concepts & entities

- [[word-embeddings]] · [[dense-retrieval]] · [[dual-encoder]] · [[vector-database]] · [[retrieval-augmented-generation]] — the embedding use cases.
- [[knowledge-distillation]] · [[quantization]] · [[edge-inference]] — the training and deployment recipe.
- [[multimodal-llm]] · [[vit]] · [[clip]] — the multimodal-encoder background.
- [[gemma]] · [[google]] · [[deepmind]].

## References

- [arXiv:2610.37364](https://arxiv.org/abs/2610.37364)
- Model docs: https://ai.google.dev/gemma/docs/embeddinggemma
