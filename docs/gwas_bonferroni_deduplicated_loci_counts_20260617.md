# Bonferroni GWAS positive SNP redundancy-reduced counts

Split: `core_ordinal/random_seed42`; p-values are train-fold GWAS p-values. Bonferroni threshold = 0.05 / 365,710 = 1.3672e-07.

Redundancy definitions:
- `unique SNP union`: exact SNP index de-duplication across traits; identical SNP counted once.
- `unique chr:position`: exact physical coordinate de-duplication.
- `clumped 10/50/100 kb`: greedy physical-window de-redundancy; within the same chromosome and window, only the smallest-p SNP is kept as the representative locus.

## Per-trait counts

| Trait | Bonferroni SNPs | unique chr:position | clumped 10kb | clumped 50kb | clumped 100kb |
|---|---:|---:|---:|---:|---:|
| CUDI_CODE_REPRO | 20061 | 20061 | 10113 | 3864 | 2265 |
| CULT_CODE_REPRO | 83711 | 83711 | 18652 | 4974 | 2656 |
| CUNO_CODE_REPRO | 38541 | 38541 | 14696 | 4484 | 2482 |
| LLT_CODE | 51650 | 51650 | 16264 | 4720 | 2555 |
| PLT_CODE_POST | 34161 | 34161 | 13587 | 4462 | 2462 |
| SDHT_CODE | 25812 | 25812 | 12230 | 4236 | 2383 |
| PTH | 71495 | 71495 | 18236 | 4906 | 2608 |
| SPKF | 71 | 71 | 63 | 51 | 44 |
| CUST_REPRO | 37984 | 37984 | 14306 | 4466 | 2457 |
| PSH | 67 | 67 | 61 | 50 | 48 |

## Cross-trait union counts

| Trait set | Sum of per-trait Bonferroni SNPs | unique SNP union | unique chr:position | clumped 10kb | clumped 50kb | clumped 100kb |
|---|---:|---:|---:|---:|---:|---:|
| all10_union | 363553 | 128068 | 128068 | 20007 | 5083 | 2686 |
| sixtrait_union_after_dropping_SPKF_PSH_CUST_REPRO_CULT_CODE_REPRO | 241720 | 107002 | 107002 | 19544 | 5013 | 2649 |

Recommended manuscript number: use the 100kb clumped count as a conservative physical-locus proxy, and keep the exact unique SNP union count in the source-data table.
