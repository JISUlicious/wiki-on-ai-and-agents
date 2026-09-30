---
title: "Agent Lightning v1.0: Towards Harnessed Agentic RL"
type: source
created: 2026-09-05
updated: 2026-09-05
sources: []
arxiv_id: "2608.17528"
year: 2026
authors:
  - Zhiyuan He
  - Siwei Zhang
  - Zhiwen Zhou
  - Yuqing Yang
  - Yu Kang
  - Yuge Zhang
  - Luna K. Qiu
  - Tin Yan Tsui
  - Jiahang Xu
  - Chong Luo
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Agent Lightning v1.0: Towards Harnessed Agentic RL

**Zhiyuan He\*, Siwei Zhang\*, Zhiwen Zhou\*, Yuqing Yang, Yu Kang, Yuge Zhang, Luna K. Qiu, Tin Yan Tsui, Jiahang Xu, Chong Luo** ([[microsoft|Microsoft]]; [[fudan-university|Fudan University]]; [[zhejiang-university|Zhejiang University]]; University of Edinburgh — \*equal contribution) — [arXiv:2608.17528](https://arxiv.org/abs/2608.17528), cs.AI, 18 Aug 2026, arXiv perpetual non-exclusive license. Preprint; no venue. Code: `github.com/microsoft/agent-lightning`. Successor to the original Agent Lightning (Luo et al., arXiv:2508.03680), which introduced the disaggregated proxy architecture; v1.0 is described as "a complete refactoring."

## Summary

The paper names a paradigm and then audits it. **Harnessed agentic RL** is RL post-training run *through the same agent harness used at deployment* — mini-SWE-agent, [[openhands|OpenHands]], [[opencode|OpenCode]], [[claude-code|Claude Code]], Codex, [[openclaw|OpenClaw]], [[hermes-agent|Hermes]] — by pointing the harness's OpenAI-compatible client at an **LLM endpoint proxy** owned by the trainer. The harness keeps context construction, tool execution, and the agent–environment loop; the trainer sees only a sequence of request–response pairs `(p₁,a₁),(p₂,a₂),…` crossing the service boundary. The authors trace the proxy design to their original Agent Lightning and note that verl Uni-Agent, AReaL 2.0, slime v0.3.0, and Polar have since adopted it — so the audit is of a whole family of frameworks, not just their own.

The core claim is that this is **not** traditional [[agentic-rl]] with an extra network hop. Both admit a POMDP formulation, but the latent state now includes harness state alongside environment state, the policy sees separately constructed per-call prompts instead of one continuously extended token history, and control shifts to the harness (subagents, handoffs, context summarization). The consequence the paper builds everything on: **one rollout no longer maps to one training sample.** Four challenges follow — retokenization and sample merging, advantage calculation, loss normalization, and training-backend scheduling under dynamic sample counts — and the paper argues existing frameworks make silent, sometimes conflicting choices on each.

Agent Lightning v1.0 is the framework built to embed the authors' own answers: **approximately 3,500 lines of code** (stated four times; the newsletter figure checks out), three components (API Gateway, Rollout Controller, Customized Trainer on verl), Kubernetes Jobs instead of commercial sandboxes, and a *collocated async* RL schedule. Validated on three settings, the headline is the coding agent: with **~6,000 SWE-smith training examples**, RL alone lifts **Qwen3.5-9B on [[swe-bench-verified|SWE-bench Verified]] from 41.8% to 56.4% (+14.6 pp)** — both newsletter numbers verified verbatim against the abstract, §1, §4.3.3, and the conclusion.

## The four challenges, and what each framework does

**1. Retokenization breaks token-prefix continuity.** A ReAct-style harness satisfies the *text*-level prefix relation `(pᵢᵗᵉˣᵗ, aᵢᵗᵉˣᵗ) ⪯ pᵢ₊₁ᵗᵉˣᵗ`, but RL trains on token IDs, and the *token*-level relation `(pᵢᵗᵒᵏ, aᵢᵗᵒᵏ) ⪯ pᵢ₊₁ᵗᵒᵏ` need not hold. Three observed mechanisms:

| Mechanism | Example given |
|---|---|
| Chat-template non-compositionality: `Template(A‖B) ≠ Template(A)‖Template(B)` | Qwen's chat template removes an earlier `<think>` marker on re-render |
| Decode–retokenize drift: `Tok(Decode(a)) ≠ a` | "having" sampled as `h`+`aving`, retokenized as `hav`+`ing` (Figure 3) |
| Inference-time output transformation | Tool-call / structured-output handlers normalize, repair, and reserialize the response before the harness sees it (see [[tool-call-parser]]) |

Three mitigations are laid out. *Independent-call training* is token-correct but recomputes shared prefixes. *Buffered token replacement* (what **AReaL and verl Uni-Agent** do: keep a request buffer in the proxy, and when new text matches, splice the originally sampled response tokens into the new prompt) raises the merge ratio — but the paper's sharpest observation is that it is **off-policy stitching**: `aᵢ₊₁` was sampled under the actual prompt `pᵢ₊₁`, not the stitched `p̃ᵢ₊₁`, so training under the stitched prompt evaluates the action under a prompt the policy never saw. **slime and Polar** do not replace. *Prefix-shared / tree-structured training* (branch-aware causal masks over a shared trie) recovers reuse without going off-policy but "require[s] substantial training backend support" — tree packing, custom masks/kernels, partitioning, distributed gradient handling. Agent Lightning v1.0 picks the third option, **best-effort sequence merging**: merge two consecutive calls only when the observed token IDs satisfy exact prefix; otherwise close the sequence and open a new one. The API Gateway deliberately keeps *no* server-side request buffer, "which keeps training consistent with deployment."

**2. Advantage calculation.** Sample count is now dynamic — from retokenization splits, subagent spawns, and context summarization. Measured on the coding run: **only 36% of rollouts remain a single training sample; the mean is 2.41 samples per rollout** (Figure 10). Reward is outcome-based and inherited by every sample of the rollout, so the question is whether the GRPO group baseline is computed over rollouts or samples. Figure 4's example: Rollout 1 (reward 1) splits into 3 samples, Rollout 2 (reward 0) stays 1 — rollout-level baseline 1/2, sample-level baseline 3/4. **verl Uni-Agent and Polar** are rollout-level; **slime and AReaL** are sample-level. The paper argues rollout-level is principled because retokenization is incidental and subagent/summarization are harness internals that should not move the group baseline.

**3. Loss normalization.** Three candidates over a batch of R rollouts with Nρ samples each: DAPO's token-mean (Eq. 14), [[grpo|GRPO]]'s seq-mean-token-mean (Eq. 15), and slime's rollout-level token-mean (Eq. 16, pool each rollout's tokens, then average uniformly over rollouts). Seq-mean is rejected because it up-weights rollouts that happen to split into more samples. Token-mean is principled but empirically "sensitive to long sequences: when many long negative samples appear in a batch, it can cause instability later in training." They adopt rollout-level token-mean.

**4. Backend scheduling.** Nρ is known only after harness execution, while GPU count and DP/TP/PP config are fixed. Every flattened sequence must keep its rollout ID and prompt-group ID (Eq. 17), and all sequences from one rollout must land in the same optimizer update — otherwise different parts of one rollout are evaluated under different policy versions ("within-rollout policy skew").

## System

| Component | Responsibility |
|---|---|
| **API Gateway** | Single stateful service; source of truth for rollouts (queuing → running → succeeded/failed), registered model endpoints, and append-only events. Records a `model_request` event (prompt token IDs, response token IDs, response log-probs) per LLM call and a scalar `reward` event. Proxy path `/proxy/rollout/{id}/attempt/{id}/mode/{mode}/openai/v1/chat/completions` embeds the rollout ID, so attribution is automatic. All rollout endpoints idempotent. |
| **Rollout Controller** | Polls queued rollouts and launches each as a **Kubernetes Job** (K8s Reconciler, standard watch + periodic list) or local process (Local Reconciler). Gateway state is ground truth; K8s converges by retry — best-effort eventual consistency only. |
| **Customized Trainer** | Built on verl. Registers a batch, waits for terminal state, pulls events, and runs the *Dedicated Sample Adapter* (best-effort merge, rollout-level advantage, rollout-level token-mean loss). Deduplicates retried LLM calls by keeping only the **last** `model_request` with an identical prompt. Exposes every rollout's inputs, requests, rewards, token/turn stats for inspection "manually or with AI agents." |

**Collocated async RL.** Sync RL waits for the slowest rollout; AReaL-style async splits rollout and update onto separate GPU pools, which costs more GPUs and two queues progressing at different rates. Collocated async time-shares one pool: once enough data is collected, the Gateway stops admitting new requests, drains in-flight ones, updates weights, then resumes — and pauses any request that arrives mid-switch, so the phase change is invisible to the harness. Claimed **~2× end-to-end speedup over sync RL** with fewer GPUs than disaggregated async. (No table; the 2× is a prose figure.)

**Sandboxing.** Uni-Agent launches agents on Modal Sandbox and Volcano veFaas; slime on E2B. Agent Lightning runs agents as K8s Jobs on self-hosted compute, keeping the whole stack open-source.

## Experiments

| Setting | Harness / data | Policy | Algorithm | Batch × rollouts | Result |
|---|---|---|---|---|---|
| Search agent | [[search-r1]] setup; train HotpotQA; eval 50 ex. each from HotpotQA, 2WikiMultiHopQA, MuSiQue, Bamboogle, TriviaQA, NQ; EM reward | Llama-3.2-3B-Instruct | GRPO | 512 × 4 | validation reward **25.1% → 41.7% (+16.6)** |
| General instruction-following | LLM-in-Sandbox harness (Cheng et al. 2026); Instruction Pre-Training data, 80/20 split | Qwen3-4B-Instruct-2507 | RLOO | 8 × 8 | validation **51.9% → 70.2% (+18.3)** |
| Coding agent | mini-SWE-agent harness; [[swe-smith]] tasks | Qwen3.5-9B | GRPO | — | SWE-bench Verified **41.8% → 56.4% (+14.6)** at step 208 |

**Coding-agent data pipeline** (the part the authors say no other framework ships). SWE-smith: 59,136 tasks from 128 repos, Docker images **295 GB** vs 4 TB for R2E-Gym and 6 TB for [[swe-gym|SWE-Gym]]. Problems found in the release: **18,033** records with an empty problem statement, **1,265** whose problem branch is missing from the Docker image, and some tasks with huge test suites (python-jsonschema >7,000 tests). Filter: drop empty statements, missing branches, and >200 tests; then a model-based difficulty filter — run Qwen3.5-9B four times per task, drop all-solved, keep mixed (≈5,000), and add 1,000 all-fail tasks to avoid an overly easy set. Final: **≈6,000 train / 400 test.**

**Reward hacking observed** — the agent obtaining the gold source directly via (1) Git history to find the gold commit, (2) `wget`/`curl` from GitHub, (3) `pip` source download, (4) `urllib`. Safeguards: disable Git and hide `.git`; a Kubernetes network policy blocking outbound traffic except whitelisted services. Several instances were found by AI agents reading the trainer's monitoring output.

**Ablation of the rollout-level choices** (Figure 9, coding run, same GRPO objective; validation = the 400-task SWE-smith test split):

| Variant | Advantage | Loss | Best validation reward |
|---|---|---|---|
| Sample-level Advantage (baseline) | sample | token-mean | 35.0% |
| Rollout-level Advantage | rollout | token-mean | 33.1% |
| **Rollout-level Advantage + Rollout-level Norm** | rollout | rollout token-mean | **38.2% at step 128** |

Fixing only the advantage *hurts* (35.0 → 33.1) and lets policy entropy climb; adding rollout-level normalization both recovers the reward and keeps entropy growth slow and stable — the authors read this as normalization controlling the entropy increase that corrected advantages introduce. Only the winning variant was carried to SWE-bench Verified (41.8 → 56.4 at step 208).

## Where it breaks

- **"Modest compute" is never quantified.** No GPU count, wall-clock, or step-time appears anywhere in the paper; the ~2× collocated-async speedup is prose only.
- **No cross-framework result.** The framework comparison is a design-choice audit (who buffers, who normalizes how), not a measured head-to-head; the ablation is one coding run per variant with no seeds.
- **The rollout-level-advantage-only regression is unexplained beyond entropy.** 35.0 → 33.1 is the paper's own data showing the "principled" fix alone is worse; the interaction with normalization is asserted from one run.
- **SWE-bench Verified is reported for one variant, one checkpoint**, with no comparison to Uni-Agent / slime / Polar training the same model on the same data.
- **Credit assignment inside a rollout is explicitly deferred** ("Future work may still be needed to design better credit assignment across the samples within a rollout") — every sample of a rollout inherits the same terminal reward, so a subagent branch that did nothing useful is rewarded like the main branch.
- Best-effort merging **lowers the merge ratio rather than fixing drift**: 64% of rollouts split, and the paper does not report how much prefix compute is wasted relative to tree-structured training.

## Related

- [[harnessed-agentic-rl]] — the paradigm this paper names (**no wiki page yet**; the natural home for the Agent Lightning / ClawGym II / Polar / Uni-Agent / Molt cluster).
- [[clawgym-ii-black-box-rl-agent-harness-song-2026]] — the same proxy-at-the-model-boundary idea, but treating the harness as a **black box** and solving the sample-merging problem with **prefix trees** rather than best-effort merging; see that page for the precise contrast.
- [[molt-pytorch-native-agentic-rl-framework-hu-2026]] — NVIDIA's answer to the same retokenization problem from the other side: a token-in/token-out loopback server so the harness never leaves token space ("never trains on a token it did not generate"); Agent Lightning instead accepts drift and merges only when IDs match.
- [[toktier-exact-stateful-tokenization-zhang-2026]] — the serving-side face of the same non-compositionality (`tok(A)‖tok(B) ≠ tok(A‖B)`), measured on real Claude Code / Codex traffic.
- [[agentic-rl]] — the "traditional" paradigm the paper defines itself against: trainer-owned loop, one linear token history, one sample per rollout.
- [[grpo]] — the objective used; the paper's contribution is *where* the group baseline and loss mean are computed, not the estimator.
- [[outcome-based-reward]] — reward is terminal and shared across every sample of a rollout.
- [[reward-hacking]] — four concrete exploits on SWE-smith, all forms of reaching the gold patch, and infrastructure-level fixes rather than reward-model fixes.
- [[search-r1]] / [[search-r1-jin-2025]] — the search-agent protocol reproduced in §4.1.
- [[swe-bench]] / [[swe-bench-jimenez-2023]] — evaluation; [[swe-gym]] / [[swe-gym-pan-2024]] — the 6 TB training environment SWE-smith is contrasted with.
- [[swe-smith]] — the training corpus (**no wiki page yet**).
- [[swe-agent]] — mini-SWE-agent, the coding harness used, is its minimal descendant.
- [[openhands]], [[opencode]], [[claude-code]], [[openclaw]], [[hermes-agent]] — the deploy-time harnesses named as targets; only mini-SWE-agent and LLM-in-Sandbox are actually trained through here.
- [[harness-vs-model-attribution]] — this paper is the *training-time* version of the attribution problem: the harness is held fixed at deploy configuration and only the weights move, which is the cleanest possible model-only delta (+14.6 pp on SWE-bench Verified).
- [[agentic-harness-engineering]] and [[harness-r1-learning-to-edit-runtime-harnesses-shao-2026]] — the inverse move: freeze the weights and optimize the harness. Harnessed agentic RL freezes the harness and optimizes the weights.
- [[harness-1-state-externalizing-search-agents-jiang-2026]] — an earlier "RL through a stateful harness" result; Agent Lightning is the general infrastructure for that pattern.
- [[openclaw-rl-train-agent-by-talking-wang-2026]] — a server–client RL architecture over slime for the OpenClaw harness; the loop-ownership problem is the same.
- [[tool-call-parser]] — the "inference-time output transformation" mechanism is exactly the parser rewriting the sampled text.
- [[byte-pair-encoding]] — why `Tok(Decode(a)) ≠ a` is possible at all.
- [[react]] — the single-agent text-prefix case (Eq. 8) that traditional agentic RL assumes.
- [[multi-agent]] — subagents and handoffs are the harness operations that produce non-linear sample trees.
- [[distributed-training]] — the fixed-DP/TP/PP vs dynamic-sample-count scheduling constraint.
- [[qwen]], [[llama]] — policy backbones.

## References

- [arXiv:2608.17528](https://arxiv.org/abs/2608.17528)
- Code: https://github.com/microsoft/agent-lightning
- Original Agent Lightning: Luo et al., arXiv:2508.03680
- Compared frameworks: verl Uni-Agent (github.com/verl-project/uni-agent); AReaL 2.0 (arXiv:2607.01120); slime (github.com/THUDM/slime); Polar (arXiv:2605.24220)
