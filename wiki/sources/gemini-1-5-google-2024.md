---
title: "Our next-generation model: Gemini 1.5"
type: source
created: 2026-10-06
updated: 2026-10-06
sources:
  - gemini-1-5-google-2024.md
authors:
  - Sundar Pichai
  - Demis Hassabis
introduces:
  - "[[gemini-1-5]]"
year: 2024
venue: Google blog (The Keyword)
url: https://blog.google/technology/ai/google-gemini-next-generation-model-february-2024/
tags:
  - 2024
  - blog
status: complete
importance: medium
---

# Our next-generation model: Gemini 1.5

**Source**: `sources/gemini-1-5-google-2024.md` ([blog.google](https://blog.google/technology/ai/google-gemini-next-generation-model-february-2024/))
**Authors**: [[google]] / [[deepmind|Google DeepMind]]. The post's notes are by Sundar Pichai and Demis Hassabis (byline from general knowledge).
**Date**: **2024-02-15**

## Summary

Google announces **[[gemini-1-5|Gemini 1.5]]**, one week after Gemini 1.0 Ultra became generally available. The first model is **Gemini 1.5 Pro**, "a mid-size multimodal model" that performs "at a similar level to 1.0 Ultra".

## Key claims

- **Architecture:** built on Transformer and **[[mixture-of-experts|MoE]]** research, which only activates the relevant expert pathways. It is more efficient to train and serve.
- **Context window:**
  - 128K tokens as standard;
  - **up to 1 million tokens** for a limited group of developers and enterprise customers, in private preview through AI Studio and Vertex AI;
  - **up to 10 million tokens** "successfully tested" in research.
  - Gemini 1.0 had 32K. The post calls 1M "the longest context window of any large-scale foundation model yet."
- **What 1M tokens holds:** 1 hour of video, 11 hours of audio, codebases of more than 30,000 lines, or more than 700,000 words.
  - Demos: the 402-page Apollo 11 transcript, a 44-minute Buster Keaton film, and more than 100,000 lines of code.
- **[[needle-in-a-haystack|Needle in a haystack]]:** finds the embedded text **99%** of the time in contexts up to 1M tokens.
- **Long-context in-context learning:** learns to translate Kalamang, a language with fewer than 200 speakers, from a grammar book given in the prompt (the MTOB benchmark).
- **Benchmarks:** beats 1.0 Pro on 87% of Google's internal benchmarks, and is "broadly similar" to 1.0 Ultra.

## Caveats

- These are vendor claims, from a blog post rather than the technical report.
- Needle-in-a-haystack tests *retrieval* of one fact. [[lost-in-the-middle-liu-2023|Lost in the Middle]] (2023) and later [[context-rot]] work show that *using* many facts across a long context is harder, so 99% NIAH does not mean effective use of all 1M tokens.

## Lineage (context-window milestones)

| Date | Milestone |
|---|---|
| March 2023 | [[gpt-4|GPT-4]]: 8K, with 32K |
| July 2023 | Lost in the Middle shows position effects |
| Feb 2024 | **Gemini 1.5 Pro**: 128K standard, 1M preview, 10M in research |
| later | Claude 1M variants, see [[long-context-llm]] |

Long context then competed with, and complemented, [[retrieval-augmented-generation|RAG]] and [[context-engineering]].

## Concepts & entities

[[gemini-1-5]] · [[long-context-llm]] · [[mixture-of-experts]] · [[needle-in-a-haystack]] · [[in-context-learning]] · [[context-rot]] · [[google]] · [[deepmind]]
