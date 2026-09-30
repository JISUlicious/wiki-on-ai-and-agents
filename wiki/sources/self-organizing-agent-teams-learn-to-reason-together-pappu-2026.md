---
title: "Self-Organizing Agent Teams Learn to Reason Together"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - self-organizing-agent-teams-learn-to-reason-together-pappu-2026.md
arxiv_id: "2609.22682"
authors:
  - Aneesh Pappu
  - Mirac Suzgun
  - Yongchan Kwon
  - Federico Bianchi
  - Batu El
  - Mykel J. Kochenderfer
  - Hancheng Cao
  - James Zou
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Self-Organizing Agent Teams Learn to Reason Together

**Aneesh Pappu, Mirac Suzgun, Yongchan Kwon, Federico Bianchi, Batu El, Mykel J. Kochenderfer, Hancheng Cao, James Zou** ([[stanford-university|Stanford]]; Together AI; Emory) — [arXiv:2609.22682](https://arxiv.org/abs/2609.22682), v1, cs.AI, 19 Sep 2026. Preprint; no venue.

## Summary

SAT learns **how a fixed, heterogeneous team of models should organize a conversation**, not how to split the problem. A strategy is a program in a small DSL: an ordered list of conversational *phases*. Each phase specifies who participates, the number of rounds, and whether information flows locally or through a broadcast summary. Phases carry phase prompts, and the strategy carries persistent per-agent roles and shared teamwork norms. Offline, one designated member reflects on archived collaborations (with model identities blinded) and mutates strategies. A separate audit strips answer-specific content, and validation probes reward cross-problem transfer. A coverage-greedy step then freezes a bank of up to 10 strategies. At test time every strategy runs on the new problem to produce "certificates", and one member acts as judge and picks the answer.

The paper's contribution is the **baseline ladder**. It compares against the strongest member; against compute-matched single-agent inference, including **linearization**, where the strongest member serially plays every role and phase of the same learned strategies at about the team's total budget; and against a **routing oracle**, a perfect per-problem selector over members' independent answers. Beating the oracle implies interaction produced answers that no member produced alone.

## Key points

- **Math/physics team** (o3-mini, Claude Sonnet 4, DeepSeek-V3; learned from **15 AIME-2024 problems**; means over 3 seeds):

| Method | AIME24 test | AIME25 | AIME26 | HMMT26 | TQA-phys | Avg |
|---|---|---|---|---|---|---|
| Best member | 64.4 | 40.0 | 42.2 | 26.3 | 71.1 | 48.8 |
| Self-consistency K=10 ‡ | 75.8 | 44.4 | 53.4 | 31.9 | 71.5 | 55.4 |
| **Linearization (o3-mini) ‡** | 74.0 | 43.3 | 68.5 | **39.4** | 68.4 | 58.7 |
| Mixture of Agents | 75.6 | 46.7 | 57.8 | 32.3 | 74.0 | 57.3 |
| Homogeneous team (3× o3-mini) ‡ | 66.7 | 46.7 | 60.0 | 36.4 | 70.2 | 56.0 |
| **SAT** | **84.7** | **60.8** | **71.2** | **39.4** | **77.2** | **66.7** |
| Routing-oracle coverage | 73.3 | 51.1 | 57.8 | 36.4 | 76.6 | 59.0 |

  ‡ = compute control. SAT beats matched linearization by **8.0 points on average** but **ties it on HMMT 2026**. The homogeneous team running the same strategies scores 56.0, so model heterogeneity carries much of the gain.
- **Knowledge/logic team** (Gemini-2.5-Flash, Llama-4-Maverick, GPT-4.1; learned from 25 GPQA problems): SAT averages **72.8 vs 72.1 for linearization**, only 0.7 points at matched compute. It trails Mixture of Agents on MMLU-Pro (82.4 vs 84.3) and the homogeneous team on BBEH (56.0 vs 58.7). It stays **below routing-oracle coverage (79.6)** even though its own candidate pool covers 87.9%. Generation outran selection.
- **Demonstrability predicts the gain.** Demonstrability is measured as how reliably a 10-model panel picks correct over incorrect team reasoning. Across 8 benchmarks its rank tracks SAT's gain over the best member at **Spearman ρ = 0.90 (p = 0.005)**. This is post hoc and correlational, with n = 8.
- Worked example (HMMT 2026 P6, all three members initially wrong): o3-mini supplies the invariant but miscounts, DeepSeek repairs the count, and Claude audits.

## What it does not show

- **Yes, it has a matched-compute baseline**, rare in this cluster, and the result is mixed. The gain over matched single-agent inference is large on math (+8.0) and near zero on knowledge and logic (+0.7). The match is "approximately the team's total inference budget", and the paper reports no tokens.
- The routing oracle takes **one** independent answer per member, while SAT runs 10 strategies. Part of "exceeding the oracle" may be extra samples rather than interaction. Self-consistency K=10 (55.4) controls for this for the strongest member only. Training sets are tiny (15 and 25 problems), and the members are deliberately older models chosen to leave headroom.

## Concepts & entities

- [[multi-agent]] — directly tests whether a team beats selection among its members. Compare [[reaching-agreement-among-reasoning-llm-agents-ruan-2025]] and [[diversity-collapse-in-multi-agent-llm-systems-chen-2026]]: debate gains are largely recoverable by picking among initial answers.
- [[single-agent-vs-multi-agent-tran-2026]] · [[scaling-behavior-single-llm-multi-agent-systems-li-2026]] — prior single-vs-multi comparisons. SAT adds the linearization control.
- [[self-consistency]] · [[inference-time-compute]] — the compute-matched controls.
- [[llm-as-a-judge]] · [[verifier]] — the selection bottleneck (87.9% coverage vs 72.8% accuracy).
- [[agentic-harness-engineering]] · [[self-improving-agent]] — the learned object is an organizational harness, evolved by reflection and frozen before deployment.
- [[aime]] · [[gpqa]].

## References

- [arXiv:2609.22682](https://arxiv.org/abs/2609.22682)
