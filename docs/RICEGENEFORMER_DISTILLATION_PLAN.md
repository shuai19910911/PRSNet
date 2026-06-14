# RiceGeneFormer 强 SNP Baseline 对齐蒸馏计划

最后更新：2026-06-14 15:30:00 CST

## 目标

当前 RiceGeneFormer full-step gated alpha0.25 主线的 deterministic full-val macro-F1 约 0.317–0.327，sampled-val best 可到 0.339±0.009；SNP-MLP balanced alpha0.4–0.6 多 seed macro-F1 约 0.350–0.354。下一阶段目标不是继续微调 graph/mapping，而是把强 SNP baseline 的可学习信号安全地对齐到 RiceGeneFormer。

## 约束

1. 不使用 val/test 标签训练蒸馏目标。
2. teacher 必须只用 train-fold GWAS top SNP 与 train labels 训练。
3. 蒸馏产物、预测、checkpoint、日志只保存在本地 `data/3krice/processed/`，不上传 GitHub。
4. GitHub 只同步轻量 docs。
5. 所有新代码必须通过 `py_compile`、smoke、静态扫描、独立代码复审。
6. 正式报告必须使用 deterministic full-val 指标；sampled validation 只作为训练过程监控。

## 当前证据

- SNP-MLP alpha0.40：macro-F1 `0.3502±0.0117`，accuracy `0.6141±0.0097`，MAE `0.5697±0.0108`。
- SNP-MLP alpha0.60：macro-F1 `0.3544±0.0112`，accuracy `0.5961±0.0086`，MAE `0.5976±0.0087`。
- RiceGeneFormer full-step sampled-val：macro-F1 `0.3388±0.0093`。
- RiceGeneFormer full-step deterministic full-val：macro-F1 约 `0.317–0.327`。
- Post-hoc threshold calibration 在 full-step checkpoint 上总体负向，不作为主线。

## 推荐路线

### Phase A：先补 deterministic evaluator

目的：统一神经模型报告口径，避免 sampled validation 与 full validation 混用。

产出：
- 本地脚本：`scripts/model/evaluate_rice_geneformer_checkpoint.py`
- 输入：processed_dir、checkpoint、role、batch_size、device
- 输出：`evaluation_manifest.json`，包含 loss、accuracy、MAE、macro-F1、Spearman、样本数、trait 列表、checkpoint config 摘要

验证：
- `py_compile`
- CPU smoke：评估 seed42 full-step checkpoint 的 val role
- `json.tool`
- 独立代码复审

### Phase B：导出 SNP-MLP teacher 概率

目的：为蒸馏提供 train/val/test 的 teacher soft labels。

推荐做法：
- 在 `baseline_snp_mlp_core_smoke.py` 增加可选 `--export-predictions` 或新建本地脚本加载 SNP-MLP checkpoint。
- 输出每个 role 的 per-sample、per-trait：
  - sample_index
  - trait_name
  - observed mask
  - true_label
  - teacher_prob_vector
  - teacher_expected_score
  - teacher_pred

安全要求：
- teacher checkpoint 与 top-SNP selection 均来自 train-fold。
- 输出用 JSONL/NPZ/TSV，不用 pickle。
- manifest 记录 teacher alpha、seed、top_snp_count、split。

### Phase C：RiceGeneFormer 蒸馏训练

在 `rice_geneformer_omtl_train.py` 增加可选蒸馏项：

- `--distill-manifest PATH`
- `--distill-weight FLOAT`
- `--distill-temperature FLOAT`
- 默认 `distill_weight=0`，保持历史行为。

损失建议：

`loss = ordinal_loss + distill_weight * KL(student_probs, teacher_probs)`

其中 student 需要从 ordinal cutpoint logits 转换为 ordinal class probabilities。若实现复杂，第一版可先蒸馏 expected ordinal score：

`MSE(student_expected_score, teacher_expected_score)`

优先使用 expected-score MSE，因为实现简单、稳定、适合先做 smoke。

### Phase D：最小实验矩阵

先只做 seed42 bounded pilot：

1. distill_weight=0.05
2. distill_weight=0.10
3. distill_weight=0.20

固定：
- gated top-SNP fusion
- balance-alpha=0.25
- selection_metric=val_macro_f1
- full-step 20 epoch / 100 steps
- deterministic full-val evaluator 做最终指标

如果 seed42 任一配置 deterministic full-val macro-F1 ≥ 0.335，再补 seed43/44。

## 停止条件

1. 若 distillation 提高 macro-F1 但 accuracy/MAE 大幅恶化，不能作为整体提升，只能作为少数类消融。
2. 若 seed42 无法超过 deterministic full-val 0.327，暂停蒸馏，改做模型报告和负结果解释。
3. 若 seed42 超过 LightGBM 0.335，再补 seed43/44；若三 seed 均值接近或超过 SNP-MLP alpha0.4，才作为主线候选。
