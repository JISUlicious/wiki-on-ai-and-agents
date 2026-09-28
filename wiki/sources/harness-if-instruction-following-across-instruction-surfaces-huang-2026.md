---
title: "Harness-IF: Evaluating Instruction Following Across Instruction Surfaces in Coding Agents"
type: source
created: 2026-08-17
updated: 2026-08-17
sources: []
arxiv_id: "2608.11727"
year: 2026
authors:
  - Zining Huang
  - Haoran Que
  - Hong Zeng
  - Ge Zhang
  - Zuo Wang
  - Jin Chen
  - Haodong Wang
  - Zhongfei Hou
  - Changxin Pu
  - Shen Yan
  - Wenhao Huang
tags:
  - 2026
status: complete
importance: high
---

# Harness-IF: Evaluating Instruction Following Across Instruction Surfaces in Coding Agents

**Zining Huang\*, Haoran Que\*, Hong Zeng\*, Ge Zhang, Zuo Wang, Jin Chen, Haodong Wang, Zhongfei Hou\*, Changxin Pu, Shen Yan†, Wenhao Huang** ([[bytedance-seed|ByteDance Seed]]; [[tsinghua-university|Tsinghua University]]; [[peking-university|Peking University]] — \*work done at ByteDance Seed, †corresponding) — [arXiv:2608.11727](https://arxiv.org/abs/2608.11727), cs.AI, 12 Aug 2026 (dated 15 Jul 2026). Preprint; release prepared under Apache-2.0 / CC-BY-4.0.

## Summary

The paper's opening line is its whole thesis: **"When a coding agent obeys a rule, it may simply have been going to do that anyway."** Existing instruction-following benchmarks (IFEval, FollowBench, ComplexBench) put rules in the user turn and score the response; coding-agent benchmarks ([[swe-bench]], [[terminal-bench]]) collapse a whole trajectory into task success. Neither tells you *which operational rule an agent actually followed* during a long harnessed workflow, and neither distinguishes compliance from coincidence.

Harness-IF attacks both gaps at once with two ideas.

**(1) Instruction *surface* as an evaluation dimension.** A deployed coding agent reads a stack, not a prompt: harness default (HD, fixed), system prompt (SP), tool description (TD), skill description (SD), project file (PF — `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`), and user instruction (UI). Five of the six are configurable. A **controlled-relocation operator** holds a rule's *meaning* fixed while moving its *delivery* across surfaces, with surface-appropriate phrasings. The paper is careful to distinguish this from **instruction hierarchy** — hierarchy asks which privileged source should win a conflict; surface asks whether delivery channel changes compliance at all.

**(2) Against-Prior Accuracy (AP-Acc).** Rules are labeled by whether they *oppose* the agent's unprompted default, observed via a **9-model zero-injection probe** (re-run the same task with the target rule withheld) where that evidence is recoverable, and curated otherwise. AP-Acc scores only the against-prior subset. It is explicitly framed as a **behavioral stratification, not a claim about training provenance**.

The results are consistent and not subtle. Across **12 frontier model builds**, accuracy spans **72.1–85.9%** while AP-Acc spans **66.1–78.6%**; **every single model is worse on against-prior rules**, by **3.6 to 7.4 points (mean 5.81)**. The inflation is model-specific, varying twofold across the cohort — so it does *not* cancel when builds are compared. And the separate conflict pilot delivers the finding most useful to anyone writing a `CLAUDE.md`: **precedence does not follow prompt depth.** System prompts, project files, and user instructions tie exactly for first, all ahead of tool and skill descriptions.

## Benchmark construction

**Library and panel.** A **642-atomic-constraint library** mined from a manual survey of public GitHub project-instruction and contributor documents (`CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, skill descriptions, tool schemas), atomized, normalized across surface phrasings, with reviewed LLM-assisted proposals filling under-covered taxonomy cells. Constraints must be **atomic, verifiable, non-trivial, and coding-agent-suitable**.

The evaluated coding panel is **60 multi-turn items** (retained from 80 candidates by quality and discriminativeness review), each injecting a pack of **25–35 rules** of which **10–27 are scorable** given the opportunities the item creates. Across the panel, **302 distinct library rules are instantiated and 256 receive at least one verdict.** Eight scenarios drawn from a 13-scenario library span backend, frontend, systems, data/ML, automation, security testing, tool orchestration, and technical documentation.

**Library composition** (three independent annotation axes; describes the 642-rule library, not the evaluated subset):

| Family | Share | | Modality | Share | | Prior | Share |
|---|---|---|---|---|---|---|---|
| Professional writing | 28% | | Require | 60% | | Against | **44%** |
| Output control | 18% | | Forbid | 13% | | Neutral | 38% |
| Code style | 18% | | Conditional require | 10% | | Align | 18% |
| Workflow | 11% | | Max limit | 7% | | | |
| Tool use | 9% | | Prefer | 6% | | | |
| Conditional | 9% | | Min limit | 3% | | | |
| Quantitative | 8% | | Allow | <1% | | | |

The 44% against-prior share is deliberate: "the panel cannot be passed by defaults alone." Professional-writing rules are exercised **only in the non-coding extension**, so the coding scorecard reports six of seven families.

**Surface admissibility.** A constraint is admissible on a surface when that surface's authoring role could plausibly carry it in deployment — "a branch-naming rule can sit in a project file or a system prompt, but not in a tool schema." When several surfaces are admissible, one is sampled uniformly. So the **main panel supports descriptive surface stratification, not a paired surface effect**; the controlled comparison lives entirely in the E0 pilot.

**Metrics.** With `z = 1` if agent *a* satisfies rule *r* in item *i*, and not-applicable verdicts excluded from the denominator:

- **Acc** — all eligible constraint instances weighted equally.
- **F-Acc** — filtered accuracy, restricted to item–rule pairs that produce *different* outcomes across the cohort.
- **DW-Acc** — discrimination-weighted, where `d_r` is the non-negative Pearson correlation between agents' pass rates on rule *r* and their overall accuracy.
- **AP-Acc** — restricted to the against-prior set P.

F-Acc and DW-Acc are explicitly **cohort-relative diagnostics** — both change if the cohort changes.

**Scale.** 60 items × 12 builds × 3 rounds = **2,160 runs**, producing **40,104 rule-level verdict rows**. Deterministic checks (regex, AST, cross-file, command-output) cover **13.3%** of eligible verdicts; a GPT-5.2 judge scores rubric constraints by **three-vote majority** and adjudicates hybrids, so **86.8% of rows involve the judge**. The like-for-like binary analysis contains **37,616 eligible verdicts, including 19,449 against-prior**.

## Results

**Table 2 — rule-level accuracy (%), 12 builds**, all four columns computed over the same eligible verdicts so Δ = Acc − AP-Acc is a like-for-like contrast:

| Model | Acc | F-Acc | DW-Acc | AP-Acc | Δ |
|---|---|---|---|---|---|
| Claude-Opus-4.7 | **85.9** | **79.3** | **88.5** | **78.6** | +7.3 |
| GPT-5.5 | 83.1 | 75.5 | 81.2 | 77.0 | +6.1 |
| Claude-Sonnet-4.6 | 82.5 | 73.9 | 82.2 | 78.5 | **+4.0** |
| Claude-Haiku-4.5 | 79.0 | 69.4 | 75.9 | 71.9 | +7.2 |
| GLM-5.1 | 78.8 | 67.9 | 75.6 | 75.2 | **+3.6** |
| Qwen-3.6-Max | 76.7 | 65.9 | 70.5 | 71.7 | +5.1 |
| Hy3 | 76.2 | 65.2 | 69.7 | 70.8 | +5.4 |
| Kimi-K2.6 | 76.1 | 65.0 | 70.3 | 70.0 | +6.1 |
| Gemini-3.1-Pro | 75.4 | 64.0 | 70.0 | 69.8 | +5.6 |
| MiniMax-M2.7 | 73.6 | 61.2 | 65.1 | 67.7 | +5.9 |
| Seed-2.0-Pro | 73.6 | 61.1 | 63.2 | 66.1 | **+7.4** |
| StepFun-3.5 | 72.1 | 59.1 | 62.5 | 66.2 | +5.9 |

Rows are ordered by unrounded Acc — MiniMax-M2.7 and Seed-2.0-Pro are separated by **0.03 points**, which is itself an argument against reading adjacent ranks.

What this table establishes:

- **The gap is universal and model-specific.** Mean **5.81 points**, range **3.6–7.4** — a twofold spread. Claude-Sonnet-4.6 and GLM-5.1 are notably "honest" (Δ = 4.0 and 3.6): Sonnet-4.6 ranks 3rd on Acc but 2nd on AP-Acc, essentially tied with Opus. Seed-2.0-Pro and Claude-Opus-4.7 are the most inflated.
- **Prior control reshuffles but does not overturn.** The top build is unchanged; **three adjacent rank pairs exchange (2–3, 4–5, 11–12)**.
- **Robustness.** A stricter common-support analysis retains **2,430 of 3,342 (item, round, rule) observation keys (72.7%)** on which every model produced a clean pass/fail; **item-clustered 95% intervals for the paired gap stay positive model by model**.
- **Shared difficulty ordering.** Correlating each model's per-rule pass-rate vector against the cohort mean over the **242 rules every model attempted** gives **0.57–0.89 (mean 0.80)** — models largely agree on what is hard, which is what makes the rank exchanges informative rather than noise.

**Failures are omissions, not excesses.** Grouping the **8,440 failures** by what the violated rule demands: **shortfall rules (require / minimum) absorb 77.1%**, overstep rules (forbid / cap) 20.8%, soft preferences 2.1%. Crucially the authors show this is **exposure, not propensity** — the panel has 27,306 shortfall instances against 8,443 overstep instances, and the two classes fail at *similar rates* (23.8% vs 20.8%). The practical implication is stated plainly: **verifiers tuned to detect excess output address only about a fifth of the failure mass.**

**Failure mass and difficulty by family:**

| Family | Share of failures | Pooled accuracy |
|---|---|---|
| Output control | 27.6% | **70.9%** (lowest) |
| Workflow | 26.3% | 76.0% |
| Code style | 15.7% | 79.2% |
| Conditional | 12.4% | 80.9% |
| Tool use | 9.5% | 81.6% |
| Quantitative | 8.6% | **82.6%** (highest) |

Output control combines the largest failure mass with the lowest pass rate — the most productive remediation target. It is the lowest-scoring family for **11 of 12 builds, and no build clears 79% on it**. Workflow's mass instead follows from its size in the panel (mid-range pass rate).

**By modality:** Preference **90.6%** > Conditional 79.7% > Numeric bounds 79.4% > **Commanding (require/forbid) 76.0%**. The authors read the preference result as default alignment — "preferences can match existing behavior, whereas commands often require departure from it," which is the AP-Acc thesis showing up on a second axis.

**E0 — surface precedence (the counterbalanced conflict pilot).** 916 runs, **nine older model builds**, four synthetic conflict pairs, **deterministic-only scoring**, counterbalanced with 458 runs per direction, 889 decisive. Mean conflict ranks (lower = higher precedence):

| Surface | Mean rank |
|---|---|
| System prompt (SP) | **2.22** (rank sum 20/9) |
| Project file (PF) | **2.22** (rank sum 20/9) |
| User instruction (UI) | **2.22** (rank sum 20/9) |
| Tool description (TD) | 3.78 |
| Skill description (SD) | 4.56 |

SP, PF and UI are an **exact three-way tie**. "By prompt depth we mean position in the assembled context, with the user turn last; a depth account predicts that later-placed instructions win. That the user instruction only ties for first is inconsistent with such an account." A pooled Bradley–Terry fit reproduces the ordering, and it survives separate direction fits, equal-cell weighting, all leave-one-pair/model-out fits, and four conservative assignments of the 27 errors; a crossed model-and-pair bootstrap preserves the **complete** ordering in **9,652/10,000 resamples**.

> [!warning] Do not over-read the precedence result
> **Only 6 of 9 individual-build fits reproduce the ordering exactly.** The authors interpret it as "a pooled cross-build tendency, not a universal hierarchy." E0 is also separate from the main panel — nine *older* builds, four *synthetic* conflicts, deterministic scoring only, never pooled with the 12-model coding results.

Still, the practical reading is clear and matters for anyone authoring agent instructions: **a rule placed in a tool or skill description is systematically weaker than the same rule in a project file** — and the project file is as strong as the system prompt. See [[skill-md-format]] and [[agent-skills]].

## Where it breaks

The paper's own reliability section is unusually candid, and the numbers there are the ones to carry forward:

- **Judge instability is the dominant uncertainty.** 86.8% of verdict rows involve an LLM judge, and under a **judge swap, agreement is 62.1% with κ = 0.163** on 116 paired clean verdicts. Inter-LLM Claude–GPT agreement is **κ = 0.4717** (n=65) — which the authors note "is not human validation." An earlier human-reference audit (under a *five*-vote rather than the released three-vote configuration) showed **69.0% agreement, κ = 0.515 over 919 rows**. Absolute levels are therefore instrument-specific; only cross-model comparisons, which share the instrument, carry the claims.
- **Test–retest ICC is 0.725 across models and 0.599 across agent–item cells.** Combined with the above: **"on common support, item-clustered intervals leave every adjacent model comparison unresolved."** The displayed order is a point ranking, full stop.
- **The prior gap size depends heavily on the subset.** On the fully deterministic subset the gap averages **13.09 points over 5,013 eligible verdicts** versus 5.81 panel-wide — because that subset over-represents pattern- and command-checked families. The *direction* is stable everywhere; the *magnitude* is not a single number.
- **Selection optimism, acknowledged.** The 60 items were retained from 80 using observed difficulty and discriminativeness, "so the reported panel may favor items that separated the pilot models."
- **Coverage.** 302 of 642 rules instantiated, 256 scored; professional writing (the single largest family at 28%) is exercised only outside the coding panel.
- **The instrument transfers, the ranking does not.** A separate 40-case non-coding panel over five domains (1,428 valid trajectories, same 12 builds) gives case-macro pass rates of **65.8–84.8%, led by GPT-5.5 rather than the coding leader** — and is never pooled with the coding results.
- **AP-Acc labels are partly curated**, not all observed: zero-injection evidence is used "where that evidence is recoverable, and curated otherwise," with label lineage and the probe/panel cohort overlap reported in the appendix.

## Related

- [[agentic-harness-engineering]] — Harness-IF measures a property the harness-optimization literature assumes: that rules installed on harness surfaces are actually obeyed. The TD/SD result says the two most common places to put agent rules are the two weakest.
- [[agent-skills]] and [[skill-md-format]] — skill descriptions rank **last** in E0 precedence (4.56), the sharpest practical warning in the paper for skill authors.
- [[context-assembly]] — surfaces are positions in the assembled context; the result that precedence does not track depth is a direct finding about assembly order.
- [[instruction-tuning]] — AP-Acc separates trained-in defaults from instruction compliance; a model can score well by having been tuned toward the same behavior the rule demands.
- [[agent-benchmark]] and [[agent-evaluation]] — the framing gap: coding-agent benchmarks collapse trajectories into task success, so rule compliance is invisible.
- [[swe-bench]] and [[terminal-bench]] — named as the task-success benchmarks Harness-IF complements (Table 1 scores them as single-surface with no rule-level scoring).
- [[llm-as-a-judge]] — 86.8% of verdicts are judge-mediated, three-vote majority, and κ = 0.163 under a judge swap; a strong cautionary datapoint for judge-scored benchmarks generally.
- [[code-generation]] and [[llm-agent]] — the setting: multi-turn coding agents over real repository fixtures.
- [[tool-use]] and [[mcp]] — tool descriptions as an instruction surface, and a weak one; the paper cites work on MCP tool descriptions affecting agent efficiency.
- [[claude-code]] — the reference agent configuration the six-surface model is drawn from; `CLAUDE.md` is the canonical project-file surface.
- [[model-or-harness-interaction-centric-taxonomy-raj-2026]] — its `model — owner · fault: model` **Instruction-Following Failure** mode is exactly what Harness-IF measures at scale; AP-Acc adds the missing control for "would it have done that anyway."
- [[agentic-context-engineering]] — surface placement as a context-engineering decision with measurable consequences.
- [[agentbench-liu-2023]], [[gaia]], [[osworld]], [[terminal-bench-benchmarking-agents-cli-merrill-2026]] — the agent-benchmark landscape it positions against.
- **No wiki page yet:** IFEval, instruction hierarchy, τ²-bench, AppWorld, SWE-bench Pro, AgentIF.

## References

- [arXiv:2608.11727](https://arxiv.org/abs/2608.11727)

## Related 2026 sources

- [[ai-native-sdlc-playbook-anthropic-2026]] — independent arrival at this paper's precedence finding: the playbook tells teams to put anything that must always hold in a hook rather than a skill, because skills are advisory
