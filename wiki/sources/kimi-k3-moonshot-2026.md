---
title: "Kimi K3: Open Frontier Intelligence"
type: source
created: 2026-08-04
updated: 2026-08-04
sources:
  - kimi-k3-moonshot-2026.md
arxiv_id: "2607.24653"
year: 2026
authors:
  - Kimi Team
  - Tongtong Bai
  - Yifan Bai
  - Yiping Bao
tags:
  - 2026
  - paper
status: complete
importance: high
---

# Kimi K3: Open Frontier Intelligence

**Kimi Team** (Moonshot AI) — [arXiv:2607.24653](https://arxiv.org/abs/2607.24653). Submitted 27 Jul 2026; arXiv comments read only "K3 tech report" (no venue). Contributors are listed alphabetically by last name with no designated lead author; weights are released at `huggingface.co/moonshotai/Kimi-K3`.

## Summary

Kimi K3 is a **2.78T-parameter** native-multimodal [[mixture-of-experts]] model with **104.2B activated parameters** and a **1M-token context window**, released open-weight by [[moonshot-ai]]. The paper's framing is that the open ecosystem has been scaling test-time compute (reasoning, agentic RL) but has stalled on the pre-training axis around the 1T-parameter class; K3 pushes both axes at once, taking the pre-trained foundation to "3T-class" scale while doing RL at 1M context.

The architecture scales information flow along three axes. **Sequence**: a hybrid attention stack of 3 Kimi Delta Attention (KDA) layers per 1 Gated MLA layer (69 KDA + 24 MLA layers total, plus a trailing global-attention layer), inheriting KDA from [[kimi-linear-kimi-team-2025]] — the delta rule with a channel-wise forget gate, chunkwise-parallel, and a fine-grained low-rank decay logit. **Depth**: Attention Residuals ([[attention-residuals-kimi-team-2026]]) let each module attend over the embedding and preceding block outputs via learned pseudo-queries, replacing purely sequential residual accumulation; K3 uses the *Block* AttnRes variant with 8 blocks of 12 layers (9 including the embedding), cutting overhead from O(Ld) to O(Nd). **Width**: *Stable LatentMoE*, which routes in a compact latent space (ℓ = 3584, 0.5× the 7168 hidden dim) so that expanding routing multiplicity does not blow up communication and expert-weight traffic — 896 routed experts with 16 active per token, plus 2 shared experts.

Post-training is RL across coding, general-agent, reasoning and knowledge domains at multiple reasoning-effort levels (low/high/max), consolidated back into one model via multi-teacher on-policy distillation. Results: K3 is consistently ahead of the other open and proprietary models in the paper's suite (Claude Opus 4.8, GPT-5.5, GLM-5.2) while trailing Claude Fable 5 and GPT-5.6 Sol overall. The paper is candid that research-level reasoning (CritPt 23.4, HLE-Full 43.5/56.0) is the weakest area.

## Key points

**Architecture (Table 1, Kimi K2 → Kimi K3)**

| | Kimi K2 | Kimi K3 |
|---|---|---|
| Layers | 61 | 93 |
| Total parameters | 1.04T | **2.78T** (↑167%) |
| Activated parameters | 32.6B | **104.2B** (↑220%) |
| Hidden dimension | 7,168 | 7,168 (=) |
| Latent MoE dimension | – | 3,584 (0.5×) |
| MoE hidden dim per expert | 2,048 | 3,072 (↑50%) |
| Routed experts | 384 | **896** (↑133%) |
| Experts active per token | 8 | **16** (↑100%) |
| Shared experts | 1 | 2 |
| Attention heads | 64 | 96 |
| Vocabulary | 160K | 160K |
| Training context length | 128K | **1M** (8×) |
| Attention | MLA | Hybrid KDA–MLA (69 KDA + 24 MLA) |
| Activation | SwiGLU | SiTU-GLU |
| ViT | – | MoonViT-V2, 401M params, 27 layers, patch 14, 12 heads |

- **Sparsity**: the paper states 896 routed / 16 active = **sparsity 56** (vs. 384/8 = 48 in Kimi K2). Two shared experts are always active on top of the 16 routed ones. *Derived, not stated in the paper:* the sparsity ladder across this Moonshot line runs [[kimi-linear-kimi-team-2025]] 32 (8 of 256, one shared) → K2 48 → K3 56, and the activated **fraction** falls from Kimi Linear's 3B/48B ≈ 6.3% to K3's 104.2B/2.78T ≈ **3.7%**. K3 activates ~35× more parameters per token than Kimi Linear's 3B while being ~1.7× sparser in proportion.
- **Stability at extreme sparsity**: the latent routed path chains ~four consecutive matmuls (W↓ → gated expert FFN → W↑), which at 2.8T scale produced exploding activations. Fixes: RMSNorm before the up-projection, **SiTU-GLU** (a bounded GLU: gate `β₁tanh(x/β₁)·σ(x)`, up `β₂tanh(x/β₂)`, with β₁=4, β₂=25 giving |f(x)| ≤ 100), and **Quantile Balancing** — expert biases set from router-score quantiles rather than auxiliary-loss-free bias updates, which break down near 10³ experts.
- **2.5× scaling efficiency**: measured as **fitted scaling-law curves of validation loss vs. training FLOPs on held-out OOD validation data** (Fig. 7). K3's curve reaches a given loss at ~2.5× fewer FLOPs than K2's. It is attributed to the *combination* of architecture + data + training-recipe changes, not to any single component, and the scaling study retuned batch size, LR, tokens-per-parameter and model shape.
- **Positional encoding**: NoPE — no explicit positional embedding; position is carried implicitly by KDA's recurrent gating/decay, so the model extrapolates to 1M tokens with no RoPE rescaling or interpolation.
- **Context curriculum**: 8K → 64K during pre-training, then 256K → 1M during cooldown (four stages), concentrating expensive long-sequence compute in a small slice of the budget.
- **Native multimodality**: vision and language jointly optimized from the start of training (interleaved visual/text tokens under one next-token objective), not a post-hoc encoder graft. MoonViT-V2 + lightweight projector.
- **Optimizer**: Per-Head Muon, with Kimi K2's weight clipping; cosine LR (scaling study favored cosine over WSD once each schedule got its own hyperparameter search), 1% linear warmup, weight decay 0.1.

**Headline benchmarks (Table 2; K3 at reasoning effort max, temperature 1.0)**

| Benchmark | Kimi K3 | Claude Fable 5 | GPT-5.6 Sol | Opus 4.8 | GPT-5.5 | GLM-5.2 |
|---|---|---|---|---|---|---|
| GPQA Diamond | 93.5 | 92.6 | **94.1** | 91.0 | 93.5 | 91.2 |
| CritPt | 23.4 | 28.6 | **32.3** | 20.9 | 27.1 | 20.9 |
| HLE-Full (no tools / tools) | 43.5 / 56.0 | **53.3 / 63.0** | 44.5 / 58.0 | 49.8 / 57.9 | 41.4 / 52.2 | – |
| DeepSWE | 67.5 | 70.0 | **73.0** | 59.0 | 67.0 | 46.2 |
| ProgramBench | **77.8** | 76.8 | 77.6 | 71.9 | 70.8 | 63.7 |
| Terminal-Bench 2.1 | 88.3 | 88.0 | **88.8** | 84.6 | 83.4 | 82.7 |
| FrontierSWE | 81.2 | **86.6** | 71.3 | 66.7 | 64.9 | 67.3 |
| SWE-Marathon (GPU kernels) | **42.0** | 35.0 | 39.0 | 40.0 | 14.0 | 13.0 |
| BrowseComp | **91.2** | 88.0 | 90.4 | 84.3 | 84.4 | – |
| DeepSearchQA (F1) | **95.0** | 94.2 | – | 93.1 | – | – |
| MCPMark-Verified | **94.5** | 87.4 | 92.9 | 76.4 | 92.9 | – |
| OSWorld-Verified | 84.8 | **85.0** | 83.0 | 83.4 | 79.0 | – |
| τ³-Banking | **33.4** | 26.8 | 33.0 | 27.6 | 31.3 | – |
| Harvey Lab-AA | **94.6** | 93.6 | 87.2 | 91.1 | 86.3 | – |
| GDPval-AA v2 (Elo) | 1686 | **1747** | 1736 | 1593 | 1491 | – |
| OmniDocBench | **91.1** | 89.8 | 85.8 | 87.9 | 89.4 | – |
| WorldVQA ForceAnswer | 51.0 | **56.7** | 41.8 | 39.1 | 38.5 | – |
| Math-Vision (– / +Python) | 94.3 / **97.8** | 94.8 / **98.6** | 95.8 / 97.8 | 86.7 / 97.1 | 92.2 / 96.8 | – |
| ZeroBench-main pass@5 (– / +Python) | 23.0 / **41.0** | 23.0 / **46.0** | 17.0 / 35.0 | 17.0 / 34.0 | 22.0 / 41.0 | – |

- Best-in-suite for K3: ProgramBench, SWE-Marathon (+7 over Fable 5), BrowseComp, DeepSearchQA, ResearchRubrics (76.2), MCPMark-Verified, AutomationBench (30.8), SpreadsheetBench 2 (34.8), τ³-Banking, Harvey Lab-AA, OmniDocBench.
- Near-misses: Terminal-Bench 2.1 88.3 vs 88.8; OSWorld-Verified 84.8 vs 85.0; CorpFin v2 71.6 vs 71.8.
- Weak spots the paper calls out: research-level reasoning (CritPt, HLE-Full), Elo-rated knowledge-work suites (GDPval-AA v2 third, AA-Briefcase 1548 second), and harder computer-use (OSWorld 2.0 58.3, SaaS-Bench 60.1).

**Cost and third-party results (Table 5, as of 23 Jul 2026)**

- BrowseComp: best score (91.2%) at **$2.03 per task** — half the cost of GPT-5.6 Sol (90.4%) and an order of magnitude cheaper than the Claude models at max effort.
- GDPval-AA v2: within 50 Elo of GPT-5.6 Sol at 13% lower cost, and **2.6× cheaper than Claude Fable 5**. AA-Briefcase: second-best score at roughly half Fable 5's cost.
- Independent leaderboards: Artificial Analysis Intelligence Index v4.1 **57.1 (#4 of 580)**; Vals Index **74.7 (#2 of 39)**; WebDev Arena **1,678 Elo (#1 of 99)**; Text Arena 1,486 (#8 of 200); Agent Arena 9.1 (#4 of 37).

**Serving and infrastructure**

- **MXFP4 QAT**: MoE expert weights quantized to MXFP4 with MXFP8 activations; attention projections, latent-MoE projections, shared experts and routers stay in higher precision. QAT runs through the *entire* post-training stage (SFT and RL), and RL rollout and training share the quantization scheme, eliminating train/inference mismatch.
- **Speculative decoding**: K3 is pre-trained with one MTP layer structured like a backbone block, which is reused directly as an EAGLE-3 draft model; the draft is fine-tuned under the same MXFP4/MXFP8 QAT configuration.
- **Serving stack**: a KDA-aware prefix cache packs the fixed-size KDA recurrent state into the same paged pool as the MLA KV cache so a prefix is reusable only when both restore together; dedicated kernels for KDA decoding, Block AttnRes and sparse latent MoE; fleet-level cache-affinity scheduling and budget-based admission control. Training side: **MoonEP** gives perfectly balanced expert execution with static computation shapes and zero-copy communication, plus KDA Context Parallelism for million-token sequences and a memory overcommit ratio up to 6.5× in real workloads.
- **Vision tokens**: a 2×2 pixel-shuffle downsample cuts visual tokens 4×, keeping inputs up to 3584×3584 pixels affordable inside the 1M-token budget.

## Concepts & entities

- [[moonshot-ai]] — publisher; releases full K3 weights open.
- [[kimi-linear-kimi-team-2025]] — source of Kimi Delta Attention, the linear-attention layer K3 uses 3-in-4; K3's 48B/3B-style sparsity comparison is drawn against it.
- [[attention-residuals-kimi-team-2026]] — the depth-mixing mechanism (Block AttnRes, N≈8) K3 adopts.
- [[linear-attention]] — KDA is a gated delta-rule linear attention; 69 of 93 layers.
- [[mixture-of-experts]] — Stable LatentMoE: 896 routed / 16 active / 2 shared, latent-width routing.
- [[long-context-llm]] — 1M-token window via NoPE plus a staged 8K→64K→256K→1M curriculum.
- [[scaling-laws]] — the 2.5× claim is a fitted loss-vs-FLOPs curve comparison against Kimi K2.
- [[multimodal-llm]] — native joint vision-language pre-training with MoonViT-V2.
- [[terminal-bench]] — Terminal-Bench 2.1, 88.3.
- [[agent-evaluation]] — large agentic suite (BrowseComp, MCPMark, OSWorld, τ³-Banking, GDPval).
- [[distributed-training]] — MoonEP expert-parallel training with balanced expert execution.
- [[kv-cache]] — state-aware prefix caching for KDA; external KV-cache retention in the RL rollout system.
- [[quantization-performance]] — post-training QAT with MoE expert weights in MXFP4 and their input activations in MXFP8.

## References

- [arXiv:2607.24653](https://arxiv.org/abs/2607.24653)
