---
title: Overview
type: overview
created: 2026-05-12
updated: 2026-10-06
tags: []
---

# Overview

A synthesis of the wiki's current contents. The corpus spans 2017–2026. It began with the foundational [[transformer-architecture|Transformer]]-era papers and has since grown mostly in one direction: how LLMs are turned into **agents** — what surrounds the model, how agents remember, how they are checked, and what it costs to run them.

## Summary

The wiki covers **580 source pages**, **189 entities**, **335 concepts**, **5 comparisons** and **15 filed queries** (as of 2026-10-06). Since the 16-source seed corpus of May 2026, most growth has come from weekly newsletter sweeps of arXiv and lab blogs, plus targeted ingests.

The corpus now has two layers:

- **The foundations (2017–2023).** The [[transformer|Transformer]]; [[bert]] and [[gpt-1]] with the [[pre-training|pre-train]] + [[fine-tuning|fine-tune]] recipe; the [[gpt-family|GPT scale-up]] and [[in-context-learning]]; [[scaling-laws]] and [[chinchilla-hoffmann-2022|Chinchilla]]; [[instructgpt-ouyang-2022|InstructGPT]] and [[constitutional-ai]]; the [[retrieval-augmented-generation|retrieval]] line ([[dpr]], [[rag]], [[retro]]); and the first agent and reasoning papers ([[chain-of-thought-prompting|CoT]], [[react|ReAct]], [[reflexion|Reflexion]], [[toolformer]]).
- **The 2026 agent era**, which is now most of the wiki: harnesses and [[claude-code|Claude Code]], [[agent-skills]], memory and [[self-improving-agent|self-improvement]], [[agent-evaluation|evaluation]], inference efficiency and local inference on [[apple-silicon|Apple Silicon]], [[large-reasoning-model|reasoning models]], typed decision models, and misuse.

For a structural map, see [[agent-three-layer-model]] (the wiki's canonical breakdown of an agent) and [[from-transformer-to-modern-agents]] (nine evolutionary tracks converging into 2026 agents).

## Key Themes

### 1. The Transformer as universal substrate

Almost every model in this wiki is built on the [[transformer-architecture|Transformer]]: [[bert|encoder-only]] for understanding, [[decoder-only-transformer|decoder-only]] for generation, [[vit|ViT]] for vision, [[multimodal-llm|multimodal extensions]] for everything else. The core invariants — [[self-attention]], [[multi-head-attention]], [[residual-connection|residuals]] and [[layer-normalization]] — change little. The serious alternatives the wiki tracks are [[state-space-model|state-space models]] and hybrids, [[looped-transformer|looped and recurrent-depth models]], and [[diffusion-language-model|diffusion language models]]. All three are still niche next to the decoder-only Transformer.

### 2. Scale, and then what to do with inference compute

[[scaling-laws|Scaling laws]] (Kaplan 2020, [[chinchilla-hoffmann-2022|Chinchilla]] 2022) made scale the dominant lever, and the [[gpt-family|GPT family]] is the corpus's most legible scaling experiment. Since 2024 the frontier has moved to *inference-time* compute: [[o1]]-style [[large-reasoning-model|reasoning models]], [[deepseek-r1]] and [[inference-time-compute]]. The 2026 sources treat effort as a dial. Higher effort mostly buys verification rather than better approaches ([[reasoning-effort]]).

### 3. From fine-tuning to prompting to agents to harnesses

- **Fine-tune everything** (2018–2020): one backbone, many task heads ([[bert]], [[gpt-1]]).
- **Prompt everything** (2020–2022): [[gpt-3]]'s [[in-context-learning|ICL]] and [[chain-of-thought-prompting|CoT]].
- **Run agents over everything** (2022–): [[react|ReAct]]'s loop with [[tool-use|tool use]], [[reflexion|Reflexion]]'s self-reflection.
- **Engineer the harness** (2025–): the model is one component, and what surrounds it decides much of the outcome. [[harness-vs-model-attribution]] records scaffold-only swings of 11–15 points on SWE-bench Verified with the model fixed, and a 6.4× harness effect on ARC-AGI-3 (confirmed against ARC Prize's own results). [[agentic-harness-engineering]], [[claude-code]] and [[agent-skills]] are the hub pages.

### 4. Retrieval, and memory that does not compound

The retrieval thread ([[dpr]], [[rag]], [[retro]]) established **non-parametric memory** as a separate lever. [[retro-borgeaud-2021|RETRO]] showed 7.5B parameters with retrieval matching 175B without it. The 2026 agent literature turned retrieval into *memory* ([[long-term-memory]], [[graph-memory]], [[memory-management]]) and then measured it. Its finding is sobering:
- Appending skills and memories does **not** compound by itself; it fragments, or grows without bound ([[catastrophic-remembering]]).
- Unvalidated memories spread mistakes between runs.
- Budget-matched tests found extraction-based graph memory losing to flat retrieval.

[[self-improving-agent]] was rewritten around these results.

### 5. Verification as the bottleneck

The recurring finding across planning, self-improvement, search and evaluation is that what decides success is **whether something checks the result against reality**:
- A world-model verifier took an 8B planner from 41.2% to 91.8%.
- Self-improvement loops work only behind an acceptance gate.
- Plain search beats elaborate harness evolution, but only when something selects the right sample.

[[verifier]] and [[verification-horizon-qwen-2026]] are the hubs. The flip side is that unanchored judges fail: [[llm-as-a-judge]] collects the evidence, such as satisfaction ratings that carry no information about task success, and judges picking the worse of two close agents 31% of the time.

### 6. Trusting the numbers

A large share of the wiki's work is separating measurement from claim:
- Vendor figures are labelled as such: [[claude-dev]], [[introducing-system-one-models-and-jev-typesafe-2026|TypeSafe's Jev launch]].
- [[benchmark-contamination]] and refreshed benchmarks are tracked.
- Contradictions are recorded and resolved against primary sources where possible, e.g. the ARC-AGI-3 baseline and DeepSeek's released configs.
- [[quantization-quality]] and [[quantization-safety]] keep genuinely open disputes visible rather than picking a side.

### 7. Inference efficiency, and running models locally

A substantial cluster covers making inference cheaper:
- [[sparse-prefill]], [[trained-sparse-attention]] and [[prefix-caching]].
- [[weight-streaming]] from SSD.
- [[edge-inference]].
- The economics of an agentic task, where cache-hit rate and effort dominate cost.

Much of it is grounded in local inference on [[apple-silicon|Apple Silicon]] through [[mlx]] and [[metal]]. It records the gaps between vendor speed claims and what a given chip actually does.

### 8. Typed decision models: old mechanism, new packaging

[[jev|Jev]] and the [[system-one-model|System One]] wave sell typed decisions with probabilities instead of text. The wiki traced the mechanism to 2020-era [[cloze-scoring]] (the MMLU readout), and the open reimplementations reached near-parity within days. Independent audits found the central claim, [[calibration]], dataset-dependent and overconfident. As a judge, Jev is well calibrated when a reference anchors the verdict and near chance without one.

### 9. Alignment, interpretability, and misuse

- [[rlhf|RLHF]] aligns behaviour without opening the black box.
- [[mechanistic-interpretability]] ([[monosemanticity]], [[emotion-concepts-anthropic-2026|emotion vectors]] in Claude Sonnet 4.5) aims to read the algorithms directly.
- [[institutional-alignment]] proposes governance at the scale of billions of agents.
- Chain-of-thought is not a complete window: [[not-all-llm-reasoning-visible-in-cot-baherwani-2026]].

On misuse, Anthropic's [[anthropic-threat-intelligence-report-september-2026|September 2026 threat report]] puts the risk in [[ai-enabled-cyber-operations|uplift across the whole kill chain]] and in [[illicit-distillation]]. Both run through **access**: proxies, resold credentials, stolen keys. [[prompt-injection]] remains without a robust defense, and typed outputs narrow it without removing it. The report is a vendor's self-account, and every page drawing on it says so.

### 10. The OpenAI → Anthropic diaspora

Many co-authors of [[gpt-2-radford-2019|GPT-2]] and [[gpt-3-brown-2020|GPT-3]] — [[dario-amodei]], [[jared-kaplan]], [[tom-brown]], [[sam-mccandlish]], [[chris-olah]] — went on to co-found [[anthropic]] in 2021. The two labs have set much of the frontier agenda since 2018.

## Open questions

- **Does harness evolution beat plain search at a matched compute budget?** Only one 2026 paper ran the control, with mixed results ([[agentic-harness-engineering]]).
- **Are typed decision models calibrated where it matters?** Calibration holds on some datasets and question types and fails on others ([[calibration]], [[jev]]).
- **When does a multi-agent system beat a single agent with the same compute?** [[multi-agent]] holds evidence both ways.
- **Is 4-bit quantization safe, and is code disproportionately damaged?** Both are still disputed ([[quantization-quality]], [[quantization-safety]]).
- **What does scalable [[institutional-alignment]] look like in practice?**
- **Does the always-appending agent loop have a stable fixed point?** [[catastrophic-remembering]] suggests not, without deliberate deletion.

## Gaps worth filling

Filled on 2026-10-06:
- the canonical agent framings ([[building-effective-agents-anthropic-2024]], [[llm-powered-autonomous-agents-weng-2023]]);
- classical agent theory ([[intelligent-agent]], [[actor-critic]]);
- the field map ([[artificial-intelligence]] ⊃ [[machine-learning]] ⊃ [[deep-learning]] ⊃ [[large-language-model]]);
- [[context-engineering]] and [[prompt-engineering]].

Still open:
- **Coinage of terms:** the first uses of "LLM", "prompt engineering", "context engineering" and "harness engineering" have no primary source in the wiki.
- **Context-window pages:** dedicated pages for the context window and the system prompt, each covered only inside other pages.
- **Housekeeping:** about 156 one-off broken links remain as placeholders for pages not yet written.
