# DeepSC `all` Branch Synchronization — 2026-09-19

This note records a direct audit of `pesqSC/DeepSC`, branch `all`, for the KD paper in `Prince_SC`.

## Revision relationship

- Inspected `DeepSC:all` HEAD: `4a67a4c3131908b1b47ce9979d9093c5b7aee370`.
- Previous paper audit used `DeepSC:BPE` HEAD: `0d3b119c9215fb22f427d3aa7c9629c9a7cdc18c`.
- `all` is 7 commits ahead of that BPE revision and 0 commits behind.
- The changes between those revisions are limited to:
  - `data_perprocess/process_multilingual_deepsc.py` (new synchronized preprocessing);
  - `dataset_multilingual.py` (synchronized multilingual BPE loader/collate support);
  - `main_multi_vocab.py` (new batch-interleaved Teacher training pipeline).
- The authoritative KD script for this paper, `train_multi_vocab_one_student.py`, is unchanged by this BPE-to-`all` delta.

## Paper scope remains unchanged

`Prince_SC` remains a receiver-only Knowledge Distillation paper: trained Teacher semantic communication system -> frozen Teacher transmitter/receiver during Student training -> compact receiver. The wider multilingual and LoRA/personalized-receiver contribution is not part of this paper.

The new synchronized preprocessing and multilingual Teacher infrastructure may be used to build the shared tokenizer/data artifacts and Teacher, but the publication comparison in this repository remains English reconstruction unless the research scope is explicitly changed.

## Updated preprocessing facts from `all`

The current synchronized preprocessor uses:

- a deterministic 70/15/15 train/validation/test split;
- preprocessing seed 48;
- ID-based alignment across `en_en`, `en_pt`, `en_es`, and `en_fr`;
- duplicate-ID rejection before indexing;
- normalized English-source equality checks across aligned files;
- NFKC normalization, lowercasing, trimming, and whitespace collapse;
- a shared SentencePiece BPE tokenizer trained only from aligned training IDs;
- requested vocabulary size 32,000;
- PAD/START/END/UNK IDs 0/1/2/3 and registered EN/PT/ES/FR control tokens;
- byte fallback enabled and character coverage 1.0;
- minimum content length 4 BPE tokens;
- maximum **complete encoded sequence length 64**.

With that full-length cap, the source sequence has the form

`<START> content <END>`

and therefore allows at most 62 content pieces. Receiver targets have the form

`<START> <LANG> content <END>`

and therefore allow at most 61 content pieces. The preprocessor saves split IDs, alignment/processing statistics, vocabulary/tokenizer metadata, and preprocessing configuration for reproducibility.

This supersedes the earlier paper draft text that described a requested 96,000-token vocabulary, punctuation-spacing normalization, content length 4–64, and a resulting stored length of 67.

## Updated Teacher-training facts from `all`

`main_multi_vocab.py` now uses the synchronized dataset and, by default:

- Teacher Transformer: 8 layers, 16 heads, `d_model=128`, `dff=512`, dropout 0.1;
- Adam learning rate `1e-4`, betas `(0.9, 0.98)`, epsilon `1e-8`, weight decay `5e-4`;
- batch size **64**;
- seed **48**;
- 50 epochs as a code default, not a selected publication budget/result;
- Rayleigh channel by default;
- training SNR sampled uniformly in dB from 2 to 18 per batch;
- validation SNR fixed at 8 dB;
- unsmoothed Teacher CE over non-PAD tokens;
- best Teacher checkpoint selected by overall validation CE;
- per-language train/validation CE and token-accuracy logging;
- a `run_config.json` manifest that records the resolved data/tokenizer paths and preprocessing configuration.

For the KD paper, multilingual Teacher outputs are infrastructure rather than a new contribution. Publication claims should use only the English reconstruction evidence unless the paper scope is explicitly changed.

## KD/Student configuration: unchanged and not yet synchronized with the new data interface

`train_multi_vocab_one_student.py` remains the authoritative current KD implementation for `Prince_SC`. Its current defaults are:

- Teacher architecture expected by the script: 8 layers / 16 heads;
- Student receiver: 4 layers / 8 heads, `d_model=128`, `dff=512`, dropout 0.1;
- Student batch size **32**;
- Student seed 42;
- 10 epochs as a code default;
- Adam learning rate `1e-4`, betas `(0.9, 0.98)`, epsilon `1e-8`, weight decay `5e-4`;
- gradient clipping at norm 1;
- Rayleigh training SNR 2–18 dB and validation at 8 dB;
- CE label smoothing 0.1;
- KD temperature 2;
- loss weights `(alpha,beta,gamma)=(0.6,0.3,0.1)`;
- KL logit distillation from Teacher to Student, multiplied by `T^2` and masked at PAD positions;
- final semantic-decoder feature alignment using the squared deviation of cosine similarity from one.

The CE-only baseline should use the identical Student architecture and training protocol with `(alpha,beta,gamma)=(1,0,0)` so that CE-only vs. KD isolates Teacher supervision.

Different Teacher and Student batch sizes are methodologically acceptable because they are different training stages. The controlled requirement is that the CE-only Student and KD Student use the same batch size and other optimization conditions. Therefore the current code supports reporting Teacher batch 64 and Student batch 32, unless the Student script is intentionally changed before the final runs.

## Required implementation synchronization before final publication runs

The new Teacher/data pipeline and current KD script are not yet fully compatible. Before the CE-only and KD publication runs:

1. Update Student data loading to consume the finalized synchronized preprocessing output (or an explicitly derived English-only view of the exact same split/tokenizer artifacts).
2. Align Student positional capacity/max length with the selected preprocessing configuration; the new full-sequence cap is 64, while the current KD script still defaults to 67.
3. Replace the hardcoded historical Teacher files `encoder_26.pth` / `decoder_26.pth` with the newly selected validation-CE Teacher checkpoint.
4. Remove the unused Student-training dependency on the test split; test must remain untouched until the final protocol/checkpoints are fixed.
5. Match CE-only and KD Student batch size, initialization policy, seed policy, training budget, optimizer, data, SNR/channel protocol, and checkpoint-selection rule.
6. Record the final run revision, tokenizer/data hashes, accepted split counts, actual vocabulary size, checkpoint hashes, and environment/hardware.

## Manuscript changes supported now

The methodology can already be corrected to state the selected/current pipeline values that are explicit in `DeepSC:all`:

- requested BPE vocabulary: 32,000 (actual final size/hash remain run-dependent);
- preprocessing: NFKC + lowercase + trimming/whitespace collapse (not punctuation spacing);
- maximum complete sequence length: 64;
- Teacher batch size: 64;
- Teacher seed: 48;
- Teacher SNR/channel/optimizer settings above.

The manuscript should **not** yet state that the final CE-only/KD Student runs use batch 64. The current authoritative KD script uses 32; if that is changed to 64 before both Student runs, update the paper and experimental manifest after the change is committed and the actual runs are recorded.

## Evaluation support present in the repository

The BPE evaluation utilities implement corpus/sentence BLEU, chrF++, multilingual SentenceTransformer cosine similarity, ROUGE-L, exact match, and BPE token accuracy. The final paper metric set, SacreBLEU signature, semantic-model revision, SNR grid, trial count, decoding configuration, and uncertainty aggregation remain publication-run decisions rather than established results.
