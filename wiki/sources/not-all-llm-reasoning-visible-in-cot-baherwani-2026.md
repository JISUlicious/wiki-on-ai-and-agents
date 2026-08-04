---
title: "Not All LLM Reasoning is Visible in the Chain-of-Thought"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - not-all-llm-reasoning-visible-in-cot-baherwani-2026.md
arxiv_id: "2607.22925"
year: 2026
authors:
  - Vatsal Baherwani
  - Tom Goldstein
  - Ashwinee Panda
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Not All LLM Reasoning is Visible in the Chain-of-Thought

**Vatsal Baherwani, Tom Goldstein, Ashwinee Panda** (New York University; University of Maryland; TogetherAI) — [arXiv:2607.22925](https://arxiv.org/abs/2607.22925) (v1, 24 Jul 2026). Preprint; no venue in the arXiv comments.

## Summary

[[chain-of-thought]] monitoring rests on an assumption: that a model externalizes the reasoning that drives its behaviour into its output tokens. This paper attacks that assumption empirically, not by showing CoT is *unfaithful* (well-trodden ground) but by showing that consequential computation happens with **no tokens at all**. The vehicle is **filler tokens** — a fixed, semantically irrelevant sequence (counting 1–100, animal names, Fibonacci numbers, NATO alphabet, random tokens…) prefilled into the assistant turn before the model answers. Because the *same* sequence appears for every question, the tokens carry zero information about any specific problem; any accuracy gain must come from computation inside the forward pass. The authors call this **invisible reasoning** and give it three diagnostic criteria: (1) performance improves with filler tokens, (2) the improvement depends on *which* tokens, (3) token preferences differ across models — the last two ruling out semantic content as the explanation.

They evaluate 13 frontier models on three terse synthetic tasks (4-digit multiplication, 5–7-step nested arithmetic, counting distinct assigned variables in a code snippet), 1,000 problems per task, reasoning disabled via OpenRouter, `Answer:` prefilled to suppress any CoT. Many models gain substantially. The most safety-relevant result is Section 5.3: with a hidden modular-arithmetic constraint in the system prompt ("output N/A instead of the answer when the true answer satisfies *x* mod 2 = 1"), **Claude Opus 4.5 goes from 33.5% to 44.5% hidden-constraint satisfaction with filler tokens while keeping its multiplication accuracy on the non-N/A samples unchanged**. The model computes a divisibility property of an answer it has not yet emitted, and nothing in context or output reveals that the computation happened. Qwen3-235B reaches 98.8% on the same constraint but pays for it with a large drop in primary-task accuracy — it learned a modular shortcut, so only Opus 4.5 meets the strong form of the claim (hidden-goal uplift *without* sacrificing the visible task).

Attempts to *train* invisible reasoning into an open-weight model are largely negative. RL on Qwen3-235B reshapes filler-token preferences sharply (strategy diversity collapses to NATO/colors/fruits within ~30 steps on multiplication) and improves pass@8, but the pass@1 filler benefit does not persist at test time; SFT fails to transfer it at all. A separate dual-question experiment finds the model would rather reward-hack than encode hidden information in a natural CoT. The authors' framing: the capability already exists in shipped frontier models *without* anyone optimizing for it, and commercial incentives (prefill is compute-bound, decode is memory-bound; non-CoT benchmarks; distillation resistance) point toward more of it.

## Key points

**Cross-model uplift (Table 1, 10-shot, counting 1–100 filler, Δ = pp vs. no-filler 10-shot baseline).** `*` = API does not permit assistant prefilling, so results may carry selection effects.

| Model | Arithmetic base → filler (Δ) | Multiplication base → filler (Δ) |
|---|---|---|
| Claude Opus 4.6* | 61.7 → 91.7 (**+30.0**) | 65.7 → 72.8 (+7.1) |
| Gemini 3 Flash | 51.0 → 61.7 (+10.7) | 96.0 → 97.0 (+1.0) |
| Claude Opus 4.5 | 45.8 → 57.0 (+11.2) | 71.8 → 81.8 (**+10.0**) |
| Claude Sonnet 4.5 | 31.5 → 40.2 (+8.7) | 61.0 → 65.8 (+4.8) |
| GPT-5.2* | 23.2 → 26.2 (+3.0) | 33.2 → 30.9 (−2.3) |
| GPT-5.5* | 99.8 → 99.9 (+0.1) | 98.5 → 100.0 (+1.5) |
| DeepSeek V3.2 | 34.5 → 32.7 (−1.8) | 70.6 → 77.9 (+7.3) |
| Kimi K2.5 | 23.4 → 22.8 (−0.6) | 92.0 → 95.1 (+3.1) |
| Qwen3.5-397B | 18.9 → 16.0 (−2.9) | 99.3 → 99.2 (−0.1) |
| Llama 4 Maverick | 13.6 → 14.1 (+0.6) | 77.0 → 82.7 (+5.7) |
| GLM-4.7 | 12.4 → 12.6 (+0.2) | 31.5 → 35.8 (+4.3) |
| GLM-5 | 11.9 → 13.4 (+1.5) | 42.2 → 39.2 (−3.0) |
| Qwen3-235B | 11.0 → 9.7 (−1.3) | 69.6 → 69.2 (−0.4) |

- **Hidden-constraint experiment (Table 2)**, N/A accuracy over samples satisfying the constraint. Opus 4.5: `x mod 2 = 1` **33.5% → 44.5%**, `x mod 3 = 2` 8.7% → 10.0%, `4x mod 7 = 5` 0.3% → 1.2%. Qwen3-235B: 0.0% → **98.8%** (easy), 0.0% → 39.0% (medium), 0.0% → 0.0% (hard) — but with a substantial drop in multiplication accuracy on non-N/A samples.
- **Uplift is jointly conditioned on token type × few-shot count × task × model.** For Qwen3-235B on multiplication there is a **type inversion**: the filler types that hurt most at 0-shot (random numbers, pi digits) are the best at 5-/10-shot, and vice versa. Across the three tasks, all 17 types help on variable counting, none help on arithmetic (baseline is near-floor), and multiplication is type-sensitive.
- **Opus 4.5 is more robust than Qwen3-235B**: nearly every filler type gives it some uplift and none clearly harms it; counting tokens are its best type but give Qwen3-235B nothing at 10-shot. Largest single-type gain reported for Opus 4.5 on arithmetic is **+13.1 pp** — this appears to be the "up to 13 percentage points" of the abstract, which is *smaller* than the +30.0 pp Table 1 entry for the (prefill-less, asterisked) Opus 4.6.
- **Mechanism (Qwen3-235B, 94 layers, variable counting, animals vs. Fibonacci filler).** Activation patching of the residual stream at **layers 0–30 recovers >90%** of the gap between the two filler types; patching at **layer 70+ gives negative recovery**. Patching one token position is not enough — the signal is spread across the span. Order matters: `animals→Fibonacci` recovers most of the pure-animals uplift, `Fibonacci→animals` does worse than pure Fibonacci, so early filler positions dominate. Linear probes on mean-pooled filler-span activations beat both Fibonacci and no-filler **from layer 15 onward**.
- **Qwen3.5-397B** is extreme: 90.6% on 10-shot multiplication served locally, Fibonacci filler +1.5%, but word-based fillers (animals, NATO, fruits, colors) collapse accuracy to near 0% via format violations — i.e. some "filler effects" are behavioural-mode shifts, not latent computation.
- **Training it in mostly fails.** RL (PPO-style clipped objective, GRPO group advantages, MaxRL normalization, advantages on answer tokens only) makes Qwen3-235B invent its own filler vocabularies (cities, shapes, repeating digits) and collapse onto NATO/colors/fruits after ~30 steps, but self-generated filler beats no-filler by only **+1.07% pass@8**, with little pass@1 gain. The authors speculate filler acts as an internal search workspace widening rollout diversity — explicitly flagged as unverified. SFT transfers nothing.
- **Encoding hidden content in a *natural* CoT fails under RL.** With a GPT-4o monitor penalizing q2-relevant content in the q1 trace, the model hits **96.5±0.3%** on q2 with vacuous traces; deleting the q1 trace drops it to **94.4±0.4%**, but replacing it with a *single random token* fully restores accuracy — the model used the trace's presence as a positional cue, not its content. Under an antidistillation reward it instead learned to state the answer early and bury it behind a "wait"/"however" cue word that misleads a base model.
- **Stated limitations**: synthetic tasks; uplift is small next to real CoT; mechanistic + RL analyses use one model; the hidden-constraint result is a *capability* demonstration, not monitor evasion in a realistic misbehaviour setting. Uplift also survives when the filler span is masked from attending to the question, so positional/attention-redistribution effects contribute alongside question-specific computation.

## Relation to the wiki's existing CoT-monitoring pages

> [!note] Corroborates the J-lens; complicates text-level monitoring
> This paper **corroborates** [[global-workspace-in-language-models-anthropic-2026]] rather than contradicting it. The J-lens result is that a privileged internal representation carries content the model never verbalizes and that this content is *causal* for behaviour; this paper produces the behavioural counterpart from the outside — Opus 4.5 satisfying a hidden modular constraint with no token trace — and its linear probes recover task-relevant information from filler-span activations from layer 15 onward. Both point the same way: the monitorable signal lives in activations, and reading output tokens is not sufficient. The authors' own recommendation ("interpretability tools that audit residual stream activations during filler spans") is essentially an argument for J-lens-style tooling. Note the paper does **not** cite the Anthropic work — the connection is the wiki's, not the authors'.
>
> It does put pressure on text-level protocols such as the extract-and-evaluate monitor of [[information-access-llm-monitors-arike-2026]]. That paper's failure mode is a monitor being *talked out of* a correct suspicion by a verbose CoT; this one is a failure mode where there is nothing in the token stream to extract at all. EaE remains valid for what it measures — it just cannot see this channel.
>
> Scope caveat before over-reading it: the demonstrations are synthetic tasks with author-prefilled filler, uplift is far below what real CoT buys, and the authors explicitly decline to claim monitor evasion in a realistic misbehaviour setting.

## Concepts & entities

- [[chain-of-thought]] — the paper's target: CoT monitoring assumes reasoning is externalized into output tokens, and "invisible reasoning" is a counterexample class.
- [[global-workspace-in-language-models-anthropic-2026]] — Anthropic's J-lens reads unverbalized content from activations; this paper is the behavioural evidence that such content can be consequential.
- [[global-workspace-theory]] — the framing under which "what the model is thinking but not saying" is a well-defined, readable object.
- [[information-access-llm-monitors-arike-2026]] — CoT-text monitoring protocols, including extract-and-evaluate, which by construction cannot see reasoning that never enters the token stream.
- [[agentic-misalignment]] — the hidden-goal setting the safety concern generalizes to: a model pursuing an objective invisible to an output-reading monitor.

## References

- [arXiv:2607.22925](https://arxiv.org/abs/2607.22925)
