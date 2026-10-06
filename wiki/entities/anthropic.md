---
title: Anthropic
type: entity
created: 2026-05-13
updated: 2026-10-06
sources:
  - gpt-3-brown-2020.md
  - anthropic-threat-intelligence-report-september-2026.md
status: complete
importance: high
tags:
  - org
---

# Anthropic

AI safety and research company, founded in 2021 by [[dario-amodei|Dario Amodei]], Daniela Amodei, [[tom-brown|Tom Brown]], [[jared-kaplan|Jared Kaplan]], [[sam-mccandlish|Sam McCandlish]], and other former [[openai|OpenAI]] researchers. Headquartered in San Francisco.

Anthropic produces the **Claude** family of LLMs (Claude 1 in 2023, Claude 2, Claude 3 Sonnet/Opus/Haiku in 2024, Claude 3.5/3.7/4 Sonnet/Opus in 2024–2025, etc.). The company has a strong public emphasis on AI safety research ([[constitutional-ai|constitutional AI]], [[mechanistic-interpretability|mechanistic interpretability]], RLAIF).

In this wiki, Anthropic is the source of the [[emotion-concepts-anthropic-2026|Emotion Concepts]] interpretability paper (2026) and is also the developer of Claude — including the Claude model used to maintain this very wiki.

## Notable publications / projects in this wiki

- [[emotion-concepts-anthropic-2026]] (will be linked at that ingest)
- Constitutional AI (Bai et al. 2022) — not yet in this wiki
- Sparse autoencoders / mechanistic interpretability work — not yet in this wiki

## Key researchers (founders)

- [[dario-amodei]] — CEO
- [[tom-brown]] — co-founder
- [[jared-kaplan]] — co-founder
- [[sam-mccandlish]] — co-founder

## References

- [[gpt-3-brown-2020]]

## Threat intelligence reporting

Anthropic publishes periodic reports on misuse of Claude: March 2025, August 2025, November 2025 (an AI-orchestrated espionage campaign), a February 2026 distillation disclosure, and the [[anthropic-threat-intelligence-report-september-2026|September 2026 report]] held here. The reports use internal **GTG** designators ("Generative Threat Groups") for actors, publish indicators of compromise, and describe the safeguards changed in response. The September 2026 report covers seven harm areas: [[ai-enabled-cyber-operations|cyber]], [[influence-operations|influence]], surveillance, conventional weapons, biological misuse, scams, and [[illicit-distillation]]. It also notes that Anthropic's Frontier Red Team built new evaluations for tactical intelligence targeting and conventional weapons development.

## Agent evaluation guidance

[[demystifying-evals-for-ai-agents-anthropic-2026|"Demystifying evals for AI agents"]] (2026-01-09) is among the most-shared practitioner guides on [[agent-evaluation]], supplying the task/trial/grader/transcript/outcome vocabulary now used across the tooling ecosystem.

## Related 2026 sources

- [[agentic-coding-trends-report-anthropic-2026]] — the 2026 predictions report — useful for its descriptive findings (AI used in ~60% of developer work but only **0–20% fully delegated**, and ~27% of AI-assisted work is newly viable rather than accelerated), with the caveat that it carries no methodology section and every case study is a customer
- [[ai-native-sdlc-playbook-anthropic-2026]] — the operational counterpart to the Trends Report: six stages, a versioned artifact chain that doubles as the audit trail, and leading/lagging measures for every play. Makes no quantitative claims at all
- [[building-with-claude-sonnet-5-5-osmani-2026]] — Osmani's Sonnet 5.5 launch guide: spec sheet, pricing, six breaking migration changes from Sonnet 5
- [[getting-the-most-out-of-opus-5-5-osmani-2026]] — Osmani on prompting Opus 5.5: define done, drop "think hard", stop rules in CLAUDE.md, subagents
- [[what-a-task-costs-on-opus-5-5-osmani-2026]] — Worked cost model for an agentic Claude Code task on Opus 5.5; what the "40% cheaper" claim actually means
- [[how-we-made-claude-ai-3x-faster-wang-2026]] — Anthropic case study: a Claude-driven two-week sprint made claude.ai ~3x faster using measurable proxies
- [[automating-eval-design-and-hillclimbing-martin-2026]] — Lance Martin on eval design and overfit-safe hill-climbing with the claude-api skill
- [[claude-dev]] — claude.dev, Anthropic's developer blog; hub of the posts ingested from it
- [[claude-opus-5-5]] — Claude Opus 5.5 (Sep 2026): cheaper tokens and cache reads, default effort medium
- [[claude-sonnet-5-5]] — Claude Sonnet 5.5 (2026-09-28): 1M context, same prices as Sonnet 5, stricter thinking semantics

## Lineage sources

- [[building-effective-agents-anthropic-2024]] — Building effective agents (Dec 2024)
- [[effective-context-engineering-anthropic-2025]] — Effective context engineering (Sep 2025)
