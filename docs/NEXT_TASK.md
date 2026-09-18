# Current Codex Task — Audit the BPE KD Implementation

## Goal

Use the implementation repository `https://github.com/pesqSC/DeepSC.git`, branch `BPE`, to establish the exact Teacher -> KD -> Student configuration that will support the IEEE paper.

The Related Work task is already complete. Do not expand Section II unless a factual correction is needed. The immediate objective is to turn implementation details into a defensible Method/System Model and to clarify how this work differs from the closest prior KD-based semantic communication papers.

## Required preparation

Read:

1. `AGENTS.md`
2. `docs/PROJECT_CONTEXT.md`
3. `docs/PAPER_PLAN.md`
4. `docs/EXPERIMENTS.md`
5. `docs/RESEARCH_DECISIONS.md`
6. `docs/LITERATURE_REVIEW_NOTES.md`

Then inspect the `BPE` branch of `pesqSC/DeepSC`.

## Primary implementation files

Prioritize:

- `train_multi_vocab_one_student.py`
- `student.py`
- `teacher.py`
- `models/transceiver.py`
- `models/tx_model.py`
- `models/rx_model.py`
- `utils/kd_utils.py`
- `utils/train_utils.py`
- `dataset_multilingual.py`
- `utils/bpe_utils.py`

Use `train_student.py`, `R_tr_kd.py`, and other KD scripts only to identify historical/alternate variants. Do not mix their hyperparameters into the active BPE configuration.

## Verified starting point

The current BPE one-Student path indicates:

- frozen/shared Teacher transmitter;
- frozen Teacher receiver;
- receiver-only Student;
- Teacher: 8 Transformer layers, 16 attention heads, `d_model=128`, `dff=512`;
- Student: 4 decoder layers, 8 attention heads, `d_model=128`, `dff=512`;
- BPE vocabulary and maximum sequence length 67;
- Rayleigh channel;
- Student-training SNR sampled from 2--18 dB;
- validation SNR default 8 dB;
- CE + KL-logit KD + semantic-decoder feature alignment;
- loss weights 0.6 / 0.3 / 0.1;
- KD temperature 2.0;
- CE label smoothing 0.1.

Treat these as implementation facts for the current candidate configuration, not as final publication results.

## Tasks

### 1. Architecture audit

Confirm from code:

- exact DeepSC Teacher architecture;
- exact Student receiver architecture;
- which components are shared/frozen;
- which components are trainable;
- dimensions of the channel encoder/decoder bottleneck;
- whether the Student differs from the Teacher only by decoder depth/heads or also by other dimensions.

Record exact file/function references in notes.

### 2. Parameter/compression audit

Using the exact BPE vocabulary/configuration used by the candidate run, compute:

- full Teacher parameter count;
- Teacher receiver-only parameter count;
- Student receiver parameter count;
- reduction percentage and compression ratio;
- model checkpoint size if measured from actual files.

Do not estimate from architecture alone if vocabulary size/checkpoint details are missing. Mark missing inputs `TBD`.

### 3. KD objective audit

Confirm the active BPE loss mathematically:

- hard-target CE term;
- temperature-scaled KL term;
- decoder-feature alignment term;
- masking behavior;
- label smoothing;
- loss weights;
- temperature.

Distinguish active code from commented/legacy feature losses.

### 4. Data and channel audit

Confirm:

- exact dataset used for the English KD study;
- BPE preprocessing/tokenizer/vocabulary size;
- split sizes;
- maximum sequence length;
- training SNR sampling;
- validation/evaluation SNRs;
- channel model.

Do not import multilingual/LoRA claims into this paper merely because the BPE branch contains multilingual infrastructure.

### 5. Baseline audit

Determine whether a 4-layer/8-head Student trained **without KD** already exists. If not, flag it as a required experiment.

The paper should ideally compare:

1. Teacher;
2. same compact Student architecture trained without KD;
3. same compact Student architecture trained with KD.

### 6. Novelty comparison

Compare the verified implementation directly against:

- Liu et al., IEEE TWC 2024 (`liu2024kdsemcom`);
- Eid et al., 2026 preprint (`eid2026krumdeepsc`).

Build a small comparison matrix covering at least:

- modality;
- Teacher/Student placement;
- receiver-only vs end-to-end compression;
- architecture reduction;
- KD signals/losses;
- channel setting;
- training/evaluation SNR treatment;
- baselines;
- deployment/complexity metrics.

Do not claim novelty until a concrete difference is supported.

## Deliverables

Update:

- `docs/EXPERIMENTS.md` with any newly verified facts;
- `docs/RESEARCH_DECISIONS.md` if a final experimental configuration is selected.

Create:

- `docs/IMPLEMENTATION_AUDIT.md`

The audit should contain exact code-derived evidence and a `TBD / requires run confirmation` section.

Do **not** write numerical Results into `PAPER-prince.tex` unless they come from validated final runs.

After the audit is complete, the next paper-writing step will be Section III (System Model) and Section IV (Knowledge Distillation Framework) based on the verified BPE implementation.
