---
title: oh-my-claude
type: entity
created: 2026-06-03
updated: 2026-06-03
sources:
  - oh-my-claude-ultrawork-2026.md
status: draft
importance: low
tags:
  - product
---

# oh-my-claude

**oh-my-claude** is a third-party, community-maintained [[claude-code|Claude Code]] plugin (by *TechDufus*, MIT-licensed) — **not** an official [[anthropic|Anthropic]] product. Its philosophy is "enhance, don't replace": it adds guardrails, a specialized agent team, and orchestration modes on top of Claude Code's native behavior.

It is the reference implementation of the [[ultrawork]] keyword: adding `ultrawork` / `ulw` to a prompt makes the plugin inject parallel-execution directives (parallel subagents, librarian-delegated file reading, mandatory TodoWrite progress, refuse-partial-work). Sibling modes include **ultraresearch** (`ulr`) and **ultradebug** (`uld`), plus an enhanced 7-step plan mode. Agents provided: librarian, advisor, critic, risk-assessor, validator, worker.

Install (requires `uv`): `/plugin marketplace add TechDufus/oh-my-claude` then `/plugin install oh-my-claude@oh-my-claude`. macOS/Linux (Windows planned).

## Related

- [[ultrawork]] — the trend this plugin implements.
- [[claude-code]] — the host runtime it extends.

## References

- [[oh-my-claude-ultrawork-2026]]
- https://github.com/TechDufus/oh-my-claude
