# SmartPRSOrdinalNet 10-trait formal run report (2026-06-20)

## Run status

Status: completed successfully.

Run directory:

`data/3krice/processed/rice_smart_prs_ordinal_net_tentrait_top2048_h64_gpu5_seed42_20260620_113036`

Log file:

`logs/rice_smart_prs_ordinal_net_tentrait_top2048_h64_gpu5_seed42_20260620_113036.log`

GPU: gpu10 physical GPU5, A100 40GB. GPU2 was busy at launch, so GPU5 was used to avoid GPU0 and avoid interrupting existing GPU2 work.

Configuration:

- Model: SmartPRSOrdinalNet（基于PRSNet的多模态专家门控有序等级模型）
- Traits: 10 core ordinal traits（10个核心有序等级性状）
- Split: random_seed42
- max_genes: 2048
- max_snps_per_gene: 32
- global_prs_snps: 512
- hidden_dim: 64
- n_snp_kernels: 4
- n_gnn_layers: 1
- attention_heads: 4
- batch_size: 32
- epochs: 90
- selection_metric: val_macro_f1（验证集宏平均F1，重视少数类别）
- Best epoch: 86
- Runtime: 606.886 seconds

## Main result

| Model | macro-F1 | Accuracy | MAE | Mean PCC | Pooled PCC | Notes |
|---|---:|---:|---:|---:|---:|---|
| v1_raw | 0.3464 | 0.5192 | 0.6775 | 0.3306 | 0.7263 | previous raw RiceGraphOrdinalFeatureNet |
| v1_cal | 0.3604 | 0.5583 | 0.6512 | 0.3306 | 0.7263 | previous validation-threshold calibration |
| v2b_cal | 0.3606 | 0.5564 | 0.6316 | 0.3512 | 0.7454 | previous best MAE/PCC candidate |
| v2c_cal | 0.3621 | 0.5490 | 0.6758 | 0.3355 | 0.7290 | previous best macro-F1 candidate |
| SmartPRSOrdinalNet_raw | 0.3604 | 0.5154 | 0.7037 | 0.3317 | 0.6950 | raw decoder, no threshold calibration |
| SmartPRSOrdinalNet_cal | **0.3722** | **0.5682** | 0.6672 | 0.3317 | 0.6950 | validation-only threshold calibration |

## Improvement summary

SmartPRSOrdinalNet_cal is the current best macro-F1 candidate.

- vs v2c_cal: +0.0101845580 absolute macro-F1; +2.8130% relative.
- vs v2b_cal: +0.0116738976 absolute macro-F1; +3.2376% relative.
- vs v1_cal: +0.0118588349 absolute macro-F1; +3.2906% relative.
- vs v1_raw: +0.0258760415 absolute macro-F1; +7.4707% relative.
- vs SmartPRSOrdinalNet_raw: +0.0118376934 absolute macro-F1; +3.2846% relative, showing calibration is useful for this architecture.

## Calibration integrity

Calibration method:

`validation_only_expected_score_threshold_calibration`

Calibration role: validation split only.

Evaluation role: test split.

Test labels were not stored in `teacher_predictions_test.npz`; test labels were loaded from the protected processed dataset only for final evaluation after thresholds were frozen.

Calibration artifacts:

- `validation_threshold_calibration.json`
- `validation_threshold_calibration_per_trait.tsv`

## Formal test metrics

### Raw SmartPRSOrdinalNet

- accuracy: 0.5153747598
- macro-F1: 0.3604038635
- MAE: 0.7037155669
- mean Pearson/PCC: 0.3316571788
- pooled Pearson/PCC: 0.6949926782
- observed labels: 3122
- test samples: 340

### Calibrated SmartPRSOrdinalNet

- accuracy: 0.5682254965
- macro-F1: 0.3722415569
- MAE: 0.6672005125
- mean Pearson/PCC: 0.3316570096
- pooled Pearson/PCC: 0.6949926926
- observed labels: 3122
- test samples: 340

## Validation calibration per trait

| Trait | n classes | val raw macro-F1 | val calibrated macro-F1 |
|---|---:|---:|---:|
| CUDI_CODE_REPRO | 2 | 0.6202 | 0.6637 |
| CULT_CODE_REPRO | 7 | 0.2917 | 0.3818 |
| CUNO_CODE_REPRO | 3 | 0.4099 | 0.4496 |
| LLT_CODE | 5 | 0.3100 | 0.3298 |
| PLT_CODE_POST | 4 | 0.2979 | 0.3801 |
| SDHT_CODE | 3 | 0.4005 | 0.4631 |
| PTH | 3 | 0.4319 | 0.4651 |
| SPKF | 5 | 0.3043 | 0.3454 |
| CUST_REPRO | 8 | 0.1561 | 0.2764 |
| PSH | 3 | 0.3401 | 0.3572 |

## Interpretation

SmartPRSOrdinalNet successfully transfers the Huihui Li-style modular fusion idea into a PRSNet-based ordinal crop model. The raw model is already close to the previous best calibrated RiceGraphOrdinalNet, and validation-only calibration pushes it to a new best macro-F1.

The trade-off is that pooled/mean PCC is lower than v2b_cal, so the result should be framed as macro-F1-prioritized and classification-oriented rather than uniformly superior on all ranking metrics.

## Recommended next step

Before making a manuscript-level headline claim, rerun SmartPRSOrdinalNet_cal with at least 3 seeds（随机种子重复）and optionally one larger configuration (`max_genes=4096`, `hidden_dim=96`) to check stability.
