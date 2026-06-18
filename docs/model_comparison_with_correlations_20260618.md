# Model comparison with classification and correlation metrics（模型分类与相关性指标对比）

Date（日期）: 2026-06-18

Metrics（指标说明）: macro-F1（宏平均F1，越高越好）, accuracy（准确率，越高越好）, MAE（mean absolute error，平均绝对误差，越低越好）, Pearson/PCC（Pearson correlation coefficient，皮尔逊相关；连续预测分数和真实等级的线性相关，越高越好）, Spearman（Spearman rank correlation，斯皮尔曼排序相关，越高越好）。

## Six-trait clean/post-hoc comparison（六性状干净重训/事后重算对比）

| 模型 / Model | 范围 / Scope | test macro-F1（测试集宏平均F1） | accuracy（准确率） | MAE（平均绝对误差） | mean trait Pearson/PCC（各性状平均皮尔逊相关） | pooled Pearson/PCC（混合标签皮尔逊相关） | mean trait Spearman（各性状平均斯皮尔曼相关） | pooled Spearman（混合标签斯皮尔曼相关） | observed labels（有效测试标签数） | 备注 / Note |
|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---|
| RiceGATE-MoE final（门控混合专家最终模型） | six-trait clean retrain（六性状干净重训） | 0.5088 | 0.6513 | 0.3640 | 0.3947 | NA | 0.3300 | 0.6911 | 1904 | threshold=0.05（门控阈值0.05）；conservative final（保守最终版） |
| Old 10-trait RiceGATE re-scored on same 6 traits（旧10性状RiceGATE在同6性状上重算） | post-hoc comparison only（仅事后比较） | 0.5197 | 0.6707 | 0.3424 | 0.4088 | NA | NA | NA | 1904 | not clean retrain（不是干净重训）；只能作为参考，不能作为主结果 |
| ExtraTrees top288 alpha0.85（极端随机树；每性状top288 SNP，alpha=0.85） | six-trait candidate expert（六性状候选专家） | 0.5033 | 0.6581 | 0.3556 | 0.3966 | 0.7580 | 0.3399 | 0.7554 | 1904 | best single six-trait ET by test macro-F1 among listed candidates（列出候选中测试macro-F1最高的单模型） |
| ExtraTrees top288 alpha1.00（极端随机树；每性状top288 SNP，alpha=1.00） | RiceGATE-MoE base expert（RiceGATE-MoE基础专家） | 0.4989 | 0.6413 | 0.3729 | 0.3916 | 0.7550 | 0.3376 | 0.7526 | 1904 | base expert used in RiceGATE-MoE final（最终RiceGATE-MoE使用的基础专家） |
| ExtraTrees top288 alpha1.05（极端随机树；每性状top288 SNP，alpha=1.05） | candidate expert（候选专家） | 0.4955 | 0.6303 | 0.3876 | 0.3929 | 0.7546 | 0.3389 | 0.7518 | 1904 | candidate expert（候选专家） |
| RiceGATE-MoE relaxed threshold=0.02（放宽门控阈值的门控混合专家） | exploratory run（探索性运行） | 0.5020 | 0.6481 | 0.3676 | NA | NA | NA | NA | 1904 | validation improved but test decreased（验证集更高但测试集下降）；metrics from six-trait result doc（指标来自六性状结果文档） |
| RiceGeneFormer raw p<0.05 union→gene（基因感知Transformer；raw p<0.05 SNP合集聚合到基因） | latest guarded GPU4 run（最新GPU4安全运行） | 0.2530 | 0.6413 | 0.3881 | -0.0138 | 0.6999 | -0.0397 | 0.6905 | 1904 | 4096 gene tokens（4096个基因token/基因标记）；47,985 SNPs covered after gene filtering（基因过滤后覆盖47,985个SNP） |

## Ten-trait context table（10性状旧基准上下文；不能和六性状干净重训直接混比）

| 模型 / Model | test macro-F1（测试集宏平均F1） | accuracy（准确率） | MAE（平均绝对误差） | mean trait Pearson/PCC（各性状平均皮尔逊相关） | pooled Pearson/PCC（混合标签皮尔逊相关） | observed labels（有效测试标签数） | 来源 / Source |
|---|---:|---:|---:|---:|---:|---:|---|
| 10-trait RiceGATE-MoE final（10性状门控混合专家最终模型） | 0.4235 | 0.6268 | 0.5830 | 0.3716 | 0.7599 | 3122 | docs/internal_model_pcc_comparison_20260617.md |
| 10-trait LightGBM top512（10性状LightGBM梯度提升树；每性状top512 SNP） | 0.3380 | 0.6182 | 0.5846 | 0.3714 | 0.7679 | 3122 | docs/internal_model_pcc_comparison_20260617.md |
| 10-trait XGBoost top512（10性状XGBoost梯度提升树；每性状top512 SNP） | 0.3161 | 0.6256 | 0.5717 | 0.3588 | 0.7577 | 3122 | docs/internal_model_pcc_comparison_20260617.md |
| 10-trait SNP-MLP seed ensemble（10性状SNP多层感知机种子集成） | 0.3950 | 0.6166 | 0.6031 | 0.3778 | 0.7558 | 3122 | docs/internal_model_pcc_comparison_20260617.md |

## Notes（说明）

- Six-trait rows use the same six included traits（六性状行都使用同一组6个保留性状）: CUDI_CODE_REPRO, CUNO_CODE_REPRO, LLT_CODE, PLT_CODE_POST, SDHT_CODE, PTH.
- RiceGATE-MoE final Pearson/PCC（RiceGATE-MoE最终模型皮尔逊相关）is taken from the existing six-trait result document where expected-score PCC was reconstructed（来自已重建连续期望分数PCC的六性状结果文档）；its Spearman values here are recomputed from saved discrete class predictions（这里的斯皮尔曼相关由保存的离散类别预测重算）, so they are conservative diagnostics（因此偏保守，仅作诊断）。
- ExtraTrees Pearson/Spearman values（ExtraTrees皮尔逊/斯皮尔曼相关）are recomputed from saved `teacher_expected_score`（教师模型期望分数）on the test split using `mask_multitask.npy`（测试集标签掩码）。
- RiceGeneFormer raw p<0.05 union values（RiceGeneFormer raw p<0.05 SNP合集结果）are recomputed from the saved best checkpoint（最佳检查点）on CPU for Pearson/Spearman（在CPU上重算相关性）；no retraining or test tuning was performed（没有重新训练或测试集调参）。
- Ten-trait context rows（10性状上下文行）are from the older 10-trait benchmark（旧10性状基准）, and should not be mixed as direct clean six-trait retrain evidence（不能作为六性状干净重训的直接证据混用）。
