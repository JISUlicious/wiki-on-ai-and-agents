---
title: "Scaling Laws for Neural Language Models"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - kaplan-scaling-2020.md
arxiv_id: "2001.08361"
venue: arxiv preprint
authors:
  - "[[jared-kaplan]]"
  - "[[sam-mccandlish]]"
  - Tom Henighan
  - "[[tom-brown]]"
  - Benjamin Chess
  - Rewon Child
  - Scott Gray
  - "[[alec-radford]]"
  - Jeffrey Wu
  - "[[dario-amodei]]"
first_author: "[[jared-kaplan]]"
introduces:
  - "[[scaling-laws]]"
year: 2020
tags:
  - 2020
status: complete
importance: high
---

# Scaling Laws for Neural Language Models

**Source**: `sources/kaplan-scaling-2020.md` (extracted from `kaplan-scaling-2020.pdf`)
**arXiv**: [2001.08361](https://arxiv.org/abs/2001.08361)
**Venue**: arxiv preprint (Jan 2020)
**Authors**: [[jared-kaplan]] (Johns Hopkins + OpenAI), [[sam-mccandlish]], Tom Henighan, [[tom-brown]], Benjamin Chess, Rewon Child, Scott Gray, [[alec-radford]], Jeffrey Wu, [[dario-amodei]] ([[openai|OpenAI]])

## Summary

This paper empirically formalizes the **power-law relationship** between cross-entropy language modeling loss and three scaling dimensions: **model size** `N` (parameters), **dataset size** `D` (tokens), and **training compute** `C`. Over more than **seven orders of magnitude** of scale, loss decreases as a clean power law in each axis, holding the other two non-bottlenecking.

Other architectural details — width vs. depth, number of attention heads, feed-forward dimension — have minimal effects within wide ranges. What matters is `N`, `D`, `C`. This was the empirical proof of concept that **capability planning** could become a budget-allocation problem rather than an architectural one.

The paper's compute-optimal recommendation: for a fixed compute budget, train **larger models on relatively less data** than naive convergence would suggest, stopping early. This recommendation directly motivated the [[gpt-3-brown-2020|GPT-3]] scale-up later that year — Kaplan, McCandlish, Henighan, Brown, Child, Radford, Wu, and Amodei are all GPT-3 co-authors.

The Kaplan laws were later **corrected** by [[chinchilla-hoffmann-2022|Hoffmann et al. 2022 ("Chinchilla")]], who showed that model size and training tokens should be scaled roughly **equally** for compute-optimal training — implying Kaplan-era models including GPT-3 were significantly *undertrained* on data. The Chinchilla correction reshaped subsequent training recipes (Llama, Gemini, GPT-4 era).

See [[scaling-laws]] for the conceptual treatment.

## Key Points

- **Power laws** in `N` (params), `D` (tokens), `C` (compute) over 7+ orders of magnitude.
- **Architectural details** (width vs. depth ratio, attention head count, etc.) have minor effects compared to scale.
- **Compute-optimal allocation**: bigger models, less data per parameter, stop before convergence. *Later corrected by Chinchilla.*
- **Sample efficiency** improves with scale: larger models extract more from each token.
- **Predictability**: loss at scale `X` can be extrapolated from runs at scale `X/1000+`, enabling capability forecasting before spending the compute.
- **Direct line to [[gpt-3-brown-2020|GPT-3]]**: most authors went on to write GPT-3 the same year. The scale-up was the explicit Kaplan-laws prediction in practice.

## Entities Mentioned

- [[jared-kaplan]] — co-lead (later Anthropic co-founder)
- [[sam-mccandlish]] — co-lead (later Anthropic co-founder)
- [[tom-brown]], [[alec-radford]], [[dario-amodei]] — all later authored [[gpt-3-brown-2020|GPT-3]]
- [[openai]] — origin org

## Concepts Discussed

- [[scaling-laws]] — the central topic; Kaplan laws are the original empirical formulation

## Notable Quotes

> "The loss scales as a power-law with model size, dataset size, and the amount of compute used for training, with some trends spanning more than seven orders of magnitude. Other architectural details such as network width or depth have minimal effects within a wide range." — Abstract

> "Larger models are significantly more sample-efficient, such that optimally compute-efficient training involves training very large models on a relatively modest amount of data and stopping significantly before convergence." — Abstract _(this recommendation was later corrected by Chinchilla)_

## References

_Original source: `sources/kaplan-scaling-2020.md`_
_See also [[chinchilla-hoffmann-2022]] for the corrective re-derivation._
