# Final per-trait summary for manuscript

Model: RiceGeneFormer + expected-score distillation w=0.10, deterministic test role, seeds 42/43/44. This is the best RiceGeneFormer test macro-F1 variant among final runs.

| Trait | Classes | Test observed | macro-F1 mean±SD | Accuracy mean±SD | MAE mean±SD | Spearman mean±SD |
|---|---:|---:|---:|---:|---:|---:|
| CUDI_CODE_REPRO | 2 | 315 | 0.639±0.019 | 0.687±0.007 | 0.313±0.007 | 0.345±0.005 |
| CULT_CODE_REPRO | 7 | 312 | 0.218±0.026 | 0.299±0.030 | 0.970±0.041 | 0.520±0.021 |
| CUNO_CODE_REPRO | 3 | 313 | 0.299±0.015 | 0.732±0.006 | 0.268±0.006 | 0.274±0.021 |
| CUST_REPRO | 8 | 340 | 0.130±0.006 | 0.208±0.015 | 2.153±0.018 | 0.285±0.016 |
| LLT_CODE | 5 | 313 | 0.291±0.013 | 0.521±0.011 | 0.508±0.006 | 0.433±0.006 |
| PLT_CODE_POST | 4 | 311 | 0.393±0.025 | 0.822±0.004 | 0.182±0.005 | 0.337±0.010 |
| PSH | 3 | 226 | 0.339±0.010 | 0.810±0.008 | 0.208±0.008 | 0.093±0.006 |
| PTH | 3 | 339 | 0.450±0.015 | 0.495±0.007 | 0.530±0.006 | 0.287±0.006 |
| SDHT_CODE | 3 | 313 | 0.361±0.010 | 0.720±0.004 | 0.283±0.004 | 0.170±0.022 |
| SPKF | 5 | 340 | 0.172±0.014 | 0.694±0.018 | 0.320±0.022 | -0.044±0.040 |
