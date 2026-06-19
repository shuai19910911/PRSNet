# PRSNet-2-paper-aligned five-trait comparison / 对齐 PRSNet-2 原文风格的五性状比较

Date（日期）: 2026-06-19

保留性状 / Included traits: CUDI_CODE_REPRO, LLT_CODE, PLT_CODE_POST, SDHT_CODE, PTH

主文策略 / Main-text strategy: 本表学习 PRSNet-2（SNP→基因→表型层级图神经网络）原文的比较方式，加入 RiceGraphOrdinalNet（本文新模型：效应量基因图有序分类网络），并只保留与方法学叙事直接相关的 PRS baseline（传统多基因评分基线）、neural baseline（神经网络基线）和 naive baseline（朴素基线）；ExtraTrees（极端随机树）和 RandomForest（随机森林）仅保留为内部探索记录，不进入主文主比较表。当前 PRS baseline（传统多基因评分基线）是 train-fold GWAS top-k additive PRS（只用训练集GWAS前k个SNP的加性多基因评分），不是完整 LD-clumped C+T（先LD剪枝再按P值阈值打分）实现。

说明 / Note: Pearson/PCC（皮尔逊相关）和 Spearman（斯皮尔曼相关）均基于 expected ordinal score（期望序数分数），不是 hard class（硬类别）编号；pooled PCC（混合标签皮尔逊相关）是辅助指标，因为不同性状类别范围不同。Majority baseline（多数类基线）的各性状预测为常数，所以 mean PCC（各性状平均皮尔逊相关）记为 NA。

| 模型 / Model | 类型 / Type | test macro-F1 mean trait（测试各性状宏平均F1均值） | pooled accuracy（混合标签准确率） | pooled MAE（混合标签平均绝对误差） | mean Pearson/PCC expected score（期望序数分数的各性状平均皮尔逊相关） | pooled Pearson/PCC expected score, secondary（期望序数分数的混合标签皮尔逊相关，辅助） | observed labels（有效标签数） |
|---|---|---:|---:|---:|---:|---:|---:|
| RiceGraphOrdinalNet（本文新模型：效应量基因图有序分类网络） | Proposed model（本文新模型） | 0.4418 | 0.5751 | 0.4507 | 0.3865 | 0.7502 | 1591 |
| PRSNet-2（SNP→基因→表型层级图神经网络） | Reference model（参考模型） | 0.4754 | 0.5864 | 0.4444 | 0.3878 | 0.7341 | 1591 |
| GWAS top-k additive PRS（训练集GWAS前k个SNP加性多基因评分） | PRS baseline（传统多基因评分基线） | 0.3319 | 0.6392 | 0.3658 | 0.3395 | 0.7622 | 1591 |
| SNP-MLP（SNP输入的普通多层感知机神经网络） | Neural baseline（神经网络基线） | 0.4476 | 0.6078 | 0.4142 | 0.3531 | 0.7449 | 1591 |
| Majority baseline（多数类基线） | Naive（朴素基线） | 0.2459 | 0.6185 | 0.3872 | NA | 0.6708 | 1591 |

## 运行文件 / Run files

- RiceGraphOrdinalNet（本文新模型）: `data/3krice/processed/rice_graph_ordinal_feature_net_fivetrait_top2048_h64_seed42_20260619`
- PRSNet-2: `data/3krice/processed/prsnet2_fivetrait_no_cuno_top4096_h64_k16_b8_gpu2_seed42_20260618`
- GWAS top-k additive PRS: `data/3krice/processed/ct_prs_fivetrait_no_cuno_top1000_seed42_20260619`
- SNP-MLP: `data/3krice/processed/snp_mlp_fivetrait_no_cuno_top256_union1024_h128_bal05_seed42_20260619`
- 五性状 SNP→gene map（SNP到基因映射）: `data/3krice/processed/snp_gene_map/rawp005_fivetrait_no_cuno_union_window10kb_top4096genes/`
- 五性状 gene graph（基因图）: `data/3krice/processed/gene_graph/mapping_ablation/rawp005_fivetrait_no_cuno_union_window10kb_top4096genes/chr_neighbor_k5/`

## 推荐主文表述 / Recommended wording

RiceGraphOrdinalNet（本文新模型：效应量基因图有序分类网络）was evaluated in the same paper-aligned（论文对齐）comparison frame as PRSNet-2（SNP→基因→表型层级图神经网络）. In this seed-42 run（42号随机种子运行）, RiceGraphOrdinalNet did not exceed PRSNet-2 on mean macro-F1（各性状宏平均F1均值）, but reached a comparable mean PCC（各性状平均皮尔逊相关） and a higher pooled PCC（混合标签皮尔逊相关，辅助指标） than PRSNet-2. Therefore it should be reported as a crop-adapted architecture candidate（作物适配架构候选） unless later validation improves the primary macro-F1（主指标宏平均F1）.

Do not claim（不要声称）: RiceGraphOrdinalNet already outperforms PRSNet-2（已经超过PRSNet-2） on the primary metric（主指标）. The current defensible claim（当前可防守表述） is that the architecture is original and biologically structured（结构原创且有生物学结构）, but the seed-42 primary-metric result still trails the PRSNet-2 reference model（参考模型）.
