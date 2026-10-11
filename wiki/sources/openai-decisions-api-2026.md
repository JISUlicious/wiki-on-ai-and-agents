---
title: "OpenAI Decisions API (guide, public beta)"
type: source
created: 2026-10-10
updated: 2026-10-10
sources:
  - openai-decisions-api-guide-2026.md
authors:
  - "[[openai]]"
introduces:
  - "[[openai-decisions-api]]"
year: 2026
venue: OpenAI API docs
url: https://developers.openai.com/api/docs/guides/decisions
tags:
  - 2026
  - docs
status: complete
importance: high
---

# OpenAI Decisions API

**Source**: `sources/openai-decisions-api-guide-2026.md` ([developers.openai.com/api/docs/guides/decisions](https://developers.openai.com/api/docs/guides/decisions), fetched 2026-10-10)
**Publisher**: [[openai]]
**Date**: public beta. The 2026-10-06 start date is from secondary coverage (MarkTechPost, the systemonemodels.org catalogue); the guide itself is undated. It says: "we expect to GA in the coming weeks."

## Summary

"The Decisions API evaluates text, images, or both and returns typed answers about **10x faster than the Responses API**." It is a dedicated `POST /v1/decisions` endpoint. **`gpt-6-luna` is the only supported model.**

**Request:** `model`, `input` (shared evidence: text and/or images), and `questions`, each named and typed.

| Type | Use | Result |
|---|---|---|
| **predicate** | Check a condition (visible damage, passage relevance) | `probability` that it is true, 0–1 |
| **choice** | Pick one of your supplied values | `choice`, plus a `probabilities` array and a `confidence` field |
| **score** | Rate against ordered levels | the probability-weighted average of the level indices, which can fall between levels |

- **Refusals:** answers can come back as `refusal`.
- **Thresholds:** "Use labeled examples from your application to set thresholds… based on the cost of false positives and false negatives."
- **Positioning:** use Decisions for these answer types. Use **Structured Outputs** to generate an object or explanation, and **function calling** to request a tool call.
- **Pricing:** $0.10 per 1M input tokens with gpt-6-luna. "You pay only for input tokens: there are no cache-read, cache-write, or output-token charges." Regional and long-context multipliers apply.
- **Compliance:** ZDR and HIPAA eligible; US and EU data residency.
- **Voice:** a "Connect voice to Decisions" section exists.

## Mapping to the System One shape

| TypeSafe ([[jev]]) | OpenAI |
|---|---|
| Choice | choice |
| Score (probability-weighted level) | score |
| Noul (yes/no probability) | predicate |
| Free output, priced per input token | Free output, priced per input token |
| Single parallel pass | "about 10x faster than the Responses API" (no single-pass claim in the guide) |

The guide never uses the phrase "System One". Third-party catalogues file it as a hosted System One model.

## Why it matters

A frontier lab has shipped the [[system-one-model]] interface as a first-party endpoint, with multimodal input. Within three weeks the category went from one startup (TypeSafe, 2026-09-15) to both OpenAI (10-06) and Microsoft (10-09).

The guide makes no calibration claim; it tells users to set thresholds from labelled data. That is more cautious than TypeSafe's framing, and consistent with the audit findings in [[independent-jev-calibration-audits-2026]].

## Concepts & entities

[[openai-decisions-api]] · [[system-one-model]] · [[calibration]] · [[function-calling]] · [[openai]]
