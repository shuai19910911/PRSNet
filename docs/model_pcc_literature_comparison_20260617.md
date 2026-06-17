# PCC/Pearson comparison: PRSNet/RiceGATE-MoE vs published crop genomic prediction models

This table separates classification accuracy (分类准确率: exact class match), macro-F1 (宏平均F1: class-balanced F1 score), MAE (mean absolute error, 平均绝对误差), and PCC/Pearson correlation (皮尔逊相关系数: continuous-score prediction accuracy). In crop genomic prediction papers, “prediction accuracy” usually means PCC/Pearson correlation, not classification accuracy.

For our ordinal classification models, PCC was computed from the continuous expected ordinal score, i.e. sum(class index times predicted probability). The main PCC value is mean per-trait PCC across the 10 traits. Pooled PCC is available in `docs/internal_model_pcc_comparison_20260617.tsv` but should be treated as secondary because traits have different class ranges.

Key interpretation: RiceGATE-MoE remains the best classification model by macro-F1, but if PCC alone is the target, top10% ExtraTrees has the highest internal mean per-trait PCC. This shows macro-F1 and PCC reward different behaviour.

## Combined comparison table

| Scope | Model | Task | macro-F1 | classification accuracy | MAE | PCC / prediction accuracy | Notes |
|---|---|---|---:|---:|---:|---|---|
| ours/internal | RiceGATE-MoE final | ordinal multi-class classification | 0.4235 | 0.6268 | 0.5830 | 0.3716 | Best classification model by macro-F1; PCC is not the selection metric. |
| ours/internal | top10% ExtraTrees | ordinal multi-class classification | 0.3741 | 0.6387 | 0.5340 | 0.4085 | Highest internal mean per-trait PCC among current 28 rows, but lower macro-F1 than RiceGATE-MoE. |
| ours/internal | top5% ExtraTrees | ordinal multi-class classification | 0.3674 | 0.6304 | 0.5583 | 0.4025 | High PCC baseline; useful as PCC-oriented comparator. |
| ours/internal | top2% ExtraTrees | ordinal multi-class classification | 0.3794 | 0.6429 | 0.5314 | 0.3957 | High classification accuracy and PCC among GWAS-filtered tree baselines. |
| ours/internal | SNP-MLP seed ensemble | ordinal multi-class classification | 0.3950 | 0.6166 | 0.6031 | 0.3778 | Best internal neural baseline by mean per-trait PCC among SNP-MLP rows. |
| ours/internal | LightGBM top512 | ordinal multi-class classification | 0.3380 | 0.6182 | 0.5846 | 0.3714 | Boosting baseline; PCC recomputed by retraining original lightweight script to recover probability scores. |
| ours/internal | RiceGeneFormer p<0.05 cap20000 class-head | ordinal multi-class classification | 0.3385 | 0.5807 | 0.6672 | 0.2982 | Graph/neural baseline; checkpoint forward only, no retraining. |
| published/literature | Cropformer regression | continuous-trait regression | not reported | not applicable | not central | DTT 0.922; PH 0.918; EW 0.763 | Same broad genotype-to-phenotype setting, but continuous traits rather than our 10 ordinal classes. |
| published/literature | Cropformer DTT three-class | single-trait three-class classification | 0.771 | 0.772 | not reported | not applicable | Closest published classification comparator, but much easier than 10-trait ordinal classification. |
| published/literature | Cropformer DTT two-class | single-trait binary classification | 0.835 | 0.834 | not reported | not applicable | Binary single-trait task; not directly comparable to our multi-class/multi-trait task. |
| published/literature | DNNGP | continuous-trait regression | not reported | not reported | not central | average across traits reported as 0.878 in accessible ResearchGate snippet; article defines prediction accuracy as correlation coefficient | Continuous regression; useful only for context on PCC scale, not for ordinal classification. |
| published/literature | GP-WAITER | continuous-trait regression | not reported | not reported | reports MSE/RMSE/MAE | uses Pearson/PCC; reports up to 77.5% relative improvement in prediction accuracy | Architecture-related but regression-oriented; do not compare absolute PCC to ordinal-class PCC directly. |
| published/literature | NetGP | continuous-trait regression / multi-omics prediction | not reported | not reported | uses SmoothL1 in model description | uses PCC; Figure 4 compares NetGP against baselines; exact figure values not extracted from HTML | Good methods-context citation; exact per-trait numeric values require figure/supplement extraction. |
| published/literature | SoyDNGP | classification and regression, depending on trait | not found in accessible text | reported for classification tasks FC/PDENS/POD/ST, exact values not extracted due publisher/PDF access blocking | not summarized here | reports superior predictive accuracy vs DeepGS/DNNGP; exact PCC/accuracy table not extracted here | BIB-relevant method; cite qualitatively unless exact table is manually recovered from PDF/supplement. |
| published/literature | CropARNet | continuous-trait genomic prediction | not reported | not reported | not summarized in abstract | ranked first for 29/53 traits; exact PCC values not in abstract | Broad crop GP context; abstract gives rank not exact PCC. |
| published/literature | ResDeepGS | continuous-trait regression | not reported | not reported | not summarized here | wheat prediction accuracy improved by 5%-9% over baselines | Regression context; useful as DeepGS-family reference. |
| published/literature | EMLGP | continuous-trait regression / genomic prediction | not reported | not reported | not summarized here | highest prediction accuracy/correlation coefficient 0.92 | Very high continuous-trait PCC; not directly comparable to ordinal-class prediction. |

## Practical manuscript wording

Recommended sentence: “Because most crop genomic prediction studies report Pearson correlation-based prediction accuracy for continuous traits, we additionally computed PCC using the expected ordinal score. RiceGATE-MoE achieved mean per-trait PCC = 0.3716 while retaining the best macro-F1 = 0.4235; PCC-optimized tree baselines reached mean per-trait PCC up to 0.4085 but had substantially lower macro-F1, highlighting the distinction between ranking-oriented prediction accuracy and minority-class-sensitive ordinal classification performance.”

## Files

- Full internal 28-row table: `docs/internal_model_pcc_comparison_20260617.tsv` and `docs/internal_model_pcc_comparison_20260617.md`.

- This literature-facing comparison: `docs/model_pcc_literature_comparison_20260617.tsv` and `docs/model_pcc_literature_comparison_20260617.md`.

## Caveats

- Published PCC values are mostly continuous-trait regression results; they should not be interpreted as directly higher/lower than our ordinal multi-class classification PCC.

- Cropformer classification results are closer in metric type, but they are single-trait binary/three-class DTT tasks, not 10-trait ordinal classification.

- SoyDNGP and NetGP exact figure/table values were not fully extracted because publisher/supplement files were blocked or figure-only; the table marks these as qualitative/context rows rather than exact numeric comparisons.

