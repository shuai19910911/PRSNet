# Six-trait retraining after removing SPKF, PSH, CUST_REPRO and CULT_CODE_REPRO

Removed traits: `SPKF`, `PSH`, `CUST_REPRO`, `CULT_CODE_REPRO`. Included traits: `CUDI_CODE_REPRO`, `CUNO_CODE_REPRO`, `LLT_CODE`, `PLT_CODE_POST`, `SDHT_CODE`, `PTH`.

The clean retrain uses `--include-traits` so the training/evaluation scripts load only the six included traits; the removed traits are not present in the model head, probability exports, gate scan, or metrics.

## Main comparison

| Model | val macro-F1 | test macro-F1 | test accuracy | test MAE | mean trait PCC | observed labels | note |
|---|---:|---:|---:|---:|---:|---:|---|
| Old 10-trait RiceGATE evaluated only on same 6 traits | NA | 0.5197 | 0.6707 | 0.3424 | 0.4088 | 1904 | Post-hoc comparison only; not a clean retrain. |
| Six-trait RiceGATE final, threshold 0.05 | 0.4206 | 0.5088 | 0.6513 | 0.3640 | 0.3947 | 1904 | Clean retrain; no gates opened at conservative threshold. |
| Six-trait RiceGATE relaxed, threshold 0.02 | 0.4294 | 0.5020 | 0.6481 | 0.3676 | NA | 1904 | Exploratory only; validation improved but test decreased. |

## Retrained six-trait ExtraTrees expert candidates

| Expert | val macro-F1 | test macro-F1 | test accuracy | test MAE |
|---|---:|---:|---:|---:|
| sixtrait_et_top256_leaf5_alpha100_argpart_seed43_20260617 | 0.4122 | 0.4558 | 0.5793 | 0.4433 |
| sixtrait_et_top288_leaf2_alpha0p85_argpart_seed43_20260617 | 0.4061 | 0.5033 | 0.6581 | 0.3556 |
| sixtrait_et_top288_leaf2_alpha100_argpart_seed43_20260617 | 0.4179 | 0.4989 | 0.6413 | 0.3729 |
| sixtrait_et_top288_leaf2_alpha100_argpart_seed44_20260617 | 0.4160 | 0.4825 | 0.6318 | 0.3855 |
| sixtrait_et_top288_leaf2_alpha1p05_argpart_seed43_20260617 | 0.4227 | 0.4955 | 0.6303 | 0.3876 |
| sixtrait_et_top320_leaf2_alpha100_argpart_seed43_20260617 | 0.4147 | 0.4957 | 0.6455 | 0.3708 |
| sixtrait_et_top512_leaf4_alpha100_argpart_seed42_20260617 | 0.4179 | 0.4845 | 0.6234 | 0.3950 |

## Interpretation

- The clean six-trait retrain improves over the all-10-trait headline macro-F1 because four low/unstable traits are removed, but it is slightly lower than simply re-scoring the old 10-trait model on the same six traits.
- The conservative six-trait RiceGATE gate threshold 0.05 opened no gates, so the final predictor is the alpha-balanced top288 ExtraTrees base ensemble.
- Lowering the gate threshold to 0.02 increased validation macro-F1 but reduced test macro-F1, so the conservative 0.05 model is retained.
