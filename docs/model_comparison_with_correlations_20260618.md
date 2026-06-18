# 六性状模型对比 / Six-trait model comparison

Date（日期）: 2026-06-18

保留性状 / Included traits: CUDI_CODE_REPRO, CUNO_CODE_REPRO, LLT_CODE, PLT_CODE_POST, SDHT_CODE, PTH

指标说明 / Metrics:
- macro-F1（宏平均F1）: 越高越好；主指标。
- accuracy（准确率）: 越高越好。
- MAE（平均绝对误差）: 越低越好。
- mean Pearson/PCC（各性状平均皮尔逊相关）: 越高越好；用于衡量连续预测分数与真实等级的相关性。
- pooled Pearson/PCC（混合标签皮尔逊相关）: 辅助指标；不同性状类别范围不同，不作为主结论。

| 模型 / Model | 类型 / Type | test macro-F1 | accuracy | MAE | mean Pearson/PCC | pooled Pearson/PCC | 结论 / Conclusion |
|---|---|---:|---:|---:|---:|---:|---|
| RiceGATE-MoE（门控混合专家） | final clean retrain（最终六性状干净重训） | 0.5088 | 0.6513 | 0.3640 | 0.3947 | NA | 主结果；macro-F1最高，作为论文主模型。 |
| ExtraTrees alpha0.85（极端随机树） | strongest single-tree baseline（最强单树基线） | 0.5033 | 0.6581 | 0.3556 | 0.3966 | 0.7580 | accuracy、MAE、PCC略好；可作为强基线。 |
| ExtraTrees alpha1.00（极端随机树） | RiceGATE base expert（RiceGATE基础专家） | 0.4989 | 0.6413 | 0.3729 | 0.3916 | 0.7550 | RiceGATE的基础专家；用于解释MoE来源。 |
| RiceGeneFormer raw p<0.05 union→gene（基因感知Transformer） | new GPU4 run（最新GPU4训练） | 0.2530 | 0.6413 | 0.3881 | -0.0138 | 0.6999 | 效果不佳；不作为主结果，只能作为负结果/探索。 |
| Old 10-trait RiceGATE re-scored on same 6 traits（旧10性状模型在6性状上重算） | post-hoc reference（事后参考） | 0.5197 | 0.6707 | 0.3424 | 0.4088 | NA | 不是干净重训，不能作为正式主结果；只说明旧模型上限。 |

## 推荐论文用法 / Recommended manuscript use

1. 主表只放前三行：RiceGATE-MoE、ExtraTrees alpha0.85、ExtraTrees alpha1.00。
2. RiceGeneFormer 放补充表或消融讨论：说明“大规模raw p<0.05 SNP union经基因聚合后仍未提升”。
3. 旧10性状post-hoc结果不要放主表；最多在补充材料说明“不是clean retrain”。

## 一句话结论 / One-sentence conclusion

在六性状干净重训中，RiceGATE-MoE（门控混合专家）取得最高test macro-F1（0.5088），但ExtraTrees alpha0.85（极端随机树）在accuracy、MAE和PCC上略优；最新RiceGeneFormer基因聚合版本表现明显不足，不建议作为主模型。
