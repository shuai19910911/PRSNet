# Supplementary per-trait test-role results

Status: draft supplementary table generated from deterministic test-role artifacts. This file contains no raw genotypes, phenotypes, predictions or model weights.

Source artifacts:
- RiceGeneFormer per-trait test metrics: `docs/FINAL_PER_TRAIT_METRICS_RGF_W010_TEST.tsv`
- LightGBM/XGBoost test-role metrics: `data/3krice/processed/*_top512_e40_test_8566247/*_metrics.tsv`
- SNP-MLP per-trait test rerun: `data/3krice/processed/snp_mlp_test_pertrait_summary_8566261.tsv`
- Machine-readable combined table: `docs/SUPPLEMENTARY_PER_TRAIT_RESULTS.tsv`
- Supplementary figure: `docs/figures/figs1_per_trait_method_comparison.svg` / `.pdf` / `.tiff`
- Figure source data: `docs/figure_source_data/figs1_per_trait_method_comparison.csv` and `figs1_best_method_counts.csv`

## Per-trait macro-F1 summary

| Trait | RGF w0.10 | LightGBM | XGBoost | SNP-MLP a0.40 | SNP-MLP a0.60 | Best by macro-F1 |
|---|---:|---:|---:|---:|---:|---|
| CUDI_CODE_REPRO | 0.639 | 0.635 | 0.621 | 0.682 | 0.676 | SNP-MLP top512 alpha0.40 |
| CULT_CODE_REPRO | 0.218 | 0.232 | 0.220 | 0.259 | 0.264 | SNP-MLP top512 alpha0.60 |
| CUNO_CODE_REPRO | 0.299 | 0.358 | 0.284 | 0.382 | 0.469 | SNP-MLP top512 alpha0.60 |
| CUST_REPRO | 0.130 | 0.193 | 0.168 | 0.175 | 0.184 | LightGBM top512 |
| LLT_CODE | 0.291 | 0.257 | 0.211 | 0.315 | 0.310 | SNP-MLP top512 alpha0.40 |
| PLT_CODE_POST | 0.393 | 0.361 | 0.343 | 0.484 | 0.472 | SNP-MLP top512 alpha0.40 |
| PSH | 0.339 | 0.323 | 0.303 | 0.316 | 0.326 | RiceGeneFormer + distill w=0.10 |
| PTH | 0.450 | 0.487 | 0.501 | 0.483 | 0.497 | XGBoost top512 |
| SDHT_CODE | 0.361 | 0.360 | 0.343 | 0.378 | 0.390 | SNP-MLP top512 alpha0.60 |
| SPKF | 0.172 | 0.174 | 0.166 | 0.168 | 0.194 | SNP-MLP top512 alpha0.60 |

## Interpretation notes

- SNP-MLP alpha0.60 is the best per-trait macro-F1 method for most traits, confirming that the top-SNP neural baseline is the strongest macro-F1 reference under the current random split.
- RiceGeneFormer remains competitive on several easier or lower-cardinality traits but is not the best method in this per-trait test-role comparison.
- CUST_REPRO and SPKF remain difficult for all methods by macro-F1, consistent with severe class imbalance and multi-class ordinal structure.
- The figure-level summary marks one best macro-F1 method per trait and highlights the largest gaps between RiceGeneFormer and the best top-SNP baseline.

Definitions: macro-F1 is the unweighted mean of per-class F1 scores, so minority classes contribute equally; MAE is mean absolute error in ordinal class units, lower is better.
