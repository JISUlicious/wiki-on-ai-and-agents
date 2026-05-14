---
title: "Toolformer: Language Models Can Teach Themselves to Use Tools"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - schick-toolformer-2023.md
arxiv_id: "2302.04761"
venue: NeurIPS 2023
authors:
  - Timo Schick
  - Jane Dwivedi-Yu
  - Roberto Dessì
  - Roberta Raileanu
  - Maria Lomeli
  - "[[luke-zettlemoyer]]"
  - Nicola Cancedda
  - Thomas Scialom
first_author: Timo Schick
introduces:
  - "[[toolformer]]"
year: 2023
tags:
  - 2023
status: complete
importance: high
---

# Toolformer: Language Models Can Teach Themselves to Use Tools

**Source**: `sources/schick-toolformer-2023.md` (extracted from `schick-toolformer-2023.pdf`)
**arXiv**: [2302.04761](https://arxiv.org/abs/2302.04761)
**Venue**: NeurIPS 2023
**Authors**: Timo Schick (first), Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, [[luke-zettlemoyer]], Nicola Cancedda, Thomas Scialom ([[fair|Meta AI Research]])

## Summary

Toolformer makes **tool use a pre-training-style problem** rather than a per-tool prompt-engineering one. The system **self-supervises its own training data** for tool use: given a small handful of demonstrations per API, the model generates many candidate "tool call" annotations on a large unlabeled corpus, executes each call, and keeps only those that **reduce the model's perplexity** on the subsequent tokens — i.e., calls where having the tool's output measurably helped predict the next words.

The result is a fine-tuned LM that learns to autonomously decide:
- **Which API to call** (out of a small toolbox: QA system, calculator, Wikipedia search, machine translation, calendar).
- **When to call it** (inserted mid-sentence at the most useful point).
- **What arguments to pass** (e.g., the exact query string).
- **How to incorporate the result** into subsequent token prediction.

In-text examples (from the paper):
- "The New England Journal of Medicine is a registered trademark of [QA('Who is the publisher of NEJM?') → Massachusetts Medical Society] the MMS."
- "Out of 1400 participants, 400 (or [Calculator(400 / 1400) → 0.29] 29%) passed the test."

The annotations are inline `[ToolName(args) → result]` tokens that the model both produces and consumes.

Toolformer's contribution is methodological: tool use can be **bootstrapped self-supervised** rather than taught via prompt engineering or labeled demonstrations. The model trains itself on what helps it predict text better. This generalized the "LLMs can use tools" pattern from the per-tool, per-prompt approach of [[react|ReAct]] / [[webgpt-nakano-2021|WebGPT]] to a more scalable training-data-generation pipeline.

The downside (and reason the paper is less directly applied than ReAct) is that the tool-use behavior is baked into the model's weights and tied to a fixed toolbox. Modern production tool use uses **structured function-calling APIs** ([[react|ReAct]]-style with JSON arguments, OpenAI function calling, Anthropic tool use), which keeps the toolbox swappable at inference time. But Toolformer's self-supervision insight has reappeared in many downstream training pipelines for tool-using and agentic LLMs.

## Key Points

- **Self-supervised training-data generation**: given a few demonstrations of an API, the LM samples many candidate annotations on raw text, runs the API, and **filters by perplexity reduction** on subsequent tokens.
- **Inline tool-call format**: `[API(args) → result]` tokens embedded in the text stream. The model treats tool calls as a special part of next-token prediction.
- **Fixed toolbox**: calculator, QA system, Wikipedia search, machine translation, calendar. (Compare to ReAct's external-action-anything approach.)
- **Performance**: Toolformer-trained 6.7B GPT-J matches or exceeds much larger LMs on tasks requiring the tools (LAMA factual recall, math word problems, multilingual QA, temporal reasoning).
- **No instruction tuning, no RLHF**: pure self-supervised fine-tuning on the bootstrapped annotations.

## Entities Mentioned

- Timo Schick — first author
- [[luke-zettlemoyer]] — senior author; long-career NLP researcher (UW + Meta AI)
- [[fair]] (Meta AI Research) — author affiliation

## Concepts Discussed

- [[toolformer]] — the method
- [[tool-use]] — the broader paradigm
- Self-supervised data generation

## Notable Quotes

> "We show that LMs can teach themselves to use external tools via simple APIs and achieve the best of both worlds. We introduce Toolformer, a model trained to decide which APIs to call, when to call them, what arguments to pass, and how to best incorporate the results into future token prediction. This is done in a self-supervised way, requiring nothing more than a handful of demonstrations for each API." — Abstract

## References

_Original source: `sources/schick-toolformer-2023.md`_
