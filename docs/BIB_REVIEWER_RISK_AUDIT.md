# BIB reviewer-risk audit for RiceGeneFormer

Last updated: 2026-06-14

Purpose: adversarial pre-submission audit for a BIB-style benchmark/diagnostic manuscript. This file records likely reviewer objections, current evidence, manuscript response, and remaining actions. It does not contain raw data, logs, model weights or credentials.

## Executive assessment

Current recommended framing: submit as a benchmark/diagnostic methods paper, not as a model-superiority or biological-discovery paper.

The manuscript is strongest when it argues that RiceGeneFormer provides a leakage-aware, ordinal-aware and gene-aware framework for fair genotype-to-phenotype evaluation in 3K rice. The manuscript is weakest if it claims that the model is the best predictor, robust under geographic shift, or biologically validated by attention. Those stronger claims are not supported by the frozen evidence.

## Reviewer-risk table

| Risk | Likely reviewer concern | Current evidence | Manuscript action | Status |
|---|---|---|---|---|
| Model does not beat strongest baseline | Balanced SNP-MLP outperforms RiceGeneFormer by macro-F1 | RiceGeneFormer w=0.10 macro-F1 0.3292±0.0057; SNP-MLP alpha0.40/0.60 macro-F1 0.3644±0.0106 to 0.3781±0.0029 | Abstract, Results, Discussion and Table 1 state this directly | Controlled |
| Contribution may look negative | Reviewers may ask why publish a method that does not win | Contribution reframed as leakage-aware benchmark, deterministic evaluation, strong baseline comparison and diagnostic boundary mapping | BIB package and manuscript title use benchmark/diagnostic language | Controlled, but cover letter must emphasize community value |
| Robustness claim unsupported | Held-out regions could expose distribution-shift failure | RiceGeneFormer below SNP-MLP alpha0.60 in Southeast Asia, South Asia and East Asia | Result 7, Figure 4, Table 2 state no cross-region robustness advantage | Controlled |
| Attention interpretation overclaim | Attention over prefix genes can be mistaken for gene discovery | Initial 2048-gene runs used early chr1 prefix; GWAS-top2048 pilot reached macro-F1 0.2990 and did not recover known-gene panel | Result 7 and Figure 4 mark attention as diagnostic only | Controlled |
| Biological graph advantage absent | STRING/chromosome graphs did not show stable gains | Graph identity pilots were close across chromosome, random, STRING and fusion graphs | Result 6 states no graph-specific advantage for current lightweight encoder | Controlled |
| Leakage concern | GWAS priors and top SNPs can leak if computed on all samples | Pipeline uses train-fold GWAS and split-specific top-SNP selection | Abstract, Methods, Result 1 and title emphasize leakage-aware design | Controlled |
| Metric choice concern | Accuracy can hide imbalanced class failure | Reports macro-F1, accuracy and MAE; class balancing experiments included | Result 1 and Table 1 define metrics and trade-offs | Controlled |
| Data/code availability not finalized | No DOI yet | Source-data directory and availability plan prepared, DOI pending | Use future-tense availability wording until DOI is minted | Open pre-submission action |
| Citation/source formatting incomplete | Public data sources need final journal reference style | Source identities tracked in DATA_SOURCES_AND_CITATIONS.md | Final reference formatting still needed | Open pre-submission action |
| Figure/source-data consistency | Reviewers may expect source data for figures | docs/source_data and figure_source_data prepared; short legends point to public source-data paths | Need final panel freeze and DOI-backed archive | Mostly controlled |

## Claim audit

### Supported main claims

1. The benchmark is leakage-aware.
   - Evidence: train-fold GWAS priors, split-specific top-SNP selection, deterministic full-role evaluation.
   - Manuscript locations: Abstract, Result 1, Methods, title.

2. RiceGeneFormer approaches but does not surpass strong marker baselines.
   - Evidence: RiceGeneFormer w=0.10 macro-F1 0.3292±0.0057; LightGBM 0.3380; SNP-MLP alpha0.40/0.60 0.3644–0.3781.
   - Manuscript locations: Abstract, Result 3, Result 4, Discussion, Table 1.

3. Gated top-SNP fusion and mild class balancing are useful within the RiceGeneFormer family.
   - Evidence: pilot and final configuration summaries; final no-distillation and w=0.10 distillation metrics.
   - Manuscript locations: Result 4, Figure 2, Figure 4.

4. Cross-region and gene-attention audits define claim boundaries.
   - Evidence: all region gaps negative vs SNP-MLP alpha0.60; prefix-gene limitation; GWAS-top2048 seed42 macro-F1 0.2990.
   - Manuscript locations: Result 7, Figure 4, Table 2.

### Claims that must not be made

1. RiceGeneFormer is state-of-the-art or superior to all baselines.
2. RiceGeneFormer is robust to geographic distribution shift.
3. Gene attention validates known rice genes or discovers new biological mechanisms.
4. STRING or chromosome graph structure improves prediction in the current implementation.
5. Code/data are already available through a DOI-backed release.
6. Raw 3K Rice/SNP-Seek files are redistributed by this repository.

## Required wording guardrails

Use:

- "approached LightGBM but remained below balanced SNP-MLP baselines"
- "diagnostic framework"
- "leakage-aware benchmark"
- "did not support a robustness or biological-discovery claim"
- "will be released before submission" until the DOI exists

Avoid:

- "outperformed strong baselines"
- "state-of-the-art"
- "validated candidate genes"
- "robust across regions"
- "data/code are available" before final DOI or release tag

## Remaining pre-submission actions

1. Mint final DOI or release identifier for code and source data.
2. Freeze final figure panels and decide whether diagnostic Table 2 is main or supplementary.
3. Convert dataset, annotation and STRING source notes into final journal reference style.
4. Add final repository DOI to Data and Code Availability statements.
5. Run one final overclaim search after DOI/reference updates.

## Automated overclaim scan on 2026-06-14

An initial regex scan over `docs/*.md` searched for high-risk expressions such as state-of-the-art, outperforms all, robust across, breakthrough, validated candidate and DOI-backed release. The scan returned only bounded or negative-control contexts, including lines that explicitly say not to claim state-of-the-art performance, not to treat expected-score distillation as a breakthrough, and not to present gene attention as validated biology. No immediate manuscript patch was required from this scan.

False-positive contexts to preserve:

- `docs/MAIN_RESULTS_TABLE.md`: "Do not state that RiceGeneFormer outperforms all baselines."
- `docs/BIB_NC_SUBMISSION_STRATEGY.md`: "the manuscript must not claim predictive state-of-the-art."
- `docs/MANUSCRIPT_DRAFT.md`: expected-score distillation is not the main breakthrough.
- `docs/MANUSCRIPT_DRAFT.md`: gene-attention candidate-gene interpretation is audited and bounded.

Repeat this scan after DOI/reference edits and after any major rewrite of the abstract, introduction or cover-letter text.

## Recommended cover-letter angle

This manuscript is best pitched as a rigorous benchmark and diagnostic study for plant genotype-to-phenotype deep learning. Its value is that it prevents overoptimistic claims by comparing a gene-aware neural architecture against strong leakage-controlled top-SNP baselines, reporting deterministic full-role metrics, and documenting negative robustness and interpretation checks. This is aligned with BIB readership because it provides reusable evaluation infrastructure and a clear account of where current gene-aware crop neural models succeed and fail.
