---
title: "Representation Engineering: A Top-Down Approach to AI Transparency"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - zou-representation-engineering-2023.md
arxiv_id: "2310.01405"
venue: arxiv preprint (Oct 2023; v4 Mar 2025)
authors:
  - "[[andy-zou]]"
  - Long Phan
  - Sarah Chen
  - James Campbell
  - Phillip Guo
  - Richard Ren
  - Alexander Pan
  - "[[dan-hendrycks]]"
first_author: "[[andy-zou]]"
introduces:
  - "[[representation-engineering]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Representation Engineering: A Top-Down Approach to AI Transparency

**Source**: `sources/zou-representation-engineering-2023.md`
**arXiv**: [2310.01405](https://arxiv.org/abs/2310.01405)
**Venue**: arxiv preprint (Oct 2023; updated through Mar 2025)
**Authors**: [[andy-zou]] (first), Long Phan, Sarah Chen, James Campbell, Phillip Guo, Richard Ren, Alexander Pan, ..., [[dan-hendrycks]] (senior) — [[center-for-ai-safety]] + CMU + UC Berkeley + Stanford + EleutherAI + others

## Summary

A landmark survey/methods paper that names and consolidates **Representation Engineering (RepE)** as a coherent research program: a top-down approach to AI transparency that places **representations** (linear directions in activation space) at the center of analysis, rather than individual neurons or circuits.

Where [[mechanistic-interpretability]] takes a bottom-up approach (reverse-engineer the network's algorithm from neurons up), RepE takes a top-down one: identify a high-level concept (honesty, harmlessness, power-seeking, emotion, fairness, ...), find the linear direction in activation space that represents it, then **read** that direction (monitoring) or **write** to it (steering / [[activation-addition|activation addition]]). The paper provides a unified methodology — reading via "Linear Artificial Tomography" (LAT) probes, controlling via vector addition — and demonstrates it across a wide range of safety-relevant phenomena.

RepE is the most-cited consolidation of the **[[steering|steering]] paradigm**. The paper's experimental coverage is exhaustive: honesty/truthfulness, memorization, emotion (anticipating the [[emotion-concepts-anthropic-2026|Anthropic 2026 emotion-concepts]] paper), bias, power-seeking, instruction-following, fairness, hallucination — for each, RepE shows you can both *detect* the concept's presence and *manipulate* it via residual-stream interventions.

This paper, together with [[activation-engineering-turner-2023|Turner's ActAdd]], defines the steering field. The associated [`representation-engineering` GitHub repo](https://github.com/andyzoujm/representation-engineering) became a standard toolkit.

## Key Points

- **Cognitive-neuroscience framing**: RepE is inspired by how neuroscientists characterize cognitive phenomena via population-level neural representations, not single neurons.
- **Two operations**:
  1. **Reading** ("Linear Artificial Tomography", LAT): given a concept, find the activation direction that encodes it. Methods include difference-in-means, linear probes, PCA on contrastive pairs.
  2. **Controlling**: modify activations along the direction to push the model toward or away from the concept. This is [[activation-addition|ActAdd]] generalized.
- **Safety-relevant concepts demonstrated**:
  - **Honesty / hallucination**: detect lying, steer toward truthful answers (related to [[inference-time-intervention|ITI]]).
  - **Harmlessness / refusal**: control whether the model refuses harmful queries.
  - **Power-seeking**: detect & reduce dispositions toward self-preservation, resource acquisition.
  - **Emotion**: precursor to [[functional-emotions|Anthropic's emotion-concepts paper]]; same linear-direction structure.
  - **Bias and fairness**: detect demographic biases; reduce them via activation steering.
- **Top-down vs. bottom-up**: complements [[mechanistic-interpretability]] (which reverse-engineers from neurons) rather than competing. RepE accepts representations as the unit of analysis without first decomposing them into circuits.
- **Practical for safety**: representations are *causal* — interventions on them change behavior in predictable ways. This makes RepE a candidate building block for alignment.

## Entities Mentioned

- [[andy-zou]] — first author (CMU + Center for AI Safety)
- [[dan-hendrycks]] — senior author; director of [[center-for-ai-safety]]
- [[center-for-ai-safety]] — primary affiliation

## Concepts Discussed

- [[representation-engineering]] — the framework
- [[steering]] — the broader paradigm
- [[linear-representation-hypothesis]] — the theoretical bedrock
- [[mechanistic-interpretability]] — complementary bottom-up alternative
- Many safety-relevant concept directions (honesty, harmlessness, power-seeking, emotion, fairness)

## Notable Quotes

> "We identify and characterize the emerging area of representation engineering (RepE), an approach to enhancing the transparency of AI systems that draws on insights from cognitive neuroscience. RepE places representations, rather than neurons or circuits, at the center of analysis." — Abstract

> "These methods can provide traction on a wide range of safety-relevant problems, including honesty, harmlessness, power-seeking, and more." — Abstract

## References

_Original source: `sources/zou-representation-engineering-2023.md`_
_Code: https://github.com/andyzoujm/representation-engineering_
