---
title: "Why Does CLAUDE.md Keep Growing? Catastrophic Remembering in Agentic Coding"
type: source
created: 2026-08-17
updated: 2026-09-30
sources: []
arxiv_id: "2608.11095"
year: 2026
authors:
  - Kushal Chakrabarti
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Why Does CLAUDE.md Keep Growing? Catastrophic Remembering in Agentic Coding

**Kushal Chakrabarti** (South Park Commons) — [arXiv:2608.11095](https://arxiv.org/abs/2608.11095), cs.AI, 11 Aug 2026. Single-author; no venue given.

> [!important] Directly load-bearing for this wiki
> This wiki is itself maintained through a `CLAUDE.md`. The paper's corpus is exactly this class of file — public GitHub repositories carrying `CLAUDE.md`, `AGENTS.md`, or `copilot-instructions.md` — so its findings apply to this repository's own instruction file, not merely to a topic it covers.

## The problem

Agentic instruction files grow without bound, stopping only when the repository retires or someone rewrites the file wholesale. Deleting the file is not the fix: prior work finds repositories carrying one finish agent tasks *faster and in fewer tokens*. But keeping everything is not free either — instruction-following degrades as constraints accumulate, and **the median file already carries 39 instructions (90th percentile: 131)**, past the point where that degradation is documented.

Three mechanisms could explain the growth, and they make **opposite, testable predictions about the deletion hazard**:

| Mechanism | Predicted deletion hazard vs. instruction age | Also predicts |
|---|---|---|
| **Instruction staleness** (requirements stop applying) | **rises** with age | — |
| **Content fragility** (brittle instructions die young) | falls with age | (compositional artifact only) |
| **Imperfect recall** (maintainers lose the rationale) | **falls** with age | also falls with **number of maintainers** |

Nobody had identified the root cause because line diffs and file sizes cannot resolve individual instructions over time. The paper's methodological move is to track **instruction lifetimes** rather than file sizes.

## The mechanism — get this part exactly right

The asymmetry is the whole paper. **Recording an instruction's [[latent-reasoning|latent reasoning]] at write time costs O(1). Reconstructing it later costs O(2^|D|) in a prompt of |D| instructions.**

Formally: prompt maintenance is online estimation of an unobservable constraint set from censored, noisy feedback. A task pairs a stated objective `o_j` with unobservable constraints `C_j ⊆ C`; the prompt `D_t` is a set of instructions; the maintainer's target is the **minimum cover** `D_*` — the smallest instruction set that maximizes expected constraint satisfaction. Every experiment scores `D_t` against `D_*` on two axes: **correctness** (expected satisfaction) and **excess size** (`|D_t|/|D_*| − 1`). A regime wins only by improving one without degrading the other.

Five assumed properties of the feedback, and note which does what:

| | Property | Behavior | Role |
|---|---|---|---|
| A1 | instructability | some `d` raises `q_c` | sets the regime |
| A2 | interference | adding `d` lowers other `q_c'` | sets the regime |
| A3 | redundancy | a second `d'` adds no `q_c` | sets the regime |
| **A4** | **censoring** | **outcomes don't name their `d`** | **blocks deletions** |
| **A5** | **stochasticity** | **`q_c < 1` even when covered** | **drives additions** |

Why deletion is exponential: instruction `d` is excess only if its contribution `Δ_c(d | D)` is ≤ 0 for *every* constraint. Estimating that means probing `D \ {d}` — and **redundancy (A3) defeats one-at-a-time probes: two instructions each covering one constraint look free individually, but deleting both breaks it.** An honest audit therefore costs O(2^|D_t|) subset probes and `D_*` is incomputable in general. Knowing *why* the maintainer added `d` — its latent reasoning `r_d` — collapses that to O(1).

The closed form. Define **imperfect recall** as the decay `ρ(d,a) = Pr(r_d recoverable at age a)`, with `ρ(·,0) = 1` and `ρ → 0` as age → ∞. A maintainer rationally deletes only when the reason survives *and* the instruction verifies as excess, so the hazard factorizes as `h(a) ≈ ρ̄(a)·s(a)`. Additions net against summed hazard, and flow balance gives:

```
|D_∞| = A / (ρ̄(a)·s)  →  ∞   as a → ∞
```

**This is the key result and it is easy to misstate.** Excess size diverges *even with additions arriving at a constant rate and the constraint set completely fixed*. The task does not have to change. The requirements do not have to go stale. The decaying memory of *why* each instruction was written is by itself sufficient to drive unbounded growth.

The name: **[[catastrophic-remembering|catastrophic remembering]]**, the inverse of catastrophic forgetting. "Under catastrophic forgetting, a gradient learner overwrites what it should have kept; under catastrophic remembering, a maintainer keeps what they should have overwritten. Both fail for the same reason: whatever would license the update is gone."

## Field measurement (1,867 repositories)

Corpus: public GitHub repos carrying `CLAUDE.md`, `AGENTS.md`, or `copilot-instructions.md` at their default branch. Every version of every file was **re-derived by walking the full commit history forward** — deliberately not anchoring on the final version and searching backwards, since that would condition on survival, which is the quantity under study.

**1,867 repositories → 1,801 multi-version files → 299,440 version-to-version transitions → 247,694 instruction lifetimes → 28,426 tracked deletions.** Matching validates at **1.000 precision and 0.933 recall** on 50 hand-annotated transitions.

**Growth**

- Among 1,576 multi-version repositories, **64.3% grow their instruction count vs. 26.6% that shrink it**, median net **+7 instructions**.
- Excluding mass rewrites, each commit adds a net **+4.9 instructions** across 19,267 commits.
- Over a file's own lifetime the mean trajectory gains **+226% in count**, **+10% in mean instruction length**, and **+140% in total size** — so the count growth is not text migrating between instruction and payload classes.

**Deletions are almost never judgments**

- **76.8% of instruction deaths arrive in a wholesale rewrite**; 77.3% in a rewrite *or* a migration to a sibling file. Both are censored as competing risks, since neither involves a maintainer judging any given instruction to be worth its place.
- A "rewrite" is defined as a commit where a file loses half or more of its instructions.
- This explains why "deletions are rare" had stood unexplained: a line diff cannot separate a deletion from a rewording.

**The ratchet** — aligning files on their first mass rewrite at t=0:

| Phase | Value |
|---|---|
| Growth before first rewrite | **4.1% instructions per commit** |
| Size at rewrite (t=0) | **59.5%** of pre-rewrite |
| Size 10 commits later | **91.5%** of pre-rewrite |
| Growth after rewrite | **4.9% instructions per commit** |

> [!warning] Correction to the common summary
> The claim often relayed is that "a wholesale rewrite resets size but leaves the growth rate intact." The measurement in §3.3 is stronger: prompts grow **faster** after a rewrite (4.9%/commit) than before it (4.1%/commit). The paper's own introduction uses the looser "growth resumes at its prior rate," but §3.3 carries the numbers. Rewriting is not neutral — it is followed by accelerated regrowth.

**Hazard evidence discriminating the three mechanisms**

- Nelson–Aalen over 28,426 deletions: **log-hazard slope −0.032 per commit**, 95% repository-stratified bootstrap CI **[−0.047, −0.019]**, excluding zero. Hazard *falls* with age — which **falsifies instruction staleness** outright, since staleness predicts a rising hazard.
- Content fragility could bend the hazard down through composition alone. Refit with a gamma frailty model (n = 28,255 deaths, censored at 50 commits); its strongest form — shared instructions with identical text — **absorbs 30.8% but leaves the slope at −0.0355** ([−0.0414, −0.0296]). Composition cannot account for it.
- The discriminating prediction only imperfect recall makes: if latent reasoning is encoded *while* a maintainer makes the change, hazard should also decay with the **number of maintainers** touching a file. It does: **β(multi-author × age) = −0.021, z = −11.7**, controlling for file activity.

Note the direction of the residual bias: matcher error that survives biases the age slope *toward* zero, so the measured decline is a **lower bound**.

## The fix: prompt comments

Not an acronym — literally comments, in the software-engineering sense: **text addressed to the next maintainer and invisible to the interpreter.** An instruction tells the executor *what to do*; a comment tells the next maintainer *why*.

**Inverse-IFEval**, a new evaluation with a known optimum. For each IFEval item `(D_j, C_j)`:
1. **Hide `D_j`** — the stated instructions become the reference minimum cover `D_*`, so `|D_*|` is known a priori and excess size becomes measurable.
2. **Keep `C_j`** — its verifiers become the world's hidden constraint set, run by the harness and never named to the maintainer.
3. **Generate a brief objective `o_j`** — a stronger model lossily summarizes `D_j` into a general task statement.

A fresh maintainer reconstructs `D_j` from `o_j` over T steps under censored, noisy feedback. The harness **strips comments before the prompt reaches the executor** and **rejects any instruction citing a past failure**, leaving the comment as the only surviving channel for latent reasoning. That enforcement is what makes the experiment clean: the treatment cannot smuggle information through the instruction channel.

**Results (Inverse-IFEval, 552 maintenance histories, `|D_*| = 2.2`):**

| T | Arm | Final count | Excess size (%) | Satisfaction @ t=T (%) |
|---|---|---|---|---|
| 15 | control (no comments) | 3.5 | **+60.4 ± 18.4** | 39.0 ± 2.8 |
| 15 | placebo (comment-shaped noise) | 3.3 | +53.2 ± 14.6 | 40.3 ± 2.7 |
| 15 | **treatment (informative comments)** | 2.1 | **−5.8 ± 6.6** | 38.2 ± 2.6 |
| 51 | control (no comments) | 6.6 | **+211.3 ± 105.3** | 44.0 ± 4.9 |
| 51 | placebo | 5.3 | +147.9 ± 79.9 | 42.6 ± 4.9 |
| 51 | **treatment** | 2.2 | **+1.4 ± 22.1** | 44.0 ± 4.7 |

Excess falls from **+211.3% to +1.4% at T=51 — a 99.3% reduction — at parity satisfaction (44.0% both arms)**. At T=15 the swing is 66.2 pp (+60.4% → −5.8%). Note the treatment arm's final count at T=51 is **2.2, exactly `|D_*|`**: the prompt settles on its minimum cover rather than merely growing more slowly.

**The effect scales with maintainer capability, in the wrong direction for optimism about better models.** Varying the maintainer across 3 tiers at T=15, the *uncommented* arm's excess rises from **+67.7% to +571.9%** — stronger agentic maintainers ratchet *harder*. At the top tier, comments deliver strictly Pareto gains in both satisfaction and excess size. A better model does not solve this; it makes it worse.

**Ablations show what a comment must carry.** Comment-shaped noise lands within the noise of the no-comment arm. A narrative of *attempts without their outcomes* is the **worst arm at +70.0%** — worse than no comment at all, because it hands successors an unvalidated premise to extend. Within the schema, the two fields recording *what happened* carry the reduction; **dropping the recurrence count alone costs 37% of it.** The comment format that works records the failure behind the instruction, a hypothesis, and how it has fared.

**Real prompts (Inverse-WildIFEval, 64 worlds).** Seed the maintainer with `|D_j| − K` true instructions plus G noisy ones drawn from other items. At K=1, G=16, the noisy instructions cost **24.1 pp of correctness on the true instructions already in the prompt** (65.6% at G=0 vs. 41.5% here; 95% CI [−33.4, −14.9] pp). Comments then lift satisfaction from **50.4% to 62.0% over 3 maintenance rounds (+11.6 pp, 95% CI [5.1, 18.3]) — the 23.1% relative gain.** Comment-shaped noise lands 2.7 pp from the uncommented arm (CI covering zero).

## Limitations

The paper's limitations section is unusually candid; these are its own, plus the judge caveat.

- **The headline 23.1% does not survive judge substitution cleanly.** WildIFEval constraints are prose with no code verifiers, so they are scored by an arm-blind LLM judge. All 6,336 verdicts were re-scored under a second judge: the criteria reproduce, but that judge measures **7.8 pp against the quoted 11.6 pp, and its 95% CI [−1.9, +9.6] does not exclude zero.** The paper is explicit that neither judge is ground truth — it measures reproduction and agreement, not accuracy. **Quote the 23.1% only with this attached.**
- **The controlled experiment runs where retention is nearly free**: covers of **2–3 instructions against a median real file's 39**, a 15-step horizon, one model as both maintainer and executor, and mechanically verifiable English constraints. §4.3 raises the difficulty (16 distractors against 4–5 human constraints) but **seeds** excess into benchmark prompts rather than growing it in real files — it prices what excess costs and what a comment recovers, never whether the ratchet persists at that size. The repository study stands in for that, and the two halves do not meet in the middle.
- **Three measurement choices were fixed and never varied**: the 50% rewrite threshold (which with the migration rule censors 77.3% of deaths), a matcher validated once on 50 author-annotated transitions against 299,440 tracked ones, and a per-corpus segmentation grammar — the largest untested degree of freedom on the corpus side.
- **Language distribution was not measured; no result speaks to non-English instructions.**
- The multi-author interaction counts **who has edited a file, not whether an instruction's own author has left** — a proxy for the mechanism, not a direct measure.
- **Explicitly open: "Whether catastrophic remembering reaches system prompts or agent skill files remains open."** The theory should apply to `SKILL.md` files and system prompts by the same argument, but the measurement does not cover them.
- **The recommendation is the main risk, per the paper's own ethics statement.** "We tell maintainers to delete instructions whose rationale they can recover, and an operator who automates that rule will delete instructions whose rationale was real. Our protocol empties a nontrivial share of prompts, and the uncommented arm scores higher on exactly those worlds." Writing a comment removes nothing and is safe; **acting on one is not.** The paper recommends keeping a person in the deletion path and holding safety-relevant instructions out of scope.

## Practical reading for this wiki's own CLAUDE.md

The mechanism implies a specific, cheap discipline that costs nothing to adopt and cannot regress behavior:

1. **Annotate at write time, not audit time.** Recording `r_d` is O(1) *only while you still have it*. Retroactive rationale reconstruction is the exponential operation the paper is about.
2. **The comment must carry outcomes, not intentions.** Attempts-without-outcomes measured *worse than no comment* (+70.0%). Record the failure that prompted the instruction, the hypothesis, and whether it worked — including recurrence, which alone carries 37% of the benefit.
3. **Distrust the wholesale rewrite as a remedy.** It cuts to 59.5% and is back to 91.5% within 10 commits, and regrowth is *faster* afterwards. It treats the symptom and leaves `ρ̄` untouched.
4. **Expect the problem to worsen as agent maintainers improve** (uncommented excess +67.7% → +571.9% across capability tiers).
5. **Keep a human in the deletion path.** Per the paper's own ethics statement, automating the delete rule is the dangerous half.

## Contradictions with existing wiki claims

> [!warning] Contradiction — "append-mostly playbooks" as a design virtue
> [[agentic-context-engineering]] (from [[agentic-context-engineering-zhang-2025]]) prescribes exactly the regime this paper measures as a pathology. ACE holds that **"contexts should be comprehensive playbooks, not concise summaries — LLMs can distill relevance at inference time, so detail should be preserved upstream,"** and builds an **"append-mostly playbook"** with **"delta updates vs. full-context rewrites."**
>
> The two agree on the *mechanism* — appends are cheap, deletions are rare — and disagree on whether that is a feature. Chakrabarti's field data says the pruning half of ACE's design ("the Curator can prune systematically rather than via opaque LLM judgment") is **not happening in practice**: +4.9 net instructions per commit, 76.8% of deaths arriving in a bulldozing rewrite, and a deletion hazard that *falls* with age.
>
> **They also disagree about what is wrong with a wholesale rewrite.** ACE says rewrites are actively harmful — *context collapse*, with its case study of 18,282 tokens at 66.7% accuracy dropping to 122 tokens at 57.1% in a single rewrite step. This paper says rewrites are **ineffective rather than harmful**: size resets to 59.5% and returns to 91.5% within 10 commits, with regrowth *faster* than before. Same operation, two different diagnoses.
>
> The sharpest reconciliation: ACE's **brevity bias** (optimizers like GEPA collapsing to short generic instructions) and this paper's ratchet are *opposite* biases acting on the same artifact. ACE describes machine-optimizer pressure toward too-short; Chakrabarti describes human-and-agent maintainer pressure toward too-long. Both are deviations from the same `D_*`, and the minimum-cover framing here is the first formal target that names what both are missing.

> [!success] Resolved contradiction — "capability grows monotonically"
> [[self-improving-agent]] claims that **"skills, prompts, or scaffold edits are appended; the agent's capability grows monotonically and side-steps the catastrophic forgetting of gradient-based continual learning."** This paper is precisely the rebuttal: side-stepping catastrophic forgetting by appending buys you **catastrophic remembering** instead, and the paper's whole framing is that these are duals, not an escape. Growth is not monotone in capability once instruction-following degrades with constraint count.
>
> **Resolved 2026-09-29:** [[self-improving-agent]]'s mechanism section was rewritten to reflect this finding.

> [!note] Horizon extension, not contradiction — memory stores that improve as they grow
> [[filesystem-based-memory-for-llm-agents-zhou-2026]] reports that **"stores get more useful as they grow at every model tested, nothing degrades within 140 tasks, early memories overwhelmingly survive."** That page already carries the caveat that **"nothing here measures months-long accumulation."** Chakrabarti's +226%-over-lifetime trajectory is exactly that missing horizon, so the two are best read as sequential rather than opposed: growth pays early and ratchets late. Zhou et al.'s finding that curation never gets cheaper (6–12 rounds per episode, maintenance replacing creation) is independently consistent with `ρ̄` decay.

**Corroborating existing pages** (no conflict): [[rethinking-evaluation-harness-evolution-wang-2026]] ("most edits memorize fixes rather than distilling strategies … accumulate prompt text that causes context bloat"), [[skillreducer-gao-2026]] (>60% of skill body content non-actionable; a less-is-more effect from removal), [[what-keeps-agent-skills-from-being-reusable-zhang-2026]] (>500-line bodies, 34K-token cold starts), [[skillopt-yang-2026]] (compact and validation-gated beats accumulating), and [[skill-zero-lu-2026]] (pruning skill files by measured on-policy helpfulness).

## Related

- [[agentic-context-engineering]] — the paper supplies the missing mechanism and a growth law for this concept: files ratchet because rationale decays faster than the instruction, not because requirements go stale. See the contradiction above.
- [[context-assembly]] — the finding that extraneous instructions cost **24.1 pp** of compliance with the *true* instructions already present is a direct cost bound on over-assembly.
- [[skill-md-format]], [[agent-skills]] — the same file class by construction, but the paper explicitly flags skill files as **untested**: "whether catastrophic remembering reaches system prompts or agent skill files remains open." Note the MerchantBench observation of agents accumulating duplicated self-authored skills is suggestive but independent evidence.
- [[claude-code]] — `CLAUDE.md` is the paper's titular artifact and a third of its corpus.
- [[prompt-compression]] — orthogonal and worth distinguishing: compression shrinks tokens for a fixed instruction set; comments shrink the *instruction set itself* toward its minimum cover, and the comments are stripped before the executor ever sees them, so they cost the executor zero tokens.
- [[memory-management]] — prompt maintenance framed as memory with an eviction problem, where eviction is blocked by censored feedback rather than by capacity.
- [[ebbinghaus-forgetting]] — `ρ(d,a)`, the recoverability decay, is a forgetting curve over rationale rather than over content.
- [[llm-agent]], [[agentic-harness-engineering]] — the maintainer/executor/harness split is a harness design proposal: give prompts a comment syntax and strip it before execution.
- [[llm-as-a-judge]] — the WildIFEval half depends entirely on an arm-blind judge, and the second-judge check is where the headline weakens.
- [[agent-evaluation]] — Inverse-IFEval is a reusable evaluation *design pattern*: invert an instruction-following benchmark so the reference instruction set becomes a known optimum, making excess size measurable.
- [[in-context-learning]], [[long-context-llm]] — the degradation of instruction-following as constraints accumulate is the cost function driving the whole argument.
- [[reward-hacking]] — contrast: the failure here is not a maintainer gaming a metric but a rational maintainer facing genuinely censored feedback. Append-only is the *optimal* estimator once provenance is gone.
- [[skill-optimization]], [[memory-evolution]], [[procedural-memory]], [[tiered-memory]], [[agent-three-layer-model]], [[dynamic-workflows]] — all assume or describe accretion into a persistent instruction artifact; the ratchet is the cost model none of them price.
- [[bad-memory-prompt-injection-agentic-gadgil-2026]] — an angle nothing currently connects: an instruction nobody can justify deleting is also a *trust* surface. Imperfect recall means stale injected instructions are the hardest ones to remove.
- [[continual-learning|catastrophic forgetting]] — the paper defines its central term, [[catastrophic-remembering]], as this concept's inverse.
- [[continual-learning]] — the paper positions prompt maintenance as continual learning over text and notes nobody has built its analogue of rehearsal or regularization.
- `[[instruction-following]]` — **no page yet**; IFEval and WildIFEval both hang off it.

## References

- [arXiv:2608.11095](https://arxiv.org/abs/2608.11095)
- Corpus: public GitHub repositories carrying `CLAUDE.md`, `AGENTS.md`, or `copilot-instructions.md`
- Benchmarks inverted: IFEval; WildIFEval
</content>

## Related 2026 sources

- [[ai-native-sdlc-playbook-anthropic-2026]] — the ratchet **prescribed as practice**: *"when Claude makes a mistake twice, the correction goes into `CLAUDE.md`"*. It does bound the file at one page, but specifies no deletion mechanism, no review cadence and no way to tell which instructions still earn their place — the exact gaps this paper measures. Its comments fix would make that bound enforceable
- [[new-rules-of-context-engineering-claude-5-shihipar-2026]] — Shihipar: Anthropic cut 80%+ of Claude Code's system prompt for Claude 5 models with no measured loss
