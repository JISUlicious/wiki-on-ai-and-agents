---
title: François Chollet
type: entity
created: 2026-09-30
updated: 2026-09-30
sources:
  - arc-agi-3-arc-prize-foundation-2026.md
  - meta-n-recursive-self-improvement-emergent-depth-kim-2026.md
status: draft
importance: medium
tags:
  - person
---

# François Chollet

Software engineer and AI researcher, creator and project lead of the Keras deep-learning library since 2015, and co-founder of ARC Prize and of Ndea. In this wiki he matters as the designer of the **ARC-AGI** benchmark series, which measures intelligence as efficiency at acquiring new skills rather than as accumulated skill. The wiki has no `arc-agi` concept page yet; the benchmark is covered through [[arc-agi-3-arc-prize-foundation-2026]] and [[agent-benchmark]].

## The ARC-AGI line

- **ARC-AGI-1 (2019).** Introduced in his paper *On the Measure of Intelligence*, which defines intelligence as skill-acquisition efficiency and proposes the Abstraction and Reasoning Corpus. See [[skill-acquisition]].
- **ARC-AGI-2 (2025).** Chollet, Knoop, Kamradt, Landers and Pinkard. It keeps the static input-output grid format with harder tasks. [[meta-n-recursive-self-improvement-emergent-depth-kim-2026|Meta^n]] reports ARC-AGI-2 dev-set scores for a GPT-5.2 backbone in its Table 2.
- **ARC-AGI-3 (2026).** [[arc-agi-3-arc-prize-foundation-2026|ARC Prize Foundation]], with Chollet as co-author. It moves to interactive, turn-based environments with no instructions and only Core Knowledge priors. Humans solve 100%; frontier AI scored below 1% as of March 2026. Scoring is action efficiency against human baselines (RHAE).

## ARC-AGI-3 as a harness test bed

Because ARC-AGI-3 rewards exploration and world-modelling, it became the main place where the wiki's sources measure **harness effects**:

- [[nvidia-oo-agents-furgale-2026|NVIDIA OO-Agents]] reports RHAE 85.1% against 13.3% for the raw model, a 6.4× harness effect.
- [[prime-agent-self-improving-rlm-harness-karten-2026|Prime Agent]] reports 30% → 95.5% but assigns the baselines differently.
- [[pro-long-programmatic-memory-fox-2026|Pro-Long]] gains +18.0 pp with an append-only log.

These figures conflict over which raw model scored what. [[harness-vs-model-attribution]] tracks the dispute. See also [[agentic-harness-engineering]].

## Related

- [[arc-agi-3-arc-prize-foundation-2026]] · [[agent-benchmark]] · [[skill-acquisition]] · [[llm-agent]] · [[harness-vs-model-attribution]]

## References

- [fchollet.com](https://fchollet.com/) — Keras creator and project lead (2015–present), co-founder of ARC Prize and Ndea
- [On the Measure of Intelligence, arXiv:1911.01547](https://arxiv.org/abs/1911.01547) — sole author, November 2019; skill-acquisition efficiency and ARC
- [ARC-AGI-2, arXiv:2505.11831](https://arxiv.org/abs/2505.11831) — Chollet, Knoop, Kamradt, Landers, Pinkard; May 2025
- [[arc-agi-3-arc-prize-foundation-2026]] · [[meta-n-recursive-self-improvement-emergent-depth-kim-2026]]
