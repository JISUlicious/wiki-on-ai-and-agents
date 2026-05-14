---
title: "Red Teaming Language Models to Reduce Harms: Methods, Scaling Behaviors, and Lessons Learned"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - ganguli-red-teaming-2022.md
arxiv_id: "2209.07858"
venue: arxiv preprint
authors:
  - Deep Ganguli
  - Liane Lovitt
  - Jackson Kernion
  - Amanda Askell
  - Yuntao Bai
  - "[[dario-amodei]]"
  - "[[tom-brown]]"
  - "[[sam-mccandlish]]"
  - "[[chris-olah]]"
  - "[[jared-kaplan]]"
first_author: Deep Ganguli
introduces:
  - "[[red-teaming]]"
year: 2022
tags:
  - 2022
status: complete
importance: high
---

# Red Teaming Language Models to Reduce Harms: Methods, Scaling Behaviors, and Lessons Learned

**Source**: `sources/ganguli-red-teaming-2022.md` (extracted from `ganguli-red-teaming-2022.pdf`)
**arXiv**: [2209.07858](https://arxiv.org/abs/2209.07858)
**Venue**: arxiv preprint (Aug 2022)
**Authors**: Deep Ganguli (first), Liane Lovitt, Jackson Kernion, Amanda Askell, Yuntao Bai, ..., [[dario-amodei]], [[tom-brown]], [[sam-mccandlish]], [[chris-olah]], [[jared-kaplan]] ([[anthropic]])

## Summary

The first **systematic, scaled** study of red-teaming language models — having humans actively try to elicit harmful outputs from models, then using the resulting attacks to study scaling behaviors and inform mitigations. Three contributions:

1. **Scaling study across 4 model types × 3 sizes**:
   - Plain LM
   - LM prompted to be helpful-honest-harmless (HHH-prompted)
   - LM with rejection sampling
   - LM trained with [[rlhf]] (an early Claude-precursor "HH model")
   - Sizes: 2.7B, 13B, 52B parameters.
   - **Finding**: RLHF models get **increasingly difficult to red-team as they scale**. The other model types show a flat trend with scale.
2. **Public release of 38,961 red-team attacks** with structured metadata, for community analysis.
3. **Methodology transparency**: detailed instructions, processes, statistical methodologies, and uncertainty quantification — establishing a template for how to red-team LLMs systematically.

The paper finds that harmful outputs span a spectrum from overt offensive language to subtler "non-violent unethical" outputs (deception, manipulation, encouragement of bad behavior). Different model types fail in different ways: prompted models fail on edge-case framings; rejection-sampling models fail more rarely but with similar profiles; RLHF models fail rarely and refuse most adversarial prompts (sometimes over-cautiously).

This paper established **red-teaming as a standard pre-deployment practice** for frontier LLMs. By 2024, every major lab runs an internal red-team or contracts with external ones; the practice is now required by the EU AI Act and US executive order on AI.

## Key Points

- **Red-team setup**: hire human crowdworkers, give them instructions to attempt to elicit harmful outputs, log the attempts and the models' responses.
- **Scaling finding (most important)**: RLHF models are harder to red-team as they grow; other model types flat with scale. This is one of the empirical foundations of the "post-training matters more than pretraining for safety" view.
- **Dataset released**: 38,961 attacks across the 4 model types × 3 sizes. Lets the community study attack distributions, identify gaps, and benchmark new defenses.
- **Methodology paper as much as a results paper**: detailed accounts of recruitment, instructions, ethics review, statistical analysis, and known limitations. Set the template for subsequent red-team papers.
- **Harm taxonomy**: offensive language, factual errors with potential harm, discrimination, privacy violations, dangerous instructions, deceptive content. Categorized for analysis.
- **Norms-establishment work**: the paper argues for shared community norms, methods, and technical standards for red-teaming.

## Entities Mentioned

- Deep Ganguli — first author; long-time Anthropic societal-impacts lead
- [[anthropic]] — origin org
- Anthropic founders (Amodei, Brown, Olah, Kaplan, McCandlish) — senior authors

## Concepts Discussed

- [[red-teaming]] — the practice
- [[rlhf]] — model type studied
- Scalable oversight (related; not yet a dedicated concept page)

## Notable Quotes

> "We find that the RLHF models are increasingly difficult to red team as they scale, and we find a flat trend with scale for the other model types." — Abstract

> "We release our dataset of 38,961 red team attacks for others to analyze and learn from." — Abstract

## References

_Original source: `sources/ganguli-red-teaming-2022.md`_
