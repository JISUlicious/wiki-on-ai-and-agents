---
title: "Claudini: Autoresearch Discovers State-of-the-Art Adversarial Attack Algorithms for LLMs"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - claudini-autoresearch-adversarial-attacks-panfilov-2026.md
arxiv_id: "2603.24511"
authors:
  - Alexander Panfilov
  - Peter Romov
  - Igor Shilov
  - Yves-Alexandre de Montjoye
  - Jonas Geiping
  - Maksym Andriushchenko
first_author: Alexander Panfilov
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# Claudini: Autoresearch Discovers State-of-the-Art Adversarial Attack Algorithms for LLMs

Surfaced via the alphaXiv / NLP-newsletter digest.

[arXiv:2603.24511](https://arxiv.org/abs/2603.24511) — Panfilov, Romov, Shilov, de Montjoye, Geiping, Andriushchenko (MATS, ELLIS Institute Tübingen / MPI-IS, Tübingen AI Center, Imperial College London), 2026.

## Summary

The paper shows that frontier coding agents can autonomously **discover novel adversarial-attack algorithms** against LLMs that beat the prior state of the art on white-box jailbreaking and prompt injection. The authors deploy agents such as Claude Code and Codex in an **autoresearch loop**: each sandboxed agent has access to a library of 30+ prior attack implementations, their results, and an evaluation script with a fixed compute budget. The agent proposes a method intended to improve on existing ones, implements it, submits it for evaluation, and iterates — fully autonomously, with each new method added to a growing pool, no human in the loop.

The pipeline produces SOTA attacks in two white-box settings. Against OpenAI's GPT-OSS-Safeguard-20B it reaches up to **80% attack success rate** on held-out CBRN queries (vs. <50% for existing methods), and against the adversarially robust Meta-SecAlign-70B it reaches **100% ASR** (vs. 82% for the best prior automated methods). Strikingly, methods developed on unrelated surrogate models for a pure random-target token-forcing task transfer directly to prompt injection on the adversarially trained model. The authors trace the lineage of discovered methods, characterize agent strategies and failure modes, and argue autoresearch-tailored attacks should be the **minimum bar for defense evaluation**.

## Key points

- Frontier agents (Claude Code, Codex) in an **autoresearch loop** discover novel white-box jailbreak / prompt-injection algorithms, no human in the loop.
- Setup: library of 30+ prior methods + fixed-compute evaluation script; agent proposes → implements → evaluates → iterates, growing a method pool.
- **GPT-OSS-Safeguard-20B**: up to **80% ASR** on CBRN queries (vs. <50% prior).
- **Meta-SecAlign-70B** (robust model): **100% ASR** via prompt injection (vs. 82% best prior automated).
- **Cross-task transfer**: attacks optimized on surrogate models (Qwen-2.5-7B, Llama-2-7B, Gemma-7B) for token-forcing transfer directly to prompt injection on a defended model.
- Argues attacks tailored by autoresearch should be the **minimum bar** for evaluating defenses going forward.

## Concepts & entities

- [[self-improving-agent]] — Autoresearch loop iteratively improves discovered attack methods.
- [[prompt-injection]] — One of the two attack regimes (against Meta-SecAlign-70B).
- [[indirect-prompt-injection]] — Related injection threat surface.
- [[red-teaming]] — Automated, agent-driven red-teaming of LLM defenses.
- [[claude-code|Claude Code]] — One of the frontier agents driving the loop.
- [[code-as-harness]] — Agents operate via a code-execution evaluation harness.

## References

- ClearHarm [21] — CBRN harmful-query set used for jailbreaking evaluation.
- Targets: GPT-OSS-Safeguard-20B [41], Meta-SecAlign-70B; surrogates Qwen-2.5-7B, Llama-2-7B, Gemma-7B.
