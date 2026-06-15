# Final BIB structure self-audit after reading 10+ similar articles

Last updated: 2026-06-15

This audit checks whether the English and Chinese RiceGeneFormer manuscripts now follow the common structure of recent Briefings in Bioinformatics benchmark, methods and genomic-prediction papers.

## Manuscripts audited

- English full manuscript: `docs/MANUSCRIPT_BIB_FULL_EN.md`
- Chinese full manuscript: `docs/MANUSCRIPT_BIB_FULL_ZH.md`
- Reading matrix: `docs/BIB_10_ARTICLE_READING_MATRIX_AND_SELF_AUDIT.md`

## 10+ BIB article basis

The rewrite was checked against 11 BIB articles:

1. GWKBR: GWAS-weighted Gaussian kernel Bayesian regression for genomic prediction.
2. WheatGP: CNN/LSTM genomic prediction for wheat.
3. Multi-population GBLUP with global/local genetic correlation.
4. Should we really use graph neural networks for transcriptomic prediction?
5. GWANN: genome-wide association neural networks for Alzheimer’s disease.
6. Community drug-response prediction cross-dataset benchmark.
7. Embedding-based drug-response prediction benchmark.
8. NanOlympicsMod m6A nanopore benchmark.
9. Somatic variant-caller and ensemble WES benchmark.
10. Machine-learning scoring functions on dissimilar blind benchmarks.
11. AMR phenotype prediction from microbial genomes.

## Structural audit

| BIB expectation | Current manuscript status | Action taken |
|---|---|---|
| Concrete searchable title without superiority claim | Pass | Title states leakage-aware ordinal benchmark and task; no SOTA wording. |
| Abstract states problem, gap, method, headline metrics, negative boundary and contribution | Pass | Abstract includes ordinal labels, leakage, RiceGeneFormer modules, baseline gap, cross-region failure and diagnostic framing. |
| Key points are concrete and evidence-based | Pass | Four bullets cover framework, leakage control, performance boundary and diagnostic audits. |
| Introduction funnels from field need to benchmark gap | Pass after rewrite | Added generalization-vs-benchmark paragraph and graph-as-hypothesis paragraph based on BIB examples. |
| Strong baselines appear before model interpretation | Pass | Results establish LightGBM/XGBoost/SNP-MLP before interpreting RiceGeneFormer. |
| Results follow evidence ladder rather than experiment list | Pass | Order is benchmark construction -> model design -> baseline ceiling -> RiceGeneFormer configuration -> distillation -> negative controls -> cross-region/gene-attention -> practical recommendation. |
| Negative findings are main text when they define claims | Pass | Graph identity, mapping windows, cross-region and gene-attention audits are explicit main Results sections. |
| Practical guidance is included | Pass after rewrite | Added “Practical interpretation from the benchmark” / “Benchmark 给出的实际使用建议”. |
| Methods are procedural and reproducible | Mostly pass | Methods state data, split, train-fold GWAS, graph construction, architecture, training, baselines, distillation, cross-region, attention export and metrics. Exhaustive command provenance remains supplementary/project docs. |
| Discussion teaches benchmark lesson, not victory narrative | Pass | Discussion emphasizes strong marker baselines, negative controls, limits and future evidence-changing tests. |
| Data/code availability is honest before DOI | Pass | Uses future tense and explicitly says DOI-backed release remains to be minted. |
| Bilingual consistency | Pass after rewrite | Chinese manuscript now mirrors the English additions on generalization, graph hypothesis and practical recommendation. |

## Remaining differences from typical BIB published articles

1. Reference formatting is still a working-anchor list, not a final BIB/OUP reference section.
2. DOI-backed code/data release is not yet minted, so the availability statement remains future-tense.
3. Some BIB articles include more formal supplementary tables for hyperparameters and per-dataset/per-trait results; this project has the content in docs/source_data and supplementary markdown, but final journal formatting is still pending.
4. The manuscript deliberately does not imitate positive-discovery BIB papers such as GWANN because the current gene-attention evidence does not support that level of biological claim.

## Final decision

The current English and Chinese manuscripts are now structurally close to BIB benchmark papers rather than Nature-style discovery papers. The biggest previous mismatch was insufficient emphasis on the BIB benchmark lesson: strong baselines can win, generalization must be separately tested, and biological graph/attention modules are hypotheses rather than validation. That mismatch has been corrected in both manuscripts.

Current status: suitable as a near-submission BIB benchmark manuscript draft, pending final reference formatting, DOI-backed release and journal-style supplementary packaging.
