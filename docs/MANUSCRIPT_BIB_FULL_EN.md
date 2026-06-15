# A leakage-aware ordinal benchmark for gene-aware genotype-to-phenotype prediction in 3K rice genomes

Running title: Leakage-aware gene-aware rice phenotype benchmark

Article type: Original Article / Review-style computational benchmark manuscript draft for Briefings in Bioinformatics

Status: near-submission manuscript draft. Quantitative claims are restricted to project artifacts verified as of 2026-06-15. Reference formatting, DOI-backed code release, and final repository accession must be completed before journal submission.

## Abstract

Plant genotype-to-phenotype prediction increasingly relies on dense genomic panels, but many crop phenotypes available at population scale are ordinal descriptor codes with strong class imbalance. This creates two linked challenges: models must respect ordered trait labels, and evaluations must prevent genome-wide association signals or model-selection procedures from leaking validation or test information. We developed RiceGeneFormer-OMTL, a gene-aware ordinal multi-task framework for 3K Rice Genome phenotype prediction, and used it to build a leakage-aware benchmark for ten core ordinal rice traits. The framework combines train-fold GWAS priors, SNP-to-gene aggregation, graph-aware gene tokens, trait-query decoding, gated top-SNP fusion and cumulative-link ordinal heads. All final neural metrics were recomputed with deterministic full-role validation or test evaluation.

RiceGeneFormer reached deterministic test macro-F1 values of 0.3229 +/- 0.0062 without distillation and 0.3292 +/- 0.0057 with weak expected-score distillation. These results approached LightGBM using train-fold top SNPs (macro-F1 0.3380), but remained below balanced SNP-MLP baselines (macro-F1 0.3644 +/- 0.0106 to 0.3781 +/- 0.0029). Gated top-SNP fusion and mild class balancing were the most useful model choices, whereas graph identity, SNP-to-gene window choice, threshold calibration and expected-score distillation produced limited or unstable gains. Cross-region benchmarks across Southeast Asia, South Asia and East Asia did not support a distribution-shift robustness advantage. Gene-attention audits further showed that bounded gene selection can constrain biological interpretation, and a train-fold GWAS-top2048 gene panel did not improve performance or known-gene recovery.

RiceGeneFormer therefore provides a reproducible diagnostic framework rather than a claim of neural-model superiority. The benchmark clarifies when gene-aware crop models approach marker-level baselines, where they fail, and which evidence is required before making robustness or biological-discovery claims.

## Key points

- RiceGeneFormer-OMTL is a leakage-aware gene-level ordinal multi-task framework for 3K rice genotype-to-phenotype prediction.
- Train-fold GWAS priors, top-SNP selection, checkpoint selection and deterministic test evaluation were separated to reduce validation/test leakage.
- RiceGeneFormer approached LightGBM performance but remained below balanced SNP-MLP baselines on macro-F1 under the current random split.
- Cross-region and gene-attention audits are retained as main findings because they define the evidence boundary for robustness and biological interpretation.

## Keywords

3K Rice Genome; genotype-to-phenotype prediction; ordinal classification; genome-wide association study; macro-F1; SNP-to-gene aggregation; benchmark; crop bioinformatics

## Introduction

Dense crop genomic resources have changed the scale at which genotype-to-phenotype prediction can be studied. In rice, the 3K Rice Genome Project and SNP-Seek resources provide thousands of accessions with genome-wide variant calls, public metadata and trait records. These resources support predictive modelling questions that differ from classical single-locus association analysis: can a model use many markers jointly, share information across traits, and retain enough biological structure to be useful beyond a black-box marker predictor? Such questions are important for computational breeding because a model that predicts only within a narrow evaluation setting may be less useful than a benchmark that identifies which forms of genomic signal are stable, which are shortcut-like, and which require stronger evidence.

A central difficulty is that many plant phenotypes available at large panel scale are descriptor codes rather than continuous measurements. Descriptor-code traits are usually categorical, and many are ordinal: the labels represent ordered states, but the spacing between adjacent states is not necessarily quantitative. They are also sparse and imbalanced. A majority class can dominate a trait, while rare agronomic states remain under-represented. This means that accuracy alone can be misleading. A model may correctly predict common classes while performing poorly for minority classes. Conversely, aggressive class balancing may improve macro-F1 while worsening accuracy and ordinal mean absolute error. A genotype-to-phenotype benchmark for such data should therefore treat ordinal structure and class imbalance as first-class design constraints, not as afterthoughts.

A second difficulty is leakage. In genomic prediction, feature selection and prior construction often use GWAS statistics. If GWAS p-values are computed on all available samples before a train/validation/test split is applied, the validation and test labels can influence model inputs. The same risk appears when top-SNP feature sets, calibration thresholds or checkpoint decisions are chosen with information from non-training roles. Leakage can be especially damaging in modest-sample settings, where a small amount of test-derived association signal can inflate model comparisons. For a fair benchmark, GWAS priors and top-SNP sets must be recomputed inside the active training fold, and final metrics must be recomputed deterministically on the full validation or test role rather than inferred from sampled training-loop summaries.

Recent BIB benchmark articles make a further distinction that is directly relevant here: benchmark competitiveness and generalization are not equivalent. Cross-dataset drug-response benchmarks, AMR phenotype prediction studies, multi-population genomic prediction papers and blind protein-ligand scoring benchmarks all show that a method can perform well when train and test samples share study-specific structure, yet fail under external datasets, divergent lineages or different population backgrounds. This logic motivates our separation of the random-split benchmark from the held-out geographic-region benchmark. The random split measures whether a method can learn useful signal under a controlled leakage-aware protocol; the region splits ask whether that signal survives a harder shift in accession origin.

Marker-level baselines are also difficult to beat. Tree models and multilayer perceptrons trained on train-fold top SNPs provide a short path from selected markers to phenotype labels. They may have limited biological organization, but they can be strong predictors. Gene-aware neural models offer an attractive alternative because they can aggregate variants into gene tokens, incorporate graph information, decode trait-specific gene modules and expose attention-like diagnostic outputs. However, these features create another overclaiming risk. A model is not biologically validated merely because it contains gene annotations or attention weights, and graph-aware architecture does not by itself demonstrate robustness to population or geographic shift. A benchmark manuscript therefore needs to compare gene-aware models against strong marker baselines and to report negative controls in the main text.

The closest article-level analogue is the BIB question of whether graph neural networks should really be used for transcriptomic prediction. That paper treats the biological network as a hypothesis to be tested, not as a guarantee of predictive gain. We adopt the same stance for RiceGeneFormer. Gene graphs, SNP-to-gene bags and attention layers are useful infrastructure for controlled experiments and interpretation audits, but their value must be demonstrated against simpler alternatives. This is why the current paper foregrounds graph identity, mapping-window, cross-region and gene-attention audits even when the outcome is negative.

Here we present RiceGeneFormer-OMTL as a leakage-aware ordinal benchmark and diagnostic framework for 3K rice genotype-to-phenotype prediction. The model combines train-fold GWAS priors, SNP-to-gene aggregation, graph-aware gene tokens, trait-query decoding, cumulative-link ordinal heads and a gated top-SNP fusion branch. We evaluate it against LightGBM, XGBoost and balanced SNP-MLP baselines that use the same train-fold top-SNP information. We then use ablations, cross-region benchmarks and gene-attention audits to define the boundary of what the current evidence supports. The resulting manuscript is deliberately conservative: RiceGeneFormer is not presented as a universal replacement for marker-level predictors. Instead, it is a reproducible framework for asking when gene-aware crop models approach strong baselines, where they fail, and what additional evidence would be needed before claiming robustness or biological discovery.

## Results

### A leakage-aware ordinal phenotype benchmark from 3K rice genomes

We first constructed a benchmark that preserved the structure of the 3K rice genotype-to-phenotype problem while enforcing split-aware leakage control. The primary genotype input was the 3K core SNP PLINK resource, processed into a dosage matrix containing 3,000 accessions and 365,710 SNPs. Phenotype records were obtained from the 3KRG morpho-agronomic descriptor spreadsheet and aligned to genotype accessions through the project metadata table. The processed phenotype matrix contained 35 selected descriptor-code traits with a matching observation mask. Ten high-coverage ordinal traits were selected as the core modelling targets for the main experiments.

The primary random split contained 1,586 training, 340 validation and 340 test accessions with supervised labels. Importantly, GWAS p-value vectors were generated only from the training samples of the active split. The resulting trait-specific p-value arrays, each of length 365,710, were then used in two places: as priors for RiceGeneFormer and as feature-selection scores for top-SNP baselines. This rule prevents validation or test labels from influencing GWAS priors, top-SNP selection or model inputs. The same principle was applied to cross-region splits and diagnostic pilots.

The benchmark also made class imbalance explicit. Several core traits were dominated by a single class in the training set. Therefore, performance was summarized with macro-F1, accuracy and ordinal MAE rather than with a single metric. Macro-F1 treats classes equally and highlights minority-class behaviour. Accuracy describes overall label correctness and is sensitive to majority-class performance. MAE measures ordinal error in class units and helps detect cases where a model improves minority-class F1 at the cost of larger ordered mistakes. This multi-metric reporting is essential because class balancing can shift the operating point rather than uniformly improve all aspects of prediction.

### RiceGeneFormer-OMTL integrates GWAS priors, gene tokens and ordinal multi-task decoding

RiceGeneFormer-OMTL was designed to test whether a gene-aware neural architecture can use the same train-fold marker information as strong top-SNP baselines while retaining biological organization. Each selected gene is represented by a GeneBag: a bounded set of mapped SNP dosages gathered from gene body plus a window around the gene. SNP-to-gene aggregation converts marker dosage into gene tokens. Train-fold GWAS p-value priors are supplied as additional evidence, allowing the model to condition gene representations on split-specific association signals without exposing validation or test labels.

Gene tokens are processed by a lightweight graph encoder. The main graph links chromosome-neighbour genes, while STRING-derived protein association and fusion graphs are used in ablations. The graph encoder is intentionally bounded and lightweight; it tests whether local message passing among gene tokens helps within the current data regime, rather than claiming to exhaust the value of biological networks. Trait-specific prediction is performed with trait-query attention. Each core ordinal trait has a query representation that attends to gene tokens and feeds a cumulative-link ordinal head.

Pilot experiments showed that direct marker signal remained important. Therefore, the final model family includes a top-SNP branch using a union of train-fold top SNPs across core traits. Rather than adding this branch unconditionally, the strongest configuration used gated top-SNP fusion, in which the model learns how much direct marker information to inject into each trait token. The final training recipe combined gated fusion with mild class-balanced ordinal loss (alpha0.25) and validation macro-F1 checkpoint selection. Expected-score distillation from an SNP-MLP teacher was tested as an optional component but was treated as exploratory because its gains were small.

### Strong top-SNP baselines define the predictive ceiling

Before interpreting the gene-aware model, we established marker-level baselines under the same leakage-control rules. LightGBM and XGBoost were trained as one classifier per core ordinal trait using train-fold top512 SNPs. On the final test role, LightGBM reached macro-F1 0.3380, accuracy 0.6263 and MAE 0.5630. XGBoost reached macro-F1 0.3161, accuracy 0.6332 and MAE 0.5512. These results show that train-fold top-SNP features alone carry substantial predictive signal and provide a stringent reference for gene-aware modelling.

The strongest baseline was a multi-task SNP-MLP trained on the union of train-fold top SNPs. Class-balance interpolation was important. With balance alpha0.40, the SNP-MLP reached test macro-F1 0.3644 +/- 0.0106, accuracy 0.6317 +/- 0.0023 and MAE 0.5639 +/- 0.0117 across three seeds. With alpha0.60, macro-F1 increased to 0.3781 +/- 0.0029, but accuracy decreased to 0.6148 +/- 0.0091 and MAE increased to 0.5899 +/- 0.0176. This trade-off is informative rather than incidental: stronger minority-class weighting improves macro-F1 but can reduce majority-class accuracy and ordinal fidelity.

These baselines define the central benchmark lesson. A gene-aware model that approaches tree-based top-SNP performance may still fall below a balanced marker-level neural baseline. Therefore, the relevant question is not whether RiceGeneFormer can be declared superior, but whether its architecture and diagnostics provide a reproducible framework for measuring the gap between gene-aware modelling and strong marker shortcuts.

### Gated top-SNP fusion and mild class balancing yield the strongest RiceGeneFormer configuration

RiceGeneFormer improved most consistently when the top-SNP branch was fused through a learned gate and the ordinal loss was mildly class-balanced. Earlier bounded pilots indicated that trait-specific attention, top-SNP input and gated fusion were more useful than graph identity or SNP-to-gene mapping changes. Stronger class balancing helped some macro-F1 pilots but degraded accuracy and MAE, so the final model used alpha0.25 as a conservative operating point.

Under deterministic full-role test evaluation, the no-distillation RiceGeneFormer configuration reached macro-F1 0.3229 +/- 0.0062 and accuracy 0.5895 +/- 0.0029 across seeds 42, 43 and 44. With weak expected-score distillation at weight 0.10, test macro-F1 increased to 0.3292 +/- 0.0057, accuracy remained 0.5894 +/- 0.0049 and MAE was 0.5948 +/- 0.0067. These values placed RiceGeneFormer near LightGBM on macro-F1 but below the best balanced SNP-MLP baselines.

Per-trait results clarified the source of this gap. RiceGeneFormer performed better on lower-cardinality or less extreme traits, such as CUDI_CODE_REPRO and PTH, and struggled on strongly imbalanced multi-class traits such as CUST_REPRO and SPKF. The pattern is consistent with the class-distribution analysis: gene-token compression and multi-task sharing do not automatically solve minority-class scarcity. For BIB-style benchmarking, these hard-trait results are important because they indicate where future improvements should be measured.

### Expected-score distillation provides only weak incremental gains

We tested whether a strong SNP-MLP teacher could transfer marker-level signal into the gene-aware model. Teacher predictions were exported for train, validation and test roles, but only train-role teacher outputs were used during RiceGeneFormer training. Student ordinal logits were converted to expected ordinal scores, and the distillation objective matched student and teacher expected scores over observed labels.

The effect was positive but small. Distillation weight 0.10 improved test macro-F1 from 0.3229 +/- 0.0062 to 0.3292 +/- 0.0057. Increasing the weight to 0.20 reduced macro-F1 to 0.3251 +/- 0.0054. The response was therefore non-monotonic and did not close the gap to balanced SNP-MLP baselines. This result supports a conservative interpretation: expected-score distillation is a weak-positive component, not the central advance. Future distillation should test probability-level or logit-level teacher targets, but the present evidence does not justify stronger claims.

### Graph identity, SNP-to-gene mapping and threshold calibration did not produce stable gains

Several biologically motivated components improved the completeness of the framework but did not generate stable predictive gains in the current bounded setting. Chromosome-neighbour, edge-matched random, STRING and chromosome+STRING fusion graphs produced closely matched pilot macro-F1 values. This indicates that the current lightweight graph encoder did not extract a graph-specific advantage, even though the graph construction and STRING mapping were successfully integrated.

SNP-to-gene mapping variants also had limited effect. Body-only, +/-2 kb, +/-5 kb, +/-10 kb and nearest-gene mappings produced similar pilot macro-F1 values. This suggests that mapping-window choice was not the main bottleneck under the current model capacity and top-SNP fusion setting. Post-hoc threshold calibration was similarly bounded: train-calibrated thresholds helped some short pilots but reduced macro-F1, accuracy or MAE for the full-step checkpoints.

These negative controls are main-text results because they prevent overinterpretation. The current implementation demonstrates a leakage-aware gene-aware benchmark framework, but it does not demonstrate a stable biological-graph advantage or a mapping-window-specific predictive effect. In BIB terms, this is still useful: the paper identifies which biologically motivated modules require stronger architectures or larger data before they can support stronger claims.

### Cross-region and gene-attention audits define the biological claim boundary

We next tested whether RiceGeneFormer showed an advantage under geographic distribution shift. Three held-out region splits were constructed for Southeast Asia, South Asia and East Asia. In each split, baseline models and RiceGeneFormer used split-specific train-fold top-SNP and GWAS rules. RiceGeneFormer was trained with the final gated top-SNP fusion and alpha0.25 recipe across three seeds.

RiceGeneFormer did not show a cross-region robustness advantage. Relative to the SNP-MLP alpha0.60 baseline, RiceGeneFormer macro-F1 gaps were -0.0551 for Southeast Asia, -0.0506 for South Asia and -0.0349 for East Asia. Therefore, the current data do not support a claim that the gene-aware model is more robust to geographic distribution shift than a strong marker-level neural baseline.

We also audited gene-attention outputs. Scores were exported from the best w=0.10 RiceGeneFormer checkpoints for seeds 42, 43 and 44 and averaged over deterministic test-role samples. This audit identified an important interpretation constraint: bounded runs used the first 2,048 genes in graph-node order, corresponding to an early chromosome-1 prefix rather than a genome-wide candidate panel. Many canonical rice genes in the preliminary audit panel, including SD1, GS3, GW2, qSW5/GW5, DEP1, Hd1 and Hd3a, were therefore not visible to the attention layer. Gn1a appeared within the visible subset and ranked within the top 200 for several traits, but this isolated observation is insufficient for a biological discovery claim.

To test whether the prefix limitation could be corrected, we constructed a split-specific GWAS-top2048 gene graph. Genes were ranked by the maximum train-fold GWAS evidence across core traits, and the top 2,048 genes were connected with chromosome-neighbour edges. A seed42 retraining pilot reached deterministic test macro-F1 0.2990, accuracy 0.5836, MAE 0.5878 and Spearman 0.2417, worse than the best current RiceGeneFormer family. Its top attention genes also did not recover the preliminary known-gene panel. Attention outputs should therefore be treated as diagnostic artifacts, not validated candidate-gene evidence.

### Practical interpretation from the benchmark

The benchmark leads to a practical recommendation rather than a single-model prescription. If the immediate goal is maximizing macro-F1 on the present 3K rice random split, the balanced SNP-MLP baselines remain the strongest option among the evaluated models. Alpha0.60 gives the highest macro-F1, whereas alpha0.40 offers a more balanced accuracy/MAE trade-off. LightGBM remains a useful strong tree baseline because it is simple, fast and competitive with RiceGeneFormer on macro-F1. RiceGeneFormer is therefore not the recommended default purely for random-split prediction accuracy.

RiceGeneFormer is most useful when the study goal is gene-aware diagnostic modelling under strict leakage control. It provides a single framework in which train-fold GWAS priors, SNP-to-gene aggregation, graph tokens, trait-specific decoding, top-SNP shortcuts, distillation, cross-region testing and attention export can be evaluated together. In this role, the model functions like the benchmarking frameworks reported in BIB drug-response, m6A, AMR and variant-calling papers: it standardizes comparisons, exposes where apparently attractive modelling choices fail, and prevents unsupported biological or robustness claims. The immediate manuscript contribution is therefore the benchmark and diagnostic protocol, not a claim that RiceGeneFormer should replace marker-level baselines in breeding pipelines.

## Methods

### Data sources and sample alignment

Genotype and phenotype resources were obtained from the public 3K Rice Genome and SNP-Seek repositories. The primary genotype input was the 3K core SNP PLINK resource, processed into a dense SNP dosage matrix containing 3,000 accessions and 365,710 SNPs. Phenotype labels were obtained from the 3KRG morpho-agronomic phenotype spreadsheet. Sample alignment used accession metadata and phenotype stock identifiers to match phenotype rows to genotype order. The benchmark stores phenotype targets as a multi-task label matrix with a matching observation mask.

### Phenotype encoding and core trait selection

The processed phenotype matrix contained 35 descriptor-code traits. Ten high-coverage ordinal traits were selected as the core target set for the main experiments. Labels were treated as ordered categorical classes. Missing phenotype entries were masked and did not contribute to training loss or metric computation. For each observed trait, labels were encoded to valid ordinal class indices while preserving the original ordering required for MAE and rank-based metrics.

### Split construction and leakage control

The primary split was random_seed42 with 1,586 training, 340 validation and 340 test accessions carrying supervised labels. All GWAS priors, top-SNP feature sets, class weights, calibration decisions and checkpoint-selection decisions were restricted to the training and validation roles appropriate for the active experiment. Test labels were used only for final deterministic evaluation. Cross-region experiments used held-out Southeast Asia, South Asia and East Asia roles with the same split-aware rule.

### Train-fold GWAS prior generation

For each split and core ordinal trait, GWAS-derived p-value vectors were generated from training samples only. Each vector had length 365,710, matching the processed SNP matrix. The p-values were used as gene-level priors in RiceGeneFormer and as feature-selection scores for baseline top-SNP models. The pipeline checked p-value shape, finite values, valid range and split-specific path identity to avoid mixing priors across roles or traits.

### SNP-to-gene mapping and graph construction

Rice gene coordinates were taken from Ensembl Plants release 61 annotation for the IRGSP-1.0 assembly. The main SNP-to-gene mapping assigned SNPs to gene bodies plus a +/-5 kb window. Body-only, +/-2 kb, +/-10 kb and nearest-gene mappings were generated for ablation. Gene graphs were built on the mapped gene node set. The main graph linked neighbouring genes along chromosomes. STRING v12.0 Oryza sativa Japonica Group protein associations were mapped to project gene identifiers with a minimum combined score of 700 and top-20 per-gene pruning. Fusion graphs combined chromosome-neighbour and STRING edges.

### RiceGeneFormer-OMTL architecture

For each selected gene, RiceGeneFormer gathers a bounded number of mapped SNP dosages and aggregates them into a gene token. Gene tokens are combined with train-fold p-value priors and projected into a shared hidden space. A lightweight graph encoder updates gene tokens using local graph neighbourhood information. Trait-query attention then produces a trait representation for each core ordinal phenotype. Each trait representation feeds a cumulative-link ordinal head. A separate top-SNP branch encodes the union of train-fold top SNPs. In the final configuration, this branch is integrated with trait tokens through a learned gate.

### Training, model selection and deterministic evaluation

Final RiceGeneFormer full-step runs used AdamW with learning rate 2e-4, weight decay 1e-4, batch size 16, hidden dimension 64, 2,048 genes, 32 SNPs per gene, top512 SNP fusion and 20 epochs. Each epoch used up to 100 training steps. The final model family used class-balanced ordinal loss with alpha0.25. Checkpoints were selected by validation macro-F1. Final validation and test metrics were recomputed by a deterministic evaluator that traversed all samples in the requested role, preventing sampled training-loop summaries from being reported as final metrics.

### Baseline models

LightGBM and XGBoost baselines were trained per trait using train-fold top512 SNPs. Observed labels were encoded as contiguous classes where needed and mapped back to ordinal labels for metric computation. The SNP-MLP baseline used the union of train-fold top SNPs across core traits. Its main architecture was a two-layer feed-forward encoder with hidden dimension 256, dropout 0.4 and multi-head trait classifiers. Class-balanced loss was computed from training labels only. Balance-alpha settings 0.40 and 0.60 were evaluated across seeds 42, 43 and 44.

### Distillation

Balanced SNP-MLP teacher predictions were exported as numeric arrays. RiceGeneFormer used only train-role teacher outputs during training. Student cumulative-link logits were converted to expected ordinal scores, and the distillation objective minimized masked mean-squared error between student and teacher expected scores. Distillation weights 0.10 and 0.20 were tested in final full-step runs.

### Cross-region evaluation

Held-out geographic splits were constructed for Southeast Asia, South Asia and East Asia. For each split, top-SNP baselines and RiceGeneFormer used split-specific train-fold GWAS p-values and feature sets. RiceGeneFormer was trained with gated top-SNP fusion, class-balance alpha0.25, validation macro-F1 checkpoint selection and deterministic full-role test evaluation. Three seeds were used for RiceGeneFormer in each region split.

### Gene-attention export and GWAS-top2048 audit

Trait-gene attention weights were exported from selected RiceGeneFormer checkpoints. Scores were defined as mean trait-to-gene attention over all samples in the requested split role. The export recorded checkpoint identity, split, graph, SNP-to-gene map, trait list, gene list and score definition. For the GWAS-top2048 audit, genes were ranked by the maximum train-fold GWAS evidence across core traits, where gene evidence was the maximum -log10 p-value among SNPs mapped to the gene. The top 2,048 genes were connected with chromosome-neighbour edges and used for a seed42 retraining pilot.

### Metrics

Macro-F1 was computed as the unweighted mean of per-class F1 scores over observed labels. Accuracy was the fraction of observed labels predicted exactly. MAE was the mean absolute difference between predicted and observed ordinal class indices. Spearman correlation was used as a rank-based summary of expected ordinal scores when available. Neural model summaries report mean +/- standard deviation across seeds unless otherwise stated. Single-seed tree baselines report zero standard deviation by convention in the summary tables.

## Discussion

RiceGeneFormer establishes a leakage-aware, gene-aware and ordinal multi-task benchmark for 3K rice genotype-to-phenotype prediction. The main message is deliberately bounded. The model approached LightGBM top-SNP performance and improved with gated top-SNP fusion, mild class balancing and weak distillation. However, it did not outperform strong balanced SNP-MLP baselines under the current random split. This is not a failure of the benchmark; it is the benchmark's central lesson. In modest-sample plant phenotype panels, direct marker-level predictors remain very strong, and gene-aware neural architectures must be compared against them under strict leakage control.

The negative and bounded findings are also informative. Graph identity did not produce a stable advantage, SNP-to-gene mapping windows had small effects, threshold calibration was not consistently helpful, and expected-score distillation was non-monotonic. These results suggest that simply adding biological structure to a neural model is insufficient. A graph module must be expressive enough, the gene panel must cover relevant candidates, and the training objective must transfer marker-level information without collapsing ordinal performance. BIB readers can use these findings as practical guidance when designing crop genomic prediction benchmarks.

The cross-region and gene-attention audits are especially important for claim discipline. RiceGeneFormer did not outperform balanced SNP-MLP in held-out geographic regions, so a robustness claim is not supported. The gene-attention audit showed that bounded gene selection can determine which genes are even visible to interpretation. The GWAS-top2048 replacement corrected this visibility problem in principle, but reduced test macro-F1 and did not recover the preliminary known-gene panel. Attention ranks should therefore be treated as diagnostic outputs unless they are supported by genome-wide coverage, curated QTL resources and independent validation.

Several limitations remain. First, the supervised phenotype panel is modest for deep multi-task learning, and several target traits have severe class imbalance. Second, the graph encoder is lightweight and not relation-aware. Third, the final release DOI and repository accession are not yet minted, so data and code availability statements remain future-tense. Fourth, known-gene interpretation requires final identifier and citation verification. Fifth, the current distillation uses expected scores rather than probability distributions or logits. Addressing these limitations will require not only more experiments but also stronger release and provenance infrastructure.

Future work should focus on evidence-changing tests. The most useful extensions are probability-level teacher distillation, relation-aware graph encoders, curated QTL/pathway overlays, external crop panels and larger multi-environment datasets. If gene-aware ordinal modelling becomes more advantageous when supervised signal increases or when transfer across environments is required, RiceGeneFormer can serve as a reusable scaffold. Until then, the present contribution is a reproducible benchmark and diagnostic framework that clarifies the current boundary between biological structure and marker-level predictive shortcuts.

## Data and code availability

Raw genotype, phenotype and accession metadata are available from the public 3K Rice Genome / SNP-Seek resources. The primary genotype input was the 3K core SNP PLINK resource, and phenotype labels were obtained from the 3KRG morpho-agronomic phenotype spreadsheet. Rice gene coordinates were obtained from Ensembl Plants release 61 for the IRGSP-1.0 assembly, and STRING graph inputs were obtained from STRING v12.0 for Oryza sativa Japonica Group. The code and lightweight source data supporting manuscript figures, final performance tables, cross-region benchmarks and gene-interpretation audits will be released in a versioned public repository before submission. Large raw genotype/phenotype files, large processed matrices, GWAS p-value arrays, checkpoints and job logs are not included in the lightweight manuscript repository by default because they can be regenerated from public inputs and released code and because redistribution policy requires final licence review. The DOI-backed release tag remains to be minted before submission.

## Figure legends

Figure 1. Leakage-aware benchmark construction for 3K rice ordinal genotype-to-phenotype prediction. The workflow aligns 3,000 accessions, 365,710 SNPs and 35 descriptor-code phenotype traits, then selects ten core ordinal traits for modelling. GWAS-derived p-value priors and top-SNP feature sets are generated within each active training fold. Trait imbalance and validated artifact counts summarize why macro-F1, accuracy and ordinal MAE are reported together.

Figure 2. RiceGeneFormer-OMTL architecture and design choices. SNP dosages are aggregated into gene tokens using SNP-to-gene bags and train-fold GWAS priors. Gene tokens are processed with a lightweight graph encoder, decoded by trait-query attention and predicted through ordinal heads. A gated top-SNP branch injects direct train-fold marker signal. Pilot evidence prioritized gated fusion and mild class balance, whereas graph identity showed no stable winner.

Figure 3. Final test performance and hard-trait behaviour. RiceGeneFormer with weak distillation approached LightGBM macro-F1 but remained below balanced SNP-MLP baselines. Accuracy and MAE show the trade-off introduced by stronger class balancing. Per-trait heatmaps and imbalance scatterplots show that strongly imbalanced traits remain the main source of macro-F1 loss.

Figure 4. Ablation boundaries, cross-region checks and gene-attention diagnostics. Expected-score distillation was weak and non-monotonic. Graph identity and SNP-to-gene mapping windows had small effects in bounded pilots. Held-out region benchmarks did not support a RiceGeneFormer advantage over balanced SNP-MLP. A GWAS-top2048 replacement graph corrected the prefix-gene visibility issue but reduced performance and did not support positive known-gene recovery.

## Working reference anchors to format before submission

1. The 3,000 rice genomes project. GigaScience 3, 7 (2014). DOI: 10.1186/2047-217X-3-7.
2. Li J-Y, Wang J, Zeigler RS. The 3,000 rice genomes project: new opportunities and challenges for future rice research. GigaScience 3, 8 (2014). DOI: 10.1186/2047-217X-3-8.
3. Wang W et al. Genomic variation in 3,010 diverse accessions of Asian cultivated rice. Nature 557, 43-49 (2018). DOI: 10.1038/s41586-018-0063-9.
4. Mansueto L et al. Rice SNP-seek database update: new SNPs, indels, and queries. Nucleic Acids Research 45, D1075-D1081 (2017).
5. Alexandrov N et al. SNP-Seek database of SNPs derived from 3000 rice genomes. Nucleic Acids Research 43, D1023-D1027 (2015).
6. Szklarczyk D et al. The STRING database in 2023: protein-protein association networks and functional enrichment analyses for any sequenced genome of interest. Nucleic Acids Research 51, D638-D646 (2023). DOI: 10.1093/nar/gkac1000.
