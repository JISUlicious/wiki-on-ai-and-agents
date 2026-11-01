---
title: "MemoryTax: The Bitter Lesson of Agent Memory"
type: source
created: 2026-10-29
updated: 2026-10-29
sources:
  - memorytax-bitter-lesson-agent-memory-peng-2026.md
arxiv_id: "2610.29823"
authors:
  - Zedong Peng
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# MemoryTax: The Bitter Lesson of Agent Memory

**Zedong Peng** ([[shanghai-jiao-tong-university|Shanghai Jiao Tong University]]) — [arXiv:2610.29823](https://arxiv.org/abs/2610.29823), v1, cs.CL, 21 Oct 2026. Short empirical study; interactive version on the author's site.

## Summary

Designed agent memory ([[mem0|Mem0]], [[zep|Zep]]) decides what to keep before any question arrives. Coding agents instead keep the raw history and let the model search it with its own tools. The paper asks whether designed memory still helps once the model is strong enough to search for itself. It runs a **7 × 3 grid**: four memory methods (full context, [[bm25|BM25]], Mem0, Zep) and three coding agents ([[pi-mono|Pi]], Codex, [[claude-code|Claude Code]]), each with GPT-4o-mini, GPT-6 Luna and [[claude-opus-5-5|Claude Opus 5.5]]. The data is one [[locomo|LoCoMo]] conversation (conv-26: 419 turns, 19 sessions, 152 questions; adversarial questions excluded). Every rollout's cost is recorded. The history fits in every model's window.

## Key points

- **Memory is a costly patch for a weak model.** With GPT-4o-mini, Mem0 scores 78% vs 74% for full context and BM25 (Zep 43%). With GPT-6 Luna, full context 92%, BM25 82%, Mem0 80%. With Opus 5.5, Mem0 and full context tie at 95%, Zep 87%.
- **The build costs more than not building.** Mem0's build alone costs more than a whole full-context rollout: $0.56 vs $0.43 (4o-mini), $0.45 vs $0.29 (Luna), $25.88 vs $17.84 (Opus). It would break even only after 372–596 questions; LoCoMo asks 152. Mem0 and Zep are never on the cost–accuracy frontier.
- **The writer sets a ceiling.** Retrieval calls no LLM, so readers can be swapped. On Zep's GPT-4o-mini graph, the three readers score 43/43/45%; on Opus's graph, 66/71/87%. Mem0: 78/83/84% from 4o-mini's store vs 89/91/95% from Opus's. A stronger reader does not recover what a weak writer left out.
- **Pi + Opus 5.5 tops the frontier**: 99% for $6.95 a rollout vs 95% for $17.84 with full context. Codex and Claude Code score 98% and 97% at $25.41 and $28.41. Pi's one miss is an answer-key error (the painting was Melanie's, not Caroline's), which the other strong cells also flag.
- **Weak models don't search.** With GPT-4o-mini, Codex opens the file for about three questions in four and Claude Code almost never; they mostly answer from the question alone (28–29%).

> [!warning] Newsletter vs paper
> The "99% at under half the cost" is accurate but confounded. Agents received the official file **with photo captions**; memory methods got turn text only. 64 of 152 questions have photo evidence, and six of the seven questions Pi wins over full context are among them. On the 88 questions without photo evidence, Pi scores 100% and full context, Mem0, Codex and Claude Code 99% each. The paper itself says it does not settle whether searching beats reading everything.

## What it does not show

- One conversation, one run per cell, and a GPT-4o-mini judge (the author lists these limits).
- The history fits every model's window. Designed memory may still matter once history exceeds the window or spans many users.
- Only default Mem0/Zep settings; no tuned or newer memory systems.

## Concepts & entities

- [[long-term-memory]] · [[memory-construction]] · [[graph-memory]] · [[mem0]] · [[zep]] · [[locomo]].
- [[agentic-search]] · [[coding-agents-effective-long-context-processors-cao-2026]] — the "let the agent grep the raw record" side.
- [[jev-mem-system-one-agentic-memory-jiang-2026]] — optimizes designed memory on the same benchmark (but includes the adversarial category).
- [[corpusmap-follow-the-entities-jeong-2026]] — finds the opposite for large corpora: a shared entity layer beats raw-corpus search.
- [[pi-mono]] · [[claude-code]] · [[claude-opus-5-5]] · [[bm25]] · [[scaling-laws]] (the Bitter Lesson framing).

## References

- [arXiv:2610.29823](https://arxiv.org/abs/2610.29823)
