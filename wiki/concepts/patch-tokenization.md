---
title: Patch Tokenization
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - vit-dosovitskiy-2020.md
status: draft
importance: medium
---

# Patch Tokenization

Patch tokenization is the technique of treating an image (or other non-text input) as a sequence of fixed-size, non-overlapping **patches**, each flattened and linearly projected to a vector embedding — analogous to wordpiece tokens in NLP. It is what allows the standard [[transformer-architecture|Transformer]] to ingest images without architectural modification.

Introduced (in its impactful form) by [[vit-dosovitskiy-2020|ViT]] in 2020 with 16×16 pixel patches. A 224×224 image becomes a sequence of 14×14=196 tokens.

## Tradeoffs

- **Smaller patches** → longer sequences → more compute (attention is O(N²)) but finer spatial resolution.
- **Larger patches** → shorter sequences → cheaper but coarser. In practice, ViT-Huge with patch size 14 produces 16×16=256 tokens for 224×224 images and is a good compute/accuracy trade-off.

## Extensions

- **Audio**: split spectrograms into time-frequency patches (Audio Spectrogram Transformer, etc.).
- **Video**: spatiotemporal patches (TimeSformer, ViViT).
- **Self-supervised**: mask random patches and predict them (MAE — masked autoencoders), analogous to [[masked-language-modeling|MLM]] for text.

## References

- [[vit-dosovitskiy-2020]]
