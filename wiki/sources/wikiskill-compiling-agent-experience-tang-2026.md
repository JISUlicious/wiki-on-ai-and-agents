---
title: "WikiSkill: Compiling Agent Experience into Persistent Knowledge for Skill Evolution"
type: source
created: 2026-09-05
updated: 2026-09-30
sources: []
arxiv_id: "2608.27454"
authors:
  - Liyan Tang
  - Cyrus Rashtchian
  - Chun-Sung Ferng
  - Andrew Tomkins
  - Da-Cheng Juan
  - Tu Vu
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# WikiSkill: Compiling Agent Experience into Persistent Knowledge for Skill Evolution

**Liyan Tang, Cyrus Rashtchian, Chun-Sung Ferng, Andrew Tomkins, Da-Cheng Juan** ([[google-research|Google Research]]), **Tu Vu** (Google Research; Virginia Tech — corresponding, with Tang) — [arXiv:2608.27454](https://arxiv.org/abs/2608.27454), cs.AI / cs.CL, 27 Aug 2026. CC BY 4.0. Preprint; no venue given.

> [!note] This wiki is an instance of the thing being studied
> §1 names its inspiration explicitly: *"Inspired by [[andrej-karpathy|Karpathy]] (2026)'s perspective on **LLM Wiki**, which advocates compiling experience into persistent, compounding knowledge, we ask: Can agent experience be similarly compiled into persistent knowledge to support long-term skill evolution?"* This vault is that perspective run as a practice rather than a benchmark. The paper is therefore both a source and a mirror, and the honest thing to record is where its design differs from this one — which the section on the ratchet below does.

## Summary

Every skill-evolution loop already has the ingredients: run an agent on training tasks, read the trajectories, propose a skill edit, keep it if validation improves. WikiSkill's observation is that in all of these — EvoSkill, Trace2Skill, [[skillopt-yang-2026|SkillOpt]] — **what the loop learned survives only inside the skill it happened to write**. Insights stay "scattered across optimization histories": a diagnosis that did not make it into an accepted edit is gone, and a rejected proposal teaches nothing to iteration k+3. The fix is a fourth artifact — a knowledge layer between raw experience and executable procedure — that persists whether or not any given edit is accepted.

The mechanism is a **three-layer workspace** plus a four-component loop, and the layer boundaries carry all the design:

| Layer | Contents | Mutability |
|---|---|---|
| `raw/` | full step-by-step traces per iteration: reasoning, tool calls, outputs, final answers | **immutable by construction** |
| `wiki/` | `patterns/` (one markdown page per failure mode or successful strategy, with actionable workarounds), `index.md` catalog, `logs.md` evolution log, `skill-impact.md` audit trail | append-and-revise; **never rolled back, never pruned** |
| `skills/` | the active skill set: `SKILL.md` plus `PURPOSE.md` mapping each skill back to the wiki patterns that motivated it | **rolled back on validation failure** |

Per iteration k: (1) the **Inference Agent** runs rollouts on `D_train` with skills S_{k−1} injected in full into its system prompt — and is *denied* wiki access; (2) the **Wiki Maintainer** samples traces (≤8 per iteration, stratified as ≤5 failing and ≤3 passing, each capped at 15,000 characters), performs root-cause analysis, creates or patch-edits pattern pages, revises `index.md`, and appends to `logs.md`; (3) the **Skill Proposer** runs [[react|ReAct]]-style — it is given only the wiki *index*, `skill-impact.md` and a pass/fail summary, then uses `read_file` on demand to pull the pattern pages and traces it wants, before emitting one **atomic** proposal targeting a single skill (create, or patch-edit); (4) **gating** applies the proposal, evaluates on `D_val`, and accepts strictly if `R(T_val) > R_best`, otherwise reverts to S_{k−1}. Regardless of the decision, the harness *programmatically* appends to `skill-impact.md` the proposal metadata, target skill, unified diff, validation score, and the Accepted/Rejected label.

That last sentence is the whole idea. The skill layer is aggressively gated and reversible; the wiki layer is a ratchet — and it is a ratchet *on purpose*, because the thing it records is precisely what gating throws away. A rejected proposal is a negative result, and negative results are the cheapest knowledge in the loop and the first thing every other framework discards.

Results are strong and consistent: WikiSkill has the best average on **all five models**, beating the strongest competing method by 3.3 / 5.1 / 10.0 / 5.8 / 12.0 points. Two second-order findings are more interesting than the headline. **Skill evolution complements model scaling** — within Qwen, gains rise +12.3 → +17.5 → +23.9 points from 4B to 9B to 27B — and yet a 9B with skills (47.4%) beats a 27B without (39.4%). And **transferred skills routinely beat self-evolved ones**, which the authors read as evidence that *discovering* procedural knowledge and *executing* it are distinct capabilities that self-evolution silently conflates.

## Key points

**Main results (Table 1)** — five benchmarks, five models, all methods starting from an empty skill set, skills injected in full at inference, scores averaged over **three independent runs of the entire evolution pipeline**, bolding by paired bootstrap (1,000 iterations, p<0.05):

| Model | Method | LiveMath | SealQA | SpreadSheet | OfficeQA | ALFWorld | **Avg** |
|---|---|---|---|---|---|---|---|
| Qwen-3.5-4B | No skill | 29.1 | 32.5 | 14.6 | 30.2 | 24.4 | 26.2 |
| | Trace2Skill | 31.5 | 37.6 | 17.5 | 31.0 | 42.8 | 32.1 |
| | EvoSkill | 41.7 | 37.3 | 18.6 | 29.5 | 41.5 | 33.7 |
| | SkillOpt | 48.7 | 33.3 | 14.0 | **34.5** | 45.3 | 35.2 |
| | **WikiSkill** | 49.7 | 39.4 | 21.1 | 28.5 | 53.7 | **38.5** |
| Qwen-3.5-9B | No skill | 28.2 | 26.3 | 24.3 | 35.9 | 34.7 | 29.9 |
| | EvoSkill | **58.1** | 34.5 | **35.4** | 34.9 | 48.5 | 42.3 |
| | SkillOpt | 48.7 | 29.4 | 29.0 | 38.0 | 55.7 | 40.2 |
| | **WikiSkill** | 56.3 | 43.1 | 33.6 | 40.5 | 63.4 | **47.4** |
| Qwen-3.6-27B | No skill | 33.9 | 27.5 | 40.8 | 42.1 | 52.8 | 39.4 |
| | EvoSkill | 57.3 | 32.9 | 59.5 | 52.5 | 64.2 | 53.3 |
| | SkillOpt | 51.9 | 34.5 | 53.2 | **54.8** | 59.2 | 50.7 |
| | **WikiSkill** | 61.9 | 41.6 | **81.7** | 53.7 | 77.6 | **63.3** |
| Gemma-4-31B | No skill | 33.9 | 30.6 | 48.3 | 43.3 | 50.4 | 41.3 |
| | SkillOpt | 40.1 | 36.1 | 63.1 | 44.4 | 61.9 | 49.1 |
| | **WikiSkill** | 56.7 | 41.2 | 68.0 | 44.2 | 64.4 | **54.9** |
| Gemini-3.5-Flash | No skill | 33.0 | 29.4 | 50.5 | 48.6 | 85.9 | 49.5 |
| | EvoSkill | 44.6 | 43.6 | 55.4 | 51.2 | 85.9 | 56.1 |
| | **WikiSkill** | 72.6 | 44.7 | 76.6 | 60.7 | 85.9 | **68.1** |

Note the honest asterisks the paper supplies itself: Gemini on ALFWorld is 85.9% for *every* method because Gemini already scores 100% on the validation split before evolution begins, so the loop terminates immediately. WikiSkill also *loses* to the no-skill baseline in a few cells — Qwen-4B on OfficeQA (30.2 → 28.5) most clearly — which the error analysis attributes to a small model being unable to execute its own multi-step long-context search procedure and reverting to default reading behaviour.

**Gains vary far more by benchmark than by method.** For Qwen-3.6-27B: SealQA +14.1, OfficeQA +11.6, ALFWorld +24.8, LiveMath +28.0, SpreadSheet **+40.9**. LiveMath benefits everywhere (+20.6 to +39.6 across all five models). The tasks where procedural knowledge pays are the ones with a discoverable, stateable workflow.

**Cross-model transfer (Table 2)** — the finding the abstract undersells:

| Inference model | Skill source | LiveMath | SpreadSheet | ALFWorld |
|---|---|---|---|---|
| Qwen-3.5-9B | None | 28.2 | 24.3 | 34.7 |
| | *self (9B)* | 56.3 | 33.6 | 63.4 |
| | Qwen-3.6-27B | 59.1 | **50.5** | **70.2** |
| | Qwen-3.5-4B | **61.0** | 25.0 | 69.2 |
| Gemma-4-31B | None | 33.9 | 48.3 | 50.4 |
| | *self (Gemma)* | 56.7 | 68.0 | 64.4 |
| | Qwen-3.5-4B | **73.1** | 37.1 | **66.9** |
| | Qwen-3.6-27B | 73.7 | **72.0** | 66.9 |
| Gemini-3.5-Flash | None | 33.0 | 50.5 | 85.9 |
| | *self (Gemini)* | 72.6 | **76.6** | — |
| | Qwen-3.5-4B | 67.5 | **18.1** | 87.3 |
| | Qwen-3.6-27B | **73.9** | 63.4 | 86.8 |

Transfer runs **upward** as well as downward — a 4B model's skills lift [[gemma|Gemma]]-4-31B from 33.9 to 73.1 on LiveMath. But the failure case is the instructive one: **Qwen-4B's SpreadSheet skills crater Gemini-3.5-Flash from 50.5% to 18.1%.** The diagnosed cause is that the small model encoded *low-level workarounds* — single-line Python commands, string-conversion rules — that protect a weak model from execution failures while forbidding a strong model from writing end-to-end scripts; plus fragmented diagnostic procedures whose redundant tool calls exhaust the interaction budget. A skill is not neutral procedural knowledge; it can be a scar from a weakness the reader does not have.

**Ablation (Table 3, Gemini-3.5-Flash, four benchmarks — ALFWorld excluded by early stopping)** — the paper's evidence that the wiki is load-bearing. Removing Skill-Proposer wiki access also removes the Wiki Maintainer, i.e. eliminates persistent accumulation entirely:

| Inference Agent wiki | Skill Proposer wiki | LiveMath | SealQA | SpreadSheet | OfficeQA | **Avg** |
|---|---|---|---|---|---|---|
| — (no skill at all) | — | 33.0 | 29.4 | 50.5 | 48.6 | 40.4 |
| yes | no | 43.8 | 42.0 | 44.4 | 51.0 | 45.3 |
| no | no | 51.3 | 38.4 | 49.9 | 55.2 | 48.7 |
| yes | yes | 64.8 | 42.8 | **80.2** | 55.6 | 60.9 |
| **no** | **yes** *(default)* | **72.6** | **44.7** | 76.6 | **60.7** | **63.7** |

Two readings. The persistent wiki is worth **+15.0 points** to the Skill Proposer (48.7 → 63.7). And giving the *Inference Agent* wiki access during training rollouts **costs 2.8 points** (63.7 → 60.9, with LiveMath falling 72.6 → 64.8) — the hypothesis being that an agent which can read the wiki solves tasks from it directly, producing trajectories that no longer diagnose what the *skill* is missing. Reading the knowledge store at task time contaminates the signal you are trying to distil.

**Skill and wiki dynamics (Table 4)** — the numbers that answer the accumulation question:

| Grouping | Skills created (proposed/accepted) | Skills edited (prop./acc.) | Skill length (lines) | Wiki patterns created | Wiki patterns edited | Pattern length |
|---|---|---|---|---|---|---|
| Qwen-3.5-4B | 3.1 / 1.6 | 4.9 / 1.3 | 126.2 | 8.8 | **18.4** | 48.2 |
| Qwen-3.5-9B | 4.6 / 1.4 | 3.4 / 0.7 | 128.6 | 7.3 | 10.9 | 26.6 |
| Qwen-3.6-27B | 4.4 / 1.5 | 3.6 / 0.8 | 118.9 | 6.5 | 17.9 | 47.7 |
| Gemma-4-31B | 4.8 / 1.3 | 3.2 / 0.8 | **45.1** | 6.3 | 13.7 | 23.7 |
| Gemini-3.5-Flash | 2.3 / 1.2 | 5.7 / 1.1 | 81.2 | **8.9** | 7.0 | 18.1 |
| LiveMath | 1.9 / 1.1 | 6.1 / 1.9 | 84.6 | **4.4** | 12.1 | 31.7 |
| SpreadSheet | 4.5 / 1.4 | 3.5 / 1.1 | **142.5** | **9.8** | 11.3 | 38.5 |

The caption is the load-bearing sentence: **"All wiki pattern creations and edits are retained."** Skill proposals are accepted at roughly 1 in 3; wiki writes are accepted at 1 in 1.

**Refinement does not stop early (Table 5)** — accepted skill updates by stage of an 8-iteration run (early = iters 0–1, mid = 2–4, late = 5–7): early 39–52% by model, mid 30–46%, late 4–21%. SealQA is the outlier with 33% mid and **28% late** — evidence, the authors argue, that accumulated knowledge keeps enabling improvements rather than saturating in the first pass.

**Setup details that bound everything above.** Splits are small: train/val/test = 35/18/124 (LiveMath), 16/10/85 (SealQA), 80/40/280 (SpreadSheet), 50/24/172 (OfficeQA), 39/18/134 (ALFWorld). The paper flags this itself — validation splits of 10–40 tasks make gating noisy, which is why every reported number averages three full pipeline runs with paired-bootstrap testing. Open models run under [[vllm|vLLM]].

**Stated limitations** — (1) skills are injected **in full** into the prompt, following prior work, deliberately eliminating retrieval and triggering as confounds; the paper concedes this is exactly what matters as libraries grow; (2) strict gating (`>` R_best) excludes *neutral* proposals that might enable later gains; (3) **"the Wiki Layer continuously accumulates pattern pages, evolution logs, and proposal diffs across iterations, but WikiSkill currently lacks an automated mechanism to prune the wiki"**; (4) no very-long-horizon tasks (hundreds of actions, multi-hour), and no online within-rollout skill adaptation.

## Consolidation or append-only? The ratchet verdict

**Verdict: append-and-revise, with no deletion anywhere — but the ratchet is deliberately quarantined off the context path, and the paper carries the one anti-ratchet ingredient [[catastrophic-remembering]] identifies as load-bearing.**

*What consolidation exists.* Pattern pages are edited with **incremental, patch-based operations — append, replace, insert text spans** — so a page can genuinely be rewritten in place as evidence accumulates, and `index.md` is revised whenever patterns change. Edits *outnumber* creations for four of five models (up to 18.4 edits vs. 8.8 creations for Qwen-4B), so the majority of wiki writes are revisions, not new files. That is real consolidation, and more than a pure append log.

*What does not exist.* No page deletion. No merge of redundant pages. No expiry. No cap on pattern count ("there is no hard limit on the number of patterns created or updated per iteration"). `raw/` is immutable by design, `skill-impact.md` is append-only by construction, and the wiki is "never rolled back regardless of the acceptance decision." The authors name the gap in their own limitations rather than leaving it to a reader.

*Whether the ratchet actually bites here.* Run the three assumptions from [[catastrophic-remembering]]:

1. **Additions are stochastic** — yes. The Wiki Maintainer decides what is warranted, with no budget.
2. **Outcomes are censored** — **no, and this is the important difference.** `skill-impact.md` is written *programmatically by the harness*, not by an LLM, and carries the unified diff, the validation score, and an explicit `Accepted`/`Rejected` label. That is precisely the provenance whose absence [[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti's]] model says makes deletion evidence never arrive — and note the sharp corollary there, that records of *attempts without outcomes* are worse than no record at all (+70.0% excess). WikiSkill records outcomes. It is, in the terms of that paper, a comment layer that happens to be machine-generated and therefore cannot drift from the truth. The paper even demonstrates the payoff in its ALFWorld case study: the rejected `goal-directed-action` proposal at iteration 0 is what informs the accepted `break-repetition-loop` at iteration 1.
3. **Instructions are redundant** — untested. Nothing measures overlap between pattern pages, and the `O(2^|D|)` audit cost applies to `patterns/` exactly as it does to a CLAUDE.md.

So assumption 2 is defused for the *ledger* and left intact for the *pattern pages*: nothing ever tells the Wiki Maintainer that a documented failure mode stopped occurring.

*Why the ratchet is not visible in the results.* The horizon is **8 iterations** producing **6.3–8.9 pattern pages of 18–48 lines**. `|D_∞| = A/(ρ̄·s)` diverges asymptotically; at K=8 with a wiki under 500 lines there is nothing to see. WikiSkill's evidence that accumulation compounds is real *at this scale* and is silent about the scale where [[continualskillbench-guan-2026|ContinualSkillBench]] found its null (100 ordered subtasks) or where [[why-does-claude-md-keep-growing-chakrabarti-2026|Chakrabarti]] measured +226% growth over a file's lifetime.

> [!note] The architectural move that matters more than pruning
> WikiSkill separates the **unbounded store** from the **bounded context surface**. Skills — the only artifact the Inference Agent reads — stay at 45–143 lines and are gated, reversible, and accepted at roughly 1 in 3. The wiki grows monotonically but is read *only* by the offline Maintainer and Proposer, and the Proposer reaches it through an index plus on-demand `read_file` rather than by loading it. The ablation makes this a measured design choice, not an accident: giving the Inference Agent wiki access **costs 2.8 average points**. This is the same shape as [[at-skills-attention-is-all-you-have-yin-2026|@skills]]' tier separation and [[rehearse-confidence-cliff-autoresearch-ji-2026|Rehearse]]'s retrieval-over-replay result — *accumulation is safe exactly to the degree that it stays out of the working context.* The cost of catastrophic remembering is paid in context pollution; WikiSkill never pays it, and defers the archival cost to a pruning mechanism it does not yet have.
>
> Worth recording plainly: **this vault has the same shape and the same gap.** Sources are immutable, pages are append-and-revise with contradiction callouts rather than overwrites, and there is no deletion or consolidation pass. It is protected by the same mechanism — the reader retrieves, it never loads the corpus — and exposed to the same unmeasured redundancy.

## Relation to the rest of the skills cluster

- **Against [[continualskillbench-guan-2026|ContinualSkillBench]]'s null** (explicit skill library 0.602 vs. plain ICL 0.605): not a refutation, because the settings barely overlap. ContinualSkillBench measures *self-authored accumulation across a 100-task stream with no gate*; WikiSkill measures *validation-gated evolution on a fixed training split, evaluated on a held-out test split*. WikiSkill's accepted-update rate of ~1 in 3 is the mechanism ContinualSkillBench lacks. The interesting untested prediction is that WikiSkill's advantage comes mostly from **rejection**, not from the wiki — and the ablation cannot distinguish these, because removing wiki access leaves gating in place and still scores 48.7 vs. 40.4 no-skill.
- **With [[demystifying-agent-skills-jiang-2026|Jiang et al.]]**: WikiSkill's full-prompt injection is the deliberate removal of Jiang's RQ4 variable. Jiang shows use-precision collapsing 29.6% → 3.3% at pool size 100 with success flat; WikiSkill sidesteps this and says so. Both papers also independently find that **outcome labels on trajectories are load-bearing** — Jiang measures 0.7462 → 0.4000 when success/failure annotations are withheld from the skill creator; WikiSkill builds the equivalent into `skill-impact.md` as infrastructure.
- **With [[what-keeps-agent-skills-from-being-reusable-zhang-2026|Zhang et al.]]**: WikiSkill's skills are 45–143 lines, well under the 500-line spec limit, and `PURPOSE.md` is a provenance file with no counterpart in the published corpus. Machine-evolved skills gated on validation are a different population from the 138K public ones — which is itself a testable claim nobody has run.
- **The negative-transfer result is a new failure mode for the wiki.** Qwen-4B's SpreadSheet skills dropping Gemini from 50.5% to 18.1% is not a defect in the spec sense; it is a *correct* skill for the wrong reader. No structural scan and no [[skill-evaluation|skill-evaluation]] protocol that fixes the model can catch it, which is exactly why [[evaluating-skills-not-just-agents-kevin-2026|ACES]] insists Skill Lift is a property of the (skill, harness, model) triple.

## Concepts & entities

- [[agent-skills]] / [[skill-md-format]] — the artifact being evolved; `PURPOSE.md` alongside `SKILL.md` is WikiSkill's own addition, linking each skill back to the wiki patterns that motivated it.
- [[skill-acquisition]] — the paper's core loop, and its sharpest empirical claim: **discovery and execution are distinct capabilities**, evidenced by transferred skills beating self-evolved ones.
- [[skill-optimization]] · [[skillopt-yang-2026]] — SkillOpt is a direct baseline (rejected-edit feedback plus epoch-wise meta guidance); WikiSkill's contribution over it is making the accumulated knowledge a *separate persistent artifact* rather than optimizer state.
- [[skill-library]] — the evolved skill set; deliberately small and fully injected, so this is not a library-scale study.
- [[skill-evaluation]] — validation gating is the acceptance signal, on splits of only 10–40 tasks, which the paper flags as the main source of run-to-run noise.
- [[catastrophic-remembering]] — the concept this paper is the strongest current test case for; see the verdict section. WikiSkill supplies recorded outcomes (assumption 2) and no pruning (deletion), at a horizon too short to distinguish the two.
- [[memory-consolidation]] / [[offline-consolidation]] — the Wiki Maintainer is an offline consolidator operating on ≤8 stratified traces per iteration; the ablation showing that *online* wiki access hurts is the sharpest available argument for keeping consolidation offline.
- [[memory-evolution]] · [[memory-construction]] · [[memory-management]] — patch-based pattern editing is memory evolution in the update-in-place sense, without an eviction policy.
- [[procedural-memory]] — skills are the procedural layer; the wiki is the declarative layer beneath it, which is the paper's actual architectural claim.
- [[experiential-learning]] · [[expel-zhao-2023]] · [[voyager]] · [[library-learning]] — the lineage of compiling trajectories into reusable procedures; WikiSkill's difference is the persistent intermediate representation.
- [[self-improving-agent]] — an instance with an explicit non-monotonic gate on the artifact that matters and a monotonic store behind it.
- [[continual-learning]] — the framing WikiSkill inverts: nothing is forgotten, by design.
- [[react]] — the Skill Proposer runs ReAct-style with `read_file`, pulling pattern pages on demand rather than receiving a fixed sample.
- [[in-context-learning]] — skills are injected in full into the system prompt, so this is prompt-space learning throughout; no weights are touched.
- [[alfworld]] — the embodied benchmark and the source of the case study (`break-repetition-loop`, "Never Return an Item to Its Origin Location").
- [[agentic-context-engineering]] / [[agentic-context-engineering-zhang-2025]] — the append-mostly playbook design WikiSkill's wiki layer resembles, and whose contradiction callout applies here too.
- [[rehearse-confidence-cliff-autoresearch-ji-2026]] · [[memharness-memory-is-reconstructed-not-replayed-wu-2026]] — independent evidence that retrieval from a store beats loading the store; WikiSkill's index-plus-`read_file` Proposer is the same pattern.
- [[skillreducer-gao-2026]] — the closest thing in the wiki to the pruning mechanism WikiSkill says it lacks.
- [[skillos-learning-skill-curation-self-evolving-agents-ouyang-2026]] · [[skill-zero-lu-2026]] — cited contemporaries in experience-driven skill evolution.
- [[continualskillbench-guan-2026]] · [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] · [[demystifying-agent-skills-jiang-2026]] · [[at-skills-attention-is-all-you-have-yin-2026]] · [[evaluating-skills-not-just-agents-kevin-2026]] — the 2026 skills cluster; see the section above.
- [[is-progressive-disclosure-all-you-need-he-2026]] — the disclosure property WikiSkill cites as motivation and then deliberately switches off.
- [[skill-routing]] / [[skill-selection]] — explicitly excluded as confounds by full-prompt injection, and named in the limitations as what matters next. *(Neither has a wiki page.)*
- [[google-research]] · [[google]] · [[deepmind]] · [[qwen]] · [[vllm]] — affiliation, model families (Gemini-3.5-Flash, Gemma-4-31B, Qwen-3.5-4B/9B, Qwen-3.6-27B) and serving stack. Virginia Tech (Tu Vu's second affiliation) and Andrej Karpathy, whose LLM Wiki perspective is the paper's stated inspiration, have no entity pages.

## References

- [arXiv:2608.27454](https://arxiv.org/abs/2608.27454)
- Karpathy (2026), *LLM Wiki* — the perspective §1 names as its inspiration
- Baselines: EvoSkill (arXiv:2603.02766), Trace2Skill, [[skillopt-yang-2026|SkillOpt]]; GEPA cited as the general prompt-optimization family deliberately not compared against
- Benchmarks: LiveMathematicianBench, SealQA, SpreadsheetBench, OfficeQA, [[alfworld|ALFWorld]]

## Related 2026 sources

- [[wfm-wiki-foundation-model-agentic-reasoning-dong-2026]] — Tencent Youtu's graph retriever for LLM-wiki-style memory; not budget-matched, and some cells self-described as unmeasured
