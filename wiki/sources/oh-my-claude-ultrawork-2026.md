---
title: "oh-my-claude — ultrawork plugin (README)"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - oh-my-claude-ultrawork-2026.md
introduces:
  - [[ultrawork]]
  - [[oh-my-claude]]
year: 2026
venue: GitHub (TechDufus/oh-my-claude)
tags:
  - 2026
status: complete
importance: medium
---

# oh-my-claude — ultrawork plugin (README)

**Source**: `sources/oh-my-claude-ultrawork-2026.md` (GitHub README, third-party community plugin — **not** official [[anthropic|Anthropic]])
**URL**: https://github.com/TechDufus/oh-my-claude
**Fetched**: 2026-06-03

## Summary

README for **[[oh-my-claude]]**, the reference community implementation of the [[ultrawork]] trend for [[claude-code|Claude Code]]. It documents the `ultrawork` prompt-keyword mechanic and the behavioral changes it injects, the specialized agent team it ships, and sibling modes. The primary value for this wiki is a concrete, primary-source account of how the *unofficial* ultrawork pattern actually works — to be read against the *official* [[ultracode]] / [[dynamic-workflows]] mechanisms.

## Key points

- **Mechanic**: add `ultrawork` (or `ulw`) anywhere in a prompt; the plugin intercepts it and injects directives that flip Claude from sequential to parallel operation.
- **Behavior changes**: parallel subagents in one message; file reading delegated to a "librarian" agent (saves main context); mandatory TodoWrite progress; stops only when all todos complete; **refuses incomplete/partial work**.
- **Agent team**: librarian, advisor, critic, risk-assessor, validator, worker.
- **Sibling modes**: `ultraresearch` (`ulr`) — exhaustive parallel web search + cite everything; `ultradebug` (`uld`) — 7-step debugging method; enhanced 7-step plan mode (Recon → Interview → Research → Gap Analysis → Risk → Write Plan → Critic Review).
- **Status**: third-party, MIT-licensed, by TechDufus; "enhance, don't replace." Install via `/plugin marketplace add TechDufus/oh-my-claude` + `/plugin install` (requires `uv`; macOS/Linux).

## Concepts & entities

- [[ultrawork]] — the trend; [[oh-my-claude]] — the plugin.
- [[claude-code]], [[multi-agent]], [[dynamic-workflows]], [[ralph-loop]].

## References

- [[ultrawork]] · [[oh-my-claude]]
- _Original source: `sources/oh-my-claude-ultrawork-2026.md`_
