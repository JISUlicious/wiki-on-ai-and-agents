---
title: Zero-Shot Chain-of-Thought
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - kojima-zeroshot-cot-2022.md
builds-on: "[[chain-of-thought-prompting]]"
status: complete
importance: medium
---

# Zero-Shot Chain-of-Thought

A degenerate-but-effective form of [[chain-of-thought-prompting|chain-of-thought prompting]]: instead of providing few-shot exemplars that demonstrate step-by-step reasoning, simply append a single **trigger phrase** to the prompt — typically `"Let's think step by step."` — and let the LLM produce its own chain of reasoning. Introduced by [[zero-shot-cot-kojima-2022|Kojima et al. (2022)]].

## The prompt template

```
Q: {question}
A: Let's think step by step.

→ LLM produces a chain of reasoning →

Therefore, the answer (arabic numerals) is

→ LLM produces the final answer →
```

The second-stage "answer extraction" prompt is needed because the chain-of-thought output doesn't always end with a clean, parseable answer.

## Why it matters

Few-shot CoT requires curating exemplars per task. Zero-shot CoT is a *single trigger phrase* that works across diverse reasoning tasks. This makes it dramatically more practical for production use:

- **No per-task prompt engineering** — just append the magic phrase.
- **Same template works on arithmetic, commonsense, symbolic, and logical reasoning.**
- **Almost no token cost** vs. few-shot exemplars that can consume hundreds of tokens.

By 2023–2024, most chat LLMs had zero-shot CoT-style reasoning baked into their default behavior — they spontaneously break down problems step-by-step without being asked, thanks to instruction-tuning data that demonstrates this.

## Limitations

- **Emergent only at scale** — small models (<~10B) often hurt by the prompt.
- **Faithfulness**: same caveat as few-shot CoT. The chain looks like the model's reasoning but isn't always a faithful trace.
- **Brittle phrasing**: subtle wording changes ("step by step" vs. "carefully") can produce different results.

## References

- [[zero-shot-cot-kojima-2022]]
