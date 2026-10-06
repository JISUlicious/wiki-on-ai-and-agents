---
title: "Prompt Engineering (Lil'Log)"
type: source
created: 2026-10-07
updated: 2026-10-07
sources:
  - prompt-engineering-weng-2023.md
authors:
  - "[[lilian-weng]]"
year: 2023
venue: "Lil'Log"
url: https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/
tags:
  - 2023
  - blog
status: complete
importance: medium
---

# Prompt Engineering (Lil'Log)

**Source**: `sources/prompt-engineering-weng-2023.md` ([lilianweng.github.io](https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/))
**Author**: [[lilian-weng]]
**Date**: **2023-03-15**, the day after GPT-4's release

## Summary

A survey of [[prompt-engineering|prompt engineering]] for autoregressive LMs. Weng defines it as "also known as **In-Context Prompting**… methods for how to communicate with LLM to steer its behavior for desired outcomes without updating the model weights." She calls it "an empirical science" whose effects "vary a lot among models". Its core goal is "**alignment and model steerability**".

Her "spicy take": many prompt-engineering papers "are not worthy 8 pages long" because the trick fits in a sentence, and the field needs shared benchmark infrastructure instead.

## Contents

- **Basic prompting:**
  - zero-shot and few-shot;
  - tips on selecting examples (semantic similarity, diversity) and ordering them;
  - few-shot biases: majority label, recency, common tokens. Calibration helps.
- **Instruction prompting:** instruction-tuned models (InstructGPT, FLAN), and in-context instruction learning.
- **[[self-consistency|Self-consistency]]** sampling.
- **[[chain-of-thought-prompting|Chain-of-thought]]:** few-shot and zero-shot ("Let's think step by step"); extensions such as Self-Ask, IRCoT and [[tree-of-thoughts|Tree of Thoughts]].
- **Automatic prompt design:** soft prompts, AutoPrompt, APE (search over candidate instructions), augmented CoT.
- **Augmented language models:**
  - retrieval (internet search, dense retrieval);
  - programming languages (PAL, PoT);
  - external APIs ([[toolformer|Toolformer]], TALM).

## Why it matters for the wiki

It is the companion to [[llm-powered-autonomous-agents-weng-2023|her agents post]] three months later: prompting techniques first, then agents built from them. Its "augmented language models" section is the bridge from prompting to [[tool-use]].

As a snapshot it is a practitioner's 2023 counterpart to the 2021 academic formalization ([[pre-train-prompt-predict-liu-2021]]), before [[context-engineering]] (2025) widened the frame.

## Concepts & entities

[[prompt-engineering]] · [[in-context-learning]] · [[chain-of-thought-prompting]] · [[self-consistency]] · [[tree-of-thoughts]] · [[tool-use]] · [[retrieval-augmented-generation]] · [[lilian-weng]]
