---
title: "CORAL: An LLM-Native Harness for Production Recommender Systems"
type: source
created: 2026-09-07
updated: 2026-09-07
sources: []
arxiv_id: "2609.02730"
venue: RecSys '26 OARS Workshop
authors:
  - Muhammad Rafay Azhar
  - Yuhang Zhou
  - Gilbert Jiang
  - Yuchen Wang
  - Rahul Sharma
  - Matthew DeSousa
  - Jiayi Liu
  - Xin Guo
  - Lizhu Zhang
  - Xiangjun Fan
year: 2026
tags:
  - 2026
  - paper
status: complete
importance: high
---

# CORAL: An LLM-Native Harness for Production Recommender Systems

**Muhammad Rafay Azhar, Yuhang Zhou, Gilbert Jiang, Yuchen Wang, Rahul Sharma, Matthew DeSousa, Jiayi Liu, Xin Guo, Lizhu Zhang, Xiangjun Fan** (all [[meta|Meta AI]]) — [arXiv:2609.02730](https://arxiv.org/abs/2609.02730), cs.CL, 2 Sep 2026. **Accepted at the RecSys '26 OARS Workshop.** CC BY 4.0.

## Why this one is worth reading precisely

Almost every agentic-harness paper in this wiki is evaluated on a benchmark. This one is evaluated with **online A/B experiments on two live Meta social platforms**, with the agent writing directly to the production control surface. That combination — a closed-loop LLM agent acting on a deployed system serving at social-network scale, with measured treatment effects rather than a leaderboard row — is rare enough that the numbers deserve to be read for exactly what they say and no more.

CORAL = **C**onstraint-**O**ptimized **R**ecommender via an **A**gentic **L**oop.

## Problem

A modern industrial recommender is a multi-stage system (retrieval → ranking → serving) governed by a large collection of hand-set parameters: retrieval budgets, ranking weights, caching and prefetch policies, per-segment treatments. These are set by engineers, not learned end-to-end, and the paper's diagnosis of the status quo is a process critique rather than a modeling one:

- each experiment probes a small region of a vast design space, so progress is **bounded by engineering headcount, not by the size of the opportunity**;
- changes are prompted by observed regressions, so the process is **reactive rather than anticipatory**;
- cycles are expensive, so parts of the system are **revisited infrequently and drift** away from their best operating point between interventions.

The drift is worst for **low-signal and new users**, "whose behavior is underrepresented in the aggregate metrics that guide most decisions."

## Mechanism

**Formulation.** A partially observed, non-stationary, *constrained* optimization. Configuration `s` over N control units; at cycle t the system yields objective `J_t(s)` (engagement) and cost `c_t(s) ≤ B` (a fixed budget). The oracle optimum `s*_t` moves with the operating context, so the goal is stated as minimizing cumulative shortfall `min_π Σ_t [J_t(s*_t) − J_t(s_t)]` subject to `c_t(s_t) ≤ B ∀t` — "the agent's aim is therefore not to converge to a single optimum but to keep the deployed configuration close to this moving target." The formulation is symmetric: it covers maximizing engagement under a cost cap **and its dual**, minimizing cost while holding engagement at target. The two deployments take one form each.

**The loop (verified: observe / reason / optimize / measure).** The policy is an LLM mapping `s_t = π(o_t, M_t)`, run on a **fixed cadence of k days, invoking components in the same fixed order every cycle rather than leaving control flow to the model**:

1. **Observe** — `o_t` summarizes per-unit operating statistics over the preceding window plus their change from the previous window.
2. **Reason** — analyze the statistics, retrieve relevant history from memory, estimate the effect of the previous configuration.
3. **Optimize** — propose bounded per-unit adjustments; the **constrained optimizer** projects them onto the budget-feasible set.
4. **Measure** — the configuration stays in effect for a full cycle; its effect is measured with an A/B experiment and written back to memory.

**Memory** spans the most recent m cycles in three stores: an **observation store** (raw `o_t`), an **assessment store** (the model's own prior natural-language judgments of what is working), and a **decision store** (past configurations plus outcomes later attributed to them). Adaptation is entirely in context — **no parameter updates, no retraining**.

**Tools**, all deterministic and returning structured results: analysis (compute statistics and deltas), retrieval (surface history from memory), attribution (estimate the effect of the previous configuration), a **constrained optimizer**, and an apply tool that writes to the control surface.

**The constrained optimizer is the load-bearing safety component, and its semantics are precise.** Given the model's bounded per-unit proposals it "returns the closest configuration that satisfies the operating budget — the projection of the proposal onto the budget-feasible set {s : c_t(s) ≤ B}." Critically: "When the proposal already respects the budget, this projection returns it unchanged; **it binds only when the proposal would overspend**, redistributing across units so that the deployed configuration provably satisfies `c_t(s_t) ≤ B`." So the budget is a hard guarantee by construction, not a prompt instruction — the division of labor is "the model does what it is best at — weighing many heterogeneous signals and articulating why a change should help — while delegating numerical feasibility to a component that can guarantee it."

**Deployment settings:** k = 3 days cadence, m = 3 cycles of memory. The authors are explicit that "we chose these as sensible defaults rather than tuning them," and name systematic selection of k and m as active work.

## What the agent was permitted to change

This is the sharpest thing to be precise about, because the answer is narrower than "optimizes a recommender" suggests. The per-cycle prompt (Appendix A) is a single decision call whose action space is closed: *"Select only from this set; do not invent actions."* Both case studies use **the same template**, differing only in placeholders:

| Placeholder | Retrieval-budget case | Serving-capacity case |
|---|---|---|
| control unit | retrieval source | user segment |
| feasible set | bounded **continuous multiplier** | ordered **discrete treatment menu** |
| engagement metric | video-viewing sessions, watch time | engaged sessions, time spent |
| operating budget | retrieval budget | serving compute budget |

Output is JSON: per unit an `adjustment` drawn from the feasible set, a `rationale`, and a `confidence` (high/medium/low), plus a cross-unit `assessment`. The agent therefore moved **numbers on a pre-declared control surface within pre-declared bounds**. It did not touch model weights, code, ranking logic, or retrieval logic. The paper says so itself in its limitations: both studies "instantiate the harness on a single class of decision — the constrained allocation of a recommender's resources across its components — and extending the same loop to qualitatively different levers, **such as the retrieval and ranking logic itself**, would further test the generality it is designed for."

## The human-in-the-loop boundary

Stated plainly and worth quoting, because it is *not* an approval gate:

> "The loop runs autonomously, applying each cycle's configuration to the live recommender directly. A human supervises its operation, tracking the effect of its changes and the decisions it makes; as confidence in the agent grows, this supervision is progressively replaced by the harness's automated guardrails — feasibility checks, bounded-change limits, and safety constraints."

So the boundary is: **the agent writes to production without per-decision human sign-off**; the human is an *observer* of decisions and their measured effects, not a gate the change must pass. Safety comes from three places instead — the closed action space (bounded feasible sets), the optimizer's hard budget projection, and the cycle cadence that keeps any single change reversible within k days. The paper positions supervision as a transitional stage: the discussion claims the loop "compress[es] a tuning cycle from several engineer-weeks to a few autonomous days, an order-of-magnitude faster turnaround **with no engineer in the loop**," while limitation #1 concedes "although the loop runs autonomously, it still operates under human supervision; strengthening the harness's automated guardrails so that this supervision can be reduced is a natural next step." Both statements are the authors'; the second is the calibrated one.

## Measured results

**Case study 1 — retrieval budget across candidate sources** (large-scale video service; control unit = retrieval source, cost = total retrieval budget consumed). Three successive A/B-evaluated rounds; R1 is a zero-shot proposal from a single window of statistics, R3 aggregates several decision cycles and is the deployed configuration:

| Metric | R1 | R2 | R3 |
|---|---|---|---|
| Watch time | **+0.13%** | neutral | **+0.15%** |
| Sessions (all users) | neutral | neutral | **+0.16%** |
| Sessions (largest market) | neutral | neutral | **+0.77%** |

"neutral" = no statistically significant change. **The progression is explicitly non-monotonic** and the paper foregrounds it: in R2 "the agent shifted budget more aggressively between sources but overcorrected, and its measured effect was neutral." The authors' reading — "rather than a gain in every round, the agent reacts to the measured effect of its own decisions and, over successive rounds, converges on a better configuration" — is the honest framing of an N=3 sequence.

The converged global allocation ran across **millions of users** for **+0.16% sessions and +0.15% watch time at no additional serving cost**, because "the reallocation consolidated the retrieval budget rather than expanding it." Extending to per-segment allocations (by engagement level and account tenure) raised video-viewing sessions **+0.23% for new low-signal users**, by shifting budget toward sources drawing on content and current-context signals and away from those relying on rich user histories.

**Case study 2 — serving capacity across user segments** (different service; control unit = user segment, treatment chosen from a menu ranging lightweight → compute-intensive, varying retrieval/ranking aggressiveness, caching and prefetch; cost = serving compute, capped). A/B test over millions of users, two rounds: round 1 "reduced serving cost substantially, saving millions of USD in annualized capacity expenditure"; reading that outcome, the agent widened the allocation to additional segments in round 2, **increasing round 1's savings by 44% while leaving engagement statistically unchanged**.

**Two-platform outcome, as claimed: verified.** Engagement up at no additional serving cost on one platform; serving cost down without degrading engagement on the other — the same harness on both, spanning the engagement–efficiency frontier.

**Compute cost.** `Cost = (T/k) · C · (τ_in·p_in + τ_out·p_out)`. The structural point: C is fixed by how the control surface is partitioned into decision groups (a handful per cycle), **not by traffic** — so cost is independent of user count and stays negligible on billion-user surfaces, "in stark contrast to per-item or per-user LLM inference."

| Case study | Calls/cycle | Input tokens/call | Output tokens/call |
|---|---|---|---|
| Retrieval-budget | ~10 | ~1,500 | ~2,500 |
| Serving-capacity | ~8 | ~2,000 | ~2,500 |

Total ≈ 10⁶ tokens per deployment, "on the order of tens of U.S. dollars" at frontier pricing. Note the table's own caveat: these are **estimated from prompt and payload sizes because "the pipelines do not log token usage."**

## Limitations

The paper's stated three:

1. **Still human-supervised.** Autonomy is the direction, not the current state; stronger automated guardrails are named as the next step.
2. **One class of decision.** Both deployments are constrained resource allocation. Retrieval and ranking *logic* is untouched.
3. **A/B evidence is expensive and setting-specific.** "A standardized way to evaluate agent-driven system optimization before deployment remains an open problem, and one we hope this work helps motivate."

Additional reading caveats that matter for citing this:

- **The backbone model is never named** — only "a general-purpose LLM" and "representative frontier-LLM pricing." There is no model-swap study, so nothing here separates harness contribution from model capability (cf. [[harness-vs-model-attribution]]).
- **Scale precision.** The abstract's "billions" describes the *platforms*; the A/B experiments are reported as spanning **millions of users**. The cost argument is what extends to billion-user surfaces.
- **Effect sizes are small in absolute terms** (+0.13% to +0.77%). At Meta scale these are real and expensive to obtain, but they are not the kind of number that survives being quoted without its denominator.
- **No baseline against a human engineer or a non-LLM optimizer.** The comparison is against the unrevised status-quo configuration. The claim that the LLM's *reasoning* beats, say, plain Bayesian optimization over the same bounded control surface is not tested — and given the action space is a low-dimensional bounded box with a projection operator, that is the obvious missing arm.
- **N is tiny per case** (3 rounds and 2 rounds). "Its decisions improve as the loop iterates" rests on those sequences, one of which regressed in the middle.

## Related

- [[agentic-harness-engineering]] — CORAL calls itself a harness in exactly this sense: context + tools + loop around a frozen model.
- [[alphaeval-evaluating-agents-in-production-lu-2026]] — the other production-evaluation datapoint in the wiki; CORAL's limitation #3 is the same gap from the deployment side.
- [[harness-continual-learning-kang-2026]] — a loop that updates persistent harness state and measures forgetting; CORAL's memory is bounded at m=3 cycles, which sidesteps that failure by construction and gives up long-horizon accumulation for it.
- [[self-improving-agent]] — improvement without weight updates, from measured consequences of the agent's own prior actions; CORAL is an unusually literal instance because the feedback is a real A/B treatment effect rather than a benchmark score.
- [[experiential-learning]], [[in-context-learning]] — the policy "improves in context, without parameter updates."
- [[continual-learning]], [[online-learning]] — the non-stationary target `s*_t` is the paper's central framing.
- [[memory-management]], [[long-term-memory]], [[episodic-memory-llm]] — the three-store observation/assessment/decision memory.
- [[mem0-chhikara-2025]], [[memgpt-packer-2023]], [[generative-agents-park-2023]] — cited as the agent-memory line CORAL builds on.
- [[tool-use]], [[function-calling]] — five deterministic tools; the optimizer is the one that supplies a hard guarantee.
- [[guardrailing]] — feasibility checks, bounded-change limits, safety constraints as the intended replacement for human supervision.
- [[structured-outputs]] — the JSON decision schema with per-unit rationale and confidence.
- [[verifier]] — the constrained optimizer is a projection-based verifier that repairs rather than rejects.
- [[learned-orchestration]] — an LLM allocating a bounded budget across system components.
- [[llm-agent]], [[agent-evaluation]] — the loop, and the "how do you evaluate this before deployment" open problem.
- [[lifecycle-of-llm-as-a-judge-recommendation-explanations-kong-2026]] — the wiki's other LLM-meets-recommender source, from the explanation side.
- [[stateless-decision-memory-for-enterprise-ai-agents-srinivasan-2026]] — contrast on how much state a production decision loop should carry.
- [[meta]] — Meta AI; both deployments are Meta social platforms.

## Concept gaps

- No page for **recommender systems** at all — retrieval/ranking/serving multi-stage architecture, the thing being optimized here.
- No page for **A/B testing / online experimentation** as an agent feedback signal, which is the mechanism that makes this loop different from a benchmark loop.
- No page for **constrained optimization / feasible-set projection** as a harness safety primitive (the pattern of an LLM proposing and a deterministic solver guaranteeing).
- No page for **agents acting on live production systems** as a category distinct from benchmark agents.

## References

- [arXiv:2609.02730](https://arxiv.org/abs/2609.02730)
