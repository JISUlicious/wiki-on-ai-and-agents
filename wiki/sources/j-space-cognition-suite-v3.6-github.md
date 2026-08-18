---
title: "J-Space Cognition Suite V3.6 (GitHub repository)"
type: source
created: 2026-08-19
updated: 2026-08-19
sources:
  - https://github.com/Tiger3807861189/J-Space-Cognition-Suite-V3.6
authors:
  - Tiger3807861189
year: 2026
venue: "GitHub / Zenodo (DOI 10.5281/zenodo.21977271)"
introduces:
  - [[j-space-cognition-suite]]
status: complete
importance: medium
tags:
  - 2026
---

# J-Space Cognition Suite V3.6 (GitHub repository)

**Repository**: [Tiger3807861189/J-Space-Cognition-Suite-V3.6](https://github.com/Tiger3807861189/J-Space-Cognition-Suite-V3.6) · Apache-2.0 · created 2026-07-22, v3.6.1 released 2026-08-17 · ~2,148 stars / 125 forks at ingest · Zenodo concept DOI `10.5281/zenodo.21971181`

An [[agent-skills|Agent Skill]] package operationalizing [[global-workspace-in-language-models-anthropic-2026|Anthropic's J-space research]] as an inference-time protocol. Ingested by inspection of the repository at commit depth 1 on 2026-08-19. See [[j-space-cognition-suite]] for the assessment; this page records what the artifact contains.

## Contents

596 KB, 18 markdown files, 3 Python files, no dependencies.

```
j-space/
  SKILL.md              16 KB   entry: premise, task classification, routing
  modules/              ~98 KB  9 selectively-loaded modules
  references/           ~75 KB  exemplars, induction-playbook, j-space-science
  scripts/jspace.py     30 KB   optional ledger controller (stdlib only)
  scripts/verify_suite.py  10 KB integrity check
tests/test_jspace.py    19 KB
```

The nine modules map five-for-five onto the paper's functional properties of conscious access — `introspection` (verbal report), `directed-focus` (directed modulation), `deep-reasoning` (internal reasoning), `broadcast` (flexible generalization), `capacity` (selectivity) — plus `self-monitoring`, `shorthand`, `markers`, and `empirics` from secondary evidence bases.

Three operating passes: `fast` (nothing loads), `full` (one or two modules plus a pre-delivery `ship` check), `loop` (ledger, seams, checkpoints, register audit, recovery).

## The science digest

`references/j-space-science.md` (35 KB) is a structured digest of the primary source plus ~20 secondary sources. Claims recorded there that match the wiki's own page on the paper: the J-space is emergent rather than designed; almost entirely **verbal** in format; carries **under 10%** of activation variance in any layer; corresponds to roughly one or two coherent ideas at a time; and is **structured in depth** across a sensory band, a workspace band at approximately layers 38–92 on a 0–100 scale, and a late motor band.

It also records the privilege test — a concept's J-space component carries only ~6–7% of its representational variance yet swapping along it succeeds ~59% of the time, against ~5% for the far larger non-J-space component — and the Soccer/Rugby substitution and injected-`lightning` results.

**Citation integrity**: all nine arXiv identifiers cited in `§15 Primary sources` were checked against `citation_title` during this ingest and **all nine resolve to real papers with matching titles**:

| ID | Title |
|---|---|
| 2605.14186 | LLMs Know When They Know, but Do Not Act on It: A Metacognitive Harness for Test-time Scaling |
| 2607.05775 | Beyond the Leaderboard: A Synthesis of Tool-Use, Planning, and Reasoning Failures in LLM Agents |
| 2607.00692 | Self-GC: Self-Governing Context for Long-Horizon LLM Agents |
| 2607.08964 | Long-Horizon-Terminal-Bench |
| 2510.27338 | Reasoning Models Sometimes Output Illegible Chains of Thought |
| 2509.15541 | Stress Testing Deliberative Alignment for Anti-Scheming Training |
| 2502.18600 | Chain of Draft |
| 2503.05179 | Sketch-of-Thought |
| 2309.06275 | Re-Reading Improves Reasoning in Large Language Models |

`THIRD_PARTY_NOTICES.md` separates Apache-2.0 suite text from quoted third-party material rather than relicensing it wholesale.

## Reported benchmarks

Self-reported, on DeepSeek V4-Flash-0731 configured against DeepSeek Harness minimal mode at `max` reasoning effort, `temperature = 1.0`, `top_p = 0.95`.

| Benchmark | Bare model | + J-Space V3.6 | Δ |
|---|---:|---:|---:|
| HLE (no tools) | 37.8 | 45.5 | +7.7 |
| HLE (tools) | 51.5 | 60.6 | +9.1 |
| Terminal Bench 2.1 | 82.7 | 87.1 | +4.4 |
| NL2Repo | 54.2 | 70.2 | **+16.0** |
| CyberGym | 76.7 | 81.7 | +5.0 |
| DeepSWE | 54.4 | 67.4 | +13.0 |
| Toolathlon-Verified | 70.3 | 77.7 | +7.4 |
| Agents' Last Exam | 25.2 | 30.1 | +4.9 |
| AutomationBench (public) | 25.1 | 31.7 | +6.6 |

Plus an efficiency table: speed 0.43 → 1.09 (2.53×), token cost 2.63 → 1.19 (2.21×).

> [!warning] The control and treatment come from different environments
> The README states that comparator values "retain the evaluation contexts published by their respective providers," while J-Space results "were collected within the project's available evaluation environment." The baseline is therefore a **vendor model card** and the treatment a **self-run harness** — not a matched A/B. The efficiency rows record "one evaluation run" each with no seeds or variance, under "fixed, uniform scaling coefficients" that make only the ratio interpretable.
>
> One README sentence concedes more than it appears to: *"J-Space tends to encourage more initiative and goal-directed exploration, making accessible artifacts and execution traces relevant to observed outcomes."* Read plainly, the assisted condition may have reached evaluation-environment artifacts the baseline did not.
>
> Every claimed delta (+4.4 to +16.0) falls inside the range that **harness substitution alone** produces on a fixed backbone — 15.36 points in [[dataspace-benchmarking-data-agents-li-2026]], 11–15 points in [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]]. The table cannot separate the skill from the scaffold.

## Code

`scripts/jspace.py` — argparse/codecs/json/os/re/sys/tempfile/time only. **No network, no subprocess, no eval/exec, no pickle.** Writes a single `.jspace/` directory in the task workspace via atomic temp-file replacement. Commands: `note` (goal, next action, hub slots, checkpoints with named verifier, open questions), `seam`, `ship`, `resume`.

Notably its state design is **slot-based rather than append-only** — a fixed hub with explicit `--core-slot` swap, one next action replaced rather than accumulated, questions closed against a recorded checkpoint. That is closer to consolidation than the designs [[catastrophic-remembering]] measures, though untested over long horizons.

## Related

- [[j-space-cognition-suite]] — the entity page and assessment.
- [[global-workspace-in-language-models-anthropic-2026]] — the primary research it reads from.
- [[agent-skills]] · [[skill-md-format]] · [[what-keeps-agent-skills-from-being-reusable-zhang-2026]]
- [[harness-vs-model-attribution]] — why the benchmark table is unreadable as a causal claim.

## References

- [[global-workspace-in-language-models-anthropic-2026]] · [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] · [[dataspace-benchmarking-data-agents-li-2026]] · [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]]
