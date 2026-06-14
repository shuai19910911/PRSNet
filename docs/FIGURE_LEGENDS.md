# Draft figure legends for RiceGeneFormer manuscript

Status: draft legends for current editable SVG/PDF/TIFF figure set generated under `docs/figures/`. These legends are written for manuscript drafting and should be shortened after the final target journal is chosen.

## Figure 1. Leakage-aware construction of an ordinal 3K rice genotype-to-phenotype benchmark

(a) Workflow used to construct the RiceGeneFormer benchmark from 3K Rice Genome resources. Genotype data were processed into a 3,000-accession by 365,710-SNP matrix, aligned to descriptor-code phenotypes, split into train, validation and test roles, and then used to compute train-fold GWAS priors before any model fitting. The train/validation/test split contained 1,586/340/340 supervised accessions; accessions without selected phenotype labels were retained only where appropriate for matrix alignment and artifact checking. (b) Class imbalance across the ten core ordinal descriptor traits in the training role, summarized as the largest-class fraction for each trait. Several traits were dominated by a single class, motivating macro-F1 reporting and class-balanced training experiments. (c) Main validated benchmark artifacts used by the modeling pipeline, including SNP count, core trait count, train-fold GWAS p-value sets, mapped gene count and graph edge count.

Source data: `docs/figure_source_data/fig1a_workflow_artifacts.csv`, `fig1b_trait_imbalance.csv`, and `fig1c_artifact_counts.csv`.

## Figure 2. RiceGeneFormer-OMTL model design and component priorities

(a) RiceGeneFormer-OMTL architecture. SNP dosages are combined with train-fold GWAS priors, aggregated into gene-level tokens through SNP-to-gene bags, processed with a lightweight graph encoder, decoded by trait-specific queries and evaluated with ordinal phenotype heads. A separate train-fold top-SNP branch is fused into trait tokens through a gated fusion module. (b) Qualitative summary of design choices that were prioritized during pilot development. Direct top-SNP signal, gated fusion and mild class balancing gave the clearest practical improvements, whereas simple expected-score distillation had only weak incremental value. (c) Pilot comparison of graph choices. Chromosome-neighbor, edge-matched random, STRING and chromosome+STRING fusion graphs produced similar bounded-pilot macro-F1 values, indicating no stable graph-identity advantage for the current lightweight mean-neighbor encoder. (d) Final RiceGeneFormer family frozen for reporting: gated top-SNP fusion, alpha0.25 class-balanced ordinal training, validation macro-F1 checkpoint selection and optional weak expected-score distillation.

Source data: `docs/figure_source_data/fig2b_design_choice_utility.csv` and `fig2c_graph_pilot_macro_f1.csv`.

Review note: panel b is intentionally qualitative and should either be moved to schematic-only form or backed by a numerical ablation table before final submission.

## Figure 3. Final predictive performance and trait-level failure modes

(a-c) Deterministic test-role comparison among final RiceGeneFormer variants and top-SNP baselines. RiceGeneFormer without distillation reached test macro-F1 0.3229±0.0062, while weak expected-score distillation with weight 0.10 reached 0.3292±0.0057. These results were close to the LightGBM top-SNP baseline (macro-F1 0.3380) but remained below balanced SNP-MLP baselines, which reached 0.3644±0.0106 at alpha0.40 and 0.3781±0.0029 at alpha0.60. Accuracy and MAE showed the expected trade-off: tree and top-SNP baselines retained higher accuracy or lower ordinal error, whereas stronger class balancing improved macro-F1 at some cost to accuracy and MAE. (d) Per-trait deterministic test performance for the best RiceGeneFormer variant (expected-score distillation weight 0.10). Binary or low-cardinality traits such as CUDI_CODE_REPRO and PTH had higher macro-F1, whereas strongly imbalanced multi-class traits such as CUST_REPRO and SPKF remained difficult. (e) Relationship between training-set class imbalance and final RiceGeneFormer macro-F1, illustrating that several low-macro-F1 traits had large dominant-class fractions.

Source data: `docs/figure_source_data/fig3abc_main_performance.csv`, `fig3de_per_trait_performance.csv`, `docs/FINAL_PER_TRAIT_METRICS_RGF_W010_TEST.tsv`, and `docs/CORE_TRAIT_CLASS_DISTRIBUTION.tsv`.

Definitions: macro-F1 is the unweighted mean of per-class F1 scores; MAE is mean absolute error in ordinal class units; alpha denotes the interpolation strength between unweighted and inverse-frequency class-balanced loss.

## Figure 4. Ablation boundaries, robustness checks and interpretation diagnostics

(a) Expected-score distillation from the SNP-MLP teacher produced only weak and non-monotonic gains. Distillation weight 0.10 gave the highest RiceGeneFormer test macro-F1 among the tested weights, whereas increasing the weight to 0.20 did not improve the final mean. (b) Graph-identity pilots showed similar macro-F1 among chromosome-neighbor, edge-matched random, STRING and chromosome+STRING fusion graphs. This result supports the conservative conclusion that the current lightweight graph encoder did not demonstrate a graph-specific performance advantage. (c) SNP-to-gene mapping ablations showed similar bounded-pilot macro-F1 across body-only, ±2 kb, ±5 kb, ±10 kb and nearest-gene mappings, suggesting that mapping-window choice was not the main bottleneck under the current configuration. (d) Held-out geographic-region tests showed that RiceGeneFormer did not outperform the strongest balanced SNP-MLP baseline in Southeast Asia, South Asia or East Asia, so the current model does not support a cross-region robustness claim. (e) Gene-attention audit showed that the original bounded 2,048-gene runs used an early chromosome-1 prefix, excluding many canonical rice genes from the attention layer. A train-fold GWAS-top2048 gene graph corrected the prefix design but reduced seed42 test macro-F1 to 0.2990 and did not recover the preliminary known-gene panel in top attention ranks.

Source data: `docs/figure_source_data/fig4a_distillation_macro_f1.csv`, `fig4b_graph_ablation.csv`, `fig4c_mapping_ablation.csv`, `data/3krice/processed/region_shift_baseline_summary_region_manual_20260614_213808.tsv`, `data/3krice/processed/region_shift_rice_geneformer_summary_region_rgf_manual_20260614_214252.tsv`, `data/3krice/processed/gene_attention_w010_test_aggregate/gene_attention_top200_aggregate.tsv`, and `data/3krice/processed/rice_geneformer_gwas_top2048_seed42_e20_s100_gpu2/test_eval/evaluation_manifest.json`.

Interpretation boundary: the graph, mapping, region-shift and gene-attention results are bounded diagnostics. They do not show that biological graphs, SNP-to-gene mapping or candidate-gene interpretation are generally unimportant; they show that these particular implementations did not yield robust predictive or biological-discovery evidence in the present 3K Rice benchmark.

## Supplementary Figure 1. Per-trait method comparison on the deterministic test role

(a) Heatmap of test macro-F1 for each core ordinal trait across RiceGeneFormer with expected-score distillation weight 0.10, LightGBM top512, XGBoost top512, SNP-MLP alpha0.40 and SNP-MLP alpha0.60. Stars mark the best macro-F1 method within each trait. (b) Number of traits for which each method was best by macro-F1. (c) Traits with the largest macro-F1 gap between the best method and RiceGeneFormer w=0.10, highlighting where the gene-aware model remains furthest from the strongest top-SNP baseline.

Source data: `docs/figure_source_data/figs1_per_trait_method_comparison.csv`, `figs1_best_method_counts.csv`, and `docs/SUPPLEMENTARY_PER_TRAIT_RESULTS.tsv`.

Interpretation boundary: this supplementary figure compares predictive performance only. It does not evaluate interpretability, biological mechanism recovery or external generalization beyond the current random-split test role.
