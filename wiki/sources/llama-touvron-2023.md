---
title: "LLaMA: Open and Efficient Foundation Language Models"
type: source
created: 2026-11-02
updated: 2026-11-02
sources:
  - llama-touvron-2023.pdf
arxiv_id: "2302.13971"
authors:
  - Hugo Touvron
  - Thibaut Lavril
  - Gautier Izacard
  - Xavier Martinet
  - Marie-Anne Lachaux
  - Timothee Lacroix
  - Baptiste Rozière
  - Naman Goyal
  - Eric Hambro
  - Faisal Azhar
  - Aurelien Rodriguez
  - Armand Joulin
  - Edouard Grave
  - Guillaume Lample
year: 2023
introduces:
  - [[llama]]
status: complete
importance: high
tags:
  - 2023
  - paper
---

# LLaMA: Open and Efficient Foundation Language Models

Touvron et al., [[meta|Meta AI]]. [arXiv:2302.13971](https://arxiv.org/abs/2302.13971), v1, 27 February 2023.

## Summary

The paper introduces [[llama|LLaMA]], models from 7B to 65B parameters trained only on publicly available data. Its aim is the best performance at each inference budget, reached by training on more tokens than is typical, about 1.4T after tokenization.

## Key points

- LLaMA-13B outperforms [[gpt-3|GPT-3]] (175B) on most benchmarks, despite being 10× smaller.
- LLaMA-65B is competitive with [[chinchilla-hoffmann-2022|Chinchilla]]-70B and PaLM-540B.
- The authors argue that a 13B model runnable on a single GPU widens access to study of LLMs.
- All models were released to the research community.
- The paper also reports evaluations of bias, toxicity and misinformation.

## What it does not show

These are base models, not chat assistants. Comparisons are against the benchmark results the paper reports, and "most benchmarks" doesn't mean all.

## Concepts & entities

- [[llama]] · [[meta]] · [[decoder-only-transformer]] · [[scaling-laws]]

## References

- [arXiv:2302.13971](https://arxiv.org/abs/2302.13971)
