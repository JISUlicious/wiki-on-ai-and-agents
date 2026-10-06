---
title: "AI Chains: Transparent and Controllable Human-AI Interaction by Chaining Large Language Model Prompts"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - ai-chains-wu-2021.md
arxiv_id: "2110.01691"
venue: "CHI 2022"
authors:
  - Tongshuang Wu
  - Michael Terry
  - Carrie J. Cai
first_author: Tongshuang Wu
introduces:
  - "[[prompt-chaining]]"
year: 2021
tags:
  - 2021
  - paper
status: complete
importance: high
---

# AI Chains

**Source**: `sources/ai-chains-wu-2021.md` ([arXiv:2110.01691](https://arxiv.org/abs/2110.01691))
**Authors**: Tongshuang Wu ([[university-of-washington|University of Washington]]), Michael Terry and Carrie J. Cai ([[google-research|Google Research]])
**Date**: October 2021 (arXiv v1, from the ID); v3 dated 2022-03-17. Published at CHI 2022 (venue from general knowledge).

## Summary

A human-computer interaction paper that introduces **Chaining**: "the output of one step becomes the input for the next, thus aggregating the gains per step." It is the earliest primary source in this wiki for **[[prompt-chaining]]**.

GPT-3-era models were broad but opaque and hard to steer on complex tasks. The authors instead break a task into LLM steps, each its own prompt. The steps use a small set of **primitive operations**, grouped as:
- validating and categorizing input (e.g. classification);
- gathering more information (e.g. factual query, ideation);
- re-organizing input (e.g. split points, compose points, rewriting).

The worked example rewrites peer feedback in three steps: *split points* extracts each problem, *ideation* brainstorms a suggestion for each, and *compose points* writes the final paragraph.

## Key findings (20-person user study)

- Chaining improved the quality of the results and significantly increased **transparency, controllability and sense of collaboration**, compared with a single-prompt baseline.
- Users found new ways to work:
  - they used subtasks to calibrate their expectations of the model;
  - they compared alternative strategies through parallel branches;
  - they **"unit-tested" individual steps** to debug unexpected output.
- Two case studies sketch further uses of chains.

## Lineage

- **Built on:** GPT-3 prompting and [[in-context-learning]].
- **Fed into:**
  - **LangChain** (repo created 2022-10-17), whose name and core abstraction is a chain of LLM calls. See [[langchain]].
  - The named workflow "prompt chaining", with programmatic *gates* between steps, in Anthropic's [[building-effective-agents-anthropic-2024|Building effective agents]] (Dec 2024).
  - [[dynamic-workflows]] (2026), where the model writes the chain itself.
- **What came later:** the paper is about human-designed decomposition for transparency. The agent era moved decomposition into the model ([[react|ReAct]], [[agentic-workflow-patterns|orchestrator-workers]]).

## Concepts & entities

[[prompt-chaining]] · [[agentic-workflow-patterns]] · [[in-context-learning]] · [[langchain]] · [[gpt-3]] · [[google-research]] · [[university-of-washington]]
