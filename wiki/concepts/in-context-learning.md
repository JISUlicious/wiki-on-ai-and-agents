---
title: In-Context Learning
type: concept
created: 2026-05-13
updated: 2026-05-13
sources:
  - gpt-3-brown-2020.md
status: complete
importance: high
---

# In-Context Learning

In-context learning (ICL) is the ability of a large language model to **learn or adapt to a new task at inference time, purely from the prompt** — without any gradient updates or fine-tuning. The model is conditioned on a task description and/or a small number of `(input, output)` demonstrations packed into its context window, and is then expected to perform the task on a new input by predicting the natural continuation.

The term was coined in [[gpt-3-brown-2020|GPT-3]] (Brown et al., 2020) and is the central organizing concept of the post-GPT-3 prompting era.

## The meta-learning frame

GPT-3 frames ICL as a two-loop process:

- **Outer loop**: pre-training — the model learns a broad set of skills and pattern-recognition abilities from web-scale text. Many implicit "tasks" are demonstrated in natural text.
- **Inner loop**: in-context learning — at inference time, the model uses the prompt's demonstrations to **identify which task it should perform** and **how the inputs map to outputs**, all within a single forward pass.

This is sometimes called "meta-learning" or "prompt-based learning". Crucially, in-context learning emerged from standard [[autoregressive-language-modeling|next-token prediction]] training — it was not an explicit training objective.

## Few-shot, one-shot, zero-shot

The number of demonstrations gives a small taxonomy:

- **[[zero-shot-learning|Zero-shot]]**: only an instruction in the prompt; no examples.
- **One-shot**: instruction + 1 example.
- **[[few-shot-learning|Few-shot]]**: instruction + `k` examples (typically 10–100 in GPT-3's evaluations).

## Why ICL improves with scale

[[gpt-3-brown-2020|GPT-3]] showed empirically that few-shot performance scales **faster** than zero-shot performance as the model grows. Roughly: larger models are better at "extracting" the implicit task structure from context, not just better at language modeling generally. Several follow-up theoretical works (Xie et al. 2021; Akyürek et al. 2022; Garg et al. 2022) explored ICL as implicit Bayesian inference or implicit gradient descent over linear models. The exact mechanism remains incompletely understood as of 2026.

## Practical consequences

- **Prompt engineering** became a discipline overnight.
- **Single model, many uses**: one base model serves arbitrary tasks via prompts; no per-task fine-tuning, no per-task model.
- **API ecosystem**: [[openai|OpenAI]]'s API launched (June 2020) on the strength of ICL — without it, exposing a single GPT-3 endpoint to thousands of customers with different needs would have been impractical.
- **Downstream techniques built on ICL**: [[chain-of-thought-prompting|chain-of-thought prompting]] ([[chain-of-thought-wei-2022|Wei et al. 2022]]), [[react|ReAct]], few-shot translation, code completion via Codex/Copilot.

## Limitations

- Context-window-bounded: only as much task data as fits in the prompt (a few thousand tokens for GPT-3, hundreds of thousands by 2024).
- Sensitive to prompt formatting, ordering of examples, and label distribution.
- Doesn't truly learn — the next prompt starts from scratch (no persistent state).
- Smaller models are weak in-context learners; ICL is fundamentally a scale phenomenon.

## References

- [[gpt-3-brown-2020]]
