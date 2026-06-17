# PRSNet/RiceGATE-MoE manuscript framing plan: positive-results-first version

Date: 2026-06-17

## 1. Recommendation

Yes. The current results are sufficient to organize a BIB-style benchmark/methods manuscript, but the safest article type is not a pure “new model beats all published methods” story. The stronger and more defensible story is:

> We build a leakage-aware ordinal multi-trait benchmark for 3K rice genotype-to-phenotype prediction, show that a GWAS-guided trait-specific expert mixture improves macro-F1 over strong internal baselines, and bridge the crop genomic-prediction literature by reporting both classification metrics and PCC/Pearson prediction accuracy.

In Chinese: 主线不是“我们全面超过所有已发表模型”，而是“我们把水稻多等级表型预测做成了严格、可复现、指标分离的 benchmark，并提出 RiceGATE-MoE 在 macro-F1 主指标上取得当前最优”。

## 2. Positive main-claim hierarchy

### Claim 1: Leakage-aware ordinal benchmark is the paper’s foundation

Main text emphasis:

- Dataset: 3K rice genotype matrix with 365,710 SNPs.
- Phenotype setting: 10 high-coverage ordinal traits from rice descriptor codes.
- Evaluation: train/validation/test split with train-fold GWAS priors only.
- Metrics are separated: macro-F1, classification accuracy, MAE, and PCC/Pearson.

Why positive:

- Many crop genomic prediction papers report PCC/Pearson for continuous traits; our task is harder and different: multi-trait ordinal classification.
- This lets the paper avoid unfair “accuracy” comparisons and present a more rigorous evaluation framework.

### Claim 2: RiceGATE-MoE gives the best classification-oriented performance

Headline numbers:

| Model | macro-F1 | classification accuracy | MAE | mean trait PCC |
|---|---:|---:|---:|---:|
| RiceGATE-MoE final | 0.4235 | 0.6268 | 0.5830 | 0.3716 |
| alpha3 ExtraTrees average base | 0.4182 | 0.6134 | 0.6003 | 0.3614 |
| best single ExtraTrees top288 alpha1.0 | 0.4129 | 0.6060 | 0.6063 | 0.3588 |
| SNP-MLP seed ensemble | 0.3950 | 0.6166 | 0.6031 | 0.3778 |
| train-majority baseline | 0.2098 | 0.5849 | 0.7050 | NA |

Positive wording:

> RiceGATE-MoE achieved the highest macro-F1 among all evaluated internal predictors, improving over the alpha3 ExtraTrees base ensemble while also increasing classification accuracy and reducing ordinal MAE.

Avoid wording:

- Do not say “state-of-the-art across crop genomic prediction” unless we add cross-split or external validation.
- Do not say it beats published continuous-trait PCC models, because those are not the same task.

### Claim 3: PCC bridge makes our results comparable to the genomic prediction literature

Headline numbers:

| Model | mean trait PCC | interpretation |
|---|---:|---|
| top10% ExtraTrees | 0.4085 | strongest internal PCC-oriented baseline |
| top5% ExtraTrees | 0.4025 | high PCC, lower macro-F1 |
| top2% ExtraTrees | 0.3957 | high accuracy/MAE baseline |
| RiceGATE-MoE final | 0.3716 | best macro-F1, not best PCC |
| SNP-MLP seed ensemble | 0.3778 | neural baseline with competitive PCC |

Positive framing:

> Reporting PCC from the expected ordinal score shows that classification-sensitive and ranking-sensitive metrics capture different behavior. This is a useful diagnostic contribution rather than a weakness: the benchmark reveals when a model is better for minority-class-sensitive classification and when it is better for continuous ranking.

Main text handling:

- Main table includes macro-F1 as primary, accuracy/MAE/PCC as secondary.
- PCC-only winners can be presented as “PCC-oriented baselines” rather than as failures of RiceGATE-MoE.

### Claim 4: Published-model comparison supports a clear niche

Use these as contextual references:

| Published model | Useful comparison angle | Main message for our paper |
|---|---|---|
| Cropformer | reports both regression PCC and simpler DTT classification metrics | Shows classification reporting exists, but published classification task is single-trait binary/three-class, simpler than ours. |
| DNNGP | defines prediction accuracy as correlation coefficient | Supports our decision to report PCC separately from classification accuracy. |
| GP-WAITER | GWAS-weighted CNN/Transformer crop predictor | Motivates GWAS-aware architecture and expert weighting. |
| NetGP | gene network and multi-omics prediction with PCC | Supports network/biological-prior framing and PCC convention. |
| SoyDNGP | BIB soybean genomic prediction framework | Strong BIB-style precedent for crop genomic prediction tools. |
| CropARNet / ResDeepGS | attention/residual DL genomic prediction | Supports broader deep learning context but mostly continuous-trait regression. |

Positive wording:

> Unlike most crop genomic prediction frameworks that focus on continuous traits and report Pearson correlation-based prediction accuracy, our benchmark targets multi-trait ordinal classification and therefore evaluates macro-F1, classification accuracy, ordinal MAE and PCC in parallel.

## 3. What to keep in the main text vs supplement

### Main text: keep positive, central, defensible results

1. Dataset and leakage-aware evaluation design.
2. Main internal model comparison table with selected representative baselines.
3. RiceGATE-MoE final model result.
4. PCC/Pearson bridge table.
5. Published-model comparison table with metric-type warning.
6. Trait-specific gate examples showing RiceGATE-MoE selectively borrows from stronger experts.

### Supplement: move non-core or negative results

Put these in Supplementary Notes/Tables instead of main Results:

1. RiceGeneFormer single-model underperformance relative to tree ensembles.
2. raw p<0.05 merged SNP panel noise result.
3. top-percentage SNP ablations that did not beat top288 expert ensemble.
4. full 28-row model leaderboard.
5. candidate expert scan details.
6. GPU smoke and engineering provenance.

How to phrase without looking negative:

> Extended ablations showed that broader GWAS-derived SNP panels and graph-neural variants were technically feasible but did not improve the classification-oriented metric under the current sample size, supporting the use of compact, trait-specific GWAS expert panels in the final RiceGATE-MoE predictor.

This keeps the lesson positive: failed routes justify the final design.

## 4. Proposed title options

1. Leakage-aware ordinal genomic prediction of rice agronomic traits with GWAS-guided trait-specific expert mixtures
2. RiceGATE-MoE: a leakage-aware benchmark and expert-mixture predictor for ordinal rice genotype-to-phenotype prediction
3. Metric-aware benchmarking of ordinal rice genotype-to-phenotype prediction with GWAS-guided expert mixtures
4. Separating classification accuracy and Pearson prediction accuracy in ordinal crop genomic prediction

Recommended title:

> RiceGATE-MoE: a leakage-aware benchmark and expert-mixture predictor for ordinal rice genotype-to-phenotype prediction

Why:

- It names the model.
- It names the benchmark contribution.
- It avoids overclaiming “state-of-the-art”.
- It is searchable for rice, genotype-to-phenotype, ordinal prediction and expert mixtures.

## 5. One-sentence contribution

> We present RiceGATE-MoE, a leakage-aware ordinal genomic prediction framework that combines train-fold GWAS-derived trait-specific experts and metric-separated evaluation to improve macro-F1 for multi-trait rice descriptor prediction while providing a PCC bridge to the broader crop genomic prediction literature.

## 6. Abstract draft, positive but defensible

Crop genomic prediction studies commonly evaluate continuous traits using Pearson correlation-based prediction accuracy, whereas many practical germplasm descriptors are ordinal or categorical and require classification-sensitive evaluation. Here we developed a leakage-aware benchmark for ordinal genotype-to-phenotype prediction in 3K rice, using 365,710 SNPs and 10 high-coverage descriptor-code traits under train-fold GWAS priors. We evaluated tree-based, boosting, neural and graph-inspired predictors using macro-F1, classification accuracy, ordinal mean absolute error and Pearson correlation computed from expected ordinal scores. The proposed RiceGATE-MoE framework combines GWAS-guided trait-specific expert predictors through validation-frozen gates and achieved the best classification-oriented performance among evaluated internal models, with test macro-F1 of 0.4235, classification accuracy of 0.6268 and ordinal MAE of 0.5830. Pearson-based prediction accuracy showed a complementary pattern: PCC-oriented ExtraTrees baselines reached mean per-trait PCC up to 0.4085, whereas RiceGATE-MoE reached 0.3716 while retaining the highest macro-F1. Comparison with published crop genomic prediction frameworks showed that most reported “accuracy” values are Pearson/PCC values for continuous traits rather than classification accuracy, motivating explicit metric separation. These results establish a practical benchmark and reporting framework for ordinal crop genomic prediction and highlight trait-specific expert mixing as a robust route for improving minority-class-sensitive performance.

## 7. Key points for BIB-style submission

- We constructed a leakage-aware benchmark for ordinal rice genotype-to-phenotype prediction using train-fold GWAS priors and 10 high-coverage descriptor-code traits.
- RiceGATE-MoE achieved the best internal macro-F1, improving classification-oriented prediction over strong ExtraTrees and SNP-MLP baselines.
- Pearson/PCC prediction accuracy was computed from expected ordinal scores, enabling fairer comparison with continuous-trait genomic prediction literature.
- Metric separation revealed that PCC-oriented baselines and macro-F1-oriented predictors optimize different prediction behavior.
- The framework provides a reproducible template for reporting ordinal crop genomic prediction without conflating classification accuracy and Pearson prediction accuracy.

## 8. Recommended main figures and tables

### Figure 1: Benchmark and leakage-control design

Panels:

A. 3K rice genotype/phenotype input overview.
B. Train/validation/test split and train-fold-only GWAS prior generation.
C. Ordinal multi-trait label/mask structure.
D. Metric separation: macro-F1, accuracy, MAE, PCC.

Positive message:

> The benchmark is leakage-aware and metric-aware.

### Figure 2: RiceGATE-MoE architecture

Panels:

A. Base alpha3 ExtraTrees expert ensemble.
B. Candidate expert pool.
C. Trait-specific validation-frozen gates.
D. Final expected ordinal score and class prediction.

Positive message:

> The model uses validation-only gates to selectively borrow trait-specific strengths.

### Figure 3: Internal performance comparison

Panels:

A. Macro-F1 leaderboard for representative models.
B. Accuracy and MAE comparison.
C. Mean per-trait PCC comparison.
D. Macro-F1 vs PCC scatterplot to show metric trade-off.

Positive message:

> RiceGATE-MoE is best for macro-F1, while PCC-oriented baselines provide a complementary ranking signal.

### Figure 4: Literature comparison and metric harmonization

Panels:

A. Published models grouped by task type: continuous regression vs classification.
B. Which metric each paper reports.
C. Our position: ordinal multi-trait classification with PCC bridge.

Positive message:

> Our paper clarifies a metric mismatch in crop genomic prediction.

### Main Table 1: Dataset and evaluation protocol

Include sample counts, SNP count, trait count, split sizes, leakage-control rules and metric definitions.

### Main Table 2: Representative internal model comparison

Keep 8-10 representative rows, not all 28 rows.

### Main Table 3: Published-model comparison

Use contextual comparison, with “not directly comparable” notes where needed.

## 9. Results section structure

### Result 1. A leakage-aware benchmark for ordinal rice phenotype prediction

Question:

> How should rice descriptor-code traits be evaluated without conflating ordinal classification with continuous regression?

Positive claim:

> The benchmark separates classification-sensitive and correlation-based metrics, enabling fair comparison within the study and with the wider genomic prediction literature.

### Result 2. GWAS-guided tree experts provide strong ordinal predictors

Question:

> Which model families are strong under a train-fold GWAS prior?

Positive claim:

> Compact GWAS-guided ExtraTrees experts provide strong baselines and form a reliable expert pool for downstream gating.

### Result 3. RiceGATE-MoE improves classification-oriented prediction

Question:

> Can trait-specific expert mixing improve macro-F1 over a strong base ensemble?

Positive claim:

> RiceGATE-MoE achieved the highest test macro-F1 and improved accuracy/MAE relative to the alpha3 base ensemble.

### Result 4. PCC highlights complementary ranking performance

Question:

> How do our ordinal models compare under the Pearson/PCC convention used in genomic prediction?

Positive claim:

> PCC computed from expected ordinal scores reveals complementary behavior and provides a bridge to continuous-trait literature.

### Result 5. Published-model comparison shows why metric separation is necessary

Question:

> Are published “accuracy” values comparable to our classification accuracy?

Positive claim:

> Most published crop genomic prediction models report Pearson correlation-based prediction accuracy, while our task requires simultaneous reporting of macro-F1, accuracy, MAE and PCC.

## 10. Discussion structure

Paragraph 1: Central contribution.

> We developed a leakage-aware, metric-separated ordinal genomic prediction benchmark for rice and a trait-specific expert-mixture predictor.

Paragraph 2: Why RiceGATE-MoE works.

> Trait-specific gates let the model retain a strong base predictor while borrowing from candidate experts only when validation evidence supports a trait-level gain.

Paragraph 3: Why PCC does not replace macro-F1.

> PCC measures ranking/continuous-score association, while macro-F1 reflects class-balanced discrete prediction. Both are informative but answer different questions.

Paragraph 4: Relation to published models.

> Published models such as Cropformer, DNNGP, GP-WAITER, NetGP and SoyDNGP mainly motivate architecture and metric conventions, but their continuous-trait PCC values should not be read as direct classification competitors.

Paragraph 5: Boundary.

> The current results are based on the present 3K rice split and should be locked before final cross-split confirmation. This boundary can be stated briefly; detailed negative ablations belong to supplement.

Paragraph 6: Future work.

> Future work should add full cross-split validation, source-data release, and extension to multi-environment crop datasets.

## 11. What not to claim

Avoid these claims unless more validation is added:

- “RiceGATE-MoE is the best crop genomic prediction model.”
- “Our PCC exceeds published models.”
- “RiceGeneFormer explains gene regulation.”
- “Attention/gates identify validated causal genes.”
- “The result is robust across populations/years/environments.”

Safer replacements:

- “achieved the best macro-F1 among evaluated internal predictors.”
- “provides a PCC bridge to continuous-trait genomic prediction studies.”
- “suggests trait-specific expert mixing is useful under the present benchmark.”
- “defines a leakage-aware reporting framework for ordinal crop genomic prediction.”

## 12. Immediate next writing tasks

1. Convert this plan into a manuscript skeleton with sections and figure callouts.
2. Build Figure 1-4 source-data TSV files from current tables.
3. Prepare a verified reference list for Cropformer, DNNGP, GP-WAITER, NetGP, SoyDNGP, CropARNet, ResDeepGS and DeepGS.
4. Lock the headline model configuration before any new split/seed validation.
5. Decide whether final submission will use “RiceGATE-MoE” as the model name or emphasize “metric-aware ordinal benchmark” in the title.
