# RiceGeneFormer submission redesign for Briefings in Bioinformatics or Nature Communications

Last updated: 2026-06-14

## 1. Strategic diagnosis

The current manuscript is scientifically usable but not yet positioned strongly enough for the two target journals in the same way.

Current core evidence:

| Model / setting | Role | Seeds | macro-F1 | Accuracy | MAE | Spearman | Interpretation |
|---|---|---:|---:|---:|---:|---:|---|
| RiceGeneFormer, no distillation | test | 42/43/44 | 0.3229 ± 0.0062 | 0.5895 ± 0.0029 | 0.5978 ± 0.0039 | 0.2556 ± 0.0053 | final gene-aware model without teacher transfer |
| RiceGeneFormer + expected-score distillation w=0.10 | test | 42/43/44 | 0.3292 ± 0.0057 | 0.5894 ± 0.0049 | 0.5948 ± 0.0067 | 0.2701 ± 0.0057 | best current RiceGeneFormer variant |
| RiceGeneFormer + expected-score distillation w=0.20 | test | 42/43/44 | 0.3251 ± 0.0054 | 0.5906 ± 0.0033 | 0.5904 ± 0.0037 | 0.2757 ± 0.0089 | stronger distillation, not better |
| LightGBM top512 | test | 42 | 0.3380 | 0.6263 | 0.5630 | n/a | strong tree baseline |
| XGBoost top512 | test | 42 | 0.3161 | 0.6332 | 0.5512 | n/a | high accuracy but lower macro-F1 |
| SNP-MLP top512, balanced alpha0.40 | test | 42/43/44 | 0.3644 ± 0.0106 | 0.6317 ± 0.0023 | 0.5639 ± 0.0117 | n/a | strong balanced baseline |
| SNP-MLP top512, balanced alpha0.60 | test | 42/43/44 | 0.3781 ± 0.0029 | 0.6148 ± 0.0091 | 0.5899 ± 0.0176 | n/a | strongest macro-F1 baseline |

Interpretation boundary:

- RiceGeneFormer is close to LightGBM by macro-F1: gap 0.0088, about 2.6% of LightGBM.
- RiceGeneFormer is still below SNP-MLP alpha0.40 by 0.0352 macro-F1 and below alpha0.60 by 0.0489 macro-F1.
- Therefore, the manuscript must not claim predictive state-of-the-art.
- The current contribution is a leakage-aware ordinal G2P benchmark and a gene-aware diagnostic framework.

Definitions:

- macro-F1: unweighted mean of per-class F1; higher values indicate better minority-class performance under imbalance.
- MAE: mean absolute error in ordinal class units; lower is better.
- Spearman: rank correlation between expected ordinal score and observed label.
- leakage-aware: all GWAS/top-SNP features are computed using only training samples of the active split.

## 2. Journal-specific feasibility

### 2.1 Briefings in Bioinformatics (BIB)

Best current fit: medium-high after targeted strengthening.

Why it can fit:

1. The work is a bioinformatics methods/benchmark paper, not only a plant application.
2. It addresses a real evaluation problem: train-fold GWAS leakage, ordinal descriptor targets, class imbalance and deterministic evaluation.
3. It contains a reusable computational workflow: genotype processing, phenotype masking, train-fold GWAS priors, SNP-to-gene mapping, graph construction, model training, deterministic evaluation and baseline alignment.
4. It reports strong baselines and negative controls instead of only favorable neural-model results.

Main BIB rejection risks:

1. Contribution may look too incremental if framed as another transformer-like model.
2. Single crop dataset may look narrow for a high-end bioinformatics journal.
3. The proposed gene-aware model does not beat the best top-SNP neural baseline.
4. Graph and mapping modules did not show strong positive gains.
5. Code/reproducibility must be public and clean.

How to reposition for BIB:

Use this central claim:

> We introduce a leakage-aware benchmark and diagnostic modeling framework for ordinal plant genotype-to-phenotype prediction, showing that strict train-fold GWAS controls and deterministic full-role evaluation materially change how gene-aware neural models should be compared with strong top-SNP baselines.

Do not use this claim:

> RiceGeneFormer is a superior deep-learning predictor for crop phenotype prediction.

BIB title direction:

1. Leakage-aware ordinal genotype-to-phenotype benchmarking in 3K rice genomes
2. RiceGeneFormer: a leakage-aware benchmark for gene-aware ordinal genotype-to-phenotype prediction
3. Benchmarking gene-aware ordinal multi-task learning against top-SNP predictors in 3K rice genomes

BIB manuscript structure:

1. Introduction: evaluation leakage and ordinal phenotype problem.
2. Benchmark construction: 3K Rice, train-fold GWAS, phenotype masks, splits.
3. RiceGeneFormer as a testbed architecture, not the sole novelty.
4. Baseline ceiling: LightGBM/XGBoost/SNP-MLP.
5. Diagnostic findings: top-SNP fusion matters, class balancing matters, graph identity and mapping do not yet matter.
6. Generalization stress tests: cross-region / cross-subpopulation if possible.
7. Reproducibility: public code, manifests, source data, exact URLs.

BIB minimum additional work before submission:

- Required: public code/reproducibility package.
- Strongly recommended: cross-region deterministic evaluation.
- Strongly recommended: full supplementary parameter table for all baselines and RiceGeneFormer runs.
- Optional but useful: one figure showing how sampled validation overestimates or differs from deterministic full-role evaluation.

### 2.2 Nature Communications (NC)

Best current fit: low without major new evidence; possible only after story upgrade.

Why current version is not enough for NC:

1. NC expects an important advance of significance to specialists within the field; a single-dataset benchmark where the proposed model does not beat the strongest baseline is unlikely to pass editorial triage.
2. The current biological finding is weak: no stable graph-specific gain, no demonstrated candidate-gene discovery, no functional validation and no cross-population generalization as main evidence.
3. NC reviewers would likely ask why the main model should matter if balanced SNP-MLP performs better.
4. A benchmark-only paper can appear in NC only if it is broad, reusable and field-setting, ideally across multiple datasets/species/tasks.

What would make NC plausible:

The story must shift from "our model predicts best" to one of these stronger claims:

Option NC-A: robustness under distribution shift

> Gene-aware ordinal multi-task learning does not dominate random-split prediction, but it improves robustness and interpretability under cross-region or cross-subpopulation distribution shift.

Evidence required:

- Cross-region or cross-subpopulation test roles.
- Same baselines under the same leakage-safe top-SNP rules.
- RiceGeneFormer must either outperform or substantially narrow the gap under distribution shift.
- Demonstrate that top-SNP baselines degrade more strongly than gene-aware model.

Option NC-B: biological discovery / mechanistic prioritization

> RiceGeneFormer provides gene-level modules that recover known rice trait biology and nominate credible candidate genes while retaining competitive prediction.

Evidence required:

- Top attended genes per trait.
- Known rice gene/QTL overlap.
- Pathway or STRING neighborhood enrichment.
- At least 2-3 trait case studies.
- Ideally independent literature support for candidate genes.

Option NC-C: reusable cross-crop framework

> A leakage-aware ordinal G2P framework generalizes from rice to another crop or task.

Evidence required:

- Additional dataset, e.g. maize G2F or soybean phenotype panel.
- Same train-fold prior principle.
- Demonstrate transfer or consistent diagnostic conclusions.

NC recommendation:

- Do not submit the current manuscript directly to NC.
- Only target NC if at least one of NC-A, NC-B or NC-C is completed.
- The most efficient NC path is NC-A + NC-B: cross-region robustness plus biological gene/QTL interpretation.

## 3. Redesigned scientific story

### 3.1 Current story: too weak for BIB/NC

Current draft story:

> RiceGeneFormer is a gene-aware ordinal model for 3K rice. It approaches LightGBM but remains below SNP-MLP.

Problem:

- This sounds honest but not strong enough for BIB/NC.
- The method becomes secondary to a stronger baseline.

### 3.2 Redesigned story for BIB

BIB-ready story:

> Crop genomic prediction benchmarks can be misleading when GWAS-derived priors, top-SNP baselines and model selection are not split-aware and role-specific. We built a leakage-controlled ordinal benchmark in 3K rice and used RiceGeneFormer as a diagnostic architecture to test which gene-aware components matter. The results show that direct top-SNP signal and class imbalance dominate random-split performance, while graph identity and mapping-window choices provide limited gains under the present sample size. This establishes a reproducible benchmark and a realistic performance boundary for gene-aware neural G2P models.

Why this works:

- The paper contributes a benchmark methodology and diagnostic conclusion.
- Strong SNP-MLP baseline becomes part of the insight, not a failure.
- Negative graph results define a useful boundary.

### 3.3 Redesigned story for NC

NC-ready story requires new evidence:

> Gene-aware ordinal G2P models are not necessarily superior on random splits, but they provide more robust and biologically interpretable predictions under realistic population shift. In 3K rice, leakage-safe train-fold evaluation shows that top-SNP models dominate random-split macro-F1, whereas gene-aware modeling narrows or reverses this gap in cross-region or cross-subpopulation settings and recovers trait-relevant genes/QTLs.

Why this could work:

- It converts a negative random-split result into a field-relevant insight.
- It addresses practical breeding deployment: performance under shifted germplasm distributions.
- It adds biological interpretability.

## 4. Critical experiments to add

### Priority 1: deterministic cross-region evaluation

Goal:

Test whether RiceGeneFormer is more robust than top-SNP baselines when test samples come from held-out geographic regions.

Required comparisons:

- RiceGeneFormer best family: gated top-SNP fusion + alpha0.25 + macro-F1 selection, with and without w=0.10 distillation if teacher export supports it.
- LightGBM top512.
- XGBoost top512.
- SNP-MLP alpha0.40 and alpha0.60.
- Majority baseline.

Metrics:

- macro-F1, accuracy, MAE, Spearman if available.
- Per-trait macro-F1.
- Mean rank across traits and regions.
- Degradation from random-split test to cross-region test.

Decision rule:

- If RiceGeneFormer beats or approaches SNP-MLP under cross-region while being lower in random split: this becomes the main manuscript result for NC/BIB.
- If RiceGeneFormer remains below SNP-MLP: still useful for BIB as benchmark evidence, but not enough for NC.

Expected resource:

- CPU baselines: 4-30 CPU, 16-80 GB RAM, hours depending grid.
- RiceGeneFormer: run on gpu10; use GPU 2 preferentially; 1 A100, 8-16 CPU, 40-80 GB RAM, about 2-8 h per split depending full-step setting.

### Priority 2: cross-subpopulation if metadata exists or can be recovered

Goal:

Test indica/japonica/aus/admixture-like population shift if subpopulation labels can be obtained from 3K metadata or SNP-Seek passport data.

Why important:

- NC/BIB reviewers will value population-shift evaluation more than random split.
- It directly tests practical breeding generalization.

If subpopulation labels cannot be recovered quickly:

- Use region split as the main distribution-shift evidence.
- State subpopulation metadata was not available in the current local 3K metadata table.

### Priority 3: trait-level gene/QTL interpretation

Goal:

Provide biological evidence that RiceGeneFormer is not just weaker than SNP-MLP, but offers gene-level interpretability.

Required outputs:

1. For each core trait:
   - top 50 genes by attention or integrated attribution.
   - gene scores averaged across seeds.
   - stability across seeds.

2. Known evidence overlap:
   - known rice genes from literature for plant height, flowering time, panicle traits, seed traits where applicable.
   - QTL overlap if a rice QTL database is available.

3. Enrichment:
   - STRING neighborhood or GO/pathway enrichment if mapping permits.
   - Report only cautiously if enrichment is weak.

Decision rule:

- If top genes overlap known biology for at least 2-3 traits: strengthen BIB and make NC more plausible.
- If no overlap: keep as supplementary negative control and avoid NC.

### Priority 4: probability/logit-level distillation instead of expected-score MSE

Goal:

Expected-score MSE is too coarse. Improve student-teacher transfer from SNP-MLP.

Options:

1. Per-trait KL divergence over ordinal class probabilities.
2. Temperature-scaled teacher probability distillation.
3. Hybrid loss: ordinal CE + alpha0.25 + KL teacher loss.

Decision rule:

- Continue only if deterministic full-val macro-F1 improves by at least 0.01 and test macro-F1 improves consistently across 3 seeds.
- Stop if gains are only sampled-validation or single-seed.

This is useful for BIB but not sufficient for NC without cross-shift or biology.

### Priority 5: repository and reproducibility package

Required for both BIB and NC:

- Public GitHub repository with scripts and documentation only.
- No raw genotype/phenotype redistribution unless license allows.
- `README.md` with exact pipeline.
- `environment.yml` or `uv`/pip equivalent if possible.
- `docs/figure_source_data/*.csv` included.
- `docs/DATA_SOURCES_AND_CITATIONS.md` included or converted to Data Availability.
- Zenodo DOI release.
- Example manifests and tiny smoke data if possible.

## 5. Revised figure plan

### BIB version

Figure 1. Leakage-aware ordinal G2P benchmark

Panels:
- 3K Rice genotype/phenotype alignment.
- Train-fold GWAS prior generation.
- Random and cross-region split design.
- Deterministic evaluator.

Figure 2. RiceGeneFormer architecture as diagnostic framework

Panels:
- SNP-to-gene GeneBag.
- GraphEncoder.
- trait-query ordinal heads.
- gated top-SNP branch.

Figure 3. Random-split performance ceiling

Panels:
- RiceGeneFormer vs LightGBM/XGBoost/SNP-MLP on test macro-F1, accuracy, MAE.
- Per-trait performance heatmap.
- Class imbalance vs macro-F1.

Figure 4. Distribution-shift evaluation

Panels:
- cross-region macro-F1 across methods.
- random-to-region degradation.
- per-trait robustness.

Figure 5. Diagnostic ablations

Panels:
- top-SNP fusion/add/gated.
- class-balance alpha.
- graph identity negative control.
- SNP-to-gene mapping negative control.

Supplementary:
- Full hyperparameters.
- Per-trait metrics.
- All seed manifests.
- Calibration/distillation diagnostics.

### NC version

NC needs fewer but stronger main figures:

Figure 1. Field-scale problem and leakage-aware benchmark.

Figure 2. Main discovery: random split favors top-SNP baselines, but cross-region/cross-population evaluation changes the ranking or reveals robustness gap.

Figure 3. RiceGeneFormer architecture and ablation evidence for the winning component.

Figure 4. Biological interpretation: trait gene modules, QTL/known gene overlap, pathway/STRING support.

Figure 5. Reproducibility and external validation, if available.

If Figure 2 or Figure 4 cannot be made strong, do not target NC.

## 6. Revised manuscript claims

### Claims safe now

1. A leakage-aware ordinal G2P benchmark was built from 3K Rice.
2. Deterministic full-role evaluation avoids sampled-validation artifacts.
3. Strong top-SNP baselines define a high predictive ceiling.
4. RiceGeneFormer approaches LightGBM but remains below balanced SNP-MLP on random split.
5. Gated top-SNP fusion and mild class balancing are the most useful RiceGeneFormer changes.
6. Current lightweight graph identity and SNP-to-gene mapping variants do not show robust predictive gains.

### Claims requiring new experiments

1. RiceGeneFormer is more robust under distribution shift.
2. RiceGeneFormer recovers trait-relevant genes or pathways.
3. Gene-aware modeling provides biological insight beyond top-SNP baselines.
4. The framework generalizes beyond rice or beyond random splits.

### Claims to avoid

1. RiceGeneFormer outperforms all baselines.
2. STRING or chromosome graph improves prediction.
3. SNP-to-gene mapping choice is biologically unimportant.
4. Expected-score distillation is a major innovation.
5. The method is broadly generalizable without cross-population or cross-species evidence.

## 7. Decision tree

### Step 1: Add cross-region deterministic evaluation

If RiceGeneFormer improves relative robustness:

- Target BIB strongly.
- Consider NC only if biological interpretation is also positive.

If RiceGeneFormer remains below SNP-MLP in all cross-region settings:

- Target BIB/GigaScience/The Plant Genome.
- Do not target NC.

### Step 2: Add gene/QTL interpretation

If interpretation is biologically coherent:

- BIB story becomes strong.
- NC becomes possible if paired with robustness.

If interpretation is weak:

- Keep the paper as benchmark/methods.
- Prefer BIB or GigaScience, not NC.

### Step 3: Decide final target

Recommended final strategy:

1. First, optimize for BIB because it is ambitious but realistic.
2. Only switch to NC if both cross-region robustness and biological interpretation produce strong positive evidence.
3. If BIB desk-rejects, transfer/reformat to GigaScience or The Plant Genome.

## 8. Concrete execution plan

### Task A: Cross-region benchmark audit and deterministic evaluator extension

Objective:

Confirm that existing region splits can be used for all models and make all baselines support `--split-name` plus deterministic `eval-role=test`.

Files likely involved:

- `scripts/model/rice_geneformer_omtl_train.py`
- `scripts/model/evaluate_rice_geneformer_checkpoint.py`
- `scripts/model/baseline_lightgbm_core_smoke.py`
- `scripts/model/baseline_xgboost_core_smoke.py`
- `scripts/model/baseline_snp_mlp_core_smoke.py`
- new `scripts/slurm/run_region_shift_benchmark.sh`
- new `docs/REGION_SHIFT_BENCHMARK_PLAN.md`

Verification:

- CPU smoke for one small role.
- JSON manifests pass `json.tool`.
- split disjointness and role counts written to summary TSV.

### Task B: Run region-shift baselines

Objective:

Evaluate LightGBM, XGBoost and SNP-MLP alpha0.40/0.60 on each region holdout.

Output:

- `data/3krice/processed/region_shift_baseline_summary_<job>.tsv`
- lightweight summary copied to `docs/REGION_SHIFT_BASELINE_SUMMARY.md`

Resource:

- CPU only, 8-30 CPU, 16-80 GB RAM, likely 1-8 h depending number of splits.

### Task C: Run region-shift RiceGeneFormer

Objective:

Train/evaluate RiceGeneFormer best configuration on each region split.

Configuration:

- top512
- gated top-SNP fusion
- trait_attention
- GraphEncoder baseline chr graph or fusion only if fixed
- alpha0.25
- macro-F1 selection
- optional w=0.10 distillation only if teacher predictions exist per split

Resource:

- Run on gpu10.
- Prefer `CUDA_VISIBLE_DEVICES=2` per user preference.
- 1 A100 GPU, 8-16 CPU, 40-80 GB RAM, about 2-8 h per split.

### Task D: Interpret gene-level signals

Objective:

Export trait-specific gene scores from RiceGeneFormer best checkpoints and test overlap with known rice trait genes/QTLs.

Files likely needed:

- new `scripts/model/export_rice_geneformer_gene_scores.py`
- new `scripts/analysis/known_rice_trait_overlap.py`
- new `docs/GENE_INTERPRETATION_RESULTS.md`

Verification:

- top gene TSV has one row per trait-gene pair.
- seed stability summary exists.
- known-gene/QTL overlap report is generated.

### Task E: Rewrite manuscript for target

BIB version:

- Rename manuscript framing from model-centered to benchmark-centered.
- Put strong SNP-MLP baseline in main text, not hidden.
- Add cross-region if available.
- Add reproducibility package section.

NC version:

- Only start if Task B/C/D give strong positive evidence.
- Lead with distribution-shift robustness or biological interpretation.
- Reduce technical ablations in main text; move to supplement.

## 9. Recommended immediate next action

Do not submit the current version to NC.

Recommended next move:

1. Build and run cross-region deterministic benchmark for all methods.
2. In parallel, implement gene-score export and known gene/QTL overlap.
3. After these two results, choose:
   - If both positive: prepare NC pre-submission style package.
   - If only robustness positive or only benchmark remains strong: submit to BIB.
   - If both weak: submit to GigaScience or The Plant Genome with benchmark/resource framing.

## 10. One-sentence final target recommendation

The realistic ambitious path is BIB after cross-region and reproducibility hardening; NC should be treated as a conditional stretch target that requires a new positive result on distribution-shift robustness and/or biologically meaningful gene/QTL interpretation.
