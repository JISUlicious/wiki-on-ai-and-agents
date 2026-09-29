---
title: "NeuroPrefetcher: Storage-Aware Sparse LLM Inference via Delta Prefetching"
type: source
created: 2026-09-10
updated: 2026-09-10
sources: []
arxiv_id: "2608.22643"
authors:
  - Nobel Dhar
  - Md Romyull Islam
  - Xuechen Zhang
  - Gongjin Sun
  - Sahidul Islam
  - Bobin Deng
  - Kun Suo
year: 2026
venue: ACM ICPP 2026
status: complete
importance: medium
tags:
  - 2026
  - paper
---

# NeuroPrefetcher: Storage-Aware Sparse LLM Inference via Delta Prefetching

**Nobel Dhar, Md Romyull Islam, Xuechen Zhang, Gongjin Sun, Sahidul Islam, Bobin Deng, Kun Suo** (Kennesaw State University, Marietta GA; Sun at Memory Solutions Lab, Samsung Semiconductor, San Jose; Suo corresponding) — [arXiv:2608.22643](https://arxiv.org/abs/2608.22643), cs.DC / cs.LG, submitted 23 Aug 2026, v1, 10 pages / 10 figures. **Venue: ACM ICPP 2026** (55th International Conference on Parallel Processing, Singapore, 28 Sep–1 Oct 2026), DOI 10.1145/3832810.3832862, CC BY 4.0. NSF-funded (SHF-2210744, OAC-2602212, ERI-2501978). **Code released:** [github.com/nobeldhar/NeuroPrefetcher](https://github.com/nobeldhar/NeuroPrefetcher) — "code, training pipeline, and measured data" per the arXiv comments field (the URL does not appear in the paper body).

> [!important] What this system is, in one paragraph
> A storage-backed decode runtime for the **model-exceeds-memory** regime: FP16 Mistral-7B (13.49 GiB) on a board with 9.0–13.2 GiB free, where the weights *cannot* be made resident. It keeps the first *d* MLP layers dense, runs the remaining MLP layers at 31–62% **enforced** activation sparsity, and after every token fetches from NVMe only the ~15–18% of active neuron rows that were not already resident for the previous token. It is the direct descendant of Apple's [[llm-in-a-flash-alizadeh-2023]] — but on **NVIDIA hardware, not [[apple-silicon|Apple Silicon]]** (Jetson AGX Orin, CUDA, Linux `io_uring`), on **SwiGLU models rather than ReLU-fied ones**, and with a **one-shot lookahead predictor** instead of per-layer prediction. The 7.9–12.0× headline is real but is measured against a thrashing `mmap` baseline and at operating points where the sparse model has lost **7–32% of its downstream accuracy** (see § Model quality). Read the speedup and the "92–96% of dense accuracy" claim as coming from *different* configurations.

## Problem

The paper's framing is a **model–memory gap**: it claims model size grows ~10× per generation while edge DRAM grows ~2×, so increasingly the model "does not merely run slowly; it no longer fits". Quantization ([[gptq-frantar-2022]], [[awq-lin-2023]]), small models (Phi-3, MobileLLM) and offloading (FlexGen, [[llama-cpp]]) all *move the fit point*; none is designed for the case where the deployed model stays larger than resident memory for the whole run, so that **storage is an active weight tier on the critical path**. That is the regime targeted, and the authors note it arises not only when a model outgrows a device but whenever the LLM shares the board with perception pipelines or other workloads.

What actually happens in that regime today is measured directly (Fig. 1, Mistral-7B on Jetson AGX Orin at `mem=14G`): [[llama-cpp]] falls back to memory-mapped weights and OS demand paging, generating **11,453 major page faults per second**, with the CPU **77% in iowait** and the GPU at **3% utilisation** while the NVMe delivers **1,638 MB/s** — the system is bound by reactive weight movement, not arithmetic. This is the storage-tier analogue of the decode-side [[arithmetic-intensity]] problem in [[llm-inference-roofline-yuan-2024]]: bytes moved per token, not FLOPs, set the speed.

The enabling observation is **temporal locality of MLP activity**: across consecutive decode tokens, **82–85% of active MLP neurons persist**, so most of the sparse weights a token needs are already on the GPU and only the *delta* — newly active rows — must cross the NVMe boundary. Exploiting it requires two things existing systems lack at once: (a) per-layer tracking of which rows are resident, and (b) knowing the next token's active set *early enough* to fetch the delta before execution reaches the layer. The paper's account of prior work: PowerInfer targets the GPU→CPU-DRAM tier (useless on unified memory, where host offload creates no capacity); DejaVu's per-layer predictors reveal a layer's set only when execution reaches it, "too late for storage-backed lookahead"; **[[llm-in-a-flash-alizadeh-2023]] "retains recently activated neurons with a backward-looking window"**, and Neuralink co-locates co-activated neurons in flash — "but none maintains a per-layer resident active set updated by token-to-token set difference" and "none provides a lookahead mechanism that predicts the next sparse working set and transfers only the incremental rows". LLM in a Flash was **never benchmarked head-to-head**: it appears in the capability study under "no ARM/JetPack release".

## Mechanism

Three pieces, applied in order.

**1. Offline sparse-centroid preparation.** Mistral-7B and Llama-3-8B use SiLU-gated SwiGLU MLPs, which produce no exact zeros. Following the authors' own prior work (Dhar et al. 2024a, IPCCC), sparsity is *enforced* with a **per-layer percentile magnitude threshold** on the intermediate activation: anything below the α-th percentile of |A| is zeroed, no retraining. Attention is left dense ("changing attention tends to have a larger effect on model quality"). The thresholded masks are then **clustered** (Dhar et al. 2025, Euro-Par) into a vocabulary of **K = 512 binary centroids per projection per layer**, each pruned back to the target sparsity, so runtime prediction becomes one classification per layer instead of 14,336 binary decisions. This is the source of nearly all the quality loss (below).

**2. Shared activation predictor.** Runs once per token after layer 0 and emits **2 × 31 = 62 centroid IDs**: one gate and one up centroid for each of layers 1–31. Inputs (Table 1): the token embedding (4096-d), pre-attention LayerNorm output (4096-d), a 128-d attention-head sketch, the post-attention residual norm (scalar), and the layer-0 gate activation's nearest centroid ID. Architecture: four projection blocks (Linear→LayerNorm→SiLU→dropout) to T = 512 each, concatenated with a centroid embedding to 8T = 4096-d; a **four-block residual trunk** (8T→4T→8T); then **62 independent linear heads** over K = 512. Trained with cross-entropy on gate + up IDs, AdamW, batch 2,048 tokens/GPU, grad-clip 1.0, on **8× A100 80 GB**, from **WikiText-2 activations**. Fixed at runtime.

| Component | Params | FP16 size |
|---|---|---|
| Input projections | 4.3M | 8.6 MB |
| Centroid embedding | 5.2M | 10.4 MB |
| Residual trunk | 67.1M | 134.2 MB |
| Output heads | 130.2M | 260.4 MB |
| **Total** | **206.8M** | **413.6 MB** |

206.8M / 7.24B = **2.86% of the base model** (checks). The heads dominate and scale with layer count; nothing scales with the MLP intermediate dimension. The predictor is always resident and "all reported speedups are net of this cost". Its accuracy is high: **~99% of layer decisions pick the correct centroid** across both models and all sparsities — so quality loss comes from the *coarseness of 512 centroids*, not from mis-prediction.

**3. Predictive delta prefetching + I/O path.** Each sparse layer *l* keeps a fixed-capacity GPU buffer of K_max rows and a slot map S⁽ˡ⁾ ∈ ℤ^14336 (slot index or −1). The active set A_t⁽ˡ⁾ is the **intersection** of the gate and up centroid masks (sufficient for SwiGLU: a neuron contributes only if both terms are active); the same set indexes gate, up and downᵀ rows — the down projection gets no separate mask, its rows just ride on the intermediate active set. Per token:

- D_in = A_t \ A_{t−1} (fetch from NVMe); D_out = A_{t−1} \ A_t (free slots). Because |A| = K_max is fixed, incoming ≈ outgoing and freed slots are reused with no compaction; row order in the buffer is irrelevant.
- Weights are repacked offline into a **neuron-centric layer-major file**: one record co-locates the gate, up and (pre-transposed) down rows of one intermediate neuron, so one read returns all three projections and consecutive neuron IDs are contiguous bytes. (The paper says 24.5 KiB per record; 3 × 4096 × FP16 = 24,576 B = **24.0 KiB**, and the paper's own 129 MiB / 231 MiB per-layer buffer sizes use 24 KiB.)
- Incoming IDs are sorted and coalesced into ranges, submitted via **Linux `io_uring` with `O_DIRECT`, queue depth 256**, batched across all sparse layers into one token-level submission, on a background thread **while the main thread executes the dense prefix on the GPU** — this overlap is the whole point of predicting all 31 layers at once.
- Reads land in page-aligned host staging memory registered with `cudaHostRegisterMapped`; a fused `launch_deinterleave_scatter` kernel on its own CUDA stream writes each record's three rows into their slots. The staging hop is **required for correctness on Jetson**: direct I/O on the unified-memory platform can bypass CPU cache coherence and expose stale data to CUDA. It is the only Jetson-specific component.
- **Memory-adaptive dense/sparse split.** An automated sweep picks the largest dense prefix *d* that fits; as memory tightens the runtime trades dense layers for sparse ones rather than falling back to whole-layer NVMe reads. Always resident regardless: embeddings, LM head, all attention projections, norms, predictor, centroid table. First token of a sequence loads each layer's full predicted set; every later token loads only the delta.

## Hardware — and it is not Apple Silicon

| | |
|---|---|
| Board | **NVIDIA Jetson AGX Orin** developer kit |
| Memory | **32 GiB unified LPDDR5**, shared by CPU and GPU (single physical pool) |
| CPU | 12-core Arm Cortex-A78AE |
| GPU | Ampere, **2048 CUDA cores**, SM 8.7 |
| Storage | **Samsung 990 PRO NVMe** on M.2, **rated 7.4 GB/s sequential read** |
| Achieved NVMe read (Fig. 1, `mem=14G`) | **NeuroPrefetcher 1,295 MB/s; llama.cpp 1,638 MB/s** — 17.5% and 22% of the rating |
| OS / stack | Linux (JetPack 6), CUDA, `io_uring`/`O_DIRECT` |
| Offline training | 8× NVIDIA A100 80 GB server |

To say it plainly for the concept page that contrasts this with Mac implementations: **there is no [[apple|Apple]] hardware, no Metal, no [[mlx]] anywhere in this paper.** The 2023 predecessor ran on an Apple M1 Max; this one runs on [[nvidia]]'s unified-memory edge SoC, and its one platform-specific component is a CUDA cache-coherence workaround. The relevant common thread with Apple Silicon is only that both are *unified-memory* machines, which is why host-offload systems (PowerInfer, vLLM, TensorRT-LLM, MLC-LLM, Candle, DejaVu) fail here: there is no separate DRAM tier to spill into.

*Derived:* the achieved ~1.3 GB/s is not the drive's limit. At 24 KiB per record that is ~53K records/s; a 990 PRO is rated for ~1.4M random 4K IOPS. The paper's claim that QD 256 "saturates the drive for this access pattern" should be read as "for coalesced 24 KiB reads submitted from an A78AE through `io_uring` with a staging copy" — its own future-work list (deeper prefetch pipelines, stronger coalescing, quantized sparse-row formats) concedes the headroom. The one direct comparison available says NeuroPrefetcher wins **while pulling *less* bandwidth than llama.cpp** (1,295 vs 1,638 MB/s): the win is fewer bytes, not faster bytes.

## Memory budgets

Total system memory is capped with the Linux kernel `mem=` boot parameter; after OS, display server and CUDA runtime, the process sees (Table 3):

| `mem=` | CUDA total (GiB) | **CUDA free (GiB)** |
|---|---|---|
| 11G | 10.69 | **9.02** |
| 12G | 11.67 | **9.99** |
| 13G | 12.65 | **11.02** |
| 14G | 13.64 | **11.94** |
| 15G | 14.62 | **13.19** |
| 16G | 15.60 | **13.88** |
| 17G | 16.56 | **14.81** |

Model: **Mistral-7B-v0.1, FP16, 7.24B params, 13.49 GiB** (7.24e9 × 2 B / 2³⁰ = 13.49 — checks), 32 layers, hidden 4096, intermediate 14,336, SwiGLU. So 9.02–13.19 GiB is model-exceeds-memory; 13.88 and 14.81 GiB are past the crossover. **Llama-3-8B appears only in the quality study**, never in throughput. Three sparsity budgets (K_max of 14,336 intermediate neurons):

| K_max | active | sparsity | resident buffer / sparse layer |
|---|---|---|---|
| 5,500 | 38.4% | **62%** | 129 MiB |
| 7,168 | 50.0% | **50%** | 168 MiB (derived) |
| 9,865 | 68.8% | **31%** | 231 MiB |

Throughput is sustained decode over 2,048 generated tokens on WikiText-2 test data. Quality is WikiText-2 perplexity and 10-shot normalised HellaSwag.

## Measured results

### Capability study (Table 4, `mem=14G`, 11.9 GiB CUDA-available)

Twenty systems, one board, one condition — model larger than resident memory.

| Outcome | Systems |
|---|---|
| **Operational (4)** | **NeuroPrefetcher 3.22 tok/s**, llama.cpp FP16 **0.34**, FlexGen all-disk 0.25 (on OPT-6.7B), Ollama 0.07 |
| Allocation failure (6) | HF Transformers, HF Transformers (disk), ZeRO-Inference, CATS, TEAL, SGLang |
| Unified-memory mismatch (6) | vLLM, TensorRT-LLM, MLC-LLM, PowerInfer, Candle, DejaVu |
| No ARM/JetPack release (4) | **LLM in a Flash**, LMDeploy, TGI, HF Optimum + ONNX GPU |

**3.22 / 0.34 = 9.5× — checks.** llama.cpp "survives through operating-system demand paging of memory-mapped weights rather than an explicit sparse execution policy"; Ollama inherits that core (and is inexplicably 5× slower than it); FlexGen finds no feasible GPU/host/disk placement on unified memory, so its all-disk number on a *different model* is "for completeness rather than as a tuned baseline". The alloc-failure class dies once free memory drops below ~1 GiB and the Jetson kernel cannot satisfy contiguous requests. So the only real competitor in the table is `mmap`.

### End-to-end throughput (Fig. 7 + Fig. 9, 62% sparsity, Mistral-7B FP16)

| CUDA free (GiB) | calibrated *d* (Fig. 9) | llama.cpp (tok/s) | NeuroPrefetcher (tok/s) | speedup (Fig. 7 label) | latency / token | NVMe I/O share |
|---|---|---|---|---|---|---|
| 9.0 | 2 | ≈0.15 | **1.7** | **11.4×** | **593 ms** | 87% |
| 10.0 | 6 | ≈0.17 | ≈2.0 | **12.0×** | 522 ms | 87% |
| 11.0 | 10 | ≈0.30 | ≈2.4 | **7.9×** | 423 ms | 86% |
| 11.9 | 16 | **0.34** | **3.22** | **9.5×** | 364 ms | 87% |
| 13.2 | 18 | ≈0.33 | **3.7** | 11.3× | 285 ms | 85% |
| 13.9 | 26 | **5.9** | ≈7.1 | **1.2×** | 136 ms | 80% |
| 14.8 | 28 | **7.7** | ≈8.5 | **1.1×** | **113 ms** | 80% |

Bold values are stated in the text; ≈ values are read off Fig. 7. The **7.9–12.0×** range spans 9.0–13.2 GiB and checks (the minimum is at 11.0 GiB, not at the tightest budget, because llama.cpp is marginally less bad there). Past the crossover llama.cpp's page-fault traffic "largely disappears" and the advantage collapses to 1.1–1.2× — the authors say this "matches the design target". At 50% and 31% sparsity the system is much slower (≈0.8→2.0 and ≈0.55→1.3 tok/s across 9.0–13.2 GiB): the larger resident buffers leave less room for dense layers, and at 9.0 GiB even *d* = 2 cannot fit them, so some layers fall back to whole-layer reads "pulling their throughput closer to llama.cpp".

**Latency is storage.** NVMe I/O is 80–87% of per-token time at every budget; GPU compute and the predictor are both small slices. From 9.0 to 14.8 GiB the dense prefix grows from 2 to 28 layers and latency falls from 593 to 113 ms.

### Dense/sparse sweep (Fig. 6, 11.9 GiB) — three regimes

Rising (each added dense layer removes a layer's worth of delta I/O *and* widens the compute window the prefetch can hide behind), peak at *d\**, then a **>3× cliff** when the prefix starves the suffix of buffer space and layers fall back to whole-layer reads. At 62% sparsity: 1.23 tok/s at *d* = 2 → **1.60 at *d\** = 14** → 0.98 at *d* = 16 → 0.19 at *d* = 32. *d\** = 8 at 50% and *d\** = 2 at 31%.

> [!warning] Fig. 6 and Fig. 7 disagree at the same budget
> The sweep at 11.9 GiB peaks at **1.60 tok/s** (*d* = 14), yet Table 4 and Fig. 7 report **3.22 tok/s** at the same 11.9 GiB, and Fig. 9 labels that budget ***d* = 16** with 364 ms/token (= 2.75 tok/s) — the very *d* that § 4.3 says has already fallen off the cliff to 0.98 tok/s. Every other budget reconciles Fig. 7 against Fig. 9 to within ~5% (e.g. 593 ms ↔ 1.7 tok/s); only the headline point does not, and the paper does not explain the ~2× gap between the sweep and the end-to-end number. The direction of the result is not in doubt (all three numbers are 3–9× above llama.cpp), but **3.22 tok/s should be quoted as "reported", not as reproduced by the paper's own sweep.**

### Delta reuse and I/O volume (Fig. 8, 62% sparsity)

**82–85% of sparse-suffix rows are reused from the previous token at every budget** (83/82/83/82/83/85/85% from 9.0 to 14.8 GiB), so NVMe traffic is confined to the 15–18% incoming rows. Per-token NVMe reads go from **103 MiB at 14.8 GiB to just under 1 GiB at 9.0 GiB** — across a ~6 GiB swing in available memory — against a 13.49 GiB full footprint. *Derived:* ~37K incoming rows × 24 KiB ≈ 0.9 GiB at 9.0 GiB, consistent. (Fig. 8a's total row counts — ≈220K/token at *d* = 2 — run ~35% above (32 − *d*) × K_max = 165K, so the resident set per layer seems larger than the nominal K_max; not explained.)

The system-level consequence (Fig. 1, right panels): **0.21 major faults/s, 4% iowait, 30% GPU utilisation** — reactive paging is gone; the GPU is still idle 70% of the time waiting on storage.

## Model quality — what the speedup costs

This is the section the abstract elides. Quality is measured for **Mistral-7B and Llama-3-8B** at all three sparsities and **only at *d* ∈ {4, 8, 14, 20}** (Fig. 10). Values below are read from a log-scale plot and are approximate; the text anchors are in bold.

**WikiText-2 perplexity (lower is better; dense ≈ 5.3 Mistral, ≈ 6.1 Llama-3):**

| *d* | Mistral 31% | 50% | 62% | Llama-3 31% | 50% | 62% |
|---|---|---|---|---|---|---|
| 4 | ≈6.5 | ≈9 | ≈16 | ≈11 | ≈22 | ≈48 |
| 8 | ≈5.7 | ≈7 | ≈11 | ≈8.5 | ≈15 | ≈28 |
| 14 | ≈5.3 | ≈6 | ≈8 | ≈7.5 | ≈11 | ≈16 |
| 20 | **near dense** | ≈5.5 | **< 7** | ≈7 | ≈9 | ≈11.5 |

**HellaSwag accuracy retention (% of each model's dense score):**

| *d* | Mistral 31% | 50% | 62% | Llama-3 31% | 50% | 62% |
|---|---|---|---|---|---|---|
| 4 | ≈91 | ≈78 | **≈68** | ≈82 | ≈70 | **≈66** |
| 8 | ≈96 | ≈87 | ≈78 | ≈90 | ≈82 | ≈74 |
| 14 | ≈96 | ≈95 | ≈88 | ≈95 | ≈90 | ≈86 |
| 20 | **95–96** (all sparsities) | | | **92–93** (all sparsities) | | |

The paper's own summary: Mistral is "more tolerant"; Llama-3-8B is "more sensitive, especially at high sparsity and small dense prefixes, with its largest perplexity increase at *d* = 4 and 62%"; increasing *d* is "a direct quality control knob". Centroids and predictor are built from WikiText-2 activations, so the HellaSwag numbers are also a transfer result — no per-task retraining.

> [!warning] The headline speedup and the headline accuracy come from different operating points
> The contributions list says the predictor "preserves 92–96% of dense accuracy". That figure is **at *d* = 20**. But the runtime's calibrated split in the model-exceeds-memory range is *d* = 2, 6, 10, 16, 18 (Fig. 9) — it only reaches *d* ≥ 20 at 13.9 GiB and above, where the speedup is 1.1–1.2×. Joining Fig. 9's *d* to Fig. 10b at 62% sparsity (the only sparsity that delivers the 7.9–12.0×), on Mistral-7B:
>
> | CUDA free | *d* | speedup | HellaSwag retention at that *d* | WikiText-2 ppl |
> |---|---|---|---|---|
> | 9.0 GiB | 2 | 11.4× | **< 68%** (*d* = 2 never measured; *d* = 4 is ≈68%) | > 16 (3× dense) |
> | 10.0 GiB | 6 | 12.0× | ≈73% (interpolated) | ≈13 |
> | 11.0 GiB | 10 | 7.9× | ≈81% (interpolated) | ≈10 |
> | 11.9 GiB | 14–16 | 9.5× | **≈88–90%** | **≈8** (+50%) |
> | 13.2 GiB | 18 | 11.3× | ≈93% | ≈7 |
> | 13.9 GiB | 26 | 1.2× | ≥ 95% | < 6.5 |
>
> So the speedup is **not free**. At the flagship 9.5× point the model has given up roughly **10–12% of its HellaSwag accuracy and ~50% higher perplexity**; at the 11.4–12.0× points it has given up roughly **a quarter to a third** of its accuracy and runs at 2.5–3× dense perplexity — and Llama-3-8B is worse at every cell. The baseline it is beating, meanwhile, is running the *exact* FP16 model. The paper never puts throughput and quality in the same table; the reader has to do this join. The honest framing is a three-way trade among memory budget, throughput and quality, which the paper does state in § 4.7 — just not next to the 9.5×.

Two further quality notes. Nearly all loss is from **clustering to 512 centroids** (the predictor is ~99% correct), so K is the lever, and a larger K makes the 62-way classification harder — the paper sweeps only K = 512. And only *SiLU-gated* models are tested; the "GeLU or SiLU" motivation is general, but there is no GELU-model result.

## Delta prefetching vs. windowing (the 2023 method)

What the paper says the 2023 limitation was, in its words: LLM in a Flash "retains recently activated neurons with a **backward-looking window**"; it improves storage-backed sparse access but does not "maintain a per-layer resident active set updated by token-to-token set difference" nor provide "a lookahead mechanism that predicts the next sparse working set and transfers only the incremental rows".

How the two actually differ, reading both papers:

| | [[llm-in-a-flash-alizadeh-2023]] windowing | NeuroPrefetcher delta prefetching |
|---|---|---|
| Residency policy | Keep every neuron active in the last *k* tokens (a sliding-window superset); evict on falling out of the window | Keep exactly the previous token's predicted set, fixed K_max slots, slot map; evict D_out, load D_in |
| What is fetched | Neurons in the current set not already in the window — *also* incremental | Only D_in = A_t \ A_{t−1} |
| When the active set is known | Per layer, from a per-layer low-rank predictor at that layer's input (DejaVu-style) — just in time | Once, after layer 0, for all 31 downstream layers — so I/O for the whole suffix overlaps the dense-prefix compute |
| Sparsity source | ReLU-fied FFNs with exact zeros (~97% sparse) | Magnitude-thresholded SwiGLU at 31–62%, clustered to 512 centroids |
| Storage layout | Row–column bundling (up column + down row) for a two-matrix ReLU FFN | Neuron-centric record of gate + up + downᵀ rows for a three-matrix SwiGLU FFN |
| Memory-adaptive split | — | Dense prefix *d* swept per budget; trades dense layers for sparse ones |
| Platform | Apple M1 Max (Metal) and Linux + RTX 4090 | NVIDIA Jetson AGX Orin, CUDA, `io_uring` |

So "fetch only the delta" is **not itself new** — windowing already loaded only the not-yet-resident neurons. What is new is the **lookahead**: because the whole suffix's I/O plan exists after layer 0, the prefetch runs concurrently with useful GPU work rather than stalling at each layer, and the fixed-capacity buffer makes the per-token I/O a stable ~15–18% of K_max rather than a window-dependent quantity. The price of one-shot prediction is coarseness (centroids instead of per-neuron masks), which is where the quality loss above comes from; the 2023 per-layer predictors could afford per-neuron output because they ran with the layer's own input in hand. The other genuine extension is dropping the ReLU requirement — at the cost of sparsity falling from ~97% to ≤62%, which is why this system still moves ~0.1–1 GiB per token where the 2023 one moved far less.

## Is the llama.cpp baseline fair?

What it is doing: FP16 GGUF weights `mmap`ed, no explicit offload policy. With 13.49 GiB of weights and 11.9 GiB free, each token's sequential pass over the layers evicts pages needed by the next pass — classic scan-larger-than-cache thrash: 11,453 major faults/s at 1,638 MB/s over a 2.9 s token means roughly **4.8 GB paged in per token**, three times the 1.6 GiB shortfall. NeuroPrefetcher at the same budget reads ~0.4–0.5 GiB per token. The 9.5× is essentially that bytes ratio, and it is the baseline's *behaviour* the paper indicts — reactive paging with no knowledge of what the next token needs.

Fair in the narrow sense: it is what actually happens if you hand llama.cpp a model that does not fit, the paper profiles it honestly, and nothing else in the 20-system table even runs. Unfair in three broader senses:

1. **Nobody deploys FP16 Mistral-7B on a 12 GiB budget.** The practitioner's move is Q4_K_M (~4.1 GiB), which fits with room to spare and would decode at an order of magnitude above 3.22 tok/s on this board ([[quantization-performance]], [[which-quantization-kurt-2026]]). The paper's answer — quantization is orthogonal and NeuroPrefetcher targets budgets where *even the quantized model* does not fit — is reasonable but **never demonstrated**: there is no quantized experiment at all, and "quantized sparse-row formats" is future work. A 4-bit model on a 3 GiB budget is the experiment that would settle it.
2. **Quality is not matched.** The baseline runs the exact model; the system runs a 62%-sparse one at 68–90% retention. A quality-matched baseline would be a smaller or quantized dense model that fits — which is exactly the comparison the paper declines.
3. **The dense baseline is unscheduled.** An explicitly streamed dense model — read each layer from NVMe in order at the same 1.3–1.6 GB/s — would need 13.49 GiB/token ≈ 9 s/token ≈ 0.11 tok/s, so `mmap` is in fact better than naive [[weight-streaming]] here. The interesting missing baseline is *predicted-set reload without delta* (fetch the full K_max set each token): that isolates how much of the win is sparsity and how much is the 82–85% reuse. Fig. 8 gives the ingredients (reuse fraction) but no such run.

Net: the ranking is credible and the mechanism is real; the *magnitude* is against a baseline nobody would ship, at a quality nobody would accept for the tighter budgets.

## Claims that did not check out, or need qualification

- **Fig. 6 (1.60 tok/s peak) vs Fig. 7 / Table 4 (3.22 tok/s) at 11.9 GiB**, and Fig. 9's *d* = 16 vs § 4.3's *d\** = 14 — unreconciled (see callout).
- **"24.5 KiB record"** — arithmetic gives 24.0 KiB; the paper's own buffer sizes agree with 24 KiB. Typo-level.
- **"92–96% of dense accuracy"** in the contributions — true only at *d* = 20, which no model-exceeds-memory budget uses.
- ***d* = 2 is never quality-measured**, though it is the operating point for the 11.4× result at 9.0 GiB.
- **Fig. 8a row totals** exceed (32 − *d*) × K_max by ~35% — resident set larger than nominal K_max, unexplained.
- **Ollama at 0.07 tok/s** despite wrapping llama.cpp (0.34) — no explanation offered.
- Everything else verified: 82–85%, 2.86% / 206.8M / 414 MB, 7.24B / 13.49 GiB, 9.02–14.81 GiB, K_max 5,500 / 7,168 / 9,865, 11,453 faults/s → 0.21, 77% → 4% iowait, 3% → 30% GPU, 80–87% I/O share, 593 / 113 ms, 103 MiB / ~1 GiB, 20 systems = 4 + 6 + 6 + 4, 9.5× and 7.9–12.0×.

## Limitations

- **Quality cost is real and largest exactly where the speedup is largest** (see § Model quality); Llama-3-8B degrades more than Mistral-7B at every cell.
- **Batch 1, single model family for throughput** (Mistral-7B only), FP16 only, no quantization experiments, one centroid count (K = 512), one predictor size.
- **NVMe I/O is still 80–87% of latency** and the drive runs at ~1.3 GB/s of a 7.4 GB/s rating — the I/O path, not the idea, is the bottleneck, and the paper says so.
- **Storage wear** is not discussed; ~0.1–1 GiB of reads per token at 1.7–3.7 tok/s is 0.4–1.7 GB/s of sustained NVMe reads for as long as the model generates.
- **First-token cost** (loading the full predicted set for every sparse layer) is not reported, and there is no prefill/TTFT measurement at all — decode only.
- **Cache-coherence staging is a Jetson artefact**; the paper claims everything else is platform-independent but demonstrates it on one board.
- **Predictor training needs an 8× A100 server and a per-model offline pipeline** (thresholds, clustering, predictor), so "no retraining" applies to the base model, not to deployment effort.
- **Only SiLU-gated models tested** despite the GeLU/SiLU framing.

## Related

- [[llm-in-a-flash-alizadeh-2023]] — the 2023 Apple paper this extends; windowing vs delta prefetching is tabulated above. That one is the Mac story; this one is the NVIDIA story.
- [[weight-streaming]] — the general pattern of serving weights from storage during decode; this source is the sparse, predicted, delta-only variant and shows naive streaming would lose to `mmap`.
- [[mlx]] — mentioned only to say plainly it is *not* involved here; the relevant unified-memory contrast is Jetson vs Apple Silicon, and this paper never touches the latter.
- [[llama-cpp]] — the baseline and the only other engine that ran; its behaviour here is `mmap` demand paging, not a deliberate offload policy. See [[gguf]] for the format.
- [[arithmetic-intensity]] · [[llm-inference-roofline-yuan-2024]] — the storage-tier version of the memory-bound-decode argument: bytes per token, now from NVMe rather than HBM.
- [[quantization-performance]] · [[which-quantization-kurt-2026]] · [[gptq-frantar-2022]] · [[awq-lin-2023]] — the obvious alternative the paper calls orthogonal but never tests against.
- [[mixture-of-experts]] — the other way to make per-token weight traffic sparse; MoE routing is *trained* and exact, whereas this is post-hoc thresholding with a quality cost. Expert prefetching from storage is the natural next application of the same delta idea — see [[flash-moe-ssd-expert-streaming-woods-2026]] (the Mac/MLX side) and [[ssd-offloading-moe-weights-considered-harmful-kyung-2025]] (the energy objection to any SSD-fed decode, which applies to this paper's 0.4–1.7 GB/s of sustained reads too).
- [[token-pruning]] · [[sparse-attention]] — sparsity along the sequence axis; this is sparsity along the neuron axis, and only in the MLP.
- [[kv-cache]] — left untouched here (attention is dense and resident); the paper cites ShadowKV/PRESERVE as the KV-side prefetch analogues.
- [[nvidia]] · [[hardware-accelerator]] — Jetson AGX Orin as the platform; contrast with the FPGA sources [[persistent-state-dataflow-accelerator-linear-attention-decode-gupta-2026]] and [[fast-prefill-fpga-sparse-attention-jayanth-2026]], which attack the same memory-bound decode from the silicon side.
- [[hbm-is-not-all-you-need-wei-2026]] · [[prefill-decode-aware-evaluation-ai-accelerators-usami-2026]] — the memory-tier framing.
- [[zero-offload-ren-2021]] — the training-side offload lineage (ZeRO-Inference fails on this board with an allocation error).
- [[llama]] — Llama-3-8B, the quality-study second model.
- [[activation-sparsity]] · [[edge-inference]] — this source was a seed for both (DejaVu, ReLU Strikes Back, CATS, TEAL, TurboSparse, PowerInfer, SparseInfer are all cited here).
- Not [[memory-management]] — in this wiki that slug means *agent* memory systems (MemGPT, Mem0), not OS paging; linked only to disambiguate.

## References

- [arXiv:2608.22643](https://arxiv.org/abs/2608.22643) · DOI [10.1145/3832810.3832862](https://doi.org/10.1145/3832810.3832862) · code [github.com/nobeldhar/NeuroPrefetcher](https://github.com/nobeldhar/NeuroPrefetcher)
- LLM in a flash: Alizadeh et al., [arXiv:2312.11514](https://arxiv.org/abs/2312.11514) — windowing + row-column bundling; the direct predecessor.
- ReLU Strikes Back: Mirzadeh et al., [arXiv:2310.04564](https://arxiv.org/abs/2310.04564) — the ReLU-sparsity result the 2023 paper relied on and this one drops.
- Deja Vu: Liu et al., ICML 2023 — per-layer contextual-sparsity predictors; the "too late for lookahead" contrast.
- PowerInfer: Song et al., SOSP 2024 — hot/cold neuron split across GPU/CPU; fails on unified memory here.
- Neuralink: Wang et al., ASPLOS 2025 — co-activation-aware flash layout for smartphones.
- CATS: Lee et al., [arXiv:2404.08763](https://arxiv.org/abs/2404.08763); TEAL: Zhao et al., ICLR 2025; TurboSparse: Song et al., [arXiv:2406.05955](https://arxiv.org/abs/2406.05955) — thresholded/learned sparsity for SwiGLU models.
- Dhar et al. 2024a (IEEE IPCCC) — the percentile-threshold sparsity method; Dhar et al. 2025 (Euro-Par) — the activation-pattern clustering. Both are the authors' own prior work.
- FlexGen: Sheng et al., ICML 2023 — offloading baseline.
- PRESERVE: Yüzügüler et al., [arXiv:2501.08192](https://arxiv.org/abs/2501.08192); ShadowKV: Sun et al., ICML 2025 — prefetch/KV analogues.
- Mistral 7B: Jiang et al., [arXiv:2310.06825](https://arxiv.org/abs/2310.06825); Llama 3: Grattafiori et al., [arXiv:2407.21783](https://arxiv.org/abs/2407.21783).
