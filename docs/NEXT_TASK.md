# Current Codex Task — Update the KD Paper to the New Publication Protocol

## Goal

Update the `Prince_SC` IEEE paper repository to reflect the current, verified direction of the KD paper while the new experiments are being rerun from scratch.

This is a manuscript/context update task, not a Results-writing task.

The paper must focus only on:

**DeepSC-style Semantic Communication Teacher -> receiver-only compact Student -> Knowledge Distillation**

Do not mix this paper with the separate multilingual/personalized-receiver/LoRA research direction.

## Repositories and branches

Paper repository:

- `prinako/Prince_SC`
- branch: `paper-kd-student`

Implementation source of truth:

- `pesqSC/DeepSC`
- branch: `BPE`
- current verified head for this task: `546884e0035f17632a7131364ab3fedfe327dd8c`
- commit message: `refactor: Improve validation and training metrics reporting in main_multi_vocab.py, enhance SNR handling, and adjust model saving logic`

Before editing implementation-dependent manuscript text, inspect the current workspace copy of `DeepSC` directly. Do not rely only on older article-repo summaries.

## Mandatory context to read first

Read all of the following before editing:

1. `AGENTS.md`
2. `docs/PROJECT_CONTEXT.md`
3. `docs/PAPER_PLAN.md`
4. `docs/EXPERIMENTS.md`
5. `docs/RESEARCH_DECISIONS.md`
6. `docs/IMPLEMENTATION_AUDIT.md`
7. `docs/LITERATURE_REVIEW_NOTES.md`
8. `.agents/skills/academic-research-writer/SKILL.md`

Then inspect these current DeepSC files on branch `BPE`:

- `main_multi_vocab.py`
- `train_multi_vocab_one_student.py`
- `data_perprocess/preprocess_text_mu_bpe.py`
- `dataset_multilingual.py`
- `student.py`
- `teacher.py`
- `models/transceiver.py`
- `models/tx_model.py`
- `models/rx_model.py`
- `utils/train_utils.py`
- `utils/model_utils.py`
- `utils/kd_utils.py`
- `utils/bpe_utils.py`
- `metrics/metrics_bpe.py`

Older scripts such as `train_student.py`, `R_tr_kd.py`, and alternate KD losses are historical/alternate variants unless explicitly used for a comparison. Do not import their settings into the active paper configuration.

---

# 1. Research scope and research question

The paper studies whether Knowledge Distillation can produce a smaller Semantic Communication receiver/Student while preserving the reconstruction quality of the original DeepSC-style Teacher across noisy channels.

Use this working research question:

> Can Knowledge Distillation produce a substantially smaller Semantic Communication receiver/Student while preserving semantic reconstruction performance of the original DeepSC-style model across noisy channels?

The controlled comparison is intended to be:

1. original Teacher;
2. compact Student trained without KD (CE-only baseline);
3. the same compact Student trained with KD.

Do not claim that this is the first use of KD in semantic communication. Liu et al. 2024 and Eid et al. 2026 already overlap substantially with text/KD/DeepSC compression.

The likely distinguishing implementation angle is receiver-only compression while retaining the original/shared transmitter, but treat global novelty as provisional until fully supported by literature and final experimental evidence.

---

# 2. New publication data/training protocol — record this as a decision

The previous candidate artifacts based on the old 90/10 workflow are historical only and must not be mixed with the new publication run.

The new publication pipeline is:

`Europarl JSON -> normalization -> 70% train / 15% validation / 15% test -> train new SentencePiece BPE on TRAIN only -> train new Teacher from scratch -> freeze new Teacher -> train matched CE-only Student and KD Student -> select checkpoints on validation -> evaluate once on untouched test`

Required decisions to record in `docs/RESEARCH_DECISIONS.md`:

- data split: **70/15/15 train/validation/test**;
- deterministic split uses preprocessing seed 48 unless the final run explicitly changes it;
- SentencePiece BPE tokenizer is trained only on the 70% training partition;
- the Teacher is retrained from scratch with the new tokenizer/data;
- old 90/10 Teacher/Student checkpoints must not be mixed with the new final experiment;
- validation is used for checkpoint selection;
- test remains untouched until final evaluation;
- after the new Teacher is selected, train the same 4-layer/8-head receiver twice: CE-only and KD;
- Teacher/CE/KD final comparisons must use the same decoding procedure and matched channel conditions/draw protocol.

Do not record final split counts, tokenizer hash, best epochs, checkpoint filenames, or final quality numbers until the new preprocessing/training artifacts are actually verified.

---

# 3. Current verified Teacher-training implementation facts

Current `DeepSC/main_multi_vocab.py` now:

- uses `train` for training and `val` for validation;
- uses training `shuffle=True` and validation `shuffle=False`;
- samples training SNR in dB per batch in range mode;
- supports fixed SNR mode correctly;
- default train SNR range is 2--18 dB;
- default validation SNR is fixed at 8 dB and converted through `snr_to_noise`;
- reports token-weighted non-PAD CE, perplexity, and teacher-forced token accuracy;
- records train SNR min/mean/max, validation SNR, LR, and epoch time;
- chooses the best Teacher checkpoint by validation CE;
- default batch size is 32;
- optimizer is Adam with LR 1e-4, betas (0.9, 0.98), eps 1e-8, weight decay 5e-4;
- seed is 42;
- default channel is Rayleigh.

The corrected CE is averaged over valid non-PAD target positions. Training `token_accuracy` is teacher-forced token accuracy, not autoregressive decoded accuracy. Keep these metric names distinct in the paper and docs.

Do not use BLEU, semantic similarity, ROUGE, exact match, or decoded token accuracy as training-batch metrics. Those belong to autoregressive evaluation.

---

# 4. Current preprocessing facts and unresolved items

Current `preprocess_text_mu_bpe.py`:

- reads Europarl JSON;
- applies NFKC normalization, lowercase, punctuation spacing, and whitespace collapse;
- creates deterministic ID-based 70/15/15 train/val/test partitions by default;
- trains one SentencePiece BPE tokenizer from training IDs only;
- default vocabulary target is 96,000;
- PAD/START/END/UNK IDs are 0/1/2/3;
- language tokens EN/PT/ES/FR follow as user-defined symbols;
- content length filter is 4--64 BPE tokens;
- encoded sequences add START + language token + END, so maximum stored sequence length is 67.

Important unresolved implementation/provenance items — keep them visible as TBD/engineering issues rather than silently claiming they are solved:

1. `main_multi_vocab.py` currently defaults to `MAX-LENGTH=68`, while preprocessing and the Student path use maximum sequence length 67. The final publication run should align this to 67 before treating it as final.
2. Preprocessing writes to a dated output directory, while some loaders/scripts still rely on undated/default data paths. Final runs must record the exact data directory/tokenizer used.
3. The split is currently ID-based. Duplicate normalized sentences are not yet explicitly grouped before splitting, so duplicate-text leakage across partitions remains a risk until checked/fixed.
4. Preprocessing should validate `val_ratio` and the total split ratios robustly.
5. Teacher checkpoint metadata is still minimal; final run provenance should include implementation revision, data/tokenizer identity, architecture, SNR settings, seed, optimizer settings, and best validation metric.

Do not state that these unresolved items are fixed unless current DeepSC code proves it at the time Codex runs.

---

# 5. Current verified KD/Student implementation facts

The active KD path is `DeepSC/train_multi_vocab_one_student.py`.

Current intended architecture:

- Teacher semantic encoder: 8 Transformer layers, 16 heads;
- Teacher semantic decoder: 8 layers, 16 heads;
- Teacher `d_model=128`, `dff=512`;
- channel encoder: 128 -> 256 -> 16;
- channel decoder: 16 -> 128 -> 512 -> 128 with residual/LayerNorm structure;
- Student is receiver-only;
- Student semantic decoder: 4 layers, 8 heads;
- Student keeps `d_model=128`, `dff=512`;
- Student has its own channel decoder, semantic decoder, embedding, and output head;
- Teacher transmitter and Teacher receiver are frozen during Student training;
- both Teacher and Student receivers see the same noisy transmitted latent realization;
- Student is initialized fresh in the current active configuration.

Current KD objective:

`L = alpha * L_CE + beta * L_KD + gamma * L_feat`

with defaults:

- `alpha = 0.6`;
- `beta = 0.3`;
- `gamma = 0.1`;
- temperature `T = 2`;
- CE label smoothing = 0.1;
- `L_KD` is temperature-scaled KL(Teacher || Student) over non-PAD target positions;
- `L_feat` aligns final semantic-decoder features using squared cosine-to-one loss over non-PAD target positions.

For the matched no-KD baseline, use the same Student architecture/data/channel/optimizer/initialization protocol with:

- `alpha=1`;
- `beta=0`;
- `gamma=0`.

Do not report the old candidate checkpoints as final results. The current Student script still contains historical hardcoded Teacher filenames such as `encoder_26.pth`/`decoder_26.pth`; the final Student runs must use the newly trained Teacher instead.

---

# 6. Update article-repo context files

Update `docs/RESEARCH_DECISIONS.md` to include the new 70/15/15 publication protocol and the rule that the Teacher is retrained from scratch before matched CE/KD Students.

Update `docs/EXPERIMENTS.md` so it clearly separates:

- **historical/candidate evidence** from the old 90/10 artifacts;
- **current publication protocol** for the new 70/15/15 rerun;
- **verified current implementation facts** from DeepSC head `546884e0035f17632a7131364ab3fedfe327dd8c`;
- **TBD items requiring final run confirmation**.

Do not erase useful historical audit evidence; label it clearly so it cannot be mistaken for final publication evidence.

Update `docs/PROJECT_CONTEXT.md` if needed so the main project question and new pipeline are consistent with the above.

Update `AGENTS.md` current-task wording if needed so it no longer says the implementation audit itself is the immediate task. The audit is sufficiently mature to proceed with manuscript Sections III--V while experiments run.

After completing this task, rewrite `docs/NEXT_TASK.md` to point to the next concrete work item rather than leaving this execution brief as the permanent task.

---

# 7. Update `PAPER-prince.tex` now

The manuscript currently contains placeholder/generic content. Update the stable parts that do not require final experimental results.

## 7.1 Title and keywords

Replace the generic title with the current working title:

**Knowledge Distillation for Efficient Transformer-Based Semantic Communication**

Keep the title provisional if appropriate in comments, but do not leave `Generic Title: Securing Semantic Communication`.

Replace the unrelated `Federated Learning` keyword with relevant IEEE-style keywords such as:

- Semantic communications;
- knowledge distillation;
- model compression;
- Transformer;
- deep learning-enabled communications.

Do not modify the author or funding/acknowledgement information unless necessary for compilation.

## 7.2 Abstract and Results

Do not write a final abstract with invented or placeholder numerical claims.

Do not write numerical Results until validated final runs exist.

It is acceptable to leave the abstract as an explicit TODO comment or a short non-numerical draft only if clearly marked as provisional. Prefer writing the final abstract after Results.

## 7.3 Introduction

Replace the placeholder `Modern Machine Learning` with a concise IEEE-style Introduction covering:

1. motivation for semantic communication;
2. DeepSC-style learned text communication;
3. deployment cost of Transformer-based semantic receivers;
4. motivation for a compact receiver while preserving a shared/original transmitter;
5. KD as the transfer mechanism;
6. prior KD overlap and the need for a controlled Teacher vs CE Student vs KD Student comparison;
7. the paper's research question and contribution framing;
8. paper organization.

Do not claim `first`, `novel`, `state of the art`, or superiority without evidence.

Contribution statements must be framed as what the paper designs/evaluates, not as unverified performance claims.

## 7.4 Related Work

Section II is already substantially developed. Preserve its thematic structure and references unless a factual correction is needed.

The final positioning paragraph may be refined to reflect the verified receiver-only Student/shared-transmitter design, but global novelty remains provisional.

---

# 8. Write Section III — System Model

Add a full `\section{System Model}` after Related Work.

Use consistent notation for:

- input sentence/token sequence;
- semantic encoder;
- channel encoder;
- power normalization;
- channel realization/noise;
- noisy latent/channel representation;
- channel decoder;
- semantic decoder;
- vocabulary projection/output distribution;
- reconstructed sentence.

Represent the Teacher transmitter as the shared/frozen path used to produce the noisy latent that is supplied to both Teacher and Student receivers during KD training.

Describe Rayleigh as the current default publication channel configuration, but avoid implying AWGN/Rician results exist unless actually run.

The implementation uses perfect inverse channel equalization in the current Rayleigh helper; describe this carefully if retained in the System Model/Methodology.

Use the implementation to verify the exact normalization and channel equations before finalizing notation.

---

# 9. Add TikZ system architecture figure

Create a dedicated folder for TikZ paper figures if one does not already exist, for example:

`figures/tikz/`

Create the System Model figure as a standalone `.tex` fragment and include it from `PAPER-prince.tex` with `\input{...}` rather than embedding a large TikZ block directly in the main manuscript.

Figure concept:

`tokens -> Teacher semantic encoder -> Teacher channel encoder -> normalization/channel -> shared noisy latent -> {Teacher receiver, Student receiver}`

Show that:

- the transmitter is shared/frozen during Student/KD training;
- Teacher receiver is frozen;
- Student receiver is trainable;
- Teacher and Student receive the same noisy latent realization.

Keep the figure IEEE-column friendly and visually simple.

Add required TikZ packages/libraries only as needed.

---

# 10. Write Section IV — Knowledge Distillation Framework

Add a full `\section{Knowledge Distillation Framework}`.

Describe the actual active implementation, not generic KD alternatives.

Include equations for:

1. hard-target CE reconstruction loss with non-PAD masking and label smoothing;
2. temperature-scaled KL logit distillation;
3. final semantic-decoder feature alignment using normalized features and squared `(1-cos)^2` loss;
4. combined weighted objective.

Use:

`alpha=0.6, beta=0.3, gamma=0.1, T=2`

as current implementation defaults/configuration facts, not as results.

Explain teacher forcing (`trg[:, :-1]` predicts `trg[:, 1:]`) and that PAD target positions are excluded from the losses.

Clearly distinguish Teacher/Student roles and frozen/trainable components.

---

# 11. Add TikZ KD-training figure

Create a second TikZ fragment under the same figure folder and include it from Section IV.

Show:

- ground-truth target -> CE loss;
- Teacher logits -> KL KD loss against Student logits;
- Teacher final decoder features -> feature loss against Student final decoder features;
- weighted sum -> Student update;
- no gradient update to Teacher/transmitter.

Keep formulas in the text; the figure should explain flow, not duplicate all equations.

---

# 12. Draft Section V — Experimental Methodology

Add a `\section{Experimental Methodology}` with only verified or deliberately selected protocol facts.

Stable items currently available:

- Europarl data source;
- NFKC/lowercase/punctuation-spacing/whitespace normalization from preprocessing;
- 70/15/15 train/validation/test publication protocol;
- SentencePiece BPE trained on train only;
- target BPE vocabulary 96,000 unless final tokenizer produces a different actual size;
- content-length filter 4--64 BPE tokens;
- expected maximum encoded sequence length 67;
- Teacher architecture: 8 layers/16 heads, `d_model=128`, `dff=512`;
- Student architecture: 4 layers/8 heads, same model/FFN width;
- Rayleigh default channel;
- train SNR range 2--18 dB per batch;
- validation SNR 8 dB;
- Adam LR 1e-4, betas (0.9,0.98), eps 1e-8, weight decay 5e-4;
- Teacher seed 42;
- current Teacher batch size 32;
- Teacher checkpoint selection by validation CE;
- CE-only and KD Student must use the same architecture and matched training/evaluation protocol.

Mark the following as TBD until verified from final runs/artifacts:

- exact accepted train/val/test pair counts;
- exact tokenizer/vocabulary hash and actual final vocabulary size if different;
- final Teacher best epoch/checkpoint;
- final CE Student best epoch/checkpoint;
- final KD Student best epoch/checkpoint;
- final test SNR grid;
- number of independent channel trials/seeds per SNR;
- final decoder choice (greedy should be the main default unless the team deliberately selects beam; whichever is chosen must be matched across models);
- final BLEU signature/configuration;
- semantic-similarity model/revision;
- hardware/GPU;
- inference latency protocol;
- FLOPs if reported.

Do not include test results or use the test split for checkpoint selection.

---

# 13. Metrics terminology for the paper

Training/validation optimization metrics:

- `CE` = non-PAD token cross-entropy;
- `PPL` = `exp(CE)` for the current unsmoothed Teacher CE objective;
- `teacher-forced token accuracy` = argmax token accuracy under teacher forcing.

Final autoregressive reconstruction metrics should be treated separately:

- SacreBLEU corpus BLEU;
- chrF++;
- semantic similarity;
- ROUGE-L if retained;
- exact match;
- decoded token accuracy if retained.

Do not call the training teacher-forced token accuracy the same thing as decoded autoregressive token accuracy.

For paper BLEU-vs-SNR plots, compute corpus BLEU per SNR/trial from grouped hypotheses/references; do not report an average of sentence BLEU as corpus BLEU.

Use the same decoding method for Teacher, CE Student, and KD Student.

---

# 14. Results and claims boundary

Do not add numerical Results, compression claims from old candidate artifacts, latency claims, storage claims, or final quality conclusions yet.

Old measured candidate parameter counts may remain in audit/context docs if clearly labeled historical/candidate, but do not present them as final publication results because the new tokenizer/Teacher pipeline is being rerun.

Do not write the final Abstract or Conclusion until validated final experiments exist.

---

# 15. Deliverables for this Codex task

Update at minimum:

- `AGENTS.md` — current-task wording only if needed;
- `docs/PROJECT_CONTEXT.md` — new protocol/context if needed;
- `docs/RESEARCH_DECISIONS.md` — record the 70/15/15 + retrain-from-scratch decision;
- `docs/EXPERIMENTS.md` — current implementation/protocol vs historical candidate evidence;
- `docs/NEXT_TASK.md` — replace this execution brief with the next concrete task after work completes;
- `PAPER-prince.tex` — title, keywords, Introduction, Section III, Section IV, draft Section V;
- TikZ figure fragments under a dedicated figure folder, referenced from `PAPER-prince.tex`.

Update `paper.bib` only if the new manuscript text requires a citation that is not already present and the source can be verified. Do not invent references.

Do not modify `IEEEtran.cls`.

---

# 16. Validation before commit

Before committing:

1. verify all implementation-dependent statements against the current DeepSC `BPE` branch;
2. check that no 90/10 candidate artifact is described as final publication evidence;
3. check that no LoRA/multilingual personalized-receiver contribution leaked into this KD paper;
4. check equation notation consistency across Sections III--V;
5. verify all citations used in new prose exist in `paper.bib` and support the claims;
6. compile `PAPER-prince.tex` if the environment supports LaTeX;
7. fix LaTeX/TikZ compilation errors introduced by the edits;
8. report exactly which files changed and which items remain TBD.

Commit the paper/context updates on `paper-kd-student` with a clear message such as:

`paper: update system model KD framework and 70-15-15 protocol`

Do not commit generated LaTeX build artifacts unless the repository already tracks them intentionally.
