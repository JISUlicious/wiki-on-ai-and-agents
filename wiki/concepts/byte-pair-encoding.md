---
title: Byte-Pair Encoding
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - transformer-vaswani-2017.md
status: draft
importance: medium
---

# Byte-Pair Encoding

Byte-pair encoding (BPE; Sennrich et al. 2016, adapted from a 1994 compression algorithm by Gage) is a subword tokenization scheme. It starts with a character-level vocabulary and iteratively merges the most frequent adjacent symbol pair, building up a vocabulary of subword units. This balances vocabulary size (smaller than word-level) with sequence length (shorter than character-level) and gracefully handles out-of-vocabulary words by decomposing them into subwords.

The [[attention-is-all-you-need-vaswani-2017|original Transformer]] used BPE with ~37k merges for WMT EN-DE and 32k word-piece tokens for EN-FR. The GPT family ([[gpt-1]] onward) uses byte-level BPE; [[bert]] uses a closely-related variant called WordPiece.

## References

- [[attention-is-all-you-need-vaswani-2017]]
