---
title: "JEV-as-a-Judge: Accept When Confident, Escalate When Unsure"
type: source
created: 2026-09-30
updated: 2026-09-30
sources:
  - jev-as-a-judge-accept-when-confident-li-2026.md
arxiv_id: "2609.26550"
authors:
  - Yubo Li
  - Yidi Miao
  - Ramayya Krishnan
  - Rema Padman
year: 2026
status: complete
importance: high
tags:
  - 2026
  - paper
---

# JEV-as-a-Judge: Accept When Confident, Escalate When Unsure

**Yubo Li, Yidi Miao, Ramayya Krishnan, Rema Padman** ([[cmu|Carnegie Mellon University]]) — [arXiv:2609.26550](https://arxiv.org/abs/2609.26550), cs.AI, v3 29 Sep 2026. Preprint; no venue given. Funded in part by NIST; OpenAI supplied API credits.

## Summary

This is the first full-length, pre-specified academic evaluation of [[jev|Jev]] (v1.13) as an [[llm-as-a-judge|LLM judge]]. It asks one practical question: can a cheap judge that only returns label probabilities settle routine verdicts itself, and pass the uncertain ones to an expensive reasoning judge? Jev is compared with 16 other judges (14 generative LLMs and 2 reward models) on 5,172 base judgments: RewardBench, JudgeBench, HaluEval and a final-answer set. Every generative judge answers under Jev's contract, which means a verdict and a probability for each label, with no rationale. Confidence is `q` = the largest label probability, which is **Jev's `Choice` output type**. Blinded human adjudication covers every item that Jev or GPT-6 Astra gets wrong within a 990-item subset.

The answer is yes, within a boundary. Where the verdict can be read off the text, Jev is close to GPT-6 at **277× lower fee** ($0.044 vs $12.18 per 1,000 judgments) and a 0.15 s median latency. Where the verdict must be *derived* (math, code, logic, expert knowledge), it trails badly. Its confidence marks that boundary well enough that a frozen cascade matches GPT-6 for less.

## Key points

- **Mid-pack accuracy.** Pooled over 4,850 public judgments, Jev ranks 8th of 15 LLM judges, 1.7 points behind GPT-6. RewardBench is a tie at 92.5% each. HaluEval is 87.3 vs 88.4 and final answers 94.0 vs 96.7. On **JudgeBench it scores 78.6 vs 93.1 (−14.6)**, and human adjudication sides with GPT-6 on 57 of the 69 disputed items. Under human-corrected labels GPT-6's leads on RewardBench (3.0) and HaluEval (2.5) exclude zero.
- **Confidence locates the errors.** At two-order q ≥ 0.9 the two judges are near-tied (94.7% vs 94.4% on 3,744 items). Only 3 of Jev's 95 order-reversal flips occur at q ≥ 0.9. Single-call error-detection AUROC is 0.875, 0.745 and 0.827 on RewardBench, JudgeBench and HaluEval, against 0.894, 0.907 and 0.831 for GPT-6.
- **Frozen cascade** (τ = 0.9, fixed before held-out outcomes were seen). It accepts 68.5% of 1,610 held-out pairs and escalates 31.5%, which hold 85% of Jev's errors. Accuracy is **93.4% vs GPT-6's 92.5% (+0.93, CI [0.24, 1.66]) at 41.4% of the fee**. On JudgeBench alone it escalates 65% and closes 94% of the gap.
- **Prospective live test** (protocol frozen before any call). This used 570 pairs from PPE and JudgeBench's Claude split, with τ picked per workload by a lower-confidence-bound rule on 100 local labels. The cascade **matched GPT-6 exactly (90.2% vs 90.2%)**, escalating 74% at 75.5% of the fee. Jev alone scored 76.1% there.
- **Where it fails.** On RM-Bench "hard" pairs, where the wrong answer is written more elaborately, Jev trails by 13.5 and AUROC drops to 0.764. On **reference-free prose** every judge is near chance but confident: Jev scores 53.5% with mean q 0.91, Brier 0.821 and **AUROC 0.498**, so no threshold helps.
- **Interface consistency.** On 48 examples, `Choice` and `Noul` probabilities for the same binary question differ by 0.055 on average, and the two Noul probabilities miss summing to 1 by 0.045. The authors say to tune thresholds for one fixed output type.

## Calibration: what it reports, and how it sits against the audits

It reports calibration explicitly, not only accuracy: ten-bin **ECE**, Brier, NLL and **reliability curves (Fig. 6)**. Jev's ECE is **0.01 on RewardBench, 0.03 on JudgeBench and 0.07 on HaluEval** (Fig. 20). Those values are as good as or better than most generative judges. On the 240 human-adjudicated HaluEval items the ordering flips: ECE 0.017 for Jev vs 0.010 for GPT-6. Temperature scaling does **not** transfer: the fitted T is 0.65, 2.15 and 4.45 on the three tasks, and it helps on only one of them.

> [!warning] Contradiction, partly: judge-mode ECE
> [[independent-jev-calibration-audits-2026]] records BargLabs' pre-registered audit: *"Jev as a judge of agent reports: **ECE 0.458**, against a pre-registered 0.05 gate; it passed 90% of defective reports."* This paper finds judge-mode ECE of **0.01–0.07** on public preference and factuality benchmarks. The two are not directly comparable. BargLabs judged free-form agent reports, and this paper's own **reference-free prose** result (near chance, mean q 0.91, AUROC 0.498) looks like that condition and agrees with it. The reconciliation is that Jev as a judge is well calibrated where a reference or a comparison pair anchors the verdict, and confidently wrong where none does. The audit page's summary line that *"judge-style use is badly miscalibrated"* is therefore too broad.

The paper **neither confirms nor tests** the audit claim that `choice` probabilities saturate like a step function. It uses Choice throughout, and its q distribution does pile up near 1 (595 of 1,850 pairs are at base-order q = 1, Fig. 7a). But its errors fall mostly below the gate, so saturation does not break the cascade on these tasks. The Choice-vs-Noul gap of 0.055 is consistent with the audits' advice not to treat the two types as interchangeable.

## What it does not show

- It is not compute-matched, and training overlap with the benchmarks is unknown. It tests one proprietary Jev version, and only serial escalation.
- Only the frozen policies, the adjudication and the prospective test were pre-specified. The multi-family comparison and the threshold sweeps are exploratory.
- The adjudicators are team members, and the audit covers only disputed items, not a random sample.
- A related study (Rao & Callison-Burch 2026b, not in the wiki) found that cheap judges repeat Jev's confident errors. The cascade here works because GPT-6 fixes 60 of Jev's 75 JudgeBench errors. Only 15 are shared.

## Related

- [[jev]] · [[system-one-model]] · [[typesafe-ai]] · [[logit-scoring]] — the object under test, and the decision-only interface.
- [[calibration]] · [[independent-jev-calibration-audits-2026]] · [[jev-research-papers-september-2026]] — see the callout above.
- [[laya]] — two open-weight checkpoints are reported in Appendix L; they are near chance on preference pairs.
- [[llm-as-a-judge]] · [[reliability-without-validity-llm-as-a-judge-norman-2026]] — the same JudgeBench and RewardBench family, and the same caution about agreement metrics.
- [[reward-modeling]] · [[verifier]] · [[inference-time-compute]] — the cascade prices reasoning effort only on escalated items.
- [[hallucination-detection]] — HaluEval as the evidence-grounded factuality task.

## References

- [arXiv:2609.26550](https://arxiv.org/abs/2609.26550)
