---
title: Distributional Semantics
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - mikolov-word2vec-2013.md
status: draft
importance: medium
---

# Distributional Semantics

The hypothesis that the **meaning of a word is captured by the distribution of contexts in which it appears**. Often summarized by Firth's slogan: "you shall know a word by the company it keeps." Words that appear in similar contexts have similar meanings.

This idea predates ML by decades (Harris 1954, Firth 1957), and motivated count-based vector-space models (LSA, HAL) long before [[word2vec]]. The 2013-era contribution was not the hypothesis itself but the empirical demonstration that distributional semantics, operationalized as a simple prediction task on billions of words, yields representations rich enough to support analogical reasoning and transfer learning at scale.

Every modern LLM is a distributional-semantics machine in disguise. [[autoregressive-language-modeling|Next-token prediction]] is the distributional hypothesis taken to its limit: model the distribution `P(w_t | context)` perfectly and you've absorbed the meaning of every word in the corpus by definition.

## References

- [[efficient-estimation-of-word-representations-mikolov-2013]]
