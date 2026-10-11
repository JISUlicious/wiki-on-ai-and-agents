---
title: Microsoft-Decision-1
type: entity
created: 2026-10-10
updated: 2026-10-10
sources:
  - microsoft-decision-1-foundry-2026.md
  - microsoft-decision-1-register-2026.md
  - microsoft-decision-1-orcarouter-no-benchmarks.md
family: qwen
predecessor: "[[qwen]]"
release_date: 2026-10-09
status: draft
importance: medium
tags:
  - model
---

# Microsoft-Decision-1

[[microsoft|Microsoft]]'s decision model, a [[system-one-model]] in all but name. It was announced on the Microsoft Foundry Blog on **2026-10-09**. Microsoft calls it "public preview"; secondary sources say "GA on 2026-10-08".

**Basis:** a post-train of **Qwen3.5-9B**. Microsoft says future versions will be rebased on OpenAI and MAI models.

**Contract:**
- text in, up to 32K tokens;
- yes/no, multiple-choice, rating, classification and rubric questions;
- a probability per option, with an abstention option supported;
- no generated text;
- input-only pricing ($0.042/M, reported).
- Foundry-only: no weights, no fine-tuning, no batch.

**Claims and evidence:**
- Xbox Research: quality "competitive with GPT-5" while running 80–100× faster on 10,000+ feedback items.
- Relayed vendor benchmarks: 83.5% average accuracy on 36 benchmarks, and 2.8× faster than Jev.
- **No published ECE or Brier score.** Microsoft tells users to validate calibration on their own data.

## Lineage

| | |
|---|---|
| Inherited | Qwen3.5-9B's knowledge and multilingual base. Post-training kept 25 supported languages. |
| Changed | Single-pass option scoring with probabilities, and the abstention design |
| Peers | [[jev]], [[openai-decisions-api]], [[kev]], [[nimble]] (all Qwen3.5-family or closed) |

## References

- [[microsoft-decision-1-2026]]
- [[system-one-ecosystem-sweep-2026-10-10]]
