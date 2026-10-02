---
title: Programmatic Tool Calling
type: concept
created: 2026-08-17
updated: 2026-09-30
sources:
  - the-bitter-lesson-of-tool-calling-patel-2026.md
status: complete
importance: high
---

# Programmatic Tool Calling

Exposing tools to a model as **typed code stubs it invokes by writing a program**, rather than as a schema it fills in with a JSON object. The model emits code; the harness executes it in a sandbox and returns the result, with chaining, branching and parallelism handled by the language rather than by the agent loop.

The contrast is with **native JSON tool calling** ([[function-calling]]), where each call is one structured object validated against a [[json-schema]] and every chained call costs another round trip through the model.

## Why the interface choice is load-bearing

JSON tool calling makes the *harness* responsible for control flow. Chaining two calls means the model must emit call A, wait for the result to re-enter its context, then emit call B. Fanning out to twenty calls means serializing twenty objects. Programmatic calling moves that control flow into code the model already knows how to write, so composition, loops and filtering are free and intermediate results need not pass through the context window at all.

This is why the paper's title invokes the bitter lesson: the advantage **tracks model coding capability across release generations**, so it is a directional bet rather than a tuning trick.

## What the comparison actually shows

[[the-bitter-lesson-of-tool-calling-patel-2026|Patel et al.]] ran 14 models on a 309-entry BFCL v4 subset with stop middleware enforcing per-entry LLM-call parity, so neither paradigm can win by simply spending more calls:

- **Matches or exceeds JSON on 11 of 14 models**; GPT-5.6-Sol and -Terra each **+10.6%**.
- **13 of 14 under parallel fan-out** — the gap widens exactly where JSON must serialize.
- Under context flooding, JSON degrades **−2.3%** while programmatic calling *improves* **+5.5%**.
- Token crossover at roughly **N ≈ 26** tools.

  - Measured end-to-end on enterprise agent tasks, the picture is weaker: [[is-bash-all-you-need-enterprise-agents-mak-2026|Is Bash All You Need?]] (Microsoft, 2026) finds programmatic tool calling saves tokens but improves score only for Opus on one benchmark, trails plain bash everywhere, and has the lowest tool-call success rate (65.2%). The 11-of-14 result above comes from echo-stub BFCL tests.

The sharpest structural result is a scaling wall: Claude Sonnet 5's JSON enumeration is 100% at N ≤ 70, 75% at N = 72, and **0% at N = 100**, while the programmatic path holds 100% at both — though GPT-5.6-Sol holds 100% JSON through N = 100, so this is a serialization limit of one implementation rather than a universal one.

> [!warning] The aggregate goes the other way
> **Macro-averaged over all 14 models, JSON scores 78.6% and programmatic 77.0%.** Programmatic calling is *behind* on the overall average, losing 14.1 points absolute in the `parallel`/`parallel_multiple` categories. The three model failures (GPT-4o, GPT-4.1, GPT-5.4-mini, −19.7 to −26.9%) trace to a single mechanical bug — literal `\n` escapes instead of real newlines breaking multiline scripts. So the honest reading is "wins on capable models and under pressure, loses on the mean," not "wins."
>
> Also: BFCL v4 uses **echo-return stubs**, so this measures argument serialization, not end-to-end tool use.

## Related

- [[function-calling]] · [[json-schema]] · [[tool-call-parser]] · [[structured-outputs]] — the JSON path this is measured against.
- [[codeact]] · [[code-as-harness]] · [[pal]] · [[code-as-policies]] — the lineage; CodeAct made the same argument for actions generally.
- [[mcp]] — the dominant tool-exposure protocol, JSON-schema-shaped.
- [[agent-skills]] — Anthropic shipped programmatic tool calling as a product feature alongside the Tool Search Tool; see [[agent-skills-survey-xu-2026]].
- [[tool-use]] · [[agentic-harness-engineering]] · [[llm-agent]]

## References

- [[the-bitter-lesson-of-tool-calling-patel-2026]]

## Related 2026 sources

- [[is-bash-all-you-need-enterprise-agents-mak-2026]] — Microsoft: bash alone beats typed tools end-to-end (+5–25 pts, 19–72% fewer tokens); programmatic tool calling mixed
- [[seeing-like-an-agent-shihipar-2026]] — Claude Code tool-design lessons: structured tools where the harness parses output, grep over RAG, fewer tools
