---
title: "Beyond the Leaderboard: A Synthesis of Tool-Use, Planning, and Reasoning Failures in Large Language Model Agents"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - beyond-the-leaderboard-albayaydh-2026.md
arxiv_id: "2607.05775"
year: 2026
authors:
  - Wael Albayaydh
  - Rui Zhao
  - Ivan Flechais
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Beyond the Leaderboard: A Synthesis of Tool-Use, Planning, and Reasoning Failures in Large Language Model Agents

**Wael Albayaydh, Rui Zhao, Ivan Flechais** (University of Oxford, Department of Computer Science) — [arXiv:2607.05775](https://arxiv.org/abs/2607.05775). Venue not stated.

## Summary

A synthesis of **27 benchmark, taxonomy, and audit papers (2023–2026) covering 19 distinct named benchmarks**, organized into a cross-cutting taxonomy of LLM agent limitations. The authors' framing is that headline leaderboard gains obscure a set of failure modes that recur across otherwise unrelated evaluation efforts, and that no prior survey had integrated tool use, planning, long-horizon reasoning, multi-agent coordination, safety, and *measurement validity* into a single taxonomy.

The **six failure clusters** map onto successive stages of an agent's reasoning-to-action pipeline: (1) tool invocation and parameter-level errors, (2) planning and constraint-satisfaction failures, (3) long-horizon degradation from context accumulation, (4) multi-agent coordination failures, (5) safety and security failures under adversarial or underspecified conditions, and (6) measurement validity problems. The derivation is disclosed rather than assumed: they listed every named error category across all 27 papers (>60 initially), then iteratively split and merged into themes. Their validity argument is convergence — MAST's three top-level categories map onto clusters 4.4/4.6, and ToolScan's seven-type and a REST-API framework's four-type tool taxonomies overlap with cluster 4.1, despite unrelated annotation processes.

Notably for a failure survey, Section 5 is an explicit account of **where agents genuinely improved** — single-turn tool selection, short-horizon web navigation, narrowly scoped coding, and safety fine-tuning as a targeted lever — and the paper's closing argument is epistemic rather than pessimistic: interpreting agent progress requires distinguishing genuine capability gains from *corrections of earlier measurement error*, since both push the same numbers in the same direction.

## Key points

**Three structural patterns claimed across clusters:** failure compounds **non-linearly** with task length; sub-skill competence does **not** compose into end-to-end success; and additional scaffolding does **not** uniformly improve reliability.

| Finding | Number | Source cited |
|---|---|---|
| TravelPlanner multi-constraint planning | **0.6%** (GPT-4) → ~10% (o1-preview) → ~65% (best verifier-augmented config) | Xie et al. 2024; arXiv:2404.11891 |
| METR time horizon | near-100% on <~4-min human tasks; **<10%** on >~4-hr tasks; 50%-reliability horizon doubling ~every 7 months | Kwa et al. 2025 |
| Long-context web agents | 40–50% short-horizon → **<10%** when the same task is embedded in a longer history | arXiv:2512.04307 |
| LoCoBench-Agent | ~**3–5% efficiency loss per 50% increase** in conversation length (non-linear) | arXiv:2511.13998 |
| MAST multi-agent | 14 failure modes / 3 categories; ~**42%** specification, ~**37%** inter-agent misalignment, ~21% verification (Cohen's κ = 0.88) | Cemri et al. 2025 |
| ToolEmu | safest evaluated agent still had severe-outcome failures in **23.9%** of cases | Ruan et al. 2023 |
| InjecAgent | ~**24%** attack success (standard ReAct GPT-4) → ~**47%** with reinforced prompt; **>80%** for some open models without safety fine-tuning | Zhan et al. 2024 |
| SWE-bench leakage correction | one SWE-agent config fell from ~**12.5% to ~4%** after leakage/test-suite correction | Aleithan et al. 2024 |
| Test augmentation | stronger auto-generated unit tests changed leaderboard **rankings in ~25–41%** of cases | Yu et al. 2025 (UTBoost) |
| WebArena | ~14% at introduction → **61.7%** ~18 months later (specialized enterprise scaffold); human ~78% | Zhou et al. 2024; Marreed et al. 2025 |
| BFCL structured output | dedicated function-calling modes produced **~77.5 vs. ~21** incorrect function calls on average vs. prompting, in the multiple-call category | Patil et al. 2025 |
| HAL | 21,000+ rollouts, 9 models × 9 benchmarks, ~**$40,000**; higher reasoning effort **reduced** accuracy in the majority of runs | Kapoor et al. 2025 |

- **The verification gap is named as a cause, not a side note.** A test-time-scaling study (arXiv:2602.18998) found *neither* sequential nor parallel scaling reliably improved agent performance, attributing this to a "context ceiling" plus a "verification gap" — the agent cannot tell which sampled trajectory actually solved the task.
- **"No-recovery bottleneck"** (arXiv:2603.06870) is offered as the mechanism behind non-linear compounding: once an agent commits to a wrong intermediate state deep in a trajectory, current architectures largely cannot detect and roll back, so one early mistake propagates. METR's own attribution of horizon growth to *reliability and error recovery* rather than raw reasoning is cited as consistent.
- **Brittleness is at the reasoning–action interface**, not in task understanding: tool errors concentrate in parameter specification and function naming. NoisyToolBench's finding is that when a required parameter is missing, models fabricate a plausible value rather than ask a clarifying question.
- **Safety and capability are only partially correlated** — an agent can parse schemas and satisfy constraints well while remaining highly injectable, since neither depends on reasoning accuracy so much as on default posture toward ambiguity and untrusted content.
- **Explicitly self-limiting:** the authors state this is a *narrative* synthesis, not a PRISMA-style systematic review; Table 2 compiles others' reported results without independent reproduction; scope is English-language text/web tasks (robotics, embodied, voice excluded).
- An unusual inclusion: the authors bring their own prior human-centered privacy work (Albayaydh & Flechais, CHI '22 / USENIX Security '23 Distinguished Paper / '24) to surface a gap — agent-safety benchmarks including ToolEmu are structured around harms to the *instructing user*, not to non-consenting bystanders affected by the agent's actions.

## Concepts & entities

- [[agent-evaluation]] — the paper is a direct contribution to evaluation methodology: it argues for reporting constraint-level alongside plan-level success, partial progress alongside binary success, and cost-adjusted rather than raw accuracy.
- [[swe-bench]] — cluster 4.6's principal case study; the ~12.5%→~4% correction and the 25–41% ranking churn under stronger tests are the paper's sharpest measurement-validity evidence.
- [[verifier]] — the "verification gap" is cited as one of two causes for test-time scaling failing to help, and MAST's third failure category is verification/termination failure. Verification recurs as a bottleneck across independent clusters.
- [[hallucination-detection]] — the survey treats tool-call errors as *functional hallucination*, explicitly distinct from textual confabulation because it manifests as structurally plausible but semantically wrong actions.
- [[harness-vs-model-attribution]] — §7.4's "uneven returns to additional scaffolding" is the negative-result counterpart: undirected scaling of scaffolding surface area is unreliable, while targeted fixes to a diagnosed failure mode help.
- [[reflexion]] — cited in §2 as one of the three architectural developments (with Toolformer and ReAct) establishing the modern agent paradigm, via verbal self-critique without gradient updates.
- [[agentic-misalignment]] — cluster 4.5 covers the adjacent surface: well-formed but harmful actions arising from underspecification (ToolEmu) or injected instructions (InjecAgent), rather than from model goal conflict.

## References

- [arXiv:2607.05775](https://arxiv.org/abs/2607.05775)
