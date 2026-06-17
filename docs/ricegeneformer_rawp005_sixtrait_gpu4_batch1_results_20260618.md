# RiceGeneFormer raw p<0.05 six-trait SNP-union GPU4 batch=1 run

Date: 2026-06-18
Host: gpu10
Physical GPU: 4
Run directory: `data/3krice/processed/rice_geneformer_rawp005_sixtrait_union_top4096_h128_snp64_b1_gpu4_guarded_seed42_20260618`

## Safety settings

- Existing GPU4 process was present before launch.
- Initial GPU4 state: 12,997 MiB used / 27,329 MiB free; utilization ~96%.
- Training batch size: 1.
- Start guard: required free VRAM >= 16,000 MiB.
- Runtime guard: kill own training if free VRAM < 8,000 MiB.
- Training completed without triggering the guard.
- After completion GPU4 returned to 12,997 MiB used / 27,329 MiB free.

## Input configuration

- Trait set: `CUDI_CODE_REPRO`, `CUNO_CODE_REPRO`, `LLT_CODE`, `PLT_CODE_POST`, `SDHT_CODE`, `PTH`.
- SNP candidate pool: raw p<0.05 six-trait unique SNP union.
- Raw union SNP count: 330,917.
- Source SNP-to-gene map: `window_10kb`.
- RiceGeneFormer gene tokens used: 4,096.
- Max SNPs per gene: 64.
- Hidden dimension: 128.
- Selection metric: validation macro-F1.
- Early stopping patience: 5.

## Training result

| Metric | Value |
|---|---:|
| train status | ok |
| epochs completed | 6 |
| best epoch | 1 |
| best validation macro-F1 | 0.2691857298 |
| best validation accuracy | 0.7090909091 |
| best validation MAE | 0.2909090909 |

## Test result

| Metric | Value |
|---|---:|
| test macro-F1 | 0.2529897356 |
| test accuracy | 0.6412815126 |
| test MAE | 0.3881302521 |
| observed test labels | 1,904 |

## Output files

- Training manifest: `data/3krice/processed/rice_geneformer_rawp005_sixtrait_union_top4096_h128_snp64_b1_gpu4_guarded_seed42_20260618/training_manifest.json`
- Test evaluation manifest: `data/3krice/processed/rice_geneformer_rawp005_sixtrait_union_top4096_h128_snp64_b1_gpu4_guarded_seed42_20260618/eval_test/evaluation_manifest.json`
- Guard log: `data/3krice/processed/rice_geneformer_rawp005_sixtrait_union_top4096_h128_snp64_b1_gpu4_guarded_seed42_20260618/gpu_guard.log`
- Run log: `data/3krice/processed/rice_geneformer_rawp005_sixtrait_union_top4096_h128_snp64_b1_gpu4_guarded_seed42_20260618/run.log`

## Interpretation

This GPU4 batch=1 guarded run is a safe opportunistic run under shared-GPU constraints. It verifies the six-trait raw-p SNP-union-to-gene RiceGeneFormer path on GPU4 without crowding the existing GPU4 process, but its test macro-F1 remains below the current six-trait RiceGATE-MoE / ExtraTrees benchmark level.
