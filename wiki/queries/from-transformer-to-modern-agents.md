---
title: "From Transformer to Modern AI Agents: Evolutionary Paths"
type: query
created: 2026-05-15
updated: 2026-05-15
sources:
  - attention-is-all-you-need-vaswani-2017.md
  - gpt-3-brown-2020.md
  - react-yao-2022.md
  - reflexion-shinn-2023.md
  - claude-code.md
  - opencode.md
  - hermes-agent.md
  - turner-activation-engineering-2023.md
  - emotion-concepts-anthropic-2026.md
tags:
  - 2026
status: complete
importance: high
---

# From Transformer to Modern AI Agents: Evolutionary Paths

The road from [[transformer|the Transformer (2017)]] to a working coding agent like [[claude-code|Claude Code]] is not a single line of descent. It is **nine distinct engineering tracks** that each solved a different bottleneck and stacked on top of one another. Modern agents are the accumulated product of all of them.

## The nine tracks

| # | Track | Question it answers | Canonical works |
|---|---|---|---|
| 1 | **Architecture** | What sequence model scales? | [[attention-is-all-you-need-vaswani-2017|Transformer]], [[bert]], [[gpt-1]], [[transformer-decoder]], [[t5]] |
| 2 | **Scaling** | How do we know it'll work at scale? | [[scaling-laws-kaplan-2020|Kaplan]], [[chinchilla-hoffmann-2022|Chinchilla]] |
| 3 | **Pre-training paradigms** | One model, many tasks? | [[gpt-2-radford-2019|GPT-2]], [[t5-raffel-2019|T5 text-to-text]], [[flan-wei-2021|FLAN]] |
| 4 | **Alignment / RLHF** | Make it useful as an assistant. | [[rlhf-christiano-2017|Christiano]], [[instructgpt-ouyang-2022|InstructGPT]], [[constitutional-ai-bai-2022|CAI]], [[dpo-rafailov-2023|DPO]] |
| 5 | **Reasoning** | Make it think before answering. | [[chain-of-thought-wei-2022|CoT]], [[zero-shot-cot-kojima-2022|Zero-shot CoT]], [[self-consistency-wang-2022|Self-Consistency]], [[tree-of-thoughts-yao-2023|ToT]] |
| 6 | **Retrieval** | External knowledge without retraining. | [[dpr-karpukhin-2020|DPR]], [[rag-lewis-2020|RAG]], [[retro-borgeaud-2021|RETRO]] |
| 7 | **Tool use / Agent loop** | Acting, not just generating. | [[webgpt-nakano-2021|WebGPT]], [[react-yao-2022|ReAct]], [[toolformer-schick-2023|Toolformer]], [[reflexion-shinn-2023|Reflexion]] |
| 8 | **Multi-agent orchestration** | Coordinating multiple agents. | [[metagpt-hong-2023|MetaGPT]], [[autogen-wu-2023|AutoGen]] |
| 9 | **[[steering]] + [[guardrailing]]** | Shape internals + restrict externals. | [[activation-engineering-turner-2023\|ActAdd]], [[representation-engineering-zou-2023\|RepE]], [[sparse-autoencoders-cunningham-2023\|SAEs]], [[red-teaming-ganguli-2022\|red-teaming]], [[indirect-prompt-injection-greshake-2023\|injection]] |

## Path-by-path summary

### Track 1 — Architecture: the substrate
The [[transformer|Transformer]] eliminated recurrence and convolution, replacing them with parallelizable [[self-attention]]. Three variants split out: encoder-only ([[bert]]), decoder-only ([[transformer-decoder]] → [[gpt-1]]), encoder-decoder ([[t5]]). Decoder-only became dominant for generation and is the substrate of every modern agent.

### Track 2 — Scaling
[[scaling-laws|Power-law scaling]] of loss with parameters, data, and compute (Kaplan 2020) made capability planning a budget problem. [[gpt-3|GPT-3 (175B)]] confirmed [[in-context-learning]] emerged at scale. [[chinchilla-hoffmann-2022|Chinchilla (2022)]] corrected the recipe — scale params and tokens equally — reshaping every post-2022 training run.

### Track 3 — Pre-training paradigms
[[pre-training]] + [[fine-tuning]] (GPT-1, BERT) → unified text-to-text framing ([[t5]]) → instruction tuning ([[flan-wei-2021|FLAN]]) → SFT-on-instructions ([[instructgpt-ouyang-2022|InstructGPT]]). The shift: one base model, many tasks via natural-language interface.

### Track 4 — Alignment / RLHF
[[rlhf-christiano-2017|Christiano (2017)]] established the preference-learning recipe (preferences → reward model → PPO). [[instructgpt-ouyang-2022|InstructGPT (2022)]] applied it to LLMs; ChatGPT shipped the result. [[constitutional-ai-bai-2022|Constitutional AI]] generalized to [[rlaif|AI-feedback]] preferences. [[dpo-rafailov-2023|DPO]] collapsed the pipeline into a single supervised loss.

### Track 5 — Reasoning
[[chain-of-thought-wei-2022|Wei (2022)]] showed CoT exemplars dramatically improve multi-step reasoning at scale. [[zero-shot-cot-kojima-2022|Zero-shot CoT]] needs just `"Let's think step by step"`. [[self-consistency-wang-2022|Self-Consistency]] introduced inference-time-scaling. [[tree-of-thoughts-yao-2023|ToT]] generalized to explicit search trees. Modern reasoning models (o1, etc.) internalize the lineage via RL.

### Track 6 — Retrieval
[[dpr|DPR]] proved [[dense-retrieval]] beats BM25 by 9–19%. [[rag-lewis-2020|RAG (Lewis 2020)]] named the paradigm. [[retro-borgeaud-2021|RETRO]] pushed retrieval into pre-training itself; 7.5B + retrieval matched 175B parametric-only.

### Track 7 — Tool use / agent loop
[[webgpt-nakano-2021|WebGPT]] fine-tuned GPT-3 to operate a browser with citations. [[react-yao-2022|ReAct]] crystallized the `Thought → Act → Obs` loop — the spine of every modern agent. [[toolformer-schick-2023|Toolformer]] self-supervised tool-call training data. [[reflexion-shinn-2023|Reflexion]] added cross-episode self-improvement via verbal RL.

### Track 8 — Multi-agent orchestration
[[metagpt-hong-2023|MetaGPT]] encoded SOPs for software-engineering teams. [[autogen-wu-2023|AutoGen]] generalized to programmable multi-agent conversations. Production agents adopt the patterns at varying levels of opinionation.

### Track 9 — [[steering]] + [[guardrailing]]: the two safety surfaces

This track is the *internal* + *external* shaping of model behavior:

- **External ([[guardrailing]])**: [[rlhf]] post-training, [[constitutional-ai|CAI]], [[red-teaming-ganguli-2022|red-teaming]], [[prompt-injection]] defenses, sandboxing.
- **Internal ([[steering]])**: intervene in activations at inference time. [[activation-engineering-turner-2023|ActAdd]] (Turner 2023) → [[contrastive-activation-addition-rimsky-2023|CAA]] (Rimsky) → [[representation-engineering-zou-2023|RepE]] (Zou) → [[sparse-autoencoders-cunningham-2023|SAEs]] (Cunningham) → [[inference-time-intervention-li-2023|ITI]] (Li) — read concepts as linear directions, write to them as steering vectors. The [[emotion-concepts-anthropic-2026|2026 emotion-concepts paper]] applies the toolkit to Claude Sonnet 4.5 with causal effects on real misalignment behaviors (blackmail, reward-hacking, sycophancy).

The two surfaces are complementary: guardrails at the boundaries, steering at the substrate. Anthropic's [[claude|Claude]] uses both — [[constitutional-ai|CAI]] training (guardrail) + [[functional-emotions|emotion-vector]] interventions (steering).

## Convergence: how the tracks compose in 2026 agents

Every 2026 agent (Claude Code, opencode, pi-mono, Hermes Agent, OpenClaw) picks up:

1. A [[transformer-architecture|Transformer]]-substrate model (Track 1) trained at compute-optimal scale (Track 2) under a [[pre-training|pre-training]] objective (Track 3).
2. [[rlhf|RLHF]] / [[constitutional-ai|CAI]] post-training for usefulness + safety (Track 4 + Track 9-external).
3. Internal [[chain-of-thought-prompting|CoT]] / [[tree-of-thoughts|ToT]] reasoning (Track 5).
4. Project / session-level [[retrieval-augmented-generation|retrieval]] (Track 6) — wiki memory, codebase RAG.
5. [[react|ReAct]]-style tool-use loops (Track 7).
6. [[react|Sub-agent]] dispatch for parallel work (Track 8).
7. [[steering]] or steering-adjacent runtime safety (Track 9-internal) — emotion-vector suppression, refusal-direction tuning.
8. Layered permission/sandbox stacks at the boundary (Track 9-external).

The five documented 2026 agents differ in *which* of these tracks they embed in core vs. push to plugins. See the [[cross-agent-comparison-2026|cross-agent comparison]] for the full matrix.

## What's missing in the wiki (suggested next ingests)

- **LLaMA** (Touvron 2023) — Chinchilla-recipe open-weight model that enabled the open-source assistant ecosystem.
- **Anthropic's Scaling Monosemanticity** (Templeton 2024) — SAEs on Claude 3 Sonnet; bridges [[sparse-autoencoders-cunningham-2023|Cunningham 2023]] to [[emotion-concepts-anthropic-2026|emotion-concepts 2026]].
- **OpenAI o1 / DeepSeek-R1** — reasoning models internalizing the CoT/ToT/SC line via RL.
- **MCP (Model Context Protocol)** — referenced by every 2026 agent doc; no dedicated source.
- **Computer use / Operator / Devin** — autonomous agent deployments at scale.

## References

This synthesis draws on ~30 source pages across the wiki. Key starting points: [[transformer]], [[gpt-3]], [[react]], [[reflexion]], [[steering]], [[guardrailing]], [[claude-code]], [[cross-agent-comparison-2026]].
