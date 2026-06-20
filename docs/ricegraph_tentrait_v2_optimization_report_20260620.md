# RiceGraphOrdinalNet 10-trait optimization report (2026-06-20)

## Overall test comparison

| Model | macro-F1 | Accuracy | MAE | Mean PCC | Pooled PCC |
|---|---:|---:|---:|---:|---:|
| v1_raw | 0.3464 | 0.5192 | 0.6775 | 0.3306 | 0.7263 |
| v1_cal | 0.3604 | 0.5583 | 0.6512 | 0.3306 | 0.7263 |
| v2b_cal | 0.3606 | 0.5564 | 0.6316 | 0.3512 | 0.7454 |
| v2c_cal | 0.3621 | 0.5490 | 0.6758 | 0.3355 | 0.7290 |

Notes: calibration uses validation labels only to tune per-trait score-to-class thresholds; test labels are used only for final evaluation.

## Best calibrated candidate vs original v1 raw, per trait

| Trait | n | classes | v1 raw macro-F1 | v2c cal macro-F1 | Delta macro-F1 | v1 raw acc | v2c cal acc | v1 raw MAE | v2c cal MAE | v1 raw PCC | v2c cal PCC |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| CUDI_CODE_REPRO | 315 | 2 | 0.6676 | 0.6717 | +0.0041 | 0.7111 | 0.7048 | 0.2889 | 0.2952 | 0.3777 | 0.4461 |
| CULT_CODE_REPRO | 312 | 7 | 0.2964 | 0.2353 | -0.0610 | 0.3269 | 0.2917 | 0.9968 | 0.9808 | 0.6329 | 0.6425 |
| CUNO_CODE_REPRO | 313 | 3 | 0.3878 | 0.4750 | +0.0872 | 0.5112 | 0.7284 | 0.4952 | 0.2780 | 0.2957 | 0.3107 |
| LLT_CODE | 313 | 5 | 0.2951 | 0.3168 | +0.0217 | 0.4952 | 0.4888 | 0.5495 | 0.5623 | 0.5113 | 0.5102 |
| PLT_CODE_POST | 311 | 4 | 0.4184 | 0.4205 | +0.0021 | 0.7781 | 0.7942 | 0.2251 | 0.2090 | 0.4623 | 0.4754 |
| SDHT_CODE | 313 | 3 | 0.3522 | 0.3525 | +0.0003 | 0.7125 | 0.5367 | 0.2875 | 0.4824 | 0.2224 | 0.2358 |
| PTH | 339 | 3 | 0.4824 | 0.4687 | -0.0137 | 0.4897 | 0.4838 | 0.5870 | 0.5752 | 0.3398 | 0.3315 |
| SPKF | 340 | 5 | 0.1827 | 0.2032 | +0.0206 | 0.4559 | 0.4765 | 0.6735 | 0.7441 | 0.1042 | 0.0854 |
| CUST_REPRO | 340 | 8 | 0.0522 | 0.1552 | +0.1030 | 0.1029 | 0.2676 | 2.1088 | 2.1794 | 0.3204 | 0.3502 |
| PSH | 226 | 3 | 0.3289 | 0.3216 | -0.0073 | 0.7035 | 0.8319 | 0.3584 | 0.1903 | 0.0389 | -0.0334 |
| ALL_10_TRAITS | 3122 |  | 0.3464 | 0.3621 | +0.0157 | 0.5192 | 0.5490 | 0.6775 | 0.6758 | 0.3306 | 0.3355 |

## Interpretation

The strongest macro-F1 candidate is v2c_cal: conservative ordinal/categorical hybrid with one graph layer plus validation-only threshold calibration. It improves macro-F1 over v1_raw, but its MAE is worse than v2b_cal, so it should be described as macro-F1-prioritized rather than uniformly superior.
