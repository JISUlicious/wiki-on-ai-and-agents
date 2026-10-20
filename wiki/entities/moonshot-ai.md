---
title: Moonshot AI
type: entity
created: 2026-07-28
updated: 2026-10-20
sources:
  - flash-kda-moonshot-2026.md
  - attention-residuals-kimi-team-2026.md
  - kimi-k3-moonshot-2026.md
  - anthropic-threat-intelligence-report-september-2026.md
status: draft
importance: medium
tags:
  - org
---

# Moonshot AI

**Moonshot AI** is a Chinese AI lab, publishing as the **Kimi Team**, behind the **Kimi** model family. Within this wiki its distinguishing characteristic is a consistent focus on **attention-architecture efficiency** — it contributes both novel mechanisms and the production kernels that make them fast, rather than treating the two separately.

## Contributions in this wiki

| Work | What it is |
|---|---|
| [[kimi-linear-kimi-team-2025\|Kimi Linear]] | introduces **Kimi Delta Attention (KDA)**, a [[linear-attention]] mechanism refining Gated DeltaNet with finer-grained gating |
| [[flash-kda-moonshot-2026\|FlashKDA]] | the CUTLASS kernel implementation of KDA — ~2× over the `flash-linear-attention` baseline on H20, MIT licensed |
| [[attention-residuals-kimi-team-2026\|Attention Residuals (AttnRes)]] ([[attention-residuals\|concept]]) | replaces fixed unit-weight residual accumulation with softmax attention over preceding layer outputs |

| [[kimi-k3-moonshot-2026\|Kimi K3]] | the flagship: 2.78T-parameter native-multimodal MoE, 104.2B activated, 1M context, open weights |

The pattern worth noting: **KDA and AttnRes are separate research lines that converge in the flagship model.** The [[kimi-k3-moonshot-2026|Kimi K3 technical report]] confirms it directly: K3's backbone is a [[mixture-of-experts]] model activating **16 of 896 routed experts** per token (plus 2 shared experts) via **Stable LatentMoE** — routing in a half-width latent space, stabilized by RMSNorm, a bounded SiTU-GLU activation, and **Quantile Balancing** — with a **1M-token context window** and native vision (MoonViT-V2, trained jointly with text from the start rather than grafted on).

To be precise about the hybrid: it is the *K3 attention stack* that is hybrid, not KDA itself. KDA is the [[linear-attention]] component; each K3 block is 3 KDA layers followed by 1 Gated MLA layer (69 KDA + 24 MLA across 93 layers), the same 3:1 ratio Kimi Linear used, with a trailing global-attention layer. Positions are carried implicitly by KDA's decay (NoPE), which is what lets K3 extrapolate to 1M tokens without RoPE rescaling.

> [!note] Draft
> Created during the FlashKDA ingest. Founding date, funding and headcount are not yet recorded. Kimi K2 is characterized only through the K3 report's comparison table (1.04T total / 32.6B activated, 61 MLA layers, 384 experts with 8 active, 128K context); K1/K1.5 and K2.5 have no pages yet.

## Named in a distillation allegation (2026)

Anthropic's September 2026 [[anthropic-threat-intelligence-report-september-2026|threat report]] says Moonshot silently forwarded Kimi customers' requests to Claude, showed Claude's answers as Kimi's, and kept the exchanges for training (GTG-16002). In one 10-day window it relayed almost 300,000 customer requests, mostly to Opus, through 5,380 fraudulent accounts, and over 23 million exchanges between May and July 2026. The report says Moonshot defeated Claude's *thinking signature* control by replaying the signature in a new session and asking Claude to expand it into the full trace. The report says the relayed traffic exposed Moonshot customers' sensitive data, and that it does not know whether Moonshot told them their requests were going to a third party. Two cases are described:

- **Military-linked surveillance.** A user Anthropic assesses as likely affiliated with the Chinese military loaded a CCTV archive about one targeted individual and asked the model whether that person was behaving abnormally. The footage came from hundreds of cameras in Chengdu, including cameras outside military facilities, institutes affiliated with the China Electronics Technology Group Corporation, and a major state-owned enterprise.
- **A state-owned enterprise's code and credentials.** An engineer building an internal system for a major state-owned enterprise revealed internal code and live credentials from multiple major Chinese companies, including high-profile technology companies. "The user had no way of knowing that their use of Kimi was being forwarded to Claude."

 This is Anthropic's allegation against a competitor. The report calls its attribution "high confidence" but publishes no attribution method, and the lab's response is not recorded here. Details and caveats are on [[illicit-distillation]].

## Related

- [[linear-attention]] · [[mixture-of-experts]] · [[flash-attention]] (the analogous kernel effort for exact attention) · [[vllm]] (day-0 K3 serving support)

## References

- [[kimi-k3-moonshot-2026]] · [[flash-kda-moonshot-2026]] · [[kimi-linear-kimi-team-2025]] · [[attention-residuals-kimi-team-2026]]
- [vLLM: Kimi K3 day-0 support](https://vllm.ai/blog/2026-07-27-k3) — secondary source, superseded for architecture details by the K3 technical report above.

## Related 2026 sources

- [[qwen3.8-flash-next]] — convergent design: Qwen's Gated DeltaNet + [[sparse-attention|sparse attention]] hybrid lands on the same 3:1 linear-to-full ratio as Kimi Linear, reached independently
- [[moba-mixture-of-block-attention-lu-2025]] — MoBA — block attention with an MoE-style gate, deployed for Kimi's long-context serving; 6.5× at 1M
- [[trained-sparse-attention]] — the lineage page
