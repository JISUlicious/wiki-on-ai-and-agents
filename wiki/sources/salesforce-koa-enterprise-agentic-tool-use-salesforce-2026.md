---
title: "Salesforce Koa: An Enterprise Language Model for Agentic Tool Use"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - salesforce-koa-enterprise-agentic-tool-use-salesforce-2026.md
arxiv_id: "2609.15066"
authors:
  - Zixiang Chen
  - Sufeng Niu
  - Yingchi Liu
  - Wenting Zhao
  - Akshara Prabhakar
  - Shubham Mehrotra
  - Silvio Savarese
  - Phil Mui
  - Huan Wang
first_author: Zixiang Chen
year: 2026
venue: "Technical report (Salesforce Agentforce & AI Research)"
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Salesforce Koa: An Enterprise Language Model for Agentic Tool Use

**Zixiang Chen, Sufeng Niu, Yingchi Liu, Wenting Zhao, Akshara Prabhakar, Shubham Mehrotra** (co-first) plus 18 others, with Phil Mui and Huan Wang as corresponding authors. Salesforce Agentforce & AI Research, trained "in partnership with [[nvidia|NVIDIA]]". [arXiv:2609.15066](https://arxiv.org/abs/2609.15066), cs.CL, 14 Sep 2026. The author list above is abbreviated.

## Summary

Koa is [[nemotron-3-super-nvidia-2026|Nemotron-3-Super-120B]] post-trained with [[grpo|GRPO]] directly from the released (already RL-post-trained) checkpoint. It uses no SFT stage and no customer data. The novel part is **spec-driven task construction**. Enterprise agents written in *Agent Script*, Salesforce's declarative language (router, subagents, typed actions, tool scopes), are compiled into a typed workflow graph that becomes an executable NeMo Gym environment. A simulation pipeline expands each spec into persona- and scenario-conditioned multi-turn sessions, some starting mid-dialogue. For public domains the workflow graph is synthesized directly. One frozen **Nemotron-3-Nano-30B helper** plays customer simulator, tool emulator *and* coverage judge. The reward is the fraction of numbered sub-questions judged resolved, where data-bound questions count only if grounded in a successful tool call. It is zeroed on any consecutive duplicate tool call. A deterministic sandbox environment uses binary state-equivalence instead.

## Key points

- **Headline (Table 1).** On [[tau-bench|τ²-Bench]] (task-weighted, 4 trials, GPT-4.1 user simulator) Koa scores **69.41** against 68.64 for the base, 54.48 for GPT-4.1, 74.00 for Claude Opus 4.8 and 83.99 for GPT-5.5. Per domain: Airline 62.0 (61.5), Retail 81.6 (79.9), Telecom 60.5 (60.5, unchanged). BFCL: **66.63** against 64.73 for the base. CRM Bench: **0.86** against 0.84, with function-call accuracy 0.71 → 0.77.
- **Where RL actually helps (Table 3).** BFCL *multi-turn* rises from 54.12 to **59.50** (Miss Function 42.5 → 54.0), while non-live AST drops from 86.17 to 84.85 and Memory from 52.90 to 52.04.
- **Reward engineering (App. I, Nano-30B proxy).** A dense partial-credit reward rose "only because partial credit inflates its own scalar (reward hacking)". Penalized-binary trended down, and additive or floored tool bonuses were flat or noisy. Only gated coverage stayed stable.
- **Scale-dependent stability (Fig. 4).** With the *same* final reward, Super-120B improves steadily to about 0.65 task resolution over about 200 steps. Nano-30B peaks near step 30 (about 0.47) and collapses to about 0.33.
- **Training.** GRPO with leave-one-out baseline, dynamic sampling, token-level truncated IS for the train/generation mismatch, and −1 advantages on malformed tool-call or thinking tokens. Five B200 nodes.

> [!warning] SFT beats RL on two of the three headline benchmarks
> In Appendix J the **SFT-only** checkpoint outscores the shipped RL checkpoint on τ²-Bench (**70.04 vs 69.41**) and CRM Bench (**0.88 vs 0.86**). RL wins only on BFCL overall (66.63 vs 65.03), and that win is driven by BFCL multi-turn. The abstract's "clearest gains on multi-turn tool use" is therefore true for BFCL multi-turn but not for τ²-Bench, which is also multi-turn and where RL's gain over the base is +0.77 and the smallest. Appendix C's SFT used a peak learning rate of 5×10⁻⁸ and moved validation loss only "marginally", so the SFT arm was deliberately weak. The authors scope this themselves: the base is already RL-trained.

## Concepts & entities

- [[nemotron-3-super-nvidia-2026]] — the base model. [[nvidia]] is the training partner.
- [[grpo]] · [[agentic-rl]] · [[reinforcement-learning]] — the recipe.
- [[tau-bench]] · [[function-calling]] · [[tool-use]] — evaluations and capability.
- [[llm-as-a-judge]] — the coverage judge is also the user simulator and the tool emulator, and it comes from a smaller model family.
- [[reward-hacking]] — the partial-credit reward failure.
- [[supervised-fine-tuning]] — the scoped SFT-vs-RL comparison.
- [[pivotrl-high-accuracy-agentic-post-training-yi-2026]] — the NVIDIA agentic post-training work it cites.

## What it does not show

- The base-to-Koa gains are small (+0.77 τ², +1.9 BFCL, +0.02 CRM), each from one run, with no variance reported.
- CRM Bench is Salesforce's own, and single-turn. No held-out enterprise spec is evaluated, so generalisation beyond the trained environments is untested.
- The training judge is never validated against humans. Offline evaluation does use a separate judge.

## References

- [arXiv:2609.15066](https://arxiv.org/abs/2609.15066)
