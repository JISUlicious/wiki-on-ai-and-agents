---
title: "LLM in a flash: Efficient Large Language Model Inference with Limited Memory"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2312.11514"
authors:
  - Keivan Alizadeh
  - Iman Mirzadeh
  - Dmitry Belenko
  - S. Karen Khatamifard
  - Minsik Cho
  - Carlo C Del Mundo
  - Mohammad Rastegari
  - Mehrdad Farajtabar
year: 2023
venue: ACL 2024
status: complete
importance: high
tags:
  - 2023
  - paper
---

# LLM in a flash: Efficient Large Language Model Inference with Limited Memory

**Keivan Alizadeh, Iman Mirzadeh, Dmitry Belenko, S. Karen Khatamifard, Minsik Cho, Carlo C Del Mundo, Mohammad Rastegari, Mehrdad Farajtabar** ([[apple|Apple]]) — [arXiv:2312.11514](https://arxiv.org/abs/2312.11514), cs.CL / cs.AI / cs.LG. v1 12 Dec 2023, v2 4 Jan 2024, **v3 30 Jul 2024 (ACL 2024 camera-ready; this page follows v3)**. Alizadeh and Mirzadeh are flagged as major contributors.

> [!important] The headline numbers changed between versions
> v1/v2 (Dec 2023 – Jan 2024) claim **"4-5x and 20-25x"** speedups on CPU and GPU. The ACL v3 abstract softens this to **"up to 4x and 20x"** and the intro to **"up to 4x, 7x, and 20x"** for CPU, Metal and NVIDIA GPU — though v3's Discussion and Appendix E still carry the old "4-5x / 20-25x" sentence verbatim. The underlying Table 3 supports **4.8× (CPU, M1 Max) and 26.4× (RTX 4090)** for OPT 6.7B against a *naive reload-everything* baseline, so the v1 phrasing was actually the accurate one; the other four models land at 2.4–4.4× on CPU. Everything below is measured from v3's tables, not from any abstract.

## Problem

A 7B model in fp16 needs ~14 GB just for its weights, which exceeds the DRAM of most personal devices. The standard fix is to make the model smaller ([[quantization]]); this paper's fix is to leave the model *on flash* and stream in only the weights each token actually needs. That is the first paper-length treatment of what this wiki calls [[weight-streaming]], and it establishes the two facts every later system inherits:

1. **Flash is an order of magnitude larger than DRAM but ~10–100× slower**, and slower still for the *random* reads sparse loading requires. The M1 Max's 1 TB SSD delivers **>6 GiB/s on a 1 GiB sequential uncached read**, but small random reads pay a latency-to-first-byte tax that collapses throughput. The paper's benchmark (Fig. 2b) finds that **≥32 KiB random reads across 32 threads** recover enough bandwidth to be usable — hence the whole design is organised around making every flash read at least that large.
2. **FFN [[activation-sparsity]] is the lever.** In a ReLU-activated model, >90% of intermediate FFN neurons are zero for any given token, so >90% of the FFN weights (which are ~⅔ of the model) need not be loaded. Attention weights and embeddings (~⅓) stay resident.

The latency model is `I/O from flash + DRAM management + compute`, and the paper explicitly declines to optimise compute — this is a paper about the first two terms.

## Mechanism

Three techniques, each targeting one term of the cost model.

### 1. Sparsity prediction + windowing (reduce bytes loaded)

**Low-rank predictor.** Following DejaVu, a per-layer rank-*r* predictor (sigmoid + threshold, default 0.5) guesses which FFN neurons ReLU will keep alive. Two differences from DejaVu: the input is the **current layer's attention output** rather than the previous layer's FFN output (later input, more accurate), and the loss is balanced across positive and negative neurons. Predictors are trained on **10,000 [[c4|C4]] samples for 2 epochs, ~4 A100-hours each**. Ranks are chosen per layer group:

| Model | Rank (most layers) | Rank ("sensitive" layers) | Threshold | Predictor overhead |
|---|---|---|---|---|
| OPT 6.7B | 128 (first 28) | 1024 (last 4) | 0.5 | avg rank 240 → **1.25% of model size**, <2.4% of non-embedding FLOPs; **2.75% of CPU / 4.8% of GPU inference time** |
| Falcon 7B (relufied) | 256 (first 28) | 1152 (last 4) | 0.5 | 4% of model size and CPU time |
| Persimmon 8B | 256 (first 32) | 1152 (last 4) | **0.7** (less sparse) | 2.85% of CPU time |
| Phi-2 (relufied) | 160 / 480 by 4-layer group | 800 (middle 8) | 0.40 | — |
| Llama 2 7B (FATReLU) | 256 | 1024 (4 middle layers) | — | 3.92% of CPU time |

On OPT the predictors average **5% false negatives and 7% false positives**; the false negatives cluster near zero pre-activation, which is why they are harmless. To keep false negatives low the predictor must *over-predict*: it marks **~3× as many neurons active as the model actually fires** (OPT and Persimmon; ~2× for Phi-2). That over-prediction is the price of the accuracy results below and is the largest source of wasted I/O in the method.

**Windowing (sliding window of active neurons).** Rather than evicting a neuron's weights after one token, keep the union of active neurons for the last *k* tokens resident, and load only the *difference*. Because aggregated neuron usage `s_agg(k)` is concave in *k* (Fig. 4a), the per-token increment `s_agg(k+1) − s_agg(k)` shrinks as the window grows — larger *k* trades DRAM for I/O.

| Model | Window *k* | FFN neurons loaded per token | FFN resident in DRAM |
|---|---|---|---|
| OPT 6.7B | 4–5 (the paper says both) | **2.4%** (vs ~10% with the predictor alone) | 24% |
| Falcon 7B | 4 | **3.1%** | 33% |
| Persimmon 8B | 4, shrunk dynamically at a 25%-of-FFN cap | — | ≤25% |
| Phi-2 | dynamic | — | — |
| Llama 2 7B | 2 | — | capped at 55% of model |

So per token the method reads **~2–3% of the FFN** — "only 2% of the FFN layer" in v1's phrasing — which is how 13.4 GB of naive traffic becomes 0.2 GB (Table 2).

### 2. Row-column bundling (increase chunk size)

FFN neuron *i* is exactly one column of the up-projection and one row of the down-projection, and both are needed iff neuron *i* fires. Storing them **contiguously in flash** means one read fetches both, doubling the chunk from `d_model × bytes` to `2·d_model × bytes` — **32 KiB for OPT in fp32** (d_model = 4096), 35.5 KiB for Falcon. That single change lifts measured random-read throughput from **1.25 GB/s to 2.25 GB/s** and halves I/O latency (164 → 87 ms, Table 2), because it moves the read size onto the plateau of Fig. 2b.

A second bundling idea — pairing each neuron with its most co-activated "closest friend" — is reported as a **negative result** (Appendix D): closest friends co-activate ≥95% of the time, but the most active neurons are everyone's closest friend, so bundling loaded them repeatedly and net traffic went *up*.

### 3. Preallocated DRAM data structure (reduce management overhead)

Naively appending new neurons means reallocating and rewriting ~25% of the resident FFN per token. Instead each layer preallocates a `Req_i × 2·d_model` matrix (Req_i = max neurons the chosen window ever needed on a C4 validation subset), plus a pointer vector (original neuron index), bias vector, `num_used`, and `last_k_active`. **Delete** = swap-with-last, an `O(c·d_model)` rewrite for *c* evicted neurons; **insert** = append at `num_used`; **inference** uses `matrix[:n, :d_model]` as the up-projection and `matrix[:n, d_model:]ᵀ` as the down-projection. This works because FFN output is invariant to neuron ordering. Management still costs **58 ms/token on CPU and 34 ms on GPU** for OPT — on GPU that is more than either I/O or compute.

On macOS the FFN was implemented as a binary extension using **dynamism-friendly Metal kernels borrowed from [[mlx|MLX]]** and `MTLStorageModeShared` unified-memory buffers so CPU and GPU share the weight cache without copies; PyTorch's MPS backend alone "demonstrates rather steep performance cliffs" under shape dynamism.

## Setup

- **Hardware:** Apple **M1 Max, 1 TB SSD** (CPU fp32; Metal fp16); Apple **M2 Ultra, 2 TB SSD** (Metal fp16); **Linux box with a 24 GB NVIDIA RTX 4090** (bf16). macOS 14.3.
- **Flash bandwidth:** 6.10 GB/s sequential is the figure used for baselines (M2 Ultra: ~6.25 GB/s, derived from 2145 ms); 1.25 GB/s for 32-thread random reads, 2.25 GB/s with bundling. Reads bypass the OS page cache (`F_NOCACHE` on macOS, DirectIO on Linux, `purge` before runs), which the paper calls a deliberately pessimised lower bound.
- **DRAM budget — what "twice the size of DRAM" means:** the model is given a DRAM allocation of **~50% of its own footprint** (65% for Phi-2, which is less sparse; Llama 2 capped at 55%). These are physically well-endowed machines; the budget is enforced by the implementation, not by the hardware. For OPT 6.7B fp16 (13.4 GB) that is ~6.5 GB, and the realised residency is **52.1%**: attention 32.3% + embeddings 3% + predictors 1.25% + FFN window 15.5% (Falcon: 52.9%). The [[kv-cache]] is budgeted separately (≈8% of model size at 2048 context for OPT) and could itself live on flash.
- **Workload:** batch 1, single sequence; 128-token C4 prompts, 256 generated tokens; HuggingFace Transformers + PyTorch.
- **Baselines:** *Naive* = reload the whole model from flash every token. *Hybrid* = keep half resident, reload the other half every token — the paper's stated **theoretical I/O lower bound for any method that does not use sparsity**. Baseline I/O latencies are *computed* best-case figures (13.4 GB ÷ 6.1 GB/s = 2196 ms), not measurements — conservative in the paper's favour.

## Measured results

**Table 2 — OPT 6.7B fp16 on M1 Max, I/O only, ablating each technique:**

| Hybrid | Predictor | Windowing | Bundling | DRAM (GB) | Flash→DRAM per token | Throughput | I/O latency |
|---|---|---|---|---|---|---|---|
| ✗ | ✗ | ✗ | ✗ | 0 | 13.4 GB | 6.10 GB/s | **2196 ms** |
| ✓ | ✗ | ✗ | ✗ | 6.7 | 6.7 GB | 6.10 GB/s | 1090 ms |
| ✓ | ✓ | ✗ | ✗ | 4.8 | 0.9 GB | 1.25 GB/s | 738 ms |
| ✓ | ✓ | ✓ | ✗ | 6.5 | 0.2 GB | 1.25 GB/s | 164 ms |
| ✓ | ✓ | ✓ | ✓ | 6.5 | 0.2 GB | 2.25 GB/s | **87 ms** |

The predictor cuts bytes 7× but *costs* 5× in throughput (scattered reads); windowing cuts bytes another 4.5×; bundling buys back 1.8× of throughput. The three compose to **25×** on I/O.

**Table 3 — end-to-end per-token latency (ms):**

| Model | Method | Backend | I/O | Mem | Compute | **Total** | Speedup vs naive | vs hybrid |
|---|---|---|---|---|---|---|---|---|
| OPT 6.7B | Naive | CPU (M1 Max) | 2196 | 0 | 986 | 3182 | — | — |
| OPT 6.7B | **All** | CPU (M1 Max) | 105 | 58 | 506 | **669** | **4.8×** | ~3.1× (derived) |
| OPT 6.7B | Naive | Metal M1 | 2196 | 0 | 193 | 2389 | — | — |
| OPT 6.7B | All | Metal M1 | 92 | 35 | 438 | 565 | 4.2× | — |
| OPT 6.7B | Naive | Metal M2 Ultra | 2145 | 0 | 125 | 2270 | — | — |
| OPT 6.7B | All | Metal M2 Ultra | 26 | 8 | 271 | 305 | 7.4× | — |
| OPT 6.7B | Naive | RTX 4090 | 2196 | 0 | 22 | 2218 | — | — |
| OPT 6.7B | **All** | RTX 4090 | 30 | 34 | 20 | **84** | **26.4×** | ~13× (derived) |
| OPT 6.7B | All + speculative (λ=4) | RTX 4090 | 38.5 | 9.5 | 12 | **60** | 37× | — |
| Falcon 7B | Naive / Hybrid / All | CPU | 2295 / 1147 / 161 | 0 / 0 / 92 | 800 / 800 / 453 | 3095 / 1947 / **706** | 4.4× | 2.8× |
| Persimmon 8B | Naive / Hybrid / All | CPU | 2622 / 1311 / 283 | 0 / 0 / 98 | 1184 / 1184 / 660 | 3806 / 2495 / **1041** | 3.7× | 2.4× |
| Phi-2 2.7B | Naive / Hybrid / All | CPU | 885 / 309 / 211 | 0 / 0 / 76 | 402 / 402 / 259 | 1287 / 711 / **546** | 2.4× | 1.3× |
| Llama 2 7B | Naive / Hybrid / All | CPU | 2166 / 974 / 279 | 0 / 0 / 152 | 929 / 929 / 563 | 3095 / 1903 / **994** | 3.1× | 1.9× |

Appendix Table 5 gives standard deviations (OPT CPU 669 ± 40 ms; GPU 84.6 ± 6.2 ms), so the "All" rows are measured, multi-run figures.

Two things the table shows that the abstract does not:

- **Part of the CPU win is compute, not I/O.** Sparse FFN evaluation halves CPU compute (986 → 506 ms). On Metal M1 the sparse path is *slower* to compute (193 → 438 ms) — gather/dynamic-shape overhead — which is why Metal M1 only reaches 4.2× despite the same I/O savings.
- **On the GPU the method is no longer I/O-bound but management-bound.** Of 84 ms, 34 ms is DRAM/GPU bookkeeping and 30 ms is flash; compute is 20 ms. Speculative decoding helps mostly by amortising that bookkeeping (9.5 ms across a λ=4 draft), giving **1.4× on top** (84 → 60 ms) versus the 1.58× speculative decoding alone would give with the model fully resident.

## Quality: what the predictor costs

The paper reports zero-shot accuracy (Table 4), MMLU (Fig. 10) and wikitext2 perplexity (Fig. 11, figure only, no numbers). Zero-shot, best predictor configuration per model:

| Model | ARC-Easy | ARC-Challenge | HellaSwag |
|---|---|---|---|
| OPT 6.7B → with predictors | 66.1 → 66.2 | 30.6 → 30.6 | 50.3 → 49.8 |
| Falcon 7B → relufied → + predictors (1152/256) | 74.62 → 72.52 → 72.35 | 40.05 → 38.23 → 36.35 | 57.77 → 54.17 → 53.16 |
| Persimmon 8B → + predictors (1152/256, thr 0.7) | 67.80 → 66.30 | 34.64 → 34.40 | 50.70 → 52.70 |
| Phi-2 → relufied → + predictors (thr 0.40) | 79.62 → 80.60 → 79.96 | 51.49 → 50.12 → 49.57 | 55.17 → 54.30 → 53.50 |

MMLU: Persimmon holds flat only with the high-rank last-4-layer predictors; **Phi-2 falls 57 → 54.3 from relufication (with distillation) and a further 2.3 points from predictors, landing at ~52**; Llama 2 7B goes **41.8 → 38.96 (ProSparse) → 38.63 (predictors)** on the authors' InstructEval run. Raising the predictor threshold trades a little more accuracy for less I/O (Persimmon 0.5 → 0.7 sweeps in Table 4). Headline: **the predictor itself costs ≤1 point on ReLU models; the *relufication* needed for non-ReLU models costs 1–3 points.**

> [!warning] The ReLU dependence
> The method is built on FFN ReLU sparsity, and **every non-ReLU model in the paper was fine-tuned into a ReLU variant before the method was applied.** OPT 6.7B (native ReLU, 97% FFN sparsity) and Persimmon 8B (native squared ReLU, less sparse, needs threshold 0.7) ran as-is. Falcon 7B (GELU) was the "relufied" model from Mirzadeh et al. 2023 (95% sparsity); Phi-2 (GELU) was relufied by the authors with a distillation loss on RefinedWeb; Llama 2 7B (SwiGLU) was the ProSparse FATReLU-sparsified checkpoint from Song et al. 2024. The paper states outright that for SwiGLU models **"replacing Swish with ReLU within the FFN doesn't ensure high amount of sparsity"** — a *thresholded* FATReLU is required to force >90% sparsity in a gated FFN. So: **no, the 2023 method does not transfer to SiLU/GELU models without modification.** A relufication fine-tune (thresholded for gated FFNs), then predictor training, is a precondition, at a 1–3 point cost; on stock SiLU models the predicted-sparse fraction is too small for windowing to reach the 2–3%-per-token regime the speedups depend on. Everything modern this wiki tracks — Llama 3, Qwen, Mistral, DeepSeek dense — is SiLU/SwiGLU, which is why [[weight-streaming]] in 2026 moved to [[mixture-of-experts]] routing (structural sparsity that needs no predictor) rather than neuron-level activation sparsity; see [[flash-moe-ssd-expert-streaming-woods-2026]]. Appendix E already flags MoE as the natural target.

## Ablations worth keeping

- **Long generation (1000 tokens, OPT, GPU):** no SSD thermal throttling; average flash latency does not grow — it is *highest for the first few tokens* while the window fills. Nucleus sampling (p = 0.9) does not diversify activations enough to hurt.
- **Power (§5.3):** the sparse-from-flash path draws *less instantaneous power* than a dense DRAM-resident model but takes longer, so **total energy per token is higher**. The paper says so plainly and defers quantification; this is the number an [[edge-inference]] deployment would actually care about.
- **Quantization is orthogonal (Appendix F):** 4-bit weights leave the active-neuron pattern unchanged (3.30% vs 3.27% average over 100 sequences), so a 7B model would need **<2 GB of DRAM instead of the baseline's 3.5 GB** on a phone — but 4-bit sparse kernels were not built, so this is a projection, not a result. Compare [[quantization-performance]], where 4-bit decode wins for the same bandwidth reason.
- **Memory–latency knob (§4.3, Fig. 7):** growing the window past the 50% budget monotonically lowers latency; no numbers in the text.

## Limitations

- **ReLU only, with fine-tuning for everything else** — see the callout; this is the constraint that dates the paper.
- **Batch 1 only; decode only.** Prompt processing is listed as future work, and prefill is where per-token weight streaming is at its worst (every neuron fires for *some* token in the prompt).
- **Half-model budget is a showcase, not a stress test.** The machines had ample memory; the "2×" figure is for the sparsest models, and Phi-2 needed 65% of its own size.
- **Baselines are computed, not measured**, and the RTX 4090 box's naive I/O reuses the M1 Max's 2196 ms figure — the Linux SSD's bandwidth is never stated. The hybrid baseline (the fair one) narrows the CPU win to ~3× (derived).
- **Predictor over-prediction (~3×)** means most loaded neurons are wasted; the authors note it as the main open efficiency problem.
- **Energy is worse than dense**, and thermal/power analysis is deferred.
- **No absolute throughput in tokens/s at fp32 on a real memory-starved device;** the smartphone case is argued, not run.

## Related

- [[weight-streaming]] — this is the canonical 2023 source for the neuron-level, predictor-driven form of the idea; the concept page should carry the ReLU caveat above.
- [[flash-moe-ssd-expert-streaming-woods-2026]] — the 2026 successor that streams MoE experts instead of neurons, dodging the activation-function dependence.
- [[mixture-of-experts]] — Appendix E names MoE FFNs as directly amenable; expert routing is the predictor for free.
- [[mlx]] · [[omlx-specprefill-github-2026]] — the Metal kernels came from MLX; oMLX is the 2026 MLX serving stack on the same unified-memory hardware.
- [[arithmetic-intensity]] · [[llm-inference-roofline-yuan-2024]] — decode at batch 1 is bandwidth-bound in DRAM; from flash it is bandwidth-bound by a memory tier ~50× slower, and the whole paper is about raising effective flash bandwidth (chunk size) while cutting bytes (sparsity).
- [[quantization]] · [[quantization-performance]] · [[gguf]] — the orthogonal lever; Appendix F shows sparsity survives 4-bit.
- [[llama-cpp]] · [[llama]] — llama.cpp's `mmap` weight loading is the "naive" baseline in practice; the Llama 2 result here needed a ProSparse checkpoint, not stock weights.
- [[kv-cache]] — kept in the DRAM budget here (~8% of model size at 2K context), with a footnote that it too could be streamed.
- [[speculative-decoding]] — composes with the method (1.4×) mainly by amortising per-token DRAM bookkeeping.
- [[zero-offload-ren-2021]] — the training-side ancestor of tiered offloading; FlexGen is the inference-side comparator the paper argues is still flash-bandwidth-bound.
- [[hardware-accelerator]] · [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] · [[hbm-is-not-all-you-need-wei-2026]] — the memory-tier framing from the other direction (on-chip vs HBM rather than DRAM vs flash).
- [[sparse-attention]] · [[token-pruning]] — the *other* sparsity families in this wiki, both prefill-side and structural; activation sparsity is per-neuron and decode-side.
- [[activation-sparsity]] · [[edge-inference]] — no pages yet.
- [[nvidia]] — the RTX 4090 backend.

## References

- [arXiv:2312.11514](https://arxiv.org/abs/2312.11514) — v3, ACL 2024.
- DejaVu: Liu et al., "Deja Vu: Contextual Sparsity for Efficient LLMs at Inference Time," ICML 2023 — the low-rank activation predictor this paper adapts (from GPU-memory loading to flash loading).
- ReLU Strikes Back: Mirzadeh et al., 2023 — same Apple group; source of the relufied Falcon 7B and the relufication recipe.
- ProSparse: Song et al., 2024 — the FATReLU-sparsified Llama 2 7B used in Appendix C.5.
- FlexGen: Sheng et al., ICML 2023 — offloading to DRAM/flash for throughput; argued here to be bound by flash→DRAM bandwidth in the memory-starved regime.
- MLX: Hannun, Digani, Katharopoulos & Collobert, 2023 — source of the dynamism-friendly Metal kernels.
- Persimmon 8B: Elsen et al., 2023 (Adept) — the squared-ReLU model that needed no relufication.
