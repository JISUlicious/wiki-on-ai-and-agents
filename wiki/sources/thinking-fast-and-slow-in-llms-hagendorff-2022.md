---
title: "Thinking Fast and Slow in Large Language Models"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - thinking-fast-and-slow-in-llms-hagendorff-2022.md
arxiv_id: "2212.05206"
venue: "arXiv (published version in Nature Computational Science, 2023, per general knowledge)"
authors:
  - Thilo Hagendorff
  - Sarah Fabi
  - Michal Kosinski
first_author: Thilo Hagendorff
year: 2022
tags:
  - 2022
  - paper
status: complete
importance: medium
---

# Thinking Fast and Slow in Large Language Models

**Source**: `sources/thinking-fast-and-slow-in-llms-hagendorff-2022.md` ([arXiv:2212.05206](https://arxiv.org/abs/2212.05206); v1 2022-12-10)
**Authors**: Thilo Hagendorff, Sarah Fabi, Michal Kosinski

A related arXiv entry, 2306.07622 (Hagendorff & Fabi, June 2023), is titled "Human-Like Intuitive Behavior and Reasoning Biases Emerged in Language Models — and Disappeared in GPT-4".

## Summary

The paper runs the classic human **System 1 traps** on LLMs:
- the **Cognitive Reflection Test (CRT)**, e.g. "A bat and a ball cost $1.10…";
- **semantic illusions**, which need no arithmetic;
- newly written variants, to avoid memorized items.

## Findings

- **Earlier models answer intuitively and wrongly, like humans.** GPT-3-davinci-003 gave intuitive (wrong) answers on about **80%** of CRT tasks.
- **Later models stop.** ChatGPT-3.5 gave intuitive answers about **15%** of the time and **ChatGPT-4 0%**. The authors call them "hyperrational".
- **Chain-of-thought turns the intuitive model deliberate.** Instructing davinci-003 to reason step by step significantly improved its CRT performance.
- **Method claim:** "investigating LLMs with methods from psychology has the potential to reveal otherwise unknown emergent traits."

## Why it matters

- **The cleanest empirical bridge between Kahneman's tests and LLMs.**
  - A next-token answer with no visible reasoning behaves like **System 1**, including its characteristic errors.
  - Explicit reasoning, through chain-of-thought or through training (ChatGPT and GPT-4), behaves like **System 2**.
- **It also shows the mapping is not fixed by architecture.** The same autoregressive mechanism moved from "intuitive" to "hyperrational" across model generations.

## Concepts & entities

[[dual-process-theory]] · [[chain-of-thought-prompting]] · [[gpt-4]] · [[system-one-models-and-dual-process-thinking]]
