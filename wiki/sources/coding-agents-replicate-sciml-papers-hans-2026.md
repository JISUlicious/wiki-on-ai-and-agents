---
title: "Coding-agents can replicate scientific machine learning papers"
type: source
created: 2026-07-22
updated: 2026-07-22
sources:
  - coding-agents-replicate-sciml-papers-hans-2026.md
arxiv_id: "2607.02134"
year: 2026
authors:
  - Atharva Hans
  - Ilias Bilionis
first_author: Atharva Hans
tags:
  - 2026
  - paper
status: complete
importance: medium
---

# Coding-agents can replicate scientific machine learning papers

**Atharva Hans, Ilias Bilionis** (School of Mechanical Engineering, Purdue University; Hans now at Eli Lilly) — [arXiv:2607.02134](https://arxiv.org/abs/2607.02134) (v2, 10 July 2026). 16 pages, cs.AI; no venue listed.

## Summary

The paper introduces **Paper-replication**, a coding-agent **skill** that turns replicating a scientific ML paper *from paper materials alone* into a target-level evidence task. The problem it attacks is that a prompt alone does not preserve progress or verify work: with direct prompting the agent stops after reproducing only part of the paper, loses track of which claim comes next, or **treats its own description of progress as evidence** — including counting figures copied out of the paper's LaTeX source, figures drawn to match the paper without implementing the method, or results from a substitute method as successful replications. The fix is framed explicitly as **harness engineering**: change the environment the agent works in (persistent workspace records + validation checks run *outside* the agent) rather than adding prompt instructions.

The workflow makes the agent inspect the paper materials `P` (LaTeX source tree, figures, tables, appendices, bibliography, data references — author code is *not* allowed in this setup) and record each computational claim as a target `t_j` in a **reproduction matrix**. Each target carries where the claim appears, which source files support it, which data and method component it needs, what output the run must produce, how that output will be judged (the **acceptance rule**), its current status, and where it must appear in the replication report. A target may be marked MATCHED only when its full **evidence bundle** `E_j = (ŷ_j, R_j, P_j, C_j, G_j)` exists — candidate output, execution record, provenance linking the output to code/config/seed and the paper passages justifying the implementation, comparison against the paper's claim under the recorded acceptance rule, and report coverage. Hashes of the paper source, assets, and rendered pages, plus directory separation of `artifacts/paper_figures/` from `artifacts/figures/`, let checks reject outputs that reuse paper-provided material. An **output artifact alone does not count as evidence**.

The completion gate is the paper's central design claim: `V_complete = V_spec ∧ V_progress ∧ V_report ∧ (⋀_j [s_j = MATCHED]) ∧ [a = ∅] ∧ [report PDF exists]`. **Completion is a workspace state, not a statement in the agent's final message.** Across twelve independent runs (three each on four SciML papers — PIFT, PINN-I, PINN-II, SINDy — using Codex with GPT-5.4 at Extra High reasoning), all twelve workspaces pass the gate and all **158 recorded targets** reach MATCHED with report coverage. The more interesting result is what the workspaces preserve *despite* uniform completion: repeated runs differ substantially in how a paper is decomposed into targets, in numerical fidelity, in elapsed time, in how much correction work was discarded, and in which kind of acceptance rule was judged appropriate for the same claim.

## Key points

- **Corpus and setup**: 4 SciML papers × 3 independent runs = 12 replications. Same paper materials, prompt templates, tools, compute environment, and a no-author-code policy. Agent = Codex with GPT-5.4, Extra High reasoning, on an M4 Max MacBook Pro (128 GB) with access via a cluster-execution skill to Purdue Gautschi (192-core / 384 GB CPU nodes; 8×H100 GPU nodes). PIFT and SINDy completed locally; some PINN targets used cluster jobs.
- **Headline**: all 12 workspaces pass the completion gate; **158/158 recorded targets MATCHED with report coverage**.
- **Target decomposition varies by run.** PINN-I records 8 targets in all three runs and SINDy records 20 in all three (decomposition ratio 1.0). PIFT records 8, 8, and 25 (ratio **3.1**); PINN-II records 9, 9, and 15 (ratio **1.7**). A Gamma-Poisson model puts PIFT's expected target count at posterior median 13.4, 95% CI [8.8, 20.5]. The extra PIFT targets come from one run splitting a composite figure into per-panel targets; the extra PINN-II targets from one run recording appendix inference claims separately.
- **Paper-anchored numeric fidelity** (a separate analysis from workspace matching, using fixed thresholds taken from each source paper's reported accuracy scale rather than the agent's own acceptance rule): 13 standardized scalar anchors × 3 runs = 39 anchor-run observations; **37 fall inside the fixed threshold**. The two outside: Schrödinger run 3 at relative L2 error 4.8 × 10⁻² against a one-percent threshold, and Navier–Stokes λ₂ run 1 at 16.4% coefficient error against a ten-percent threshold. **Both still count as MATCHED in their workspaces** because the workspace check applies the target's own recorded acceptance rule — a concrete demonstration that MATCHED is not a universal fidelity metric.
- **Run-to-run numeric spread is large but often within the paper's accuracy class.** The clean Burgers λ₂ coefficient error takes values **7.3%, 0.14%, and 0.014%** across three runs — three orders of magnitude apart, all under the 10% threshold. Several run-3 PINN-II coefficients came out *more precise* than the values printed in the paper. Posterior median headroom (base-ten margin inside threshold): 0.51 for PINN-I, 1.75 for PINN-II, 0.42 for SINDy — i.e. discrepancies sit about **3.2×, 57×, and 2.6×** inside their thresholds. Run-residual scale is largest for PINN-II at posterior median 1.16 (95% CI [0.86, 1.66]), roughly a **factor of 14** in discrepancy across reruns.
- **Non-scalar targets** are judged claim-specifically: SINDy recovers **exact sparse support in every run for every system**, with trajectory figures preserving attractor, bifurcation, and slow-manifold geometry; PIFT's posterior standard deviation decreases monotonically with inverse temperature in all three runs, the Allen–Cahn prior and posterior stay bimodal with exactly two modes, and the diffusion coefficient is sharply identified while the reaction coefficient is not. PINN table targets preserve reported accuracy *patterns* as data size, collocation points, capacity, RK stages, and noise vary, even where individual cells don't match every printed digit.
- **Effort varies ~2× even conditioning on paper.** Elapsed replication time ranges **1.2 to 13.0 hours**; posterior medians 6.9 h (PINN-II), 5.0 h (PINN-I), 2.2 h (PIFT), 1.9 h (SINDy). Posterior probability a PINN paper takes longer than a non-PINN paper: 0.947–0.972.
- **Correction work is recorded, not overwritten.** 25 superseded tracked executions across the corpus — 11 in PINN-I, 10 in PINN-II, 3 in PIFT, 1 in SINDy. Within PINN-I, run 1 records 71 tracked executions (11 superseded) while runs 2 and 3 each record 12 with none superseded.
- **Judgment variation**: fraction of aligned paper claims where *every* run of that paper recorded the same acceptance-rule type — SINDy 19/20 = 0.95, PIFT 8/11 = 0.73, PINN-I 4/8 = 0.50, PINN-II 5/11 = 0.46. The PINN papers vary because repeated runs sometimes classify the same claim as a numeric target and sometimes as a structural one. Two completed workspaces can support the same claim while accepting different *kinds* of evidence as sufficient.
- **Skill implementation**: two layers — persistent agent instructions (`SKILL.md`, plus separate Codex and Claude Code prompt files adapting them per interface; `references/` files defining the workspace contract, author-code rule, compute environment, and acceptance-rule types) and workspace utilities (`scripts/paper_replication.py`) that create and check the manifest, reproduction matrix, task ledger, specification files, and report template. Instructions state the workflow rules "in a form the agent can reopen after an interruption or after losing earlier chat context" — the workspace, not the chat transcript, is the record of the run.
- **Stated limitations**: 4 papers, 3 runs each, one agent interface / model / reasoning setting. Critically, **there is no ablation without the skill**, so the study characterizes what Paper-replication *produces* rather than estimating its effect versus an unstructured prompt. Only 13 scalar anchors. Hash checks do not rule out every transformed copy or adversarial reuse of paper material (none was observed). A passed check is "recorded evidence for the target, not proof that no other implementation choices could also be consistent with the paper."

## Concepts & entities

- [[agent-skills]] — Paper-replication is implemented as a coding-agent skill (`SKILL.md` + `references/` + `scripts/`), and the paper is a concrete case study of a skill that encodes a workflow contract rather than domain knowledge.
- [[verifier]] — external validation checks and the completion gate are the core mechanism: evidence-gated completion instead of self-reported completion.
- [[code-as-harness]] — the paper names its own approach "harness engineering": changing the environment (persistent files + external checks) rather than the prompt.
- [[agent-evaluation]] — the argument that replication should be evaluated by the *recorded process*, not solely by exact numerical agreement; and that a completion label alone hides decomposition, fidelity, effort, and judgment variation.
- [[llm-as-a-judge]] — contrast case: acceptance rules here are recorded *before* judging and checked externally, rather than left to the agent's in-context assessment.
- [[self-improving-agent]] — related through the harness/scaffold axis: durable workspace records are what let an agent resume and correct itself across interruptions.
- [[hallucination-detection]] — the specific failure modes the checks target (copied paper figures, substitute methods, output that merely *resembles* the reported result) are agent-level confabulation of evidence.

## References

- [arXiv:2607.02134](https://arxiv.org/abs/2607.02134) — "Coding-agents can replicate scientific machine learning papers", Atharva Hans, Ilias Bilionis, Purdue University, 2026 (v2).
