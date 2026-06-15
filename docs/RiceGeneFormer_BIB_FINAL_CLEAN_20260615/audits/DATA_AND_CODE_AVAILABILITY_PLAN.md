# Data and code availability plan for RiceGeneFormer

Last updated: 2026-06-14 23:02 CST

## Purpose

This file converts the current manuscript evidence into a submission-ready data/code availability plan. It does not contain raw genotype data, phenotype data, model weights, credentials or private links. It is intended to support a BIB-style benchmark/diagnostic manuscript.

## Availability routes by artifact class

| Artifact class | Examples in this project | Access route | Planned location / identifier strategy | Notes and risk flags |
|---|---|---|---|---|
| Reused public genotype and phenotype data | 3K Rice Genome / SNP-Seek core SNP PLINK resource, 3KRG morpho-agronomic phenotype spreadsheet, accession metadata | Reused public source | Cite the 3K Rice Genome and SNP-Seek papers; list the SNP-Seek / 3KRG download page URLs in Data Availability | Do not redistribute raw 3KRG files unless the repository licence and size policy are explicitly checked. State that raw data are available from the public source. |
| Reused public annotation data | Ensembl Plants release 61 `Oryza_sativa.IRGSP-1.0.61.gff3.gz` | Reused public source | Cite Ensembl Plants and provide release-61 FTP URL | The manuscript must state release 61, not only the current live Ensembl Plants release. |
| Reused public interaction data | STRING v12.0 Oryza sativa Japonica Group taxon 39947 aliases and links files | Reused public source | Cite STRING 2023 and provide STRING v12.0 bulk download URLs or database landing page | State score threshold and pruning in Methods; do not upload raw STRING bulk files unless licence permits. |
| Lightweight processed benchmark summaries | final metrics tables, per-trait summaries, figure source CSVs, region-shift summaries, gene-attention summary TSVs, manifests | Public repository / source data | Archive a release of this GitHub repository or a separate Zenodo record; include source-data files under `docs/figure_source_data/` and selected lightweight TSV/JSON summaries | These are the most important files to make reviewer-reproducible. They are small and should be public with DOI. |
| Analysis and training code | preprocessing, baseline, RiceGeneFormer training, deterministic evaluation, gene-score export, GWAS-top graph builder | Public code repository | GitHub repository plus a versioned Zenodo DOI before submission | The current Git remote is `git@github.com:shuai19910911/PRSNet.git`. Create a release/tag for the submitted version. |
| Raw processed matrices and large local artifacts | genotype matrix, phenotype matrix, split arrays, GWAS p-value arrays, graph arrays, checkpoints, logs | Not uploaded by default; public only if size/licence allows | State that derived large files can be regenerated from public inputs using the released code; optionally deposit selected non-restricted processed artifacts if repository size and licences allow | Current project policy says GitHub does not upload data, logs or weights. Avoid claiming these are deposited until a repository record exists. |
| Model checkpoints and teacher predictions | RiceGeneFormer `checkpoint_best.pt`, teacher arrays, local output directories | Not required for paper claims if code and summary outputs are public; optional repository deposit | If deposited, use Zenodo/Figshare/OSF with clear licence and file manifest | Checkpoints are large and may not be necessary for a BIB benchmark if deterministic metrics and code are available. |

## Files that should be included in a lightweight public release

Recommended minimum public release contents:

- `README.md`
- `docs/MANUSCRIPT_DRAFT.md`
- `docs/MAIN_RESULTS_TABLE.md`
- `docs/SUPPLEMENTARY_PER_TRAIT_RESULTS.md`
- `docs/SUPPLEMENTARY_PER_TRAIT_RESULTS.tsv`
- `docs/CORE_TRAIT_CLASS_DISTRIBUTION.tsv`
- `docs/FINAL_PER_TRAIT_METRICS_RGF_W010_TEST.tsv`
- `docs/FIGURE_LEGENDS.md`
- `docs/DATA_SOURCES_AND_CITATIONS.md`
- `docs/DATA_AND_CODE_AVAILABILITY_PLAN.md`
- all CSV files in `docs/figure_source_data/`
- `docs/REGION_SHIFT_BENCHMARK_RESULTS.md`
- `docs/GENE_INTERPRETATION_RESULTS.md`
- the code under `scripts/` needed to regenerate preprocessing, baselines, training, deterministic evaluation and audits

Recommended optional lightweight local outputs to archive or convert into source-data tables:

- `data/3krice/processed/region_shift_baseline_summary_region_manual_20260614_213808.tsv`
- `data/3krice/processed/region_shift_rice_geneformer_summary_region_rgf_manual_20260614_214252.tsv`
- `data/3krice/processed/gene_attention_w010_test_aggregate/gene_attention_top200_aggregate.tsv`
- `data/3krice/processed/gene_attention_w010_test_aggregate/gene_attention_aggregate_manifest.json`
- `data/3krice/processed/gene_attention_known_gene_overlap_w010_test/known_gene_overlap_manifest.json`
- `data/3krice/processed/rice_geneformer_gwas_top2048_seed42_e20_s100_gpu2/test_eval/evaluation_manifest.json`

Current packaged source-data directory:

- `docs/source_data/README.md`
- `docs/source_data/SOURCE_DATA_MANIFEST.json`
- `docs/source_data/region_shift_baseline_summary.tsv`
- `docs/source_data/region_shift_rice_geneformer_summary.tsv`
- `docs/source_data/gene_attention_w010_top200_aggregate.tsv`
- `docs/source_data/gene_attention_w010_aggregate_manifest.json`
- `docs/source_data/known_gene_overlap_manifest.json`
- `docs/source_data/gwas_top2048_seed42_evaluation_manifest.json`
- `docs/source_data/gwas_top2048_graph_manifest.json`

Before public release, copy any selected local `data/` summaries into a clean `docs/source_data/` or `supplementary_data/` directory and exclude raw matrices, p-value arrays, checkpoints and logs.

## Draft Data Availability statement

Data Availability

The raw genotype, phenotype and accession metadata used in this study are publicly available from the 3K Rice Genome / SNP-Seek data resources. The primary genotype input was the 3K core SNP PLINK resource, and phenotype labels were obtained from the 3KRG morpho-agronomic phenotype spreadsheet. Rice gene coordinates were obtained from Ensembl Plants release 61 for the IRGSP-1.0 assembly (`Oryza_sativa.IRGSP-1.0.61.gff3.gz`). Protein-association data used for the STRING graph were obtained from STRING v12.0 for Oryza sativa Japonica Group (taxonomy ID 39947). Source URLs and citation anchors are recorded in `docs/DATA_SOURCES_AND_CITATIONS.md`.

The source data underlying the manuscript figures, final performance tables, cross-region benchmark summaries and gene-interpretation audit summaries will be deposited in a public versioned repository before submission. The current working repository is `git@github.com:shuai19910911/PRSNet.git`; a permanent DOI or release identifier has not yet been assigned. Large derived matrices, intermediate GWAS p-value arrays, model checkpoints and job logs are not included in the manuscript repository by default because they can be regenerated from the public input data and released code, and because the project policy avoids redistributing large raw or derivative data files without final licence review.

## Draft Code Availability statement

Code Availability

The code used for data preprocessing, train-fold GWAS prior construction, baseline training, RiceGeneFormer training, deterministic evaluation, cross-region benchmarking and gene-attention audits will be released in a versioned public repository before submission. The repository will include a README, environment notes, scripts needed to reproduce the reported benchmark summaries and lightweight source-data files for the manuscript figures and tables. A Zenodo, Figshare, OSF or equivalent DOI-backed archive should be created for the exact submitted version.

## Repository and citation actions

1. Decide whether the manuscript release will be a GitHub release archived by Zenodo, or a separate Zenodo/Figshare/OSF dataset record.
2. Create a clean release directory that excludes raw data, model weights, logs, credentials and temporary job outputs.
3. Keep `docs/source_data/` synchronized with final manuscript source-data needs.
4. Finalize the README/data dictionary for every public TSV/CSV/JSON file, including columns, units, metric definitions and provenance.
5. Create a release tag and DOI-backed archive after the manuscript files are frozen.
6. Replace the placeholder availability wording with the final DOI/accession.

## Missing information / risk flags

- No final DOI or release identifier exists yet.
- The target journal has not been finalized, so exact repository and source-data formatting may still change.
- Raw 3KRG redistribution should not be claimed unless the data-use terms and repository policy are explicitly checked.
- Candidate-gene IDs in the preliminary gene-attention audit are not yet final citation-verified and should not be treated as a curated biological resource.
- If model checkpoints are not deposited, the manuscript should clearly state that reported metrics are supported by source-data tables and that checkpoints can be regenerated with released code and public inputs.

## 中文核对

- 当前不能写“所有数据已上传 DOI 仓库”，因为还没有 DOI。
- 可以写“原始 3K Rice/SNP-Seek、Ensembl、STRING 数据来自公开数据库；本文 source data 和代码将在投稿前通过带 DOI 的版本仓库公开”。
- 不建议上传 raw genotype/phenotype、checkpoint、日志和大矩阵；建议只公开轻量 source data、结果表、脚本和文档。
- 投稿前必须补：最终 GitHub release/Zenodo DOI、source-data README、文件清单和 licence。
