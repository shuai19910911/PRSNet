# SmartPRSOrdinalNet external validation on BGLR wheat (2026-06-20)

## Dataset

Dataset: BGLR wheat（BGLR R包公开小麦数据集，CIMMYT 599份小麦材料）

Source: CRAN BGLR 1.1.4 `wheat` data.

Data shape:

- Samples（样本/品系）: 599
- Markers（分子标记）: 1279 DArT markers（DArT分子标记，0/1存在缺失型标记）
- Phenotypes（表型）: 4 grain-yield mega-environment traits（4个籽粒产量环境性状）

The original phenotypes are continuous grain-yield values. To test the ordinal SmartPRSOrdinalNet（有序等级预测模型）, each trait was discretized into 4 ordinal bins（4个有序等级） using train-only quantile thresholds（只用训练集分位数阈值）. Validation/test labels were assigned using these frozen train thresholds.

Split:

- train（训练集）: 359
- validation（验证集）: 120
- test（测试集）: 120
- seed（随机种子）: 42

## Model adaptation

Model: SmartPRSOrdinalNet_external_pseudogene_graph_prs_gate

Because this external wheat dataset has markers but no gene coordinate map in the BGLR package, I used a conservative external adapter:

1. Marker matrix（分子标记矩阵） -> pseudo-gene groups（伪基因块，把相邻marker分组）.
2. Multi-kernel marker aggregation（多核marker聚合，模拟PRSNet SNP-to-gene层）.
3. Chain graph message passing（链式图消息传递，相邻伪基因块之间传递信息）.
4. Trait attention（性状注意力，为每个环境性状读取不同marker块信号）.
5. PRS branch（多基因评分分支，使用训练集估计的marker效应）.
6. Expert gate（专家门控，融合ordinal/categorical/PRS专家输出）.
7. Validation-only threshold calibration（只用验证集调阈值校准）.

Best completed configuration:

- pseudo-gene groups（伪基因块）: 32
- hidden_dim（隐藏维度）: 32
- graph layers（图层数）: 1
- kernels（聚合核）: 4
- best epoch（最佳轮次）: 7
- completed epochs（完成轮次）: 15
- elapsed_seconds（运行时间）: 27.878

A second configuration (`n_gene_groups=16`, `hidden_dim=64`) was also tried, but it was worse, so it is not used as the best external candidate.

## Overall test comparison

| Model | macro-F1 | Accuracy | MAE | Mean PCC | Pooled PCC | Notes |
|---|---:|---:|---:|---:|---:|---|
| Majority_train_only | 0.1078 | 0.2750 | 1.4542 | NA | NA | train-only majority class baseline（多数类基线） |
| ExtraTrees_all_markers | **0.3470** | **0.3667** | **0.9375** | **0.3995** | **0.4000** | ExtraTrees（极端随机树，强CPU树模型基线） |
| SmartPRSOrdinalNet_raw | 0.2614 | 0.3083 | 1.1667 | 0.2335 | 0.2319 | raw neural output（神经网络原始输出） |
| SmartPRSOrdinalNet_cal | 0.3328 | 0.3438 | 0.9958 | 0.2335 | 0.2319 | validation-only calibrated（验证集阈值校准） |

## Interpretation

SmartPRSOrdinalNet transferred to a non-rice plant dataset successfully and substantially beat the naive majority baseline:

- SmartPRSOrdinalNet_cal vs Majority: +0.2249908359 absolute macro-F1; +208.7404% relative.
- Calibration helped the neural model strongly: SmartPRSOrdinalNet_cal vs SmartPRSOrdinalNet_raw: +0.0713987605 absolute macro-F1; +27.3164% relative.

However, on this small wheat dataset, ExtraTrees_all_markers remained stronger:

- SmartPRSOrdinalNet_cal vs ExtraTrees: -0.0142293232 absolute macro-F1; -4.1006% relative.

So this is a mixed but useful external validation result: the architecture runs and generalizes beyond rice, but for small marker-only wheat data without a real gene map/biological graph, a strong tree baseline still wins slightly.

## Per-trait test table

| Trait | n | classes | raw macro-F1 | cal macro-F1 | delta | raw acc | cal acc | raw MAE | cal MAE | PCC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| GY_ME1 | 120 | 4 | 0.2792 | 0.2602 | -0.0190 | 0.3083 | 0.2667 | 1.2667 | 1.0083 | 0.1332 |
| GY_ME2 | 120 | 4 | 0.2634 | 0.3285 | +0.0652 | 0.3167 | 0.3417 | 1.0417 | 1.0083 | 0.4080 |
| GY_ME4 | 120 | 4 | 0.2944 | 0.3334 | +0.0390 | 0.3333 | 0.3417 | 1.0750 | 1.0250 | 0.2504 |
| GY_ME5 | 120 | 4 | 0.2085 | 0.4090 | +0.2004 | 0.2750 | 0.4250 | 1.2833 | 0.9417 | 0.1424 |

## Artifacts

Best run directory:

`data/external/wheat_bglr/runs/smart_prs_ordinal_seed42_fast30_20260620`

Key files:

- `training_manifest.json`
- `model_comparison.tsv`
- `per_trait_test_table.tsv`
- `training_metrics.tsv`
- `checkpoint_best.pt`
- `teacher_predictions_train.npz`
- `teacher_predictions_val.npz`
- `teacher_predictions_test.npz`

Integrity checks:

- Python compile passed.
- Static scan passed: no hardcoded secrets, shell injection, eval/exec, unsafe pickle, or SQL injection.
- `training_manifest.json` parses with strict JSON tooling.
- `checkpoint_best.pt` exists and is non-empty.
- `teacher_predictions_test.npz` deliberately excludes test labels (`y`) and continuous phenotypes (`y_continuous`).

## Manuscript-facing conclusion

This wheat run should be reported as external feasibility evidence, not as a headline superiority result. It supports the claim that the SmartPRSOrdinalNet architecture is portable across plant species/datasets, but it also shows that when external data lacks real SNP-to-gene mapping and biological graph priors, a strong marker-level tree baseline can remain competitive or slightly better.
