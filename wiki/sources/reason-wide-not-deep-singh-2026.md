---
title: "Reason Wide, Not Deep: Amortizing the Reasoning Premium into Distilled Skills"
type: source
created: 2026-08-17
updated: 2026-10-08
sources:
  - reason-wide-not-deep-singh-2026.md
arxiv_id: "2608.07885"
year: 2026
authors:
  - Agamdeep Singh
  - Srishti Gautam
  - Priyanshu Gupta
  - Nikita Mehrotra
  - Tanmay Bakshi
  - Sumit Gulwani
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Reason Wide, Not Deep: Amortizing the Reasoning Premium into Distilled Skills

**Agamdeep Singh, Srishti Gautam, Priyanshu Gupta, Nikita Mehrotra, Tanmay Bakshi, Sumit Gulwani** ([[microsoft|Microsoft]]) — [arXiv:2608.07885](https://arxiv.org/abs/2608.07885), cs.AI, 8 Aug 2026. CC BY 4.0. Preprint; no venue given.

## Summary

Reasoning modes beat non-reasoning modes on agentic tasks and charge a 3–6× output-token premium **on every single episode, forever**, because reasoning tokens are regenerated from scratch each time. Reading the traces reveals why that is wasteful: within a fixed domain much of the deliberation is not instance-specific problem solving but **re-derivation of episode-invariant procedure** — the retail agent reasons its way (again) to "don't call account lookup before the customer gives an email"; the household agent rediscovers (again) that `heat X` is one atomic command. The paper's sharpest single statistic: in τ²-retail, one recurring bug — calling an authentication tool with a fabricated argument — appears in **59% of non-reasoning training rollouts and accounts for 94% of observed tool errors**. Recurring computation is exactly what amortization is for.

**Passive skill distillation** is the method, and its virtue is that it is almost embarrassingly simple. Roll out the model once on a training split (35–50 tasks per domain). Point an off-the-shelf coding agent ([[claude-code|Claude Code]] with Claude Sonnet 5) at the directory of trajectory logs with a fixed natural-language instruction. The agent **writes and runs its own analysis code** over the corpus — error-type frequencies, action n-grams, loop detection, win/loss contrasts — reads the individual episodes the statistics point at, and compiles **40–130 lines of markdown** whose rules are concrete and traceable to transcript evidence. Example rule from the retail skill, quoted verbatim in the paper: *"Before calling `find_user_id_by_email`, check that the customer's message actually contains a real email address … this bug appeared in 13 of 22 rollouts and accounted for 17 of 18 tool errors."* The skill is then appended verbatim to the **non-reasoning** model's system prompt. Nothing else changes — same harness, decoding, tools.

Crucially the distiller has **no environment access**; it only reads trajectory files and mode-level pass rates. No weights updated, no new rollouts collected, no per-instance search at deployment, and the skill sits in a fixed **cacheable prefix**. One pass over logs production systems already store, at **$1.28–$2.44 per domain**.

The framing that makes it more than an engineering trick: reasoning is **deep search** inside one episode, re-paid every deployment; corpus distillation is **wide search** across episodes, paid once. They buy the same commodity — procedural domain knowledge — on different cost structures, and where the knowledge is episode-invariant, width strictly dominates.

## Key points

**Main results** — held-out success (mean of 3 seeds) and mean output tokens per episode; "reduction" is vs. the same model's think mode:

| Benchmark | Mode | GPT-5.4-mini score | tokens | red. | Qwen3.6-27B score | tokens | red. |
|---|---|---|---|---|---|---|---|
| ALFWorld | think | 0.713 | 3,723 | – | 0.773 | 9,232 | – |
| | no-think | 0.567 | 952 | 3.9× | 0.827 | 991 | 9.3× |
| | **no-think + skill** | **0.787** | 832 | 4.5× | **0.980** | 619 | 14.9× |
| SSB-Verified | think | 0.613 | 3,291 | – | 0.560 | 2,826 | – |
| | no-think | 0.447 | 960 | 3.4× | 0.640 | 2,432 | 1.2× |
| | **no-think + skill** | 0.560 | 831 | 4.0× | **0.673** | 2,729 | 1.0× |
| τ²-telecom | think | **0.450** | 2,143 | – | **0.933** | 6,058 | – |
| | no-think | 0.192 | 421 | 5.1× | 0.883 | 985 | 6.2× |
| | **no-think + skill** | 0.333 | 597 | 3.6× | **0.933** | 1,026 | 5.9× |
| τ²-retail | think | 0.350 | 1,615 | – | **0.633** | 4,124 | – |
| | no-think | 0.325 | 536 | 3.0× | 0.600 | 1,058 | 3.9× |
| | **no-think + skill** | **0.408** | 565 | 2.9× | 0.558 | 1,180 | 3.5× |

For GPT-5.4-mini the think mode leads no-think by +14.6, +16.6, +25.8, +2.5 points; the skill recovers **55%–100%+** of that gap on all four, and **beats the reasoning mode outright on ALFWorld (0.787 vs 0.713) and τ²-retail (0.408 vs 0.350)** — with **zero reasoning tokens**.

> [!warning] Internal inconsistency in the paper's own token-reduction range
> The abstract and conclusion claim "**2.7–6×** fewer output tokens"; the contributions list says "2.7–5×"; Section 5.1 says "**2.9–4.5×**". Only the last is reproducible from Table 1 (GPT-5.4-mini skill rows: 4.5 / 4.0 / 3.6 / 2.9×). The Qwen rows span 1.0–14.9×, which is not 2.7–6× either. **Use 2.9–4.5× for GPT-5.4-mini**; treat the 2.7–6× headline as unsupported by the reported table.

Beating the teacher is not paradoxical, and the paper explains why: **a rule aggregated over 50 training episodes is more reliable than a derivation the reasoning model must reproduce correctly each time**. The reasoning model itself occasionally falls into the ALFWorld appliance-door loop the skill forbids outright.

**The skill also makes episodes shorter, not just cheaper per turn.** On ALFWorld and SSB the skill undercuts even the plain no-think baseline in tokens, because fewer flailing retries means shorter episodes — **21.8 vs 27.0 turns** on ALFWorld. Mechanistically: the missed-transform failure drops from **35.9% → 11.5%** of transform tasks, and stall loops from **28.7% → 5.3%**, together accounting for most of the win-rate gain. The worked example is a held-out "put a cool tomato in microwave" task where the baseline treats "cool" as a property rather than a required action, then issues `look` twenty times until the 40-step cap; the skill supplies the `cool X with fridge` rule plus a no-op-loop break, and the same model finishes in 30 steps.

**Distillation source ablation** (GPT-5.4-mini; both skills injected into the same non-reasoning model):

| Benchmark | Think-distilled | No-think-distilled |
|---|---|---|
| ALFWorld | 0.813 | 0.787 |
| SpreadsheetBench | 0.460 | **0.560** |
| τ²-telecom | 0.325 | 0.333 |
| τ²-retail | **0.458** | 0.408 |

Mixed, in both directions: close on ALFWorld and telecom, paired wins retail by ~5 points, **no-think-only wins SSB-Verified by 10 points**. The conclusion the authors draw is deliberately narrow — reasoning traces are *not a prerequisite* — and it has a real operational payoff: the full loop (**deploy cheap agent → collect logs → distill → redeploy**) can run without ever invoking a reasoning model. Their proposed mechanism for the SSB reversal is elegant: reasoning traces record **what the model believed**, while environment feedback in failed no-think trajectories records **what was true**; verbose narratives may anchor the distiller on the former. They flag this as un-isolated, and note each skill was distilled once so distillation variance is uncontrolled.

**vs. an automatic prompt optimizer** — [[gepa|GEPA]], 120 metric-call budget, Claude Sonnet 5 reflection:

| Domain | no-skill | think | GEPA | Ours | GEPA cost | Our cost |
|---|---|---|---|---|---|---|
| τ²-retail | 0.325 | 0.350 | 0.392 | **0.458** | $2.26 | **$1.28** |
| τ²-telecom | 0.192 | 0.450 | 0.308 | **0.325** | $13.02 | **$2.44** |

Higher on both domains at **4.1× lower production cost** ($3.72 vs $15.28). The gap is driven by GEPA's active rollouts per proposed prompt — extending the telecom budget to 240 metric calls produced a **byte-identical prompt**, i.e. convergence, not budget starvation. The deeper structural point: active optimizers spend an evaluation-rollout budget before any savings accrue, **and cannot run at all where fresh rollouts are unavailable**; passive distillation runs on logs alone.

**Amortization economics** — per episode the skill saves T_think − T_skill output tokens, e.g. **2,143 − 597 = 1,546** on telecom, essentially the model's entire 1,572-token reasoning budget, while adding only a fixed cacheable input prefix. The one-time few-dollar cost is repaid once cumulative savings exceed it; every subsequent episode is pure savings.

**Where it breaks** — the residual think-over-skill gap is itself the finding. Telecom (0.450 vs 0.333) and SSB-Verified (0.613 vs 0.560) mark knowledge that is **not episode-invariant**: telecom hinges on long instance-specific dependency chains in a dual-control environment (which line, which plan, what the user just toggled), and spreadsheet tasks embed one-off logical structure no fixed prompt anticipates. There the two mechanisms are **complements** — a skill to stop re-buying the invariants, reasoning reserved for the instances that need it.

The Qwen3.6-27B replication is the honest counterweight. Distillation helps on 3 of 4 (ALFWorld 0.980, +15.3 over no-think; telecom matching think at 0.933 with 5.9× fewer tokens) — but **retail regresses −4.2 points**, and the authors' reading is that with a near-zero think/no-think gap and an already-competent base, added rules may **over-constrain**. Note also that Qwen's own reasoning mode is unreliable, *hurting* on ALFWorld (−5.4) and SSB-Verified (−8.0), so "recovering the reasoning gap" is not even the right frame for that model.

**Stated limitations** — skills distilled once per model–domain pair, so evaluation variance is measured (3 seeds) but **distillation variance is not**, and the Qwen retail regression suggests the process is not uniformly reliable. Two models, four domains. Skills are **model-specific and per-domain; cross-model transfer is untested**.

## Concepts & entities

- [[agent-skills]] — the deployment artifact: a markdown skill in the system prompt. Notable inversion of the usual framing — here a skill is valued as a *substitute for an expensive reasoning mode*, measured in reasoning-gap recovery per token.
- [[skill-acquisition]] — acquisition by an *external* coding agent reading logs, not by the acting model itself.
- [[knowledge-distillation]] — distillation without weight updates: procedure extracted from trajectories into natural language.
- [[reasoning-effort]] — the exact knob being amortized (`reasoning_effort` ∈ {none, medium}; `enable_thinking` ∈ {false, true}).
- [[test-time-compute]] / [[inference-time-compute]] — the deep-vs-wide search reframing is a direct argument about where to spend it.
- [[chain-of-thought]] — the per-episode re-derivation this replaces with a compiled rule.
- [[experiential-learning]] — closest kin: NL insights extracted from trajectories, but framed as a reasoning-cost substitute rather than a memory.
- [[prefix-caching]] — why the skill's deployment cost is negligible: fixed cacheable system-prompt prefix.
- [[context-assembly]] — the skill is a static prefix, not retrieved per instance.
- [[prompt-optimization]] — the compared family (GEPA, OPRO, DSPy/MIPROv2, TextGrad); this is the passive, log-only alternative.
- [[agent-benchmark]] / [[agent-evaluation]] — ALFWorld, SpreadsheetBench-Verified, τ²-bench telecom/retail.
- [[alfworld]] — text-based embodied household tasks, ReAct-style, 40-step cap, held-out random-50.
- [[tau-bench]] — τ²-bench dual-control conversational customer service with a simulated user; 40-task held-out splits.
- [[llm-agent]] — the system class.
- [[voyager]] / [[reflexion]] / [[expel-zhao-2023]] — the experiential-learning lineage the paper positions against.
- [[claude-code]] — the distiller harness (with Claude Sonnet 5).
- [[microsoft]] — author affiliation.
- [[self-improving-agent]] — the deploy → log → distill → redeploy loop, notably closable without ever invoking a reasoning model.

## References

- [arXiv:2608.07885](https://arxiv.org/abs/2608.07885)
- GEPA: Agrawal et al., ICLR 2026 — [arXiv:2507.19457](https://arxiv.org/abs/2507.19457)
- [[expel-zhao-2023]] — insight extraction from experience, the NL-artifact precursor
- [[agent-skills-survey-xu-2026]] — where externally-authored skills sit in the acquisition taxonomy
