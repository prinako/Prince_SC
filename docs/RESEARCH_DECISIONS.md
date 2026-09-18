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

## Research integrity rules

- Do not fabricate experimental values or citations.
- Distinguish confirmed results from plans/TBDs.
- Verify novelty claims with a dedicated literature search.
- Prefer the final published version of a reference over a preprint when available.
- Keep the KD paper contribution separate from the multilingual/LoRA contribution.
