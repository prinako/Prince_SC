# Prince_SC — Codex Instructions

## Repository purpose

This repository contains the IEEE paper for the Knowledge Distillation (KD) stage of the Semantic Communication research project.

The paper focuses on this pipeline only:

**Semantic Communication Teacher -> Knowledge Distillation -> smaller Student model**

Do not merge this contribution with the separate multilingual/personalized-receiver work based on LoRA unless the user explicitly asks for cross-paper discussion.

## Current task

The current active paper task is defined in:

- `docs/NEXT_TASK.md`

Read that file before starting substantive work. At present, the task is to research, verify, and draft **Section II: Related Work** only, including literature notes and verified bibliography updates.

## Research objective

Investigate whether Knowledge Distillation can reduce the size and computational cost of a Transformer-based text semantic communication model while preserving semantic reconstruction quality across noisy channel conditions.

The core comparison should distinguish:

1. the original Teacher semantic communication model;
2. a smaller Student trained without KD, when such a baseline is available;
3. the smaller Student trained with KD.

Do not invent results, architecture dimensions, parameter counts, loss weights, SNR values, datasets, or checkpoints. If the repository/context files do not establish a value, mark it as `TBD` and ask for or locate the experimental record before using it as a paper fact.

## Required context

Before substantial paper work, read:

- `docs/NEXT_TASK.md`
- `docs/PROJECT_CONTEXT.md`
- `docs/PAPER_PLAN.md`
- `docs/EXPERIMENTS.md`
- `docs/RESEARCH_DECISIONS.md`

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

For this repository, keep the contribution centered on model compression/knowledge transfer from the SC Teacher to the Student.

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
