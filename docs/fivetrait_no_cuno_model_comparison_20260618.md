# 五性状删除 CUNO_CODE_REPRO 后模型比较 / Five-trait no-CUNO model comparison

Date（日期）: 2026-06-18

保留性状 / Included traits: CUDI_CODE_REPRO, LLT_CODE, PLT_CODE_POST, SDHT_CODE, PTH

说明 / Note: Pearson/PCC（皮尔逊相关）和 Spearman（斯皮尔曼相关）均基于 expected ordinal score（期望序数分数），不是 hard class（硬类别）编号；pooled PCC（混合标签皮尔逊相关）是辅助指标，因为不同性状类别范围不同。Majority baseline（多数类基线）的各性状预测为常数，所以 mean PCC（各性状平均皮尔逊相关）记为 NA。

| 模型 / Model | test macro-F1 mean trait（测试各性状宏平均F1均值） | pooled accuracy（混合标签准确率） | pooled MAE（混合标签平均绝对误差） | mean Pearson/PCC expected score（期望序数分数的各性状平均皮尔逊相关） | pooled Pearson/PCC expected score, secondary（期望序数分数的混合标签皮尔逊相关，辅助） | observed labels（有效标签数） |
|---|---:|---:|---:|---:|---:|---:|
| PRSNet-2（多基因风险评分网络第二版） | 0.4754 | 0.5864 | 0.4444 | 0.3878 | 0.7341 | 1591 |
| ExtraTrees alpha0.85（极端随机树） | 0.4987 | 0.6380 | 0.3790 | 0.4076 | 0.7760 | 1591 |
| ExtraTrees alpha1.00（极端随机树） | 0.4991 | 0.6304 | 0.3903 | 0.4029 | 0.7745 | 1591 |
| RandomForest alpha0.85（随机森林） | 0.4506 | 0.6480 | 0.3664 | 0.4178 | 0.7804 | 1591 |
| Majority baseline（多数类基线） | 0.2459 | 0.6185 | 0.3872 | NA | 0.6708 | 1591 |

## 运行文件 / Run files

- PRSNet-2: `data/3krice/processed/prsnet2_fivetrait_no_cuno_top4096_h64_k16_b8_gpu2_seed42_20260618/`
- ExtraTrees alpha0.85: `data/3krice/processed/extratrees_fivetrait_no_cuno_top288_alpha085_seed44_20260618/`
- ExtraTrees alpha1.00: `data/3krice/processed/extratrees_fivetrait_no_cuno_top288_alpha100_seed44_20260618/`
- RandomForest alpha0.85: `data/3krice/processed/randomforest_fivetrait_no_cuno_top288_alpha085_seed44_20260618/`
- 五性状 SNP→gene map（SNP到基因映射）: `data/3krice/processed/snp_gene_map/rawp005_fivetrait_no_cuno_union_window10kb_top4096genes/`
- 五性状 gene graph（基因图）: `data/3krice/processed/gene_graph/mapping_ablation/rawp005_fivetrait_no_cuno_union_window10kb_top4096genes/chr_neighbor_k5/`
