# Current Codex Task — Related Work

## Goal

Start the IEEE paper from **Section II: Related Work** for the Knowledge Distillation (KD) semantic communication paper.

For this task, focus only on literature research, source verification, bibliography preparation, and drafting the Related Work section. Do not write Results, Abstract, Conclusion, or invent experimental details.

## Required preparation

Before writing, read:

1. `AGENTS.md`
2. `docs/PROJECT_CONTEXT.md`
3. `docs/PAPER_PLAN.md`
4. `docs/EXPERIMENTS.md`
5. `docs/RESEARCH_DECISIONS.md`
6. `.agents/skills/academic-research-writer/SKILL.md`

Use the `academic-research-writer` skill and its source-verification and IEEE-citation guidance.

## Research question for this literature review

The Related Work section must establish what has already been done in:

- Transformer/deep-learning-based text semantic communication;
- efficient, lightweight, or compressed semantic communication models;
- Knowledge Distillation for model compression, especially Transformers and sequence models;
- Knowledge Distillation used directly in semantic communication or closely related communication-system settings;
- the remaining research gap that motivates evaluating a compact Student distilled from a larger text semantic communication Teacher.

The paper's scope is strictly:

**Semantic Communication Teacher -> Knowledge Distillation -> smaller Student model**

Do not mix the paper contribution with the separate LoRA/multilingual/personalized-receiver research direction.

## Literature-search strategy

Perform a dedicated search before making any novelty claim.

Use combinations of terms such as:

- `semantic communication knowledge distillation`
- `text semantic communication knowledge distillation`
- `DeepSC knowledge distillation`
- `semantic communications model compression`
- `lightweight semantic communication transformer`
- `efficient semantic communication neural network`
- `semantic communication pruning distillation compression`
- `knowledge distillation transformer compression`
- `knowledge distillation sequence generation transformer`
- `teacher student semantic communication`

Prioritize:

- IEEE Xplore;
- ACM Digital Library;
- peer-reviewed journal and conference versions;
- reputable publishers and proceedings;
- published versions over arXiv when both exist.

Foundational older papers are allowed when necessary, especially for Knowledge Distillation and Transformer foundations. For the semantic-communication state of the art, prioritize recent literature.

## Source-verification rules

For every source used in the paper:

- verify the title;
- verify the complete author list or correct IEEE-author representation;
- verify venue;
- verify publication year;
- verify volume/issue/pages when applicable;
- verify DOI when available;
- confirm whether the source is peer-reviewed or a preprint;
- confirm that the paper actually supports the claim attached to its citation.

Do not copy citation metadata from an unverified secondary webpage when the publisher or proceedings record is available.

Do not fabricate or guess BibTeX fields.

## Novelty / research-gap rule

Do **not** write statements such as:

- `This is the first work...`
- `No prior work has...`
- `Knowledge Distillation has not been applied to semantic communication...`

unless the dedicated literature search provides sufficient evidence for that claim.

If related KD-for-semantic-communication papers exist, describe them accurately and distinguish this paper through concrete differences such as:

- text modality;
- Transformer architecture;
- Teacher/Student placement;
- receiver/model compression objective;
- training objective;
- evaluation across noisy channels;
- quality-efficiency trade-off.

If the exact novelty boundary is still uncertain, use cautious wording such as `This work investigates...` and document the unresolved novelty question in the literature notes.

## Required thematic structure

Draft Section II thematically, not as a paper-by-paper list.

Recommended flow:

### 1. Text Semantic Communication

Cover the evolution from conventional bit-level communication toward learned semantic representations, with DeepSC-style Transformer text semantic communication as the principal baseline/foundation.

Explain only what is needed to position this paper.

### 2. Efficient / Lightweight Semantic Communication

Review work aimed at reducing computation, memory, transmission overhead, model size, or deployment cost in semantic communication.

Distinguish model-side efficiency from communication-side bandwidth/channel efficiency.

### 3. Knowledge Distillation and Transformer Compression

Introduce KD as Teacher-to-Student knowledge transfer and review strong Transformer/sequence-model distillation work relevant to this architecture.

Do not over-expand into a generic KD survey; connect each cited method to why KD is appropriate for the semantic communication Student.

### 4. KD in Semantic / Communication Systems and Research Gap

Identify papers that use KD directly in semantic communication or closely related learned communication systems.

Conclude with the specific gap this paper will address, using only claims supported by the search.

The final IEEE section may use subsections if they improve clarity, but keep it concise enough for a conference paper.

## Deliverable 1 — Literature notes

Create or update:

`docs/LITERATURE_REVIEW_NOTES.md`

For each candidate source, record at least:

- BibTeX/citation key;
- full title;
- authors;
- year;
- venue;
- DOI or authoritative source URL when available;
- peer-reviewed / preprint status;
- theme/category;
- 2-4 sentence relevance summary;
- exact claim(s) it can support in the paper;
- whether it should be cited in Section II;
- any uncertainty or verification issue.

Also include a short `Research-gap assessment` summarizing what the search supports and what remains uncertain.

## Deliverable 2 — Bibliography

Update `paper.bib` with the verified sources actually cited in the Related Work section.

Requirements:

- preserve useful existing entries;
- avoid duplicate entries for the same publication;
- prefer stable, descriptive citation keys;
- use published metadata when available;
- include DOI where verified;
- do not add papers that are not relevant merely to increase reference count.

## Deliverable 3 — IEEE Related Work section

Update `PAPER-prince.tex` by adding a polished:

`\section{Related Work}`

The section must:

- use concise IEEE-style technical English;
- synthesize literature rather than list papers;
- cite every externally sourced factual claim;
- clearly connect prior work to the Teacher -> KD -> Student problem;
- distinguish semantic-communication efficiency from general model compression;
- end with a carefully supported research-gap paragraph;
- avoid any experimental result or parameter value that is still `TBD`.

Do not draft the Abstract or Results in this task.

## Quality target

Prefer a smaller set of highly relevant, verified sources over a long bibliography of weakly related papers. The full paper will eventually need broader coverage, but this task should establish a defensible Related Work foundation.

Before finishing:

- cross-check every `\cite{...}` against `paper.bib`;
- ensure every new bibliography entry is cited or intentionally documented for later use;
- check for duplicate papers;
- verify that the Related Work does not claim contributions belonging to the LoRA/multilingual paper;
- note any unresolved literature gap explicitly in `docs/LITERATURE_REVIEW_NOTES.md` rather than hiding uncertainty.

## Expected output from Codex

After making the changes, report:

1. files changed;
2. number and categories of verified sources reviewed;
3. which sources were added to `paper.bib`;
4. the final thematic structure of Section II;
5. whether the search found prior KD work directly in semantic communication;
6. the exact research-gap wording used or why it remains provisional;
7. any citations or claims that still require human verification.
