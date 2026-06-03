---
title: Claude Opus 4.8
type: entity
created: 2026-06-03
updated: 2026-06-03
sources:
  - claude-code-dynamic-workflows-harness-2026.md
  - claude-effort-and-ultracode-docs-2026.md
family: claude
predecessor: [[claude]]
release_date: 2026
status: draft
importance: medium
tags:
  - model
---

# Claude Opus 4.8

**Claude Opus 4.8** is the largest tier of [[anthropic|Anthropic]]'s [[claude|Claude]] 4.x generation — the most capable Opus model as of mid-2026, and the model maintaining this wiki. It is the model that **enables [[dynamic-workflows|dynamic workflows]] in [[claude-code|Claude Code]]**: per the [[claude-code-dynamic-workflows-harness-2026|"A Harness for Every Task"]] post, Opus 4.8's capability is what lets Claude write a *custom, task-specific* orchestration harness on the fly rather than relying on a generic static workflow.

It also offers a **fast mode** in Claude Code (faster output without downgrading to a smaller model), toggleable via `/fast`, available on Opus 4.8 / 4.7 / 4.6.

Opus 4.8 supports the full [[reasoning-effort|`effort`]] range including **`xhigh`** (long-horizon agentic work, million-token budgets) and **`max`**; the default is `high` on all surfaces. It uses [[adaptive-thinking|adaptive thinking]] — manual `budget_tokens` thinking is rejected (400), so effort is the control for thinking depth. Its `xhigh` level is what [[claude-code|Claude Code]]'s [[ultracode]] mode builds on.

## Lineage

Part of the [[claude]] family, succeeding the Claude 4.7 Opus / 4.6 Sonnet line. Architectural details are not officially disclosed (closed-weights, like prior Claude generations). Offers a 1M-token context variant consistent with the 4.x generation.

> [!note] Draft
> This stub records only what the dynamic-workflows sources state plus general Claude-family context. Enrich on a future ingest of an Opus 4.8-specific announcement.

## References

- [[claude-code-dynamic-workflows-harness-2026]]
- [[claude]] — the family page.
- [[dynamic-workflows]] — the capability Opus 4.8 enables.
