# Project Context

## 1. Research background

The project studies deep learning-based text Semantic Communication (SC), using a Transformer-style end-to-end communication model inspired by DeepSC.

The conceptual communication chain is:

`input sentence -> semantic encoder -> channel encoder -> wireless channel -> channel decoder -> semantic decoder -> reconstructed sentence`

In the implementation history of the wider DeepSC project, the trained communication model exposes components named `encoder`, `channel_encoder`, `channel_decoder`, `decoder`, and `dense`. These names are useful implementation context, but the final paper must describe the architecture based on the exact experiment used for publication rather than assuming undocumented dimensions.

## 2. Focus of this repository

`Prince_SC` is the paper repository for the Knowledge Distillation stage only.

The research story is:

`larger trained SC Teacher -> knowledge transfer -> compact SC Student`

The Teacher represents the original/high-capacity semantic communication model. The Student is a smaller model intended to reduce deployment cost while retaining as much semantic communication performance as possible.

## 3. Main research question

Can a smaller Student semantic communication model trained through Knowledge Distillation approach the semantic reconstruction performance of the larger Teacher across different channel conditions while reducing model complexity?

Supporting questions include:

- How much does KD improve a compact Student relative to training the same Student without distillation?
- How does Teacher/Student performance change with SNR and channel conditions?
- What parameter, storage, inference-time, or computational reduction is achieved?
- Does the compressed Student preserve both lexical reconstruction quality and semantic meaning?

## 4. Evaluation perspective

The project has used or discussed the following evaluation dimensions:

- BLEU;
- semantic similarity / semantic metrics;
- performance across SNR values;
- direct Teacher vs. Student outputs;
- consistent decoding procedures, including greedy decoding and beam decoding when evaluated;
- model size / number of parameters;
- inference cost or latency when measured.

Only metrics actually generated for the final KD experiments should appear as results in the paper.

## 5. Relationship to the multilingual/LoRA research

A separate research direction extends the Student toward personalized and multilingual receivers using parameter-efficient adaptation such as LoRA. That work includes the idea of a shared transmitter/semantic encoder with multiple specialized receivers and language adaptation (e.g., Portuguese, and potentially Spanish/French).

That multilingual/LoRA work is **not** the contribution of this KD paper.

It can be mentioned as future work or broader project context only when useful and explicitly appropriate.

## 6. Known historical implementation context

The wider project has previously worked with DeepSC-style checkpoints where encoder-side and decoder-side states were loaded separately. Historical paths seen during experimentation included entries such as:

- `results/base_model/old/encoder_41.pth`
- `results/base_model/old/decoder_21.pth`
- `results/base_model/old/decoder_35.pth`
- `results/base_model/2026-08-23/encoder_01.pth`
- `results/base_model/2026-08-23/decoder_01.pth`

These are historical implementation notes, not automatically the final checkpoints for this paper. Confirm the exact Teacher checkpoint before citing an experiment.

## 7. Research integrity rule

This repository is an article workspace, not the authoritative source of experimental numbers unless those numbers have been copied here from validated runs. When information is missing, retain `TBD` rather than reconstructing values from memory or making an estimate.
