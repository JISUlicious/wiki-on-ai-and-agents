---
title: Reasoning Effort (the `effort` parameter)
type: concept
created: 2026-06-03
updated: 2026-06-03
sources:
  - claude-effort-and-ultracode-docs-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Reasoning Effort (the `effort` parameter)

**Effort** is [[anthropic|Anthropic]]'s single-knob control over how eagerly [[claude|Claude]] spends tokens when responding — a trade-off between thoroughness and token efficiency, exposed on the [[anthropic|Anthropic]] API as `output_config.effort` and in [[claude-code|Claude Code]] via the `/effort` command. Crucially, it is **a behavioral signal, not a strict token budget**, and it affects **all tokens** in a response — text, tool calls and their arguments, and (when enabled) extended thinking. Lower effort means Claude not only writes less but also makes fewer tool calls and skips preamble. The default is `high` on all surfaces (API and Claude Code); setting `high` explicitly is identical to omitting the parameter.

## The five levels

| Level | What it is | Availability (per docs) |
|---|---|---|
| `max` | Absolute maximum capability, no constraints on token spend | Opus 4.8, Mythos Preview, Opus 4.7, Opus 4.6, Sonnet 4.6 |
| `xhigh` | Extended capability for long-horizon work (30+ min, million-token budgets) | Opus 4.8, Opus 4.7 |
| `high` | High capability; equals omitting the parameter (default) | all supported models |
| `medium` | Balanced; moderate token savings | all |
| `low` | Most efficient; significant savings, some capability reduction; good for subagents | all |

Supported on [[claude-opus-4-8|Opus 4.8]], Opus 4.7, Opus 4.6, Sonnet 4.6, Opus 4.5, and the Mythos Preview — no beta header required. For coding/agentic work the docs recommend **starting at `xhigh`**, using `high` as the floor for intelligence-sensitive work, and reserving `max` for genuinely frontier problems (it can cause *overthinking* on structured-output tasks).

## Relationship to thinking

For Opus 4.8 / 4.7 / 4.6 and Sonnet 4.6, effort is the recommended control for **[[adaptive-thinking|adaptive thinking]]** depth — it replaces the older `budget_tokens` knob (deprecated). At `high`/`xhigh`/`max` Claude almost always thinks deeply; at lower levels it may skip thinking on simple problems. On [[claude-opus-4-8|Opus 4.8]], manual `budget_tokens` thinking is rejected (400) — effort + adaptive thinking is the only path. This makes effort the practical successor to per-request thinking budgets in the broader [[test-time-compute|test-time-compute]] story (cf. [[chain-of-thought]], [[sleep-time-compute]]).

## Why it matters

Effort decouples *capability dialing* from *prompt engineering*: instead of padding prompts to coax more reasoning (or trimming them for speed), a caller sets one parameter that scales the whole response budget — including agentic tool-call volume. It is the API substrate beneath [[claude-code|Claude Code]]'s `/effort` menu, whose top entry, **[[ultracode]]**, pairs the `xhigh` level with standing permission to launch [[dynamic-workflows|multi-agent workflows]].

## Related

- [[ultracode]] — Claude Code's `xhigh` + auto-workflow mode (not itself an API effort level).
- [[adaptive-thinking]] — the thinking mode effort controls.
- [[dynamic-workflows]] — what `ultracode` is licensed to spawn.
- [[claude-opus-4-8]] — the model whose `xhigh`/`max` levels enable long-horizon agentic work.

## References

- [[claude-effort-and-ultracode-docs-2026]] — the authoritative effort docs.
- Docs: https://platform.claude.com/docs/en/build-with-claude/effort
