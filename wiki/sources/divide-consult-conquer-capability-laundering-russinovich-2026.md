---
title: "Divide, Consult, Conquer: Capability Laundering Through Aligned LLMs"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - divide-consult-conquer-capability-laundering-russinovich-2026.md
arxiv_id: "2609.15383"
authors:
  - Mark Russinovich
  - Blake Bullwinkel
  - Giorgio Severi
  - Cristian Ovadiuc
year: 2026
venue: "arXiv (cs.CR)"
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Divide, Consult, Conquer: Capability Laundering Through Aligned LLMs

**Mark Russinovich, Blake Bullwinkel, Giorgio Severi, Cristian Ovadiuc** ([[microsoft]]) — [arXiv:2609.15383](https://arxiv.org/abs/2609.15383), cs.CR, 14 Sep 2026.

> [!note] Scope of this page
> This page records the threat model, the measured uplift and the defensive recommendations, at the level of the paper's abstract and discussion. It deliberately omits method detail.

## Summary

Safety evaluation mostly asks whether an aligned model refuses a harmful request. This paper studies an attack in which no harmful request is ever sent. A weaker, unaligned model run locally keeps the harmful objective to itself, splits it into subproblems that each look like ordinary technical work, sends those fragments to an aligned frontier model in separate sessions, and recombines the answers outside the frontier model's view. The authors call this **capability laundering**, and the resulting gain **capability uplift**.

It differs from jailbreaking: the aligned model is never persuaded to do the harmful task, and it never produces the final artifact. Each exchange can pass per-request input and output classifiers on its own merits, because the decisive context never leaves the orchestrator.

## Key results

- **Cybersecurity.** On CyBench, a Gemma-4-31B orchestrator (see [[gemma]]) recovered 8 of 14 uplift candidates when consulting GPT-5.5, and 7 of 9 when consulting Opus 4.8. On BountyBench recovery was lower (3 of 9 with GPT-5.5).
- **CBRN.** On an expert rubric for a hypothetical attack chain, consultation raised the orchestrator's mean score from **62.3 to 83.1** out of 100. The authors stress this measures response quality, not operational success.
- **The orchestrator matters.** Recovery depended heavily on the orchestrator's own planning and integration ability. A different 30B orchestrator recovered none of its candidates, and a smaller Gemma recovered far fewer.
- **Disclosure.** The authors reported the findings to affected model providers before publication.

## Defensive implications (the paper's recommendations)

- **Composition-aware monitoring.** Assess a user's requests jointly rather than one by one. Keep the provenance of earlier requests and answers, and ask whether a new answer completes a suspicious chain. This has to reason over semantic dependencies, not surface wording, since the attack spreads the task across abstractions.
- **Two hard problems it raises.** The monitor must tell malicious composition apart from legitimate multi-step engineering. And its view is incomplete, because an attacker can spread consultations across accounts, providers or time. Cross-session aggregation still raises the attacker's cost.
- **Evaluate unalignment defenses by orchestration ability.** For open models, refusal rates are not enough. What matters is whether a modified model keeps the long-horizon planning needed to orchestrate outside help.

## What it does not show

Each experiment uses one orchestrator and one consultant channel; attacks spread across identities or providers are named as future work, not measured. Scores are rubric-based quality measures, not demonstrated real-world capability. No composition-aware defense is built or tested, so the proposed mitigations are unevaluated.

## Concepts & entities

- [[ai-safety]] · [[red-teaming]] — the evaluation gap: per-request refusal does not bound what many permitted requests can add up to.
- [[ai-enabled-cyber-operations]] — the cybersecurity half of the measured uplift.
- [[illicit-distillation]] — a related pattern of extracting frontier capability through many individually acceptable queries.
- [[prompt-injection]] — a different threat class: there the attacker subverts the model; here the model behaves as intended on every request.
- [[microsoft]] · [[gemma]]

## References

- [arXiv:2609.15383](https://arxiv.org/abs/2609.15383)
- Related monitoring work cited by the paper: Chen et al., "Monitoring decomposition attacks in LLMs with lightweight sequential monitors", [arXiv:2506.10949](https://arxiv.org/abs/2506.10949)
