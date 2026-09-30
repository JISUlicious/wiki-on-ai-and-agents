---
title: "Mo' Models, Mo' Problems: How to best select model pools when designing Multi-Agent Systems"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - mo-models-mo-problems-model-pool-selection-marjanovic-2026.md
arxiv_id: "2609.17306"
authors:
  - Sara Vera Marjanović
  - Jiacheng Xu
  - Aleksandr Laptev
  - Grigor Nalbandyan
  - Erik Arakelyan
  - Evelina Bakhaturina
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Mo' Models, Mo' Problems: How to best select model pools when designing Multi-Agent Systems

**Sara Vera Marjanović et al.** (University of Copenhagen; [[nvidia|NVIDIA]]) — [arXiv:2609.17306](https://arxiv.org/abs/2609.17306), cs.MA, 15 Sep 2026. Preprint.

## Summary

With nearly 3 million models on Hugging Face, which ones should go into a heterogeneous [[multi-agent|multi-agent system]]? The paper assembles a pool of **23 models**: six families, 2B to 1.6T parameters, including four science specialists. It collects 5 generations per model on HLE, [[gpqa|GPQA-Diamond]] and FrontierScience-Olympiad. It then compares 8 selection strategies for picking k ∈ {3, 5, 10, 15, 20} models: size, family, LLM-chosen, accuracy, correct-answer diversity (IoU), error diversity, and two accuracy×diversity blends. Each pool is run through three simple MAS designs: a **before-generation router** (a clustering router in the AvengersPro style) and two **after-generation** combiners, **majority vote** and an **LLM-judge tournament** (GenSelect with gpt-oss-120b).

The headline is the gap between **oracle potential and achieved performance**. The oracle always improves with more models. The real systems almost always get worse.

## Key points

- **More models nearly always hurts.** Most deliberately chosen pools score below their own best single model, and often below a random pool. After-generation systems decline most steeply as k grows.
- **Single-family pools are the only reliable wins.** Gemma4, and sometimes OLMo3, beat the single best model. Pools with the highest predicted oracle gains (most accurate or largest models) do not deliver them.
- **Homogeneous ensembles work.** On HLE the best single model goes from 29.4% pass@1 to 32.2% with majority@5 and **36.5%** with judge@5. The combination methods are sound; heterogeneity is what breaks them.
- **Routers want correct-answer diversity.** IoU diversity gives the best router results, but only IoU and family pools ever beat the base model under routing.
- **Model signals.** Accurate models get the *same* questions right (Mantel r = 0.931) but make only weakly correlated errors (r = 0.384). Size correlates moderately with accuracy (r_s = 0.583). Physics and chemistry fine-tunes of Llama-3.1-8B do **not** beat their generalist base even in their own domains. Specialisation cannot be read off a model card.
- The judge agrees with a human annotator on 93% of decisions (κ 0.63).

## What it does not show

- It covers only simple MAS designs: one router backbone, no debate or during-generation interaction, and no tools or retrieval.
- The LLM-judge setup may be limited by context length at large k.
- The domain is science reasoning only. Family pools could not be tested at every k.
- The conclusion says pool growth "always seems to impair" performance, which is stronger than the abstract's "often degrades."

> [!note] Extends [[multi-agent]]
> That page already argues against *"uncritical 'more agents = better' framing"* using SIMAS's non-monotonic agent count and the diversity-collapse result. This paper adds the heterogeneous-pool side. Adding *different* models also degrades performance, and too much solution diversity pollutes the answer space. That cuts against the view that diversity is the cure for collapse.

## Related

- [[multi-agent]] · [[scaling-behavior-single-llm-multi-agent-systems-li-2026]] · [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]] — the prior evidence this extends.
- [[when-is-routing-meaningful-huot-2026]] · [[agent-as-a-router-zhou-2026]] — the routing side; behavioural differentiation as a precondition.
- [[self-consistency]] — the homogeneous majority-vote baseline that does work.
- [[llm-as-a-judge]] · [[mixture-of-experts]] — the judge-selection combiner, and the classical routing analogy.

## References

- [arXiv:2609.17306](https://arxiv.org/abs/2609.17306)
