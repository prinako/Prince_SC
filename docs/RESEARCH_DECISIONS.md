# Research Decisions

This file records decisions that should persist across ChatGPT/Codex sessions.

## 2026-09-18 — Separate the KD paper from the multilingual/LoRA paper

**Decision:** `Prince_SC` will focus on the Semantic Communication model compressed through Knowledge Distillation: Teacher -> Student.

**Excluded from the present contribution:** LoRA-based language adaptation, personalized multilingual receivers, and the shared-TX/multi-receiver contribution.

**Reason:** separating the stages makes the research question, novelty, baselines, and experimental evidence easier to isolate and defend in an IEEE paper.

## 2026-09-18 — Use an IEEE conference LaTeX article repository

**Decision:** use the existing `IEEEtran` conference skeleton in `PAPER-prince.tex` and bibliography in `paper.bib`.

`IEEEtran.cls` is treated as template infrastructure and should not be edited casually.

## 2026-09-18 — Dedicated working branch

**Decision:** develop the KD article on branch `paper-kd-student` rather than modifying `main` directly.

## 2026-09-18 — Academic-writing agent skill

**Decision:** add the `academic-research-writer` skill from `endigo/claude-skills` to the repository.

The skill includes academic-writing guidance, source-verification guidance, IEEE citation guidance, a paper template, and an IEEE reference formatter.

For Codex, expose the skill under `.agents/skills/academic-research-writer/`. The earlier Claude-compatible copy under `.claude/skills/academic-research-writer/` may remain in place.

## 2026-09-18 — Persistent Codex project context

**Decision:** store research context in version-controlled files (`AGENTS.md` and `docs/*.md`) so Codex sessions do not depend on access to prior ChatGPT conversation history.

## 2026-09-18 — Related Work positioning after dedicated search

**Finding:** KD-based Transformer text semantic communication and compact no-KD comparisons already appear in Liu et al., IEEE TWC 2024 (doi:10.1109/TWC.2023.3336941). A closely overlapping DeepSC compression preprint by Eid et al. also appeared in September 2026 (arXiv:2609.13405).

**Writing decision:** Section II describes Prince_SC as an investigation of the reconstruction-quality/model-complexity trade-off. Its specific novelty remains provisional pending confirmation of the implementation and experiments; text modality, Transformer use and noisy-channel evaluation alone do not distinguish the paper.

**Evidence:** See `docs/LITERATURE_REVIEW_NOTES.md` for verified metadata, claim support, search limitations and follow-up checks.

## 2026-09-18 — DeepSC implementation source is the BPE branch

**Decision:** use `https://github.com/pesqSC/DeepSC.git`, branch `BPE`, as the implementation source of truth for the KD paper.

The paper repository (`Prince_SC`) is for manuscript/context; the DeepSC BPE branch is where architecture, KD objective, BPE preprocessing, channel configuration, checkpoints, and evaluation behavior must be verified.

**Current candidate training path:** `train_multi_vocab_one_student.py`.

The code currently indicates a receiver-only Student trained from a frozen/shared Teacher transmitter and frozen Teacher receiver. The candidate BPE architecture reduces the Teacher receiver from 8 Transformer layers / 16 heads to a 4-layer / 8-head Student while keeping `d_model=128` and `dff=512`.

The candidate BPE KD objective combines hard-target CE, temperature-scaled KL logit distillation, and semantic-decoder feature alignment with default weights 0.6/0.3/0.1 and temperature 2.0. The default training channel is Rayleigh, with Student training SNR sampled from 2--18 dB.

These are **verified implementation facts for the current code path**, not final publication results. Final checkpoint choice, parameter counts, compression ratio, dataset split sizes, evaluation grid, decoding method, and metrics remain to be confirmed.

**Important:** older/alternate KD scripts on the same branch (`train_student.py`, `R_tr_kd.py`) use different architectures/loss configurations. Do not merge those settings into the current BPE method unless intentionally presenting an ablation or history.

## Research integrity rules

- Do not fabricate experimental values or citations.
- Distinguish confirmed results from plans/TBDs.
- Verify novelty claims with a dedicated literature search.
- Prefer the final published version of a reference over a preprint when available.
- Keep the KD paper contribution separate from the multilingual/LoRA contribution.
- Treat code defaults as candidate configuration until the exact publication run/checkpoints are designated.

## 2026-09-18 — Publication rerun supersedes historical candidates

**70/15/15 → new train-only BPE → new Teacher from scratch → matched CE Student / KD Student → untouched test evaluation.**

The split is selected policy, not a completed-run result. Use preprocessing seed 48 and common record IDs across relevant data files. Train a fresh tokenizer from training IDs only, select the retrained Teacher on validation, then freeze it for both Student runs. Use the same four-layer/eight-head Student architecture, smoothing, optimizer, seed/initialization policy, budget, data, channel conditions, and decoding across CE-only and KD experiments. The final test partition is reserved until the protocol and checkpoints are fixed. Old 90/10 Teacher/Student/tokenizer artifacts cannot enter this pipeline.

**Decision:** the new Teacher must be trained from scratch. The CE-only Student uses alpha/beta/gamma=1/0/0 with the same 0.1 Student CE smoothing as KD. Corpus BLEU is aggregated over grouped test hypotheses/references per SNR/trial; mean sentence BLEU is not a substitute. Decoding and channel draws must be matched. Final run identities, counts, metric signatures and measurement settings remain TBD in EXPERIMENTS.md.

**Implementation confirmation:** DeepSC `0d3b119c9215fb22f427d3aa7c9629c9a7cdc18c` uses train/val loaders and Teacher MAX-LENGTH=67. This fixes the earlier 68/67 configuration issue. Remaining path, duplicate-content, split-ratio validation, unused test loading, historical Teacher filename, and run-manifest issues are tracked in IMPLEMENTATION_AUDIT.md. No final run or numerical result was selected.
