---
title: ChatGPT
type: entity
created: 2026-10-06
updated: 2026-10-07
sources:
  - introducing-chatgpt-openai-2022.md
  - chatgpt-plugins-openai-2023.md
  - introducing-gpts-openai-2023.md
release_date: 2022-11-30
status: complete
importance: high
tags:
  - product
---

# ChatGPT

[[openai|OpenAI]]'s conversational assistant. It launched as a **free research preview** at chat.openai.com in late **November 2022**: "ChatGPT: Optimizing Language Models for Dialogue" ([[introducing-chatgpt-openai-2022]]). The day, 2022-11-30, is general knowledge; it is consistent with the 2022-12-01 archive capture.

ChatGPT is the product, distinct from the models behind it ([[gpt-family]]). It is the moment LLMs reached a mass audience.

## How the launch model was built

The launch post calls it "a sibling model to [[instructgpt|InstructGPT]]", fine-tuned from a **GPT-3.5-series** model that "finished training in early 2022", on Azure.

1. **SFT:** AI trainers wrote dialogues, playing both user and assistant.
2. **Reward model:** trainers ranked alternative completions sampled from their own chats.
3. **[[ppo|PPO]]**, repeated over several iterations ([[rlhf]]).

**Stated limitations at launch:**
- plausible but wrong answers, because RL has no source of truth;
- sensitivity to rephrasing;
- verbosity, because trainers preferred longer answers, plus over-optimization;
- guessing instead of asking clarifying questions;
- imperfect refusals, backed by the Moderation API.

## Timeline (from ingested sources)

| Date | Milestone |
|---|---|
| 2022-11 | Launch (research preview) |
| 2023-03-14 | [[gpt-4\|GPT-4]] |
| 2023-03-23 | [[chatgpt-plugins\|Plugins]] alpha: browsing, code interpreter, third-party tools |
| 2023-06-13 | [[function-calling\|Function calling]] in the API |
| 2023-07 | Custom Instructions (mentioned in the GPTs post) |
| 2023-11-06 | **GPTs**: custom ChatGPTs from instructions, knowledge and tools. **Actions** replace plugins. GPT Store announced. One model picker covers DALL·E, browsing and data analysis. ([[introducing-gpts-openai-2023]]) |
| 2024-03-19 / 04-09 | Plugins wound down ([[chatgpt-plugins-wind-down-2024]]) |
| later | Operator and computer use ([[computer-use-agent]]) |

## Framing in the wiki

- **Model vs product:** ChatGPT is a product layer: a chat UI, memory and instructions, tools, and safety systems around a changing series of models. It is a consumer instance of the model + harness split ([[agent-three-layer-model]]).
- **Agent typology:** the [[intelligent-agent-wikipedia|Wikipedia article on intelligent agents]] cites ChatGPT as a *goal-based agent*.
- **Lineage:** [[gpt-3]] → [[instructgpt]] → **ChatGPT** → plugins and function calling → GPTs → agents.

## References

- [[introducing-chatgpt-openai-2022]]
- [[chatgpt-plugins-openai-2023]]
- [[introducing-gpts-openai-2023]]
