---
title: "System One ecosystem sweep (2026-10-10)"
type: source
created: 2026-10-10
updated: 2026-10-13
sources:
  - systemonemodels-hub.md
  - microsoft-decision-1-register-2026.md
  - decision-models-microsoft-devto.md
year: 2026
venue: "systemonemodels.org (independent hub); The Register; DEV Community"
url: https://systemonemodels.org/
tags:
  - 2026
status: complete
importance: medium
---

# System One ecosystem sweep (2026-10-10)

**Sources** (all secondary):
- `sources/systemonemodels-hub.md`: [systemonemodels.org](https://systemonemodels.org/), "the independent hub for AI decision models", a community catalogue of vendors, open models, projects and recipes;
- The Register (2026-10-10, see [[microsoft-decision-1-2026]]);
- a DEV Community post, "Decision models are a new category, and Microsoft just joined it".

Vendor facts are as the hub reports them unless a primary source is linked. This follows [[system-one-ecosystem-sweep-2026-09-28]].

## What changed since 2026-09-28

**The frontier labs arrived:**

| Model / API | Vendor | Date | Basis | Price (input; output free) | Weights | Primary source in wiki |
|---|---|---|---|---|---|---|
| [[openai-decisions-api\|Decisions API]] | OpenAI | 2026-10-06 (public beta) | gpt-6-luna, multimodal | $0.10 / M | closed | [[openai-decisions-api-2026]] |
| [[microsoft-decision-1\|Microsoft-Decision-1]] | Microsoft | 2026-10-09 (public preview) | Qwen3.5-9B post-train | $0.042 / M | closed | [[microsoft-decision-1-2026]] |
| Clef / Clef-flash | Cloudflare | 2026-10-01 | 27B / 9B, multimodal | Workers AI | Apache-2.0 | — |
| Clef-omni | Cloudflare | 2026-10-09 | 30B MoE (3B active), audio and video | $0.15 / M | Apache-2.0 | — |
| pplx-decider-v1.1-27b | Perplexity | (not dated) | multimodal | $0.02 / M | Apache-2.0 | — |
| d1, d1-3B, d1-omni-600M | Liquid AI | hosted 2026-09-29; open 2026-10-07 | — | $0.04 / M (OpenRouter) | open (3B, 600M) | — |
| Solar Decide | Upstage | beta 2026-09-22 | Solar Mini 4, 512K context | — | closed | — |
| Mercury Decide | Inception (maker of the Mercury diffusion LMs, general knowledge) | (not dated) | — | $0.02 / M | closed | — |

**Other entries listed:**
- closed: Decider 1 (meraGPT), Drex 1.5 (Nace.AI, 9B, restrictive licence), Span-01 (Respan, a behaviour classifier for agent traces), Kai 1 (Hanzo), Celeris-1 Decision (optional one-sentence reason);
- open: [[kev\|Kev]], [[laya\|Laya]], CLM (contrastive), GLiNER2.5-Decide (a 340M encoder from Fastino);
- also: Tev1-4B-experimental (Together).

The Register adds:
- Strands Decider 2B (open);
- Snowflake;
- Surogate Rune;
- H2O-Lightning-4B (Qwen3.5-4B);
- "**more than 100 such models**".

The open, fully reproducible [[s1-decision-model-raghavan-2026|s1]] (Gemma 4 26B) appeared on 2026-10-08.

## Patterns

1. **Convergence on one interface.** Nearly every entry answers the same three shapes, choice, score and yes/no ("Noul" in TypeSafe's naming; "predicate" at OpenAI), with output that is free and priced per input token. Several serve TypeSafe's `/v1/systemone` route directly.
2. **Commodity bases.** Most are post-trains of open models: Qwen3.5 (Microsoft, H2O, Kev, Tev1, Nimble), Gemma 4 (s1). That is consistent with the wiki's finding that the mechanism is [[logit-scoring]] plus calibration fine-tuning, not a new architecture.
3. **Spread to multimodal and agent traces.** Clef-omni (audio and video), OpenAI (images), Span-01 (agent-trace behaviours).
4. **Calibration evidence is uneven.** s1, Jev, Intern-Decision and Liquid d1 publish ECE or Brier scores. Microsoft and OpenAI publish none and tell users to calibrate thresholds on their own data.
5. **Use cases** (the hub's recipe categories): routing and approval gates, retrieval and citation judging, injection and policy screening, feature extraction, fast move selection inside game, robot or agent loops.

## Caveats

- The hub is community-run and self-described as independent. Some vendor rows may be self-submitted.
- Dates and prices change fast. Only the OpenAI and Microsoft rows and s1 have primary sources in the wiki.
- Several names (Quyet-1.0-Large, Intern-Decision, Surogate Rune) are mentioned without any primary source.

## Concepts & entities

[[system-one-model]] · [[openai-decisions-api]] · [[microsoft-decision-1]] · [[kev]] · [[laya]] · [[jev]] · [[calibration]] · [[logit-scoring]] · [[dual-process-theory]]

## Lineage sources

- [[ollama-decision-models-2026]] — Clef, Clef-flash and Clef-omni Apache-2.0 confirmed from HF metadata; Tev1 declares no licence
