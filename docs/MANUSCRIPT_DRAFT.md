# Draft manuscript scaffold: RiceGeneFormer

Working title: Gene-aware ordinal multi-task learning benchmarks genotype-to-phenotype prediction in 3K rice genomes

Status: writing can start from this scaffold. Claims are bounded to the evidence available as of 2026-06-14.

## Abstract — structured draft placeholder

Plant genotype-to-phenotype prediction increasingly requires models that account for gene structure, trait imbalance and the ordinal nature of many crop descriptor phenotypes. However, deep learning pipelines are difficult to compare fairly against strong marker-based baselines because genome-wide association signals, SNP-to-gene mappings and validation procedures can introduce hidden leakage or unstable model-selection effects. Here we developed RiceGeneFormer-OMTL, a gene-aware ordinal multi-task learning framework for 3K Rice Genome phenotype prediction. The model combines train-fold GWAS priors, SNP-to-gene aggregation, graph-aware gene tokens, trait-query decoding, gated top-SNP fusion and ordinal multi-task losses, and is evaluated with deterministic full-role validation and test procedures. On ten core ordinal rice descriptor traits, the final RiceGeneFormer family reached deterministic test macro-F1 values of 0.3229±0.0062 without distillation and 0.3292±0.0057 with weak expected-score distillation. These results approached a LightGBM top-SNP baseline (macro-F1 0.3354) but remained below balanced top-SNP SNP-MLP baselines (macro-F1 approximately 0.350–0.354). Ablations showed that gated top-SNP fusion and mild class balancing were more important than graph identity, SNP-to-gene mapping choice, post-hoc threshold calibration or simple expected-score distillation. These results provide a leakage-aware plant genotype-to-phenotype benchmark and clarify both the promise and current limits of gene-aware neural models under modest crop phenotype sample sizes.

Notes before final abstract:
- Replace “approximately 0.350–0.354” with exact selected baseline table wording.
- Add raw data citation/accession once verified.
- Decide target journal word limit before shortening.

## Introduction — paragraph jobs

### Paragraph 1: field-scale context

Plant breeding increasingly depends on the ability to predict agronomic and morphological phenotypes from dense genomic variation. In crops such as rice, large genotype panels now provide hundreds of thousands of markers across thousands of accessions, creating opportunities to move beyond single-locus association tests toward multi-trait predictive models.

Evidence to add: citations for genomic selection, 3K Rice Genome, rice descriptor phenotypes.

### Paragraph 2: bottleneck

Many available crop phenotypes are not continuous measurements but descriptor codes, often ordinal and imbalanced across classes. Treating these phenotypes as ordinary regression or flat classification can obscure ordered class structure and can overstate performance when majority classes dominate.

Evidence to add: class distribution table from docs/CORE_TRAIT_CLASS_DISTRIBUTION.tsv.

### Paragraph 3: modeling gap

Marker-based baselines can be strong because top GWAS SNPs capture direct predictive shortcuts, but they do not explicitly organize variation by genes, pathways or trait-specific gene modules. Conversely, gene-aware neural models are attractive but require careful leakage control, fair baselines and deterministic evaluation before biological or methodological claims can be made.

Evidence to add: final baseline table from docs/MANUSCRIPT_STARTER.md.

### Paragraph 4: present study

We therefore built RiceGeneFormer-OMTL, a leakage-aware gene-level ordinal multi-task framework for 3K Rice genotype-to-phenotype prediction. The model integrates train-fold GWAS priors, SNP-to-gene aggregation, gene graph encoding, trait-query decoding, gated top-SNP fusion and class-balanced ordinal training, and is compared against LightGBM, XGBoost and SNP-MLP top-SNP baselines.

### Paragraph 5: contribution and boundary

Our results show that RiceGeneFormer approaches tree-based top-SNP performance and provides a reproducible framework for gene-aware crop G2P modeling, but strong balanced SNP-MLP baselines remain superior in macro-F1 under the present random split. The study therefore frames RiceGeneFormer as a benchmarked modeling platform and diagnostic analysis rather than as a universal replacement for marker-based predictors.

## Results

### Result 1. A leakage-aware ordinal phenotype benchmark from 3K rice genomes

Claim opening:
We first constructed a leakage-aware genotype-to-phenotype benchmark from the 3K Rice Genome panel, focusing on ordinal descriptor traits with sufficient phenotype coverage.

Evidence to write:
- Genotype matrix: 3,000 accessions × 365,710 SNPs.
- Phenotype/mask matrix: 3,000 × 35 traits.
- Main core ordinal set: 10 traits.
- Main split: 1,586 train, 340 validation, 340 test supervised accessions.
- Train-fold GWAS p-values generated per trait and split.
- SNP-to-gene and graph artifacts validated.

Suggested figure/table:
- Figure 1: data processing and leakage-control workflow.
- Supplementary Table 1: trait metadata and class distribution.

### Result 2. RiceGeneFormer-OMTL integrates GWAS priors, gene tokens and ordinal multi-task decoding

Claim opening:
We designed RiceGeneFormer-OMTL to combine marker-level predictive signal with gene-level structure while preserving the ordinal nature of descriptor phenotypes.

Evidence to write:
- SNP-to-gene aggregation from train-fold GWAS priors.
- lightweight GraphEncoder over mapped gene nodes.
- trait-query decoder and ordinal heads.
- top-SNP gated fusion.
- class-balanced ordinal loss and macro-F1 checkpoint selection.

Suggested figure:
- Figure 2: architecture diagram.

### Result 3. Strong top-SNP baselines define the predictive ceiling

Claim opening:
Before interpreting the gene-aware model, we established marker-level baselines using the same train-fold GWAS top-SNP information.

Evidence:
- LightGBM top512: macro-F1 0.3354, accuracy 0.6271, MAE 0.5547.
- XGBoost top512: macro-F1 0.3115, accuracy 0.6274, MAE 0.5662.
- SNP-MLP alpha0.40: macro-F1 0.3502±0.0117, accuracy 0.6141±0.0097, MAE 0.5697±0.0108.
- SNP-MLP alpha0.60: macro-F1 0.3544±0.0112, accuracy 0.5961±0.0086, MAE 0.5976±0.0087.

Interpretation boundary:
SNP-MLP improves macro-F1 by emphasizing minority classes, but this comes with accuracy/MAE trade-offs as alpha increases.

### Result 4. Gated top-SNP fusion and mild class balancing yield the strongest RiceGeneFormer configuration

Claim opening:
RiceGeneFormer improved most when direct top-SNP signal was fused through a learned gate and the ordinal loss was mildly class-balanced.

Evidence:
- gated top-SNP fusion improved validation loss/accuracy/MAE versus additive fusion.
- balanced alpha0.25 plus macro-F1 selection improved minority-class macro-F1.
- full-step no-distillation deterministic test macro-F1: 0.3229±0.0062.
- deterministic test accuracy: 0.5895±0.0029.

Suggested table:
- Main Table 1: final model and baseline metrics.

### Result 5. Expected-score distillation provides only weak incremental gains

Claim opening:
We next tested whether a strong SNP-MLP teacher could transfer marker-level predictive information into RiceGeneFormer through expected-score distillation.

Evidence:
- w=0.10 deterministic test macro-F1: 0.3292±0.0057.
- w=0.20 deterministic test macro-F1: 0.3251±0.0054.
- no-distillation test macro-F1: 0.3229±0.0062.

Interpretation boundary:
Distillation slightly improved the best RiceGeneFormer test macro-F1 at w=0.10, but the effect was small and not monotonic with weight. It should be treated as weak-positive rather than a main breakthrough.

### Result 6. Graph identity, SNP-to-gene mapping and threshold calibration did not produce robust gains

Claim opening:
Several biologically motivated components improved the completeness of the modeling framework but did not produce stable predictive gains in the current bounded setting.

Evidence:
- chr-neighbor, prefix-random, STRING and chr+STRING fusion graphs produced close multi-seed results.
- SNP-to-gene mapping variants body-only, ±2 kb, ±5 kb, ±10 kb and nearest-gene were close in seed42 bounded pilots.
- train-calibrated threshold tuning degraded full-step accuracy/MAE and did not consistently improve macro-F1.

Interpretation boundary:
These results do not prove that biological graphs are unhelpful in general; they show that the current lightweight mean-neighbor GraphEncoder and bounded 3K Rice setting did not support a graph-specific performance claim.

## Methods

### Dataset processing

Write details:
- raw genotype source and exact citation/accession pending verification.
- genotype matrix shape and encoding.
- phenotype sheet selection and ID alignment.
- trait filtering into core ordinal traits.
- split construction.

### Train-fold GWAS priors

Write details:
- p-values computed only on training samples for each split/trait.
- p-value array length equals SNP count.
- top-SNP selection constrained under split/trait directories.

### SNP-to-gene mapping and graph construction

Write details:
- rice gene annotation source pending citation verification.
- window_5kb main mapping.
- body-only, ±2 kb, ±10 kb and nearest-gene ablations.
- chr-neighbor graph, random graph, STRING graph, chr+STRING fusion graph.

### RiceGeneFormer-OMTL architecture

Write details:
- gene bag aggregation.
- gene prior embedding.
- GraphEncoder.
- trait attention / trait-query decoder.
- ordinal heads.
- top-SNP gated fusion.

### Training and model selection

Write details:
- AdamW, learning rate, weight decay, batch size, epochs, steps.
- balanced alpha0.25 for final RiceGeneFormer.
- checkpoint selection by validation macro-F1.
- deterministic full-role evaluation.

### Baselines

Write details:
- LightGBM and XGBoost per trait with train-fold top SNPs.
- SNP-MLP union of top SNPs, class-balanced alpha grid.
- metrics and seed handling.

### Distillation

Write details:
- SNP-MLP teacher export.
- expected-score target.
- masked MSE on observed labels.
- weights 0.10 and 0.20.

## Discussion

### Main message

RiceGeneFormer provides a leakage-aware, gene-aware, ordinal multi-task benchmark for plant genotype-to-phenotype prediction. It approaches tree-based top-SNP performance but does not outperform strong balanced SNP-MLP baselines on the current 3K Rice random split.

### Biological/modeling interpretation

The results suggest that, in modest-sample plant phenotype panels, direct marker-level signal remains difficult for gene-token neural models to surpass. Gene-aware structure provides a principled modeling framework and interpretability hooks, but predictive superiority requires stronger graph encoders, better phenotype coverage, or improved teacher integration.

### Limitations

- Current claims use random split only as main result.
- Cross-subpopulation/region generalization is not yet the main evidence.
- Raw data citation and redistribution policy must be finalized.
- Current GraphEncoder is lightweight and not relation-aware.
- Expected-score distillation is weaker than probability/logit distillation could be.

### Future work

- Relation-aware pathway/co-expression graph encoders.
- KL/logit teacher distillation from strong SNP models.
- Cross-region and cross-subpopulation tests.
- Multi-environment maize/soybean transfer.
- Larger phenotype panels and continuous trait integration.

## Immediate writing checklist

Ready now:
- Results scaffold.
- Methods scaffold.
- main performance numbers.
- per-trait RiceGeneFormer final table.
- class-distribution table.

Still required before submission:
- raw data accession/URL verification.
- per-trait baseline metrics if required for supplementary comparison.
- figure-ready architecture diagram.
- code/data repository and DOI strategy.
- target journal decision.
