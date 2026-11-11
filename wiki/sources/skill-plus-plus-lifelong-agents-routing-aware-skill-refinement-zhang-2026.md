---
title: "Skill++: Towards Lifelong Agents via Routing-Aware Skill Refinement"
type: source
created: 2026-11-10
updated: 2026-11-10
sources:
  - skill-plus-plus-lifelong-agents-routing-aware-skill-refinement-zhang-2026.md
tags:
  - 2026
  - paper
status: complete
importance: medium
authors:
  - Hanrong Zhang
  - Minghao Liang
  - Jiawei Wu
  - Yankai Chen
  - Shicheng Fan
  - Zhiwei Liu
  - Enze Ma
  - Junwei Yuan
  - Tianyi Wang
  - Dehai Min
  - Kening Zheng
  - Xue Liu
  - Philip S. Yu
year: 2026
venue: arXiv preprint
arxiv_id: "2611.00544"
---

# Skill++: Towards Lifelong Agents via Routing-Aware Skill Refinement

**Source:** Hanrong Zhang, Minghao Liang, …, Philip S. Yu (University of Illinois Chicago, McGill University, Trip.com Group and others) — [arXiv:2611.00544](https://arxiv.org/abs/2611.00544), cs.AI, v1 31 Oct 2026. Preprint.

## Summary

Under progressive disclosure, an LLM router picks [[agent-skills|skills]] by name and description only and loads the body later. As a library grows, a new skill with similar metadata can **displace an earlier skill that used to be routed correctly**, even though neither the earlier skill nor the router changed. The paper calls this *routing interference* and frames it as **lifelong skill routing**, a class-incremental [[continual-learning]] problem: earlier skills must stay selectable (stability) and new ones must still win their own queries (plasticity). With the original metadata, 62 of 115 queries that were once routed correctly fail by the time the library reaches 731 skills.

Skill++ edits only the **incoming** skill's name and description once, at insertion. It does not retrain the router and needs no labelled queries. It retrieves the most similar earlier skills from a record of originals, then applies three components: (1) **description differentiation**, which rewrites the description from the skill's own body and centres it on what separates the skill from its closest predecessor; (2) **scope exclusions**, a "Does not cover …" clause naming related work handled elsewhere; (3) **name clarification**, which qualifies a name that overlaps an earlier one.

## Key points

- **Main result** (SRA-Bench/BigCodeBench, 198 multi-skill queries, library grows from 27 to 731 skills over ten sessions, Claude Haiku 4.5 router, two-call shortlist-and-rerank): full coverage FC@5 at the last session is **76.3% vs 36.4%** with the original metadata. The best baseline reaches 67.8% and CC-Pages 60.1%.
- **Two error types.** *Substitution*, where a similar new skill takes a required skill's slot, falls from 46.0% to 6.5% of Vanilla's errors. *Crowding*, where generic or redundant skills fill the top-K, falls from 41.5% to 20.0%. Differentiation and name clarification mostly fix substitution; exclusions mostly fix crowding.
- **Plasticity is kept.** Refined new skills rank first for their own queries 56.9% of the time vs 51.9% under Vanilla.
- **Generalisation.** Gains hold on SkillRet and SkillRouter (libraries of about 2,000 skills), across five routers (GPT-5.4-mini, GPT-5.6 Luna/Sol, Qwen3.7-Flash, Haiku 4.5; +23.1 pp on average at the last session), and inside [[codex-cli|Codex CLI]] (+34.3 pp with GPT-5.6 Luna, +11.1 pp with Sol). Gains are largest where the router alone is weakest.
- **Ablation.** Removing differentiation costs the most (76.3 → 56.6 FC@5).
- **Baselines.** Body-distilled descriptions, A-MEM-style rewriting of neighbouring skills, retriever fine-tuning at each checkpoint, and Cluster-Contrastive Capability Pages.

## Caveats

The evaluation queries are benchmark-annotated multi-skill queries in mostly coding domains. Earlier skills keep their original metadata, so the gain comes entirely from how new arrivals are described. Text-only skills; the authors leave multimodal skills to future work.

## Concepts & entities

- [[skill-routing]] · [[skill-selection]] · [[skill-library]] · [[skill-md-format]] · [[agent-skills]] — the routing-by-metadata setting.
- [[continual-learning]] · [[memory-evolution]] · [[a-mem]] — the framing, and the neighbour-rewriting baseline it adapts.
- [[compositional-skill-routing-gao-2026]] · [[skillmoo-gong-2026]] · [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] — related skill-routing work in the wiki.
- [[codex-cli]] · [[tool-search]].

## References

- [arXiv:2611.00544](https://arxiv.org/abs/2611.00544)
