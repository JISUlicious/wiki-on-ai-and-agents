---
title: "Effort (Claude API Docs) — effort levels & ultracode"
type: source
created: 2026-06-03
updated: 2026-06-03
sources:
  - claude-effort-and-ultracode-docs-2026.html
introduces:
  - [[reasoning-effort]]
  - [[ultracode]]
year: 2026
venue: platform.claude.com docs
publisher: Anthropic
tags:
  - 2026
status: complete
importance: high
---

# Effort (Claude API Docs) — effort levels & ultracode

**Source**: `sources/claude-effort-and-ultracode-docs-2026.html` (official [[anthropic|Anthropic]] API documentation)
**URL**: https://platform.claude.com/docs/en/build-with-claude/effort
**Fetched**: 2026-06-03

## Summary

The authoritative documentation for the **`effort`** parameter — Anthropic's single control over how eagerly [[claude|Claude]] spends tokens, trading thoroughness against efficiency. It defines [[reasoning-effort|the five effort levels]] (`low`/`medium`/`high`/`xhigh`/`max`), their per-model availability, and how effort interacts with [[adaptive-thinking|adaptive thinking]]. It is also the canonical statement that [[claude-code|Claude Code]]'s **[[ultracode]]** mode is *not* an API effort level but a Claude-Code convenience pairing `xhigh` with a standing workflow-launch grant.

## Key points

- **Effort is a behavioral signal, not a token budget.** It affects *all* tokens — text, tool calls + arguments, and (when enabled) thinking. Lower effort ⇒ fewer/combined tool calls, less preamble.
- **Levels**: `max` (no constraints), `xhigh` (long-horizon, 30+ min, million-token budgets; Opus 4.8 & 4.7 only), `high` (default; == omitting the param), `medium` (balanced), `low` (most efficient; good for subagents).
- **Default is `high` on all surfaces** including the API and Claude Code; pass `effort` explicitly to change it. For Opus 4.7/4.8 the docs recommend **starting at `xhigh`** for coding/agentic work; reserve `max` (overthinking risk on structured output).
- **Thinking**: effort replaces deprecated `budget_tokens`; on [[claude-opus-4-8|Opus 4.8]] manual `budget_tokens` is rejected (400) — effort + adaptive thinking only.
- **ultracode** (verbatim): *"ultracode appears in Claude Code's effort menu, but it is not an additional API effort level... Ultracode pairs the `xhigh` effort level with standing permission for Claude Code to launch multi-agent workflows, granted through Mid-conversation system messages."*
- Set via `output_config.effort` on the API; available on Opus 4.8, Mythos Preview, Opus 4.7/4.6/4.5, Sonnet 4.6, no beta header.

## Concepts & entities

- [[reasoning-effort]] — the parameter this page defines.
- [[ultracode]] — the Claude Code mode it disambiguates from the API.
- [[adaptive-thinking]], [[dynamic-workflows]], [[claude-opus-4-8]], [[claude-code]], [[anthropic]].

## References

- [[reasoning-effort]] · [[ultracode]]
- _Original source: `sources/claude-effort-and-ultracode-docs-2026.html`_
