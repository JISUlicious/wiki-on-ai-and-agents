---
title: "Agentic Context Engineering: Evolving Contexts for Self-Improving LMs"
type: source
created: 2026-05-17
updated: 2026-05-17
sources:
  - agentic-context-engineering-zhang-2025.md
arxiv_id: "2510.04618"
venue: ICLR 2026
authors:
  - Qizheng Zhang
  - Changran Hu
  - Shubhangi Upasani
  - Boyuan Ma
  - Fenglu Hong
  - Vamsidhar Kamanuru
  - Jay Rainton
  - Chen Wu
  - Mengmeng Ji
  - Hanchen Li
  - Urmish Thakker
  - James Zou
  - Kunle Olukotun
year: 2025
introduces:
  - "[[agentic-context-engineering]]"
  - "[[self-evolving-context]]"
tags:
  - 2025
status: complete
importance: high
---

# Agentic Context Engineering: Evolving Contexts for Self-Improving Language Models

Qizheng Zhang, Changran Hu, et al. (Stanford, SambaNova Systems, UC Berkeley). [arXiv:2510.04618](https://arxiv.org/abs/2510.04618). Published at ICLR 2026. Equal first-authorship between Qizheng Zhang (Stanford) and Changran Hu (SambaNova / Berkeley); senior authors include James Zou and Kunle Olukotun (both Stanford).

## Summary

ACE formalizes the model's **prompt/context as an evolving software artifact** rather than a static string or a one-shot summary. The framework treats context as a structured, itemized "playbook" of bullets — each with metadata (id, helpful/harmful counters) and content (a strategy, domain concept, or failure mode) — that accumulates and refines over time. A three-role agentic architecture maintains this playbook: a **Generator** produces a reasoning trajectory on a new query, a **Reflector** critiques that trajectory to extract concrete lessons, and a **Curator** commits those lessons as compact **delta updates** to the playbook via deterministic, non-LLM merge logic. A grow-and-refine step periodically de-duplicates bullets via semantic embeddings.

The delta-update mechanism is the load-bearing idea: by replacing monolithic LLM rewrites with localized, itemized edits, ACE sidesteps both **brevity bias** (where prompt optimizers like GEPA collapse to short generic instructions) and **context collapse** (where iterative full-context rewriting erodes detail — a case study shows context dropping from 18,282 tokens at 66.7% accuracy to 122 tokens at 57.1% in a single rewrite step). Results: **+10.6% on agent benchmarks** (AppWorld) and +8.6% on financial-domain benchmarks, with 86.9% lower adaptation latency on average. On the AppWorld leaderboard, ACE with the open-source DeepSeek-V3.1 matches the top-ranked production agent IBM-CUGA (powered by GPT-4.1) on the overall average and surpasses it on the harder test-challenge split. Conceptually, the paper is the **academic counterpart to industry "agent skills" and memory tooling** ([[anthropic-agent-skills-2025|Anthropic Agent Skills]], [[a-mem-xu-2025|A-MEM]], Dynamic Cheatsheet) — placing prompt-engineering, agent memory, and self-improvement under one rubric.

## Key Points

- **Three-role architecture (Generator / Reflector / Curator).** Inspired by Dynamic Cheatsheet but with a strict division of labor: Generator emits a reasoning trajectory and flags which existing bullets were useful or misleading; Reflector distills concrete insights (with optional iterative refinement); Curator commits structured delta items. Separating reflection from curation is shown by ablation to materially lift quality.
- **Delta updates instead of full-context rewrites.** Context is a list of itemized bullets with stable IDs; updates are local edits and appends merged by deterministic non-LLM logic. This avoids the latency, cost, and information loss of monolithic LLM rewrites, and lets multiple deltas merge in parallel for batched adaptation.
- **Sidesteps brevity bias and context collapse.** Argues contexts should be **comprehensive playbooks, not concise summaries** — LLMs can distill relevance at inference time, so detail should be preserved upstream. Demonstrates context collapse empirically on AppWorld with Dynamic Cheatsheet as a baseline.
- **Self-improvement without labels.** ACE adapts using execution feedback and environment signals alone (no ground-truth labels), boosting AppWorld accuracy by up to 17.1% from execution feedback.
- **AppWorld benchmark headline.** Open-source DeepSeek-V3.1 + ACE matches GPT-4.1-powered IBM-CUGA on the overall AppWorld average and beats it on test-challenge — a strong claim that *better context* can substitute for a stronger base model.
- **Bridges prompt-engineering with agent-memory research.** Cites and builds on GEPA, TextGrad, Reflexion, Dynamic Cheatsheet, and A-MEM; positions context adaptation as the central paradigm for self-improving LLM systems alongside long-context inference and KV-cache reuse.

## Entities

- [[stanford-university]] — Qizheng Zhang, James Zou, Kunle Olukotun.
- [[sambanova]] — Changran Hu (also Berkeley), Shubhangi Upasani, Boyuan Ma, and co-authors.
- [[uc-berkeley]] — Changran Hu, Hanchen Li.

## Concepts

- [[agentic-context-engineering]] — the named framework introduced here.
- [[self-evolving-context]] — the broader paradigm of treating context as a dynamic artifact.
- [[agent-skills]] — industry-side counterpart (Anthropic) to ACE's playbook idea.
- [[memory-management]] — closely related: bullets-with-counters resemble structured agent memory entries.
- [[in-context-learning]] — ACE pushes ICL into a long-horizon, accumulating regime.
- [[appworld]] — primary agent benchmark used; ACE's headline result is on the AppWorld leaderboard.

## Quotes

> "ACE prevents collapse with structured, incremental updates that preserve detailed knowledge and scale with long-context models."

> "contexts should function not as concise summaries, but as comprehensive, structured playbooks that are detailed, inclusive, and rich with domain insights."

> "at step 60 the context contained 18,282 tokens and achieved an accuracy of 66.7, but at the very next step it collapsed to just 122 tokens, with accuracy dropping to 57.1 — worse than the baseline accuracy of 63.7 without adaptation."

> "On the AppWorld benchmark leaderboard, ACE surpasses the top-1-ranked production-level agent IBM-CUGA (powered by GPT-4.1) while using an open-source model (DeepSeek-V3.1)."

## References

- Suzgun et al. 2025 — Dynamic Cheatsheet (closest precursor; ACE adopts its agentic spirit while fixing context collapse).
- Agrawal et al. 2025 — GEPA (prompt optimizer cited as exhibiting brevity bias).
- Yuksekgonul et al. 2025 — TextGrad.
- Shinn et al. 2023 — Reflexion.
- Xu et al. 2025 — A-MEM (see [[a-mem-xu-2025]]).
- Trivedi et al. 2024 — AppWorld benchmark.
- Marreed et al. 2025 — IBM-CUGA (top AppWorld leaderboard entry).
