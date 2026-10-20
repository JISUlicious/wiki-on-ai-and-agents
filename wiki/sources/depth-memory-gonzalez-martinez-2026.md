---
title: "Depth Memory"
type: source
created: 2026-10-20
updated: 2026-10-20
sources:
  - depth-memory-gonzalez-martinez-2026.md
tags:
  - 2026
  - paper
status: complete
importance: medium
authors:
  - David González-Martínez
  - Philipp Nazari
  - T. Konstantin Rusch
  - Shiwei Liu
introduces:
  - [[depth-mixing]]
  - [[linear-depth-memory]]
year: 2026
venue: "arXiv preprint (cs.LG)"
arxiv_id: "2610.18809"
---

# Depth Memory

**Source**: `sources/depth-memory-gonzalez-martinez-2026.md` (text extracted from the arXiv PDF). [arXiv:2610.18809](https://arxiv.org/abs/2610.18809), v1, 2026-10-13.

| | |
|---|---|
| **Authors** | David González-Martínez and Philipp Nazari (equal contribution), T. Konstantin Rusch, Shiwei Liu |
| **Affiliations** | ELLIS Institute Tübingen, Max Planck Institute for Intelligent Systems, Tübingen AI Center; also ETH Zürich (Nazari) and Liquid AI (Rusch) |
| **Code and data** | The codebase builds on DepthBench (Wang et al., 2026) and OLMo-core. The paper links code and checkpoints; the URLs were not recoverable from the PDF text. |

## Summary

Transformer research has innovated along two axes: *sequence mixing* (softmax attention, [[linear-attention]], [[state-space-model|SSMs]]) and *channel mixing* (gated MLPs, [[mixture-of-experts|MoE]]). The third axis, **depth**, has mostly stayed with the ResNet-style [[residual-connection]]: each layer's output is added to a running sum. Two recent alternatives work at scale:
- **[[hyper-connections|Manifold-Constrained Hyper-Connections (mHC)]]**, from DeepSeek, keep an *expanded* residual stream;
- **[[attention-residuals|Attention Residuals (AttnRes)]]**, from the Kimi Team, softmax-attend over earlier layer outputs.

The paper calls this family **[[depth-mixing|depth mixing]]**.

It then reframes depth mixing as **storing and retrieving information across layers**. Its **Depth Memory (DM)** framework describes any depth mixer as a tuple of *streams* plus two operators:
- **Read** forms each layer's input from the streams, using a query.
- **Write** updates the streams with the layer's output, using a key and value.

Both operate per token. Three existing designs fit the framework:

| Design | Stream | Read | Write | Kind |
|---|---|---|---|---|
| PreLN (standard [[residual-stream]]) | `x` | identity | `x + δ` | — |
| AttnRes | growing key–value cache of all layer outputs | softmax attention | append | **explicit** (keeps every representation) |
| mHC | expanded residual matrix `X` | `X·q` | `X·H_res + v·kᵀ` | **implicit** (fixed-size, additive) |

The paper's own instance is **[[linear-depth-memory|Linear Depth Memory (LDM)]]**, an implicit depth mixer built from first principles:
- **Linear attention over depth.** The memory stream is a [[linear-attention]]-style matrix. Removing the softmax from depth attention turns the growing cache into a fixed-size sum of outer products, `M = Σ v kᵀ`.
- **Exact depth regression.** LDM reads that sum as an approximation to a regression, "reconstruct earlier layer outputs from their keys". It then solves the regression **exactly with recursive least squares (RLS)**.
- **Why exact is affordable here.** Sequence-mixing models need parallel, approximate solvers (delta rule, DeltaProduct, MesaNet), because tokens must be processed in parallel. Layers already run one after another, so a sequential solver costs little extra.

LDM therefore carries **three streams** across depth: the normal residual stream, a matrix memory stream, and an inverse-covariance stream (the RLS state).

## Key points

- **The three streams of LDM:**
  - **Read:** `x + γ ⊙ M(g ⊙ q)`. The query `q` and read gate `g` come from a small latent (H = 32 or 64) computed from the residual stream.
  - **Write:** an RLS update with learnable per-layer forgetting factors `λ`, per-token importance weights `ω` and a ridge penalty. The update uses the gain `κ = P k / (λ + kᵀ P k)`, the prediction error `u = v − M k`, `M ← M + u κᵀ`, and `P ← (P − κ kᵀ P)/λ`.
  - **Groups:** the state is split into G = 16 groups, each with value dimension equal to the attention head dimension.
  - **Key dimension:** small (D_k = 8 in the main runs).
  - **Overhead:** about **1% more parameters** than PreLN, similar to mHC.
- **Hardware-aware implementation** (CuTe DSL kernels for Hopper and Blackwell GPUs):
  1. **State unwinding.** The RLS update is reversible, `M_{l−1} = M_l − u κᵀ`. The backward pass rebuilds earlier states from the final one plus a small per-layer context, so per-layer states need not be stored.
  2. **Windowed writes.** Writes inside a window of E = 4 layers are deferred, and each Write is fused with the next layer's Read, which cuts memory traffic.
- **Results** (pretraining on FineWeb-Edu: 400M Llama-like on 50B tokens, 1.5B Qwen-like on 100B tokens; learning rate swept per method):

  | Model | Val ppl | WikiText ppl | LAMBADA ppl | Avg zero-shot acc |
  |---|---|---|---|---|
  | **400M** PreLN | 12.73 | 22.59 | 25.32 | 49.05 |
  | AttnRes (Full) | 12.35 | 21.46 | 19.62 | 49.98 |
  | AttnRes (Block) | 12.12 | 20.86 | 20.71 | **50.61** |
  | mHC | 12.23 | 21.20 | 20.23 | 49.55 |
  | LDM, D_k = 4 | 12.14 | 20.84 | 19.59 | 50.20 |
  | **LDM** | **12.08** | **20.70** | **19.10** | 50.46 |
  | **1.5B** PreLN | 9.48 | 15.06 | 10.49 | 58.41 |
  | AttnRes (Block) | 9.26 | 14.54 | 8.84 | 59.42 |
  | mHC | 9.25 | 14.44 | 9.05 | 59.68 |
  | **LDM** | **9.15** | **14.21** | **8.42** | **59.71** |

  - LDM has the lowest perplexity at both scales.
  - Its zero-shot average is best at 1.5B but **second at 400M**, behind Block AttnRes. At 1.5B the averages of mHC and LDM differ by 0.03 points.
  - **Matched state size:** at 400M, LDM with D_k = 4 holds the same number of memory entries as mHC's expanded stream and still beats mHC.
- **Training-loss figure (1.5B).** The figure marks two multipliers, ×1.22 and ×1.09, for the tokens LDM needs to reach each baseline's final loss. The extracted figure text doesn't make clear which multiplier belongs to which baseline.
- **Throughput** (B200, Qwen-like 1.7B and 14B):
  - LDM beats Full AttnRes in every setting.
  - It beats mHC in all training settings and most inference settings.
  - It stays close to Block AttnRes, especially at decode.
  - These are relative throughput charts (% of PreLN); the paper gives no absolute speeds in the main text.
- **Ablations** (400M, 15B tokens; change in held-out perplexity):
  - **Keeping the separate residual stream is essential.** Feeding the layer only the memory readout, or replacing the stream with the layer output, does **worse than PreLN**.
  - **Exact RLS beats approximations.** Swapping RLS for the delta rule or a plain outer-product update still beats PreLN but trails RLS. That fits sequence-model results where more accurate test-time regression solvers help.
  - **What to store and how to query:** storing the residual `x` instead of the layer output `δ` hurts most. Queries and keys from `δ`, or static per-layer queries, also hurt. More groups give small, steady gains.
  - **Negligible at this scale:** fixing the write weight to 1, or turning forgetting off (`λ = 1`).
- **Aspect ratios** (the DepthBench protocol, fixed budgets of 400M and 1.6B):
  - LDM beats PreLN at every width/depth shape.
  - At 400M it prefers moderate shapes; plain Hyper-Connections win only the deepest one.
  - At 1.6B it is best at every shape, and its perplexity keeps falling with depth while HC stops improving.
- **Limitations (stated):**
  - only tested up to 1.6B parameters and 100B tokens;
  - can add peak training memory;
  - the space of other DM instances (other linear recurrences, nonlinear Read/Write) is unexplored.

## Entities mentioned

- [[moonshot-ai]] — the Kimi Team, authors of AttnRes; Block AttnRes ships in [[kimi-k3-moonshot-2026|Kimi K3]]
- [[deepseek]] — authors of mHC (Xie et al., 2026), used in the DeepSeek-V4 line ([[deepseek-v4.1-flash-kv-cache-compression-deepseek-2026]])

## Concepts discussed

- [[depth-mixing]] — the third mixing axis, after sequence and channel; the paper's framing
- [[linear-depth-memory]] — the proposed method
- [[attention-residuals]] · [[hyper-connections]] — the two baselines the framework absorbs
- [[residual-connection]] · [[residual-stream]] — PreLN as the one-stream special case
- [[linear-attention]] — LDM is linear attention over depth, not over tokens
- [[test-time-memory]] — the regression view of associative memory that LDM moves from the sequence axis to the depth axis
- [[layer-normalization]] — pre-norm (PreLN) as the baseline

## Notable quotes

> "we consider the mixing of representations over depth as a problem of storing and retrieving information across layers" — González-Martínez et al., abstract

## References

- [[attention-residuals-kimi-team-2026]] — the AttnRes baseline (Full and Block)
- [[kimi-k3-moonshot-2026]] — Block AttnRes in production
- [[qwen3.8-flash-next]] — Gated Residual, another production depth mixer (not evaluated in this paper)

_Original source: `sources/depth-memory-gonzalez-martinez-2026.md` (PDF: `sources/depth-memory-gonzalez-martinez-2026.pdf`)_
