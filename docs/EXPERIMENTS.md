# Experiments — Source of Truth for the KD Paper

This file separates **confirmed information** from **planned/TBD information**. Codex must not turn TBD fields into factual statements in the paper.

## 1. Teacher

Role: high-capacity/original text Semantic Communication model.

Known component naming from the wider implementation:

- semantic encoder: `encoder`
- channel encoder: `channel_encoder`
- channel decoder: `channel_decoder`
- semantic decoder: `decoder`
- output projection/head: `dense`

Final architecture dimensions: **TBD**

Final parameter count: **TBD**

Final checkpoint used for paper experiments: **TBD**

Historical checkpoints exist in the wider project, but none is designated here as the publication Teacher until explicitly confirmed.

## 2. Student

Role: compact model trained to reproduce the target behavior while reducing complexity.

Final Student architecture: **TBD**

Which layers/dimensions are reduced: **TBD**

Final parameter count: **TBD**

Compression ratio relative to Teacher: **TBD**

Student-without-KD baseline available: **TBD**

## 3. Knowledge Distillation

Teacher frozen during Student training: **TBD — confirm from KD training code**

Hard target loss: **TBD**

Soft/logit distillation loss: **TBD**

Feature/representation distillation: **TBD**

Temperature: **TBD**

Loss weights: **TBD**

Distillation location(s): **TBD**

Do not assume standard KD equations until these items are checked against the actual training implementation.

## 4. Dataset and preprocessing

Final dataset: **TBD**

Training split size: **TBD**

Validation split size: **TBD**

Test split size: **TBD**

Vocabulary/tokenization: **TBD**

Maximum sequence length: **TBD**

The wider project has also used Europarl for multilingual work, including English/Portuguese aligned data. Do not automatically reuse those multilingual dataset details in this KD paper unless the KD runs actually use them.

## 5. Channel configuration

Channel model(s): **TBD**

Training SNR: **TBD**

Evaluation SNR values/range: **TBD**

Channel uses during Teacher training vs. Student KD: **TBD**

## 6. Decoding

Greedy decoding has been used/discussed in the wider experiments.

Beam decoding has also been compared/discussed.

Final decoding method for quantitative paper comparison: **TBD**

If both are reported, hold all other evaluation settings constant and explain the comparison clearly.

## 7. Metrics

Metrics known to be relevant in the project:

- BLEU;
- semantic similarity / semantic metric;
- parameter count and model size;
- inference latency/complexity if measured.

Exact BLEU implementation/configuration: **TBD**

Exact semantic metric/model: **TBD**

Latency hardware and timing protocol: **TBD**

## 8. Required result matrix

At minimum, aim to populate:

| Model | KD | Params | Size | BLEU @ SNR(s) | Semantic metric @ SNR(s) | Latency |
|---|---:|---:|---:|---:|---:|---:|
| Teacher | N/A | TBD | TBD | TBD | TBD | TBD |
| Student baseline | No | TBD | TBD | TBD | TBD | TBD |
| Student KD | Yes | TBD | TBD | TBD | TBD | TBD |

Additional Student sizes or ablations can be added when validated.

## 9. Reproducibility checklist

Before Results are treated as final, record:

- random seed(s);
- software versions;
- hardware/GPU;
- batch size;
- optimizer;
- learning rate and schedule;
- epochs/early stopping;
- exact Teacher checkpoint;
- exact Student checkpoint;
- channel simulation configuration;
- decoding configuration;
- metric code/version.
