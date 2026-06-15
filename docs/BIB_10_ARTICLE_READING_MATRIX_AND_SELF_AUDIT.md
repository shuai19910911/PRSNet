# BIB 10+ article reading matrix and manuscript self-audit for RiceGeneFormer

Last updated: 2026-06-15

Purpose: record a deliberate reading pass over at least ten Briefings in Bioinformatics (BIB) articles that are structurally close to RiceGeneFormer: benchmark papers, genomic prediction papers, machine-learning bioinformatics methods, cross-dataset generalization studies and negative-control diagnostic studies. This file controls the next manuscript rewrite.

## Articles read

### 1. GWKBR: a novel method integrating machine learning and Bayesian inference framework to improve genomic prediction accuracy

- URL: https://academic.oup.com/bib/article/27/2/bbag133/8554110
- DOI: 10.1093/bib/bbag133
- Similarity to RiceGeneFormer: very high. It is a genomic prediction paper using GWAS-derived weights and benchmarking across species/traits.
- Structure observed: strong abstract with method name, conceptual motivation, comparison set and multi-dataset result; Introduction builds from additive/non-additive genetic effects to limitations of GBLUP/Bayes/kernel methods; Methods first defines model, then datasets, comparison methods and evaluation; Results emphasizes accuracy across 23 traits and comparative ranking.
- Writing lesson: for RiceGeneFormer, the introduction should explicitly position marker baselines, GWAS weighting and non-additive/gene-aware modeling before model architecture. The manuscript should include a fair baseline list early, not after model promotion.

### 2. WheatGP, a genomic prediction method based on CNN and LSTM

- URL: https://academic.oup.com/bib/article/26/2/bbaf191/8119370
- DOI: 10.1093/bib/bbaf191
- Similarity: very high. Crop genomic prediction, deep learning, multiple traits, cross-species adaptability and interpretability.
- Structure observed: starts with food security and breeding need; then high-dimensional marker challenge; then CNN/LSTM rationale; Results reports trait-level prediction accuracy, comparison with rrBLUP/SVR/XGBoost/DNNGP, dimensionality-reduction efficiency and SHAP interpretation.
- Writing lesson: RiceGeneFormer needs a clearer crop-breeding entry point and a more explicit statement that top-SNP baselines are not weak strawmen. Figure 2 should teach model modules; Figure 3 should include baseline comparison and hard-trait behavior.

### 3. Improving multi-population genomic prediction accuracy using multi-trait GBLUP models which incorporate global or local genetic correlation information

- URL: https://academic.oup.com/bib/article/25/4/bbae276/7690343
- DOI: 10.1093/bib/bbae276
- Similarity: high. Genomic prediction across populations and genetic correlation; useful for our cross-region framing.
- Structure observed: the paper treats population/environment differences as a central methodological issue and builds Results around whether global/local correlation improves prediction. It reports model variants in a staged manner and emphasizes when combining populations fails.
- Writing lesson: RiceGeneFormer should not hide cross-region results. The cross-region failure should be written as a diagnostic benchmark result analogous to multi-population GP boundary testing.

### 4. Should we really use graph neural networks for transcriptomic prediction?

- URL: https://academic.oup.com/bib/article/25/2/bbae027/7591104
- DOI: 10.1093/bib/bbae027
- Similarity: very high in argumentative style. It asks whether a biologically structured neural architecture actually improves prediction over simpler baselines.
- Structure observed: title is a question; abstract is candid; Results compare GNNs with simpler models and emphasize computation-performance trade-off; Discussion explains why biological networks may not add predictive information.
- Writing lesson: This is the closest style model for our negative graph result. RiceGeneFormer should explicitly say that graph-aware architecture is useful infrastructure but current graph identity does not create stable gains.

### 5. Genome-wide association neural networks identify genes linked to family history of Alzheimer’s disease

- URL: https://academic.oup.com/bib/article/26/1/bbae704/7945355
- DOI: 10.1093/bib/bbae704
- Similarity: high. Neural networks applied to GWAS/gene-level interpretation, with post hoc biological enrichment.
- Structure observed: method is positioned as complementary to classical GWAS, not a replacement; gene hits are followed by enrichment, PPI and druggability analyses; code and UK Biobank provenance are clearly stated.
- Writing lesson: RiceGeneFormer lacks enough positive biological validation to mimic this claim. Therefore we should explicitly contrast our gene-attention outputs with the level of evidence needed for a discovery paper.

### 6. Benchmarking community drug response prediction models: datasets, models, tools, and metrics for cross-dataset generalization analysis

- URL: https://academic.oup.com/bib/article/27/1/bbaf667/8422735
- DOI: 10.1093/bib/bbaf667
- Similarity: very high. Benchmark framework, cross-dataset generalization, no single model dominates, reusable workflow.
- Structure observed: problem is inconsistent benchmarking; contributions are datasets + models + workflow + metrics; Results emphasize external performance drops; Discussion says the goal is not model superiority but a standardized benchmark that cuts through hype.
- Writing lesson: RiceGeneFormer should use the same stance: no model-superiority claim, benchmark and diagnostic framework, external/cross-region evaluation as main evidence.

### 7. Benchmark of embedding-based methods for accurate and transferable prediction of drug response

- URL: https://academic.oup.com/bib/article/24/3/bbad098/7085476
- DOI: 10.1093/bib/bbad098
- Similarity: high. It develops a method but also benchmarks representative methods across cross-panel/cross-dataset settings.
- Structure observed: Abstract combines new method, benchmark, cross-panel transfer and web server; Results move from method design to benchmark to transferability to tool release.
- Writing lesson: RiceGeneFormer should make the benchmark artifact and source-data release visible, even though the DOI is still pending. Without DOI, use future-tense availability.

### 8. Benchmarking computational methods for m6A profiling with Nanopore direct RNA sequencing

- URL: https://academic.oup.com/bib/article/25/2/bbae001/7590315
- DOI: 10.1093/bib/bbae001
- Similarity: high for benchmark mechanics. It compares many tools across datasets and reports precision/recall trade-offs and workflow reproducibility.
- Structure observed: pipeline name, containerized workflow, datasets, tools, reference sets, precision-recall trade-offs, depth sensitivity and practical recommendations.
- Writing lesson: RiceGeneFormer Methods should be more procedural: data sources, split control, GWAS priors, model, baselines, cross-region and attention audit must be reproducible in order.

### 9. A benchmarking study of individual somatic variant callers and voting-based ensembles for whole-exome sequencing

- URL: https://academic.oup.com/bib/article/26/1/bbae697/7960049
- DOI: 10.1093/bib/bbae697
- Similarity: high for benchmark structure and cost-performance reporting.
- Structure observed: compares 20 callers and thousands of ensembles; reports best individual tools, best ensembles and practical cost-effective recommendation; separates SNV and indel result blocks.
- Writing lesson: RiceGeneFormer should separate final performance, ablation and practical recommendation. The practical recommendation is not “use RiceGeneFormer everywhere” but “use this framework to test gene-aware models against balanced SNP baselines.”

### 10. Machine-learning scoring functions trained on complexes dissimilar to the test set already outperform classical counterparts on a blind benchmark

- URL: https://academic.oup.com/bib/article/22/6/bbab225/6308682
- DOI: 10.1093/bib/bbab225
- Similarity: high for leakage/generalization logic. It explicitly studies similarity between train and test and the meaning of benchmark competitiveness.
- Structure observed: begins with a controversy; designs dissimilar-training benchmark; compares classical vs ML scoring functions; interprets when benchmark similarity matters.
- Writing lesson: RiceGeneFormer should discuss random split vs cross-region split as different evidence levels, not as interchangeable validation.

### 11. Assessing computational predictions of antimicrobial resistance phenotypes from microbial genomes

- URL: https://academic.oup.com/bib/article/25/3/bbae206/7665136
- DOI: 10.1093/bib/bbae206
- Similarity: high for phenotype-from-genome benchmark and lineage/generalization risk.
- Structure observed: compares ML methods and rule-based ResFinder across 78 species-antibiotic datasets; emphasizes that ML excels for closely related strains, while rule-based catalog methods can generalize better for divergent genomes.
- Writing lesson: RiceGeneFormer should explicitly state that strong within-split performance does not equal cross-region robustness, mirroring lineage-divergence logic.

## Common BIB structure distilled from the 11 articles

1. Title is concrete and searchable; it often contains method name or benchmark target, but avoids vague biological hype.
2. Abstract is a mini-paper: problem, gap, method/framework, datasets, baseline comparison, main boundary, availability.
3. Key points are specific and evidence-based, usually 4-5 bullets.
4. Introduction typically has 5-7 paragraphs: field need, data/model challenge, existing methods, benchmark/evaluation gap, proposed framework, contribution and boundary.
5. Results are not just a metric list. They move by evidence ladder: benchmark construction -> method design -> baseline comparison -> final performance -> transfer/generalization -> ablation/diagnostic -> practical recommendation.
6. Methods are detailed and procedural. Dataset identity, preprocessing, split construction, tool/model parameters, evaluation metrics and code release are explicitly described.
7. Figures are evidence maps. BIB benchmark papers often use: pipeline figure, data/model schematic, main performance comparison, cross-dataset/generalization matrix, ablation/diagnostic figure and supplementary per-dataset/per-trait details.
8. Discussion is not a victory lap. Strong BIB papers state when simpler models win, what benchmark failures mean and how users should choose methods.
9. Availability is important. Code, workflow, web server or source data are usually visible in either Abstract, Methods or Data availability.
10. Negative findings are acceptable when they clarify method choice, benchmark reliability or generalization limits.

## Difference audit against current RiceGeneFormer manuscript

| Dimension | BIB pattern from 10+ papers | Current RiceGeneFormer gap | Required optimization |
|---|---|---|---|
| Intro depth | 5-7 paragraphs, gradually narrowing from biological need to benchmark gap | Current draft is close but shorter than BIB examples | Expand with one paragraph on why top-SNP baselines are strong and one paragraph on why graph/attention claims need audits |
| Results order | Benchmark -> method -> baselines -> final model -> diagnostics -> practical guidance | Current order mostly matches | Add explicit practical recommendation subsection |
| Methods detail | Reproducible workflow, hyperparameters, datasets, metrics | Current Methods good but compressed | Expand split/GWAS/baseline/cross-region/gene-attention procedural detail |
| Generalization | Cross-dataset/cross-population is central | Cross-region included but could be more prominent | Keep cross-region as main Result subsection and connect it to benchmark-validity logic |
| Claims | Honest about where method does not win | Current claims are conservative | Preserve this; do not imitate positive-discovery BIB papers |
| Figures | Main figures should be dense but readable; supplementary per-trait/per-dataset heatmaps common | Figures were improved but Figure 2 remains more schematic than benchmark papers | Acceptable for method figure; keep Figure 4 diagnostic and Figure S1 per-trait heatmap |
| Availability | DOI/code/source data visible | DOI pending | Keep future-tense but state exact intended release artifacts |

## Final rewrite rules applied

1. Rewrite manuscript as a BIB benchmark article, not a Nature-style discovery article.
2. Make cross-region and gene-attention audits main-text results, not afterthoughts.
3. Add a practical guidance paragraph: for immediate prediction, balanced SNP-MLP is stronger; for gene-aware diagnostics, RiceGeneFormer is useful as a controlled framework.
4. Keep “approached LightGBM but below SNP-MLP” as the central honest result.
5. Add explicit comparison to the evidence level of GWANN-like biological claims: our attention audit does not meet that threshold.
6. Preserve source-data and DOI pending status; do not claim release is already available.
