---
title: ultracode (Claude Code mode)
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - claude-effort-and-ultracode-docs-2026.md
  - claude-code-dynamic-workflows-2026.md
builds-on:
  - [[reasoning-effort]]
  - [[dynamic-workflows]]
status: complete
importance: high
tags:
  - 2026
---

# ultracode (Claude Code mode)

**ultracode** is an **official** [[claude-code|Claude Code]] setting, selectable from the `/effort` menu (`/effort ultracode`). It is *not* an additional API [[reasoning-effort|effort]] level — the API accepts only `low`/`medium`/`high`/`xhigh`/`max`. Instead, ultracode **combines two things**: the `xhigh` effort level **plus standing permission for Claude Code to launch [[dynamic-workflows|multi-agent workflows]] on its own**, granted through mid-conversation system messages. With ultracode on, Claude plans a workflow for **every substantive task in the session** rather than waiting to be asked.

## Behavior

- **Set with `/effort ultracode`.** A single request can fan into several workflows in sequence — e.g. one to *understand* the code, one to *make* the change, one to *verify* it.
- **Applies to every task in the session**, so each request uses substantially more tokens and takes longer than at lower effort.
- **Session-scoped**: ultracode lasts for the current session and resets when you start a new one. Drop back to routine behavior with `/effort high`.
- It is the autonomous counterpart to asking for a workflow explicitly: under ultracode, *Claude* decides when a [[dynamic-workflows|dynamic workflow]] is warranted.

> [!note] ultracode vs. the API
> Per the effort docs, ultracode is a Claude-Code-surface convenience: `xhigh` effort + a standing workflow-launch grant. To replicate it against the raw API you build an "orchestration mode" yourself using mid-conversation system messages — there is no `effort: "ultracode"` value.

## Disambiguation: ultracode vs. [[ultrawork]]

These are easy to conflate (and are often mentioned together) but are different things:

| | **ultracode** | **[[ultrawork]]** |
|---|---|---|
| Origin | **Official** Anthropic / Claude Code | **Community** trend; not an Anthropic feature |
| Trigger | `/effort ultracode` | A prompt keyword injected by third-party plugins (e.g. `oh-my-claude`) |
| Mechanism | `xhigh` effort + standing permission to launch [[dynamic-workflows]] | Plugin intercepts the prompt and injects parallel-execution directives |

See [[ultrawork]] for the community side.

## Related

- [[reasoning-effort]] — the `effort` taxonomy; ultracode sits on top of `xhigh`.
- [[dynamic-workflows]] — what ultracode is licensed to spawn automatically.
- [[claude-code]] — the host; [[claude-opus-4-8]] — the model whose `xhigh` underpins it.
- [[ultrawork]] — the unofficial community analogue.

## References

- [[claude-effort-and-ultracode-docs-2026]] — authoritative definition (effort docs).
- [[claude-code-dynamic-workflows-2026]] — announcement; introduces ultracode as the auto-workflow path.
- Docs: https://platform.claude.com/docs/en/build-with-claude/effort and https://code.claude.com/docs/en/workflows
