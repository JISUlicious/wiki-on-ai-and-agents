---
title: Red-Teaming (LLMs)
type: concept
created: 2026-05-14
updated: 2026-05-14
sources:
  - ganguli-red-teaming-2022.md
status: complete
importance: high
---

# Red-Teaming (LLMs)

Red-teaming in the LLM context means **adversarial probing** of a deployed or pre-deployment language model to discover harmful or undesirable behaviors. Borrowed from cybersecurity (where red teams simulate attackers against defenders), it became a standard pre-deployment practice for frontier LLMs after [[red-teaming-ganguli-2022|Ganguli et al. (2022)]]'s systematic study and dataset release.

## Why it's needed

LLM safety/alignment trained via [[rlhf]] or [[constitutional-ai|Constitutional AI]] is **not robust** out of the box. Adversarial prompts can elicit harmful outputs the model would refuse under naive phrasing — through obfuscation, hypothetical framing, role-play, encoding tricks, multi-turn manipulation, etc. Red-teaming maps the failure surface so it can be patched (via additional preference data, output filtering, or system-level mitigations).

## Standard practice (post-Ganguli 2022)

- **Recruit human red-teamers** — sometimes general crowdworkers, often domain experts (chemistry, biology, cybersecurity, child safety, etc. for high-stakes models).
- **Provide structured instructions** — what kinds of harm to attempt; what categories to log.
- **Log all attempts and outputs** — structured datasets enable analysis, replay, and benchmarking.
- **Iterate** — feed findings into the next round of training (RLHF preference data, constitutional principles, hard-coded filters).
- **External red-teams** — frontier labs now contract with external red-team firms (METR / ARC Evals, Apollo Research, etc.) for pre-deployment evaluations of dangerous capabilities (cybersecurity, bioweapons, autonomous replication).

## Findings that became canonical

From Ganguli 2022 specifically:

- **RLHF models get harder to red-team as they scale**; plain-prompted models do not. This is one foundation of the "post-training matters" view.
- **Subtle harms dominate**: most successful attacks elicit not overtly offensive content but more subtle problems — manipulation, deception, misleading advice. These are also harder to filter automatically.
- **Diminishing returns**: after some iterations of red-team-then-train, marginal attacks get progressively harder to find.

## Regulatory / norm consequences

By 2024, red-teaming is **required** by:
- US Executive Order on AI (Oct 2023) for frontier models above certain compute thresholds.
- EU AI Act (2024) for general-purpose AI models with systemic risk.
- Voluntary commitments from major labs (OpenAI, Anthropic, Google DeepMind, Microsoft, Meta).

External evaluation organizations (METR, AISI, NIST AISIC) have emerged to provide independent assessments.

## References

- [[red-teaming-ganguli-2022]]
