---
title: OpenAI Decisions API
type: entity
created: 2026-10-10
updated: 2026-10-10
sources:
  - openai-decisions-api-guide-2026.md
release_date: 2026-10-06
status: draft
importance: medium
tags:
  - product
---

# OpenAI Decisions API

[[openai|OpenAI]]'s hosted typed-decision endpoint, `POST /v1/decisions`, in public beta from about **2026-10-06**. That date is from secondary coverage; the guide says GA is expected "in the coming weeks". **gpt-6-luna** is the only model.

- **Questions:**
  - **predicate**: the probability that a condition is true;
  - **choice**: a supplied value plus per-option probabilities and a confidence;
  - **score**: the probability-weighted mean of ordered levels.
- **Inputs:** text and images. Answers can be refusals.
- **Speed:** "about 10x faster than the Responses API."
- **Pricing:** $0.10 per 1M input tokens; no output or cache charges.
- **Compliance:** ZDR and HIPAA eligible; US and EU residency.
- **No calibration claim.** Users are told to set thresholds from labelled examples, weighing false positives against false negatives.

It is functionally a hosted [[system-one-model]], mapping onto Jev's Noul / Choice / Score. OpenAI's guide does not use the "System One" name.

## References

- [[openai-decisions-api-2026]]
- [[system-one-ecosystem-sweep-2026-10-10]]
