---
title: Multimodal LLM
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-4-openai-2023.md
  - vit-dosovitskiy-2020.md
status: draft
importance: high
---

# Multimodal LLM

A multimodal LLM accepts input — and increasingly produces output — in multiple modalities: text, images, audio, video, and sometimes more (structured data, embeddings of arbitrary signals). The dominant architectural pattern combines a [[transformer-architecture|Transformer]] LM (text) with one or more modality-specific encoders that map non-text input into the model's token-embedding space.

## Image-input architecture (the common case)

1. An image encoder — typically a [[vit|ViT]]-style patch encoder, sometimes contrastively trained à la CLIP — produces a sequence of visual tokens.
2. A projection layer maps visual tokens into the LLM's embedding dimension.
3. The LLM consumes the combined sequence [text tokens, image tokens] and produces a text response.

This is the basic recipe behind LLaVA, MiniGPT-4, BLIP-2, and many open-source variants. Closed-source production systems ([[gpt-4|GPT-4V]], Claude 3+, Gemini) likely use similar shapes with proprietary refinements.

## Beyond images

- **Audio in/out**: GPT-4o (May 2024), Gemini, Claude (limited).
- **Video**: Gemini 1.5 Pro, GPT-4o, sometimes via per-frame patch encoding.
- **Native multimodal training**: Gemini was pre-trained on mixed modalities from the start rather than post-hoc attached. The trend is toward earlier and earlier fusion.

## Significance

- Removes the OCR / caption / transcript bottleneck for downstream tasks.
- Enables fundamentally new use cases: visual reasoning, document understanding, screen / computer use agents, accessibility.
- Validates the [[transformer-architecture|Transformer]] as a **general** sequence architecture, not text-specific — see [[vit-dosovitskiy-2020|ViT]] for the first major demonstration.

## References

- [[gpt-4-openai-2023]]
- [[vit-dosovitskiy-2020]]
