# RiceGeneFormer BIB/SCI figure redesign plan

Last updated: 2026-06-15

## Core diagnosis of the current figures

The current figure set is too sparse and too schematic for a serious SCI/BIB benchmark manuscript. It has three main problems:

1. Too few evidence layers. A benchmark paper should show data structure, leakage control, model design, main comparisons, hard-case behaviour, robustness boundaries, and interpretation boundaries as separate but visually connected evidence blocks.
2. Too much diagram-like presentation. The current figures look like project slides because several panels are workflow summaries instead of source-data-driven panels.
3. Insufficient reviewer-facing diagnostics. A skeptical reviewer will ask whether the baseline gap, class imbalance, cross-region failure, and gene-attention limitation are visible directly in the figures.

## Redesigned main-figure suite

### Figure 1. Benchmark data landscape and leakage-safe evaluation design

Core conclusion: the benchmark is a controlled leakage-aware ordinal phenotype benchmark rather than a generic model demo.

Panel map:
- a. End-to-end workflow from public 3K Rice/SNP-Seek inputs to split-aware GWAS priors, top-SNP features, RiceGeneFormer, baselines and deterministic evaluation.
- b. Sample and feature scale card: 3,000 accessions, 365,710 SNPs, 35 descriptor-code traits, 10 core ordinal traits, train/val/test 1,586/340/340.
- c. Trait coverage and class imbalance lollipop/bar plot for the ten core traits.
- d. Leakage-control matrix showing which roles are allowed to influence GWAS, feature selection, checkpoint selection, calibration and final test reporting.

Why this is better: it converts the method claim into a reviewable benchmark design figure.

### Figure 2. RiceGeneFormer architecture and evidence flow

Core conclusion: RiceGeneFormer is a gene-aware diagnostic framework that deliberately combines gene tokens with a controlled top-SNP shortcut.

Panel map:
- a. Architecture schematic: SNP dosage -> SNP-to-gene bags -> gene tokens + train-fold GWAS prior -> graph encoder -> trait-query attention -> ordinal heads.
- b. Gated top-SNP fusion branch with direct marker signal and learned gate.
- c. Model component utility panel: top-SNP input, gated fusion, trait-query attention, class balancing and distillation as positive/weak/neutral effects.
- d. Graph construction summary: chromosome-neighbour, random, STRING and fusion graph options.

Why this is better: separates architecture from empirical claims and avoids over-selling graph biology.

### Figure 3. Main benchmark performance against strong top-SNP baselines

Core conclusion: RiceGeneFormer approaches LightGBM but remains below balanced SNP-MLP.

Panel map:
- a. Forest/dot-whisker plot of macro-F1 with mean +/- SD for all final methods.
- b. Accuracy versus MAE trade-off scatter, with macro-F1 encoded by size or color.
- c. Compact rank table/heatmap for macro-F1, accuracy and MAE.
- d. Difference-from-best baseline panel showing RiceGeneFormer gap to SNP-MLP alpha0.60 and LightGBM.

Why this is better: the main result becomes visually honest and reviewer-proof.

### Figure 4. Hard traits, class imbalance and ordinal behaviour

Core conclusion: the remaining error is concentrated in hard imbalanced ordinal traits.

Panel map:
- a. Per-trait macro-F1 heatmap across methods.
- b. Largest-class fraction versus RiceGeneFormer macro-F1 scatter with trait labels.
- c. Per-trait accuracy and MAE paired bars for RiceGeneFormer.
- d. Hard-trait quadrant: low macro-F1 / high MAE traits highlighted.

Why this is better: this is the missing biological/phenotype-data explanation for model limitations.

### Figure 5. Cross-region benchmark and generalization boundary

Core conclusion: random-split benchmark competitiveness does not imply cross-region robustness.

Panel map:
- a. Cross-region macro-F1 heatmap across methods and held-out regions.
- b. Region-wise method ranking or best-method marker.
- c. RiceGeneFormer gap to SNP-MLP alpha0.60 by region.
- d. Majority-baseline context for accuracy/MAE, showing why macro-F1 is needed.

Why this is better: this directly matches the BIB benchmark lesson and prevents unsupported robustness claims.

### Figure 6. Ablation and diagnostic boundary checks

Core conclusion: several biologically motivated modules are useful infrastructure but not stable performance drivers in the current data regime.

Panel map:
- a. Distillation weight curve with non-monotonic macro-F1.
- b. Graph identity pilot comparison: chromosome-neighbour, random, STRING, fusion.
- c. SNP-to-gene mapping window comparison.
- d. GWAS-top2048 replacement result versus final RiceGeneFormer family.
- e. Short conclusion tile: graph identity and mapping window do not support stronger biological claims.

Why this is better: negative controls are made visible rather than hidden.

### Figure 7. Gene-attention interpretation audit

Core conclusion: attention output is diagnostic only; bounded gene selection constrains biological interpretation.

Panel map:
- a. Top attention genes per selected trait as a ranked dot plot.
- b. Seed-rank stability versus mean attention for top genes.
- c. Visible gene-set chromosome-prefix schematic showing that current max_genes=2048 is not genome-wide.
- d. Known-gene visibility/recovery matrix for SD1, GS3, GW2, qSW5/GW5, DEP1, Hd1, Hd3a and Gn1a.
- e. GWAS-top2048 audit tile: corrected visibility in principle but lower performance and no known-gene recovery.

Why this is better: this turns a weakness into a credible interpretation audit.

## Supplementary figure set

- Supplementary Figure 1. Full per-trait x method performance matrix with macro-F1, accuracy and MAE.
- Supplementary Figure 2. Seed-level final metrics and training trajectories for RiceGeneFormer and SNP-MLP where available.
- Supplementary Figure 3. Source-data/provenance map: which files feed each figure.
- Supplementary Figure 4. Full top-200 gene-attention tables summarized by trait.

## R implementation plan

The new figures should be generated with R only, using ggplot2 + patchwork. Each figure should export:

- editable SVG
- PDF
- 600 dpi TIFF
- PNG preview
- per-panel source-data CSV/TSV files

The target style is BIB/SCI benchmark style:

- white background, no decorative gradients
- consistent method colors across all figures
- small but readable fonts, no overlapping labels
- dot-whisker / heatmap / lollipop / tile panels rather than slide-like cartoons
- explicit negative findings in figure titles or panel annotations

## Current blocker

The current execution environment does not have R/Rscript installed. `Rscript --version` returns command not found. Therefore, I can write the R plotting script and design package now, but I cannot render the final R figures in this environment until R is installed or the script is run in an R-enabled environment.
