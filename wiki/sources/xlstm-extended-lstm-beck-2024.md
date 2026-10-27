---
title: "xLSTM: Extended Long Short-Term Memory"
type: source
created: 2026-10-27
updated: 2026-10-27
sources:
  - xlstm-extended-lstm-beck-2024.md
tags:
  - 2024
  - paper
status: complete
importance: medium
authors:
  - Maximilian Beck
  - Korbinian Pöppel
  - Günter Klambauer
  - Johannes Brandstetter
  - [[sepp-hochreiter]]
introduces:
  - [[xlstm]]
year: 2024
venue: arXiv preprint
arxiv_id: "2405.04517"
---

# xLSTM: Extended Long Short-Term Memory

[arXiv:2405.04517](https://arxiv.org/abs/2405.04517) (v2, 2024-12), from JKU Linz and NXAI, with [[sepp-hochreiter]], the inventor of the [[lstm|LSTM]], as last author. Code: NX-AI/xlstm.

## What it is

The paper asks how far the [[lstm]] goes when scaled with modern techniques. It fixes three LSTM weaknesses: storage decisions can't be revised, storage capacity is small, and recurrent memory mixing makes training impossible to parallelize.

**[[xlstm|xLSTM]] has two cell types:**
- **sLSTM** (scalar memory) uses **exponential gating** with a normalizer and a log-space stabilizer, and keeps **memory mixing** (recurrent matrices). That enables state tracking but is not parallelizable.
- **mLSTM** (matrix memory) uses a d×d covariance update and is **fully parallelizable**, with no memory mixing:

  ```
  C_t = f_t C_{t−1} + i_t v_t k_tᵀ
  n_t = f_t n_{t−1} + i_t k_t
  h_t = o_t ⊙ (C_t q_t / max(|n_tᵀ q_t|, 1))
  ```

  This is linear attention with an exponential input gate and a forget-gate decay ([[linear-attention]]).

**Block ratio:** xLSTM[a:b] means a mLSTM blocks per b sLSTM blocks. For example, [7:1] with 48 blocks is 42 mLSTM + 6 sLSTM.

## Results

- **SlimPajama, 300B tokens, 1.3B parameters:**

  | Model | Val PPL | Avg accuracy | PPL at 16k (trained at 2k) |
  |---|---|---|---|
  | **xLSTM[1:0]** | **8.89** | **58.48** | 9.01 |
  | xLSTM[7:1] | 9.00 | 58.10 | **8.92** |
  | Mamba | 9.14 | 58.41 | 14.00 |
  | Llama | 9.44 | 56.99 | **337.83** |
  | RWKV-4 | 9.83 | 54.78 | 13.75 |

- **Ablation,** from vanilla LSTM to xLSTM: perplexity 2417.86 (LSTM) → 35.46 (residual backbone) → 26.01 (up-projection) → 17.70 (exponential gating) → 13.48 (matrix memory).
- **Formal languages:** sLSTM solves parity (1.0). Llama scores 0.03 and Mamba 0.13; memory mixing is what solves it.
- **Scaling:** the slopes are similar and only the offsets differ. xLSTM leads Mamba by about as much as Mamba leads Llama.

## Limitations

- The sLSTM can't be parallelized.
- The mLSTM kernels are about 4× slower than FlashAttention or Mamba's scan.
- The d×d memory is compute-heavy.
- Results are sensitive to forget-gate initialization.
- The paper states no venue or licence.

## Related

[[xlstm]] · [[lstm]] · [[sepp-hochreiter]] · [[state-space-model]] · [[linear-attention]] · [[long-short-term-memory-hochreiter-1997]]
