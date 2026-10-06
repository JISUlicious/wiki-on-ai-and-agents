---
title: Prompt Engineering
type: concept
created: 2026-10-06
updated: 2026-10-06
sources:
  - pre-train-prompt-predict-liu-2021.md
  - effective-context-engineering-anthropic-2025.md
  - building-effective-agents-anthropic-2024.md
status: complete
importance: high
---

# Prompt Engineering

Designing the text input (template, instructions, examples) so that a pre-trained model performs a task without changing its weights.

**Formal definition** ([[pre-train-prompt-predict-liu-2021|Liu et al., Jul 2021]]): "the process of creating a prompting function f_prompt(x) that results in the most effective performance on the downstream task."

## Timeline

| When | Milestone |
|---|---|
| 2019–2020 | Cloze-style prompting of masked LMs (LAMA, PET; see [[cloze-scoring]]) |
| May 2020 | [[gpt-3\|GPT-3]]: few-shot [[in-context-learning]] makes prompting the main interface; the informal term spreads |
| **Jul 2021** | **[[pre-train-prompt-predict-liu-2021\|Pre-train, Prompt, and Predict]]**: names "prompt-based learning" as NLP's fourth paradigm and formally defines prompt engineering, answer engineering, and manual vs automated, discrete vs continuous prompts |
| Oct 2021 | [[prompt-chaining]] ([[ai-chains-wu-2021\|AI Chains]]): multiple prompts composed |
| Jan 2022 | [[chain-of-thought-prompting\|Chain-of-thought]]: prompting for reasoning |
| 2022–2023 | Practitioner era: prompt guides; "prompt engineer" as a job title |
| Dec 2024 | Anthropic: "prompt engineering your tools", tool descriptions as prompts ([[building-effective-agents-anthropic-2024]]) |
| Sep 2025 | [[context-engineering]] framed as its "natural progression" |

> [!note] Coinage
> The wiki has no primary source for the *first* use of the phrase. Liu et al. 2021 is the formalization point, not the coinage.

## What it does and doesn't cover

Prompt engineering optimizes a **single, mostly static** input, especially the system prompt. Agents add tool results, history, retrieved documents, skills and memory on every turn. Managing all of that is [[context-engineering]].

The agent-skills survey ([[agent-skills-survey-xu-2026]]) gives the progression as: prompt engineering (2022–23) → tool use and [[function-calling]] (2023–24) → skill engineering (2025–).

## Related

[[in-context-learning]] · [[cloze-scoring]] · [[chain-of-thought-prompting]] · [[prompt-chaining]] · [[context-engineering]] · [[prompt-injection]], the adversarial counterpart.

## References

- [[pre-train-prompt-predict-liu-2021]]
- [[effective-context-engineering-anthropic-2025]]
- [[building-effective-agents-anthropic-2024]]
