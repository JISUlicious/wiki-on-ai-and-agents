---
title: "Policy Optimization Methods in LLM Post-Training"
type: query
created: 2026-05-22
updated: 2026-05-22
sources:
  - rlhf-christiano-2017.md
  - instructgpt-ouyang-2022.md
  - constitutional-ai-bai-2022.md
  - dpo-rafailov-2023.md
  - deepseek-r1-2025.md
  - search-r1-jin-2025.md
  - memory-r1-yan-2025.md
status: complete
importance: high
tags:
  - 2026
---

# Policy Optimization Methods in LLM Post-Training

A synthesis of how the wiki's coverage of [[ppo]], [[grpo]], [[dpo]], [[rlhf]], [[rlaif]], [[constitutional-ai]], and the agentic-RL thread fit together as one family.

## What it is

**Policy optimization** is the family of methods for *adjusting a model's output distribution* (its **policy**, in RL terms) to maximize a reward, preference, or correctness signal — rather than to imitate next-token labels. In the LLM stack it is the dominant **post-training** mechanism: after pre-training and SFT, the policy is shaped to be helpful, harmless, capable at tools, or correct at reasoning. The umbrella is [[reinforcement-learning]] in classical framing, but a large modern subfamily ([[dpo|DPO]] and successors) skips the RL loop and optimizes preferences directly.

Three axes characterize any policy-optimization method:

1. **Reward source**: human labels, AI labels, outcome signal (test passes / answer correct), rule-based formula.
2. **Mechanism**: full RL loop (sample-act-reward-update) vs. closed-form supervised loss on preferences.
3. **Critic**: separate value network needed (PPO) or eliminated (GRPO, DPO).

## Taxonomy

### Family A — RL-based policy gradient

These keep an explicit "policy → sample → reward → gradient" loop. Reward comes from a learned model ([[rlhf]]), an AI judge ([[rlaif]]), or an outcome verifier ([[agentic-rl]]).

| Method | Year / Origin | Critic? | Reward source | Defining feature |
|---|---|---|---|---|
| [[ppo]] — Proximal Policy Optimization | Schulman et al. 2017, [[openai]] | **Yes** (value net) | Reward model | Clipped ratio between new/old policy; PPO was the workhorse for [[instructgpt]] / ChatGPT |
| [[grpo]] — Group Relative Policy Optimization | DeepSeek 2024 (DeepSeekMath; canonized by [[deepseek-r1]]) | **No** — group-mean baseline | Reward model or rule-based outcomes | Samples `G` candidates per prompt, normalizes advantages by group mean/std; cuts memory cost |
| **TRPO** — Trust Region Policy Optimization | Schulman 2015 | Yes | RL reward | Predecessor of PPO; harder to implement, mostly historical |
| **REINFORCE / PG** | Williams 1992 | No | RL reward | Classical baseline; rarely used at LLM scale |
| **RLOO** — REINFORCE Leave-One-Out | Ahmadian et al. 2024 | No | Reward model | Multi-sample REINFORCE baseline; competitor to GRPO |

### Family B — Direct preference optimization (no RL loop)

These reformulate the **KL-regularized RLHF objective** as a closed-form loss on preference pairs `(y_w, y_l)`. There is no on-policy sampling, no critic, and no reward model — just two forward passes per pair.

| Method | Year | Defining feature |
|---|---|---|
| [[dpo]] — Direct Preference Optimization | Rafailov et al. 2023 ([[dpo-rafailov-2023]], NeurIPS 2023 Outstanding runner-up) | Closes the loop algebraically; **two forward passes per pair**; default for open-weight community fine-tunes (Zephyr, Tülu, OpenChat, Llama-3 community) |
| **IPO** — Identity PO | Azar et al. 2023 | Square loss; fixes a Bradley-Terry theoretical issue |
| **KTO** — Kahneman-Tversky Optimization | Ethayarajh et al. 2024 | Needs only **binary good/bad** signals, not pairwise |
| **ORPO** — Odds-Ratio PO | Hong et al. 2024 | Combines SFT and preference loss in one odds-ratio formulation |
| **SimPO** — Simple PO | Meng et al. 2024 | Drops the reference-model term entirely |

### Higher-level pipelines

What end-to-end alignment runs look like:

| Pipeline | Stages | Method choice |
|---|---|---|
| [[rlhf]] — RL from Human Feedback | SFT → RM → policy opt | Classical: SFT → human-labeled [[preference-modeling]] → [[reward-modeling]] → [[ppo|PPO]] |
| [[rlaif]] — RL from AI Feedback | SFT → AI-judge → RM → policy opt | AI evaluator (often guided by a [[constitutional-ai|constitution]]) replaces humans in step 2 |
| [[constitutional-ai]] — Anthropic's CAI | SFT → critique → revise → RLAIF | Written principles bootstrap a self-critic that generates the preference data |
| [[agentic-rl]] — agentic RL | (SFT cold-start) → multi-step tool-use traces with outcome reward → GRPO | [[deepseek-r1]] (pure RL with rule-based rewards), [[search-r1]] (tool-use RL with retrieved-token masking), [[swe-gym]] (SWE-agent training) |
| **Iterated DPO** | SFT → DPO → re-sample → DPO again | The DPO equivalent of "online RLHF" — refresh preference data each iteration |

## How they differ

### PPO vs. GRPO (the modern RL pair)

Both are policy-gradient RL with KL regularization to a reference policy. **GRPO replaces PPO's learned critic with a per-prompt group baseline.** Three practical consequences:

- **Memory**: GRPO saves the value-network parameters and activations — material for 70B+ LLMs.
- **Stability**: GRPO's baseline is empirically derived from on-policy samples each step; PPO's critic is a slow-moving learned estimate. GRPO tends to be steadier.
- **Reward shape**: GRPO works best when you can sample several candidates per prompt and rank them — natural for reasoning (multiple completions, score each by correctness) and tool-use (multiple trajectories, score each by outcome). PPO is fine with per-token dense rewards.

After [[deepseek-r1]], GRPO has effectively replaced PPO as the default for reasoning/agent RL. PPO is still standard at the closed labs whose recipes pre-date this shift.

### RL family vs. DPO family

- **DPO is one supervised loss over preference pairs.** No on-policy sampling, no critic, no RM. Cost ≈ plain SFT.
- **PPO/GRPO are on-policy loops** — sample, evaluate, gradient — and they can chase a moving reward signal (verifiable correctness, tool-use outcomes). DPO can't do that directly: it needs offline preference pairs.

Practical rule of thumb:
- **Preference data only, no verifiable reward** → DPO (or a variant).
- **Verifiable reward / outcome signal / multi-turn rollouts** → GRPO (or PPO).
- **Both available** → typically SFT → DPO → GRPO sequentially.

### Outcome-based RL vs. preference-based RL

A 2024–2026 distinction: **rule-based / outcome-based rewards** (does the test pass? is the answer correct?) don't need a reward model at all. [[deepseek-r1]] showed that pure-RL with rule-based rewards (correctness + format) elicits emergent reasoning. [[search-r1]] does the same for tool-use with outcome rewards. This collapses the RLHF pipeline from "SFT → RM → PPO" to just "RL on outcomes," which is much cheaper and avoids reward-model hacking entirely — at the cost of only working when correctness is mechanically checkable.

## Where each method shows up in the wiki

- **[[ppo|PPO]]**: [[instructgpt]], [[rlhf]], [[memory-r1]] (compared with GRPO), [[constitutional-ai]] historical stack
- **[[grpo|GRPO]]**: [[deepseek-r1]], [[search-r1]], [[memory-r1]], [[mem-alpha]] (RL-trained memory ops), [[agentic-rl]]
- **[[dpo|DPO]]**: [[ui-tars-1]] (DPO in the iterative training stage), most open-weight community fine-tunes; variants in [[dpo-rafailov-2023]]
- **[[reward-modeling]]**: [[rlhf-christiano-2017]], [[instructgpt-ouyang-2022]], [[constitutional-ai-bai-2022]]
- **[[preference-modeling]] (Bradley-Terry)**: underlies everything above

## What ties them together

All five method families (PPO, GRPO, DPO, IPO/KTO/etc., outcome-RL) optimize the **same underlying objective** in different forms:

```
maximize E_{x, y ~ π_θ}[r(x, y)] − β · KL(π_θ ‖ π_ref)
```

— maximize expected reward, stay close to a reference model. The methods differ in *how reward is sourced* (human labels / AI judge / outcome / closed-form via preferences) and *how the optimization is structured* (on-policy RL loop / supervised classification / multi-stage pipeline). The 2023–2026 trajectory has been toward eliminating moving parts — drop the critic ([[grpo|GRPO]]), drop the reward model ([[dpo|DPO]]), drop the reward model AND the human (outcome-RL like [[deepseek-r1]]).

## Related pages

- [[reinforcement-learning]] — the umbrella concept
- [[agentic-rl]] — the 2025–2026 paradigm where agents are trained end-to-end with RL on multi-step traces
- [[preference-modeling]] / [[reward-modeling]] — the underlying machinery for human/AI preference learning
- [[reward-hacking]] — the failure mode every policy-optimization method has to manage
- [[downsides-of-looped-transformers]] — the adjacent thread on depth-recursion as an alternative to token-recursion for reasoning compute

## References

- [[rlhf-christiano-2017]] — Christiano et al. 2017, the foundational RL-from-preferences paper
- [[instructgpt-ouyang-2022]] — Ouyang et al. 2022, the canonical SFT → RM → PPO recipe
- [[constitutional-ai-bai-2022]] — Bai et al. 2022, the canonical RLAIF recipe
- [[dpo-rafailov-2023]] — Rafailov et al. 2023, the DPO paper
- [[deepseek-r1-2025]] — DeepSeek 2025, GRPO + rule-based rewards eliciting emergent reasoning
- [[search-r1-jin-2025]] — Jin et al. 2025, GRPO for multi-turn tool-use with retrieved-token masking
- [[memory-r1-yan-2025]] — Yan et al. 2025, PPO/GRPO compared for RL-trained memory operations
