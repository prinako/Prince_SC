# Experiments — Source of Truth for the KD Paper

Updated 2026-09-18 after direct inspection of the workspace DeepSC repository. **Code defaults, measured candidate artifacts, and final publication results are different evidence levels. No final run is selected here.** Exact references, loss equations, artifact hashes, checks, and prior-work comparison appear in [IMPLEMENTATION_AUDIT.md](IMPLEMENTATION_AUDIT.md).

## 1. Authority and scope

Implementation: `https://github.com/pesqSC/DeepSC.git`, branch `BPE`, inspected commit `b66afb331500f0e193b5d99f03a1f289b4f16e7f`. The authoritative Student training script is **`train_multi_vocab_one_student.py`**. Its supporting model, data, preprocessing, KD, channel, and evaluation code were inspected directly. The existing modified `test_BPE.ipynb` was inspected as working-tree evidence and preserved.

The paper studies **English text reconstruction with a frozen Teacher transmitter and a smaller receiver trained by KD**. Multilingual tokenizer infrastructure does not establish a multilingual KD contribution. LoRA and personalized multilingual receivers remain outside scope. `train_student.py`, `R_tr_kd.py`, and alternate losses do not define this configuration.

## 2. Architecture and trainability — code verified

| Component | Teacher | Student |
|---|---|---|
| Semantic encoder | 8 layers, 16 heads | Uses the frozen Teacher transmitter |
| Channel encoder | Linear 128→256, ReLU, Linear 256→16 | Same transmitted/noisy latent |
| Channel decoder | Linear 16→128→512→128, ReLU, residual, LayerNorm | Same architecture; independently trainable |
| Semantic decoder | 8 layers, 16 heads | 4 layers, 8 heads |
| Model / feed-forward width | 128 / 512 | 128 / 512 |
| Vocabulary / output head | Candidate V=96,000; untied Linear 128→V | Same vocabulary and head dimensions |
| Positional capacity in current script | 67 | 67 |
| Dropout default | 0.1 | 0.1 |
| During Student training | Transmitter and entire Teacher receiver frozen, eval mode | Entire receiver trainable, train mode |

The Student has no semantic/channel encoder. Both receivers see the **same realization** of the noisy transmitted tensor. Student initialization is fresh; the `--init-student-from-teacher` flag is unused. Attention projections retain width 128: reducing head count alone does **not** reduce their parameter count. The layer reduction supplies the parameter saving.

## 3. Candidate artifacts and measured parameters

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
- Features: final semantic decoder vectors, each normalized by `L2 norm + 1e-8`; average **squared deviation of their cosine similarity from one**, `(1−cos)²`, over non-PAD shifted targets. This is not raw feature MSE or channel-decoder matching.

Teacher forcing uses `trg[:, :-1]` to predict `trg[:, 1:]`; language and EOS tokens contribute, PAD does not. Transmitter and Teacher forward passes run under `no_grad`. Channel-decoder feature MSE is commented out. Exact equations and masking caveats are in the audit.

## 5. Data and BPE — measured candidate, not final split selection

`EurParallelDatasetBPE(args.en, split)` defaults to **en_en** and the hardcoded undated directory `./data/train/europarl_bpe`. The vocabulary flag alone does not redirect dataset files. Undated defaults are missing in this workspace; dated artifacts exist.

Candidate: `data/train/europarl_bpe/2026-09-12/`. Its JSON mapping matches all 96,000 SentencePiece entries; the tokenizer includes 256 byte pieces. PAD/START/END/UNK IDs are 0/1/2/3 and EN/PT/ES/FR IDs are 4/5/6/7.

| Artifact | Pairs | Unique source sequences | Stored sequence length |
|---|---:|---:|---:|
| `train_en_en.pkl` | **1,391,914** | 1,391,875 | 7–67 |
| `test_en_en.pkl` | **154,651** | 154,650 | 7–66 |

All inspected English pairs have identical source/target sequences and `[START, EN, content, END]` structure. **Eight unique token sequences occur in both splits.** The training script uses `test` for validation and best-checkpoint selection; it does not provide an untouched third split. These counts must not be described as independent train/validation/test evidence.

Preprocessing code uses Europarl JSON, NFKC/lowercase/punctuation/whitespace normalization, an ID-based 90/10 split with default seed 48, and SentencePiece BPE trained from training IDs across enabled language pairs. It filters content lengths 4–64 and adds three special tokens. Exact corpus release, preprocessing invocation, tokenizer training provenance, split repair, and final held-out test remain **TBD**. The separate 2026-09-11 tokenizer has V=32,000 and must not be mixed with the 96,000-token checkpoints.

## 6. Channel and SNR — validation discrepancy

Default channel: **Rayleigh**; AWGN/Rician are optional code paths, not established publication experiments. The transmitter emits 16 real values (8 complex symbols) per source position. Power normalization caps batch-wide RMS at one; it does not increase power below that threshold. Padding participates in this operation.

Rayleigh uses one complex fading coefficient for the whole batch, adds independent real Gaussian noise, and equalizes with the exact channel inverse (perfect CSI). No multiuser interference is implemented in this active path.

Training samples SNR uniformly in dB from **2 to 18 per batch**, or uses `--snr-db` in fixed mode. Its helper uses `sigma=1/sqrt(2*10^(SNR/10))`.

**Validation instead uses `sigma=10^(−args.snr_db/20)` and ignores `--val-snr-db`.** At the default label 8 dB, training sigma is 0.2815043, validation sigma is 0.3981072: validation noise variance is twice as large, a **3.0103 dB discrepancy under the training convention**. Do not state that training and validation use a consistent 8 dB conversion. Final SNR convention and evaluation grid remain TBD.

## 7. Optimization and checkpoint selection

Defaults: batch 32; 10 epochs; Adam LR 1e-4, betas (0.9, 0.98), epsilon 1e-8; gradient clipping 1.0; seed 42; workers 0. Weight decay is hardcoded **5e-4**, ignoring the parsed flag. No scheduler, resume state, or early stopping is configured.

Best checkpoint minimizes the validation **composite KD loss**, not BLEU. Epoch values are batch means, not corpus token-weighted means. CE_PPL is a smoothed-CE diagnostic, not standard corpus perplexity. CSV epoch numbering is one ahead of checkpoint numbering. Same-day saves can overwrite prior checkpoints and append to an existing CSV.

Student metadata stores epoch, validation loss, loss weights, temperature, and channel, but omits architecture, tokenizer/data hashes, SNR, seed, optimizer/RNG state, and implementation revision. Candidate `student_03.pth` records epoch 3 and the default KD weights/temperature/Rayleigh; this is insufficient to reconstruct its run.

Audit environment, **not proven training environment**: Python 3.12.13, PyTorch 2.13.0+cu130, NumPy 2.4.2, SentencePiece 0.2.2, SacreBLEU 2.6.0, NLTK 3.10.2, sentence-transformers 5.4.1. Publication hardware and run environment remain TBD.

## 8. Baselines, decoding, and metrics

No matching **4-layer/8-head CE-only Student** run was identified in the inspected scripts/artifact inventory. Sixteen Student checkpoints were examined; the sole 4-layer/96,000-token candidate has active KD metadata. Absence from this workspace is not proof no such run exists elsewhere. Train or locate and verify this required baseline.

`--alpha 1 --beta 0 --gamma 0` expresses the CE-only objective in this script, with the same label smoothing and frozen transmitter; it still computes the unused Teacher/KD terms. This is a proposed baseline recipe, **not an executed experiment**, and path/protocol issues must be resolved first.

BPE greedy and beam utilities exist. The notebook compares Teacher **greedy** with Student **beam size 10**, length penalty 0.7; its 0–29 dB sweep is a sentence demonstration, not validated dataset-level evidence. Final comparisons must use matched decoding and channel draws.

Available metrics include SacreBLEU sentence BLEU (exponential smoothing), corpus BLEU, and sentence-transformer cosine similarity. The default semantic model is `sentence-transformers/paraphrase-multilingual-mpnet-base-v2`; notebook alternatives exist. A mean of sentence BLEU is not corpus BLEU, and NLTK [0,1] scores must not be mixed with SacreBLEU [0,100] scores. Final metric signature, semantic model/revision, aggregation, seeds, latency protocol, and hardware are TBD.

| Required model | Receiver | Candidate parameter count | Validated final quality / latency |
|---|---|---:|---|
| Teacher | 8 layers / 16 heads | 26,922,752 | TBD |
| Student without KD | 4 layers / 8 heads | 25,864,448 if V=96,000 | Run not identified; TBD |
| Student with KD | 4 layers / 8 heads | 25,864,448 | TBD |

## 9. Before publication experiments and Methods drafting

1. Resolve paths and the Student positional-buffer/run-provenance discrepancy; designate exact Teacher, Student, vocabulary, and data artifacts.
2. Establish a consistent SNR/noise convention and separate validation from untouched test data; address token-sequence overlap.
3. Produce the matched CE-only baseline and KD ablations; fix shared decoding, SNR grid, seeds/channel trials, and metric signatures.
4. Measure reconstruction quality and deployment cost on the finalized protocol. Parameter savings alone establish neither quality preservation nor faster inference/storage savings.
5. Write Section III (System Model) and Section IV (KD Framework) from the verified architecture/objective, explicitly retaining unresolved experimental choices as TBD. Use the audit's prior-work matrix to bound the contribution. No final Results or novelty claim is authorized by this audit.
