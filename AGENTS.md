# Prince_SC — Codex Instructions

## Repository purpose

This repository contains the IEEE paper for the Knowledge Distillation (KD) stage of the Semantic Communication research project.

The paper focuses on this pipeline only:

**Semantic Communication Teacher -> Knowledge Distillation -> smaller Student model**

Do not merge this contribution with the separate multilingual/personalized-receiver work based on LoRA unless the user explicitly asks for cross-paper discussion.

## Implementation source of truth

The implementation repository is:

- `https://github.com/pesqSC/DeepSC.git`
- active implementation branch: `BPE`

When a paper statement depends on the actual model, KD loss, dataset/tokenization, channel configuration, checkpoint, or evaluation code, inspect the `BPE` branch before writing it as fact.

Relevant files currently include:

- `train_multi_vocab_one_student.py` — current BPE receiver-KD training path;
- `student.py` — receiver-only Student definition;
- `teacher.py` — Teacher helper functions;
- `models/transceiver.py` — DeepSC architecture;
- `models/tx_model.py` and `models/rx_model.py` — separated transmitter/receiver wrappers;
- `utils/kd_utils.py` — KD and feature-distillation losses;
- `utils/train_utils.py` — training/validation helpers;
- `dataset_multilingual.py` and `utils/bpe_utils.py` — BPE data/tokenization support.

Older/alternate KD scripts such as `train_student.py` and `R_tr_kd.py` exist on the branch. Do not silently mix their hyperparameters or loss definitions with the BPE configuration used for the final paper.

## Current task

The current active paper task is defined in:

- `docs/NEXT_TASK.md`

Read that file before starting substantive work. The Related Work task has been completed. The current follow-up is to audit the BPE implementation, confirm the exact Teacher/Student/KD configuration, and use it to refine the paper's novelty boundary against the closest prior work.

## Research objective

Investigate whether Knowledge Distillation can reduce the size and computational cost of a Transformer-based text semantic communication receiver while preserving semantic reconstruction quality across noisy channel conditions.

The core comparison should distinguish:

1. the original Teacher semantic communication model/receiver;
2. a smaller Student trained without KD, when such a baseline is available;
3. the smaller Student trained with KD.

Do not invent results, architecture dimensions, parameter counts, loss weights, SNR values, datasets, or checkpoints. If the repository/context files do not establish a value, mark it as `TBD` and locate the experimental record before using it as a paper fact.

## Required context

Before substantial paper work, read:

- `docs/NEXT_TASK.md`
- `docs/PROJECT_CONTEXT.md`
- `docs/PAPER_PLAN.md`
- `docs/EXPERIMENTS.md`
- `docs/RESEARCH_DECISIONS.md`
- `docs/LITERATURE_REVIEW_NOTES.md` when working on positioning or citations.

Keep those files updated when a research decision or confirmed experimental fact changes.

## Paper files

- Main LaTeX file: `PAPER-prince.tex`
- Bibliography: `paper.bib`
- IEEE class: `IEEEtran.cls`

Target format: IEEE conference paper using `IEEEtran`.

Do not modify `IEEEtran.cls` unless explicitly requested.

## Academic writing workflow

Use the repository skill at:

- `.agents/skills/academic-research-writer/SKILL.md`

Use it for literature review, Introduction, Related Work, citation verification, reference formatting, and academic revisions.

For literature claims:

- prefer peer-reviewed sources and published versions;
- prioritize IEEE Xplore, ACM Digital Library, major journals/publishers, and authoritative conference proceedings;
- use arXiv as a preprint source only when a published version is unavailable or when the preprint status is relevant;
- never fabricate a citation, DOI, venue, page number, result, or novelty claim;
- verify claims such as `first`, `novel`, or `state of the art` with a dedicated literature search before writing them.

## Scope boundary with the other paper

The broader research project also studies a shared semantic transmitter with personalized receivers, multilingual adaptation, and LoRA (including Portuguese and potentially other languages). That is a separate paper/research direction.

The active `BPE` implementation branch contains multilingual/BPE code because it is shared by the wider project. For this paper, use only the English/KD evidence needed to study Teacher-to-Student receiver compression unless the user explicitly expands the scope.

## Working branch

Current paper-development branch:

`paper-kd-student`

## Style and editing

- Write concise technical English suitable for IEEE.
- Preserve established terminology across sections.
- Prefer evidence-backed statements over broad claims.
- Keep equations and symbols consistent once introduced.
- Keep experimental methodology reproducible.
- Separate measured findings from hypotheses and planned evaluations.
- Do not rewrite author/funding information without explicit instruction.
