---
title: "SSD Offloading for LLM Mixture-of-Experts Weights Considered Harmful in Energy Efficiency"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2508.06978"
authors:
  - Kwanhee Kyung
  - Sungmin Yun
  - Jung Ho Ahn
year: 2025
venue: IEEE Computer Architecture Letters
status: complete
importance: medium
tags:
  - 2025
  - paper
---

# SSD Offloading for LLM Mixture-of-Experts Weights Considered Harmful in Energy Efficiency

**Kwanhee Kyung, Sungmin Yun, Jung Ho Ahn** (Seoul National University; Ahn corresponding) — [arXiv:2508.06978](https://arxiv.org/abs/2508.06978), cs.AR, submitted 9 Aug 2025, v1, **4 pages / 6 figures, accepted at IEEE Computer Architecture Letters** (DOI 10.1109/LCA.2025.3592563). Funded in part by **Samsung Electronics' Mobile eXperience (MX) Business** and Korea's MSIT/IITP — worth knowing when reading the mobile coda in §VI.

> [!important] This is an analytical model, not a measurement
> Nothing in the letter is measured on hardware. Every result comes from the authors' own simulator (modified from their Duplex accelerator paper, MICRO 2024; code at `github.com/scale-snu/SSD-offloading`), driven by **three per-bit energy constants** — HBM3 **4.2 pJ/b**, DDR5-7200 **9.2 pJ/b**, NAND Flash read **102.4 pJ/b** — a synthetic workload in which every token picks experts **uniformly at random**, and **a single decode step** (the first one after a 1,024-token prefill). The Flash constant, which carries the whole argument, is derived from **FlashPower (Mohan, Gurumurthi & Stan, DATE 2010)** — a fifteen-year-old NAND power model — by dividing energy-per-read-operation by page size. The DRAM constants come from Micron DDR4/DDR5 datasheets, a 2023 VLSI stacked-DRAM paper, and O'Connor et al.'s Fine-Grained DRAM (MICRO 2017), with NVLink and memory-interface I/O energy added on top. The paper says all of this plainly; it is a back-of-the-envelope made rigorous, and its conclusions are exactly as strong as 102.4 pJ/b.

## Problem

[[mixture-of-experts|MoE]] models are almost entirely expert weights — **96.1%** of [[deepseek-r1|DeepSeek-R1]]'s 671B parameters — yet each token touches only a sliver of them: **~25.0%** for Mixtral 8x7B (top-2 of 8), **~3.5%** for DeepSeek-R1 (top-8 of 256 plus one shared expert), **~1.6%** for Llama 4 Maverick (top-1 of 128 plus one shared). That sparsity is the whole premise of *MoE-prefetching* (Pre-gated MoE, Hwang et al., ISCA 2024): park the gated experts in a cheaper, larger tier — CPU DDR or SSD — and hide the fetch behind compute by having layer *i*'s gate predict layer *i+1*'s experts. Latency can be hidden. The letter's point is that **energy cannot**: a byte read from Flash costs the same joules whether it is read early or late, and Flash costs roughly 25× more joules per byte than HBM.

Because decode is where sparsity is sharpest (one token per request per step) and where [[arithmetic-intensity]] is ~1 FLOP/B — so that energy per token is essentially *bytes moved × pJ/b of the tier they came from* — the analysis is confined to the decode stage. The question is simply: what is the per-token energy of decoding when the gated experts live in HBM, in DDR, or on SSD?

## The model — three constants and a simulator

**Per-bit energies (Fig. 2, "representative server system", NVLink4.0):**

| Tier | Energy | Basis |
|---|---|---|
| HBM3 read or write | **4.2 pJ/b** | Micron datasheets [4, 7], VLSI 2023 stacked DRAM [8], O'Connor et al. MICRO 2017 [9] for interface scaling |
| DDR5-7200 read or write | **9.2 pJ/b** | same sources; **stated only in Fig. 2, never in the prose** |
| NAND Flash read | **102.4 pJ/b** | FlashPower, DATE 2010 [5]: energy per read op ÷ page size |

Values "account for both the energy consumed within the memory/storage chips and that consumed in the external I/O paths" (NVLink plus memory-interface energy). The tiers compose along a path: with GPUDirect Storage the SSD case is Flash read → HBM write → HBM read, i.e. **102.4 + 4.2 + 4.2 = 110.8 pJ/b, a ~26× premium over the 4.2 pJ/b HBM-only baseline** (the paper's arithmetic). *Derived:* the DDR path is 9.2 + 4.2 + 4.2 = **17.6 pJ/b, ~4.2× HBM** — the paper never states this number, but it is what makes CPU offload merely bad and SSD offload harmful.

**System (Table I and §IV):** a DGX H100 mirror — up to eight H100 SXM5 80 GB per node, NVLink4.0 at **450 GB/s per direction** (NVLink5.0, used in Figs. 4–6, doubles it), InfiniBand at 8 × 50 GB/s between nodes, **256 GB of DDR5-7200 and NVMe SSDs per GPU**. Everything BF16. Background power is charged in every scenario: **57 W per 256 GB of DDR** (static) and **70 W per GPU** (idle); SSD idle power is deemed negligible (Samsung 9100 PRO datasheet) and dropped. GPU compute energy is cache access plus MACs. MoE layers use expert parallelism; FC layers and shared experts use [[tensor-parallelism|tensor parallelism]] within a node and data parallelism across nodes ([[distributed-training]]).

| Model | Params | Nodes × GPUs | Dense FFN / MoE decoders | Experts | top-k |
|---|---|---|---|---|---|
| Mixtral 8x7B | 47B | 1 × 4 | – / 32 | 8 gated | 2 |
| DeepSeek-R1 | 671B | 4 × 8 | 3 / 57 | 256 gated + 1 shared | 8 |
| Llama 4 Maverick | 400B | 1 × 8 | 24 / 24 | 128 gated + 1 shared | 1 |
| Llama 3.3 (dense) | 70B | 1 × 8 | 80 / – | – | – |

Two modelling choices matter more than anything downstream. First, **"the aggregated read bandwidth from CPU memory and SSD to the GPU matches the unidirectional bandwidth of the NVLink interconnect"** — SSD and DDR are both assumed to deliver 450–900 GB/s into HBM, which is why Fig. 5 has one offload bar per interconnect rather than one per tier. This is *extremely* generous to SSD on latency (a PCIe 5.0 NVMe drive reads at ~14 GB/s; saturating NVLink4.0 takes dozens of them per GPU) and is the reason the paper can say latency is "manageable". Second, **token-to-expert assignment is uniform random**, per decoder — the assumption least favourable to any caching scheme (below).

## Results

### Energy penalty versus batch size (Fig. 4, NVLink5.0)

Text-stated ranges: SSD offload costs **3.8×–12.5× the HBM baseline for Mixtral and 4.7×–9.8× for DeepSeek-R1, varying with batch size**, and **2.1×–3.6× more than DDR offload**. The abstract's headline "up to ~12×" is therefore **Mixtral 8x7B, not DeepSeek-R1**; R1 never exceeds 9.8×. Baseline absolute energies are printed on the figure; the SSD/HBM ratios other than the four stated in the text are read off the right-hand axis (±0.5×):

| Model | Batch | Baseline HBM (J/token) | SSD ÷ HBM | *Derived* SSD (J/token) |
|---|---|---|---|---|
| Mixtral | 1 | 3.21 | ≈ 8.7× | ≈ 28 |
| Mixtral | 4 | 1.51 | ≈ 11× | ≈ 17 |
| Mixtral | **16** | 0.47 | **12.5×** (text) | ≈ 5.9 |
| Mixtral | 64 | 0.13 | ≈ 11× | ≈ 1.4 |
| Mixtral | 256 | 0.05 | ≈ 8× | ≈ 0.4 |
| Mixtral | 1024 | 0.03 | **3.8×** (text) | ≈ 0.11 |
| DeepSeek-R1 | 4 | 11.96 | ≈ 8.5× | ≈ 100 |
| DeepSeek-R1 | **16** | 4.99 | **9.8×** (text) | ≈ 49 |
| DeepSeek-R1 | 64 | 1.91 | ≈ 9.5× | ≈ 18 |
| DeepSeek-R1 | 256 | 0.63 | ≈ 8× | ≈ 5 |
| DeepSeek-R1 | 1024 | 0.28 | **4.7×** (text) | ≈ 1.3 |

Fig. 1 — the abstract's figure — is the DeepSeek-R1 batch-1024 point on NVLink4.0: normalised energy **HBM 0.20 : DDR 0.32 : SSD 1.00**, i.e. **4.9× vs HBM and 3.1× vs DDR, with SSD absolute = 1.4 J per token**. The 4.7×/4.9× pair is the same point under NVLink5.0/4.0 and the two are consistent (0.28 J × 4.7 ≈ 1.3 J). **There is no batch-1 data point for DeepSeek-R1 anywhere in the paper**; R1's sweep starts at batch 4.

The batch dependence is **non-monotonic**, and the mechanism is worth stating precisely because the intuitive story (more tokens → more experts → worse) is the wrong one:

- **Below saturation** (batch 1→16 for Mixtral), per-token background energy (508 W of idle GPU + DDR static on the 4-GPU Mixtral node) amortises away while the SSD term is still growing with the number of distinct experts touched, so the *ratio* rises.
- **At saturation** — "around batch size 16 for Mixtral and 64 for DeepSeek-R1" — the batch touches essentially every expert, the SSD term is maximal, and the ratio peaks (12.5×, 9.8×). Fig. 5's per-GPU active-expert fractions show the same thing: Mixtral 55% → 95% → 100% at batch 1 → 4 → 16; R1 41% → 75% → 100% at 4 → 16 → 64.
- **Above saturation**, the expert bytes per batch are fixed, so per-token *access* energy falls as 1/batch while compute energy grows — and the ratio falls back toward the compute floor: 3.8× and 4.7× at 1024.

**SSD never becomes acceptable at any batch size the paper sweeps.** Even at 1024, where amortisation is greatest, Flash access is still **73% (Mixtral) and 80% (R1) of all per-token energy**, versus **11% and 15%** for the equivalent HBM reads in the baseline.

### Latency: prefetching works (Fig. 5)

With prefetching and NVLink5.0, the per-batch token-generation latency penalty falls to **as low as 1.32× (Mixtral) and 1.25× (DeepSeek-R1)** — both at batch 1024 — and to roughly 2–2.5× on NVLink4.0 (figure read). Absolute per-batch latencies run ~5–20 ms (Mixtral) and ~8–37 ms (R1, batch 4–1024) for the baseline. The paper's own summary of the trade: **"sacrificing energy efficiency for manageable latency and the ability to use larger models than fit in device memory."** Remember that the latency result rests on the NVLink-saturating bandwidth assumption above.

### When could Flash win? (Fig. 6, §V-B)

The second experiment changes the comparison: a **400B MoE that needs SSD (Llama 4 Maverick) versus a 70B dense model that fits in HBM (Llama 3.3)**, in a GPU + SSD system on NVLink5.0, as Flash read energy is scaled down from today's 102.4 pJ/b. At batch 1 Llama 4 activates **17B parameters (4.3% of 400B), 17.8% of which come from the SSD-resident gated experts**; Llama 3.3 always reads all 70B. The full table (per-token energy ratio, Llama 4 ÷ Llama 3.3; **<1 means SSD wins**):

| Batch \ Flash energy (×102.4 pJ/b) | 1.0 | 0.9 | 0.8 | 0.7 | 0.6 | 0.5 | 0.4 | 0.3 | 0.2 | 0.1 |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 1.14 | 1.08 | 1.03 | **0.97** | 0.91 | 0.85 | 0.80 | 0.741 | 0.68 | 0.63 |
| 2 | 1.78 | 1.67 | 1.55 | 1.44 | 1.33 | 1.22 | 1.11 | **0.999** | 0.89 | 0.78 |
| 3 | 2.49 | 2.32 | 2.16 | 2.00 | 1.83 | 1.67 | 1.50 | 1.339 | 1.18 | 1.01 |
| 4 | 3.15 | 2.94 | 2.72 | 2.51 | 2.29 | 2.08 | 1.86 | 1.649 | 1.43 | 1.22 |

The paper's reading: SSD wins "only when the Flash read energy is significantly reduced — specifically, to approximately 1/10th of its current value (∼10 pJ/b)", and "most prominent at low batch sizes (batch size < 3)". So the viability threshold is **102.4 pJ/b today → ~10 pJ/b target**, which would put Flash reads at roughly DDR5's 9.2 pJ/b.

> [!warning] The "order of magnitude" is stricter than the paper's own table
> At **batch 1, break-even is at 0.7× (~72 pJ/b) — a 30% improvement, not 10×** — and today's Flash already leaves the 400B MoE only **1.14×** the per-token energy of the 70B dense model. At batch 2 break-even is 0.3× (~31 pJ/b). The 10× figure is what it takes for the win to be *decisive* at batch 1–2 (0.63, 0.78) and to reach parity at batch 3 (1.01); at batch 4 Flash never wins in the range plotted. The abstract's "roughly by an order of magnitude" is the robust-across-batch number, and it is the number to quote, but the batch-1 row is the one that matters for a single-user device and it says something considerably milder.

## Prefetching hides time, not bytes — and caching is not modelled

The reasoning behind "prefetching cannot mitigate the energy penalty" is exactly as simple as it sounds: energy is charged per bit read from a tier, prefetching only changes *when* the bits are read, and "**latency-hiding via prefetching does not alter these underlying energy consumption figures**". The paper does not model prefetch *mispredictions* either, so its prefetcher is an oracle — which is conservative for the thesis, since a real Pre-gated MoE predictor moves *more* bytes than the ideal, not fewer.

What the model does and does not count as reuse:

- **Within a batch, within a layer: yes.** An expert selected by several tokens in the same batch is fetched once per decoder per step. This is the only amortisation in the paper, and it is what makes the ratio fall above saturation.
- **Across decode steps: no.** Fig. 3(c) offloads *all* gated experts; non-MoE weights and shared experts stay in HBM. There is no HBM-resident expert cache, no hot-expert set, no LRU, and no temporal locality — the simulation runs **one decode step** with **uniform random routing**, under which cross-step reuse would be worthless anyway unless HBM held most of the experts, and the paper's HBM baseline already holds all of them. The intermediate scenario every consumer system actually lives in — *some* experts cached in fast memory, skewed routing, streaming the misses — is simply absent.

That gap is the point of contact with the SSD-streaming literature the paper cites but does not engage: [[llm-in-a-flash-alizadeh-2023|LLM in a Flash]] is reference [2], used only for the sentence "Flash offers lower cost per bit but higher read energy and lower bandwidth"; its windowing and neuron-reuse mechanisms — techniques that *reduce bytes moved*, which is the one thing that would move this paper's numbers — go unmentioned. See [[weight-streaming]] and [[flash-moe-ssd-expert-streaming-woods-2026]].

## Scope: a datacenter argument with a mobile coda

The framing is datacenter throughout: DGX H100 nodes, NVLink, 256 GB of DDR5 per GPU, batch up to 1024, and the counterfactual is always *a model that fits* — either the same model in HBM (§V-A) or a smaller dense model in HBM (§V-B). The paper never evaluates the case where the alternative to SSD offload is that the model does not run at all, and it gives **no LPDDR or UFS energy numbers**.

It does, however, address edge devices directly — as the *future* beneficiary rather than the present problem. §VI: "This scenario is particularly relevant to **mobile/edge platforms**, which typically operate at small batch sizes, maximizing MoE sparsity, face strict battery constraints, and possess analogous LPDDR/UFS memory hierarchies. Thus, if Flash reaches the ∼10 pJ/b read energy target, mobile represents a **prime domain** to potentially benefit from large, accurate MoE models using energy-efficient Flash offloading." The Samsung MX funding line reads the same way.

> [!note] What this letter does and does not say about a laptop
> For a concept page that wants to note the argument is weaker on a consumer machine, the paper's own numbers support the caveat better than its title does:
> - The single-user regime is batch 1, where the paper has **no DeepSeek-R1 point at all** and where its MoE-vs-dense table says today's Flash costs **1.14×**, not 12×.
> - On a laptop the fast tier is LPDDR, whose nearest analogue here is the **9.2 pJ/b DDR row**, so the relevant penalty is *SSD-versus-DDR* — **2.1×–3.6×** in the datacenter model — rather than SSD-versus-HBM.
> - The paper is BF16 throughout; 4-bit experts ([[quantization]], [[gguf]]) divide the Flash term by four, and expert caching with skewed routing divides it again. Neither is modelled.
> - What does *not* transfer is the latency story: consumer NVMe reads at 5–14 GB/s, not the 450–900 GB/s the model assumes, so on a laptop the binding constraint on SSD streaming is **bandwidth, not energy** (*derived*). The paper's "latency is manageable, energy is not" inverts on a Mac.

## Relation to the other HBM-economics paper

[[hbm-is-not-all-you-need-wei-2026|Wei et al.]] and this letter are two halves of one statement about where the memory hierarchy should sit relative to inference phase. Wei et al. show that HBM's *bandwidth* is wasted in prefill (MBU below 1% at 16K tokens) and move prefill to cheap GDDR. Kyung et al. show that in decode — where every active expert byte is read once per step and energy is bytes × tier pJ/b — the cheaper tiers below HBM are the wrong place for weights on *energy*, even when latency is hidden. Read together: **HBM is a decode technology on both axes** — the only phase that needs its bandwidth is also the phase where its 4.2 pJ/b is decisive — and the hierarchy can be pushed *down* for prefill compute but not for decode weights. The same [[arithmetic-intensity]] argument underlies both, and [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] and [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] make the corresponding points for SRAM accelerators and FPGA state, respectively.

## Limitations

- **Pure model, no measurement.** One simulator, three constants, synthetic uniform routing, a single decode step. No SSD, GPU or DDR was ever instrumented.
- **The load-bearing constant is from a 2010 NAND power model.** 102.4 pJ/b comes from FlashPower (DATE 2010) with no cross-check against contemporary 3D TLC/QLC parts or measured NVMe drive power; the only modern SSD citation (Samsung 9100 PRO) is used for idle power. The whole result scales linearly with this number, and Fig. 6 shows how sensitive the conclusion is to it.
- **SSD bandwidth is assumed to saturate NVLink**, which makes the "latency is manageable" half of the trade-off a statement about an idealised storage array, not a drive.
- **No caching, no prefetch misprediction, no quantization.** Each would move the numbers, in different directions (caching and quantization down, misprediction up).
- **No batch-1 DeepSeek-R1 result**, though R1 is the model named in the abstract; the "up to ~12×" is Mixtral at batch 16.
- **The 10× threshold overstates its own table** at batch 1 (break-even at 0.7×).
- **Table I lists DeepSeek-R1 as 3 dense + 57 MoE decoders (60)**; the DeepSeek-V3 report specifies 61 layers with 58 MoE — a one-layer discrepancy, immaterial to the conclusions but a sign the configs were transcribed rather than loaded.
- **Background power is charged uniformly**, including 57 W per 256 GB of DDR that the HBM baseline does not use for weights — mildly conservative for the paper's thesis.
- Prefill is out of scope by design; so is anything about accuracy — the "larger MoE is more accurate than smaller dense" premise of §V-B is asserted, not evaluated.

## Related

- [[mixture-of-experts]] — the sparsity that motivates offloading and, at low batch, the only thing that could make Flash viable; this paper adds the activation-saturation batch sizes (~16 for 8 experts, ~64 for 256).
- [[deepseek-r1]] · [[deepseek-r1-2025]] — the 671B / 256-expert / top-8 configuration; 96.1% expert weights, 3.5% active per token.
- [[weight-streaming]] — the consumer-side practice this paper indicts from the datacenter side; the batch-1 and DDR numbers above are the honest transfer.
- [[llm-in-a-flash-alizadeh-2023]] — the paper's ref [2], and the bytes-reducing mechanisms it does not model.
- [[flash-moe-ssd-expert-streaming-woods-2026]] — expert streaming from SSD in 2026; the caching scenario absent here.
- [[hbm-is-not-all-you-need-wei-2026]] — the prefill half of the "HBM is a decode technology" argument (above).
- [[arithmetic-intensity]] · [[llm-inference-roofline-yuan-2024]] — why decode energy is bytes × pJ/b and why the ratio collapses toward the compute floor at batch 1024.
- [[hardware-accelerator]] — this wiki's collection of memory-hierarchy-vs-phase papers; this one is the storage tier.
- [[kv-cache]] — the "Attention (memory/storage access)" sliver in Figs. 1 and 4 is the KV read; at 1,024-token prompts it is negligible next to expert weights, which is itself a useful calibration.
- [[zero-offload-ren-2021]] — the DDR-offload precedent on the training side; the same 9.2-vs-4.2 pJ/b gap applies.
- [[quantization]] · [[gguf]] · [[llama-cpp]] — the levers a real SSD-streaming deployment uses and this model omits.
- [[distributed-training]] · [[scalable-moe-training-megatron-core-yan-2026]] — expert parallelism as modelled here.
- [[distserve-zhong-2024]] · [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] · [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] — the phase-vs-hardware family.
- [[llm-memory-and-compute-bottlenecks]] — the wiki's inference-bottleneck query; weights-as-bytes-per-token is the decode-side term.
- [[memory-management]] — note: this wiki's page of that name is about *agent* memory lifecycles, not the hardware memory hierarchy; linked for disambiguation only.
- [[scaling-laws-kaplan-2020]] — the paper's ref [1], cited for the parameter-growth trend that creates the capacity problem.
- [[edge-inference]] · [[energy-efficiency]] — no pages yet; this source is a natural seed for both.
- [[nvidia]] · [[deepseek]] · [[llama]] — the platform and the model families.

## References

- [arXiv:2508.06978](https://arxiv.org/abs/2508.06978) — IEEE Computer Architecture Letters, 2025, DOI 10.1109/LCA.2025.3592563.
- Kaplan et al., "[[scaling-laws|Scaling Laws]] for Neural Language Models," [arXiv:2001.08361](https://arxiv.org/abs/2001.08361) — ref [1].
- Alizadeh et al., "LLM in a Flash," [arXiv:2312.11514](https://arxiv.org/abs/2312.11514) — ref [2]; the Flash cost/energy/bandwidth framing.
- Hwang et al., "Pre-gated MoE," ISCA 2024 — ref [3]; the prefetching scheme modelled.
- Mohan, Gurumurthi & Stan, "FlashPower: A detailed power model for NAND flash memory," DATE 2010 — ref [5]; source of the 102.4 pJ/b.
- O'Connor et al., "Fine-Grained DRAM," MICRO 2017 — ref [9]; DRAM interface energy scaling.
- Yun, Kyung et al., "Duplex: A Device for Large Language Models with MoE, GQA, and Continuous Batching," MICRO 2024 — ref [11]; the simulator this study modifies, and the parallelism scheme it inherits.
- Simulator: `https://github.com/scale-snu/SSD-offloading`.
