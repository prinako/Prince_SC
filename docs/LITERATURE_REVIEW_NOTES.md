# Related Work literature review

Search and verification date: 2026-09-18. Scope: text Semantic Communication Teacher -> KD -> compact Student. This is a targeted review, not an exhaustive systematic review.

## Search and verification method

Searched combinations of `semantic communication knowledge distillation`, `text semantic communication knowledge distillation`, `DeepSC distillation compression`, `semantic communications model compression`, `lightweight DeepSC`, and recent 2025/2026 text/KD work; followed exact titles, DOI records, and references in retrieved papers. Sources were discovered through web search and checked against IEEE records, author manuscripts, institutional repositories, ACL Anthology proceedings, and arXiv records. Google Scholar was not independently queried. Search results unrelated to wireless semantic communication were screened out.

For six IEEE publications, title, complete authors, venue, publication year, volume/issue/pages and DOI were checked against publisher-deposited Crossref metadata at `https://api.crossref.org/works/{DOI}`. Some direct IEEE pages were blocked or returned empty content; those cases use publisher metadata plus primary author manuscripts for claims, not secondary citation aggregators. ACL metadata and PDFs were checked directly. Publication year refers to the assigned journal issue, not the year embedded in the DOI. No citation-count or performance-superiority claim is made.

Ten sources are cited: eight peer-reviewed publications and two explicitly identified preprints (the foundational Hinton source and the very recent Eid source). The latter is retained because of its unusually close overlap with the proposed paper; no published version was found by exact-title search. arXiv copies of published papers are reading copies, not additional bibliography entries. Full-text inspection below means relevant passages were checked; it does not imply independent reproduction of results.

## Source records

### `xie2021deepsc` — text semantic communication foundation

- **Title:** Deep Learning Enabled Semantic Communication Systems.
- **Authors:** Huiqiang Xie; Zhijin Qin; Geoffrey Ye Li; Biing-Hwang Juang.
- **Publication:** IEEE Transactions on Signal Processing, vol. 69, pp. 2663–2675, 2021; peer-reviewed journal article.
- **DOI / primary sources:** [10.1109/TSP.2021.3071210](https://doi.org/10.1109/TSP.2021.3071210); [author manuscript and abstract](https://arxiv.org/abs/2006.10685).
- **Relevance:** DeepSC establishes the Transformer-based text transmission setting. Its reconstruction objective and sentence-similarity evaluation motivate retaining both meaning and lexical fidelity when compressing an SC model.
- **Supported claims:** DeepSC uses Transformers for text SC; semantic recovery is an objective beyond bit/symbol accuracy; sentence similarity is an evaluation dimension.
- **Cite in Section II:** Yes, subsection A.
- **Verification / limits:** Publisher metadata checked through Crossref; method claims checked against the author abstract. No architecture dimensions or performance values are transferred to Prince_SC.

### `xie2021lite` — model-side efficiency

- **Title:** A Lite Distributed Semantic Communication System for Internet of Things.
- **Authors:** Huiqiang Xie; Zhijin Qin.
- **Publication:** IEEE Journal on Selected Areas in Communications, vol. 39, no. 1, pp. 142–153, 2021; peer-reviewed journal article.
- **DOI / primary sources:** [10.1109/JSAC.2020.3036968](https://doi.org/10.1109/JSAC.2020.3036968); [author version](https://arxiv.org/abs/2007.11095).
- **Relevance:** L-DeepSC reduces model redundancy and weight precision for constrained devices. It also addresses distributing model weights, which is distinct from the channel resources used for a semantic message.
- **Supported claims:** Pruning and weight quantization reduce deployment/model distribution costs; model compression and semantic transmission efficiency must be distinguished.
- **Cite in Section II:** Yes, subsection B.
- **Verification / limits:** IEEE abstract and publisher metadata verified. Online publication was in 2020; the assigned journal issue is 2021. Its reported compression ratio is not reused as a Prince_SC result.

### `do2026biddeepsc` — recent compact SC architecture

- **Title:** BidDeepSC-1.58b: 1.58-Bit Bidirectional Slimmable Semantic Communication System.
- **Authors:** Tung Son Do; Thanh Phung Truong; Quang Tuan Do; Dongwook Won; Wonjong Noh; Sungrae Cho.
- **Publication:** IEEE Transactions on Vehicular Technology, vol. 75, no. 4, pp. 6254–6269, 2026; peer-reviewed journal article.
- **DOI / primary sources:** [10.1109/TVT.2025.3622754](https://doi.org/10.1109/TVT.2025.3622754); [author-hosted manuscript](https://uclab.re.kr/publications_openaccesspdf/tung_tvt_2025_open.pdf); [institutional record](https://cau.scholarworks.kr/item/096adafc-8fe2-4078-8cef-56475498604e).
- **Relevance:** The design combines bidirectional parameter sharing, quantization, and slimmable widths. It broadens the efficiency discussion beyond simply reducing Transformer depth.
- **Supported claims:** Shared encoder–decoder architecture, low-bit quantization, and adaptable network widths are established SC efficiency strategies.
- **Cite in Section II:** Yes, subsection B.
- **Verification / limits:** Publisher metadata, institutional record and author abstract checked. DOI/early publication uses 2025; journal issue is April 2026. The section does not claim that this work excludes distillation or is a pure pruning baseline.

### `hinton2015distilling` — foundational KD

- **Title:** Distilling the Knowledge in a Neural Network.
- **Authors:** Geoffrey Hinton; Oriol Vinyals; Jeff Dean.
- **Publication:** arXiv:1503.02531, 2015; cited as a preprint, not a verified peer-reviewed proceedings article. The arXiv comments mention the NIPS 2014 Deep Learning Workshop.
- **DOI / primary sources:** [10.48550/arXiv.1503.02531](https://doi.org/10.48550/arXiv.1503.02531); [manuscript](https://arxiv.org/pdf/1503.02531).
- **Relevance:** Provides the soft-output Teacher-to-Student formulation. It explains the general knowledge-transfer motivation without establishing the actual Prince_SC loss.
- **Supported claims:** Softened output distributions convey Teacher information beyond hard labels.
- **Cite in Section II:** Yes, subsection C; foundational-preprint exception is explicit.
- **Verification / limits:** Author list, title, date, identifier and formulation checked in the primary manuscript. No invented workshop proceedings, pagination or peer-review status.

### `kim2016sequencekd` — sequence-model distillation

- **Title:** Sequence-Level Knowledge Distillation.
- **Authors:** Yoon Kim; Alexander M. Rush.
- **Publication:** Proceedings of the 2016 Conference on Empirical Methods in Natural Language Processing, pp. 1317–1327, 2016; peer-reviewed conference paper, Association for Computational Linguistics.
- **DOI / primary sources:** [10.18653/v1/D16-1139](https://doi.org/10.18653/v1/D16-1139); [proceedings metadata](https://aclanthology.org/D16-1139/); [PDF](https://aclanthology.org/D16-1139.pdf).
- **Relevance:** Studies word- and sequence-level transfer in translation. Its comparisons of greedy and beam decoding motivate controlling inference procedure when assessing compact generators.
- **Supported claims:** Distillation can operate at different sequence granularities; decoding affects the quality/efficiency comparison.
- **Cite in Section II:** Yes, subsection C.
- **Verification / limits:** Metadata and relevant full-text passages checked, including Sections 2–4. The models are recurrent/LSTM, not Transformers; no channel-noise result is implied.

### `jiao2020tinybert` — Transformer compression

- **Title:** TinyBERT: Distilling BERT for Natural Language Understanding.
- **Authors:** Xiaoqi Jiao; Yichun Yin; Lifeng Shang; Xin Jiang; Xiao Chen; Linlin Li; Fang Wang; Qun Liu.
- **Publication:** Findings of the Association for Computational Linguistics: EMNLP 2020, pp. 4163–4174, 2020; peer-reviewed proceedings paper.
- **DOI / primary sources:** [10.18653/v1/2020.findings-emnlp.372](https://doi.org/10.18653/v1/2020.findings-emnlp.372); [metadata](https://aclanthology.org/2020.findings-emnlp.372/); [PDF](https://aclanthology.org/2020.findings-emnlp.372.pdf).
- **Relevance:** Demonstrates Transformer-specific transfer at multiple representation levels. General and task-specific training stages are relevant design precedents, not assumptions about the current SC implementation.
- **Supported claims:** Embedding, attention, hidden-state and prediction transfer; two-stage distillation framework.
- **Cite in Section II:** Yes, subsection C.
- **Verification / limits:** Full author list and metadata verified; Section 3 supports the transfer mechanisms. Its NLU results are not evidence of noisy-channel reconstruction performance.

### `liu2024kdsemcom` — closest peer-reviewed precedent

- **Title:** Knowledge Distillation-Based Semantic Communications for Multiple Users.
- **Authors:** Chenguang Liu; Yuxin Zhou; Yunfei Chen; Shuang-Hua Yang.
- **Publication:** IEEE Transactions on Wireless Communications, vol. 23, no. 7, pp. 7000–7012, 2024; peer-reviewed journal article.
- **DOI / primary sources:** [10.1109/TWC.2023.3336941](https://doi.org/10.1109/TWC.2023.3336941); [author manuscript](https://arxiv.org/pdf/2311.13789); [Warwick accepted-publication record](https://wrap.warwick.ac.uk/id/eprint/181356/).
- **Relevance:** This directly overlaps text SC, Transformer encoders/decoders, KD and compression. It studies limited training data and multi-user interference, including compact no-KD baselines and lexical/semantic evaluation.
- **Supported claims:** Prior KD-based text SC exists; compression and robustness to noise/interference are studied; BLEU, sentence similarity and no-KD compact baselines already appear.
- **Cite in Section II:** Yes, subsection D and positioning paragraph.
- **Verification / limits:** Publisher-deposited metadata supplies the hyphenated final title and 2024 issue date. Author-manuscript Section IV, especially simulation settings and baseline definitions on PDF pp. 8–9, supports the comparisons. A publisher full-text comparison remains advisable before detailed numerical reuse; none is made here.

### `albaseer2024dynamickd` — resource-aware SC distillation

- **Title:** Tailoring Semantic Communication at Network Edge: A Novel Approach Using Dynamic Knowledge Distillation.
- **Authors:** Abdullatif Albaseer; Mohamed Abdallah.
- **Publication:** ICC 2024 – IEEE International Conference on Communications, pp. 1455–1460, 2024; peer-reviewed conference paper.
- **DOI / primary sources:** [10.1109/ICC51166.2024.10623077](https://doi.org/10.1109/ICC51166.2024.10623077); [author manuscript](https://arxiv.org/pdf/2401.10214); [institutional record](https://elmi.hbku.edu.qa/en/publications/tailoring-semantic-communication-at-network-edge-a-novel-approach/).
- **Relevance:** KD customizes semantic models to heterogeneous device resources and communication constraints. This establishes another deployment-oriented use of KD within SC.
- **Supported claims:** Dynamic KD adapts semantic models to edge-device constraints.
- **Cite in Section II:** Yes, subsection D.
- **Verification / limits:** Publisher metadata and manuscript abstract checked. The section does not assume that its application or reconstruction objective matches Prince_SC.

### `ding2026robustkd` — recent robust SC distillation

- **Title:** Large-Scale Model-Enabled Semantic Communication via Robust Knowledge Distillation and Lightweight Architecture Search.
- **Authors:** Kuiyuan Ding; Caili Guo; Yang Yang; Zhongtian Du; Walid Saad.
- **Publication:** IEEE Transactions on Cognitive Communications and Networking, vol. 12, pp. 7716–7730, 2026; peer-reviewed journal article.
- **DOI / primary sources:** [10.1109/TCCN.2026.3687590](https://doi.org/10.1109/TCCN.2026.3687590); [earlier author version](https://arxiv.org/abs/2508.02148).
- **Relevance:** Combines compact encoder architecture search with robust Teacher-to-Student transfer. Its image-classification task differs from reconstructing transmitted sentences.
- **Supported claims:** KD and architecture search have been combined for compact, channel-robust SC encoders evaluated on image classification.
- **Cite in Section II:** Yes, subsection D.
- **Verification / limits:** Final publisher abstract and Crossref metadata checked. The older arXiv title differs; the bibliography uses the published title and pagination. No unspecified issue number is supplied.

### `eid2026krumdeepsc` — recent directly overlapping preprint

- **Title:** Krum-Inspired Central Teacher Selection and Residual Channel Bottlenecks for Efficient DeepSC.
- **Authors:** Rami Eid; Mostafa Jammoul; Omar Kaaki; Maria Slim; Mariette Awad; Hadi Sarieddeen.
- **Publication:** arXiv:2609.13405, submitted September 11, 2026; preprint, peer review not established.
- **Primary source:** [arXiv record](https://arxiv.org/abs/2609.13405).
- **Relevance:** Studies a compressed DeepSC Student with teacher selection and residual channel representations. Its text-quality and efficiency evaluations under channel variability overlap the intended Prince_SC evaluation.
- **Supported claims:** A recent preprint directly studies distilled compact DeepSC Students with these mechanisms and evaluation dimensions.
- **Cite in Section II:** Yes, explicitly as a preprint in subsection D.
- **Verification / limits:** Metadata and abstract checked, not an independent full methodological assessment. No published version found. The arXiv DOI is marked pending registration, so the bibliography uses its stable arXiv URL instead. Recheck publication status and inspect its full protocol before final novelty claims.

## Research-gap assessment

The search does **not** establish an unoccupied text/Transformer/KD research gap. Liu et al. already cover these elements, no-KD compact baselines, lexical/semantic quality and channel variability. Eid et al. further overlap the proposed quality–efficiency evaluation, although that work remains a preprint. Neither using a smaller Student nor plotting BLEU against SNR is by itself a defensible novelty claim. The proposed three-way comparison is good experimental practice, not evidence of originality.

Prince_SC can presently be positioned as an investigation of a specific compression/quality trade-off. Its potential distinction must be checked against the actual compressed components, Teacher/Student architecture, loss and transfer locations, data and channel protocol, and measured deployment costs. All remain TBD in the experimental record. Do not claim receiver-only compression, a new loss, a new robustness mechanism, or a missing prior baseline without evidence.

Exact provisional wording in Section II:

> This work investigates the reconstruction-quality--model-complexity trade-off when a compact Transformer-based text semantic communication Student learns from a larger Teacher. Its intended evaluation separates the Teacher, a compact Student trained without KD where available, and the Student trained with KD under consistent channel and decoding conditions. The specific distinction from prior KD-based semantic communication remains provisional until the compression target, training objective, and experimental protocol are confirmed.

Next research action: inspect the actual KD implementation and validated runs, then prepare a direct comparison with Liu and Eid before finalizing contribution bullets. No claim that KD is first applied to semantic communication is warranted. LoRA, multilingual adaptation and personalized receivers remain outside the contribution.

## Repository validation and limitations

- Ten newly added bibliography entries are cited in Section II; published/preprint versions are not duplicated.
- Existing bibliography records are retained. Eleven exact duplicate legacy entries were removed while preserving one unchanged copy of each; their old metadata was not independently reverified.
- The bibliography style is now `IEEEtran` for IEEE reference rendering; the class and author/funding content are unchanged.
- Citation-key consistency, duplicate-key/title/DOI checks, balanced braces and section placement are checked locally. Full PDF/BibTeX compilation and visual reference inspection are unavailable because this environment has no TeX toolchain.
- No outstanding unverified metadata is knowingly used in the new entries. Final novelty assessment requires the experimental record, full-protocol comparison with the closest studies, and a publication-status refresh for Eid. Numerical claims from these sources have deliberately not been imported.
