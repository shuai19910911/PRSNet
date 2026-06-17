# Internal model comparison with PCC/Pearson correlation

Generated from existing prediction exports and checkpoint forward passes on 2026-06-17. PCC/Pearson is computed between true ordinal labels and the continuous predicted ordinal score. Main PCC is the mean of per-trait Pearson correlations; pooled PCC is reported as a secondary diagnostic across all observed trait-sample labels.


| Rank | Model                                         | macro-F1 | classification accuracy | MAE    | mean trait PCC | pooled PCC | PCC score source                       |
| ---- | --------------------------------------------- | -------- | ----------------------- | ------ | -------------- | ---------- | -------------------------------------- |
| 1    | RiceGATE-MoE final                            | 0.4235   | 0.6268                  | 0.5830 | 0.3716         | 0.7599     | reconstructed_final_expected_score     |
| 2    | alpha3 ExtraTrees average base                | 0.4182   | 0.6134                  | 0.6003 | 0.3614         | 0.7577     | reconstructed_base_expected_score      |
| 3    | alpha3 ExtraTrees per-trait tuned             | 0.4170   | 0.6137                  | 0.5987 | 0.3630         | 0.7585     | reconstructed_per_trait_expected_score |
| 4    | alpha3 ExtraTrees global tuned                | 0.4147   | 0.6079                  | 0.6095 | 0.3609         | 0.7571     | reconstructed_global_expected_score    |
| 5    | best single ExtraTrees top288 alpha1.0        | 0.4129   | 0.6060                  | 0.6063 | 0.3588         | 0.7570     | saved_expected_score                   |
| 6    | ExtraTrees top288 5-seed ensemble             | 0.4092   | 0.6079                  | 0.6063 | 0.3608         | 0.7568     | reconstructed_per_trait_expected_score |
| 7    | ExtraTrees top288 p-value order               | 0.4081   | 0.6051                  | 0.6076 | 0.3609         | 0.7570     | saved_expected_score                   |
| 8    | ExtraTrees top256 3-seed ensemble             | 0.4066   | 0.5983                  | 0.6143 | 0.3627         | 0.7573     | reconstructed_per_trait_expected_score |
| 9    | ExtraTrees top256/top288 10-model ensemble    | 0.4058   | 0.6009                  | 0.6121 | 0.3628         | 0.7573     | reconstructed_per_trait_expected_score |
| 10   | SNP-MLP seed ensemble                         | 0.3950   | 0.6166                  | 0.6031 | 0.3778         | 0.7558     | reconstructed_per_trait_expected_score |
| 11   | RandomForest top288 alpha1.0                  | 0.3836   | 0.6185                  | 0.6003 | 0.3665         | 0.7591     | saved_expected_score                   |
| 12   | best single SNP-MLP seed44                    | 0.3802   | 0.5990                  | 0.6111 | 0.3736         | 0.7554     | saved_expected_score                   |
| 13   | SNP-MLP seed43                                | 0.3794   | 0.6163                  | 0.5939 | 0.3691         | 0.7504     | saved_expected_score                   |
| 14   | top2% ExtraTrees                              | 0.3794   | 0.6429                  | 0.5314 | 0.3957         | 0.7708     | saved_expected_score                   |
| 15   | SNP-MLP seed44 macro-selected                 | 0.3772   | 0.5974                  | 0.6153 | 0.3580         | 0.7520     | saved_expected_score                   |
| 16   | SNP-MLP seed42                                | 0.3748   | 0.6063                  | 0.6294 | 0.3741         | 0.7468     | saved_expected_score                   |
| 17   | top0.5% ExtraTrees                            | 0.3744   | 0.6348                  | 0.5679 | 0.3856         | 0.7692     | saved_expected_score                   |
| 18   | top10% ExtraTrees                             | 0.3741   | 0.6387                  | 0.5340 | 0.4085         | 0.7738     | saved_expected_score                   |
| 19   | top1% ExtraTrees                              | 0.3731   | 0.6352                  | 0.5522 | 0.3935         | 0.7700     | saved_expected_score                   |
| 20   | top5% ExtraTrees                              | 0.3674   | 0.6304                  | 0.5583 | 0.4025         | 0.7726     | saved_expected_score                   |
| 21   | RiceGeneFormer p<0.05 cap20000 class-head     | 0.3385   | 0.5807                  | 0.6672 | 0.2982         | 0.7358     | rgf_checkpoint_class_score             |
| 22   | LightGBM top512                               | 0.3380   | 0.6182                  | 0.5846 | 0.3714         | 0.7679     | retrained_lightgbm_expected_score      |
| 23   | RiceGeneFormer top2% merged SNP               | 0.3233   | 0.5794                  | 0.6807 | 0.2763         | 0.7468     | rgf_checkpoint_class_score             |
| 24   | RiceGeneFormer p<0.05 cap20000 expected-score | 0.3168   | 0.5054                  | 0.6970 | 0.2729         | 0.7345     | rgf_checkpoint_ordinal_score           |
| 25   | XGBoost top512                                | 0.3161   | 0.6256                  | 0.5717 | 0.3588         | 0.7577     | retrained_xgboost_expected_score       |
| 26   | RiceGeneFormer GWAS top2048 early pilot       | 0.2990   | 0.5836                  | 0.5878 | 0.2712         | 0.7454     | rgf_checkpoint_ordinal_score           |
| 27   | train-median class baseline                   | 0.2103   | 0.5887                  | 0.6140 | NA             | 0.6654     | train_median_class_prediction          |
| 28   | train-majority class baseline                 | 0.2098   | 0.5849                  | 0.7050 | NA             | 0.4707     | train_majority_class_prediction        |


Notes:

- mean trait PCC = average Pearson correlation across the 10 core ordinal traits. This is closest to how genomic prediction papers report prediction accuracy per trait.
- pooled PCC = Pearson correlation after pooling all observed ordinal labels across traits; because traits have different class ranges, use it only as a secondary diagnostic.
- expected_score/probability score uses sum(class index times predicted probability); class_prediction uses the final discrete predicted class when a model has no probability output.
- RiceGeneFormer entries were re-evaluated by loading saved checkpoints and running CPU test forward passes only; no training or test-tuning was performed. Missing genotype code 3 was imputed to dosage 1.0, matching the training script convention.

