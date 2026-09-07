---
title: "Evaluating Skills, Not Just Agents: Agentic Continuous Evaluation of Skills"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.20614"
venue: "Extended preprint of versions accepted at Agent Skills '26 (ACM CAIS 2026) and the KDD 2026 Workshop on Enterprise AI Agents"
authors:
  - Christopher Kevin
  - Narendran Raghavan
  - Jean-Francois Puget
  - Roshni Malani
  - Meghana Puvvadi
  - Moshe Abramovitch
  - Mohit Gupta
  - Rama Akkiraju
  - Subodh Prabhu
  - Yogesh Dangi
  - Wei Luo
  - Seong Hee Lee
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# Evaluating Skills, Not Just Agents: Agentic Continuous Evaluation of Skills

**Christopher Kevin\*, Narendran Raghavan, Jean-Francois Puget, Roshni Malani, Meghana Puvvadi, Moshe Abramovitch, Mohit Gupta, Rama Akkiraju, Subodh Prabhu, Yogesh Dangi, Wei Luo, Seong Hee Lee** ([[nvidia|NVIDIA]] — \*corresponding) — [arXiv:2608.20614](https://arxiv.org/abs/2608.20614), cs.AI, 20 Aug 2026. 15 pages; extended preprint incorporating versions accepted at **Agent Skills '26 (ACM CAIS 2026)** and the **KDD 2026 Workshop on Enterprise AI Agents: From Prototypes to Production** (oral). Open-source implementation: NVIDIA **SkillEvaluator**.

## Summary

Every shipping [[agent-skills|Agent Skills]] review gate today reads the *artifact*: frontmatter linting, structural rules, an [[llm-as-a-judge|LLM-judge]] rubric on the prose, a security scan. This paper's claim is that none of those answers the deployment question — *does this skill help a live agent finish the task, under the same model, sandbox and grading policy?* — and that the gap is not a hypothetical. It is measured twice, and the second measurement is the one that matters.

**ACES** (Agentic Continuous Evaluation of Skills) is a repository-native framework built on three principles: *write once, evaluate everywhere* (one evaluation contract across harnesses); *differential measurement* (paired with-skill vs. baseline runs isolate the marginal contribution); and *developer-guided evaluation* (author intent, expressed in an `EVAL.md`, outranks anything an LLM generates). Mechanically: an adapter reads a skill directory plus its `evals/evals.json` and emits **two** ephemeral containerized tasks per case — one with the target skill staged, one with it withheld — holding question, harness, model, workspace, fixtures, supporting skills, verifier and judge fixed. Both runs are normalized into **ATIF** (Agent Trajectory Interchange Format), a versioned JSON schema where a trajectory is an ordered list of steps carrying source, message, tool calls and observation. Six default metrics grade the ATIF trace. **Skill Lift** is the mean with-minus-baseline delta across the active metric set.

Two design details carry most of the methodological weight. **Reference decoys**: a naive baseline empties the workspace, which conflates the skill's *content* with the mere fact that any skill exists to be found, inflating lift. ACES keeps an author-configured support set (`api-debugger`, `log-triage`, `config-validator`, `release-planner`) fixed in *both* arms, so the agent must still perform discovery and routing and the delta measures what the target adds. **Isolation vs. group mode**: run the same skill alone (Lift_iso = content contribution) and amid the configured neighbours (Lift_grp = content + routing), and read the difference as a **routing premium** — a near-zero or negative premium flags a skill whose name and description fail to differentiate it from siblings, even when its isolated content is good. That is a directly actionable authoring signal, and it is the paper's sharpest idea.

The empirical result is a **positive but decomposed** effect. Across 947 paired cases, mean composite Skill Lift is 0.2134 — but the gains concentrate in *process* metrics (skill execution +0.3263, behavior check +0.2983, skill efficiency +0.2758) rather than final-answer accuracy (+0.1431). And on the 62 production skills with both scan and live data, **neither static score predicts live lift at all** (Spearman ρ = −0.0181 and −0.0266, both indistinguishable from zero).

## Key points

**Evidence inventory (Table 6)** — the filtering is reported in full, which is unusual and welcome:

| Evidence item | Count |
|---|---|
| Scanned skills | 145 |
| Unique live-eval skill variants | 89 |
| Scaling-study variants excluded | 25 |
| Production skills in main inventory | **64** |
| Production skills with scored paired cases | **58** |
| Primary agent harnesses | **4** |
| Production skill–agent cells / cells with scored cases | 201 / 177 |
| Paired trial files / run logs | 289 / 289 |
| **Scored paired task cases** | **947** |
| Condition runs / reward rows | 578 / 2,077 |
| Saved ATIF trajectories | 2,022 |
| Recorded agent steps / tool calls | 14,335 / 11,642 |
| Behavior observations | 9,091 |

All four numbers the wiki was asked to check verify exactly: **947 paired cases, 58 of 64 production skills, 4 primary harnesses, "Agent Trajectory Interchange Format."**

**The six default metrics** — three deterministic, three judged, each weighted 1/6 (an explicitly "inspectable diagnostic default," not a causal claim):

| Metric | Kind | Contract |
|---|---|---|
| `security` | deterministic | trace scan for destructive ops, leaked secrets, out-of-workspace access |
| `skill_execution` | deterministic, 4 sub-checks | activation (read the expected `SKILL.md`), script invocation, workflow order (read before execute), error recovery |
| `skill_efficiency` | deterministic, 2 sub-checks | routing (read *only* allowed skills) and tool efficiency (productive calls / total; waste = `--help` fishing, `ls` at wrong paths, installs during task time) |
| `accuracy` | LLM judge, T=0 | five binary criteria: right skill identified, right action, factual consistency with ground truth, task actually addressed, response actionable |
| `goal_accuracy` | RAGAS `AgentGoalAccuracyWithReference` (LLM fallback) | did the full conversation, tool calls included, achieve the stated goal |
| `behavior_check` | LLM judge, per-behavior | yes/no on each free-form `expected_behavior` string; score = fraction satisfied |

`expected_behavior` is the paper's smallest and most expressive mechanism — a plain sentence ("read `git-skill/SKILL.md` before executing", "confirmed destructive operation with the user"), composable without schema change, writable without a DSL, enforceable per-behavior. A security behavior is auto-appended when missing, so every case carries a baseline safety check.

**Headline live result (947 paired cases)**:

| Quantity | With skill | Baseline | Lift |
|---|---|---|---|
| Composite (6-metric mean) | 0.7460 | 0.5326 | **0.2134** (95% CI [0.1967, 0.2301]) |
| `accuracy` | 0.7760 | 0.6329 | 0.1431 |
| `goal_accuracy` | 0.6887 | 0.4720 | 0.2167 |
| Outcome-only (mean of the two above) | — | — | **0.1799** |

Composite lift is **positive in 689 cases, zero in 171, negative in 87** (median 0.1717) — the mean is not a small positive tail. The authors are explicit that the 947-case CI is descriptive and *not* 947 independent skills; clustering checks bootstrap over production skills ([0.1898, 0.2350]) and over skill–agent cells ([0.1880, 0.2385]), both positive and consistent.

**Where the lift lands (Figure 8)** — this is the paper's argument for trajectory grading:

| Metric | Mean paired lift |
|---|---|
| `skill_execution` | **+0.3263** |
| `behavior_check` | **+0.2983** |
| `skill_efficiency` | **+0.2758** |
| `goal_accuracy` | +0.2167 |
| `accuracy` | +0.1431 |
| Overall composite | +0.2134 |

Skill efficiency has the third-largest mean lift but is positive in **only 41.7%** of paired cases — a high-variance tradeoff where some runs buy correctness or workflow completion with efficiency. (The paper does not report a `security` lift value in text.)

**Cross-harness (Figure 10a)** — matched deltas against each harness's *own* baseline, explicitly not a model ranking, and on uneven coverage:

| Harness | Mean overall lift | Scored cells / paired cases |
|---|---|---|
| [[opencode\|OpenCode]] | **0.3611** | 34 / 211 |
| [[claude-code\|Claude Code]] | 0.2904 | 56 / 251 |
| Codex | 0.1264 | 50 / 259 |
| Terminus-2 | 0.0896 | 37 / 226 |

A 4× spread in measured lift for the same skills across harnesses. The Codex model sweep adds the corollary: with GPT-5.5 the **baseline rises sharply, so measured Skill Lift shrinks while absolute performance stays high**. The authors draw the right conclusion — skill value must be re-measured after model updates, because a stronger agent needs less help.

**Routing stress test (25 variants, 5 skills × {1,5,10,20,50} visible skills × 2 harnesses)** — held out of the headline:

| Visible skills | Mean overall lift | Mean wall time | With-skill pass rate |
|---|---|---|---|
| 1 | 0.133–0.149 (range over 1–20) | 258 s | 0.725 |
| 20 | (same band) | 451 s | — |
| 50 | — | **1,290 s** | **0.55** |

Lift is flat from 1 to 20 visible skills while latency nearly doubles; at 50 the pass rate falls to 0.55 and wall time is 5× the single-skill case. This is the operational reason ACES reserves live evaluation for release candidates rather than every edit.

**The two scan tiers, and the fulcrum.** Tier 1 is ~50 deterministic rules over Correctness (0.35), Discoverability (0.25), Reliability (0.25), Efficiency (0.15), scored 0–100. Tier 2 is a 9-criterion (+1 conditional) LLM-judge rubric on the documentation. On the 145-skill corpus, structural scores run **61 to 98, mean 79.2, median 79.8, σ = 4.9**; **94.5% clear the 70-point gate but only 48.9% reach 80**; 86.2% pass the judge rubric at threshold 70. And the two disagree: **Spearman ρ = 0.14, Pearson r = 0.08.** Disagreement clusters in two legible patterns — correct fields with vague instructions (passes structure, fails rubric) and content-rich skills that drop declared metadata (fails structure, passes rubric).

The violation profile is the other half of the story: **99.3% of skills fail to declare tools in frontmatter, 97.9% omit a Limitations section, 97.2% omit `author`, 91.7% omit `tags`** — and 94.5% still pass. The authors' phrase is exact: the declared frontmatter contract is **"aspirational rather than enforced."**

**Static scores are not runtime evidence (§6.7)** — the number the abstract does not lead with, and the more important one:

| Predictor (n = 62 production skills with both) | Spearman ρ vs. overall live lift | 95% CI (Fisher-z) |
|---|---|---|
| Tier 1 structural score | **−0.0181** | [−0.2667, 0.2327] |
| Tier 2 LLM-judge score | **−0.0266** | [−0.2745, 0.2247] |

The paper's own gloss is careful and correct: these are "consistent with no useful monotonic runtime proxy in these scan scores, **rather than evidence of a negative relationship**."

**Negative lift as a debugging signal.** 87 of 947 cases and 11 of 201 cells have negative mean lift. Trajectories separate two classes: execution instability (a with-skill run produced no scored trials while the baseline completed) and genuine regression (the agent found the skill but produced a truncated or meta-level response, skipped verification, or burned tool calls without improving the answer). One anonymized case had activation and behavior checks *improve* while goal accuracy and efficiency fell enough to make overall lift negative. Live pairing therefore distinguishes "never discovered" from "discovered but misused" — both invisible to a scan.

**Stated limitations** — (1) 145 skills skewed to System Access (49), Deployment (41), Platform (29), Data Infra (21), with Troubleshooting (3), Dev-Tooling (1) and Other (1) essentially unrepresented; (2) four harnesses with uneven cell coverage, no claim of generality beyond them; (3) the paired design cannot identify an environment-independent "intrinsic" skill contribution — lift is marginal value *under the declared workspace and baseline policy*; (4) the three-judge spread of ~1.5 points on a 0–10 scale is for **document** rubrics — live inter-judge agreement, human calibration and judge-uncertainty propagation are **unmeasured**; (5) VPN-gated enterprise skills get network errors in both arms, so their absolute `goal_accuracy` is a lower bound; (6) the raw-row audit conflates timeouts with other no-score causes, so no timeout rate is reported. A conservative imputation check (treating every baseline-scored/with-skill-unscored case as a with-skill failure) drops composite lift from 0.2153 to **0.1840** — still positive.

## Does ρ = 0.14 contradict the 138K-skills audit?

[[what-keeps-agent-skills-from-being-reusable-zhang-2026|Zhang et al.]] *is* a structural scan with a defect taxonomy — 31 detectors, 89.3% spec violation, 91.8% carrying a defect, 67.0% with a routing defect. If structural defects did not predict quality, that would gut it. **Verdict: not a contradiction. The two papers measure different things on four separate axes, and ACES says so itself.**

- **Different dependent variable.** ρ = 0.14 is *structural scan vs. LLM-judge document rubric* — two static readers of the same prose disagreeing with each other. Neither is a quality ground truth. Zhang's one functional result is a different quantity again: does a routing defect degrade *lexical retrievability*? It does, mildly and in the predicted direction — R1-clean 88.5% Hit@1 vs. R1-defective 82.6%.
- **Different independent variable.** Zhang reports **per-defect binary prevalence** across 31 detectors. ACES Tier 1 is a **weighted scalar aggregate** over ~50 rules. Aggregation destroys exactly the signal Zhang preserves: a skill shipping a live API key and one duplicating its name as an H1 both move the same 0–100 dial.
- **Severe restriction of range.** ACES's 145 skills have σ = 4.9 on a 0–100 scale, per-source means clustered in 78.5–80.2, and 94.5% above the gate. There is almost no variance left to correlate with. Zhang's corpus is 138,133 unfiltered public skills where 47.0% of repos contribute exactly one file and defect density spans 1.48 (<50 lines) to 4.76 (>500 lines).
- **Power.** n = 62, CI [−0.27, +0.23]. That interval comfortably contains Zhang's plausible effect size. ACES's own wording — "consistent with no useful monotonic runtime proxy," not "evidence of a negative relationship" — is the correct reading and is not a refutation.

> [!warning] What *is* contradicted: the inference, not the finding
> Zhang's own claims survive intact. What does not survive is the **ROI story people read off them** — that fixing detected defects (the three-rule linter at 71.9% coverage, the 58.4% LLM repair rate) buys runtime value. ACES supplies the first direct test of that bridge and finds the aggregate score carries **zero** monotonic information about live [[skill-evaluation|Skill Lift]] (ρ = −0.018), on a corpus where the frontmatter contract is 91–99% unmet and it does not seem to matter. Both papers are honest about the gap in their own terms — Zhang states functional evidence exists "for R1 alone, under BM25 alone"; ACES states scans "surface useful authoring issues" and are worth running. The unsupported claim is the one neither paper makes and every skill linter implies: **that a passing structural score is evidence a skill works.**

The constructive reading is that they are complementary at different granularities. Zhang tells you *which defect* is worth a detector at ecosystem scale; ACES tells you that the *composite score* built from such detectors is not a deployment gate. The missing experiment — which neither paper runs — is Zhang's **per-defect binaries** regressed against ACES's **live lift** on an unfiltered corpus.

## Relation to the rest of the 2026 skills cluster

- The **routing premium** (Lift_grp − Lift_iso) is the first *metric* proposed for the quantity [[at-skills-attention-is-all-you-have-yin-2026|@skills]] argues about qualitatively (description-matching as a bounded attention budget) and [[demystifying-agent-skills-jiang-2026|Jiang et al.]] measure indirectly (use-precision collapsing 29.6% → 3.3% at pool size 100 while success stays flat). ACES's 1→50 visible-skill sweep is the same experiment with lift instead of precision as the readout, and it reproduces the shape: **flat value, rising cost** — 0.133–0.149 lift from 1 to 20 skills while wall time goes 258 s → 451 s, then a pass-rate collapse to 0.55 at 50.
- **Process gains dominate outcome gains** (+0.3263 execution vs. +0.1431 accuracy), which is ACES's independent, metric-side confirmation of Jiang's mechanism finding that **procedural anchoring accounts for 65.7% of skill mechanisms and explicit knowledge injection for 4.5%**. Two very different methods — an LLM judge naming mechanisms on 528 triples, and six graded runtime metrics on 947 paired cases — land on the same split.
- Against [[continualskillbench-guan-2026|ContinualSkillBench]]'s null (library 0.602 vs. plain ICL 0.605), ACES is *not* a counterexample: it measures **curated, author-owned, single-skill marginal value under a fixed support set**, not self-authored accumulation over a task stream. That is the split [[catastrophic-remembering]] already draws, and ACES lands cleanly on the curated side of it.
- The 4× cross-harness spread in lift for identical skills (OpenCode 0.3611 vs. Terminus-2 0.0896) is a skills-layer instance of the attribution problem in [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] and [[harness-vs-model-attribution]]: **a skill's measured value is a property of the (skill, harness, model) triple, not of the skill.**

## Concepts & entities

- [[agent-skills]] / [[skill-md-format]] — the artifact under evaluation; ACES adds no format fields, only an adjacent `evals/` directory framed as "the skill's test suite."
- [[skill-evaluation]] — the paper's home concept, and the strongest current argument that it must be *runtime* evaluation; the ρ ≈ 0 result is the wiki's first direct measurement that artifact scores do not proxy for it.
- [[skill-routing]] / [[skill-selection]] — the routing premium is a proposed operational metric for both. *(Neither has a wiki page; ACES would be a primary source for the first and the strongest instrumentation proposal for the second.)*
- [[skill-library]] — the 50-visible-skill stress condition is a small library-scale experiment with pass rate and wall time as the readouts.
- [[skill-optimization]] — trajectory-grounded dataset refinement (`create-dataset --refine --from-results`) is optimization applied to the *evaluation assets* rather than the skill, which is the inverse of the usual loop.
- [[agent-evaluation]] / [[agent-benchmark]] — ACES's positioning: benchmarks fix the task set and vary the agent; ACES fixes the agent and varies one skill.
- [[llm-as-a-judge]] — three of six metrics are judged; `behavior_check` is a rubric-anchored G-Eval-style judge over free-form author assertions, and live judge agreement is explicitly unmeasured.
- [[agent-as-a-judge]] — the adjacent framing for trajectory-level rather than answer-level grading.
- [[context-assembly]] — the visible-skill sweep is a context-budget experiment: latency rises monotonically while value does not.
- [[guardrailing]] — `security` is deliberately kept as a *separate* metric so a safety failure stays visible when task behavior looks fine; the static supply-chain and [[prompt-injection]] scanner remains an external integration.
- [[terminal-bench]] · [[swe-bench]] · [[agentbench-liu-2023]] · [[mcp]] — the fixed-task-set benchmarks ACES positions itself as orthogonal to.
- [[nvidia]] — sole affiliation; the open-source implementation is NVIDIA SkillEvaluator.
- [[claude-code]] · [[opencode]] · [[openai-codex-skills-2025]] — three of the four primary harnesses (the fourth is Terminus-2); ATIF is emitted natively by structured-tool-calling harnesses and reconstructed heuristically from plain-text logs (e.g. `cursor-cli`).
- [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] — the structural-scan counterpart; see the section above.
- [[demystifying-agent-skills-jiang-2026]] · [[at-skills-attention-is-all-you-have-yin-2026]] · [[agentic-skills-in-the-wild-liu-2026]] · [[continualskillbench-guan-2026]] — the 2026 skills-evidence cluster; ACES is the enterprise/CI-side entry.
- [[is-progressive-disclosure-all-you-need-he-2026]] — the disclosure pattern whose routing cost ACES prices in wall time.
- [[catastrophic-remembering]] — the routing stress test is the accumulation cost measured in latency and pass rate rather than in file size.
- [[harness-vs-model-attribution]] · [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] — the 4× cross-harness lift spread.
- [[agent-skills-cross-vendor]] · [[agent-skills-survey-xu-2026]] · [[agentskills-io-spec-2025]] — the format and ecosystem context.
- [[nvidia-nemo-agentic-metrics]] · [[demystifying-evals-for-ai-agents-anthropic-2026]] — adjacent vendor guidance on agent evaluation metrics and CI regression testing, both cited in spirit by §5.

## References

- [arXiv:2608.20614](https://arxiv.org/abs/2608.20614)
- Open-source implementation: NVIDIA SkillEvaluator
- Cited building blocks: G-Eval (rubric-anchored judging), RAGAS (`AgentGoalAccuracyWithReference`), Harbor (sandboxed execution and ATIF-native task lifecycle)
- Compared against: SkillsBench (84-task fixed set, pass-rate deltas), Skills in the Wild (34k-skill retrieval), SkillTester, SkillAudit
