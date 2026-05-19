---
title: GAIA
type: concept
created: 2026-05-17
updated: 2026-05-17
sources:
  - gaia-mialon-2023.md
status: complete
importance: high
tags:
  - 2025
---

# GAIA

**GAIA** (General AI Assistants) is a 2023 benchmark from [[meta]] FAIR, [[huggingface]], and AutoGPT for evaluating tool-augmented [[llm-agent]]s on real-world questions that are *conceptually simple for humans yet hard for frontier LLMs*. Despite predating the modern agent wave, GAIA has become the canonical general-assistant evaluation cited by virtually every major 2024–2026 agent paper.

## Core mechanism

- **466 questions across 3 difficulty levels** — 166 developer set + 300 leaderboard-only, split by annotator-measured step/tool counts: Level 1 (≤1 tool, ≤5 steps, ~6 min human time), Level 2 (~5–10 steps, multiple tools), Level 3 (arbitrarily long action sequences, ~17 min human time).
- **Tool-use heavy real-world tasks** — questions require browsing the open changing web (355 Qs), coding (154), multi-modality (138), and diverse filetype reading (129; PDFs, spreadsheets, images, audio, video). Designed to be non-gameable: answers are factoid strings/numbers/lists deliberately absent from plain-text pretraining data.
- **Human 92% vs GPT-4 + plugins ~15%** — a striking inversion of the usual benchmark-saturation narrative; GPT-4 + plugins scores 0% on Level 3, AutoGPT-4 underperforms even bare GPT-4 at Level 2, and a plain search engine solves only a subset of Level 1.
- **Quasi-exact-match evaluation** — strict "FINAL ANSWER: …" template with deterministic match on normalised answers; no LLM-judge, no rubric — keeps the eval cheap and reproducible at million-rollout scale.
- **Cited by every major 2024–2026 agent paper** — UI-TARS, DGM, browser-agent and multi-tool assistant lines all use GAIA as their north-star general-assistant eval, alongside code-domain analogues like [[swe-bench-live]] and [[swe-gym]].

## What it advances

GAIA departed from the prevailing "make tasks harder for humans" benchmark philosophy and instead anchored on the **t-AGI** framing: a system that matches human performance on questions humans solve in ~6–17 minutes is a competent general assistant. By being cheap to evaluate, contamination-resistant, and tool-use-heavy, it became the default substrate for agent research.

## Relation to other concepts

- Sits in the [[agent-benchmark]] family alongside [[swe-bench-live]], [[swe-gym]], OSWorld, and AppWorld.
- Evaluates [[llm-agent]] capability, particularly [[tool-use]] proficiency and web browsing.
- Hosted on [[huggingface]] at `huggingface.co/gaia-benchmark`; authored at [[meta]] FAIR with AutoGPT contributors.
- Complements code-focused benchmarks (SWE-bench family) and GUI-focused benchmarks ([[osworld]], WindowsAgentArena).

## References

- [[gaia-mialon-2023]] — [arXiv:2311.12983](https://arxiv.org/abs/2311.12983)
- Leaderboard: <https://huggingface.co/gaia-benchmark>
