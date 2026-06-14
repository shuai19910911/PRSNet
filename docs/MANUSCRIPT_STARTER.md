# RiceGeneFormer manuscript starter package

Last updated: 2026-06-14 21:16:03 CST

## 1. Detected writing axes

- Paper type: research article.
- Sections prepared here: title candidates, one-sentence argument, Results/Experiments scaffold, Methods scaffold, Discussion boundaries, data/code availability notes.
- Language mode: Chinese lab notes to English manuscript later; this document is written in Chinese for project control.
- Journal mode: generic computational genomics / plant genomics journal; can later be adapted to Briefings in Bioinformatics, Plant Biotechnology Journal, Horticulture Research, or Nature Communications style.

## 2. One-sentence paper argument

In plant genotype-to-phenotype prediction with sparse ordinal descriptor phenotypes, we show that a gene-aware ordinal multi-task architecture can integrate train-fold GWAS priors, SNP-to-gene aggregation, graph-aware gene tokens, and trait-specific decoding on the 3K Rice Genome dataset, achieving performance close to tree-based top-SNP baselines while revealing that simple biological graph smoothing and expected-score teacher distillation provide limited incremental gains under the current sample size.

Boundary: this claim is intentionally bounded to internal random-split 3K Rice validation/test experiments. It does not claim superiority over strong top-SNP SNP-MLP baselines and does not yet demonstrate cross-subpopulation or cross-environment generalization.

## 3. Terminology ledger

- RiceGeneFormer-OMTL: Rice Gene-aware Ordinal Multi-Task Transformer; the main neural model family.
- OMTL: ordinal multi-task learning; each trait is modeled with ordered class thresholds where applicable.
- 3K Rice Genome: rice genotype dataset used here; processed matrix has 3,000 accessions and 365,710 SNPs.
- Core ordinal traits: 10 high-coverage rice descriptor-code traits selected for the current main experiments.
- Train-fold GWAS priors: per-trait p-values computed only on training samples of the active split to prevent leakage.
- SNP-to-gene mapping: assignment of SNPs to rice genes using gene body plus windows or nearest-gene rules.
- GraphEncoder: lightweight mean-neighbor gene graph encoder used for chromosome-neighbor, random, STRING, and fusion graphs.
- top-SNP branch: dense side branch using train-fold GWAS top SNPs; default final setting uses top512.
- gated top-SNP fusion: learned gate that injects top-SNP information into trait tokens.
- macro-F1: unweighted mean of per-class F1 scores; emphasizes minority classes.
- accuracy: overall correct-class rate; often favors majority classes in imbalanced descriptor traits.
- MAE: mean absolute error over ordinal class labels; lower is better.
- Spearman: rank correlation between predicted expected ordinal score and observed label.
- expected-score distillation: teacher-student loss matching the student expected ordinal score to SNP-MLP teacher expected scores.

## 4. Current paper-ready claims and evidence

### Claim 1 — A leakage-aware plant genotype-to-phenotype benchmark was built from 3K Rice.

Evidence:

- Genotype matrix: 3,000 accessions × 365,710 SNPs.
- Phenotype/mask matrix: 3,000 × 35 traits.
- Main random split: train/val/test = 1,586/340/340 supervised samples; 734 unused for traits without selected labels.
- Main trait set: 10 core ordinal descriptor traits.
- Train-fold GWAS p-values were generated for 10 traits × 4 splits, each p-value vector length 365,710.
- SNP-to-gene mapping and graphs were generated locally and validated by shape, finite-value, and path constraints.

Writing status: ready for Methods and Dataset subsection.

### Claim 2 — The model training pipeline is complete and reproducible.

Evidence:

- RiceGeneFormer training, checkpointing, deterministic full-role evaluation, threshold calibration, teacher export, and distillation scripts all passed py_compile, smoke tests, JSON manifest validation, static scans, and independent review.
- Deterministic evaluator reports full val/test metrics over all role samples rather than sampled validation batches.
- All model-selection metrics are manifest-backed: best epoch, best selection score, best_val_* and final_val_* are separately recorded.

Writing status: ready for Methods, Reproducibility, and Supplementary Methods.

### Claim 3 — The final RiceGeneFormer family approaches tree baselines but remains below strong top-SNP SNP-MLP baselines.

Main deterministic test comparison:

| Model / setting | Split | macro-F1 | Accuracy | MAE | Spearman | Notes |
|---|---:|---:|---:|---:|---:|---|
| RiceGeneFormer full-step, no distillation | test | 0.3229 ± 0.0062 | 0.5895 ± 0.0029 | 0.5978 ± 0.0039 | 0.2556 ± 0.0053 | gated top-SNP fusion, alpha0.25, macro-F1 selection |
| RiceGeneFormer + expected-score distillation w=0.10 | test | 0.3292 ± 0.0057 | 0.5894 ± 0.0049 | 0.5948 ± 0.0067 | 0.2701 ± 0.0057 | best distillation variant by test macro-F1 |
| RiceGeneFormer + expected-score distillation w=0.20 | test | 0.3251 ± 0.0054 | 0.5906 ± 0.0033 | 0.5904 ± 0.0037 | 0.2757 ± 0.0089 | stronger weight, not better than w=0.10 |
| LightGBM top512 | test | 0.3380 | 0.6263 | 0.5630 | n/a | single baseline run |
| XGBoost top512 | test | 0.3161 | 0.6332 | 0.5512 | n/a | single baseline run |
| SNP-MLP top512, alpha0.40 | test | 0.3644 ± 0.0106 | 0.6317 ± 0.0023 | 0.5639 ± 0.0117 | n/a | strongest balanced accuracy/macro-F1 tradeoff |
| SNP-MLP top512, alpha0.60 | test | 0.3781 ± 0.0029 | 0.6148 ± 0.0091 | 0.5899 ± 0.0176 | n/a | highest macro-F1 baseline |

Important wording boundary:

- Do not write that RiceGeneFormer outperforms all baselines.
- Correct claim: RiceGeneFormer approaches LightGBM macro-F1 and provides a gene-aware multi-task framework, but strong top-SNP SNP-MLP remains the best predictive baseline under the present random-split test evaluation.

Writing status: ready for a balanced Results subsection.

### Claim 4 — Model improvements were driven more by top-SNP fusion and class-balanced selection than by graph identity.

Evidence summary:

- top-SNP side branch improved best validation loss and macro-F1 versus early minimal models.
- gated top-SNP fusion improved validation loss/accuracy/MAE versus additive fusion.
- mild balanced ordinal loss with alpha0.25 plus macro-F1 checkpoint selection raised RiceGeneFormer pilot macro-F1 into the 0.299–0.310 range before full-step training.
- full-step training raised deterministic test macro-F1 to approximately 0.323–0.329 depending on distillation.
- chr-neighbor, prefix-random, STRING, and chr+STRING fusion graphs were very close across seeds; no graph type showed stable structure-specific advantage.
- SNP-to-gene mapping variants body-only, ±2 kb, ±5 kb, ±10 kb, nearest-gene were close in bounded pilots.

Writing boundary:

- Current graph results support feasibility of graph integration, not a claim that chromosome-neighbor or STRING graph biology improves prediction.
- Current mapping results support robustness to mapping choices, not a new biological conclusion.

Writing status: ready for Ablation/Diagnostics subsection.

### Claim 5 — Simple expected-score distillation is not the main breakthrough.

Evidence:

- w=0.10 test macro-F1: 0.3292 ± 0.0057, a small increase over no-distillation 0.3229 ± 0.0062.
- w=0.20 test macro-F1: 0.3251 ± 0.0054, lower than w=0.10.
- Full-val gains were similarly small: no distillation 0.3232 ± 0.0054, w=0.10 0.3249 ± 0.0023, w=0.20 0.3262 ± 0.0019.

Writing boundary:

- Distillation should be presented as an exploratory negative/weak-positive result.
- Future work should mention probability-distribution KL distillation or structured teacher fusion instead of increasing expected-score MSE weight.

Writing status: ready for Discussion limitation/future-work paragraph.

## 5. Recommended manuscript structure

### Title candidates

1. RiceGeneFormer: leakage-aware gene-level ordinal multi-task learning for rice genotype-to-phenotype prediction
2. Gene-aware ordinal multi-task learning benchmarks genotype-to-phenotype prediction in 3K rice genomes
3. Integrating GWAS priors and gene-aware transformers for ordinal phenotype prediction in rice
4. A leakage-aware 3K rice benchmark for gene-aware genotype-to-phenotype modeling

Preferred current title: "Gene-aware ordinal multi-task learning benchmarks genotype-to-phenotype prediction in 3K rice genomes". It is accurate because the strongest contribution is a rigorous benchmark and model family, not universal superiority.

### Abstract skeleton

1. Context: plant genotype-to-phenotype prediction increasingly needs models that respect gene structure, trait imbalance, and ordinal descriptor phenotypes.
2. Gap: many pipelines either use top markers directly or treat descriptor-code phenotypes as ordinary classification/regression targets, while leakage-safe gene-aware deep models remain difficult to compare fairly.
3. Approach: introduce RiceGeneFormer-OMTL, using train-fold GWAS priors, SNP-to-gene aggregation, graph-aware gene tokens, gated top-SNP fusion, and ordinal multi-task losses.
4. Evidence: on 3K Rice, deterministic test macro-F1 reached 0.3229 without distillation and 0.3292 with weak expected-score distillation; LightGBM reached 0.3380 test macro-F1 and top-SNP SNP-MLP reached 0.3644–0.3781 test macro-F1.
5. Interpretation: top-SNP fusion and mild class balancing mattered most; graph identity, mapping variants, threshold calibration, and simple expected-score distillation gave limited or unstable gains.
6. Boundary/impact: the study provides a reproducible plant G2P benchmark and a calibrated assessment of where gene-aware neural models currently help and where top-SNP baselines still dominate.

### Results section ladder

1. Build a leakage-aware 3K Rice ordinal G2P benchmark.
2. Introduce RiceGeneFormer-OMTL and validate training/evaluation integrity.
3. Establish classical and neural top-SNP baselines.
4. Improve RiceGeneFormer through trait attention, top-SNP branch, gated fusion, mild class balancing, and full-step training.
5. Diagnose graph, SNP-to-gene mapping, threshold calibration, and distillation contributions.
6. Freeze final model family and compare against baselines under deterministic full-role evaluation.

### Methods section scaffold

1. Dataset processing and phenotype selection.
2. Random split and leakage-control design.
3. Train-fold GWAS prior generation.
4. SNP-to-gene mapping and graph construction.
5. RiceGeneFormer-OMTL architecture.
6. Ordinal multi-task loss and class balancing.
7. top-SNP branch and gated fusion.
8. Baseline models: LightGBM, XGBoost, SNP-MLP.
9. Deterministic full-role evaluation and metrics.
10. Distillation and calibration analyses.
11. Compute environment and reproducibility.

### Discussion scaffold

1. Main finding: rigorous gene-aware neural modeling can approach tree baselines but does not yet beat strong top-SNP MLP baselines.
2. Why top-SNP baselines remain strong: small supervised sample size, high-dimensional marker shortcut signal, imbalanced descriptor traits.
3. What RiceGeneFormer adds: leakage-aware gene-level modeling, ordinal multi-task formulation, explicit ablation framework, and interpretable architecture hooks.
4. Negative/neutral findings: current lightweight GraphEncoder identity, SNP-to-gene mapping variants, threshold calibration, and expected-score distillation are not robust breakthroughs.
5. Future work: cross-subpopulation/region splits, pathway/co-expression graphs with relation-aware encoders, KL/logit distillation, larger phenotype panels, multi-environment crop transfer.

## 6. Figure and table plan

### Main Figure 1 — Dataset and leakage-safe workflow

Panels:

- 3K Rice genotype/phenotype alignment.
- Train/val/test split.
- Train-fold GWAS prior generation.
- SNP-to-gene mapping and graph construction.
- Deterministic evaluation.

### Main Figure 2 — RiceGeneFormer architecture

Panels:

- SNP dosage input.
- GWAS-aware gene bag aggregation.
- GraphEncoder.
- trait-query decoder.
- top-SNP gated fusion.
- ordinal multi-task heads.

### Main Figure 3 — Main performance comparison

Panel/table:

- RiceGeneFormer no distillation, w=0.10, w=0.20.
- LightGBM, XGBoost, SNP-MLP alpha0.4/0.6.
- macro-F1, accuracy, MAE.

### Main Figure 4 — Ablation summary

Panels:

- top-SNP branch / gated fusion / class-balanced alpha effects.
- graph identity comparison.
- SNP-to-gene mapping comparison.
- distillation weight comparison.

### Supplementary Tables

- Trait list and class distributions.
- GWAS p-value artifact validation.
- Mapping statistics.
- Graph statistics.
- Per-trait metrics for final models.
- All seed-level model metrics.

## 7. Data and code availability draft notes

Do not invent repository DOIs yet. Current draft-ready statement should be a placeholder:

- Public source data: 3K Rice genotype and phenotype source files should be cited by their official repositories/URLs in the manuscript.
- Processed data: processed genotype matrices, phenotype masks, GWAS p-values, SNP-to-gene mappings, graph files, checkpoints, and predictions are currently local and not suitable for GitHub because of size.
- Code: lightweight scripts and documentation are in the PRSNet GitHub repository; release state must be finalized before submission.
- Model weights/results: local only for now; if the journal requires reproducibility artifacts, deposit scripts/manifests plus selected non-sensitive processed artifacts to Zenodo/OSF/Figshare or an institutional repository.

Unresolved before submission:

1. Confirm which raw 3K Rice genotype and phenotype accessions/URLs should be cited.
2. Decide whether processed matrices can be redistributed or must be regenerated from raw public sources.
3. Decide code-release repository and archival DOI.
4. Decide whether model checkpoints/predictions will be deposited or regenerated by scripts.

## 8. What is still needed before drafting full prose

Writing can start now for Results/Methods scaffold, but these items are needed before final manuscript submission:

1. Per-trait final metrics table for RiceGeneFormer and SNP-MLP.
2. Trait class distribution table.
3. Raw data citation/accession verification.
4. Figure-ready architecture diagram.
5. Decide target journal and word/figure limits.
6. Decide whether to include cross-region split results now or explicitly defer to future work.

## 9. Recommended writing decision

Start the article now as a benchmark-and-methods paper, not as a "deep model beats all baselines" paper.

Core framing:

- Primary contribution: leakage-aware plant G2P benchmark + gene-aware ordinal multi-task modeling framework.
- Main positive result: RiceGeneFormer approaches tree baselines and supports rigorous ablations.
- Main honest limitation: strong top-SNP SNP-MLP remains superior in predictive macro-F1.
- Strong discussion value: the study identifies what did not work robustly, which is useful for future crop foundation-model work.
