---
title: "Continual Learning Mechanisms Compose for Long-Horizon Memorization"
type: source
created: 2026-09-15
updated: 2026-09-15
sources:
  - continual-learning-mechanisms-compose-long-horizon-memorization-zhang-2026.md
arxiv_id: "2609.06986"
authors:
  - Zheyuan Zhang
  - Alvin Zhang
  - Daniel Khashabi
  - Tianmin Shu
year: 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# Continual Learning Mechanisms Compose for Long-Horizon Memorization

**Zheyuan Zhang, Alvin Zhang (equal contribution), Daniel Khashabi, Tianmin Shu (equal advising)**, Johns Hopkins University. [arXiv:2609.06986](https://arxiv.org/abs/2609.06986), cs.LG, v1 7 Sep 2026. No venue stated. Project page: `compose-cl.github.io`.

## Summary

The paper asks whether a model's *weights* can serve as a memory that survives many later updates. It calls this "long-horizon memorization". A [[qwen|Qwen3-4B-Base]] model with a rank-32 [[lora]] on every attention and MLP projection is fine-tuned on **100 query–answer tasks in sequence**. Raw examples from earlier tasks are not kept, and no task ID is given at inference (the domain-incremental setting). Every classical [[continual-learning]] mechanism fails alone at this horizon. The hypothesis is that mechanisms targeting different sources of forgetting compose. Three **anchors** say *what* each update must preserve:

- a **data anchor**: unconditional generative replay, where the frozen previous model writes 300 pseudo-sequences from a task-agnostic seed token;
- a **function anchor**: self-distillation on current-task inputs, in the style of Learning without Forgetting;
- a **weight anchor**: Synaptic Intelligence, or online EWC during the search.

A **low-rank allocation rule** says *where* updates are kept. **Shared LoRA** trains one adapter throughout. **Merged LoRA** folds each task's adapter into the dense weights and starts a fresh adapter and optimizer, as ReLoRA does. A 90-configuration task-level successive-halving search, followed by a 2⁴ factorial, finds that **all three anchors plus merged LoRA** is the only composition in the top 3 on every dataset.

## Key results

Three datasets, each 100 tasks: **Symbol-QA** (10,000 random key–value pairs), **LLM-QA** (10,000 generated fictional facts), **Real-QA** (5,000 public-QA items the base model gets wrong in 5/5 samples). Final retention is accuracy on all 100 tasks after the last update, averaged over 3 seeds.

| Method | Symbol-QA | LLM-QA | Real-QA | Average |
|---|---|---|---|---|
| Naive sequential SFT (shared LoRA, no anchors) | 1.0 | 1.4 | 1.3 | **1.2** |
| Best standalone mechanism | 4.2 | 7.5 | 12.5 | **8.1** |
| SI + self-distillation + replay + merged LoRA | 18.5 | 41.8 | 44.3 | **34.9** (28×) |
| Best per dataset | 23.2 (SD+R+M) | 41.8 (all four) | 54.8 (SI+R+M) | n/a |

- **Replay and merged LoRA carry the gain.** Their main effects are +9.5 to +19.3 and +5.9 to +20.5 points. Their interaction is positive and significant on all three datasets. Alone, their gains sum to 3.9 / 7.7 / 13.9 points; together they add 15.6 / 31.0 / 46.9 points over naive SFT (**super-additive**).
- SI and SD are conditional. SD's gain shrinks when replay is present (SD×R −3.5 on LLM-QA, −10.3 on Real-QA). SI conflicts with merged LoRA on Symbol-QA (−3.0), because its importance weights point at adapter coordinates that were just reinitialized.
- **Memory half-life** (tasks until retention halves): naive 1 / 1 / 2; best single mechanism 4 / 6 / 11; best composition 19 / 32 / 44. **All curves keep falling.** Composition changes the *timescale* of forgetting and does not stop it.
- Search: no standalone mechanism survives to the 50-task stage, and every survivor at 100 tasks pairs a data anchor with merged LoRA. Adapters that grow with the task count (O-LoRA, sequential OSRM) do not consistently raise retention.

**Verification of the newsletter claim.** "1.2% → 34.9% over 100 tasks, three anchors + merged LoRA" checks out. The three anchors are generative replay, self-distillation and SI. One qualification: "naive sequential fine-tuning" here is a **shared LoRA adapter** with no anchors, not full-parameter fine-tuning.

**What it does not show.** Recall is tested only on the queries the model was trained on, so the paper says nothing about paraphrased queries. The authors say so themselves. More seriously, memorization **destroys general capability**. Qwen3-4B-Base averages 69.4% on GSM8K / MATH / MGSM / MMLU-Redux. After 100 tasks with merged LoRA it scores 13.0% (LLM-QA stream) and 13.4% (Real-QA stream), and under 8% after Symbol-QA, where MMLU-Redux sits near chance. O-LoRA keeps more (26.8% / 28.8%), which is still more than 40 points below the base model. So the 34.9% is bought with most of the model's other abilities. The results come from one 4B backbone, one final task order, and three seeds.

## Related

- [[continual-learning]]: the forgetting side. The winning recipe draws one mechanism from each classical family (replay, regularization, capacity allocation), which supports that page's framing. The half-life result also supports its claim that gradient methods overwrite: they overwrite more slowly when composed, but they still overwrite.
- [[catastrophic-remembering]]: the dual failure. Appending memory hoards; this is the **gradient-based side of that dual**, where a fixed-size state has to forget. The task-growing adapters (O-LoRA, OSRM) did not buy retention either.
- [[lora]] · [[understanding-lora-as-knowledge-memory-back-2026]]: LoRA used as a parametric memory store. The Back et al. warning that LoRA memory "can catastrophically fail" appears here at the 100-task scale.
- [[continualskillbench-guan-2026]]: the in-context counterpart, also 100 sequential subtasks per domain.
- [[knowledge-distillation]] (the function anchor is self-distillation) · [[fine-tuning]] · [[memory-consolidation]] · [[ebbinghaus-forgetting]] (R(a) is a forgetting curve over memory age).
- [[in-context-learning]] · [[retrieval-augmented-generation]]: the non-parametric alternatives the paper sets aside, since their information "must be supplied again".
- [[mmlu]] (MMLU-Redux is the held-out knowledge check).

## References

- [arXiv:2609.06986](https://arxiv.org/abs/2609.06986) (v1, 7 Sep 2026).
- Anchors: generative replay (Shin et al. 2017; LAMOL, Sun et al. 2019); Learning without Forgetting (Li & Hoiem 2017); EWC (Kirkpatrick et al. 2017), online EWC (Schwarz et al. 2018), SI (Zenke et al. 2017). Composition precedent: Dark Experience Replay (Buzzega et al. 2020).
- Allocation: LoRA (Hu et al. 2022); ReLoRA (Lialin et al. 2024); O-LoRA (Wang et al. 2023a); OSRM (Zhang & Zhou 2025).
- Setting: three continual-learning scenarios (Van de Ven & Tolias 2019); successive halving (Jamieson & Talwalkar 2016); AgentOdyssey (Zhang et al. 2026).
