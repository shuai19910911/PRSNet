# GWAS corrected positive loci counts by trait

Split: `core_ordinal/random_seed42`; GWAS p-values were generated from the train fold only. Each trait tested 365,710 SNPs. Corrected positive loci are counted at corrected P/q < 0.05.

Definitions: Bonferroni positive = raw p <= 0.05 / n_tests. BH-FDR positive = Benjamini-Hochberg q <= 0.05. Raw p<0.05 is shown only as an uncorrected reference.

| Trait | n classes | non-missing | Bonferroni positives | BH-FDR positives | raw p<0.05 | min raw p | min Bonferroni p | min BH-FDR q |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| CUDI_CODE_REPRO | 2 | 2111 | 20061 | 146299 | 178111 | 4.851e-32 | 1.774e-26 | 1.774e-26 |
| CULT_CODE_REPRO | 7 | 2094 | 83711 | 220589 | 234159 | 1.175e-38 | 4.299e-33 | 1.171e-36 |
| CUNO_CODE_REPRO | 3 | 2095 | 38541 | 159337 | 182722 | 7.706e-35 | 2.818e-29 | 2.818e-29 |
| LLT_CODE | 5 | 2095 | 51650 | 170633 | 192758 | 1.175e-38 | 4.299e-33 | 2.590e-35 |
| PLT_CODE_POST | 4 | 2090 | 34161 | 110893 | 140680 | 1.175e-38 | 4.299e-33 | 7.001e-36 |
| SDHT_CODE | 3 | 2093 | 25812 | 112905 | 144571 | 1.826e-22 | 6.678e-17 | 6.678e-17 |
| PTH | 3 | 2262 | 71495 | 197634 | 216174 | 1.175e-38 | 4.299e-33 | 1.323e-35 |
| SPKF | 5 | 2264 | 71 | 11664 | 69275 | 1.358e-10 | 4.966e-05 | 4.966e-05 |
| CUST_REPRO | 8 | 2264 | 37984 | 190047 | 211415 | 1.964e-32 | 7.182e-27 | 3.887e-27 |
| PSH | 3 | 1526 | 67 | 5420 | 44634 | 1.160e-11 | 4.243e-06 | 4.243e-06 |

## Interpretation note

- Bonferroni is the stricter headline count for GWAS-significant SNPs.
- BH-FDR is less strict and better reflects the number of loci retained under false-discovery-rate control.
- PSH and SPKF have the fewest corrected positives, matching their weak genetic-signal interpretation in the model diagnostics.
- CULT_CODE_REPRO, PTH, LLT_CODE, CUST_REPRO and CUNO_CODE_REPRO have many corrected positives, but many positives can also reflect population structure or broad correlated SNP blocks, so these counts should be interpreted with leakage-aware train-fold GWAS only.
