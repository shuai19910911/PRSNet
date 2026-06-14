# Main results table for RiceGeneFormer manuscript

Status: draft Table 1 source text. Values are copied from deterministic final evaluations and baseline summaries already recorded in project progress. This file is lightweight and contains no raw data, model weights or predictions.

## Table 1. Final RiceGeneFormer and top-SNP baseline performance

| Model / setting | Role | Seeds | macro-F1 | Accuracy | MAE | Spearman | Interpretation |
|---|---|---:|---:|---:|---:|---:|---|
| RiceGeneFormer, no distillation | test | 42/43/44 | 0.3229 ± 0.0062 | 0.5895 ± 0.0029 | 0.5978 ± 0.0039 | 0.2556 ± 0.0053 | Final gene-aware model without teacher transfer |
| RiceGeneFormer + expected-score distillation w=0.10 | test | 42/43/44 | 0.3292 ± 0.0057 | 0.5894 ± 0.0049 | 0.5948 ± 0.0067 | 0.2701 ± 0.0057 | Best RiceGeneFormer variant by test macro-F1 |
| RiceGeneFormer + expected-score distillation w=0.20 | test | 42/43/44 | 0.3251 ± 0.0054 | 0.5906 ± 0.0033 | 0.5904 ± 0.0037 | 0.2757 ± 0.0089 | Stronger distillation weight; not better than w=0.10 |
| LightGBM top512 | val | single run | 0.3354 | 0.6271 | 0.5547 | n/a | Strong tree baseline, high accuracy/MAE trade-off |
| XGBoost top512 | val | single run | 0.3115 | 0.6274 | 0.5662 | n/a | Similar accuracy to LightGBM but lower macro-F1 |
| SNP-MLP top512, balanced alpha0.40 | val | 42/43/44 | 0.3502 ± 0.0117 | 0.6141 ± 0.0097 | 0.5697 ± 0.0108 | 0.3162 ± 0.0038 | Best balanced macro-F1/accuracy trade-off baseline |
| SNP-MLP top512, balanced alpha0.60 | val | 42/43/44 | 0.3544 ± 0.0112 | 0.5961 ± 0.0086 | 0.5976 ± 0.0087 | 0.3103 ± 0.0108 | Highest macro-F1 baseline, with accuracy/MAE cost |

## Definitions

- macro-F1: unweighted mean of per-class F1 scores; higher is better and minority classes have equal weight.
- Accuracy: fraction of observed labels for which the predicted class equals the observed class; higher is better.
- MAE: mean absolute error in ordinal class units; lower is better.
- Spearman: rank correlation between predicted expected ordinal score and observed class label; higher is better.
- alpha: interpolation strength between unweighted and inverse-frequency class-balanced loss. alpha0.40 and alpha0.60 are partially class-balanced, not fully balanced.

## Manuscript wording boundary

Do not state that RiceGeneFormer outperforms all baselines. The supported statement is:

"RiceGeneFormer approached the LightGBM top-SNP baseline and improved with gated top-SNP fusion and mild class balancing, but balanced top-SNP SNP-MLP baselines remained the strongest predictors by macro-F1 under the current random split."

## Notes before final table submission

1. Tree baselines are currently reported from validation-role runs, whereas RiceGeneFormer final variants are deterministic test-role runs. This is acceptable for the current draft comparison only if explicitly labeled, but final submission should either regenerate deterministic test metrics for all baselines or present validation and test tables separately.
2. Add per-trait baseline metrics if Supplementary Table comparison is required.
3. Decide whether to present Spearman for tree baselines after exporting probability-derived expected scores under the same deterministic evaluator.
