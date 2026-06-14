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

Plant breeding increasingly depends on the ability to predict agronomic and morphological phenotypes from dense genomic variation. In crops such as rice, large genotype panels now provide hundreds of thousands to millions of markers across thousands of accessions, creating opportunities to move beyond single-locus association tests toward multi-trait predictive models. The 3K Rice Genome resources are especially useful for this purpose because they combine broad germplasm diversity with public genotype, phenotype and variety information.

### Paragraph 2: bottleneck

Many crop phenotypes available at panel scale are not continuous measurements but descriptor codes. These labels are often ordinal, sparse across accessions and strongly imbalanced across classes. Treating them as ordinary regression targets ignores their ordered class structure, whereas treating them as flat classification targets can overstate performance when majority classes dominate. A useful genotype-to-phenotype benchmark therefore needs both ordinal-aware objectives and metrics such as macro-F1 that expose minority-class behavior.

### Paragraph 3: modeling gap

Marker-based models remain difficult to beat because train-fold GWAS top SNPs can capture direct predictive shortcuts. However, these models do not explicitly organize variants by genes, pathways or trait-specific gene modules, limiting their use as gene-aware modeling frameworks. Gene-level neural architectures offer a route to combine marker information with biological structure, but they introduce their own risks. In particular, genome-wide association priors, SNP-to-gene mappings and model-selection procedures can leak information or create unstable performance estimates if they are not evaluated under strict split-aware controls.

### Paragraph 4: present study

We therefore built RiceGeneFormer-OMTL, a leakage-aware gene-level ordinal multi-task framework for 3K Rice genotype-to-phenotype prediction. The model integrates train-fold GWAS priors, SNP-to-gene aggregation, gene graph encoding, trait-query decoding, gated top-SNP fusion and class-balanced ordinal training. We compared this model family against LightGBM, XGBoost and SNP-MLP baselines that used the same train-fold top-SNP information, and we recomputed final neural-model metrics with deterministic full-role validation and test evaluation.

### Paragraph 5: contribution and boundary

Our results show that RiceGeneFormer approaches tree-based top-SNP performance and provides a reproducible framework for gene-aware crop genotype-to-phenotype modeling. At the same time, strong balanced SNP-MLP baselines remain superior in macro-F1 under the present random split. The study therefore frames RiceGeneFormer as a benchmarked modeling platform and diagnostic analysis rather than as a universal replacement for marker-based predictors. This bounded framing is important: the contribution is a leakage-aware ordinal benchmark and model family, together with a clear account of which biologically motivated components helped, which did not and where stronger evidence is still required.

## Results

### Result 1. A leakage-aware ordinal phenotype benchmark from 3K rice genomes

We first constructed a genotype-to-phenotype benchmark from the 3K Rice Genome panel in which every GWAS-derived modeling input was restricted to the active training fold. The final genotype matrix contained 3,000 accessions and 365,710 SNPs, and the aligned phenotype and mask matrices contained 35 descriptor-code traits. From these traits we selected ten core ordinal phenotypes for the main modeling experiments, using a random split with 1,586 training, 340 validation and 340 test accessions carrying supervised labels. This split was also used to compute trait-specific GWAS p-values for model priors and top-SNP baselines, preventing validation or test samples from influencing p-value features.

The resulting benchmark retained the structure needed for gene-aware modeling while exposing the main statistical difficulty of the task. The ten core traits varied in class count and coverage, and several were dominated by a single training-set class. This imbalance made accuracy alone insufficient: a model could obtain high accuracy by favoring majority classes while still performing poorly on minority classes. We therefore treated macro-F1, accuracy and ordinal MAE as complementary metrics rather than interchangeable summaries.

### Result 2. RiceGeneFormer-OMTL integrates GWAS priors, gene tokens and ordinal multi-task decoding

We designed RiceGeneFormer-OMTL to test whether gene-aware neural modeling could use the same train-fold marker information as strong SNP baselines while preserving gene structure and ordinal phenotype semantics. The model aggregated SNP dosage into gene tokens through SNP-to-gene bags, incorporated train-fold p-value priors, propagated information with a lightweight graph encoder, decoded traits through trait-specific queries and predicted ordinal descriptor classes with cumulative-link heads. A direct top-SNP branch was also included because pilot experiments showed that marker-level shortcuts remained a strong source of predictive signal.

The final model family used gated top-SNP fusion rather than simple additive fusion. In this design, top-SNP representations were passed through a learned gate before being added to trait tokens, allowing the model to modulate direct marker signal separately for each trait. The final training recipe combined this gated fusion with mild class-balanced ordinal loss (alpha0.25) and validation macro-F1 checkpoint selection. Expected-score distillation from a SNP-MLP teacher was retained as an optional variant, but was not treated as the core modeling advance because its effect was small.

### Result 3. Strong top-SNP baselines define the predictive ceiling

Before interpreting RiceGeneFormer, we established marker-level baselines using train-fold top SNPs selected under the same leakage-control rules. LightGBM reached macro-F1 0.3354, accuracy 0.6271 and MAE 0.5547, while XGBoost reached macro-F1 0.3115, accuracy 0.6274 and MAE 0.5662. These tree baselines showed that top-SNP features alone carried substantial predictive signal and formed a stringent reference point for the gene-aware model.

Balanced SNP-MLP baselines further raised the macro-F1 ceiling. With class-balance interpolation alpha0.40, SNP-MLP reached macro-F1 0.3502±0.0117, accuracy 0.6141±0.0097 and MAE 0.5697±0.0108 across three seeds. Increasing the interpolation strength to alpha0.60 gave macro-F1 0.3544±0.0112, but reduced accuracy to 0.5961±0.0086 and increased MAE to 0.5976±0.0087. Thus, stronger class balancing improved minority-class macro-F1, but the improvement came with a measurable accuracy and ordinal-error trade-off.

### Result 4. Gated top-SNP fusion and mild class balancing yield the strongest RiceGeneFormer configuration

RiceGeneFormer improved most when direct top-SNP signal was fused through a learned gate and the ordinal loss was mildly class-balanced. Earlier bounded pilots showed that trait-specific attention, top-SNP input and gated fusion improved validation loss or macro-F1 more consistently than graph identity or SNP-to-gene mapping changes. Mild class balancing then improved minority-class behavior when paired with macro-F1 checkpoint selection, whereas full class balancing reduced accuracy and MAE too strongly.

Under deterministic full-role test evaluation, the final no-distillation RiceGeneFormer configuration reached macro-F1 0.3229±0.0062 and accuracy 0.5895±0.0029 across seeds 42, 43 and 44. These scores placed RiceGeneFormer close to the LightGBM macro-F1 baseline but below the strongest balanced SNP-MLP baselines. Per-trait analysis showed that the model performed best on lower-cardinality or less extreme traits such as CUDI_CODE_REPRO and PTH, while strongly imbalanced multi-class traits such as CUST_REPRO and SPKF remained the main source of macro-F1 loss.

### Result 5. Expected-score distillation provides only weak incremental gains

We next tested whether a strong SNP-MLP teacher could transfer marker-level predictive information into RiceGeneFormer through expected-score distillation. With distillation weight 0.10, deterministic test macro-F1 increased to 0.3292±0.0057, while weight 0.20 reached 0.3251±0.0054. The best tested weight therefore gave a small gain over the no-distillation model, but the improvement was not monotonic and remained below the balanced SNP-MLP baselines.

This result supports a conservative interpretation. Expected-score distillation can provide a weak positive signal, but it does not solve the gap between gene-aware neural modeling and strong top-SNP neural baselines. More direct probability-level or logit-level distillation may be a better future direction, but the current evidence does not justify presenting expected-score distillation as the main breakthrough.

### Result 6. Graph identity, SNP-to-gene mapping and threshold calibration did not produce robust gains

Several biologically motivated components improved the completeness of the RiceGeneFormer framework but did not produce robust predictive gains in the current bounded setting. Chromosome-neighbor, edge-matched random, STRING and chromosome+STRING fusion graphs produced closely matched multi-seed pilot results. This indicates that the current lightweight mean-neighbor graph encoder did not extract a graph-specific advantage, even though the graph infrastructure and external STRING mapping were successfully integrated.

SNP-to-gene mapping variants also had limited effect. Body-only, ±2 kb, ±5 kb, ±10 kb and nearest-gene mappings produced similar seed42 bounded-pilot macro-F1 values, suggesting that mapping-window choice was not the main bottleneck under the current model capacity and top-SNP fusion setting. Post-hoc threshold calibration was similarly bounded: train-calibrated thresholds slightly helped some short pilot checkpoints but reduced macro-F1, accuracy or MAE for the full-step checkpoints. These negative controls define the current claim boundary: RiceGeneFormer is a leakage-aware and gene-aware benchmark framework, but the present implementation does not demonstrate a stable biological-graph or mapping-specific predictive advantage.

## Methods

### Dataset processing

Genotype and phenotype resources were obtained from the 3K Rice Genome/SNP-Seek data repository. The primary genotype input for this benchmark was the core SNP PLINK resource, which was converted into a dense unsigned-integer dosage matrix with 3,000 accessions and 365,710 SNPs. Phenotype records were taken from the 3KRG morpho-agronomic spreadsheet. The phenotype workbook was parsed by selecting the sheet containing accession identifiers, and sample IDs were aligned to the genotype order through the 3K Rice metadata table and phenotype stock identifiers.

Phenotypes were encoded as masked multi-task targets. The full processed phenotype matrix contained 35 selected descriptor-code traits, with a matching binary mask indicating observed labels. The main experiments used ten high-coverage ordinal traits, hereafter the core ordinal trait set. We used the `random_seed42` split as the primary benchmark split. Supervised accessions were assigned to 1,586 training, 340 validation and 340 test samples; samples without selected labels were retained only for matrix alignment and artifact validation.

### Train-fold GWAS priors

GWAS-derived features were generated only from training samples in the active split. For each split and core ordinal trait, the pipeline produced a one-dimensional p-value vector with length equal to the SNP count (365,710). These p-value arrays were used in two places: as gene-level prior information for RiceGeneFormer and as feature-selection scores for top-SNP baselines. The implementation constrained every p-value path to the expected split and trait directory, checked array shape, finite values and p-value range, and rejected duplicate or out-of-scope GWAS rows.

Top-SNP features were selected by ranking train-fold p-values within each trait. For tree baselines, one model was trained per trait using the corresponding top SNPs. For SNP-MLP and RiceGeneFormer top-SNP fusion, the model used the union of train-fold top SNPs across the core ordinal traits. This design allowed all models to use direct marker information while preventing validation or test labels from contributing to SNP selection.

### SNP-to-gene mapping and graph construction

Rice gene coordinates were taken from Ensembl Plants annotation for the IRGSP-1.0 assembly. The main SNP-to-gene map assigned SNPs to genes using gene bodies plus a ±5 kb window. To test mapping sensitivity, we also generated body-only, ±2 kb, ±10 kb and nearest-gene maps. Each mapping artifact recorded the SNP-to-gene edge table, gene-to-SNP index and gene metadata, and was checked for SNP-index bounds and consistency with the genotype matrix.

Gene graphs were built on the mapped gene node set. The baseline chromosome-neighbor graph linked nearby genes along chromosomes, and a degree-matched random graph served as a negative control. We also built a STRING v12.0 rice graph for Oryza sativa Japonica Group by mapping STRING protein associations to project gene IDs, applying a minimum combined score of 700 and pruning to the top 20 edges per gene. A chromosome+STRING fusion graph was generated by taking the edge union of the chromosome-neighbor and STRING graphs. For bounded graph pilots with 2,048 genes, an additional prefix-random control was used to match local edge density.

### RiceGeneFormer-OMTL architecture

RiceGeneFormer-OMTL is a compact gene-aware ordinal multi-task model. For each selected gene, the model gathers up to a fixed number of mapped SNP dosages and aggregates them into a gene token. Gene tokens are combined with train-fold p-value priors, projected into a shared hidden space and processed by a lightweight graph encoder. The graph encoder uses local neighborhood information from the selected graph and was designed as a bounded message-passing component rather than a full relation-aware graph neural network.

Trait-specific prediction is performed with trait-query attention. Each core ordinal trait has a trait representation that attends to the gene-token sequence, and an ordinal head predicts cumulative-link logits for that trait. A separate top-SNP branch encodes the train-fold top512 SNP union. In the final model family, top-SNP information is fused into trait tokens through a learned gate, allowing direct marker signal to be modulated separately across traits rather than added unconditionally.

### Training and model selection

RiceGeneFormer was trained with AdamW using a learning rate of 2e-4, weight decay of 1e-4, batch size 16, hidden dimension 64, 2,048 genes, 32 SNPs per gene, top512 SNP fusion and 20 epochs for the final full-step runs. Each epoch used up to 100 training steps, approximately covering the 1,586 training samples at the selected batch size. Final RiceGeneFormer runs used class-balanced ordinal loss with interpolation alpha0.25. This mild weighting was chosen because stronger class balancing increased macro-F1 in some pilots but degraded accuracy and ordinal MAE.

Checkpoints were selected by validation macro-F1 for the final model family. The training manifest records the selection metric, best epoch, best selection score, best validation metrics and final validation metrics separately, preventing sampled final-epoch values from being mistaken for selected checkpoint performance. Final reported RiceGeneFormer metrics were recomputed with a deterministic evaluator that traversed all validation or test samples in the requested role.

### Baselines

LightGBM and XGBoost baselines were trained as one classifier per core ordinal trait using the same train-fold top-SNP selection rule. The tree baselines used 512 top SNPs per trait and bounded estimator settings for the primary smoke-style comparison. XGBoost labels were encoded to contiguous training classes where necessary and then mapped back to the original ordinal labels before computing accuracy, MAE, macro-F1 and rank correlation.

The SNP-MLP baseline used the union of train-fold top SNPs across the ten core traits. The main SNP-MLP architecture used a two-layer feed-forward encoder with hidden dimension 256, dropout 0.4 and multi-head trait classifiers. Class-balanced loss was computed only from training labels. The balance-alpha grid interpolated between unweighted and inverse-frequency class weights; alpha0.40 and alpha0.60 were the strongest final baseline settings, with three seeds used for mean and standard deviation estimates.

### Distillation

Teacher predictions were exported from the balanced SNP-MLP baseline as numeric arrays for train, validation and test roles. For RiceGeneFormer distillation, only train-role teacher outputs were used during training. The student ordinal logits were converted to expected ordinal scores by summing valid cumulative probabilities, and the distillation objective matched these expected scores to SNP-MLP teacher expected scores with a masked mean-squared-error loss over observed labels.

The total training loss was the ordinal multi-task loss plus a weighted distillation term. We tested expected-score distillation weights 0.10 and 0.20 in the final full-step setting. Weight 0.10 gave the best RiceGeneFormer test macro-F1 among the tested variants, but gains were small and non-monotonic, so distillation was treated as an exploratory weak-positive component rather than the main model improvement.

## Discussion

### Main message

This study establishes RiceGeneFormer as a leakage-aware, gene-aware and ordinal multi-task framework for plant genotype-to-phenotype prediction. The final model approached tree-based top-SNP performance and improved when direct marker signal was fused through a learned gate, but it did not outperform strong balanced SNP-MLP baselines on the current 3K Rice random split. The central contribution is therefore not a claim of universal neural-model superiority. It is a benchmarked framework that makes the comparison between gene-aware architectures and strong marker-level baselines explicit, reproducible and bounded.

### Biological/modeling interpretation

The results suggest that direct marker-level signal remains difficult for gene-token neural models to surpass in modest-sample plant phenotype panels. Top-SNP baselines benefit from a short path between selected markers and phenotype labels, while gene-token models must compress many SNPs into gene representations before trait prediction. This compression can provide biological organization and interpretability hooks, but it may also dilute sparse predictive SNP signals when phenotype sample size is limited.

The negative graph and mapping results should be interpreted as implementation-specific rather than biological evidence against gene structure. Chromosome-neighbor, STRING and fusion graphs were successfully integrated, but the lightweight mean-neighbor encoder did not yield a stable graph-specific advantage. A relation-aware encoder, better pathway coverage, stronger co-expression graphs or larger phenotype panels could change this conclusion. Similarly, the weak effect of SNP-to-gene window choice may reflect the dominance of the top-SNP branch and the bounded 2,048-gene training setting rather than true insensitivity to regulatory architecture.

The class-balancing results highlight a second modeling tension. Increasing minority-class emphasis improved macro-F1 for SNP-MLP and for some RiceGeneFormer pilots, but stronger weighting reduced accuracy and ordinal MAE. This trade-off is expected for imbalanced descriptor phenotypes and should be reported rather than hidden. For breeding applications, the appropriate operating point may depend on whether the priority is majority-class screening accuracy, minority-class discovery or ordinal ranking fidelity.

### Limitations

Several limitations define the current claim boundary. First, the main evidence uses a random split, while cross-subpopulation and cross-region generalization remain secondary or future analyses. Second, the phenotype panel is modest for deep multi-task learning, and several core traits have severe class imbalance or rare classes. Third, the current graph encoder is lightweight and not relation-aware, so it cannot test the full value of biological pathway or protein-association structure. Fourth, expected-score distillation uses a coarse teacher target; probability-level or logit-level distillation could transfer more information from SNP baselines. Finally, data-accession formatting, release provenance and redistribution policy still need to be finalized before submission.

### Future work

Future work should focus on tests that change the evidence boundary rather than small variants of the current pilots. The most direct extensions are cross-region and cross-subpopulation evaluation, relation-aware pathway or co-expression graph encoders, and probability-level teacher distillation from strong SNP models. Larger phenotype panels and multi-environment crop datasets would also test whether gene-aware ordinal multi-task learning becomes more advantageous when more supervised signal is available. In that setting, RiceGeneFormer can serve as a reusable scaffold for integrating marker effects, gene structure and trait-specific decoding across crop species.

## Immediate writing checklist

Ready now:
- Introduction prose draft.
- Results prose draft.
- Methods prose draft.
- Discussion prose draft.
- main performance numbers.
- draft main results table in `docs/MAIN_RESULTS_TABLE.md`.
- per-trait RiceGeneFormer final table.
- class-distribution table.

Still required before submission:
- exact citation formatting, license wording and local-release provenance for data/annotation/STRING resources; see `docs/DATA_SOURCES_AND_CITATIONS.md`.
- per-trait baseline metrics if required for supplementary comparison.
- final figure polish and legend shortening; current draft legends are in `docs/FIGURE_LEGENDS.md`.
- code/data repository and DOI strategy.
- target journal decision.
