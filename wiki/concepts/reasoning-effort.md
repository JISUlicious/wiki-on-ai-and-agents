---
title: Reasoning Effort (the `effort` parameter)
type: concept
created: 2026-06-03
updated: 2026-10-10
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

> [!warning] Contradiction — effort defaults changed with the Claude 5.5 models
> The guidance above (default `high` on all surfaces; start coding and agentic work at `xhigh`) describes the Opus 4.7/4.8 generation. For [[claude-opus-5-5|Opus 5.5]] the default is **`medium`**, and at a given level it thinks more than Opus 5. For [[claude-sonnet-5-5|Sonnet 5.5]] the default is `high` on the API but `medium` in Claude Code, and its levels are recalibrated so old settings do not carry over. Anthropic's 5.5-era advice is to start at `medium` and keep `xhigh`/`max` for work where a gain has been measured.
>
> [[using-claude-code-spending-your-effort-shihipar-2026|Spending your effort]] (Anthropic-internal Terminal-Bench 3.0) adds why: higher effort mostly buys verification. "A bug its tests missed" failures fell from 40 to 14 between low and max, while "picked the wrong reading" failures rose from 25 to 47. Treat the `xhigh` recommendation above as specific to the 4.x models.

## Related

- [[ultracode]] — Claude Code's `xhigh` + auto-workflow mode (not itself an API effort level).
- [[adaptive-thinking]] — the thinking mode effort controls.
- [[dynamic-workflows]] — what `ultracode` is licensed to spawn.
- [[claude-opus-4-8]] — the model whose `xhigh`/`max` levels enable long-horizon agentic work.

## References

- [[claude-effort-and-ultracode-docs-2026]] — the authoritative effort docs.
- Docs: https://platform.claude.com/docs/en/build-with-claude/effort

## Related 2026 sources

- [[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]] — DeepSeek-V4.1-Flash: encoder–decoder prefill split, cross-layer sparse KV reuse, FP4 KV; indexer cut to 32 heads, top-k 512
- [[building-with-claude-sonnet-5-5-osmani-2026]] — Osmani's Sonnet 5.5 launch guide: spec sheet, pricing, six breaking migration changes from Sonnet 5
- [[what-a-task-costs-on-opus-5-5-osmani-2026]] — Worked cost model for an agentic Claude Code task on Opus 5.5; what the "40% cheaper" claim actually means
- [[using-claude-code-spending-your-effort-shihipar-2026]] — Shihipar: on Terminal-Bench 3.0, more effort buys verification and edge-case testing, not better approaches
- [[claude-opus-5-5]] — Claude Opus 5.5 (Sep 2026): cheaper tokens and cache reads, default effort medium
- [[claude-sonnet-5-5]] — Claude Sonnet 5.5 (2026-09-28): 1M context, same prices as Sonnet 5, stricter thinking semantics

## Lineage sources

- [[dualformer-su-2024]] — auto mode decides how much to think (2024)
