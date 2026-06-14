# RiceGeneFormer gene-interpretation audit

Last updated: 2026-06-14 22:26:58 CST

## Purpose

This document records the first gene-level interpretation audit after the BIB/NC redesign. The goal is to determine whether the current RiceGeneFormer checkpoints can support a biological interpretation claim, such as recovery of known rice trait genes or QTL-like loci.

Terms:

- gene attention score: the mean trait-to-gene attention weight from `trait_gene_attention`, averaged over all samples in the selected split role.
- top-k gene: a gene ranked by mean attention score for a trait.
- known-gene overlap: a preliminary check against a small manually curated list of well-known rice genes; IDs still require final citation and identifier verification before manuscript use.

## Implemented export path

New script:

- `scripts/model/export_rice_geneformer_gene_scores.py`

Outputs generated from the current best random-split RiceGeneFormer family, w=0.10 expected-score distillation, deterministic test role:

- `data/3krice/processed/gene_attention_w010_seed42_test/gene_attention_top.tsv`
- `data/3krice/processed/gene_attention_w010_seed43_test/gene_attention_top.tsv`
- `data/3krice/processed/gene_attention_w010_seed44_test/gene_attention_top.tsv`
- `data/3krice/processed/gene_attention_w010_test_aggregate/gene_attention_top200_aggregate.tsv`
- `data/3krice/processed/gene_attention_w010_test_aggregate/gene_attention_aggregate_manifest.json`
- `data/3krice/processed/gene_attention_known_gene_overlap_w010_test/known_gene_attention_ranks.tsv`
- `data/3krice/processed/gene_attention_known_gene_overlap_w010_test/known_gene_overlap_manifest.json`

Validation completed:

- `py_compile` passed for the export script.
- CPU smoke export passed on a full-step checkpoint with val role.
- Seed42/43/44 test-role exports produced 2,000 top-gene rows each: 10 traits × top 200 genes.
- Aggregate NPZ/TSV and manifest were generated without non-finite values.

## Critical finding: current checkpoints are biologically truncated

The existing bounded RiceGeneFormer training uses `max_genes=2048` with the first genes in `gene_nodes.tsv` order. Because `gene_nodes.tsv` is ordered by genome coordinate, the current model effectively sees only an early chromosome-1 prefix rather than a genome-wide top-gene panel.

Consequence:

- Many canonical rice genes are not even available to the current model attention layer.
- Therefore, current attention export cannot be used as strong biological evidence.
- This is not a data-processing bug; it is a bounded-training design limitation.

Examples from the preliminary known-gene rank table:

| Known gene | RAP-style ID used in audit | Status in current 2048-gene model |
|---|---|---|
| SD1 | gene:Os01g0883800 | not in first 2048 genes |
| GS3 | gene:Os03g0407400 | not in first 2048 genes |
| GW2 | gene:Os02g0244100 | not in first 2048 genes |
| qSW5/GW5 | gene:Os05g0187500 | not in first 2048 genes |
| DEP1 | gene:Os09g0441900 | not in first 2048 genes |
| Hd1 | gene:Os06g0275000 | not in first 2048 genes |
| Hd3a | gene:Os06g0157700 | not in first 2048 genes |

Only `Gn1a` (`gene:Os01g0197700`) is present in the current bounded gene set and appears within top 200 for several traits, including rank 3 for `LLT_CODE` and rank 21 for `PSH`. This is interesting but insufficient as a manuscript-level biological validation claim because the candidate panel is heavily biased toward early chromosome 1.

## Top-gene pattern in current checkpoints

The current top attention genes repeatedly concentrate on early chromosome 1 loci, especially:

- `gene:Os01g0161000`
- `gene:Os01g0229800`
- `gene:Os01g0564950`
- `gene:Os01g0563700`
- `gene:Os01g0563500`
- `gene:Os01g0195700`
- `gene:Os01g0197700` (`Gn1a` candidate ID in the preliminary panel)

Interpretation:

- The pattern may reflect strong train-fold GWAS priors and local model attention within the bounded prefix.
- It should not be interpreted as genome-wide biological discovery.
- The repeated early-chr1 concentration is a warning signal for model-input selection bias.

## Corrective action started

To make gene-level interpretation biologically meaningful, I added a split-specific GWAS-top gene graph builder:

- `scripts/preprocess/build_gwas_top_gene_graph.py`

Generated graph:

- `data/3krice/processed/gene_graph/baseline/gwas_top2048_random_seed42_chr_neighbor_k5/`

Graph construction:

- active split: `random_seed42`
- trait group: `core_ordinal`
- SNP-to-gene map: `window_5kb`
- selected genes: top 2,048 genes by train-fold GWAS evidence
- score definition: max over traits of max `-log10(train-fold GWAS p-value)` across SNPs mapped to each gene
- graph edges: chromosome-neighbor k=5 among the selected genes
- output: 2,048 nodes and 20,300 directed edges

Validation:

- `py_compile` passed.
- graph manifest passed `json.tool`.
- RiceGeneFormer CPU smoke with this graph passed with `status=ok`, `genes_used=64`, `graph_edges_used=610`.

The first seed42 GPU pilot has completed on `gpu10` physical GPU2 with the same main architecture but the GWAS-top2048 gene graph.

Result:

- output directory: `data/3krice/processed/rice_geneformer_gwas_top2048_seed42_e20_s100_gpu2/`
- deterministic test macro-F1: `0.2990`
- test accuracy: `0.5836`
- test MAE: `0.5878`
- test Spearman: `0.2417`

This is worse than the current w=0.10 distillation random-split RiceGeneFormer test macro-F1 (`0.3292±0.0057`) and worse than the no-distillation full-step family (`0.3229±0.0062`). Therefore, replacing the prefix genes with a simple train-fold GWAS-top gene panel does not improve prediction in this first seed42 pilot.

Gene attention was also exported for this seed42 GWAS-top model:

- `data/3krice/processed/gene_attention_gwas_top2048_seed42_test/gene_attention_top.tsv`
- `data/3krice/processed/gene_attention_gwas_top2048_seed42_test/gene_attention_manifest.json`

The preliminary known-gene panel did not appear in the top 300 attention genes for any trait, and the audited canonical genes were not selected into the GWAS-top2048 panel. This weakens the immediate biological-discovery route.

## Journal implication

Current state after this audit:

- NC biological-discovery route is not yet supported.
- BIB route remains viable, and this audit actually strengthens the benchmark/diagnostic narrative by identifying a bounded-training interpretation pitfall.
- For manuscript claims, do not say RiceGeneFormer recovers known rice genes yet.
- Safer claim: current bounded gene-attention export exposed a gene-selection bias, motivating a leakage-safe GWAS-top gene panel for interpretable follow-up.

## Next decision rule

Decision after the GWAS-top2048 seed42 pilot:

1. Do not spend GPU time on seed43/44 for this exact GWAS-top2048 graph unless a stronger biological candidate panel or QTL interval resource is added.
2. Keep the prefix-gene and GWAS-top gene-attention results as diagnostic limitations, not as positive biological evidence.
3. For BIB, use this as a reproducibility/benchmarking caution: bounded gene selection can materially constrain interpretability.
4. For NC, current gene-attention evidence is insufficient. A credible NC route would require a proper QTL database overlap or an externally validated candidate-gene analysis, not just attention ranks.
