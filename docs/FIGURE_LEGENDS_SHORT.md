# Short figure legends for BIB submission: RiceGeneFormer

Status: compact legends derived from `docs/FIGURE_LEGENDS.md`. These are intended for manuscript insertion after the final figure panel layout is frozen.

## Figure 1. Leakage-aware construction of an ordinal 3K rice genotype-to-phenotype benchmark

(a) Workflow for constructing the benchmark from 3K Rice Genome resources. Genotypes were encoded as a 3,000-accession by 365,710-SNP dosage matrix, aligned to descriptor-code phenotypes and split into train, validation and test roles before train-fold GWAS priors were computed. (b) Class imbalance across the ten core ordinal traits in the training role, summarized by the largest-class fraction. (c) Validated benchmark artifacts used by downstream models, including SNPs, core traits, train-fold GWAS p-value sets, mapped genes and graph edges.

Source data: `docs/figure_source_data/fig1a_workflow_artifacts.csv`, `fig1b_trait_imbalance.csv`, `fig1c_artifact_counts.csv`.

## Figure 2. RiceGeneFormer-OMTL architecture and model-design priorities

(a) RiceGeneFormer-OMTL maps SNP dosage and train-fold GWAS priors into gene tokens, propagates local graph context, decodes traits with trait-specific queries and predicts ordinal descriptor classes. A train-fold top-SNP branch is fused into trait tokens through a gated module. (b) Development summary showing that direct top-SNP signal, gated fusion and mild class balancing were the most useful practical choices. (c) Graph pilots comparing chromosome-neighbor, random, STRING and chromosome+STRING fusion graphs. (d) Final reporting configuration: gated top-SNP fusion, alpha0.25 class-balanced ordinal training, validation macro-F1 checkpoint selection and optional weak expected-score distillation.

Source data: `docs/figure_source_data/fig2b_design_choice_utility.csv`, `fig2c_graph_pilot_macro_f1.csv`.

## Figure 3. Final predictive performance and trait-level failure modes

(a-c) Deterministic test-role performance for final RiceGeneFormer variants and train-fold top-SNP baselines. RiceGeneFormer reached macro-F1 0.3229±0.0062 without distillation and 0.3292±0.0057 with w=0.10 expected-score distillation, approaching LightGBM (0.3380) but remaining below balanced SNP-MLP baselines (0.3644±0.0106 to 0.3781±0.0029). (d) Per-trait performance of the best RiceGeneFormer variant. (e) Relationship between trait imbalance and macro-F1, highlighting strongly imbalanced traits as major failure modes.

Source data: `docs/figure_source_data/fig3abc_main_performance.csv`, `fig3de_per_trait_performance.csv`, `docs/FINAL_PER_TRAIT_METRICS_RGF_W010_TEST.tsv`, `docs/CORE_TRAIT_CLASS_DISTRIBUTION.tsv`.

## Figure 4. Ablation boundaries, robustness checks and interpretation diagnostics

(a) Expected-score distillation gave weak, non-monotonic gains, with w=0.10 outperforming w=0.20. (b) Graph-identity pilots showed no stable advantage for chromosome-neighbor, STRING or fusion graphs over matched controls with the current lightweight encoder. (c) SNP-to-gene mapping windows produced similar bounded-pilot performance. (d) Held-out geographic-region tests showed that RiceGeneFormer did not outperform the strongest balanced SNP-MLP baseline in Southeast Asia, South Asia or East Asia. (e) Gene-attention auditing showed that the original 2,048-gene bounded runs used an early chromosome-1 prefix; a train-fold GWAS-top2048 replacement reduced seed42 test macro-F1 to 0.2990 and did not recover the preliminary known-gene panel in top attention ranks.

Source data: `docs/figure_source_data/fig4a_distillation_macro_f1.csv`, `fig4b_graph_ablation.csv`, `fig4c_mapping_ablation.csv`, `docs/source_data/region_shift_baseline_summary.tsv`, `docs/source_data/region_shift_rice_geneformer_summary.tsv`, `docs/source_data/gene_attention_w010_top200_aggregate.tsv`, `docs/source_data/gwas_top2048_seed42_evaluation_manifest.json`.

Interpretation boundary: panels d and e are diagnostic negative controls. They do not rule out biological graphs, gene-aware modeling or candidate-gene interpretation in general; they show that the current implementation does not support robustness or biological-discovery claims.

## Supplementary Figure 1. Per-trait method comparison on the deterministic test role

(a) Heatmap of test macro-F1 for each core ordinal trait across RiceGeneFormer w=0.10, LightGBM, XGBoost and SNP-MLP alpha0.40/0.60. Stars mark the best method for each trait. (b) Number of traits for which each method was best by macro-F1. (c) Traits with the largest macro-F1 gap between the best method and RiceGeneFormer, showing where the gene-aware model remains furthest from the strongest marker baseline.

Source data: `docs/figure_source_data/figs1_per_trait_method_comparison.csv`, `figs1_best_method_counts.csv`, `docs/SUPPLEMENTARY_PER_TRAIT_RESULTS.tsv`.
