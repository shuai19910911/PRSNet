# BIB submission package: RiceGeneFormer

Last updated: 2026-06-14

## Recommended title

A leakage-aware ordinal benchmark for gene-aware genotype-to-phenotype prediction in 3K rice genomes

## Alternative titles

1. Benchmarking gene-aware ordinal genotype-to-phenotype prediction in 3K rice genomes
2. RiceGeneFormer exposes the limits of gene-aware neural genotype-to-phenotype prediction under strong SNP baselines
3. A diagnostic benchmark for leakage-aware gene-level genotype-to-phenotype prediction in rice
4. Gene-aware ordinal multi-task learning approaches but does not surpass strong SNP baselines in 3K rice genomes

Most defensible title: title 1. It states the contribution as a benchmark, not as a performance breakthrough.

## BIB-ready abstract draft

Plant genotype-to-phenotype prediction increasingly relies on dense genomic panels, but many crop phenotypes are ordinal descriptor codes with strong class imbalance. This creates two linked challenges: models must respect ordered trait labels, and evaluations must prevent genome-wide association signals or model-selection procedures from leaking validation or test information. We developed RiceGeneFormer-OMTL, a gene-aware ordinal multi-task framework for 3K Rice Genome phenotype prediction, and used it to build a leakage-aware benchmark for ten core ordinal rice traits. The framework combines train-fold GWAS priors, SNP-to-gene aggregation, graph-aware gene tokens, trait-query decoding, gated top-SNP fusion and cumulative-link ordinal heads. All final neural metrics were recomputed with deterministic full-role validation or test evaluation.

RiceGeneFormer reached deterministic test macro-F1 values of 0.3229±0.0062 without distillation and 0.3292±0.0057 with weak expected-score distillation. These results approached LightGBM using train-fold top SNPs (macro-F1 0.3380), but remained below balanced SNP-MLP baselines (macro-F1 0.3644±0.0106 to 0.3781±0.0029). Gated top-SNP fusion and mild class balancing were the most useful model choices, whereas graph identity, SNP-to-gene window choice, threshold calibration and expected-score distillation produced limited or unstable gains. Cross-region benchmarks across Southeast Asia, South Asia and East Asia did not support a distribution-shift robustness advantage. Gene-attention audits further showed that bounded gene selection can constrain biological interpretation, and a train-fold GWAS-top2048 gene panel did not improve performance or known-gene recovery.

RiceGeneFormer therefore provides a reproducible diagnostic framework rather than a claim of neural-model superiority. The benchmark clarifies when gene-aware crop models approach marker-level baselines, where they fail, and which evidence is required before making robustness or biological-discovery claims.

## Key points

- RiceGeneFormer-OMTL integrates train-fold GWAS priors, SNP-to-gene aggregation, graph-aware gene tokens, trait-query decoding and ordinal multi-task losses for 3K rice descriptor phenotypes.
- Deterministic test evaluation shows that RiceGeneFormer approaches LightGBM but remains below balanced SNP-MLP top-SNP baselines by macro-F1.
- Cross-region and gene-attention audits do not support robustness or biological-discovery claims for the current model.
- The main contribution is a leakage-aware benchmark and diagnostic framework for fair evaluation of gene-aware crop genotype-to-phenotype models.

## Keywords

Genotype-to-phenotype prediction; rice; 3K Rice Genome; ordinal classification; multi-task learning; genome-wide association study; gene-aware neural networks; benchmark; data leakage; macro-F1.

## One-sentence contribution

In 3K rice genotype-to-phenotype prediction, we show that a leakage-aware gene-level ordinal framework can approach but not surpass strong train-fold top-SNP baselines, supported by deterministic test metrics, ablations, cross-region checks and gene-attention audits, with claims bounded to benchmarking and diagnostics rather than robustness or biological discovery.

## Claim-evidence map

| Claim | Evidence | Status |
|---|---|---|
| The benchmark is leakage-aware | Train-fold GWAS p-values, split-specific top-SNP selection, deterministic full-role evaluation | Supported |
| RiceGeneFormer approaches tree baselines | RiceGeneFormer w=0.10 macro-F1 0.3292±0.0057 vs LightGBM 0.3380 | Supported |
| RiceGeneFormer does not beat strongest SNP baselines | SNP-MLP alpha0.40/0.60 macro-F1 0.3644–0.3781 | Supported |
| Cross-region robustness is not demonstrated | Region-shift gaps vs SNP-MLP alpha0.60 are negative in all three held-out regions | Supported |
| Gene attention is not positive biological validation | Prefix-gene truncation and GWAS-top2048 seed42 pilot macro-F1 0.2990 with no top300 known-gene recovery | Supported |
| The paper is suitable as a BIB-style benchmark/diagnostic study | Reproducible pipeline, strong baselines, ablations, negative controls and source-data package | Supported, pending final DOI |

## Missing before submission

1. Final repository DOI or accession for code and source data.
2. Target-journal reference formatting.
3. Final decision on whether diagnostic Table 2 remains main text or supplement.
4. Shortened figure legends matched to final figure panel layout.
5. Final source-data DOI and release tag.

## BIB style standard for final rewrite

The final manuscript should follow `docs/BIB_STYLE_AND_SUBMISSION_WRITING_GUIDE.md`. The target is a BIB-style benchmark/methods article of approximately 7,000-8,000 main-text words unless the final submission system enforces a stricter article-type limit. The current abstract already fits the recommended 250-300 word range. The final rewrite should use an evidence-ladder Results structure, detailed Methods sufficient for reproduction, up to 5 key points, up to 6-8 keywords, and explicit data/code availability wording that remains future-tense until a DOI or release tag exists.
