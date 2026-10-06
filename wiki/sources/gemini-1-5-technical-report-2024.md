---
title: "Gemini 1.5: Unlocking multimodal understanding across millions of tokens of context"
type: source
created: 2026-10-07
updated: 2026-10-07
sources:
  - gemini-1-5-technical-report-2024.md
arxiv_id: "2403.05530"
venue: arXiv technical report
authors:
  - Gemini Team, Google
introduces:
  - "[[gemini-1-5]]"
year: 2024
tags:
  - 2024
  - paper
status: complete
importance: high
---

# Gemini 1.5 technical report

**Source**: `sources/gemini-1-5-technical-report-2024.md` ([arXiv:2403.05530](https://arxiv.org/abs/2403.05530))
**Authors**: Gemini Team, [[google|Google]]
**Date**: v1 March 2024, from the arXiv ID. The version read here is **v5 (2024-12-16)**, which covers the updated May 2024 Gemini 1.5 Pro and **Gemini 1.5 Flash**.

## Summary

The report behind [[gemini-1-5-google-2024|the February 2024 announcement]]. Gemini 1.5 is "a new family of highly compute-efficient multimodal models" that recalls and reasons over **millions of tokens** of mixed text, video and audio. The models:
- **Gemini 1.5 Pro**: a **sparse [[mixture-of-experts|mixture-of-experts]] Transformer** building on Gemini 1.0 and Google's MoE line (Shazeer 2017, GShard, Switch, and others).
- **Gemini 1.5 Flash**: a dense Transformer decoder with the same 2M+ context, designed for TPU serving latency. It computes attention and feed-forward in parallel and is **online-distilled** from Pro.

Both were trained on multiple 4096-chip TPUv4 pods.

## Key results

- **Single needle in a haystack** ([[needle-in-a-haystack]]):
  - text recall is **100% up to 530K tokens**, **99.7% at 1M** and **99.2% at 10M**;
  - Claude 2.1 scored 98% at 200K.
  - Recall stays near-perfect for video (hours) and audio (days) too.
- **Multiple needles are harder.**
  - With **100 unique needles** to retrieve in one turn, Gemini 1.5 Pro (February version) keeps about **70% recall up to 128K** and **over 60% up to 1M**.
  - GPT-4 Turbo averaged about 50% at 128K, its limit.
- **MRCR** (multi-round co-reference: reproduce the right one of many similar earlier outputs in a long conversation):
  - Gemini 1.5 Pro overtakes GPT-4 Turbo (04-09) and Claude 3 Opus at around 32K tokens;
  - Flash always strictly dominates Claude 3 Haiku.
- **Next-token loss keeps improving** with context up to at least 10M tokens.
- **Kalamang:** given a 500-page reference grammar, a dictionary and about 400 parallel sentences *in context*, the model translates English↔Kalamang (a language with fewer than 200 speakers) about as well as a person who learned from the same materials.
- **Generalist quality:** the updated 1.5 Pro beats 1.0 Ultra on most benchmarks "while requiring significantly less compute to train".
- **Real-world tasks:** time savings of 26–75% reported across 10 job categories (vendor study).

## Why it matters for the wiki

- **The strongest evidence for long context, and its limits.** Recall of a single fact is near-perfect at millions of tokens. Retrieving *many* facts at once falls to about 60% at 1M. That is the gap [[context-rot]] and [[context-engineering]] are about.
- **"Long context vs RAG" becomes concrete:** a whole codebase or book fits in the prompt, and in-context learning works from a grammar book. See [[long-context-llm]] and [[retrieval-augmented-generation]].
- **The pattern of a big MoE plus a distilled small model** (Pro + Flash) recurs across 2024–2026 model families.

## Concepts & entities

[[gemini-1-5]] · [[long-context-llm]] · [[needle-in-a-haystack]] · [[context-rot]] · [[mixture-of-experts]] · [[in-context-learning]] · [[google]] · [[deepmind]]
