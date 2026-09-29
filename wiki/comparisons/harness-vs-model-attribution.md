---
title: "Harness vs. Model: Performance Attribution (factorial studies)"
type: comparison
created: 2026-06-22
updated: 2026-09-29
sources:
  - cross-agent-comparison-doc-2026.md
  - stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026.md
  - terminal-bench-benchmarking-agents-cli-merrill-2026.md
status: complete
importance: high
tags:
  - 2026
---

# Harness vs. Model: Performance Attribution

**The question:** does a study run the factorial — *agent A with model B, agent B with model A* — to separate the contribution of the **harness** (scaffold) from the **base model**? **Yes.** Agentic benchmark **leaderboards are exactly this grid**: each entry is a `(harness, model)` pair, so the same model appears under multiple harnesses and the same harness under multiple models. This page collects the production-level evidence and reconciles it with the wiki's research-harness factorials ([[agentic-harness-engineering]]).

> [!note] Two senses of "harness comparison"
> This page is the **measured performance** cut (who wins, and how much is the scaffold). For the **architectural** comparison of the five agents see [[cross-agent-comparison-2026]]; for the **component anatomy** see [[fundamental-components-of-harness]].

## Same model, different harness (scaffold effect)

Holding the **model fixed** and swapping only the harness, from the Terminal-Bench 2.0 leaderboard:

| Model (fixed) | Harness A | Harness B | Gap = scaffold effect |
|---|---|---|---|
| GPT-5.3-Codex | Factory **Droid** — 77.3% | OpenAI **Simple Codex** — 75.1% | **2.2 pts** |
| Claude Opus 4.6 | KRAFTON **Terminus-KIRA** — 74.7% | Bigai **TongAgents** — 71.9% | **2.8 pts** |
| (same weights) | LangChain agent — **66.5% (rank 5)** | their earlier scaffold — 52.8% | **13.7 pts** |

A bad-vs-good scaffold swing reaches **~13–15 pts**; a good-vs-good swing is typically **~2–6 pts**. [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026|Zhang et al. 2026]] quantify scaffold-only variation (model held fixed) at **11 pts for GPT-5 and 15 pts for Kimi K2 on SWE-bench Verified**, a **45.9%→55.4%** lift on SWE-bench Pro from swapping the SEAL scaffold for Claude Code, and HAL-leaderboard swings of **up to ~48 pts**. This matches the wiki's controlled result: [[agentic-harness-engineering-lin-2026|AHE]] lifts Terminal-Bench 2 from **69.7%→77.0%** with the model held fixed, beating human-designed Codex (71.9%).

## Same harness, different model

The inverse is cleanest on **provider-agnostic** harnesses. Because [[opencode]] and [[pi-mono]] run ~20–25 providers, they are the natural fixed-scaffold substrate: *"Run Opus through OpenCode and you get Claude-Code-like scores; run GPT-5.5 and you get Codex-like scores"* — the harness is roughly constant, the model moves the number. [[claude-code]] resists this test (Anthropic-first), which is **why no clean five-agent × N-model public grid exists** — only per-harness leaderboard rows.

## The headline finding (and the caveat)

| Finding | Source |
|---|---|
| A model's benchmark score is **meaningless without disclosing the harness** | [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] (arXiv:2605.23950) |
| Scaffold-only variation: **11 pts (GPT-5) / 15 pts (Kimi K2)** on SWE-bench Verified; up to **~48 pts** on HAL | [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] |
| [[terminal-bench]] is a **(harness × model) leaderboard** by design — 89 hard CLI tasks, no model finishes them | [[terminal-bench-benchmarking-agents-cli-merrill-2026]] |
| Harness benefit **scales inversely with model saturation** — weak models gain most | [[agentic-harness-engineering-lin-2026]] (+10.1pp DeepSeek-V4 vs +2.3 GPT-5.4) |
| "Updating" a harness is flat across model tiers; "benefiting" is non-monotonic (peaks mid-tier) | [[harness-updating-is-not-harness-benefit-lin-2026]] |
| Feedback **quality** predicts success (R²≈0.99) far better than raw budget (R²≈0.33) | [[scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026]] |

> [!warning] At the frontier, the model still dominates
> The scaffold is the *tiebreaker*, not the headline, among top systems: a model gap typically exceeds the good-vs-good scaffold gap (~2–6 pts). Harness engineering matters most for **non-frontier models** and for **closing the gap to** the frontier, less for re-ranking frontier models against each other.
>
> **Moving-target caveat on the numbers:** at the [[terminal-bench-benchmarking-agents-cli-merrill-2026|Terminal-Bench paper]]'s publication, the top entry was **Codex CLI + GPT-5.2 at 63%** on the 89-task v2 (frontier resolution <65%, small models ~15%; Terminus 2 is the *neutral* reference scaffold, Harbor the execution harness). The *live* leaderboard has since climbed to ~77–83% (e.g. Codex CLI + GPT-5.5) as newer models/harnesses land — so absolute scores age fast; the *attribution* finding (scaffold variance) is the durable result, not any single leaderboard row.

## Reconciliation with the research-harness cluster

The production leaderboards and the wiki's research-harness factorials agree:

```mermaid
graph LR
  Q[["Factorial: harness × model"]] --> P["Production leaderboards (Terminal-Bench, SWE-bench): (harness,model) rows"]
  Q --> R["Research harnesses (model frozen, scaffold optimized)"]
  P --> F[["Finding: scaffold = 2–6 pts (good); 13–15 pts (good vs bad); frontier model still dominates"]]
  R --> F
  R --> AHE[["agentic-harness-engineering"]]
  P --> DISC[["'Disclose the harness' (arXiv 2605.23950)"]]
```

Both say the same thing from opposite directions: **you cannot attribute an agent's score to the model alone.** The research side proves it by *optimizing* the scaffold on a frozen model; the production side proves it by *reading off* the variance across leaderboard rows.

## Gaps
- No public study runs the **full five-agent ([[claude-code]]/[[opencode]]/[[pi-mono]]/[[hermes-agent]]/[[openclaw]]) × N-model** grid — Claude Code's provider lock-in blocks the clean version.
- The leaderboard rows mix harness *and* harness-config *and* prompt, so per-row attribution is coarse; *Efficient Benchmarking of AI Agents* (arXiv:2603.23749) addresses methodology.

## Related
- [[cross-agent-comparison-2026]] — architectural comparison of the five agents.
- [[fundamental-components-of-harness]] — the 8-component harness anatomy.
- [[agentic-harness-engineering]] — optimizing the harness as an object; the research-side factorials.
- [[terminal-bench]] — the shared yardstick these grids are read from.
- [[price-reversal-phenomenon-chen-2026]] — the cost dimension of model×harness pairing.

## References

- [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026]] — the disclosure thesis (Zhang et al., Tulane; arXiv:2605.23950)
- [[terminal-bench-benchmarking-agents-cli-merrill-2026]] — the Terminal-Bench benchmark (Merrill et al., Stanford/Laude/Anthropic; arXiv:2601.11868); see [[terminal-bench]]
- [[agentic-harness-engineering-lin-2026]] · [[harness-updating-is-not-harness-benefit-lin-2026]] · [[scaling-laws-agent-harnesses-effective-feedback-compute-zhang-2026]]
- [[cross-agent-comparison-2026]] · [[fundamental-components-of-harness]]
- Leaderboards: benchmarkingagents.com/terminal-bench · morphllm.com/terminal-bench-2 · digitalapplied.com (SWE-bench Verified scaffolding analysis)

## Related 2026 sources

Surfaced via newsletter ingests; see [[index]].

- [[the-harness-effect-token-economics-writer-2026]] — 22 tasks × 6 models with only the orchestration layer swapped: efficiency gains are model-invariant, but quality gain tracks base-model strength at r=0.99
- [[rethinking-evaluation-harness-evolution-wang-2026]] — harness *evolution* fails to beat plain test-time scaling at matched budgets (67.4 vs 72.3)
- [[nvidia-oo-agents-furgale-2026]] — RHAE 85.1% vs 13.3% for the raw model on ARC-AGI-3 — a 6.4× harness effect on one backbone

> [!success] Resolved contradiction — the 13.3% baseline may be the wrong model
> [[prime-agent-self-improving-rlm-harness-karten-2026|Prime Agent]] (2026) reports the ARC-AGI-3 reference figures with a different attribution: **13.3% is GPT-5.6 *Terra***, while GPT-5.6 **Sol** — the backbone NVIDIA used — is listed at **38.3%** ([[responses-api|Responses API]]) and **7.0%** (ARC harness). Against 38.3% the NVIDIA harness effect is **2.2×**, not 6.4×; against 7.0% it is 12.2×.
>
> Both papers cite external ARC Prize numbers rather than running the raw baseline themselves, so this is a citation conflict, not a measurement conflict. **Treat 6.4× as unverified** until the raw Sol baseline is pinned down. The qualitative claim — that the harness is worth a large multiple on this benchmark — survives either way.
>
> A second data point in the same table cuts against the newer system: **on the same backbone (GPT-5.6 Sol), NVIDIA's 85.1% beats Prime Agent's 78.3%.** Prime Agent's headline 95.5% comes from switching to Claude Opus 5, not from a better harness — and its authors say the reference numbers "situate the result rather than isolate a causal harness effect."
>
> Note also that a widely-circulated "NVIDIA AVO at 100% on ARC-AGI-3" figure (AI Agents Weekly, 2026-08-22) **could not be verified against any paper** and is recorded only as product news.
>
> **Resolved 2026-09-29 against ARC Prize's own results pages** ([Sol](https://arcprize.org/results/openai-gpt-5-6-sol), [Terra](https://arcprize.org/results/openai-gpt-5-6-terra), both dated 2026-07-09). GPT-5.6 **Sol** at max reasoning effort averages **13.33% on the 25-game Public set** and 7.78% on Semi-Private; **Terra** averages **2.3%** Public and 0.80% Semi-Private. So 13.3% belongs to Sol, and Prime Agent's legend mislabels it as Terra. NVIDIA's comparison is like-for-like: same model, same game set, same effort. **The 6.4× figure (85.1 / 13.33) stands**, though evaluation budgets differ, NVIDIA itself calls it indicative, and its 85.1% is a self-reported community-leaderboard entry. The alternatives do not hold as raw baselines. 7.0% and 7.8% are **Semi-Private** scores (7.0% matches Sol at XHigh effort, 7.8% at Max), so dividing a Public-set 85.1% by them crosses game sets. 38.3% is OpenAI's self-reported Public-set result with Responses API retained reasoning and compaction switched on, a harnessed configuration rather than a raw baseline.

> [!warning] Superseded 2026-09-29: this update compared across game sets
> The reasoning below — that 38.3% is itself harnessed — is right. The conclusion is wrong: 7.8% is Sol's **Semi-Private** score, while NVIDIA's 85.1% and the 13.33% baseline are both on the **Public** set. The ~11× multiple therefore crosses game sets, and 6.4× is the like-for-like figure (see the resolved callout above). Kept below for the record.

> [!note] Update 2026-09-07 — a third number, and it points the other way
> OpenAI's **GPT-6 Astra launch comparison** reports **GPT-5.6 Sol at 7.8%** on ARC-AGI-3. That is a third independent figure for the same raw model, and it sits next to Prime Agent's **7.0%** ("ARC harness") rather than its 38.3% ("Responses API").
>
> The most likely reconciliation: **38.3% is itself a harnessed number** — the Responses API supplies its own agentic loop — while ~7–8% is the genuinely raw baseline, and NVIDIA's 13.3% is a fourth measurement condition. If the raw Sol baseline is ~7.8%, NVIDIA's 85.1% is roughly an **11× harness effect, larger than the 6.4× originally recorded**, not smaller.
>
> So the direction of the correction has flipped since this callout was first written. The scaffold-attribution conclusion strengthens; only the specific multiplier was wrong. All three figures remain vendor- or third-party-reported with no common protocol, which is precisely the disclosure problem [[stop-comparing-llm-agents-without-disclosing-the-harness-zhang-2026|Zhang et al.]] argue makes cross-system agent numbers uncomparable — this page is now its own worked example.
>
> Separately, Astra is reported at **99.9%** on ARC-AGI-3. If that holds, the benchmark is saturated and ceases to discriminate harnesses at all.
- [[j-space-cognition-suite]] — a worked example: every claimed gain (+4.4 to +16.0) falls inside the harness-substitution band this page quantifies, and the control column comes from a vendor model card rather than a matched run
- [[ai-native-sdlc]] — why this page matters operationally: SDLC narratives attribute outcomes to model capability, while scaffold-only variation moves them 11–15 points with the model fixed
- [[frognano-training-a-4b-coding-agent-via-online-task-synthesis-kim-2026]] — harness sensitivity depends on the model: an 8.3%→37.2% harness swing for a 4B base, none for MiniMax-M2.5 (66.5% under both)
- [[co-evolving-harnesses-and-models-on-policy-correction-yu-2026]] — harness and fine-tuning recipe interact: a recipe that helps under one harness hurts under another
