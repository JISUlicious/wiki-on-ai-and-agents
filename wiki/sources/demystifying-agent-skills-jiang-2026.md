---
title: "Demystifying Agent Skills: Why They Work—Until They Don't"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.14036"
year: 2026
authors:
  - Zhiyuan Jiang
  - Fangrui Huang
  - Hanwen Xing
  - Xander Wu
  - Yipeng Gao
  - Rui Cao
  - Mengdi Wang
  - Shilong Liu
  - Yijiang Li
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Demystifying Agent Skills: Why They Work—Until They Don't

**Zhiyuan Jiang\*, Fangrui Huang\*, Hanwen Xing, Xander Wu, Yipeng Gao, Rui Cao, Mengdi Wang†, Shilong Liu†, Yijiang Li†** ([[princeton-university|Princeton]]; UC San Diego; [[stanford-university|Stanford]]; USC; Johns Hopkins — \*equal contribution, †corresponding) — [arXiv:2608.14036](https://arxiv.org/abs/2608.14036), cs.AI, 14 Aug 2026. Preprint; no venue given.

## Summary

Every skills paper so far reports one number: did the skill-augmented agent solve more tasks? This one asks the question underneath — *when* do [[agent-skills|skills]] help, *why* do they work, and *where* do they fail — and answers it with a controlled design plus a **contrastive trajectory analysis** that makes the mechanism observable. The central control is representation. For each task, the authors collect a balanced pool of successful and failed trajectories, then build two artifacts from the *same* trajectories: **Workflow Memory** (cleaned procedural traces, in the [[procedural-memory|Agent-Workflow-Memory]] sense) and a **Skill** (the same traces distilled into a standardized [[skill-md-format|SKILL.md]] placed in the execution environment). A fixed-budget composition grid varies the source evidence from five successes / zero failures (`5s0f`) to `0s5f`. Two harness–model pairings run the protocol — Codex + GPT-5.3-Codex and Gemini CLI + Gemini-3.1-Pro-Preview — on Terminal-Bench 2.0 (89 tasks), Terminal-Bench Pro (200-task public split) and SkillsBench (86 tasks, 11 domains), under Harbor with n=5 trials per task.

The headline mechanism finding comes from the taxonomy. **8,135 trial records** are normalized into a manifest (7,837 with transcripts); 240 trajectories are open-coded, yielding **238 valid unique labels**, which are consolidated into **three categories and twelve skill-use modes**. A human annotator independently checked all 238 labels against 714 trajectories and re-mapped them to the twelve modes with **95.8% exact agreement, Cohen's κ = 0.952**. An LLM judge then labels **528 paired triples** (raw / workflow / skill on the same task: 144 SkillsBench, 186 TB-2.0, 198 TB-Pro) and names the *mechanism* by which the injected artifact acted. The answer: **procedural anchoring accounts for 65.7% of skill mechanisms; explicit knowledge injection for 4.5%**. Skills do not usually supply missing facts. They stabilize action — which setup steps to run, which tool sequence to follow, which checks to perform, which pitfalls to avoid.

The second finding is that the *representation* of experience matters more than the experience itself. On the 528 triples, skills succeed **61.9%** vs 59.1% raw vs 55.9% workflow memory; the only statistically clean paired difference is **Skill over Workflow Memory, +6.06 pp, 95% bootstrap CI [+0.76, +11.36]** — from identical source trajectories. Skill over Raw (+2.84) and Workflow over Raw (−3.22) both have CIs crossing zero. Workflow memory's specific failure is *process overload*: timeout exhaustion hits 10.6% of workflow runs vs 1.7% raw and 4.4% skill. Skills' specific failure is *misapplication*: `skill_guidance_misapplied_or_ignored` appears in 10.0% of skill runs vs 0.8% raw — the abstraction that makes a skill useful also creates a new failure surface, because a skill is not self-executing.

The third finding is the retrieval result the abstract leads with, and it needs careful reading. When the whole candidate pool is placed in the execution environment (Arm 3), the precision of the skills the agent *actually uses* against SkillsBench's ground-truth annotations falls from **29.6% at pool size 5 to 3.3% at 100** — while downstream success is essentially flat, **36.4% → 39.3%**. Recall at k=100 stays at 54.3–73.6%, so agents *do* find the right skill; they also inspect many wrong ones and do not restrict use to it. "Exact ground-truth invocation is neither sufficient nor necessary."

## Key points

**Table 1 — task success, Workflow Memory vs Skill, by trajectory mixture** (TB-Pro: 130 trials per condition; infrastructure errors count as failures):

| Harness + model | Raw (TB-2 / SkillsBench / TB-Pro) | Mix | TB-2 WM / Skill | SkillsBench WM / Skill | TB-Pro WM / Skill |
|---|---|---|---|---|---|
| Codex + GPT-5.3-Codex | 0.5935 / 0.5083 / 0.5394 | 5s0f | 0.4452 / **0.7548** | 0.5250 / **0.7250** | 0.7333 / 0.7455 |
| | | 3s2f | 0.4194 / **0.7806** | 0.6417 / 0.6250 | 0.6970 / 0.7333 |
| | | 1s4f | 0.2710 / 0.7097 | 0.5167 / 0.6167 | 0.6121 / 0.5818 |
| | | 0s5f | 0.2839 / 0.5161 | 0.5833 / 0.4500 | 0.4788 / 0.4303 |
| Gemini CLI + Gemini-3.1-Pro-Preview | 0.5000 / 0.4762 / 0.5615 | 5s0f | 0.6231 / **0.7923** | 0.5524 / **0.7429** | 0.5308 / **0.6692** |
| | | 3s2f | 0.6462 / 0.7462 | 0.5619 / 0.6667 | 0.6462 / 0.5462 |
| | | 1s4f | 0.5846 / 0.6923 | 0.4857 / 0.6000 | 0.5923 / 0.5692 |
| | | 0s5f | 0.5231 / 0.4769 | 0.4286 / 0.4095 | 0.4769 / 0.4615 |

Three things to see. Workflow memory *hurts* Codex on TB-2 at every mixture (0.27–0.45 vs 0.59 raw) — the process-overload failure in raw numbers. Skills built from success-only pools are the strongest cell almost everywhere; failure-only pools (`0s5f`) drop skills below raw in 5 of 6 harness×benchmark cells. And TB-Pro is the weak spot: Gemini skills fall below raw in 4 of 6 mixtures.

**Table 11 — the twelve modes** (share of each arm's 528 trajectories):

| Category | Mode | Raw | Workflow | Skill |
|---|---|---|---|---|
| SC1 anchoring | skill_guided_success | 10.4% | 0.4% | **61.6%** |
| SC1 | workflow_guided_success | 0.0% | 54.5% | 0.0% |
| SC1 | autonomous_clean_success | 48.7% | 0.8% | 0.2% |
| SC2 execution | environment_infrastructure_failure | 5.3% | 1.7% | **0.2%** |
| SC2 | output_format_schema_mismatch | 7.4% | 3.8% | 3.2% |
| SC2 | background_service_lifecycle_failure | 2.7% | 2.5% | 0.8% |
| SC2 | shell_code_corruption | 1.1% | 1.9% | 0.2% |
| SC2 | algorithmic_logic_error | 8.3% | 11.0% | 7.4% |
| SC2 | static_verification_without_runtime | 12.5% | 12.5% | 11.7% |
| SC3 invocation | timeout_budget_exhaustion | 1.7% | **10.6%** | 4.4% |
| SC3 | skill_guidance_misapplied_or_ignored | 0.8% | 0.4% | **10.0%** |
| SC3 | capability_or_safety_limit | 1.1% | 0.0% | 0.4% |

SC2 execution-layer failures fall from 37.3% (raw) and 33.3% (workflow) to **23.5%** with skills — environment and tooling problems are "highly skillable." But the two reasoning-shaped modes barely move: algorithmic logic error 8.3 → 7.4%, static-verification-without-runtime 12.5 → 11.7%. Skills fix operational fragility; they do not fix a wrong algorithm or force runtime validation. Category totals: SC1 326/528 skill vs 294/528 workflow; SC2 124 vs 197 raw vs 176 workflow; SC3 **78 vs 19 raw**.

**Mechanism labels** — procedural_anchor 65.7%, knowledge_injection 4.5% (the remaining labels are failure_warning, none, and counterproductive; per-label shares for those are not reported in the main text).

**Is it just "any compact procedural text"?** No. On 26 Terminal-Bench-2 tasks with Gemini (130 trials each): raw 50.0%, instruction-derived short plan **47.7%**, workflow-derived test-first template 59.2%, workflow memory 62.3%, **skill 79.2%**. A generic plan is worse than nothing; the distilled skill is 17 points above the next best.

**Cost (83-task matched intersection, per-task averages)**: raw 64.1% success / 555.7K tokens; workflow 64.8% / 426.2K; skill 69.6% / 521.5K. Skill beats raw on both axes (+5.5 pp, −34.2K), but costs **+95.3K tokens over workflow memory** for its +4.8 pp — "better but costlier."

**Outcome labels matter once failures enter the pool (Table 16)**. Withholding success/failure annotations from the skill creator changes little at `5s0f` but collapses skills built from mixed pools — Gemini TB-2 at `3s2f`: **0.7462 with labels vs 0.4000 without**; Codex TB-2 `3s2f`: 0.7806 vs 0.5871. The creator needs to know which trajectory to trust.

**Retrieval (Table 4, SkillsBench, averaged over pairings)** — three *independent* measurements over identical pools (ground-truth skill + k−1 distractors), never chained:

| Pool | Metric | k=5 | 10 | 20 | 50 | 100 |
|---|---|---|---|---|---|---|
| Random | Arm 1 embedding top-1 P | 97.7 | 95.5 | 95.5 | 92.0 | 84.1 |
| | Arm 2 explicit-selection P | 78.1 | 77.9 | 82.1 | 76.5 | 69.8 |
| | Arm 3 actual-use P | 25.9 | 23.2 | 19.5 | 8.6 | 4.4 |
| | Arm 3 success | 31.8 | 36.8 | 40.1 | 36.3 | 41.9 |
| Similar | Arm 1 | **70.5** | 63.6 | 60.2 | 56.8 | **53.4** |
| | Arm 2 | 53.1 | 52.9 | 47.1 | 48.6 | 43.7 |
| | Arm 3 P | 34.5 | 22.3 | 15.7 | 7.3 | 3.7 |
| | Arm 3 success | 41.7 | 39.6 | 39.2 | 39.5 | 39.6 |
| Dissimilar | Arm 1 | 96.6 | 96.6 | 96.6 | 94.3 | 93.2 |
| | Arm 2 | 78.9 | 81.6 | 82.6 | 78.4 | 77.8 |
| | Arm 3 P | 28.6 | 19.2 | 9.0 | 4.4 | 1.7 |
| | Arm 3 success | 35.7 | 36.9 | 33.7 | 38.8 | 36.4 |

Per pairing in Arm 3: Gemini actual-use precision 16.9% → 0.7% with success flat at 36–39%; Codex (GPT-5.4 here, because GPT-5.3-Codex access lapsed — so RQ4 is within-pairing only) 42.3% → 5.9% with success *rising* 35.4% → 42.0%. Offline, the embedding retriever (Qwen3-Embedding-0.6B) holds 88.3% → 76.9% top-1 and explicit agent selection 70.0% → 63.7%; **semantic confusability, not pool size, is the dominant offline stressor** (similar pools: 70.5 → 53.4 for embeddings; Codex explicit selection 51.9 → 31.9). Selection recall at k=100 is 70.5–85.2% — agents include the right skill *along with* distractors.

**Stated limitations** — terminal/tool benchmarks only (no web, no open-ended collaboration); two harness–model pairings; taxonomy induced from a stratified sample of ~3% of records, so rare modes may be under-represented; the 528-triple labeling is LLM-assisted (human-validated at the label and aggregation stages, not exhaustively); one representative trial per arm per triple; token metadata incomplete for some skill runs. Two caveats the paper does not list: **RQ1–RQ2 are same-task** — skills are distilled from a task's own prior attempts and evaluated on that task, so this is within-task experience representation, not cross-task library transfer; and **RQ3 (cross-harness transfer of Codex-built artifacts into Gemini CLI) is reported only as Figure 4**, with no numeric table and no discussion in the findings section.

## Does this contradict or corroborate ContinualSkillBench?

[[continualskillbench-guan-2026|ContinualSkillBench]] found that an agent maintaining an explicit self-authored skill library over a 100-task stream scores **0.602 vs 0.605 for plain in-context retention** of the same feedback — no aggregate advantage. The verdict here is **neither a clean contradiction nor a clean corroboration; each paper's strongest result lands on the other's blind spot.**

- *On representation, Jiang cuts against the null.* ContinualSkillBench's ICL condition is "same experience, not compiled into skills"; Jiang's Workflow-Memory condition is "same experience, kept as cleaned traces." Jiang finds compiling those traces into a SKILL.md is worth **+6.06 pp, CI excluding zero** — representation is not a wash. But the comparison is same-task (a skill distilled from a task's own prior runs, closer to a [[reflexion|Reflexion]] memo than a library entry), and Skill-vs-Raw itself is *not* significant on the paired sample (+2.84, CI [−2.27, +7.95]). So it does not establish that a library compounds across tasks — which is what ContinualSkillBench measured.
- *On libraries at scale, Jiang corroborates.* RQ4 is the library experiment: as the pool grows to 100, precision of the skills actually used collapses to 3.3% and **success does not move** (36.4 → 39.3). The specific artifact the library holds for a task is barely used correctly, and it does not matter. That is ContinualSkillBench's "fragmented library adds selection burden without proportional utility," measured from the other side — and it matches Jiang's own retrieval-free ceiling: SC3 invocation failures rise fourfold when skills are present at all.
- *The reconciliation the two papers jointly suggest*: curated, success-derived, outcome-labeled skills for **operational** failures (setup, formatting, service lifecycle) are real and large; self-authored accumulation across heterogeneous tasks, selected by the agent from a growing pool, is where the value disappears. Neither paper disputes the other on that split.

> [!note] What the 29.6% → 3.3% number is and is not
> It is **execution-time use precision** with the whole pool on the filesystem — the agent browsing a directory, not matching against resident descriptions. Recall stays above 54%, so the collapse is mostly *over-browsing* (touching many skills), not *missing* the right one. The nearest thing to [[at-skills-attention-is-all-you-have-yin-2026|@skills]]' "auto-trigger slots" is Arm 2, explicit selection from a listed pool, which degrades only mildly overall (70.0 → 63.7) but badly for Codex on confusable pools (51.9 → 31.9). This paper therefore **corroborates the direction** of the trigger-scarcity claim while suggesting the bottleneck is confusability and agent discipline rather than a fixed slot count.

## Concepts & entities

- [[agent-skills]] / [[skill-md-format]] — the artifact; the distilled SKILL.md is the treatment, placed in the execution environment per Anthropic's usage protocol rather than inlined.
- [[skill-evaluation]] — the paper's stated contribution: move from aggregate success to mechanism-level, paired, trajectory-grounded evaluation.
- [[skill-library]] — RQ4 is a controlled library-scale experiment: 5–100 skills, three distractor regimes.
- [[skill-routing]] / [[skill-selection]] — Arms 1–3 are three definitions of "did the agent pick the right skill," and they disagree by 70 points at k=100. *(No wiki pages yet; both are needed — this paper would be a primary source.)*
- [[procedural-memory]] — Workflow Memory is the direct-trace form; the paper's whole argument is that distillation beats preservation.
- [[experiential-learning]] · [[reflexion]] · [[expel-zhao-2023]] — the same-task, outcome-labeled learning this RQ1 setup most resembles.
- [[skill-acquisition]] — acquisition here is LLM distillation from a balanced success/failure pool, with outcome labels shown to be load-bearing.
- [[in-context-learning]] — the retained-context baseline family; see the ContinualSkillBench section.
- [[llm-as-a-judge]] — the 528-triple mechanism labels are judge-assigned; human validation covers the label-grounding and taxonomy-mapping stages (κ = 0.952) but not the full paired dataset.
- [[terminal-bench]] / [[terminal-bench-benchmarking-agents-cli-merrill-2026]] — TB-2.0 (89 tasks) and TB-Pro (200 public of 400) are the main testbeds; SkillsBench (86 tasks) has no wiki page yet.
- [[openai-codex-skills-2025]] — Codex CLI is one of the two harnesses; GPT-5.3-Codex for RQ1–3, GPT-5.4 for RQ4.
- [[dense-retrieval]] — Arm 1 is Qwen3-Embedding-0.6B cosine ranking over skill descriptions.
- [[self-improving-agent]] — the paper's closing frame: skill-based self-improvement is a lifecycle (represent → retrieve → invoke → adapt), not a memory-injection step.
- [[catastrophic-remembering]] — RQ4's flat success under collapsing use-precision is the library-scale form of accumulation ≠ compounding.
- [[memory-consolidation]] — distillation as consolidation; workflow memory as its absence, with timeout exhaustion as the measured cost.
- [[continualskillbench-guan-2026]] · [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] · [[agentic-skills-in-the-wild-liu-2026]] · [[reason-wide-not-deep-singh-2026]] — the 2026 skills-evidence cluster this paper's mechanism story slots into: Singh's distilled skills are procedural anchors by construction; Liu's 34K-pool degradation is Arm 3 in the wild.
- [[is-progressive-disclosure-all-you-need-he-2026]] — independently found sibling-description saturation; Jiang's Arm 2 confusability result is the skill-pool version.
- [[skill-rag-failure-state-aware-retrieval-wei-2026]] · [[compositional-skill-routing-gao-2026]] — retrieval/routing designs whose value this paper's Arm 3 result bounds: better selection may not move success.

## References

- [arXiv:2608.14036](https://arxiv.org/abs/2608.14036)
- [[agent-skills-survey-xu-2026]] — survey framing of skills as procedural knowledge packages
- [[anthropic-agent-skills-2025]] — the usage protocol followed (skills as environment resources)
