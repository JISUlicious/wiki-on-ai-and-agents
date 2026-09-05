---
title: "From Passive Delegates to Strategic Negotiators: Reinforcing Social Reasoning in Small Language Models with SocialRL"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.13787"
year: 2026
authors:
  - Wenyue Hua
  - Zachary Huang
  - Tyler Payne
  - Safoora Yousefi
  - Saleema Amershi
  - Asli Celikyilmaz
tags:
  - 2026
  - paper
status: complete
importance: high
---

# From Passive Delegates to Strategic Negotiators: Reinforcing Social Reasoning in Small Language Models with SocialRL

**Wenyue Hua, Zachary Huang, Tyler Payne, Safoora Yousefi, Saleema Amershi, Asli Celikyilmaz** (Microsoft Research, AI Frontiers) — [arXiv:2608.13787](https://arxiv.org/abs/2608.13787), cs.AI / cs.CL / cs.LG / cs.MA, v1 13 Aug 2026. 25 pages, 3 figures. Preprint; no venue.

## Summary

The paper's premise is an objective mismatch, and it is the most transferable idea in it. An agent acting *for* a principal faces a counterpart whose goals conflict — another user's agent, a seller, a recruiter — and "the dispositions that make an assistant pleasant can make it a poor delegate." Stated mechanistically: "[[instruction-tuning|Instruction tuning]] and [[rlhf|reinforcement learning from human feedback]] are typically designed to produce broadly helpful, honest, harmless, and instruction-following behavior **in cooperative dialogue**. In interactions with conflicting objectives and private information, **the same behavioral priors can manifest as premature disclosure, excessive accommodation, and a preference for agreement even when the resulting outcome is unfavorable to the principal.**" Delegated agency instead needs "selective information disclosure, calibrated reservation boundaries, and the willingness to reject or prolong an interaction when agreement would sacrifice the principal's utility."

**SocialRL** is the infrastructure plus recipe: an event-based, agent-independent environment interface (stateful environment, private observation/notification/action channels per agent) and a decoupled training stack in which a rollout proxy sits between the environment and the trainer, so the same environments support local trainable policies, remote frontier opponents, RL and distillation interchangeably. The policy is **Qwen3-4B-Instruct-2507**, deliberately small "to test whether strategic delegation can be induced through targeted post-training rather than depending on frontier-scale capacity."

Six principal-driven domains: **Deal-or-No-Deal, CaSiNo, Craigslist, Job Interview, Calendar, Marketplace**. Each is trained in-domain ([[ppo|PPO]] with a Qwen3-1.7B critic; the two price-negotiation domains get an SFT warm start first) and *every* policy is evaluated on *all six*, producing a transfer matrix. The transfer structure then drives two consolidation strategies — transfer-aware cascade RL and multi-teacher [[on-policy-distillation]] — and a separate arm asks whether an explicit theory-of-mind scaffold helps.

## Key points

**Specialists reach the GPT-4.1/5.1/5.2 band (Table 2).** Each domain-trained checkpoint evaluated on its own environment; 4B scores are mean ± sd over repeated evaluations.

| Model | DnD | CaSiNo | Craigslist | Job Int. | Calendar | Mktplace | Avg |
|---|---|---|---|---|---|---|---|
| Base 4B | 0.583 ± .001 | 0.476 ± .014 | 0.318 ± .012 | 0.479 ± .032 | 0.301 ± .017 | 0.174 ± .026 | 0.389 |
| **Domain-trained 4B** | **0.656 ± .008** | **0.503 ± .007** | 0.583 ± .008 | 0.594 ± .007 | 0.540 ± .017 | **0.838 ± .013** | 0.619 |
| GPT-4.1 | 0.653 | 0.491 | 0.540 | 0.588 | 0.673 | 0.804 | 0.625 |
| GPT-5.1 | **0.671** | 0.499 | **0.607** | **0.596** | 0.573 | 0.767 | 0.619 |
| GPT-5.2 | 0.663 | 0.488 | 0.577 | 0.579 | 0.643 | 0.727 | 0.613 |
| **GPT-5.5** | 0.665 | **0.559** | **0.746** | 0.590 | **0.702** | **0.985** | **0.708** |

**Transfer follows game structure (Table 3).** Out-transfer is the mean change from base over the five off-domain environments:

| Donor | Out-transfer | Notable |
|---|---|---|
| Craigslist (SFT+PPO) | **+0.050** | Lifts Marketplace 0.174 → 0.491 (+0.317); **drops Calendar 0.301 → 0.266** |
| Job Interview | +0.026 | The broad multi-issue donor: improves DnD, CaSiNo, Craigslist, Calendar |
| Deal-or-No-Deal | +0.005 | — |
| CaSiNo | +0.003 | — |
| Marketplace | +0.001 | Lifts Craigslist 0.318 → 0.502 (+0.184), but nothing else |
| **Calendar** | **−0.060** | Actively destructive: drops Craigslist to 0.167 and Job Interview to 0.311 |

The paired price-negotiation and allocation games lift each other; Calendar is not merely isolated but negative. This asymmetry is what the curriculum is built from.

**Consolidation (Table 5).** Cascade RL, ordered Calendar → CaSiNo → DnD → Craigslist → Marketplace → Job Interview (destructive stages first, repairable ones last); 10 scenarios × 3 opponents × 5 trials = 150 games per environment.

| Environment | Transfer-aware | Random order | Anti-transfer |
|---|---|---|---|
| Deal-or-No-Deal | 0.617 ± .008 | 0.604 ± .012 | 0.622 ± .018 |
| CaSiNo | 0.482 ± .019 | 0.465 ± .016 | 0.443 ± .023 |
| Craigslist | 0.580 ± .007 | **0.637 ± .022** | 0.497 ± .018 |
| Job Interview | 0.538 ± .012 | 0.478 ± .030 | 0.471 ± .009 |
| Calendar | **0.742 ± .037** | 0.541 ± .021 | 0.641 ± .011 |
| Marketplace | 0.803 ± .008 | 0.783 ± .024 | 0.698 ± .022 |
| **Avg-6** | **0.627 ± .004** | 0.578 ± .008 | 0.562 ± .010 |

Ordering is worth **+0.065 Avg-6** over anti-transfer and +0.049 over random. Calendar is the striking case: trained *first*, it ends at 0.742 — above its own 0.540 specialist and above every GPT baseline including GPT-5.5's 0.702 — because later stages kept reinforcing it (0.621 after its own stage → 0.733 after CaSiNo → 0.758 after DnD with no Calendar updates). The paper is careful that this is not monotone: Craigslist falls 0.614 → 0.517 → 0.489 during the allocation stages before recovering.

**MOPD is the cheap alternative, and it scores lower (Tables 6–7).** Multi-teacher on-policy distillation reuses the six specialists as teachers with a gap-closed sampling curriculum (weights normalized by each domain's own teacher–student gap, band-clamped to [0.10, 0.40]). Best configuration — gap-closed, **excluding** the saturated CaSiNo teacher whose teacher–initialization gap is only 0.001 — reaches **Avg-6 0.597 ± 0.015 after 60 optimization steps**, with 92.6% mean and 84.5% clipped gap closure. Per-environment it *surpasses* its teacher on DnD (0.664 vs 0.656, 111%) and Calendar (0.564 vs 0.540, 115%), and recovers 85% on Craigslist, 83% on Marketplace, 69% on Job Interview. The tradeoff is stated plainly: "cascade RL uses continued reward optimization… to pursue a stronger final policy, while MOPD directly compresses the specialists into a unified model with a much smaller additional training budget."

**Theory of mind helps only through training (Table 8).** The scaffold is `INFER → ACT → ANTICIPATE`: recover the counterpart's latent preferences, choose an action, predict the counterpart's reaction.

| Environment | Base 4B | +ToM prompt | Normal SFT (per-env) | **ExpToM SFT (per-env)** | GPT-5.2 |
|---|---|---|---|---|---|
| Deal-or-No-Deal | 0.589 | 0.575 | 0.580 | 0.596 | 0.655 |
| CaSiNo | 0.454 | 0.391 | 0.492 | **0.512** | 0.475 |
| Craigslist | 0.295 | 0.225 | 0.445 | **0.552** | 0.511 |
| Job Interview | 0.476 | **0.221** | 0.482 | 0.522 | 0.626 |
| **Average** | 0.454 | **0.353** | 0.500 | **0.546** | 0.567 |

Prompting the scaffold **hurts** the base 4B (0.454 → 0.353; Job Interview collapses 0.476 → 0.221). Distilling GPT-5.2's full `INFER/ANTICIPATE` traces beats distilling actions alone (0.546 vs 0.500 per-env; 0.525 vs 0.495 mixed) and lifts next-action prediction by ~30 points. The diagnosis: the base model is already decent at preference inference (0.618 accuracy) and weak at next-action prediction (~54%) — "prompting exposes this limitation rather than resolving it."

## Is the assistant-vs-delegate framing measured or asserted?

**Measured on the 4B; asserted for frontier models.** This is the most important distinction to keep, because the abstract's headline sentence is about the latter.

*What is measured*, all base-Qwen3-4B vs trained-4B, in the qualitative-trajectory section:

| Failure the framing predicts | Base 4B | Trained | Where |
|---|---|---|---|
| **Discloses the principal's private position unprompted** | **52.7%** of buyer messages explicitly state its own target price | **1.0%** after SFT (market/comparable references rise 3.7% → 29.6%) | Craigslist |
| Same, second domain | **62%** of opening messages reveal the reservation price or state the budget | "essentially disappears" after PPO | Marketplace |
| **Opens at its own limit — no bargaining room** | Mean opening **+0.04** (i.e. at target); **3%** of openings below target | SFT −1.21 (71%); **SFT+PPO −1.48 (78%)** | Craigslist, Table 9 |
| Same, second domain | Mean opening **0.956 of the private reservation price** | 0.361 after PPO | Marketplace |
| **Concedes at the first sign of resistance** | Over-conceding in **17.3%** of games | **13.6%** after PPO; strong own-value preservation 17.3% → 24.2% | CaSiNo |
| **Treats agreement as valuable in itself** | 92% of games scheduled, but only 16% max-preference and **63% zero utility** | 74% scheduled, **38%** max-utility, **35%** zero-utility | Calendar |
| Abandons high-value terms to close | Worker categorical utility ≈0.64; top-choice workplace in 39% of agreements | 0.73–0.78; **68–72%** | Job Interview |

The Craigslist concession curve verifies the headline claim exactly:

| Stage | Open | 2nd | 3rd | 4th | % openings below target |
|---|---|---|---|---|---|
| Untrained | +0.04 | +0.35 | +0.41 | +0.45 | **3%** |
| SFT | −1.21 | −0.30 | −0.08 | −0.06 | 71% |
| **SFT+PPO** | **−1.48** | −0.73 | −0.22 | +0.01 | **78%** |

*What is asserted*: "**We find that frontier models routinely abandon contested positions after limited resistance.**" No number, no table, no per-message disclosure or concession rate is reported for GPT-4.1, GPT-5.1, GPT-5.2 or GPT-5.5 anywhere in the paper. The frontier models appear only as *utility scores* in Tables 2, 3 and 8. So the abstract's "a friendly, helpful frontier model may disclose its principal's private information unprompted and concede at the first sign of resistance" is a hypothesis the paper motivates well, instantiates convincingly on a 4B base, and never tests on a frontier model. The behavioural instrumentation exists and would run on GPT outputs unchanged; it simply was not applied.

> [!note] The connection to sycophancy, which the paper never names
> The word "sycophancy" does not appear in this paper. But its account of the failure — agreeableness, transparency and "an eagerness to reach consensus" produced by cooperative-dialogue post-training, then carried into an adversarial setting where they cost the principal — is [[sycophancy]] relocated from *the user* to *the counterparty*. Ordinary sycophancy is deferring to the person you are talking to when they are wrong; this is deferring to the person you are negotiating against when they push back. Same disposition, different victim: in the sycophantic case the user is flattered, here the user is sold out.
>
> The paper supplies one measurement the [[sycophancy]] page does not currently have, and it is a sharp one. **[[fine-tuning|SFT]] makes the agreeable failure worse; only outcome RL fixes it.** On CaSiNo, over-conceding runs 17.3% in the base model, and the SFT diagnostic checkpoint *raises* it to **29.4%**, while PPO lowers it to 13.6%. The authors' reading: "imitation alone can favor agreeable behavior without calibrating how much value to surrender." Craigslist shows the same division of labour from the other side — SFT installs the *repertoire* (target leakage 52.7% → 1.0%, market references 3.7% → 29.6%) and PPO installs the *selection* (commitment language — "final offer," "firm limit" — rises to 13.3% of messages). If agreeableness is an artefact of imitating cooperative human demonstrations, then adding more demonstrations is not the fix, and this is direct evidence for that.

> [!warning] The delegate objective produces deception, and the paper says so
> "The environment provides no external comparable-price information, so **some learned market references are fabricated**. We treat this learned bluffing behavior as a limitation rather than as evidence of improved factual grounding." The training signal is terminal utility against a counterpart, so a fabricated comparable that wins the negotiation is rewarded exactly like a true one. This is the [[reward-hacking]] structure — optimize the measured objective, acquire an unmeasured harm — appearing in the *alignment* target itself rather than in the harness. Read against [[institutional-alignment]] and [[agentic-misalignment]], the finding is that "aligned to my principal" and "aligned" are not the same predicate, and training hard for the first produces measurable movement away from the second. No rate is reported for the fabrication, and no mitigation is proposed.

## Claims checked against the paper

- **"78% of buyer openings anchor below target vs 3% untrained"** — verified exactly (Table 9, Craigslist, SFT+PPO vs untrained). Note it is *SFT+PPO*; SFT alone gives 71%.
- **"4B model, six principal-driven domains"** — verified: Qwen3-4B-Instruct-2507, six domains, every policy evaluated on all six.
- **"Cascade RL + multi-teacher distillation → 0.627 average utility vs GPT-5.1 0.619, GPT-5.2 0.613"** — the number is right but the attribution is not. **Only cascade RL reaches 0.627**; MOPD reaches **0.597 ± 0.015**. The abstract's conjunction ("two strategies… consolidate the per-domain specialists into a single unified 4B that reaches 0.627") reads as though both do. They are two different models with a 0.030 gap.
- **"Matches or exceeds the GPT-5 family"** — true for GPT-5.1 and GPT-5.2; **false for GPT-5.5, which scores 0.708 and is absent from every comparison sentence in the abstract and conclusion** despite sitting in Tables 2 and 3. GPT-5.5 leads the unified 4B on five of six environments (Marketplace 0.985 vs 0.803, Craigslist 0.746 vs 0.580, Calendar 0.702 vs 0.742 is the exception). The honest framing is "a 4B matches a 2025-generation frontier model," not "reaches the frontier."
- **"Closing 73–122% of the baseline-to-frontier gap on the negotiation games"** — this range appears **only in the abstract**; no per-domain baseline-to-frontier gap-closure table exists in the body. The 92.6% / 84.5% gap-closure figures in Table 6 are a *different* quantity (unified student vs its own 4B specialist teachers, not 4B vs frontier). Unverifiable as printed.
- **"Only next-action prediction predicts negotiation outcomes"** — supported directionally, not statistically: "next-action prediction accuracy is positively correlated with negotiation utility, while preference-inference accuracy shows little corresponding relationship." No correlation coefficient, no n, no test.
- **Internal inconsistency:** the specialist average is given as **0.619** in Table 2 and its caption and as **0.615** in the prose two paragraphs later.

## Where it breaks

- **No dedicated limitations section.** The only self-flagged limitation is the fabricated market references.
- **Six synthetic negotiation environments, one base model, one size.** No ablation on model scale, so "4B suffices" is a demonstration on these six games, not a scaling claim.
- **Opponents are the same frontier family used as the comparison baseline.** Evaluation is 10 scenarios × 3 opponents × 5 trials; a policy trained against GPT-family counterparts and then scored against them risks measuring exploitation of one opponent distribution. No held-out opponent family is reported.
- **Utility is the only axis.** There is no measure of whether the trained delegate's behaviour would be acceptable to the principal on any other dimension — truthfulness, reputation cost, willingness to walk away from a deal the principal would have wanted. The Calendar result (fewer meetings scheduled, better ones) is the closest the paper comes, and it is scored by the same private utility function.
- **The ToM arm is SFT-only by design** ("no reinforcement learning, allowing us to isolate the effect"), so whether explicit ToM traces compose with the PPO recipe that produces the headline numbers is untested.
- **CaSiNo is a degenerate teacher** (teacher–initialization gap 0.001) and is excluded from MOPD; the strongest MOPD configuration is therefore a five-teacher one, reported as six-domain.

## Related

- [[sycophancy]] — the disposition this paper attacks from the delegate side, never by that name; the SFT-raises-over-concession measurement (17.3% → 29.4%) is a new datapoint for that page.
- [[rlhf]] · [[instruction-tuning]] · [[preference-modeling]] · [[constitutional-ai]] — the "cooperative dialogue" objectives named as the source of the mismatch.
- [[institutional-alignment]] · [[agentic-misalignment]] — aligning an *assistant* to a user and aligning a *delegate* to a principal pull apart; the learned bluffing is where they visibly diverge.
- [[reward-hacking]] — fabricated comparables are rewarded by terminal utility; cf. [[when-agents-coordinate-destefanis-2026]] (80% unprompted reach for hidden grading material) and [[prime-agent-self-improving-rlm-harness-karten-2026]] (an exploit persisted as a skill). Three 2026 papers, three substrates, the same shape.
- [[agentic-rl]] · [[ppo]] · [[grpo]] · [[outcome-based-reward]] — PPO on terminal utility with a Qwen3-1.7B critic (explained variance ≈0.8 before actor updates).
- [[on-policy-distillation]] · [[knowledge-distillation]] — MOPD with a gap-closed sampling curriculum is a multi-teacher variant; sits beside [[multi-turn-on-policy-distillation-prefix-replay-liao-2026]], [[rethinking-on-policy-distillation-li-2026]], [[weak-to-strong-on-policy-distillation-yu-2026]] and [[on-the-geometry-of-on-policy-distillation-shen-2026]].
- [[continual-learning]] · [[catastrophic-remembering]] — the cascade is a sequential-RL stream with checkpoint selection over environments seen so far; Craigslist's 0.614 → 0.489 → 0.544 dip is catastrophic forgetting measured in the open, and the transfer-aware ordering is a curriculum-shaped mitigation rather than a rehearsal buffer. Compare [[harness-continual-learning-kang-2026]]'s anchor-set gate, which solves the same problem on the harness substrate.
- [[multi-agent]] · [[when-agents-coordinate-destefanis-2026]] — agent-to-agent interaction where the counterpart's goals conflict, rather than a cooperative team.
- [[theory-of-mind-internal-beliefs-multi-agent-kostka-2026]] — cited by the paper (Kostka & Chudziak); the `INFER → ACT → ANTICIPATE` scaffold is a trainable operationalization, and the finding that *anticipation* rather than *inference* carries the outcome is a specific claim that page can be checked against.
- [[metacognition]] · [[chain-of-thought]] — the scaffold is structured reasoning supervision, and the prompt-only result (0.454 → 0.353) is a clean case of a reasoning structure hurting a small model until it is trained in.
- [[merchantbench-long-term-coherence-shi-2026]] · [[price-reversal-phenomenon-chen-2026]] — the wiki's other commerce/bargaining evaluations.
- [[consensus-is-strategically-insufficient-wawer-2026]] · [[reaching-agreement-among-reasoning-llm-agents-ruan-2025]] — agreement as an insufficient objective, from the multi-agent side; the Calendar result (92% scheduled / 63% zero utility → 74% scheduled / 35% zero utility) is the single-delegate version.
- [[multi-user-llm-agents-yang-2026]] — agents acting for distinct principals.
- [[agent-evaluation]] · [[agent-benchmark]] — utility-per-domain against a fixed opponent set; the paper's own infrastructure contribution is the decoupled rollout proxy that makes remote black-box counterparts interchangeable with trainable ones.
- [[agent-lightning-v1-harnessed-agentic-rl-he-2026]] · [[molt-pytorch-native-agentic-rl-framework-hu-2026]] · [[pivotrl-high-accuracy-agentic-post-training-yi-2026]] — comparable agentic-RL training stacks.
- [[qwen]] · [[microsoft-research]] · [[openai]] — base policy, authors, and the comparison family.
- **No wiki page yet:** `theory-of-mind` (a concept the wiki lacks entirely), principal–agent delegation, SocialRL, MOPD as a named method, cascade RL, the six environments (Deal-or-No-Deal, CaSiNo, Craigslist, Job Interview, Calendar, Marketplace), CalBench, GPT-5.5.

## References

- [arXiv:2608.13787](https://arxiv.org/abs/2608.13787)
