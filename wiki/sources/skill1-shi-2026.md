---
title: "Skill1: Unified Evolution of Skill-Augmented Agents via Reinforcement Learning"
type: source
created: 2026-06-02
updated: 2026-06-02
sources:
  - skill1-shi-2026.md
arxiv_id: "2605.06130"
authors:
  - Yaorui Shi
  - Yuxin Chen
  - Zhengxi Lu
  - Yuchun Miao
  - Shugui Liu
  - Qi Gu
  - Xunliang Cai
  - Xiang Wang
  - An Zhang
first_author: Yaorui Shi
year: 2026
introduces:
  - "[[skill-acquisition]]"
tags:
  - 2026
status: complete
importance: medium
---

# Skill1: Unified Evolution of Skill-Augmented Agents via Reinforcement Learning

- **Source**: `sources/skill1-shi-2026.md`
- **arXiv**: [arXiv:2605.06130](https://arxiv.org/abs/2605.06130) (v3, 2026-05-12)
- **Authors**: Yaorui Shi, Yuxin Chen, Zhengxi Lu, Yuchun Miao, Shugui Liu, Qi Gu, Xunliang Cai, Xiang Wang, An Zhang (University of Science and Technology of China; Meituan; National University of Singapore; Zhejiang University; Wuhan University). Shi and Chen contributed equally; Gu and Zhang are corresponding authors.
- **Year**: 2026
- **Code**: https://github.com/AlphaLab-USTC/Skill1

## Summary

Skill1 trains a **single RL policy to co-evolve the entire skill lifecycle** — selection, utilization, and distillation — toward one shared task-outcome reward, rather than optimizing each stage in isolation or with separate reward sources. A [[skill-library]] lets an [[llm-agent]] reuse successful strategies across tasks, but maintaining it requires three coupled capabilities: the agent *selects* a relevant skill, *utilizes* it during execution, and *distills* new skills from experience. The paper's thesis is that prior methods break this coupling — they apply policy gradients to only a subset of the lifecycle (e.g., RetroAgent gives no gradient to selection; SkillRL freezes selection after cold-start SFT), and they draw rewards from heterogeneous sources (task outcome for one stage, self-assessed quality or heuristic matching for another), producing "partial and conflicting evolution." Skill1's workflow runs all three stages through the same policy πθ: generate a natural-language query to retrieve top-K candidate skills via a frozen encoder, re-rank the candidates to pick one, solve the task conditioned on that skill over multi-turn interaction, then reflect on the trajectory to distill a new reusable skill back into the library.

The technical core is **credit assignment that splits one terminal reward `r(τ) ∈ {0,1}` across three capabilities operating at different temporal scopes**. Utilization is credited directly by the outcome. Selection is credited by the *low-frequency trend* — a per-skill utility score `U(s)` maintained as an exponential moving average of outcomes, which smooths per-episode noise and rewards consistently effective skills (re-ranking is supervised by NDCG against the utility ordering). Distillation is credited by the *high-frequency variation* `r(τ) − Û`, the deviation of the current outcome from the library's best trend, rewarding skills that improve on what the library already covers and discouraging redundant ones. All terms are optimized jointly with [[grpo]] in a single gradient step. This is the **unified-RL angle on agent skills**: it positions Skill1 against the text-edit / prompt-optimization line of work — [[skillopt-yang-2026|SkillOpt]] and [[skillmoo-gong-2026|SkillMOO]] evolve a *skill document* via accept-on-validation text edits without ever touching model weights — whereas Skill1 trains the policy weights themselves end-to-end (see [[skill-optimization]] for the side-by-side). On ALFWorld, Skill1 reaches a **97.5% average success rate**, beating the prior best (RetroAgent) by 2.6 points and ranking first on five of six task types; on WebShop it is best across all methods. Ablations show that removing any single stage's credit signal degrades all three capabilities, evidencing their mutual dependence.

## Key Points

- **Unified RL objective.** A single policy πθ produces four generation segments per rollout — the selection query, the re-ranking permutation, the action sequence, and the distilled skill — and they are optimized together: `J = J_util + λ₁·J_rerank + λ₂·J_distill`. No external retriever-trainer, teacher model, or auxiliary reward network.
- **Three jointly-learned sub-capabilities.** *Selection* (query generation + candidate re-ranking), *utilization* (multi-turn skill-conditioned action), and *distillation* (reflecting a `(strategy, scenario-description)` skill out of the trajectory) are all differentiable through the same policy.
- **One task-outcome reward, decomposed by temporal frequency.** The terminal `r(τ)` directly rewards utilization; its low-frequency trend (per-skill EMA utility `U(s)`) credits selection; its high-frequency variation (`r(τ) − Û`) credits distillation. This is the paper's signature move — per-capability credit assignment with no auxiliary models or extra rollouts.
- **Library mechanics.** A skill `s` = natural-language `strat` (how to act) + `desc` (when it applies). New skills are admitted only when `r(τ) = 1`. Capacity is capped at 5,000; eviction retires the lowest `U(s)·log n(s)` (low-utility *and* infrequently selected), preserving well-tested high-utility skills.
- **Mixed optimizers per segment.** Utilization and the query prefix use GRPO group-relative advantages; re-ranking uses a REINFORCE-style objective (different rollouts retrieve different candidate sets, so intra-group comparison is meaningless); distilled-skill advantages are normalized separately from utilization.
- **Contrast with prompt-edit / text-space optimization.** Skill1 trains weights via RL toward task outcomes; [[skillopt-yang-2026|SkillOpt]] and [[skillmoo-gong-2026|SkillMOO]] optimize the skill *text* offline with held-out-validation-gated edits and never update weights. Skill1 also optimizes *which* skill is used and *how*, dimensions the text-edit optimizers leave fixed.
- **Results.** ALFWorld 97.5% avg (+2.6 over RetroAgent), first on 5/6 task types; WebShop best overall (≈82.9% success / 89.7 score). Base model Qwen2.5-7B-Instruct, frozen encoder all-MiniLM-L6-v2, GRPO with G=16, lr 1e-6, empty initial library.
- **Co-evolution evidence.** Training dynamics show selection precision, utilization success rate, and library quality rising together under the shared signal; ablating any credit signal (`w/o Select.`, `λ₁=0`, `λ₂=0`) degrades all three.

## Method Detail

**Workflow (POMDP framing).** State `S = (x, e, B)`: task instruction, environment state, persistent library. A trajectory `τ = (q, z, a₁,o₁,…,a_T,o_T, s_new)`. Selection: query `q ~ πθ(·|x)` retrieves `B_K = top-K sim(E(q), E(s.desc))` via frozen encoder `E`, then a permutation `σ ~ πθ(·|x, B_K)` re-ranks and picks `z`. Utilization: `a_t ~ πθ(·|x, z.strat, o_{≤t})` for up to `T` turns; `G` rollouts per task, each with its own selection/utilization/distillation. Distillation: `πθ` reflects on `τ` to emit `s_new.strat` and `s_new.desc`.

**Reward assignment.**
- Utilization: `R^util_i = r(τ_i)`.
- Selection: query gradients flow through the utilization objective (better query → better candidates → higher outcome); re-ranking uses `R^rerank_i = NDCG(σ_i, argsort(−U(B_K)))`, where utility updates as `U(s) ← (1−α)U(s) + α·r(τ_i)` for **all** retrieved candidates (co-retrieval treated as relevance evidence).
- Distillation: `R^distill_i = r(τ_i) − Û_i`, with `Û_i = max_{s∈B_K} U(s)` the library baseline.

**Optimization.** `J_util` via GRPO over rollouts; `J_rerank` REINFORCE-style (`(1/NG)·Σ R^rerank_i · log πθ(σ_i | x_i, B_K)`); `J_distill` via GRPO with separately-normalized advantages; combined as `J = J_util + λ₁ J_rerank + λ₂ J_distill`. `U(s)` updated non-parametrically.

## Entities Mentioned

- [[university-of-science-and-technology-of-china]] — lead affiliation (Shi, Wang, Zhang)
- [[meituan]] — industry collaborator (Chen, Lu, Miao, Gu, Cai)
- [[national-university-of-singapore]]
- [[qwen-2-5]] — Qwen2.5-7B-Instruct base policy for Skill1 and all baselines
- [[alfworld]] — text-based household benchmark; Skill1 reaches 97.5% avg
- [[webshop]] — online-shopping simulator benchmark
- [[react]], [[reflexion]] — training-free baselines
- [[mem0]] — heuristic-memory baseline (also tested w/ GRPO)

## Concepts Discussed

- [[skill-acquisition]] — the lifecycle this paper makes fully learnable
- [[skill-optimization]] — parent concept; frames Skill1 (RL/weights) vs. SkillOpt/SkillMOO (text-edit)
- [[agent-skills]] — reusable strategy + scenario-description artifacts
- [[skill-library]] — the persistent store Skill1 selects from and distills into
- [[reinforcement-learning]]
- [[grpo]] — base optimizer; Skill1 extends it with stage-specific credit decomposition
- [[agentic-rl]]
- [[llm-agent]]

### RL-trained-policy siblings (cross-links)

- [[memory-r1]] — the analogous move for the *memory* lifecycle: ADD/UPDATE/DELETE/NOOP learned as an RL policy from downstream reward. Skill1 is the *skill* lifecycle counterpart (select/utilize/distill).
- [[mem-alpha]] — RL-trained memory management; sibling in the "make the agent's own bookkeeping a learned policy" family.
- [[voyager]] — the foundational skill-library agent; accumulates reusable code skills but selects/distills via prompting (LLM-as-controller), not weight-trained RL. Skill1 closes that gap by training the controller.

## Notable Quotes

> "We propose Skill1, a framework that trains a single policy to co-evolve skill selection, utilization, and distillation toward a shared task-outcome objective."

> "All learning derives from a single task-outcome signal. Its low-frequency trend credits selection and its high-frequency variation credits distillation."

> "Existing methods optimize these capabilities in isolation or with separate reward sources, resulting in partial and conflicting evolution."

> "Skill0 (Lu et al., 2026) internalizes skills into model parameters with zero external skills; Skill1 co-evolves all three stages through one policy model and a unified task outcome signal."

## Significance

Skill1 is the RL pole of the agent-skill design space (cf. [[skill-optimization]]). Where the text-space optimizers ([[skillopt-yang-2026|SkillOpt]], [[skillmoo-gong-2026|SkillMOO]]) keep weights frozen and evolve a portable skill *document* via validation-gated edits — cheap, controllable, zero added inference cost — Skill1 argues that the *selection* and *utilization* decisions are themselves learnable and that leaving any stage unoptimized creates a bottleneck (a policy that uses skills well still loses if it keeps routing to the wrong ones). Its most reusable idea is the **frequency decomposition of a single reward** into a slow per-skill utility trend (for selection) and a fast per-episode variation (for distillation), which sidesteps the usual need for separate intrinsic rewards or auxiliary critics. It sits naturally alongside [[memory-r1]] and [[mem-alpha]] as part of a 2025–2026 thread that turns an agent's self-maintenance loops (memory, skills) from prompted heuristics into outcome-trained policies. Stated limitations: only two text-based environments (no visual/deep-search settings), and the fixed 5,000-entry library may become a scalability bottleneck as task diversity grows.

## References

_Original source: `sources/skill1-shi-2026.md`_
