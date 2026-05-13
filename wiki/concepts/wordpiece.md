---
title: WordPiece
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - bert-devlin-2018.md
status: draft
importance: medium
---

# WordPiece

WordPiece (Schuster & Nakajima 2012; Wu et al. 2016) is a subword tokenization algorithm. Like [[byte-pair-encoding|BPE]], it builds a subword vocabulary by iteratively merging frequent units, but where BPE picks merges by frequency, WordPiece picks merges that **maximize the likelihood of the training data** under a unigram language model.

In practice WordPiece and BPE produce similar tokenizations and similar downstream results. WordPiece is used by [[bert]] (30k vocabulary) and by some Google translation systems. The GPT family uses byte-level BPE.

WordPiece typically marks non-initial subwords with a prefix (`##`) — e.g., the word "playing" might tokenize as `play ##ing`.

## References

- [[bert-devlin-2018]]
