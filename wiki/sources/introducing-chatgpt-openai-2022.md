---
title: "ChatGPT: Optimizing Language Models for Dialogue"
type: source
created: 2026-10-07
updated: 2026-10-07
sources:
  - introducing-chatgpt-openai-2022.md
authors:
  - "[[openai]]"
introduces:
  - "[[chatgpt]]"
year: 2022
venue: OpenAI blog
url: https://openai.com/blog/chatgpt/
tags:
  - 2022
  - blog
status: complete
importance: high
---

# ChatGPT: Optimizing Language Models for Dialogue

**Source**: `sources/introducing-chatgpt-openai-2022.md`, captured from the Wayback snapshot of 2022-12-01 (openai.com blocks automated fetches)
**Author**: [[openai]]
**Date**: launched November 2022. The post isn't dated in the capture. The day, 2022-11-30, is general knowledge, consistent with the 2022-12-01 snapshot.

## Summary

OpenAI introduces **[[chatgpt|ChatGPT]]**, "a sibling model to [[instructgpt|InstructGPT]]". The dialogue format lets it:
- "answer followup questions";
- "admit its mistakes";
- "challenge incorrect premises";
- "reject inappropriate requests".

It launched as a **free research preview** at chat.openai.com, with the stated aim of collecting user feedback.

## Method

- **Same recipe as InstructGPT**, "with slight differences in the data collection setup" ([[rlhf|RLHF]]):
  1. **SFT:** human AI trainers wrote conversations playing *both* the user and the assistant, with model-written suggestions to help.
  2. **Reward model:** trainers ranked several sampled alternatives to model messages taken from their own chats with the chatbot.
  3. **[[ppo|PPO]]** fine-tuning against the reward model, over several iterations.
- **Base model:** fine-tuned from a model in the **GPT-3.5 series**, which "finished training in early 2022", on Azure AI supercomputing infrastructure.

## Limitations the post lists

- **Plausible-sounding but incorrect answers.** Fixing this is hard: RL has no source of truth, and training for caution makes the model refuse questions it could answer.
- **Sensitive to small rephrasings** of the input.
- **Excessively verbose.** Trainers preferred longer answers, plus "well-known over-optimization issues". This is the *verbosity bias* that [[judging-llm-as-a-judge-zheng-2023|LLM judges]] later inherited.
- **Guesses** instead of asking clarifying questions.
- **Imperfect refusals.** The Moderation API is used to warn about or block unsafe content.

## Lineage

| | |
|---|---|
| Built on | [[gpt-3]] → GPT-3.5 · [[instructgpt-ouyang-2022\|InstructGPT]] (SFT → RM → PPO) · [[webgpt\|WebGPT]] and Codex deployment lessons ("iterative deployment") |
| Followed by | [[gpt-4]] (Mar 2023) · [[chatgpt-plugins]] (2023-03-23) · [[function-calling]] (2023-06-13) · GPTs (2023-11-06) |

## Concepts & entities

[[chatgpt]] · [[instructgpt]] · [[rlhf]] · [[ppo]] · [[reward-modeling]] · [[openai]]
