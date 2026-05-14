---
title: Instruction Tuning
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - wei-flan-2021.md
  - ouyang-instructgpt-2022.md
builds-on: "[[fine-tuning]]"
status: complete
importance: high
---

# Instruction Tuning

Instruction tuning is fine-tuning a pre-trained LM on a collection of `(instruction, input, output)` tuples drawn from many NLP tasks, with each task expressed via **natural-language instructions**. The trained model generalizes to **unseen instruction types** at inference time — turning a raw LM into a general-purpose instruction-following assistant.

Introduced concurrently in two papers:

- **[[flan-wei-2021|FLAN]]** (Wei et al., Sep 2021): fine-tune on 60+ NLP datasets verbalized as instructions; show generalization to held-out task types. Pure SFT, no preferences.
- **[[instructgpt-ouyang-2022|InstructGPT]]** (Ouyang et al., Mar 2022): SFT on labeler-written demonstrations + [[rlhf|RLHF]] on labeler-ranked responses. Goes beyond plain SFT to optimize on preferences.

Both papers point at the same insight: **pre-trained LMs need to be exposed to diverse natural-language task descriptions** to become useful assistants. The difference is the data source — public NLP datasets verbalized as instructions (FLAN) vs. labeler-written demonstrations on real-API-like prompts (InstructGPT) — and the algorithm — pure SFT (FLAN) vs. SFT+RLHF (InstructGPT).

## The dominant recipe (2022–)

The InstructGPT recipe became the de-facto post-training pipeline for general-purpose assistants:

1. **Pre-train** a large LM on web text ([[autoregressive-language-modeling|autoregressive LM]]).
2. **SFT**: fine-tune on (instruction, response) demonstrations.
3. **Preferences**: collect (preferred, dispreferred) pairs over multiple responses.
4. **Optimize against preferences**: via [[rlhf|RLHF]]/PPO, or [[dpo|DPO]] and variants.

Almost every deployed LLM (ChatGPT, Claude, Gemini, Llama-Chat, Mistral-Instruct, Qwen-Chat) has gone through some version of this pipeline.

## Variants and successors

- **Self-Instruct** (Wang et al. 2022): synthesize instruction-response pairs using the LLM itself.
- **Alpaca** (Taori et al. 2023): apply Self-Instruct to LLaMA — early example of a community-fine-tuned assistant.
- **Vicuna**, **WizardLM**, **OpenChat**, etc. — variations on the same theme using ShareGPT, Evol-Instruct, or other data sources.
- **Flan-T5**, **Flan-PaLM** (Chung et al. 2022): scale up the FLAN dataset (1.8k+ tasks) and apply to T5/PaLM.

## Why it works

- **Pre-training learns *capabilities***; the model just doesn't know it should *follow instructions* as opposed to extending random web text.
- **Instruction tuning teaches the format and intent**: a small dose of supervised demonstrations is enough to dramatically shift how the model presents itself.
- **Emergent at scale**: instruction tuning consistently *hurts* small models (<~10B) and *helps* large ones — another emergence story.

## References

- [[flan-wei-2021]]
- [[instructgpt-ouyang-2022]]
