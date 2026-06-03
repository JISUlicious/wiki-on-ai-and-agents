---
title: "AutoHarness: Improving LLM Agents by Automatically Synthesizing a Code Harness"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - autoharness-lou-2026.md
arxiv_id: "2603.03329"
authors:
  - Xinghua Lou
  - et al.
first_author: Xinghua Lou
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# AutoHarness: Improving LLM Agents by Automatically Synthesizing a Code Harness

[arXiv:2603.03329](https://arxiv.org/abs/2603.03329) — Xinghua Lou et al. (Google DeepMind), 2026. Surfaced via the alphaXiv / NLP-newsletter digest.

## Summary

LLM agents frequently attempt actions that the environment strictly prohibits — in the Kaggle GameArena chess competition, 78% of Gemini-2.5-Flash losses were illegal moves rather than strategic blunders. People typically fix this with hand-written "harnesses" that gate model outputs, but those are brittle and labor-intensive per game. AutoHarness instead has the LLM **synthesize its own code harness** through a few rounds of iterative code refinement using feedback from the environment, framing harness generation as a search over programs.

The search uses **Thompson-sampling-guided tree search** with the LLM as a mutation operator proposing code refinements, balancing exploration of new logic against exploitation of partially-working harnesses. The harness can be constrained (learn just an `is_legal_action()` conditioning function in a fixed rejection-sampling loop) or fully open (a **code-as-policy** setup where code proposes actions directly with no LLM at decision time). The synthesized harness prevents *all* illegal moves across 145 TextArena games, letting the smaller Gemini-2.5-Flash outperform the larger Gemini-2.5-Pro — and the pure code-policy beats both Gemini-2.5-Pro and GPT-5.2-High on average reward over 16 one-player games, more cheaply.

## Key points

- Reframes the agent as "LLM + harness"; here the LLM completes itself by coding its own harness — a learned rejection sampler whose acceptance condition is task-learned.
- Program search via Thompson-sampling tree search (online, multi-turn); heuristic value = legal-action success rate; LLM acts as code mutation operator.
- Two regimes: constrained (learn `is_legal_action()`) vs. open code-as-policy (no LLM calls at execution).
- Eliminates all illegal moves across 145 TextArena games (1P and 2P).
- Small model + synthesized harness beats larger model: Flash wins 9/16 (56.3%) 2P games vs. Gemini-2.5-Pro.
- Code-policy outperforms Gemini-2.5-Pro and GPT-5.2-High on average reward over 16 1P games — and is more cost effective.
- Positions code-as-harness as a scalable alternative to fine-tuning on trajectories or hand-coding harnesses.

## Concepts & entities

- [[code-as-harness]] — the paper is a direct, named instantiation of this framework.
- [[llm-agent]] — the agent = LLM + harness framing.
- [[tool-creation]] — agent authoring its own scaffolding code.
- [[self-improving-agent]] — iterative refinement against environment feedback.
- [[constrained-decoding]] — rejection-sampling harness enforces action legality.
- [[deepmind]] — author affiliation.

## References

- [[code-as-harness]] — core framework.
- [[code-as-policies-liang-2022]] — code-as-policy precursor (Liang et al.).
- Tang et al. 2024 — Thompson-sampling program search basis.
