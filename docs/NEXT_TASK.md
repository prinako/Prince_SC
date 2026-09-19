# Next Task — Complete the New Publication Experiments

The manuscript Sections I–V and two TikZ diagrams are drafted. Final Results, Abstract and Conclusion remain pending. Read AGENTS.md, PROJECT_CONTEXT.md, RESEARCH_DECISIONS.md, EXPERIMENTS.md and the current-state IMPLEMENTATION_AUDIT.md addendum first. Verify live DeepSC status/branch/HEAD; last inspected BPE revision: `0d3b119c9215fb22f427d3aa7c9629c9a7cdc18c`.

## Required sequence

1. Verify the new 70/15/15 preprocessing artifacts and freshly trained Teacher. Establish exact data root, split IDs, corpus/tokenizer hashes, actual vocabulary, accepted counts, normalized-content overlap policy, run revision/settings and selected validation-CE checkpoint. Teacher length 67 is fixed in the inspected code; verify saved buffers too. No old 90/10 tokenizer or checkpoint may be reused.
2. Resolve remaining implementation prerequisites before final runs: robust split-ratio checks, forced fresh tokenizer provenance, dated/undated data paths, unused Student test loading, complete manifests, and a common validation checkpoint-selection rule for CE/KD Students. Keep test reserved for final evaluation.
3. Configure Student training to load the newly selected Teacher rather than hardcoded historical `encoder_26.pth`/`decoder_26.pth`. Validate tokenizer, vocabulary, architecture and positional-buffer compatibility; do not mask loading errors.
4. Train a fresh receiver-only four-layer/eight-head CE baseline (alpha/beta/gamma=1/0/0; smoothing 0.1). Freeze the new Teacher transmitter. Record seeds, initialization, budget, optimizer, channel and best validation checkpoint.
5. Train the same receiver with KD (0.6/0.3/0.1, temperature 2), with matched data, initialization policy, optimizer, budget and channel protocol. Preserve validation/test separation. Confirm all run-dependent settings from artifacts.
6. Before opening test for quality evaluation, fix the SNR grid, independent channel/noise trials, batching, seed policy, matched greedy/beam decoding, corpus BLEU signature, semantic model/revision, optional metrics and uncertainty aggregation. Evaluate Teacher/CE/KD on identical test/channel conditions. Compute corpus BLEU per SNR/trial, never mean sentence BLEU labeled corpus BLEU.
7. Recompute final receiver and full-system parameters; measure serialized size and any claimed latency/memory/FLOPs on specified hardware with warm-up/synchronization/length controls. Generate traceable quality-versus-SNR plots and comparison tables. Run controlled loss ablations if included.
8. Update the experimental manifest and write Results, then Abstract and Conclusion only from validated evidence. Compile the manuscript and inspect both TikZ figures on a TeX-enabled system; the present environment lacks a TeX toolchain.

Do not launch long training merely to fill a TBD without verifying the prerequisites. Do not include LoRA, personalized/multilingual receiver adaptation or old candidate counts as final evidence. Do not push unless explicitly requested.
