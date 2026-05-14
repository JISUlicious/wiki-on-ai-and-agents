---
title: InstructGPT
type: entity
created: 2026-05-14
updated: 2026-05-14
sources:
  - ouyang-instructgpt-2022.md
family: gpt
predecessor: "[[gpt-3]]"
successors:
  - "[[gpt-4]]"
parameters: "1.3B / 6B / 175B"
release_date: 2022-01-27
status: complete
importance: high
tags:
  - model
---

# InstructGPT

InstructGPT is [[openai|OpenAI]]'s instruction-following variant of [[gpt-3]], introduced in [[instructgpt-ouyang-2022|"Training language models to follow instructions with human feedback"]] (Ouyang et al., 2022). Released as `text-davinci-001` (and follow-ups) on the OpenAI API in January 2022 — predating ChatGPT by ~10 months.

## The recipe

Three stages applied to a GPT-3 base model:

1. **SFT (Supervised Fine-Tuning)**: ~13k labeler-written `(prompt, response)` demonstrations on a wide range of tasks. Standard supervised cross-entropy.
2. **RM (Reward Model)**: ~33k prompts × ~4–9 ranked responses. Train a separate fine-tuned LM with a scalar head to predict labeler rankings via pairwise Bradley-Terry loss.
3. **PPO**: fine-tune the SFT policy to maximize the RM's score, with KL regularization toward the SFT initialization.

## Significance

InstructGPT is the **direct lineal ancestor of ChatGPT** and every general-purpose chat LLM since. The November 2022 ChatGPT release was a slightly improved variant of the same pipeline applied to GPT-3.5, with dialogue-specific training data added. The 1.3B-vs-175B preference result — labelers preferring the 100×-smaller-but-aligned model — became the canonical justification for spending compute on post-training alignment rather than just on scaling pre-training.

The recipe also became the template for every other lab's assistant model: Claude (using Constitutional AI variations), Gemini, Llama-Chat, Mistral-Instruct, Qwen-Chat — all follow some version of SFT → preferences → policy optimization.

## Successors

- **GPT-3.5** (text-davinci-002, text-davinci-003): incremental iterations on the same pipeline.
- **ChatGPT** (Nov 2022): GPT-3.5 + dialogue-specific data, deployed as a consumer chatbot. Made LLMs mainstream.
- **GPT-4** (Mar 2023) — same general post-training shape, undisclosed details.

## References

- [[instructgpt-ouyang-2022]]
