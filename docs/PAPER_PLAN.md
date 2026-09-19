# IEEE Paper Plan — Knowledge Distillation for Semantic Communication

## Working title

Primary working title:

**Knowledge Distillation for Efficient Transformer-Based Semantic Communication**

Alternative title:

**Lightweight Semantic Communication Receivers via Knowledge Distillation**

The title is provisional until the final contribution and compression target are fully confirmed.

## Central paper message

Transformer-based text semantic communication can be costly to deploy. This paper evaluates whether a compact Student can inherit the behavior of a larger Semantic Communication Teacher through Knowledge Distillation while preserving semantic reconstruction performance over noisy channels.

## Planned structure

### Abstract

Write last. It should contain:

- problem/context;
- compression objective;
- KD approach;
- experimental setting;
- quantitative main findings;
- conclusion.

Do not place placeholder performance numbers in the abstract.

### I. Introduction

Cover:

1. semantic communication motivation;
2. DeepSC-style end-to-end text communication;
3. deployment cost of high-capacity Transformer models;
4. why compact receivers/models matter;
5. KD as a model-compression/knowledge-transfer mechanism;
6. gap addressed by this work;
7. contributions;
8. paper organization.

The contribution bullets should be finalized only after the exact experiments are fixed.

### II. Related Work

Organize thematically rather than as a list of papers:

- deep learning-based text semantic communication / DeepSC;
- efficient or lightweight semantic communication;
- Knowledge Distillation and Transformer compression;
- KD in communication or related sequence-generation settings;
- research gap leading to this paper.

Before claiming that KD for this exact SC setting is new, perform a dedicated literature search.

### III. System Model

Describe:

- source sentence and token sequence;
- semantic encoder;
- channel encoder;
- channel model;
- channel decoder;
- semantic decoder;
- output distribution / reconstructed sequence;
- Teacher and Student roles.

Use notation consistently across all later equations.

### IV. Knowledge Distillation for Semantic Communication

Describe the actual training objective used in the implementation.

Possible components, only if implemented:

- hard-label/cross-entropy reconstruction loss;
- soft-target/logit distillation with temperature;
- hidden-state or representation matching;
- weighted combined objective.

A generic KD form that may be adapted after implementation confirmation is:

`L = lambda_hard L_hard + lambda_KD L_KD (+ lambda_feat L_feat)`.

The implementation audit now confirms CE + temperature-scaled KL(Teacher || Student) + final decoder squared-cosine alignment, with weights 0.6/0.3/0.1 and temperature 2. The manuscript uses alpha/beta/gamma notation; these are configuration facts, not results.

### V. Experimental Methodology

Document reproducibly:

- dataset and preprocessing;
- vocabulary/tokenization;
- Teacher architecture;
- Student architecture;
- parameter counts;
- compression ratio;
- optimizer and learning-rate schedule;
- KD hyperparameters;
- channel model(s);
- training SNR;
- evaluation SNR range;
- decoding strategy;
- BLEU setup;
- semantic metric setup;
- hardware and inference measurement procedure if latency is reported.

### VI. Results and Discussion

Recommended comparisons:

- Teacher;
- Student without KD;
- Student with KD;
- optional multiple Student capacities;
- optional KD ablation (temperature/loss weights/features) if runs exist.

Recommended plots/tables:

- BLEU vs. SNR;
- semantic similarity vs. SNR;
- parameter/model-size table;
- inference cost/latency table if measured;
- representative reconstruction examples, selected by a reproducible criterion rather than cherry-picking.

Interpret trade-offs rather than reporting metrics alone.

### VII. Conclusion

Summarize:

- what was compressed;
- how KD affected performance;
- efficiency/quality trade-off;
- limitations;
- future work.

Multilingual/personalized LoRA receivers may be mentioned as future work but are not part of the present contribution.

## Immediate writing order

1. Confirm the exact Teacher/Student/KD experimental design in `docs/EXPERIMENTS.md`.
2. Perform and verify the KD + semantic communication literature search.
3. Write Introduction and Related Work with verified references.
4. Write System Model and KD Method from the actual implementation.
5. Add Experimental Methodology from validated configuration.
6. Add Results only after final plots/tables are generated.
7. Write Abstract and Conclusion last.

## Current status — 2026-09-18

Sections I and III–V and two TikZ figures are drafted; Section II is preserved with receiver-only positioning clarified. The publication pipeline is 70/15/15 -> new train-only BPE -> new Teacher from scratch -> matched CE/KD receivers -> untouched test evaluation. Historical 90/10 artifacts are excluded. Follow NEXT_TASK.md for final experiments, then write Results, Abstract and Conclusion. PDF compilation and visual inspection remain pending because no TeX engine is installed in this environment.
