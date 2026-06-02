---
title: "SkillReducer: Optimizing LLM Agent Skills for Token Efficiency"
type: source
created: 2026-06-02
updated: 2026-06-02
sources:
  - skillreducer-gao-2026.md
arxiv_id: "2603.29919"
authors:
  - Yudong Gao
  - Zongjie Li
  - Yuanyuan Yuan
  - Zimo Ji
  - Pingchuan Ma
  - Shuai Wang
first_author: Yudong Gao
year: 2026
introduces:
  - "[[skill-optimization]]"
tags:
  - 2026
  - paper
status: complete
importance: high
---

# SkillReducer: Optimizing LLM Agent Skills for Token Efficiency

**Source file**: [[skillreducer-gao-2026]]
**arXiv**: [arXiv:2603.29919](https://arxiv.org/abs/2603.29919) (v1, 31 Mar 2026, cs.SE)
**Authors**: Yudong Gao, Zongjie Li, Zimo Ji, Shuai Wang (HKUST); Yuanyuan Yuan ([[tsinghua-university|Tsinghua University]]); Pingchuan Ma (Zhejiang University of Technology)
**Affiliation**: The Hong Kong University of Science and Technology (primary); Tsinghua University; Zhejiang University of Technology
**Year**: 2026

## Summary

SkillReducer reframes [[agent-skills|agent skills]] as **software artifacts with a token-cost problem** and applies classical software-engineering techniques to debloat them. The starting observation is that skills, despite being designed to *save* tokens (reusable knowledge loaded on demand rather than repeated in every conversation), as currently authored often *increase* token consumption: every token of skill content injected into the context window incurs both monetary cost (a 10K-token skill costs $0.03–$0.15 per invocation) and attention dilution. The authors' large-scale empirical audit of **55,315 publicly available skills** finds systemic inefficiency in both functional components of a skill — the routing **description** (the L1 metadata used to decide whether to invoke the skill) and the **body** (the L2 instructions injected on invocation): **26.4% of skills lack a routing description entirely** (44.1% are missing or under 20 tokens), **over 60% of body content is non-actionable** background/examples/templates (only 38.5% is core rules), and reference-heavy skills inject tens of thousands of tokens regardless of task relevance (100 SkillHub skills bundle 1.67M reference tokens). The diagnosed root cause is the **absence of separation of concerns** in skill authoring: specification, documentation, and data are mixed into monolithic files.

The contribution is a **two-stage, training-free optimizer** that compresses both layers while preserving — and on average slightly improving — functional quality. **Stage 1 (routing layer)** treats a description's semantic clauses as the "changes" in [delta debugging](https://en.wikipedia.org/wiki/Delta_debugging) and routing correctness as the predicate, running DDMIN against a fast *simulated routing oracle* (with TF-IDF-selected distractors plus an LLM-generated adversarial "shadow" skill) to find the 1-minimal description, then validating in the real Claude Code CLI with selective restore; missing descriptions are generated from the body. **Stage 2 (body layer)** adapts program slicing: an LLM taxonomy classifier segments the body into core-rule / background / example / template / redundant items, keeps only core rules always-loaded, and converts the rest into on-demand reference modules (the monolithic→tiered [progressive-disclosure](#) transformation), gated by a faithfulness check and a task-based eval with a self-correcting feedback loop. Evaluated on **600 skills** plus the external SkillsBench benchmark, SkillReducer achieves **48% mean description compression and 39% mean body compression while improving functional quality by 2.8%** — a "less-is-more" effect where removing non-essential content reduces distraction in the context window. Benefits transfer across **five models from four families** (mean retention 0.965) and to an independent agent framework (OpenCode, retention 0.944). This is the **efficiency** counterpart to the security-and-architecture framing of [[agent-skills-survey-xu-2026]]: where that survey catalogs the skill paradigm, SkillReducer treats skill bloat as an optimization target.

## Key Points

- **Two-stage, structure-aware optimizer (training-free).** Unlike flat token-level pruning ([LLMLingua](#)) or summarization, SkillReducer exploits the bipartite structure of skills — one stage per functional layer — and adapts three SE techniques: software debloating, delta debugging, and program slicing.
- **Stage 1 = delta-debugging description compression.** A simulated oracle (DeepSeek-R1 router, ~2s/call, with TF-IDF distractors + an LLM adversarial shadow skill) drives DDMIN over *semantic clauses* (O(n log n) oracle calls) to the 1-minimal description; Phase 2 re-validates each kept clause in the real Claude Code CLI and greedily restores deleted clauses if a query stops triggering. Result: **48% mean description reduction** (56.5% for the 171 skills with existing descriptions; 44.6% for the 429 with generated descriptions), **100% routing preservation** across all 536 non-obsolete skills.
- **Stage 2 = taxonomy-driven progressive disclosure.** A DeepSeek-V3 classifier sorts paragraph-level body items into core-rule / background / example / template / redundant; only core rules stay always-loaded; the rest become on-demand `examples.md` / `templates.md` / `background.md` modules with "when"/topic annotations; references are deduped against the body and dropped below 30 tokens. Result: **39% mean body reduction** (~1,000 tokens/skill; 75% on SkillsBench).
- **+2.8% functional quality — the "less-is-more" effect.** Compressed Condition C beats original Condition A on average (0.742 vs 0.722, p=0.002), with an **86.0% pass rate** (scoreC ≥ scoreA), 25.3% improving and only 14.0% regressing; **only 4.7% of skills are true compression failures** (the rest are skill obsolescence or eval noise). The effect scales with body length (+11.8pp for long official skills). Independent support cited from "Lost in the Middle" and context-length-degradation studies.
- **The empirical audit of real skills is itself a headline finding.** Across 55,315 wild skills: **26.4% have no description** (44.1% missing/<20 tokens), **>60% of body content is non-actionable** (only 38.5% core rules, 40.7% background, 12.9% examples, 7.6% templates, 0.3% redundant), 14.8% bundle reference files that can inject massive token volumes. The five-category taxonomy is structurally grounded (GMM clustering yields five matching clusters, silhouette 0.393).
- **Directly relevant to L1/L2 progressive-disclosure token budgets.** Stage 1 optimizes the L1 metadata/routing layer; Stage 2 automates the monolithic→tiered (L2 core + L3 on-demand) restructuring that the [[skill-md-format|SKILL.md]] three-level disclosure model assumes but most authors never do (67.5% of skills are single-file monolithic). End-to-end expected input savings of 26.8% (using core ratio ρ=0.383 and reference-load rate p=0.3).
- **Generalization and economics.** Retention 0.939–0.986 across GLM-5, DeepSeek-V3, Qwen3-max, GPT-OSS-120B, Qwen2.5-7B (mean 0.965), including a weak 7B model and a tool-call-less model using flat injection; transfers to the OpenCode framework. Compressing 600 skills costs ~$14–18 (one-time, ~20–40 LLM calls/skill), amortized within a few hundred invocations.
- **Beats four budget-matched baselines** (p ≤ 0.003): LLMLingua (retention 0.820 — perplexity pruning discards predictable-but-critical tokens), direct LLM compression (0.918), truncation (0.845), random sentence removal (0.750); SkillReducer reaches 0.949.
- **Skill obsolescence as an ecosystem signal.** 10.7% of skills did not trigger even with their original description, and Condition D (no skill) already passes 98.9% of SkillsBench — as models get stronger some skills become redundant, motivating model-adaptive skill lifecycle management (Gate 2 can flag retirement candidates). Weaker models benefit most from skills.

## Entities Mentioned

- [[tsinghua-university]] — co-author affiliation (Yuanyuan Yuan)
- [[claude-code]] — the agent runtime used for Phase 2 real-trigger routing validation; the skill protocol all 600 skills target
- [[deepseek]] — DeepSeek-V3 (compression/classification) and DeepSeek-R1 (simulated routing oracle) are the workhorse models; the evaluator is Qwen3.5 under a separate session to prevent leakage
- HKUST (Hong Kong University of Science and Technology), Zhejiang University of Technology — author affiliations (no wiki pages yet)

## Concepts Discussed

- [[skill-optimization]] — the concept this paper anchors: post-authoring, build-time debloating of skills for token efficiency
- [[agent-skills]] — the artifact being optimized; SkillReducer treats skills as software with an optimization ecosystem they currently lack
- [[skill-md-format]] — the SKILL.md description + body + references structure and its three-level progressive disclosure (L1/L2/L3) that Stage 2 automates
- [[context-assembly]] — skill content is one contributor to the context window; the "less-is-more"/attention-dilution finding is a context-assembly result
- [[agent-skills-survey-xu-2026]] — companion academic anchor for the skill paradigm; SkillReducer is the efficiency-and-debloating counterpart to that survey's architecture/security framing (both cite the ~26% empirical skill-quality figure, from different angles)
- [[skillopt-yang-2026]] — adjacent skill-optimization work (cross-reference)
- Delta debugging, program slicing, software debloating — the SE techniques adapted for Stages 1 and 2 (no dedicated pages yet)
- Prompt compression — the prior art SkillReducer outperforms (LLMLingua, Selective Context, Gisting, RECOMP) by being structure-aware rather than flat (no dedicated page yet)

## Notable Quotes

> "Ironically, skills as currently authored often increase rather than decrease token consumption."

> "26.4% lack routing descriptions entirely, over 60% of body content is non-actionable, and reference files can inject tens of thousands of tokens per invocation."

> "S KILL R EDUCER achieves 48% description compression and 39% body compression while improving functional quality by 2.8%, revealing a less-is-more effect where removing non-essential content reduces distraction in the context window."

> "These three findings share a common root cause: the absence of separation of concerns in skill authoring. Authors mix specification (core rules), documentation (background and examples), and data (templates and references) into monolithic files."

> "As LLMs grow more capable, skills may become redundant ... Yet skills remain essential for weaker models ... This tension motivates adaptive skill lifecycle management."

## References

_Original source: `sources/skillreducer-gao-2026.md`_
_arXiv: [arXiv:2603.29919](https://arxiv.org/abs/2603.29919) (v1, 2026-03-31, cs.SE)_

Selected works cited:
- Zeller & Hildebrandt (2002) — Delta debugging ("Simplifying and isolating failure-inducing input"), IEEE TSE
- Weiser (1984) — Program slicing, IEEE TSE
- Quach, Prakash & Yan (2018) — Software debloating, USENIX Security
- Jiang et al. (2023) — LLMLingua: prompt compression [arXiv via EMNLP] — the primary baseline
- Mu, Li & Goodman (2023) — Gisting / learning to compress prompts with gist tokens, NeurIPS
- Nielsen (2006) — Progressive disclosure (Nielsen Norman Group)
- N. F. Liu et al. (2024) — "Lost in the Middle: How Language Models Use Long Contexts," TACL — cited for the less-is-more / distraction finding
- Shi et al. (2023) — "Large Language Models Can Be Easily Distracted by Irrelevant Context," ICML
- Du et al. (2025) — "Context Length Alone Hurts LLM Performance" [arXiv:2510.05381]
- X. Li et al. (2026) — SkillsBench [arXiv:2602.12670] — the external benchmark
- Anthropic (2025) — Claude Code docs → [[claude-code]]
- DeepSeek-AI (2024/2025) — DeepSeek-V3 / DeepSeek-R1 → [[deepseek]]
- Qwen Team (2025) — Qwen3 technical report (Qwen3.5 evaluator) [arXiv:2505.09388]
