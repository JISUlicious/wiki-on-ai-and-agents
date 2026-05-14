---
title: "ReAct: Synergizing Reasoning and Acting in Language Models"
type: source
created: 2026-05-13
updated: 2026-05-13
sources:
  - react-yao-2022.md
arxiv_id: "2210.03629"
venue: ICLR 2023
authors:
  - "[[shunyu-yao]]"
  - Jeffrey Zhao
  - Dian Yu
  - Nan Du
  - Izhak Shafran
  - Karthik Narasimhan
  - Yuan Cao
year: 2022
tags:
  - 2022
status: complete
importance: high
---

# ReAct: Synergizing Reasoning and Acting in Language Models

**Source**: `sources/react-yao-2022.md` (extracted from `ReAct Yao 2022 2210.03629.pdf`)
**arXiv**: [2210.03629](https://arxiv.org/abs/2210.03629)
**Venue**: ICLR 2023
**Authors**: [[shunyu-yao]] (first), Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan ([[princeton-university|Princeton]]; also a co-author on [[gpt-1-radford-2018|GPT-1]]), Yuan Cao ([[google-research|Google Brain]])

## Summary

ReAct introduces a prompting framework that **interleaves natural-language reasoning with action execution** in a single LLM trajectory. The model alternates among three tokens types per step: `Thought:` (a free-form reasoning step), `Act:` (an action to take, e.g., search a knowledge base or move in an environment), and `Obs:` (the observation returned by the environment after the action). Repeat until the task is solved.

ReAct is positioned as a fix for the failure modes of pure [[chain-of-thought-prompting|chain-of-thought]] (hallucinated facts, runaway reasoning errors) by **grounding** reasoning in real-world observations gathered via actions. Conversely, it improves pure action-only agents by letting the LLM **plan, track progress, and handle exceptions** in natural language between actions.

Empirically, ReAct:

- **Knowledge-intensive QA / fact verification** (HotpotQA, FEVER, with a simple Wikipedia search/lookup API): outperforms CoT-only by reducing hallucination, and outperforms Act-only by enabling better plan adjustment.
- **Interactive decision-making** (ALFWorld text-based home tasks; WebShop online-shopping environment): outperforms imitation learning and RL baselines by **+34%** absolute success on ALFWorld and **+10%** on WebShop, using only **1–2 in-context examples**.

ReAct is a foundational paper for the **LLM-agent paradigm**: the framework of LLM + tools + observation loop that underlies AutoGPT, LangChain agents, OpenAI's function-calling-based agents, Claude tool-use, and almost every modern production agent system.

## Key Points

- **Format**: `Thought → Act → Obs → Thought → Act → ... → Finish[answer]`. All three streams are tokens in the same LLM context.
- **Actions** can be:
  - **Information retrieval** (search Wikipedia, lookup keyword, finish with answer) for QA tasks.
  - **Environment commands** (go to <location>, take <object>, open <thing>, put <object> in/on <receptacle>) for embodied tasks.
- **Grounds CoT**: each thought can be informed by the latest observation, rather than the model spiraling on its own internal state.
- **Plans for action**: each thought can decompose, re-plan, or adjust based on results — a major gain over reflex-style "Act-only" agents.
- **Few-shot only**: ReAct works at few-shot scale (1–6 demonstrations); no fine-tuning needed for many tasks.
- **Models tested**: PaLM-540B (primary), GPT-3, smaller models for ablations.

## Entities Mentioned

- [[shunyu-yao]] — first author, leading author on subsequent agent / tree-of-thoughts work
- [[princeton-university]]
- [[google-research]] — Brain Team
- [[palm]] — primary base LLM in experiments

## Concepts Discussed

- [[react]] — the framework
- [[tool-use]] — the broader concept of LLM-as-controller-of-external-tools
- [[llm-agent]] — the broader paradigm
- [[chain-of-thought-prompting]] — what ReAct grounds and improves
- [[hallucination]] — addressed by retrieval/action grounding
- [[in-context-learning]] — only mechanism used (no fine-tuning)

## Notable Quotes

> "In this paper, we explore the use of LLMs to generate both reasoning traces and task-specific actions in an interleaved manner, allowing for greater synergy between the two: reasoning traces help the model induce, track, and update action plans as well as handle exceptions, while actions allow it to interface with and gather additional information from external sources such as knowledge bases or environments." — Abstract

> "On two interactive decision making benchmarks (ALFWorld and WebShop), ReAct outperforms imitation and reinforcement learning methods by an absolute success rate of 34% and 10% respectively, while being prompted with only one or two in-context examples." — Abstract

## References

_Original source: `sources/react-yao-2022.md`_
_Project page: https://react-lm.github.io/_
