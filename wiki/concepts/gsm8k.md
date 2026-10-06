---
title: GSM8K
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - training-verifiers-math-word-problems-cobbe-2021.md
status: draft
importance: medium
---

# GSM8K

**Grade School Math 8K**: 8.5K linguistically diverse grade-school math word problems with natural-language step-by-step solutions. The solutions carry `<<…>>` calculator annotations. Introduced by [[openai|OpenAI]] in [[training-verifiers-math-word-problems-cobbe-2021|Cobbe et al. (Oct 2021)]] together with trained [[verifier]]s.

## Why it mattered

- The problems need only elementary arithmetic, but they take several steps. In 2021 even 175B GPT-3 did poorly on them, which made GSM8K a clean probe of multi-step reasoning.
- It became the standard reasoning benchmark of 2022–2024:
  - [[chain-of-thought-prompting|chain-of-thought]] (2022) and [[self-consistency]] reported their headline gains on it;
  - verifier, best-of-N and process-supervision work all used it.
- It is now **saturated** for frontier models, and contamination is a known concern ([[benchmark-contamination]]). Harder successors include MATH, [[aime|AIME]] and [[gpqa|GPQA]].

## Related

[[verifier]] · [[chain-of-thought-prompting]] · [[self-consistency]] · [[benchmark]] · [[process-reward-model]]

## References

- [[training-verifiers-math-word-problems-cobbe-2021]]
