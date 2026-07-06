---
title: "AutoMem: Automated Learning of Memory as a Cognitive Skill"
type: source
created: 2026-07-06
updated: 2026-07-06
sources:
  - automem-stanford.pdf
arxiv_id: "2607.01224"
authors:
  - Shengguang Wu
  - Hao Zhu
  - Yuhui Zhang
  - Xiaohan Wang
  - Serena Yeung-Levy
first_author: Shengguang Wu
year: 2026
tags:
  - 2026
status: complete
importance: medium
---

# AutoMem: Automated Learning of Memory as a Cognitive Skill

Surfaced via the NLP newsletter (Top AI Papers of the Week, 2026-06-28–07-05).

## Summary

AutoMem (Stanford) treats **memory management as a trainable cognitive ability** — importing the cognitive-science notion of *metamemory* (knowing what to encode, when to retrieve, how to organize) into LLM agents. File-system operations are promoted to **first-class memory actions** alongside task actions, so the model itself decides how to manage its memory. The memory skill improves along two axes that both resist manual tuning (episodes run for thousands of steps; a single memory mistake can hide long before surfacing): the *structure* that supports it (prompts, file schemas, action vocabulary) and the *proficiency* of the model exercising it. AutoMem automates both with two loops.

## Key points

- **Metamemory framing**: memory ops (read / write / search over a file-system memory) live in the agent's action space, letting the model learn what to remember, when to retrieve, and how to organize — rather than a fixed external-memory scheme.
- **Loop 1 (scaffold optimization)**: a strong LLM reviews complete agent trajectories and iteratively revises the memory structure (prompts, file schemas, action vocabulary) shaping how the agent uses its memory files.
- **Loop 2 (proficiency training)**: the agent's own good memory decisions, mined across many episodes, become training signal to sharpen memory proficiency directly.
- Evaluated on three procedurally generated long-horizon games — **Crafter, MiniHack, NetHack**.
- Optimizing memory alone (without changing task-action behavior) improved a base agent's performance ~2x–4x, making a 32B open-weight model (Qwen2.5-32B-Instruct) competitive with frontier systems like Claude Opus 4.5 and Gemini 3.1 Pro Thinking.
- Core claim: memory management is an *independently learnable* skill and a high-leverage objective for long-horizon tasks.

## Concepts & entities

From [[stanford-university]]. Core to [[memory-management]] — reframing it as a trainable action space — and an instance of a [[self-improving-agent]] via the two self-optimizing loops (scaffold review and proficiency training on the agent's own decisions).

## References

- [arXiv:2607.01224](https://arxiv.org/abs/2607.01224) — "AutoMem: Automated Learning of Memory as a Cognitive Skill", Wu et al. (Stanford University), 2026. Project page: https://autolearnmem.github.io/
