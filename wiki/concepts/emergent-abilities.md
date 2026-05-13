---
title: Emergent Abilities
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - chain-of-thought-wei-2022.md
status: draft
importance: medium
---

# Emergent Abilities

In the LLM literature, an **emergent ability** is a capability that is absent (at near-random performance) at small model scales and **abruptly appears** at some scale threshold. The term was popularized by Wei et al., "Emergent Abilities of Large Language Models" (2022, separate paper from [[chain-of-thought-wei-2022|the CoT paper]] but same first author).

Canonical examples cited:

- **3-digit arithmetic** — emerges around 13B parameters in [[gpt-3]].
- **[[chain-of-thought-prompting|Chain-of-thought reasoning]]** — emerges around 100B parameters; smaller models hurt by CoT prompts.
- **Multi-step word problems** (GSM8K, MultiArith).
- **Word-in-context evaluations** like the BIG-Bench "Word in Context" task.

## The controversy

Schaeffer, Miranda, and Koyejo ("Are Emergent Abilities of Large Language Models a Mirage?", NeurIPS 2023) argued that apparent emergence is often an **artifact of the metric** — accuracy or exact-match scores are discontinuous (e.g., a math problem either correct or wrong), while underlying probabilities improve smoothly with scale. Switching to smooth metrics (e.g., per-token log-likelihood of the correct answer) erases many "emergent jumps."

This doesn't fully eliminate the phenomenon — some capabilities do appear genuinely sharply — but it tempers the more dramatic emergence narratives.

## Significance

Emergent abilities matter because they imply that scaling is not just a matter of marginal improvement: at some scales, *qualitatively new* capabilities appear. This shapes both research prioritization (it's worth pushing to larger scales for capabilities you can't yet observe) and safety thinking (capabilities that will appear at the next scale-up may be unpredictable).

## References

- [[chain-of-thought-wei-2022]]
