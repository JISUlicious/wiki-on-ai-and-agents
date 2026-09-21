---
title: Bespoke Labs
type: entity
created: 2026-09-21
updated: 2026-09-21
sources:
  - bespoke-nimble-github-2026.md
status: draft
importance: low
tags:
  - org
---

# Bespoke Labs

AI lab publishing small, task-specific open models and the data-curation recipes behind them. Its relevant line of work is **verifier-style models**: models fine-tuned to answer one narrow, typed question well rather than to generate text.

- **Bespoke-MiniCheck-7B** — fact-checking model that returns a single JSON tuple, `{"is_claim_supported_by_context": p}`. Both the serving approach and the data-curation method behind [[nimble|Bespoke-Nimble-9B]] are, by the team's own statement, derived from it.
- **Bespoke-Nimble-9B** — an open [[system-one-model|System One]]-style decision model, released 2026-09-18 as a response to [[jev|Jev]].

The house style is worth noting: the Nimble README documents its own limits, its narrow evaluation, and a bug it found mid-evaluation. That is unusually candid for a model release and stands in contrast to [[typesafe-ai|TypeSafe AI]]'s announcement of Jev.

## Related

- [[nimble]] · [[jev]] · [[system-one-model]] · [[calibration]]

## References

- [[bespoke-nimble-github-2026]]
