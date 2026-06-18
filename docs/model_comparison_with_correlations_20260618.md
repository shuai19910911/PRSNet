# 六性状 PRSNet-2 首轮结果 / Six-trait PRSNet-2 first run

Date（日期）: 2026-06-18

本轮已废弃 RiceGATE-MoE（门控混合专家）作为主线；主线切回 PRSNet-2（多基因风险评分网络第二版）。下表不再列 RiceGATE-MoE。

保留性状 / Included traits: CUDI_CODE_REPRO, CUNO_CODE_REPRO, LLT_CODE, PLT_CODE_POST, SDHT_CODE, PTH

## 主表 / Main table

| 模型 / Model | 架构 / Architecture | test macro-F1（测试宏平均F1） | accuracy（准确率） | MAE（平均绝对误差） | mean Pearson/PCC（各性状平均皮尔逊相关） | pooled Pearson/PCC（混合标签皮尔逊相关） | 结论 / Conclusion |
|---|---|---:|---:|---:|---:|---:|---|
| PRSNet-2（多基因风险评分网络第二版） | SNP→gene multi-kernel（SNP到基因多核聚合） + GIN-like graph message passing（类GIN图消息传递） + trait attention readout（性状注意力读出） | 0.4521 | 0.5777 | 0.4480 | 0.3804 | 0.7224 | 已完成首轮；有相关性信号，但macro-F1暂低于ExtraTrees强基线。 |
| ExtraTrees alpha0.85（极端随机树） | strongest single-tree baseline（最强单树基线） | 0.5033 | 0.6581 | 0.3556 | 0.3966 | 0.7580 | 当前必要强基线；首轮PRSNet-2尚未超过它。 |
| RiceGeneFormer raw p<0.05 union→gene（基因感知Transformer） | previous exploratory graph model（前序探索图模型） | 0.2530 | 0.6413 | 0.3881 | -0.0138 | 0.6999 | 负结果/探索对照，不作主模型。 |

## PRSNet-2 首轮训练设置 / PRSNet-2 run setting

- split（划分）: random_seed42
- train/val/test samples（训练/验证/测试样本数）: 1586 / 340 / 340
- observed test labels（测试有效标签数）: 1904
- genes used（使用基因数）: 4096
- SNPs per gene（每基因SNP上限）: 64
- SNP kernels（SNP核数量）: 16
- hidden dim（隐藏维度）: 64
- graph（基因图）: chr_neighbor_k5, 40600 directed edges（有向边）
- best epoch（最佳轮次）: 11
- epochs completed（完成轮次）: 16 / 25, early stopped（早停）
- GPU（图形处理器）: gpu10 GPU4, CUDA_VISIBLE_DEVICES=4

## 文件 / Files

- result dir（结果目录）: `data/3krice/processed/prsnet2_rawp005_sixtrait_top4096_h64_k16_b8_gpu4_seed42_fix1_20260618/`
- manifest（训练清单）: `data/3krice/processed/prsnet2_rawp005_sixtrait_top4096_h64_k16_b8_gpu4_seed42_fix1_20260618/training_manifest.json`
- metrics（逐epoch指标）: `data/3krice/processed/prsnet2_rawp005_sixtrait_top4096_h64_k16_b8_gpu4_seed42_fix1_20260618/training_metrics.tsv`
- best checkpoint（最佳模型检查点）: `data/3krice/processed/prsnet2_rawp005_sixtrait_top4096_h64_k16_b8_gpu4_seed42_fix1_20260618/checkpoint_best.pt`

## 一句话结论 / One-sentence conclusion

PRSNet-2（多基因风险评分网络第二版）已按六性状、无RiceGATE-MoE主线完成首轮训练，test macro-F1=0.4521、mean Pearson/PCC=0.3804；目前还没超过ExtraTrees强基线，需要继续调参或改进PRSNet-2细节。
