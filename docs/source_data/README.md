# RiceGeneFormer manuscript source data

This directory contains lightweight source-data tables and JSON manifests that support the BIB-style RiceGeneFormer benchmark manuscript. It intentionally excludes raw genotype/phenotype files, large processed matrices, GWAS p-value arrays, model checkpoints, teacher predictions, logs and credentials.

## Files

| File | Type | Description | Source in local analysis tree |
|---|---|---|---|
| `region_shift_baseline_summary.tsv` | TSV | Cross-region baseline summary for LightGBM, XGBoost and SNP-MLP alpha0.40/0.60. | `data/3krice/processed/region_shift_baseline_summary_region_manual_20260614_213808.tsv` |
| `region_shift_rice_geneformer_summary.tsv` | TSV | Cross-region RiceGeneFormer summary for held-out Southeast Asia, South Asia and East Asia. | `data/3krice/processed/region_shift_rice_geneformer_summary_region_rgf_manual_20260614_214252.tsv` |
| `gene_attention_w010_top200_aggregate.tsv` | TSV | Aggregated top-200 trait-gene attention ranks from w=0.10 RiceGeneFormer seed42/43/44 deterministic test-role checkpoints. | `data/3krice/processed/gene_attention_w010_test_aggregate/gene_attention_top200_aggregate.tsv` |
| `gene_attention_w010_aggregate_manifest.json` | JSON | Manifest for the aggregated w=0.10 gene-attention export. | `data/3krice/processed/gene_attention_w010_test_aggregate/gene_attention_aggregate_manifest.json` |
| `known_gene_overlap_manifest.json` | JSON | Preliminary known-gene overlap audit manifest. This is an audit artifact, not a curated biological database. | `data/3krice/processed/gene_attention_known_gene_overlap_w010_test/known_gene_overlap_manifest.json` |
| `gwas_top2048_seed42_evaluation_manifest.json` | JSON | Deterministic test evaluation for the seed42 GWAS-top2048 RiceGeneFormer pilot. | `data/3krice/processed/rice_geneformer_gwas_top2048_seed42_e20_s100_gpu2/test_eval/evaluation_manifest.json` |
| `gwas_top2048_graph_manifest.json` | JSON | Manifest for the split-specific train-fold GWAS-top2048 chromosome-neighbor graph. | `data/3krice/processed/gene_graph/baseline/gwas_top2048_random_seed42_chr_neighbor_k5/graph_manifest.json` |
| `SOURCE_DATA_MANIFEST.json` | JSON | File-level manifest for this source-data directory, including original local source paths and byte sizes. | generated during source-data packaging |

## Metric definitions

- `macro-F1`: unweighted mean of per-class F1 scores. Higher values indicate better average class-wise performance, especially for minority classes.
- `accuracy`: fraction of observed labels for which the predicted class equals the observed class.
- `MAE`: mean absolute error in ordinal class units. Lower values are better.
- `Spearman`: rank correlation between predicted expected ordinal score and observed ordinal label.
- `alpha`: class-balance interpolation strength for neural baselines, where larger values increase the influence of inverse-frequency class weights.

## Interpretation boundaries

1. Cross-region summaries show diagnostic held-out geographic-region performance. They do not support a RiceGeneFormer robustness advantage over balanced SNP-MLP baselines.
2. Gene-attention summaries are post-hoc diagnostic artifacts. They should not be interpreted as causal gene discovery or validated candidate-gene evidence.
3. The known-gene overlap manifest is preliminary. Gene identifiers and citations require final verification before any biological claim.
4. The GWAS-top2048 graph and seed42 pilot were used to test whether the original prefix-gene limitation could be corrected. This pilot reduced predictive performance and did not support extending the route to seed43/44.

## Reproducibility notes

The raw public data sources are listed in `../DATA_SOURCES_AND_CITATIONS.md`. The data/code release plan is listed in `../DATA_AND_CODE_AVAILABILITY_PLAN.md`. These source-data files should be archived together with the exact code version used for manuscript submission, ideally via a DOI-backed GitHub/Zenodo release or equivalent repository record.
