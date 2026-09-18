# Experiments — Source of Truth for the KD Paper

This file separates **verified implementation facts** from **final publication choices/results**. Codex must not turn provisional items into final paper claims.

## 0. Implementation source

Implementation repository:

- `https://github.com/pesqSC/DeepSC.git`
- branch: `BPE`

Primary BPE KD training path currently inspected:

- `train_multi_vocab_one_student.py`

Supporting files:

- `student.py`
- `teacher.py`
- `models/transceiver.py`
- `models/tx_model.py`
- `models/rx_model.py`
- `utils/kd_utils.py`
- `utils/train_utils.py`
- `dataset_multilingual.py`
- `utils/bpe_utils.py`

Important: older/alternate scripts (`train_student.py`, `R_tr_kd.py`, `kd_training_loss.py`) contain different KD configurations. Do not merge their hyperparameters into the BPE configuration unless explicitly comparing variants.

## 1. Teacher — current BPE candidate

Role: high-capacity DeepSC text semantic communication model. During Student KD training, the transmitter and Teacher receiver are frozen.

Components:

- semantic encoder: `encoder`
- channel encoder: `channel_encoder`
- channel decoder: `channel_decoder`
- semantic decoder: `decoder`
- output projection/head: `dense`

Current BPE script configuration:

- Transformer layers: **8**
- model dimension: **128**
- attention heads: **16**
- feed-forward dimension: **512**
- dropout: **0.1**
- maximum sequence length: **67**

Current code-default Teacher checkpoint directory:

`./checkpoints/deepsc-Rayleigh/multilingual_bpe/2026-09-15`

with:

- `encoder_26.pth`
- `decoder_26.pth`

This checkpoint path is a verified code default, but it is **not yet designated as the final publication Teacher checkpoint**.

Final Teacher parameter count: **TBD — compute from the exact BPE vocabulary/checkpoint used for the publication run**.

## 2. Student — current BPE candidate

Role: receiver-only Student. It receives the noisy latent representation from the frozen/shared Teacher transmitter and contains:

- `channel_decoder`
- semantic `decoder`
- output `dense` head

Current BPE Student configuration:

- Transformer decoder layers: **4**
- model dimension: **128**
- attention heads: **8**
- feed-forward dimension: **512**
- dropout: **0.1**
- maximum sequence length: **67**

Compared with the current BPE Teacher configuration, the Student reduces decoder depth from 8 to 4 layers and attention heads from 16 to 8 while keeping `d_model=128` and `dff=512`.

Final Student parameter count: **TBD**

Final compression ratio relative to the Teacher receiver: **TBD**

Student-without-KD baseline available: **TBD — must be trained/evaluated explicitly if used in the paper**.

## 3. Knowledge Distillation — current BPE candidate

The current BPE path freezes both the transmitter and Teacher receiver during Student training.

The implemented training objective combines three terms:

`L = alpha * L_CE + beta * L_KD + gamma * L_feat`

Current defaults:

- `alpha = 0.6`
- `beta = 0.3`
- `gamma = 0.1`
- temperature `T = 2.0`

### Hard-target term

`L_CE` is masked token cross-entropy with label smoothing **0.1**.

### Logit distillation term

`L_KD` is temperature-scaled KL divergence between Student and Teacher token distributions, masked over non-PAD tokens and scaled by `T^2`.

### Feature distillation term

The active BPE path applies feature alignment to the **semantic decoder outputs** (`s_dec_out` vs. `t_rx_dec`).

The function `feature_distillation_loss_cosine_normalized` normalizes Teacher and Student features, computes token-wise cosine similarity, and penalizes deviation from perfect alignment with an MSE-to-one objective over non-PAD target positions.

Channel-decoder feature matching exists in comments/alternate utilities but is **not active in the current BPE training path**.

## 4. Dataset and BPE preprocessing — current candidate

Current training class:

`EurParallelDatasetBPE`

Current default language pair used by the one-Student script:

- `en_en`

Current vocabulary file:

`./data/train/europarl_bpe/vocab_bpe.json`

Tokenization: **BPE**

Maximum sequence length: **67**

Training split size: **TBD — derive from the actual BPE dataset files used in the publication run**

Validation/test split size: **TBD**

Although the implementation branch contains multilingual datasets and language-pair options, this KD paper should use only the English/KD evidence needed for the Teacher-to-Student compression study unless the scope is explicitly changed.

## 5. Channel configuration — current BPE candidate

Current default channel:

- **Rayleigh**

Student training SNR mode:

- random SNR sampled uniformly from **2 dB to 18 dB** per batch when `snr_mode=range`

Validation SNR default:

- **8 dB**

The implementation also exposes AWGN and Rician options, but they are not automatically part of the publication experiment matrix.

Final evaluation SNR grid for BLEU/semantic plots: **TBD**

## 6. Optimization and reproducibility — current BPE defaults

- batch size: **32**
- epochs: **10**
- learning rate: **1e-4**
- optimizer: **Adam**
- Adam betas: **(0.9, 0.98)**
- epsilon: **1e-8**
- weight decay: **5e-4**
- gradient clipping: **1.0**
- random seed: **42**

Final publication hardware/GPU and software versions: **TBD**

## 7. Decoding

Greedy decoding and beam decoding have both been used/discussed in the wider project.

Final decoding method for the quantitative KD paper comparison: **TBD**

If both are reported, hold all other evaluation settings constant.

## 8. Metrics

Metrics known to be relevant:

- BLEU;
- semantic similarity / semantic metric;
- parameter count;
- model size;
- inference latency/complexity if measured;
- optionally perplexity/CE for training diagnostics only.

Exact BLEU implementation/configuration: **TBD**

Exact semantic metric/model: **TBD**

Latency hardware and timing protocol: **TBD**

## 9. Required result matrix

At minimum, aim to populate:

| Model | KD | Receiver architecture | Params | Size | BLEU @ SNR(s) | Semantic metric @ SNR(s) | Latency |
|---|---:|---|---:|---:|---:|---:|---:|
| Teacher | N/A | 8 layers / 16 heads | TBD | TBD | TBD | TBD | TBD |
| Student baseline | No | 4 layers / 8 heads | TBD | TBD | TBD | TBD | TBD |
| Student KD | Yes | 4 layers / 8 heads | TBD | TBD | TBD | TBD | TBD |

## 10. Important implementation distinction

Older `train_student.py` uses a different candidate configuration (12-layer Teacher and 2-layer Student) and currently has feature distillation commented out. `R_tr_kd.py` also defines another receiver-only KD formulation. These are useful historical/ablation references but should not be mixed with the current BPE experiment description.

## 11. Reproducibility checklist before Results are final

Confirm and record:

- exact BPE vocabulary size;
- exact train/validation/test split sizes;
- final Teacher checkpoint;
- final Student checkpoint;
- Teacher and Student parameter counts;
- compression ratio;
- Student-without-KD training recipe;
- final channel evaluation SNR grid;
- decoding configuration;
- BLEU configuration;
- semantic metric/model;
- random seeds used for final runs;
- software versions;
- hardware/GPU;
- inference timing procedure if latency is reported.
