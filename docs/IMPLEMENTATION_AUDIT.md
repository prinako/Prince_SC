# Implementation Audit — BPE Receiver Knowledge Distillation

Audit date: 2026-09-18. Status: **implementation inspection and candidate-artifact measurements complete; publication run confirmation and controlled experiments outstanding**.

## Current-state addendum — publication protocol, 2026-09-18

Current authoritative Student script SHA-256: `3b6faf47f33a2ef8ade1339e14e7ca260120727e4c1762f7443cde43d8b41092`.

Verified clean DeepSC `BPE` HEAD: `0d3b119c9215fb22f427d3aa7c9629c9a7cdc18c`, later than the supplied `546884e0035f17632a7131364ab3fedfe327dd8c`. Direct inspection confirms:

- `main_multi_vocab.py:243`: Teacher MAX-LENGTH is **67**, so the 68/67 issue is fixed.
- Preprocessing lines 213–234 and 802–818: deterministic **70/15/15** split, seed 48; lines 241–274 and 950 onward train BPE from training IDs only. Force a new tokenizer or use a clean output directory because an existing model can otherwise be reused.
- Normalization lines 81–103 applies NFKC/lowercase/**punctuation spacing**, not punctuation removal. This corrects the earlier audit description.
- Teacher `main_multi_vocab.py:41–117,121–210` uses train/val, per-batch uniform 2–18 dB, fixed validation 8 dB through `snr_to_noise`, and token-weighted non-PAD metrics. `utils/train_utils.py` uses unsmoothed CE; teacher-forced accuracy is not decoded accuracy. Teacher initialization is fresh; best checkpoints minimize validation CE (lines 302–355). Adam defaults and batch 32 are recorded in EXPERIMENTS.md.
- Student lines 401–420 now validate on **val**, not test. An unused `test_set` is still instantiated and loads test data; remove it before publication training. Historical hardcoded Teacher filenames remain at lines 445–446.
- Architecture, KD objective, shared noise, and previous SNR/weight-decay/feature-denominator fixes remain unchanged in the cumulative diff.

The chosen protocol is **70/15/15 → new train-only BPE → new Teacher → matched CE/KD Students → untouched test evaluation**. All earlier 90/10 measurements/checkpoints below are **HISTORICAL/CANDIDATE ONLY**, not publication evidence. No final artifact was verified or selected during this manuscript update. Final parameter totals depend on the new tokenizer/configuration and remain TBD.

Remaining issues: dated preprocessing versus undated loaders (Teacher prefixes vocabulary paths separately); lack of normalized-content grouping before ID splitting; missing `val_ratio`/sum validation; conditional tokenizer reuse; unused Student test loading; historical Teacher filenames; minimal run metadata; Student CSV/batch-aggregation limitations; common CE/KD checkpoint-selection criterion; matched decoding/trials and final metric/hardware protocol. Current preprocessor pools enabled language files for train-only BPE; record that corpus composition without claiming multilingual KD experiments. The complete final-run TBD register is in EXPERIMENTS.md.

**Reading the historical audit below:** its code references, hashes, test-as-validation statements, 90/10 split, old Teacher noise recipe, checkpoint counts, and pending-writing notes describe the earlier audit revisions. This addendum supersedes those statements for current implementation/protocol; it preserves them for provenance. Sections I and III–V are now drafted, with final results still pending.

## 1. Historical evidence boundary and inspection record

Authority is `DeepSC/train_multi_vocab_one_student.py`, not historical Student scripts. Workspace repositories inspected directly:

- `Prince_SC`, branch `paper-kd-student`, starting commit `7ed272c3497a1ed5f6d6b9567787dd846c553f02`;
- sibling `DeepSC`, branch `BPE`, original audit commit `b66afb331500f0e193b5d99f03a1f289b4f16e7f`; fixes verified at current revision `dee57fbff8c42fce74e2a9974b25f15b74143548`.

The original audit read and preserved a pre-existing modification to `test_BPE.ipynb`. At the verified revision this identical cleaned notebook is committed, and the DeepSC working tree is clean. No implementation, checkpoint, dataset, or notebook was changed, and no training or reconstruction-quality experiment was run. CPU checks used the existing DeepSC `.venv` with CUDA disabled. The academic-research-writer skill guided evidence separation and the primary-source comparison below.

The source map uses paths relative to DeepSC; line numbers identify the original audit revision unless explicitly updated below. The current training script has shifted subsequent lines by four; the feature helper has shifted subsequent lines by six. Notebook cell numbers below are zero-based and refer to the inspected working tree.

| Evidence | Location |
|---|---|
| Defaults, seeding, local validation | `train_multi_vocab_one_student.py:43–234` |
| Active training, freezing, losses | `train_multi_vocab_one_student.py:237–386` |
| Vocabulary/data/model construction, optimizer, saves | `train_multi_vocab_one_student.py:388–611` |
| Receiver-only Student | `student.py:8–24` |
| Attention, encoder/decoder, channel modules, DeepSC | `models/transceiver.py:27–282` |
| Shared transmitter / receiver wrappers | `models/tx_model.py`, `models/rx_model.py` |
| Active feature loss / logit KD | `utils/kd_utils.py:77–98,161–242` |
| Masks, channel, normalization, noise conversion | `utils/model_utils.py:286–423` |
| CE, save/load, CSV semantics | `utils/model_utils.py:495–611,677–705` |
| Dataset and dynamic padding | `dataset_multilingual.py:7–74` |
| Raw preparation / BPE preparation | `data_perprocess/create_en_en_base.py`; `data_perprocess/preprocess_text_mu_bpe.py:81–103,213–274,296–421,448–488,530–658,719–872` |
| Detokenization, text encoding, autoregressive decoding | `utils/bpe_utils.py:17–213,226–457,461 onward` |
| Text metrics and aggregation | `metrics/metrics_bpe.py:41–125,267–349,406 onward` |
| Candidate artifact association / evaluation demonstrations | `test_BPE.ipynb`, cells 3, 7–9, 26, 31, 48, 52, 58 |

`teacher.py`, `utils/train_utils.py`, and `main_multi_vocab.py` were also inspected to distinguish supporting helpers and possible Teacher training from the active KD path. Imported `build_teacher` and `validate_multi_epoch` do not control the active model construction/validation. `masked_ce_loss` and `create_masks` imported through `train_utils` resolve to `model_utils`. Historical `train_student.py` and `R_tr_kd.py` supply no configuration facts in this audit.

### Revision verification — 2026-09-18

The cumulative diff from the original audit to `dee57fbff8c42fce74e2a9974b25f15b74143548` changes only the authoritative training script, `utils/kd_utils.py`, and `test_BPE.ipynb`. The following fixes are verified in current code:

| Item | Current evidence | Status |
|---|---|---|
| Consistent SNR conversion | Training and validation use imported `snr_to_noise`; divergent local helper removed | Fixed |
| Validation SNR flag | `train_multi_vocab_one_student.py:483`: `snr_to_noise(float(args.val_snr_db))` | Fixed |
| CLI weight decay | `train_multi_vocab_one_student.py:496`: `weight_decay = args.weight_decay` | Fixed |
| Feature denominator | `utils/kd_utils.py:103–104`: valid count clamped to at least one | Fixed |
| Notebook cleanup | All code-cell execution counts are null and outputs empty; committed hash matches the previously audited working tree | Verified |

Focused checks evaluated the actual validation and optimizer argument expressions with distinct train/validation SNR values and a nondefault weight decay. CPU checks confirmed finite zero feature loss and gradients on all-PAD targets, and unchanged squared-cosine loss on a mixed mask. Notebook JSON checks confirmed cleared execution/output fields. No training or quality evaluation was run. The fixes do not establish how older artifacts were trained, or resolve the independent checkpoint, split, baseline, and evaluation-protocol issues below.

## 2. Architecture and forward path

Let B be batch size, L source length, M decoder-input length, and V vocabulary size. The Teacher semantic encoder maps `[B,L]` tokens to `[B,L,128]`. It uses a learned V×128 embedding, scaled by sqrt(128), sinusoidal positional buffers, and eight Transformer encoder layers. Each encoder layer has 16-head self-attention, a 128→512→128 ReLU feed-forward network, residual connections, and post-LayerNorm.

The channel encoder is Linear 128→256, ReLU, Linear 256→16. The transmitter wrapper applies power normalization and the selected channel to produce `[B,L,16]`. That same noisy tensor is consumed by both receivers; no second independent channel draw is used for Teacher targets.

Each channel decoder is Linear 16→128 followed by the 128→512→128 ReLU branch, residual from the first projection, and LayerNorm. Its output memory is `[B,L,128]`. Each semantic decoder layer uses masked self-attention, cross-attention to this memory, a 128→512→128 ReLU network, three residual/post-LayerNorm operations, and dropout. Decoder embeddings and output projections are **not tied**.

| Configuration | Teacher | Student |
|---|---:|---:|
| Semantic encoder layers / heads | 8 / 16 | Absent |
| Semantic decoder layers / heads | 8 / 16 | 4 / 8 |
| Model width / FFN width | 128 / 512 | 128 / 512 |
| Attention head dimension | 8 | 16 |
| Channel real-valued width | 16 | Input from shared transmitter |
| Current positional capacity | 67 | 67 |
| Default dropout | 0.1 | 0.1 |

Teacher depth/heads are parsed arguments; Student depth 4 and heads 8 are hardcoded at training-script lines 475–485. Attention projection matrices remain 128×128 for either head count. Thus heads change the partition of features, not projection parameter totals. Internal attention/feed-forward dropout includes hardcoded 0.1 values; the exposed dropout flag is not a complete global override.

The transmitter wrapper and Teacher receiver reuse the instantiated Teacher components. All their parameters have `requires_grad=False`, their modules use `eval()`, and their forward passes use `no_grad`. The Student channel decoder, embedding, four decoder layers, and output head all train. No Student parameters are copied from the Teacher; `--init-student-from-teacher` is parsed but not read. Both receivers are teacher-forced during training and validation, not autoregressively decoded for the loss.

## 3. Exact active objective

At shifted target position i, let y_i be the hard target, m_i=1[y_i≠PAD], N=sum_i m_i, z_i^S/z_i^T the logits, and h_i^S/h_i^T the final semantic decoder outputs before the vocabulary projection. The encoder consumes the full source; the decoder consumes `trg[:, :-1]`, with targets `trg[:, 1:]`. Source padding and target padding/causal attention masks are separate from the loss mask.

With epsilon_ls=0.1 and V vocabulary entries, CE is

```text
q_i(v) = (1 − epsilon_ls) 1[v = y_i] + epsilon_ls / V
L_CE = − sum_i m_i sum_v q_i(v) log softmax(z_i^S)_v / max(N, 1).
```

Smoothing distributes mass over all classes, including the PAD class; PAD target positions themselves are ignored. The language token and EOS remain valid targets.

With temperature T=2,

```text
p_i^T = softmax(z_i^T / T),  p_i^S = softmax(z_i^S / T)
L_KD = T² sum_i m_i sum_v p_i^T(v) log[p_i^T(v) / p_i^S(v)] / max(N, 1).
```

The active `F.kl_div(student_log_probs, teacher_probs)` therefore computes **KL(Teacher || Student)**. Teacher targets are detached; the code uses a sum over vocabulary and a mean over non-PAD token positions, not `batchmean`.

The feature term is

```text
u_i^S = h_i^S / (||h_i^S||_2 + 1e−8)
u_i^T = h_i^T / (||h_i^T||_2 + 1e−8)
c_i = dot(u_i^S, u_i^T)
L_feat = sum_i m_i (1 − c_i)² / max(N, 1)
L = 0.6 L_CE + 0.3 L_KD + 0.1 L_feat.
```

This is squared cosine-to-one alignment of the **final decoder features**, not ordinary vector MSE, not layer-by-layer matching, and not an active channel-decoder loss. The commented channel-decoder MSE and generic alternative utilities must not enter the Methods description. Because the norm uses an additive epsilon, the implemented c_i is a stabilized cosine expression.

CPU synthetic checks verified the CE formula, KL masking, and zero gradient at PAD positions for all three terms. The original revision lacked a feature-loss denominator guard. The current revision clamps the count to at least one; a finite all-PAD input now returns zero loss and zero gradients. Mixed-mask behavior is unchanged. Actual audited sequences contain non-PAD targets; the original edge case was not evidence that the candidate run encountered NaNs.

## 4. Data, tokenizer, and split audit

The training script reads `vocab_size` from the JSON and constructs `EurParallelDatasetBPE(args.en, 'train')` and `(..., 'test')`. Default `args.en` is `en_en`; other language flags are unused in this one-Student loop. The dataset independently defaults to `./data/train/europarl_bpe`; overriding `--vocab-file` does not change that directory. `collate_fn` pads variable-length pairs with zero, consistent with the candidate PAD ID but not automatically with an arbitrary replacement vocabulary.

The undated default vocabulary/data files are absent. The notebook explicitly associates the September 12 dated artifacts and September 15/17 checkpoints, so these were audited as a **candidate association**, not proven run provenance.

### Preprocessing recipe found in code

The BPE preprocessor reads Europarl parallel JSON; English reconstruction uses en_en. It normalizes with NFKC, lowercase, punctuation removal, and whitespace collapse. It shuffles English record IDs with default seed 48 and partitions 90/10; other language pairs follow those IDs. Tokenizer training uses unique normalized source/target texts from training IDs across enabled en_en/en_pt/en_es/en_fr inputs. Thus English-only KD may still use a jointly trained multilingual vocabulary; this infrastructure is not a multilingual KD experiment.

SentencePiece defaults request BPE V=96,000, character coverage 1, byte fallback, PAD/BOS/EOS/UNK IDs 0/1/2/3, language symbols 4–7, identity internal normalization, no input subsampling, and `hard_vocab_limit=False`. The JSON records the actual vocabulary size. Encoded content length must be 4–64 on both sides; it is filtered, not truncated. Three special tokens yield maximum length 67. Inference `text_to_indices_bpe` instead truncates content to `max_len−3` and uses a normalization helper without the preprocessor's explicit NFKC step.

These are current-code facts. Exact original preprocessing command, corpus release, source hashes, and tokenizer-training inputs are not recorded alongside the candidate artifacts; do not infer them solely from matching shapes.

### Direct artifact measurements

For `data/train/europarl_bpe/2026-09-12`, all 96,000 SentencePiece piece-to-ID mappings match the JSON. All 256 byte pieces are present. Special IDs match the current code. Full pickle scans establish:

| Measurement | Train | File named test |
|---|---:|---:|
| Pairs | 1,391,914 | 154,651 |
| Unique source token sequences | 1,391,875 | 154,650 |
| Duplicate rows beyond unique sources | 39 | 1 |
| Minimum / maximum stored length | 7 / 67 | 7 / 66 |
| Minimum / maximum observed ID | 1 / 95,999 | 1 / 95,992 |
| All source sequences equal target | Yes | Yes |
| All have START, EN prefix and END suffix | Yes | Yes |

The intersection contains **eight unique source token sequences**. An ID-based split does not prevent duplicated normalized content from crossing partitions. The current script uses the file named `test` for every validation pass and best-checkpoint selection. It cannot simultaneously serve as an untouched final test set.

The September 11 alternative has V=32,000, train/test sizes 1,391,436/154,599 and six overlapping unique sequences. It is a distinct artifact set and is not the basis of the candidate parameter totals below.

## 5. Channel, power, and SNR

`power_normalize` computes RMS over the entire batch tensor, including source padding and special-token positions, and divides only when RMS exceeds one. It is a power **cap**, not exact unit-power normalization for every sentence or batch. A synthetic constant-0.1 tensor remains at RMS 0.1.

`Channels.Rayleigh` samples one real/imaginary Gaussian pair with variance 1/2, forms a 2×2 complex-multiplication matrix, reshapes channel outputs into real/imaginary pairs, applies fading, adds iid real Gaussian noise with the supplied standard deviation, and multiplies by the exact inverse fading matrix. A single coefficient is shared across the entire batch and all sequence positions. The receiver therefore assumes perfect channel knowledge. There are eight complex channel symbols per source position; do not describe 16 real coordinates as 16 complex symbols. The active path has no multiuser interference, estimated CSI, or fading-specific Student receiver.

Training samples SNR uniformly in dB per batch from [2,18] in range mode. Fixed mode uses `args.snr_db`. Both use the imported helper

```text
sigma_train(s) = 1 / sqrt(2 × 10^(s/10)).
```

Validation now uses the same imported helper with `s = args.val_snr_db` (default 8 dB). Both paths therefore produce sigma **0.2815042799 at 8 dB**. The validation argument was checked with a value different from the training argument.

Historically, the original audit revision used a separate `10^(−s/20)` helper and the training SNR argument for validation, producing twice the noise variance at equal nominal SNR. This discrepancy and the ignored validation flag are **fixed in the current revision**. Older artifact provenance remains unconfirmed; no rerun is inferred from the fix. The RMS cap and Rayleigh equalization still affect realized signal/noise behavior. Validation draws fresh channel noise; it is not a fixed bank of channel realizations.

AWGN and Rician (K=1 in the helper) are selectable, but code availability is not evidence of completed experiments. Final evaluation SNR grid is unselected. The notebook's integer 0–29 dB loop concerns a hand-entered sentence. Legacy `performance.py` is a word-level full-model path, including an unresolved `SNR_to_noise` import, and must not supply the current BPE evaluation protocol.

`main_multi_vocab.py` suggests a possible Teacher-training recipe, but the checkpoint lacks provenance tying it to that script. In that script, training samples noise standard deviation between the 5/10 dB endpoints once per epoch and validation uses 0.1. These are **not** established training conditions of `encoder_26.pth`/`decoder_26.pth` and are not Student-training settings.

## 6. Parameter and serialized-size measurements

Actual classes were instantiated on PyTorch's `meta` device for V=96,000, max length 67, width 128, FFN 512, and the authoritative layer/head settings. `sum(p.numel() for p in module.parameters())` was compared with saved component tensors excluding registered positional buffers. All candidate parameter totals agree:

| Component | Teacher parameters | Student parameters |
|---|---:|---:|
| Semantic encoder | 13,874,176 | — |
| Channel encoder | 37,136 | — |
| Channel decoder | 134,144 | 134,144 |
| Semantic decoder, including embedding | 14,404,608 | 13,346,304 |
| Dense vocabulary projection | 12,384,000 | 12,384,000 |
| Receiver subtotal | **26,922,752** | **25,864,448** |
| Full Teacher | **40,834,064** | — |

The shared transmitter has 13,911,312 parameters. Receiver reduction is `(26,922,752−25,864,448)/26,922,752 = 3.9308909%`; Teacher-RX/Student-RX ratio is **1.0409173×**. Deployed transmitter-plus-Student total is 39,775,760, or **2.5917185%** fewer parameters than the full Teacher. The fixed receiver embedding alone has 12,288,000 parameters. These are candidate architectural measurements, not quality/latency results or a selected publication run.

| Actual file relative to DeepSC | Bytes | Positional-buffer shape |
|---|---:|---|
| `results/base_model/2026-09-15/encoder_26.pth` | 55,726,451 | `[1,67,128]` |
| `results/base_model/2026-09-15/decoder_26.pth` | 107,802,247 | `[1,67,128]` |
| `results/student/2026-09-17/student_03.pth` | 152,651,397 | **`[1,96000,128]`** |

The candidate Student checkpoint is larger than the Teacher receiver file. The extra positional-buffer capacity accounts for **49,117,696 additional bytes** compared with capacity 67, before serialization overhead. Positional encodings are registered buffers, not trainable parameters.

Strict component loading into the current default architecture passed for all Teacher components and Student channel decoder/dense. Student semantic decoder loading failed solely on `pos_encoding.pe`: saved `[1,96000,128]`, expected `[1,67,128]`. Notebook cell 8 constructs Student positional capacity from vocabulary size, explaining compatibility with that notebook but not with the authoritative current training script. Changing the training script's single max-length argument to 96,000 would also conflict with the candidate Teacher's 67-position buffers. The provenance mismatch needs an explicit resolution; this audit did not crop buffers or rewrite artifacts.

Checkpoint tensors show eight/four layers and V=96,000, but cannot verify attention head count because heads do not change projection shapes. Teacher files have no run metadata. Student metadata records epoch 3, validation loss 2.6883684599989075, T=2, weights 0.6/0.3/0.1, and Rayleigh. That loss is an unvalidated historical checkpoint field, **not a paper result**, and does not prove training SNR, seed, decoding quality, or tokenizer identity.

## 7. Optimization, baseline availability, and evaluation evidence

Adam trains only Student parameters with defaults LR 1e-4, betas (0.9,0.98), epsilon 1e-8; weight decay now uses `args.weight_decay`, default 5e-4. The original ignored-flag issue is fixed, and a nondefault value was checked. Batch size is 32, gradient norm clipping 1.0, epochs 10, workers 0. Seed 42 initializes Python/NumPy/PyTorch and deterministic cuDNN settings; the script does not establish a multiple-seed experimental protocol. There is no scheduler, early stopping, optimizer resume state, or comprehensive run manifest.

Epoch diagnostics average per-batch losses, giving a final short batch equal weight. CE_PPL is based on smoothed CE and is not corpus perplexity. Saves go to `save_dir/one_student/YYYY-MM-DD`; each epoch writes receiver component states, and best selection minimizes validation composite loss. CSV helper numbering adds one to the caller's already incremented epoch, so CSV starts at 2 while checkpoints start at 1. Date-only destinations can mix repeated runs. Metadata omits the command, source revision, architecture, SNR settings, tokenizer/data hashes, optimizer state, and RNG state.

### Matched no-KD baseline

The local inventory contained 16 Student checkpoints: eleven August 12 two-layer/V=96,327 artifacts, four August 24 two-layer/V=32,000 artifacts, and the September 17 four-layer/V=96,000 artifact. Their inspected metadata describe KD; no documented matching four-layer/eight-head CE-only run was identified. Older shapes are not substitutes for a capacity-controlled baseline. Saved head counts remain unverified.

Required comparison: frozen identical transmitter plus (a) Teacher receiver, (b) four-layer/eight-head CE-only Student, and (c) the same Student with KD. Keep training data, initializations/seeds, budget, smoothing, channel sampling, decoding, and metric definitions controlled. In the current script `alpha=1,beta=0,gamma=0` makes the objective CE-only, but still computes Teacher and KD forward terms; those runs would not establish optimized CE-only training time. No such run was started in this audit. CE+logit and CE+feature ablations would isolate each active KD contribution.

### Decoding and metrics

`SeqtoTextBPE` strips PAD/START/language symbols, stops at EOS, and uses SentencePiece decoding when its model is supplied. Greedy and beam functions start with `[START, language]`; channel decoding is reused within generation. Beam defaults are width 5 and length penalty 0.7, with GNMT-style length normalization and optional forbidden tokens. The notebook explicitly selects width **10** for the Student while the Teacher uses **greedy**. They share a channel realization but not a decoding budget, so this demonstration cannot isolate a KD effect.

`DeepSCMetrics` supports SacreBLEU sentence scores with exponential smoothing and corpus BLEU (0–100), chrF++, ROUGE-L, exact match, token accuracy, and normalized sentence-embedding cosine. Its default embedding model is `sentence-transformers/paraphrase-multilingual-mpnet-base-v2`, without a pinned model revision. `summarize_by_snr` averages sentence BLEU; this is not corpus BLEU. Notebook alternatives use multilingual MiniLM, NLTK BLEU with method-1 smoothing (0–1), and an explicitly fake random-embedding demonstration. These are alternatives/demos, not a final consistent semantic-quality evaluation.

No reproducible dataset-wide BPE Teacher/CE-only/KD evaluation with matched decoding and final run identities was established. Final BLEU signature, semantic model/revision, held-out data, SNR grid, channel trials, seeds/confidence intervals, hardware, synchronization/warm-up, batch size, generated lengths, and latency aggregation remain TBD. File sizes above are actual serialization measurements; no FLOP, latency, energy, or quality-preservation claim follows from them.

## 8. Closest prior work and contribution boundary

Primary sources checked: Liu et al., *IEEE TWC* 2024, bibliography key `liu2024kdsemcom`, [publication DOI](https://doi.org/10.1109/TWC.2023.3336941), [author manuscript](https://arxiv.org/pdf/2311.13789), especially Algorithm 3, Eqs. (26)–(28), Tables I/III and numerical evaluation; Eid et al., **2026 preprint**, key `eid2026krumdeepsc`, [version 1](https://arxiv.org/html/2609.13405v1), especially the architecture, distillation, and experimental sections. Publication metadata and broader search limitations remain in [LITERATURE_REVIEW_NOTES.md](LITERATURE_REVIEW_NOTES.md). Each prior-work column below is supported by its linked primary source.

| Dimension | Liu et al. — [source](https://arxiv.org/pdf/2311.13789) | Eid et al. — [preprint](https://arxiv.org/html/2609.13405v1) | Prince_SC — audited candidate |
|---|---|---|---|
| Modality | Text | Text | English text reconstruction |
| Teacher/Student placement | Complete transmitter–receiver Students | Complete Student; five-Teacher ensemble | Frozen transmitter and Teacher receiver; train receiver only |
| Compression scope | End-to-end | End-to-end | Receiver replacement with unchanged transmitter |
| Architecture reduction | 4→2 Transformer layers; additional channel pruning/quantization variants | 4→2 layers; FFN 512→256; width 128, eight heads | Decoder 8→4 layers; heads 16→8; width 128 and FFN 512 unchanged |
| KD signals | CE and KL-based output/component transfer | CE, logit KL, feature MSE, decoder cosine, residual term | Smoothed CE + KL(Teacher||Student) + squared cosine-to-one decoder alignment |
| Channel | Rayleigh/perfect CSI; interference scenarios | AWGN/Rayleigh, perfect CSI; residual two-stream transmitter | Rayleigh/perfect CSI; batch-shared fading; identical noisy latent for both receivers |
| SNR treatment | No-interference: Teacher 10–15, transfer 15–18, evaluation 0–18 dB | Train 5–10; evaluate {0,3,6,9,12,18} dB | Train uniform 2–18 dB; validation 8 uses the same conversion (fixed revision); final grid TBD |
| Baselines | Matched no-KD Students, DeepSC, conventional schemes | Single Teacher, ensemble, single-Teacher KD, ablations | Matched CE-only baseline still required |
| Deployment metrics | Parameters, size, training/inference time | Non-embedding parameters and inference timing | Full receiver parameters and actual file bytes measured; latency TBD |

The supported distinction from these two configurations is **KD for a replacement receiver while retaining the original frozen transmitter**, rather than training a compressed end-to-end link. This is a positioning boundary against these sources, not proof of global novelty. Text modality, Transformers, compact Students, and noisy-channel KD already occur in prior work. Decoder cosine supervision also occurs in Eid et al.; the squared form alone does not establish a new contribution.

A defensible investigation would test whether this particular receiver replacement preserves reconstruction quality against a same-capacity CE-only receiver across a controlled channel protocol, then measure its actual deployment cost. The 96,000-token vocabulary makes the measured parameter saving modest; comparisons excluding embeddings are not directly comparable to the full totals here. BPE vocabulary provenance, head changes, or multilingual support should not be promoted to novelty claims without additional evidence.

## 9. TBD / requires run confirmation

| Priority | Unresolved item | Evidence needed before final claims |
|---|---|---|
| 1 | Candidate Student is incompatible with current max-length construction | Original run command/revision or a documented corrected rerun; explicitly resolve positional buffer |
| 1 | Default vocabulary/data/checkpoint paths do not locate available artifacts | Consistent explicit data root and immutable artifact manifest |
| 1 | Older checkpoint runs are not tied to the corrected validation/noise implementation | Record corrected revision and SNR arguments in publication runs; determine which historical runs require repetition |
| 1 | Test file used for model selection; eight cross-split duplicate sequences | Separate validation and held-out test with documented content overlap policy |
| 1 | No matched CE-only baseline located | Verified four-layer/eight-head baseline artifact and run record |
| 2 | Exact publication Teacher/Student and tokenizer provenance | Designated hashes, training/preprocessing commands, corpus version, source revision |
| 2 | Final quality evaluation unspecified | Matched decoding, corpus metric signature, semantic model/revision, SNR grid, seeds/trials |
| 2 | Deployment benefit beyond parameter count unmeasured | Comparable serialized deployment artifacts and measured latency/memory/complexity protocol |
| 2 | Loss-component contribution untested | Controlled CE/logit/feature ablations and uncertainty reporting |

These pending experiments do not prevent completion of this implementation audit. They prevent treating defaults and exploratory artifacts as validated publication results. No final configuration was selected, so no final-run decision was added to `RESEARCH_DECISIONS.md`.

The next **paper-writing** task is Section III (System Model) and Section IV (Knowledge Distillation Framework), using the verified frozen-transmitter architecture and exact objective. Mark protocol choices as provisional until the issues above are resolved; numerical Results remain unwritten. Related Work does not require expansion based on this audit.

## 10. Artifact identity and reproducibility

All paths below are relative to DeepSC. Hashes were computed directly; these establish the inspected files, not their original training provenance.

| Artifact | SHA-256 |
|---|---|
| Current `train_multi_vocab_one_student.py` | `7aa7355aac29a946d408012ddee4e6ad38139201a96890b1a36f3be7d736998b` |
| Current committed `test_BPE.ipynb` (same as original audited working tree) | `5f8dc51b56b5eb1a13062b6ad671e67bad735908e28deb52d7771ebdaadec53e` |
| `results/base_model/2026-09-15/encoder_26.pth` | `6194bd33ca8f15620286ac28ba1e2bd44e1d19f7a6faefea01be39e0d3614917` |
| `results/base_model/2026-09-15/decoder_26.pth` | `44b80a2e2992214429a009822ff2a3fd48950d7e2077e451c9038675fb94bcab` |
| `results/student/2026-09-17/student_03.pth` | `cfdaae2a68761db3a2e807b641ee142f1a4c42178e37afcfe45a3f7024a9450f` |
| `data/train/europarl_bpe/2026-09-12/vocab_bpe.json` | `7d5b0b27e46af10b5be36f5bb30faf33743cf0b6b98b57f65b604bede24aa5b3` |
| `data/train/europarl_bpe/2026-09-12/tokenizer_bpe.model` | `9c2f17db7d01410e08b5a21cde3acc4aa3c0b503f8b5bd177cd11511cd84170a` |
| `data/train/europarl_bpe/2026-09-12/train_en_en.pkl` | `de35551e69db4835a08fa696468f9bac703e63a47fec142040af274141640c11` |
| `data/train/europarl_bpe/2026-09-12/test_en_en.pkl` | `932a63849353694f1eb9072657a922433caf28e81778952bddfe62dbb6aee279` |

Audit runtime: Python 3.12.13, PyTorch 2.13.0+cu130, NumPy 2.4.2, SentencePiece 0.2.2, SacreBLEU 2.6.0, NLTK 3.10.2, sentence-transformers 5.4.1. These are inspection-environment versions, not recovered training-environment metadata.

Minimal reproduction of model counts and the strict-loading discrepancy, run from the DeepSC root with its existing environment:

```bash
CUDA_VISIBLE_DEVICES='' PYTHONDONTWRITEBYTECODE=1 .venv/bin/python - <<'PY'
import json
from pathlib import Path
import torch
from models.transceiver import DeepSC
from student import Student

root = Path('data/train/europarl_bpe/2026-09-12')
V = json.loads((root / 'vocab_bpe.json').read_text())['vocab_size']
with torch.device('meta'):
    teacher = DeepSC(8, V, V, 67, 67, 128, 16, 512, 0.1)
    student = Student(4, V, V, 67, 67, 128, 8, 512, 0.1)
count = lambda model: sum(p.numel() for p in model.parameters())
rx = sum(count(getattr(teacher, k)) for k in ('channel_decoder', 'decoder', 'dense'))
print('V, full Teacher, Teacher RX, Student RX:', V, count(teacher), rx, count(student))
print('RX reduction %, ratio:', 100 * (rx-count(student))/rx, rx/count(student))
for file, model, keys in [
    ('results/base_model/2026-09-15/encoder_26.pth', teacher, ('encoder','channel_encoder')),
    ('results/base_model/2026-09-15/decoder_26.pth', teacher, ('channel_decoder','decoder','dense')),
    ('results/student/2026-09-17/student_03.pth', student, ('channel_decoder','decoder','dense')),
]:
    state = torch.load(file, map_location='cpu', weights_only=True)
    print(file, Path(file).stat().st_size, 'bytes')
    for key in keys:
        try:
            getattr(model, key).load_state_dict(state[key], strict=True, assign=True)
            print(key, 'PASS')
        except RuntimeError as exc:
            print(key, 'FAIL', str(exc))
PY
```

Dataset checks loaded the trusted local pair pickles, compared all token sequences and prefix/suffix IDs, and intersected source-sequence sets; tokenizer checks compared every SentencePiece ID/piece with the JSON map. Safe `torch.load(..., weights_only=True)` was used for artifact inspection. Hash/file-size measurement, actual-class parameter enumeration, strict component loading, complete dataset scans, and synthetic loss/noise checks underpin the findings above. None substitutes for a final experimental run.
