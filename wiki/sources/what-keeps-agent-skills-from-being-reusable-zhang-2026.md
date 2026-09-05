---
title: "What Keeps Agent Skills from Being Reusable? Evidence from 138K SKILL.md Files"
type: source
created: 2026-08-17
updated: 2026-08-17
sources:
  - what-keeps-agent-skills-from-being-reusable-zhang-2026.md
arxiv_id: "2608.08453"
venue: "Workshop on Agent Skills (Design, Evaluation, and Optimization of Procedural Knowledge for LLM Agents), San Jose, 2026"
year: 2026
authors:
  - Chi Zhang
  - Yimin Liu
  - Xinze Chen
  - Ping Ji
tags:
  - 2026
  - paper
status: complete
importance: high
---

# What Keeps Agent Skills from Being Reusable? Evidence from 138K SKILL.md Files

**Chi Zhang, Xinze Chen** (The Graduate Center, CUNY), **Yimin Liu** ([[ohio-state-university|The Ohio State University]]), **Ping Ji** (Hunter College, CUNY) — [arXiv:2608.08453](https://arxiv.org/abs/2608.08453), cs.AI, 9 Aug 2026. CC BY 4.0. Dataset: `huggingface.co/datasets/FayeZC/SkillMD-138K`.

## Summary

The [[agent-skills|Agent Skills]] premise is that a `SKILL.md` turns task experience into a *portable software artifact* — something that still works outside the task, repository, platform, and conversation that produced it. This paper asks the empirical question nobody had asked at scale: does the published ecosystem actually meet that bar? The answer is no, and the failure mode is boring rather than exotic. Not malicious skills, not clever injections — **ordinary packaging problems**: weak routing metadata, bloated non-actionable bodies, and unorganized resources.

The method is static analysis at ecosystem scale. The authors crawl **138,133 unique SKILL.md files from 20,556 public repositories** (GitHub code search with 40+ sharded queries to beat the 1,000-result cap, plus repository cloning and the agentskills.in registry API), deduplicated by SHA-256 with deliberately minimal filtering — no star, freshness, or author filters, since prevalence questions need the unpopular and stale repos included. They then build a **two-tier taxonomy of 7 categories and 31 checks**, organized around the *lifecycle of reuse* (a skill must be selected → loaded → supported by resources → executed safely → ported). Tier 1 (14 checks) is derived clause-by-clause from the official spec and [[claude-code|Claude Code]] docs; Tier 2 (17 checks) comes from coding a stratified 300-skill sample for recurring non-spec failures, clustered against the literature on prompt injection, instruction hierarchy, hardcoded secrets, and portability. Each of the 31 detectors is a regex/structural rule validated by a paired positive/negative mutation fixture.

The headline: **89.3% violate the official specification** and **91.8% carry at least one detected defect**, stable at **88.8–94.6%** across lenient/baseline/strict thresholds on a deterministic 10,000-skill sample. The average skill has 2.5 defects (median 2).

To their credit the authors work hard against over-reading their own number. They explicitly warn that 91.8% "should not be read as '92% of skills are broken'" — it bundles shipped API keys with cosmetic H1-heading duplication — which is why they report the Tier 1 89.3% figure throughout and add a severity-prioritization discussion. And they run exactly one *functional* experiment, on the one category whose harm can be measured from the artifact alone.

## Key points

**The taxonomy** — 7 categories, 31 checks:

| Tier | Category | Checks | What it governs |
|---|---|---|---|
| 1 (spec) | R1. Routing Metadata | 6 | missing / too-short / too-long / non-functional descriptions; routing info misplaced in body |
| 1 | R2. Body Content | 5 | >500 lines, non-actionable body, explaining the obvious, name-as-H1, description duplicated in body |
| 1 | R3. Resource Organization | 3 | excessive inline code, too many examples, monolithic (no `scripts/`/`references/`/`assets/`) |
| 2 (best practice) | R4. Prohibited Content | 4 | install instructions, changelogs, license text, TODO/FIXME |
| 2 | R5. Behavioral Safety | 6 | hardcoded credentials, safety bypasses (`--no-verify`), injection patterns, unguarded `rm -rf`, error suppression, user-path leakage |
| 2 | R6. Portability | 4 | hardcoded model names (`gpt-4o`), platform paths, platform tool calls, OS commands (`pbcopy`) |
| 2 | R7. Persona & Scope | 3 | persona redefinition ("you are a…"), instruction overrides, scope mismatch |

**Prevalence** — the three most common individual defects are all Tier 1 spec violations:

| Defect | Rate |
|---|---|
| R1.3 missing trigger guidance ("use when …") | **52.3%** |
| R2.4 name duplicated as H1 heading | 44.3% |
| R3.2 too many inline examples | 32.1% |
| **≥1 routing defect (R1 any)** | **67.0%** |
| R1.4 non-functional description | 13.5% |

Over half of all published skills have descriptions that cannot effectively drive routing — which matters because the description *is* the entire startup-time selection surface under progressive disclosure.

**Routing stress test** — the one functional result, a BM25 index over frontmatter descriptions only, 20,000 sampled skills, deliberately chosen as a **lower-bound probe** rather than a production simulation:

| Query type | Group | n | Hit@1 | MRR |
|---|---|---|---|---|
| Metadata | R1-clean | 6,514 | **88.5%** | 0.906 |
| Metadata | R1-defective | 13,226 | 82.6% | 0.855 |
| Name/path | R1-clean | 6,505 | 26.7% | 0.347 |
| Name/path | R1-defective | 13,331 | 24.3% | 0.322 |

A ~6 pp Hit@1 gap. The authors are careful: shipped harnesses use an LLM-as-selector over the full description set, not lexical scoring, and semantic selection "may compress this gap." R2–R7 get no end-to-end measurement at all.

**Provenance and platform associations** (Kruskal–Wallis H=594.17, p<10⁻¹²⁵ across platforms):

- **Spec awareness is the strongest quality correlate**: skills whose descriptions follow "[Verb] [what]. Use when [trigger]" average **1.83 defects vs 3.00** for spec-unaware (Cliff's δ = −0.40, medium).
- **Cursor** skills are cleanest (avg 1.55 defects, 13.5% defect-free); **OpenClaw** marketplace skills worst (avg 2.29, 3.1% defect-free). [[claude-code|Claude Code]] is the largest identifiable platform at 18.7%, but 58.3% of skills sit in generic `skills/` paths and cannot be attributed at all.
- **AI-marked skills** (19,857 with explicit generation markers vs 118,276 unmarked): **3.23 vs 2.34** defects, +38.0% (Cliff's δ = +0.28, small); defect-free at 5.0% vs 8.8%. The gap concentrates in the dangerous categories — **R5 safety 18.9% vs 8.2% (2.3×)** and **R6 portability 12.8% vs 4.6% (2.8×)**. The paper flags this as association, not causation, and lists the confounds (self-disclosure bias, auto-stamping platforms) itself.
- **Defect density scales with size**: Spearman ρ=0.508; skills >500 lines average **4.76** defects vs **1.48** for skills <50 lines. Defects cluster — only 33.0% of defective skills are confined to one category, and safety co-occurs with portability at 2.19× chance.

**The "conversation-first provenance" hypothesis** — labelled a hypothesis, not a finding. The recurring co-occurrence of weak routing metadata + name-as-H1 + install instructions + hardcoded paths + model names + safety bypasses is exactly what a chat transcript looks like when serialized to disk. A developer solves a task interactively, saves the conversation as `SKILL.md`, and ships it. The authors explicitly note they did *not* measure text overlap with chat transcripts, which is what would be needed to test it.

**Exemplars (RQ4)** — 2,406 skills from repos with ≥10,000 stars, of which **419 have zero detected defects**. Two working styles: *minimal-and-precise* (React's verify skill, 24 lines, imperative commands + common mistakes) and *structured-workflow* (Discourse's service-authoring skill, 220 lines, eight-phase workflow with gate conditions). Commendably, the paper notes that 4 of the 5 "shared traits" are **tautological** — a zero-defect skill necessarily passes those detectors. The one non-circular observation, from manual reading: exemplars encode **project-specific local procedural knowledge** rather than tutorial content the LLM could already generate.

**Enforcement ROI** — simulating 12 cumulative linter configs ordered by marginal coverage gives a clear elbow at **three rules**: G1 (trigger-complete description) alone catches **33.4%** of defect instances; adding G4 (no name-as-H1) and G7 (externalize code/examples) reaches **71.9% coverage while flagging 85.9% of skills**. The full 12-guideline suite covers 99.4%; adding G9+G10 lifts *critical* (routing + safety) coverage from 74.7% to 98.6%.

**Repair** — an LLM given the detected-defect list and relevant guidelines fixes **58.4%** of defects on 200 defective skills while introducing only **0.06 new defects per skill**. But safety is the exception: **0% fix rate for bypass (R5.2) and injection (R5.3)** defects, which is why the proposed workflow makes safety a non-bypassable gate *after* repair. (Internal validity: repair used GPT-4o-mini, and the 0% figures come from small per-category subsamples used only to motivate the gate.)

**GitHub-issue corroboration** — 6,233 issues from 12 agent platform repos filtered to 761 relevant, 303 classified into R1–R7. Routing is both most prevalent (67.0%) and most reported (69 issues). Concrete failures: skills that never activate despite matching descriptions; **28 plugin skills totaling 34K tokens causing 4+ minute cold starts**; published skills shipping live API keys. Of 108 ClawHub security-scan false-positive appeals, **63 (58.3%)** came from credential patterns in legitimate security-auditing skills — keyword scanners cannot tell a skill that *handles* credentials from one that *leaks* them.

**Stated limitations** — regex detectors risk false positives, worst for the two most context-dependent checks (R5.3 injection fires on red-team skills that legitimately discuss injection; R7.1 persona fires on skills designed to teach a role), so **per-category R5/R7 prevalence should be read as upper bounds**; no stratified manual precision audit was run. Treating the spec as ground truth means legitimate deviations (narrative skills, long-but-actionable bodies) count as defects. Public GitHub + one registry only, with heavy repo skew (top repo = 17,284 skills = 12.9%; top 5 = 23.5%; 47.0% of repos contribute exactly one skill). Platform claims hold only for the attributable 41.7%. And functional evidence exists for R1 alone, under BM25 alone.

> [!note] Read alongside the prevalence number
> The paper's own summary of prior work is the sharpest framing of the stakes: SkillsBench found curated skills improve task completion by **16.2 percentage points while self-generated skills provide no measurable benefit**. This paper supplies a mechanism for the second half — most published skills are not written in a form that can be selected, loaded, or ported.

## Concepts & entities

- [[agent-skills]] — the standard being audited; the paper measures the gap between the format's portability promise and the published corpus.
- [[skill-md-format]] — the file format; every Tier 1 check derives from its spec clauses (name ≤64 chars, description ≤1,024 chars, body <500 lines, `scripts/`/`references/`/`assets/`).
- [[skill-routing]] — the frontmatter description as the startup selection surface; 67.0% of skills are defective here, and it is the only category with functional evidence.
- [[skill-evaluation]] — a complementary axis: artifact quality *upstream* of task-success benchmarks.
- [[skill-optimization]] — the repair loop and three-rule linter are optimization applied to existing artifacts.
- [[skill-library]] — an ecosystem-scale portrait of what public libraries actually contain.
- [[context-assembly]] — body bloat and monolithic skills are context-budget failures; the 34K-token / 4-minute cold-start issue is the concrete cost.
- [[bm25]] — the deliberately weak lexical retriever used as a lower-bound routing probe.
- [[prompt-injection]] / [[indirect-prompt-injection]] — R5.3 and the instruction-hierarchy framing behind R7.
- [[guardrailing]] — the argued need for a non-bypassable safety gate, since LLM repair fixes 0% of bypass/injection defects.
- [[claude-code]] — largest identifiable platform (18.7%) and a documentation source for Tier 1.
- [[llm-agent]] — the consumer of these artifacts.
- [[ohio-state-university]] — co-author affiliation.
- [[skillreducer-gao-2026]] — prior work cited for non-actionable body content and missing routing descriptions.
- [[agentic-skills-in-the-wild-liu-2026]] — cited for brittleness of realistic skill retrieval at 34K-skill scale.
- [[is-progressive-disclosure-all-you-need-he-2026]] — the controlled study of the disclosure pattern this paper's R2/R3 checks assume.

## References

- [arXiv:2608.08453](https://arxiv.org/abs/2608.08453)
- Dataset: https://huggingface.co/datasets/FayeZC/SkillMD-138K
- [[agentskills-io-spec-2025]] — the official specification the Tier 1 checks encode
- [[agent-skills-survey-xu-2026]] — the survey context for skill packaging as a paradigm

## Related 2026 sources

> [!warning] What structural defects do *not* predict
> [[evaluating-skills-not-just-agents-kevin-2026|NVIDIA's ACES]] scanned 145 skills and found structural scan score correlates with an LLM-judge document rubric at **Spearman ρ = 0.14** — and, more pointedly, with *live paired-run Skill Lift* at **ρ = −0.018** (n = 62, CI [−0.267, +0.233]).
>
> This does not refute anything measured here. The one functional result on this page — clean vs defective retrieval at **88.5% vs 82.6% Hit@1** — is in the predicted direction and sits comfortably inside that confidence interval. The two studies differ in dependent variable (scan-vs-scan versus scan-vs-runtime), in aggregation (31 per-defect binaries versus one weighted scalar, which flattens a shipped API key and a duplicated H1 onto the same dial), and in range (138,133 unfiltered public skills versus 62 skills that had already passed a review gate, 94.5% of them above it).
>
> What the ACES result does kill is an **inference neither paper makes but every skill linter implies**: that fixing structural defects buys runtime value. The three-rule linter covering 71.9% of defect instances and the 58.4% LLM repair rate are real, and their downstream payoff is unmeasured. The missing experiment is these per-defect binaries regressed on live lift.
