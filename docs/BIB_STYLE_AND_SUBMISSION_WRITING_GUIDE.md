# BIB style and submission writing guide for RiceGeneFormer

Last updated: 2026-06-15

Purpose: translate the current Briefings in Bioinformatics (BIB) author instructions and recent BIB benchmark/methods article patterns into concrete writing requirements for the RiceGeneFormer manuscript. This file is a writing standard for turning `docs/MANUSCRIPT_DRAFT.md` into a near-submission manuscript.

## Sources checked

### Official / near-official sources

1. Oxford Academic BIB author guidelines: `https://academic.oup.com/bib/pages/author-guidelines`
2. Oxford Academic BIB manuscript preparation page: `https://academic.oup.com/bib/pages/msprep_submission`
3. DOAJ BIB journal metadata page: `https://doaj.org/toc/1477-4054`

Key official points extracted:

- BIB is peer-reviewed and fully open access.
- Scope: bioinformatics and computational biology with a clear molecular foundation; it serves life-science users who need awareness of data sources, analytical tools, computational methods and interpretation strategies.
- It explicitly covers genomics, phenomics, systems biology, network biology, computational biology and molecular-basis clinical/medical informatics.
- Submission is through Manuscript Central.
- Review model is single-anonymized; manuscripts passing editorial screening are typically sent to three reviewers.
- The journal receives more than 2,000 submissions per year and publishes less than one quarter.
- Average first decision is around 20 days; the journal aims for decisions within about two months where possible.
- Format-free submission is allowed, but manuscripts must be formatted at revision/acceptance.
- Manuscripts exceeding suggested word limits by 1,000 words or more are usually returned immediately.
- Title page, short abstract, up to 5 key points and up to 6 keywords are expected for review/software-style articles.
- Authors should avoid slang and marketing language; the journal should not be used to market a product or service.
- BIB follows scientific/professional style conventions, including CSE-style terminology discipline, SI units and correct genotype/phenotype typography.

### Recent BIB benchmark/review article patterns checked

1. Jia, Hu and Zhao. `Benchmark of embedding-based methods for accurate and transferable prediction of drug response`. Briefings in Bioinformatics 24(3), bbad098, 2023. DOI: 10.1093/bib/bbad098.
2. Sherkatghanad et al. `Using traditional machine learning and deep learning methods for on- and off-target prediction in CRISPR/Cas9: a review`. Briefings in Bioinformatics 24(3), bbad131, 2023. DOI: 10.1093/bib/bbad131.
3. Paltun, Kaski and Mamitsuka. `Machine learning approaches for drug combination therapies`. Briefings in Bioinformatics 22(6), bbab293, 2021. DOI: 10.1093/bib/bbab293.
4. Additional search results showed recent BIB content around benchmarking computational methods, deep learning frameworks and genomic prediction tools.

Observed article-style patterns:

- BIB accepts long, detailed, method-heavy manuscripts when the manuscript teaches a reusable computational lesson.
- Articles use clear section headings and often uppercase or title-style major section headings.
- Abstracts are compact but information-rich, typically one paragraph or a short block that moves from biological/computational problem to method/evaluation to practical resource.
- Key points are common and should be 3-5 bullets, each a concrete contribution rather than a generic impact claim.
- Benchmark/methods articles are expected to describe data sources, preprocessing, feature construction, model classes, evaluation protocols and public availability in enough detail for reuse.
- Strong baselines are central. BIB readers expect comparisons with well-performing traditional or simpler methods such as SVM, random forest, XGBoost, Elastic Net, DNN or other task-specific baselines.
- Negative or bounded findings are acceptable if they clarify a reusable methodological lesson.
- Public web servers, code repositories or curated benchmark data strengthen the paper; when not available yet, availability language must remain future-tense until DOI/release exists.

## BIB writing style rules for this manuscript

### Overall tone

Use a practical, expert, benchmark-oriented tone.

Good style:

- precise, technical, reproducible
- honest about negative results
- comparative rather than promotional
- useful to non-specialist life scientists and computational practitioners
- explicit about data sources, models and evaluation conditions

Avoid:

- Nature Communications-style broad biological discovery rhetoric
- claiming model superiority when balanced SNP-MLP is stronger
- marketing words such as `powerful`, `revolutionary`, `state-of-the-art`, `breakthrough`
- vague terms such as `robust` unless tied to exact tests and results
- hiding failure cases in the supplement

### Argument shape

The whole paper should follow this chain:

`field need -> benchmark leakage problem -> ordinal/imbalanced phenotype challenge -> gene-aware method -> fair comparison with strong top-SNP baselines -> diagnostic boundary findings -> reusable benchmark/source-data contribution`

One-sentence argument:

In 3K rice genotype-to-phenotype prediction, RiceGeneFormer shows that a leakage-aware gene-level ordinal framework can approach but not surpass strong train-fold top-SNP baselines, supported by deterministic test metrics, ablations, cross-region checks and gene-attention audits, with claims bounded to benchmarking and diagnostics rather than robustness or biological discovery.

## Target manuscript structure and word budget

The official BIB pages warn against exceeding suggested word limits, but the exact article-type word caps were not fully exposed in the accessible extraction. Therefore, use a conservative, submission-safe target of approximately 7,000-8,000 main-text words excluding references, tables, figure legends and supplementary material. If the final submission system shows a stricter article-type limit, compress to that limit.

Recommended full manuscript budget for RiceGeneFormer:

| Component | Target length | Purpose | Detail level |
|---|---:|---|---|
| Title | 12-16 words | Searchable benchmark claim | Include rice, genotype-to-phenotype, gene-aware or ordinal benchmark; avoid overclaiming. |
| Abstract | 250-300 words | Mini-paper | Include problem, leakage/ordinal challenge, method, strongest metrics, boundary, contribution. Current 270-word draft fits. |
| Key points | 4 bullets, 20-30 words each | Editor/reviewer triage | Each bullet must be a concrete contribution or finding. |
| Keywords | 6-8 terms | Indexing | Include genotype-to-phenotype, rice, ordinal classification, GWAS, benchmark, macro-F1. |
| Introduction | 900-1,200 words | Funnel into benchmark gap | 5-6 paragraphs: breeding/genomic prediction context, ordinal phenotypes, leakage/fair comparison, gene-aware modeling gap, study response, contribution/boundary. |
| Results | 2,500-3,200 words | Evidence ladder | 6-7 subsections matching current Result 1-7; each begins with the tested question and ends with boundary. |
| Methods | 2,000-2,800 words | Reproducibility | Detailed enough to reimplement preprocessing, splits, GWAS priors, model, baselines, evaluation, audits. Put exhaustive hyperparameter tables in supplement if too long. |
| Discussion | 900-1,200 words | Meaning and limitations | Main message, why strong SNP baselines matter, interpretation of negative controls, class-balance trade-off, limitations, future work. |
| Data and Code Availability | 180-300 words before DOI; 120-220 after DOI | Access mapping | Must state raw public sources, code/source-data release, exclusions, DOI placeholder/final DOI. |
| Figure legends | 100-180 words per main figure, up to 220 for complex diagnostic Figure 4 | Panel mapping | Short legends go in manuscript; detailed source paths remain in source-data docs. |
| Tables | 1 main performance table + 1 diagnostic table or supplement | Evidence compression | Table 1 main; Table 2 can stay main only if space permits. |
| Supplementary information | unlimited practical detail | Reproducibility | Per-trait metrics, class distributions, hyperparameters, source-data dictionary, additional audits. |

## Section-by-section writing standard

### Title

Target title:

`A leakage-aware ordinal benchmark for gene-aware genotype-to-phenotype prediction in 3K rice genomes`

Why it fits BIB:

- concrete task: genotype-to-phenotype prediction
- biological system: 3K rice genomes
- computational framing: leakage-aware ordinal benchmark
- avoids unsupported superiority

Do not use titles implying performance victory or biological discovery.

### Abstract

Use the current 270-word abstract as the base. It is already BIB-compatible because it contains:

1. Problem: dense genomic panels and ordinal imbalanced phenotypes.
2. Evaluation gap: leakage from GWAS/model-selection procedures.
3. Method: RiceGeneFormer-OMTL components.
4. Quantitative result: macro-F1 values against LightGBM and SNP-MLP.
5. Boundary: no robustness or biological discovery claim.
6. Contribution: reproducible diagnostic framework.

Final polish target:

- Keep 250-300 words unless the submission system specifies otherwise.
- One paragraph is acceptable; two short paragraphs may improve readability.
- Include only one or two headline metric comparisons.
- Keep the negative result explicit: `remained below balanced SNP-MLP baselines`.

### Key points

BIB-style key points should be concrete, not promotional. Use 4 bullets:

1. Method/framework contribution.
2. Fair benchmark/evaluation contribution.
3. Main performance comparison.
4. Diagnostic boundary result.

Avoid generic bullets such as `This study is important for crop breeding` unless tied to a concrete reusable artifact.

### Introduction

Target: 5-6 paragraphs, 900-1,200 words.

Required paragraphs:

1. Field need: genomic prediction in breeding and public rice panels.
2. Data/label challenge: descriptor-code phenotypes are ordinal, sparse and imbalanced.
3. Evaluation challenge: GWAS priors and top-SNP selection can leak information; macro-F1 is needed with accuracy/MAE.
4. Modeling gap: marker baselines are strong but biologically unstructured; gene-aware models are attractive but can overclaim.
5. Present study: RiceGeneFormer-OMTL design and leakage-aware benchmark.
6. Contribution and boundary: benchmark/diagnostic framework; not a universal replacement for SNP baselines.

Detail standard:

- Cite 3K Rice/SNP-Seek, genomic prediction, GWAS/top-SNP baselines, ordinal classification and BIB-style ML benchmark references.
- Do not list many papers without a narrowing logic.
- End with what the paper demonstrates and what it does not demonstrate.

### Results

Target: 6-7 subsections, 2,500-3,200 words.

Use current Result 1-7 but expand to BIB submission depth:

1. Benchmark construction.
   - Must report accessions, SNPs, traits, split sizes, train-fold GWAS rule.
   - Explain why macro-F1, accuracy and MAE are jointly needed.

2. Model design.
   - Explain each component by motivation: GWAS priors, SNP-to-gene aggregation, graph tokens, trait queries, top-SNP fusion, ordinal heads.
   - Avoid architectural hype; make design readable to computational biologists.

3. Strong baselines.
   - Put LightGBM, XGBoost and SNP-MLP before RiceGeneFormer interpretation.
   - This matches BIB benchmark norms: establish strong reference before claiming value.

4. Final RiceGeneFormer configuration.
   - Report no-distillation and w=0.10 test metrics.
   - Explain why close-to-LightGBM but below SNP-MLP is still informative.

5. Distillation.
   - Present as weak positive / non-monotonic.
   - Do not call it a breakthrough.

6. Negative controls / ablations.
   - Graph identity, mapping windows and calibration.
   - BIB readers accept this if framed as diagnostic methodology.

7. Cross-region and gene-attention audits.
   - Keep this in main Results because it prevents overclaiming.
   - Explicitly state that robustness and biological validation are not supported.

Per-subsection pattern:

`Question -> protocol -> result -> comparison -> interpretation boundary`

### Methods

Target: 2,000-2,800 words main Methods plus supplementary tables.

Required Methods subsections:

1. Data sources and sample alignment.
2. Phenotype encoding and core ordinal trait selection.
3. Split construction and leakage control.
4. Train-fold GWAS prior generation.
5. SNP-to-gene mapping and gene graph construction.
6. RiceGeneFormer-OMTL architecture.
7. Training, checkpoint selection and deterministic evaluation.
8. Baselines.
9. Distillation.
10. Cross-region evaluation.
11. Gene-attention export and GWAS-top2048 audit.
12. Metrics and statistical reporting.
13. Code/data availability and source-data packaging.

Detail standard:

- Every number needed to reproduce the benchmark must appear in Methods or supplement.
- Every split-specific rule must state that validation/test labels were not used for GWAS or feature selection.
- Baseline hyperparameters must be described enough for reviewer reproduction.
- If exact command lines are too long, provide a supplementary command table or repository scripts.

### Discussion

Target: 900-1,200 words.

Required movement:

1. Main contribution: leakage-aware, gene-aware, ordinal benchmark.
2. Why strong SNP baselines matter.
3. Why negative graph/mapping/attention findings are useful, not fatal.
4. Class imbalance and metric trade-off.
5. Limitations.
6. Future work tied to evidence boundary.

Tone:

- BIB Discussion should teach a reusable computational lesson.
- Do not simply restate Results.
- Treat the model-performance gap as a central insight, not an embarrassment.

### Data and Code Availability

Current status: future-tense only until DOI exists.

Required final statement must include:

- Raw public data sources: 3K Rice/SNP-Seek, Ensembl Plants release 61, STRING v12.0.
- Released code repository and exact release DOI.
- Released lightweight source data directory.
- Exclusions: raw genotype/phenotype, large matrices, p-value arrays, checkpoints, logs unless deliberately deposited.
- Regeneration route from public inputs and released code.

Do not write `data/code are available` until the DOI/release tag exists.

## Figure and table strategy

### Main figures

Use 4 main figures:

1. Leakage-aware benchmark construction.
2. RiceGeneFormer architecture and design priorities.
3. Final performance and trait-level failure modes.
4. Ablation, robustness and interpretation diagnostics.

This fits BIB because it moves from resource construction to method to benchmark result to diagnostic boundary.

### Supplementary figures/tables

Use supplementary material for:

- per-trait method heatmap
- full per-trait metrics
- class distributions
- hyperparameters
- command/provenance table
- source-data manifest
- additional ablations if too dense for main text

### Tables

Table 1 should remain main text: final model/baseline performance.

Table 2 decision:

- Main text if the manuscript emphasizes diagnostic benchmark contribution.
- Supplement if space is tight, but the Result 7 text must still mention region-shift and gene-attention boundaries clearly.

## Citation strategy for BIB-ready draft

Reference categories needed:

1. 3K Rice Genome / SNP-Seek data source.
2. Rice phenotype / morpho-agronomic descriptor source.
3. Genomic prediction and crop G2P baseline literature.
4. GWAS/top-SNP and leakage/fair benchmarking references.
5. Ordinal classification / cumulative-link modeling references.
6. Multi-task learning / transformer or attention references if used.
7. STRING and Ensembl Plants references.
8. BIB/related benchmark ML papers, including the drug-response benchmark and CRISPR benchmark/review.

Target reference count: approximately 45-70 for a full BIB methods/benchmark paper. Keep references focused and real; verify DOI before final insertion.

## RiceGeneFormer-specific rewrite checklist

Before claiming the paper is submission-ready:

- [ ] Main text is 7,000-8,000 words or under final BIB article-type limit.
- [ ] Abstract is 250-300 words and includes explicit negative boundary.
- [ ] Up to 5 key points and up to 6-8 keywords are present.
- [ ] Introduction ends with benchmark/diagnostic contribution, not model superiority.
- [ ] Results start each subsection with the tested question.
- [ ] Table 1 appears before or near the primary performance text.
- [ ] Cross-region and gene-attention audits remain visible in main text.
- [ ] Methods state train-fold-only GWAS/top-SNP selection in every relevant subsection.
- [ ] Baseline hyperparameters and seeds are specified in main or supplement.
- [ ] Data/code availability contains final DOI or explicitly says DOI pending.
- [ ] No raw data, checkpoints or logs are included in public source-data directory.
- [ ] Overclaim scan passes after all edits.

## Immediate implications for current draft

Current strengths:

- Title and abstract already match BIB benchmark/diagnostic style.
- Source-data and availability plan are unusually well prepared for the current writing stage.
- Reviewer-risk audit already controls the strongest rejection risks.
- The manuscript honestly presents the SNP-MLP gap and negative robustness/interpretation audits.

Current gaps before a fully submission-ready manuscript:

1. Introduction needs final literature-supported prose and citations rather than only paragraph jobs.
2. Results need expansion from compact draft to polished BIB evidence-ladder prose.
3. Methods need more reproducibility detail and likely a supplementary hyperparameter/provenance table.
4. References must be verified and inserted in BIB/OUP style.
5. DOI-backed code/source-data release remains the largest non-writing blocker.

## English style rules for final rewrite

- Prefer short topic sentences.
- Use past tense for performed experiments and present tense for stable conclusions.
- Use `macro-F1` consistently, with a definition at first use.
- Use `test role`, `validation role`, and `held-out region` consistently; avoid vague `performance set` wording.
- Define abbreviations at first use: GWAS, SNP, MAE, STRING, OMTL.
- Avoid em dash-heavy prose.
- Do not use `leverage` as a filler verb; use `use`, `combine`, `integrate`, or `test`.
- Every paragraph must either set up a problem, report evidence, compare models, or define a boundary.

## 中文执行说明

这份指南的结论是：BIB 不是要求把故事写成 Nature-style biological discovery，而是要求“对生物信息学用户有实用价值、可复现、讲清楚数据/工具/方法/解释边界”。因此 RiceGeneFormer 当前最优写法不是掩盖 SNP-MLP 更强，而是把它写成一个严格 benchmark 和 diagnostic framework：告诉读者 gene-aware neural model 在 3K rice ordinal phenotype 上哪里接近强 baseline、哪里失败、为什么这些失败对后续方法设计有价值。
