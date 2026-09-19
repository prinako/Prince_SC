# Experiments — Source of Truth for the KD Paper

Updated 2026-09-18 after direct inspection of the workspace DeepSC repository. **Code defaults, measured candidate artifacts, and final publication results are different evidence levels. No final run is selected here.** Exact references, loss equations, artifact hashes, checks, and prior-work comparison appear in [IMPLEMENTATION_AUDIT.md](IMPLEMENTATION_AUDIT.md).

## Current publication protocol — supersedes old 90/10 candidates

**70/15/15 → new train-only BPE → new Teacher from scratch → matched CE Student / KD Student → untouched test evaluation.**

The split is selected policy, not a completed-run result. Use preprocessing seed 48 and common record IDs across relevant data files. Train a fresh tokenizer from training IDs only, select the retrained Teacher on validation, then freeze it for both Student runs. Use the same four-layer/eight-head Student architecture, smoothing, optimizer, seed/initialization policy, budget, data, channel conditions, and decoding across CE-only and KD experiments. The final test partition is reserved until the protocol and checkpoints are fixed. Old 90/10 Teacher/Student/tokenizer artifacts cannot enter this pipeline.

## 1. Authority and scope

Implementation: `https://github.com/pesqSC/DeepSC.git`, branch `BPE`, current verified commit `0d3b119c9215fb22f427d3aa7c9629c9a7cdc18c`. The authoritative Student training script is **`train_multi_vocab_one_student.py`**. Its supporting model, data, preprocessing, KD, channel, and evaluation code were inspected directly. The cleaned `test_BPE.ipynb` is now committed with cleared execution/output fields; its contents match the working-tree version inspected in the original audit. The DeepSC working tree was clean at revision verification.

The paper studies **English text reconstruction with a frozen Teacher transmitter and a smaller receiver trained by KD**. Multilingual tokenizer infrastructure does not establish a multilingual KD contribution. LoRA and personalized multilingual receivers remain outside scope. `train_student.py`, `R_tr_kd.py`, and alternate losses do not define this configuration.

## 2. Architecture and trainability — code verified

| Component | Teacher | Student |
|---|---|---|
| Semantic encoder | 8 layers, 16 heads | Uses the frozen Teacher transmitter |
| Channel encoder | Linear 128→256, ReLU, Linear 256→16 | Same transmitted/noisy latent |
| Channel decoder | Linear 16→128→512→128, ReLU, residual, LayerNorm | Same architecture; independently trainable |
| Semantic decoder | 8 layers, 16 heads | 4 layers, 8 heads |
| Model / feed-forward width | 128 / 512 | 128 / 512 |
| Vocabulary / output head | Requested V=96,000; final V TBD; untied Linear 128→V | Same vocabulary and head dimensions |
| Positional capacity in current script | 67 | 67 |
| Dropout default | 0.1 | 0.1 |
| During Student training | Transmitter and entire Teacher receiver frozen, eval mode | Entire receiver trainable, train mode |

The Student has no semantic/channel encoder. Both receivers see the **same realization** of the noisy transmitted tensor. Student initialization is fresh; the `--init-student-from-teacher` flag is unused. Attention projections retain width 128: reducing head count alone does **not** reduce their parameter count. The layer reduction supplies the parameter saving.

## 3. HISTORICAL 90/10 artifacts and measured parameters

**Historical/candidate evidence only. None of these artifacts or counts is a final publication result; all final counts must be recomputed from the new vocabulary and checkpoints.**

The default checkpoint directory is `./checkpoints/deepsc-Rayleigh/multilingual_bpe/2026-09-15`, with hardcoded filenames `encoder_26.pth` and `decoder_26.pth`. These defaults do not resolve to the available artifacts. The notebook associates the following actual files with the 2026-09-12 BPE data:

- `results/base_model/2026-09-15/encoder_26.pth`;
- `results/base_model/2026-09-15/decoder_26.pth`;
- `results/student/2026-09-17/student_03.pth`.

Their tensors establish V=96,000 and decoder depths 8/4. Head counts are code facts, not recoverable from these tensor shapes. The notebook association does not prove the original training command, tokenizer identity, or code revision.

Counts below were computed from the actual model classes and checked against checkpoint component tensors, excluding positional buffers:

| Candidate component / deployment | Parameters |
|---|---:|
| Teacher semantic encoder | 13,874,176 |
| Teacher channel encoder | 37,136 |
| Teacher channel decoder | 134,144 |
| Teacher semantic decoder, including embedding | 14,404,608 |
| Teacher output projection | 12,384,000 |
| Full Teacher | **40,834,064** |
| Teacher receiver only | **26,922,752** |
| Student receiver only | **25,864,448** |
| Shared transmitter plus Student receiver | 39,775,760 |

Receiver saving: **1,058,304 parameters, 3.9309%**, or Teacher-RX/Student-RX **1.0409×**. Full deployed pipeline saving is **2.5917%**. Do not describe the halved decoder depth as a 50% reduction of receiver parameters. The large embedding and untied vocabulary projection dominate both receivers.

Measured serialized files: Teacher transmitter **55,726,451 bytes**, Teacher receiver **107,802,247 bytes**, Student receiver **152,651,397 bytes**. The Student file is larger despite fewer parameters: its positional buffer is `[1,96000,128]`, versus `[1,67,128]` in the current script and Teacher. Strict loading into the current Student configuration **fails on this buffer**. Therefore these files do not yet support a deployment-storage reduction claim or an exact current-script run claim. Resolve provenance/configuration before publication; no checkpoint was modified during this audit.

## 4. Active KD objective

`L = 0.6 L_CE + 0.3 L_KD + 0.1 L_feat`, temperature **T=2**.

- CE: token cross-entropy, label smoothing **0.1**, average over non-PAD shifted targets.
- KD: **KL(Teacher || Student)** between temperature-softened token distributions, averaged over the same mask and multiplied by **T²**.
- Features: final semantic decoder vectors, each normalized by `L2 norm + 1e-8`; average **squared deviation of their cosine similarity from one**, `(1−cos)²`, over non-PAD shifted targets, with the denominator clamped to at least one (all-PAD inputs return zero for finite features). This is not raw feature MSE or channel-decoder matching.

Teacher forcing uses `trg[:, :-1]` to predict `trg[:, 1:]`; language and EOS tokens contribute, PAD does not. Transmitter and Teacher forward passes run under `no_grad`. Channel-decoder feature MSE is commented out. Exact equations and masking caveats are in the audit.

## 5. HISTORICAL 90/10 data measurements

**The following September 11/12 artifacts predate the selected 70/15/15 publication pipeline.**

`EurParallelDatasetBPE(args.en, split)` defaults to **en_en** and the hardcoded undated directory `./data/train/europarl_bpe`. The vocabulary flag alone does not redirect dataset files. Undated defaults are missing in this workspace; dated artifacts exist.

Candidate: `data/train/europarl_bpe/2026-09-12/`. Its JSON mapping matches all 96,000 SentencePiece entries; the tokenizer includes 256 byte pieces. PAD/START/END/UNK IDs are 0/1/2/3 and EN/PT/ES/FR IDs are 4/5/6/7.

| Artifact | Pairs | Unique source sequences | Stored sequence length |
|---|---:|---:|---:|
| `train_en_en.pkl` | **1,391,914** | 1,391,875 | 7–67 |
| `test_en_en.pkl` | **154,651** | 154,650 | 7–66 |

All inspected English pairs have identical source/target sequences and `[START, EN, content, END]` structure. **Eight unique token sequences occur in both splits.** The historical training script used `test` for validation and checkpoint selection. Current Teacher and Student validation loaders now use `val`; the Student still instantiates an unused `test_set`, which should be removed from publication training. These counts must not be described as independent train/validation/test evidence.

Historical preprocessing used Europarl JSON, NFKC/lowercase/punctuation-spacing/whitespace normalization, an ID-based 90/10 split with default seed 48, and SentencePiece BPE trained from training IDs across enabled language pairs. It filters content lengths 4–64 and adds three special tokens. Exact corpus release, preprocessing invocation, tokenizer training provenance, split repair, and final held-out test remain **TBD**. The separate 2026-09-11 tokenizer has V=32,000 and must not be mixed with the 96,000-token checkpoints.

## 6. Channel and SNR — conversion fixed

Default channel: **Rayleigh**; AWGN/Rician are optional code paths, not established publication experiments. The transmitter emits 16 real values (8 complex symbols) per source position. Power normalization caps batch-wide RMS at one; it does not increase power below that threshold. Padding participates in this operation.

Rayleigh uses one complex fading coefficient for the whole batch, adds independent real Gaussian noise, and equalizes with the exact channel inverse (perfect CSI). No multiuser interference is implemented in this active path.

Training samples SNR uniformly in dB from **2 to 18 per batch**, or uses `--snr-db` in fixed mode. Its helper uses `sigma=1/sqrt(2*10^(SNR/10))`.

**Fixed in verified revision `dee57fbff8c42fce74e2a9974b25f15b74143548`:** validation now calls the same `snr_to_noise` helper using **`args.val_snr_db`**, default 8 dB. At 8 dB both paths use sigma 0.2815043. The previous factor-of-two variance discrepancy is resolved in current code; this does not retroactively establish the settings or validity of older checkpoint runs. The final evaluation grid and publication run remain TBD.

## 7. Optimization and checkpoint selection

Defaults: batch 32; 10 epochs; Adam LR 1e-4, betas (0.9, 0.98), epsilon 1e-8; gradient clipping 1.0; seed 42; workers 0. Weight decay now uses **`args.weight_decay`**, default **5e-4**; a nondefault CLI value was checked. No scheduler, resume state, or early stopping is configured.

Best checkpoint minimizes the validation **composite KD loss**, not BLEU. Epoch values are batch means, not corpus token-weighted means. CE_PPL is a smoothed-CE diagnostic, not standard corpus perplexity. CSV epoch numbering is one ahead of checkpoint numbering. Same-day saves can overwrite prior checkpoints and append to an existing CSV.

Student metadata stores epoch, validation loss, loss weights, temperature, and channel, but omits architecture, tokenizer/data hashes, SNR, seed, optimizer/RNG state, and implementation revision. Candidate `student_03.pth` records epoch 3 and the default KD weights/temperature/Rayleigh; this is insufficient to reconstruct its run.

Audit environment, **not proven training environment**: Python 3.12.13, PyTorch 2.13.0+cu130, NumPy 2.4.2, SentencePiece 0.2.2, SacreBLEU 2.6.0, NLTK 3.10.2, sentence-transformers 5.4.1. Publication hardware and run environment remain TBD.

## 8. Baselines, decoding, and metrics

In the historical inventory, no matching **4-layer/8-head CE-only Student** run was identified in the inspected scripts/artifact inventory. Sixteen Student checkpoints were examined; the sole 4-layer/96,000-token candidate has active KD metadata. Absence from this workspace is not proof no such run exists elsewhere. Train or locate and verify this required baseline.

`--alpha 1 --beta 0 --gamma 0` expresses the CE-only objective in this script, with the same label smoothing and frozen transmitter; it still computes the unused Teacher/KD terms. This is a proposed baseline recipe, **not an executed experiment**, and path/protocol issues must be resolved first.

BPE greedy and beam utilities exist. The notebook compares Teacher **greedy** with Student **beam size 10**, length penalty 0.7; its 0–29 dB sweep is a sentence demonstration, not validated dataset-level evidence. Final comparisons must use matched decoding and channel draws.

Available metrics include SacreBLEU sentence BLEU (exponential smoothing), corpus BLEU, and sentence-transformer cosine similarity. The default semantic model is `sentence-transformers/paraphrase-multilingual-mpnet-base-v2`; notebook alternatives exist. A mean of sentence BLEU is not corpus BLEU, and NLTK [0,1] scores must not be mixed with SacreBLEU [0,100] scores. Final metric signature, semantic model/revision, aggregation, seeds, latency protocol, and hardware are TBD.

| Required model | Receiver | Final publication parameter count | Validated final quality / latency |
|---|---|---:|---|
| Teacher | 8 layers / 16 heads | TBD | TBD |
| Student without KD | 4 layers / 8 heads | TBD | TBD |
| Student with KD | 4 layers / 8 heads | TBD | TBD |

## 9. Current publication implementation and remaining TBDs

Verified DeepSC head: `0d3b119c9215fb22f427d3aa7c9629c9a7cdc18c` (clean `BPE` working tree). `main_multi_vocab.py:243` now defaults to **MAX-LENGTH=67**: the reported Teacher 68 vs Student 67 mismatch is fixed. Historical Student checkpoints with a 96,000-position buffer remain historical and are not repaired or reused.

`main_multi_vocab.py:41–117,121–210,230–355` now trains on `train` and validates on `val`, shuffling only training. It samples uniform 2–18 dB per batch, validates at 8 dB using the common noise helper, and supports fixed SNR. Teacher optimization: Adam LR 1e-4, betas (0.9,0.98), epsilon 1e-8, weight decay 5e-4, batch 32, seed 42, fresh initialization. Teacher CE has **label smoothing 0.0** (`utils/train_utils.py:13–92,122–192`), token-weighted epoch CE, exp(CE) perplexity, and teacher-forced token accuracy. Best Teacher saves minimize validation CE; CSV includes SNR min/mean/max, validation SNR, LR and epoch time. Default budgets (Teacher 50, Student 10 epochs) are code defaults, not selected final best epochs.

Preprocessing `make_split_ids` (lines 213–234) now returns deterministic 70/15/15 partitions; training-text iteration (lines 241–274) excludes validation/test IDs. ID partition proportions precede length filtering. Tokenizer training is conditional: a pre-existing model can be reused unless `--force-retrain-tokenizer` is set. Publication preprocessing must force a new tokenizer or use a verified empty output location. Enabled source files currently include en_en/en_pt/en_es/en_fr; record the final tokenizer corpus composition, without treating it as multilingual reconstruction evidence. Normalization spaces punctuation; it does not remove it.

Remaining engineering/provenance issues:

- Both training scripts use the dataset's undated default directory, while preprocessing appends a date. Teacher vocabulary paths are additionally prefixed with `./data/train/`. A vocabulary override does not redirect dataset pickles. Record and align actual paths explicitly.
- ID splitting does not group normalized duplicate sentences. Check/group content before freezing the publication split; final overlap status is TBD.
- Preprocessing validates `train_ratio` but not `val_ratio` or their sum. Validate nonnegative ratios and total below one before publication preprocessing.
- Student validation uses `val`, but the unused `test_set` is still loaded at lines 401–403. Remove this unnecessary test dependency before Student training.
- Student Teacher filenames remain hardcoded `encoder_26.pth`/`decoder_26.pth`. Configure the newly selected Teacher explicitly; never rename an old checkpoint to satisfy these defaults.
- Teacher checkpoint files still lack a complete run manifest. Student metadata and CSV limitations described above remain.
- Current Student best selection uses the configured composite objective, unlike Teacher validation CE. Choose and document a common CE/KD Student selection rule before running the controlled comparison.

Final **TBD register** (no final artifacts selected in this update): corpus release/source hashes; normalized-duplicate policy and measured overlap; valid split manifest/IDs and accepted train/val/test counts; exact data directories; tokenizer corpus composition, invocation, hash and actual V; Teacher/CE/KD checkpoints, hashes, training budgets and best epochs; source revision and full training environment; initialization and repeated-run seed policy; common Student checkpoint-selection rule; evaluation SNR grid and independent channel/noise trials; matched decoding parameters and batching; corpus BLEU signature; semantic model/version/revision and normalization; retained optional metrics and uncertainty aggregation; hardware/GPU; inference warm-up, synchronization, sequence lengths and timing aggregation; final receiver/full-system parameters, compression, serialized size, memory, latency and FLOPs if reported; final quality and ablation results.

The manuscript now drafts Sections I and III–V, preserves Section II, and leaves Results, Abstract, and Conclusion pending. Follow [NEXT_TASK.md](NEXT_TASK.md) for the experiment sequence.
