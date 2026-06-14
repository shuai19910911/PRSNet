# Region-shift benchmark results for RiceGeneFormer BIB/NC redesign

Last updated: 2026-06-14

## Purpose

This document records the first deterministic cross-region benchmark requested for the BIB/NC redesign. The goal is to test whether RiceGeneFormer is more robust than top-SNP baselines when the test set is a held-out geographic region rather than a random split.

Terms: macro-F1 is the unweighted mean of F1 across ordinal classes; MAE is mean absolute error in ordinal class units; lower MAE is better. Distribution shift means the test samples come from a different region than most training samples.

## Inputs and provenance

- Baseline summary: `/home/user/zhangzhishuai/myhermes/PRSNet/data/3krice/processed/region_shift_baseline_summary_region_manual_20260614_213808.tsv`
- RiceGeneFormer summary: `/home/user/zhangzhishuai/myhermes/PRSNet/data/3krice/processed/region_shift_rice_geneformer_summary_region_rgf_manual_20260614_214252.tsv`
- Splits: `region_leaveout_Southeast_Asia`, `region_leaveout_South_Asia`, `region_leaveout_East_Asia`.
- Baselines: LightGBM top512 seed42, XGBoost top512 seed42, SNP-MLP top512 alpha0.40/0.60 seeds42/43/44.
- RiceGeneFormer: gated top-SNP fusion, alpha0.25 balanced ordinal loss, val_macro_f1 checkpoint selection, seeds42/43/44, chr_neighbor_k5 graph, window_5kb SNP-to-gene map, 2048 genes, 20 epochs max, early-stop patience 3.
- GPU execution: `gpu10`, `CUDA_VISIBLE_DEVICES=2`, torch 2.6.0+cu124.

## Main table

| Held-out region | Method | Seeds | macro-F1 | Accuracy | MAE | Spearman |
|---|---|---:|---:|---:|---:|---:|
| Southeast_Asia | RiceGeneFormer gated alpha0.25 | 42,43,44 | 0.2438 ± 0.0106 | 0.5626 ± 0.0065 | 0.6417 ± 0.0010 | 0.0946 ± 0.0055 |
| Southeast_Asia | LightGBM | 42 | 0.2628 | 0.6130 | 0.6319 | n/a |
| Southeast_Asia | XGBoost | 42 | 0.2499 | 0.6142 | 0.6144 | n/a |
| Southeast_Asia | SNP-MLP alpha0.40 | 42,43,44 | 0.2835 ± 0.0073 | 0.6115 ± 0.0129 | 0.6210 ± 0.0227 | n/a |
| Southeast_Asia | SNP-MLP alpha0.60 | 42,43,44 | 0.2989 ± 0.0156 | 0.5893 ± 0.0055 | 0.6568 ± 0.0195 | n/a |
| South_Asia | RiceGeneFormer gated alpha0.25 | 42,43,44 | 0.2327 ± 0.0064 | 0.5881 ± 0.0031 | 0.6124 ± 0.0045 | 0.0589 ± 0.0121 |
| South_Asia | LightGBM | 42 | 0.2563 | 0.6064 | 0.6383 | n/a |
| South_Asia | XGBoost | 42 | 0.2421 | 0.6120 | 0.6246 | n/a |
| South_Asia | SNP-MLP alpha0.40 | 42,43,44 | 0.2687 ± 0.0067 | 0.5924 ± 0.0110 | 0.6356 ± 0.0125 | n/a |
| South_Asia | SNP-MLP alpha0.60 | 42,43,44 | 0.2833 ± 0.0032 | 0.5796 ± 0.0174 | 0.6512 ± 0.0190 | n/a |
| East_Asia | RiceGeneFormer gated alpha0.25 | 42,43,44 | 0.2693 ± 0.0141 | 0.5324 ± 0.0194 | 0.6083 ± 0.0275 | 0.2494 ± 0.0119 |
| East_Asia | LightGBM | 42 | 0.2900 | 0.5834 | 0.5530 | n/a |
| East_Asia | XGBoost | 42 | 0.2827 | 0.5786 | 0.5509 | n/a |
| East_Asia | SNP-MLP alpha0.40 | 42,43,44 | 0.3037 ± 0.0063 | 0.5858 ± 0.0029 | 0.5577 ± 0.0089 | n/a |
| East_Asia | SNP-MLP alpha0.60 | 42,43,44 | 0.3042 ± 0.0061 | 0.5753 ± 0.0153 | 0.5870 ± 0.0397 | n/a |

## Interpretation

- Southeast_Asia: best macro-F1 is SNP-MLP alpha0.60 at 0.2989; RiceGeneFormer is 0.2438, gap -0.0551.
- South_Asia: best macro-F1 is SNP-MLP alpha0.60 at 0.2833; RiceGeneFormer is 0.2327, gap -0.0506.
- East_Asia: best macro-F1 is SNP-MLP alpha0.60 at 0.3042; RiceGeneFormer is 0.2693, gap -0.0349.

Across all three held-out regions, RiceGeneFormer did not beat the strongest SNP-MLP baseline on macro-F1. It also did not show a clear distribution-shift robustness advantage in this first region-shift run. Therefore, the NC route based on cross-region robustness is not supported by the current results.

However, this result is still useful for the BIB story: it strengthens the leakage-aware benchmark narrative by showing that strong top-SNP baselines remain difficult to beat even under geographic holdout, and it provides a realistic boundary for gene-aware neural G2P methods.

## Journal decision after this experiment

- BIB: still viable if framed as a strict benchmark/diagnostic framework with honest negative results and reproducibility.
- NC: not recommended from cross-region robustness alone. NC would now require strong biological interpretation evidence, such as known rice gene or QTL overlap from RiceGeneFormer gene scores.

## Next execution target

Proceed to gene-level interpretation: export trait-specific RiceGeneFormer gene scores, aggregate across seeds, and test overlap with known rice trait genes or QTLs. This is now the most important remaining experiment for deciding whether any NC route remains plausible.
