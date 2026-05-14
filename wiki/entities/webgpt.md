---
title: WebGPT
type: entity
created: 2026-05-14
updated: 2026-05-14
sources:
  - nakano-webgpt-2021.md
family: gpt
predecessor: "[[gpt-3]]"
successors: []
parameters: "fine-tuned variants of GPT-3 (760M, 13B, 175B)"
release_date: 2021-12-17
status: complete
importance: medium
tags:
  - model
---

# WebGPT

WebGPT is the [[gpt-3]]-derived browser-using agent introduced by [[webgpt-nakano-2021|Nakano et al. (2021)]] at [[openai|OpenAI]]. Fine-tuned to operate a text-based web-browsing environment for long-form question answering with citation grounding.

The model emits text actions (`Search query`, `Click link`, `Find on page`, `Quote text`, etc.) that drive a custom browsing environment. It collects quoted passages from the pages it visits and produces a final answer that cites those quotes. The fine-tuning pipeline (behavior cloning → reward model → rejection sampling / RL) is structurally identical to [[instructgpt-ouyang-2022|InstructGPT]] from a few months later — both are early productions of the SFT + preferences post-training recipe.

WebGPT was never released as a public product on its own, but its DNA is in:

- ChatGPT's "Browse with Bing" / "Browse" mode (initially deprecated, later re-introduced).
- OpenAI's "Operator" (2024), an autonomous web-browsing agent.
- Anthropic's Computer Use (2024) and similar "browse and act" capabilities.
- Perplexity, Phind, and other search-augmented chat products.

The two enduring contributions: (1) demonstrating that GPT-3 could be fine-tuned to operate a tool with structured text actions, and (2) the citation-grounding training pattern, which made factual-accuracy human evaluation tractable.

## References

- [[webgpt-nakano-2021]]
