---
title: "WebGPT: Browser-assisted question-answering with human feedback"
type: source
created: 2026-05-14
updated: 2026-05-14
sources:
  - nakano-webgpt-2021.md
arxiv_id: "2112.09332"
venue: arxiv preprint
authors:
  - Reiichiro Nakano
  - Jacob Hilton
  - Suchir Balaji
  - Jeff Wu
  - Long Ouyang
  - John Schulman
first_author: Reiichiro Nakano
introduces:
  - "[[webgpt]]"
year: 2021
tags:
  - 2021
status: complete
importance: high
---

# WebGPT: Browser-assisted question-answering with human feedback

**Source**: `sources/nakano-webgpt-2021.md` (extracted from `nakano-webgpt-2021.pdf`)
**arXiv**: [2112.09332](https://arxiv.org/abs/2112.09332)
**Venue**: arxiv preprint (Dec 2021)
**Authors**: Reiichiro Nakano (first), Jacob Hilton, Suchir Balaji, Jeff Wu, Long Ouyang, John Schulman, and others ([[openai|OpenAI]])

## Summary

This paper fine-tunes [[gpt-3]] to **operate a web browser via text actions** for long-form question answering. The model is given a text-based browsing environment with commands like `Search`, `Click`, `Find`, `Quote`, `Back`, and `End`. Given a question, the model interacts with the browser — issuing search queries, clicking links, scrolling, quoting passages — and finally produces an answer with citations to the quoted material.

The training recipe combines several pieces (predating but conceptually similar to [[instructgpt-ouyang-2022|InstructGPT]]):

1. **Behavior cloning**: collect demonstrations of humans answering questions using the same browsing environment. Fine-tune GPT-3 to imitate those trajectories.
2. **Reward model**: collect pairwise comparisons of model answers; train an RM on the preferences.
3. **Rejection sampling against the RM** (best-of-N) at inference. Some experiments also use RL against the RM.

Trained on ELI5 (questions asked by Reddit users). The best model's answers are preferred by humans **56%** of the time over the demonstrators' answers and **69%** over the highest-voted Reddit answer.

WebGPT is the first **high-profile demonstration of a fine-tuned LLM operating a tool** (a web browser) with **citation grounding**. It is the direct ancestor of:

- **ChatGPT browsing** (and later "with Web" features in every chat product).
- **Computer-use agents** (Anthropic Computer Use, OpenAI Operator, etc.) — same idea, broader action space.
- **Search-augmented LLMs** generally — Bing chat, Perplexity, Google's AI overviews.

The citation-grounding insight is also significant: by requiring the model to *quote sources* while browsing, human evaluators can verify factual accuracy more easily, which makes the RLHF reward signal cleaner. This was a precursor to the broader "verifiable claims" alignment story.

## Key Points

- **Text-based browsing environment**: actions are Search query, Click link, Find on page, Quote text, Back, End. The model emits structured action strings.
- **Three-stage training**: behavior cloning → reward model → rejection sampling / RL. Same shape as [[instructgpt-ouyang-2022|InstructGPT]].
- **Citation grounding**: model must collect references (quoted passages) during browsing; these are presented alongside the final answer for human verification.
- **ELI5 task**: Reddit's r/explainlikeimfive — long-form questions calling for paragraph-length explanations.
- **Result**: 56% preferred over demonstrators, 69% over top-voted Reddit answers. First production-worthy answer-with-citations agent.
- **Architectural lineage**: the action-text interface is the proto-version of modern tool-use APIs (OpenAI function calling, Anthropic tool use, MCP). Trade text emission for structured JSON, and you have today's tool-using agents.

## Entities Mentioned

- Reiichiro Nakano — first author
- John Schulman — senior author (PPO co-creator); long-time OpenAI alignment-team lead
- Long Ouyang, Jeff Wu — also on [[instructgpt-ouyang-2022|InstructGPT]] paper
- [[openai]] — origin org
- [[webgpt]] — the model

## Concepts Discussed

- [[tool-use]] — WebGPT is the canonical early demonstration
- [[llm-agent]] — agent-style browse-and-answer
- [[rlhf]] — training pipeline
- [[hallucination]] — citation grounding mitigates

## Notable Quotes

> "We fine-tune GPT-3 to answer long-form questions using a text-based web-browsing environment, which allows the model to search and navigate the web. ... To make human evaluation of factual accuracy easier, models must collect references while browsing in support of their answers." — Abstract

> "This model's answers are preferred by humans 56% of the time to those of our human demonstrators, and 69% of the time to the highest-voted answer from Reddit." — Abstract

## References

_Original source: `sources/nakano-webgpt-2021.md`_
