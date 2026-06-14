# Data sources and citation audit for RiceGeneFormer

Status: lightweight citation-control note prepared on 2026-06-14 19:35:53 CST. This file records source URLs and citation anchors for manuscript drafting. It does not include raw data, logs, model weights or credentials.

## 1. 3K Rice Genome genotype and phenotype sources

### Primary dataset used in this project

The project uses the 3K Rice Genome / SNP-Seek download resources hosted by IRRI/AWS mirrors. The local working inputs are:

- `raw/3K_coreSNP-v2.1.plink.tar.gz`: main genotype source used for the current 365,710-SNP matrix.
- `raw/phenotype.xlsx`: 3K Rice Genome morpho-agronomic phenotype spreadsheet.
- `raw/3K_list_sra_ids.txt`: metadata table used for sample accession matching.

The 42basepairs mirror of the SNP-Seek download page identifies itself as the data repository for the 3K Rice Genomes Project and Oryza SNP Project hosted by IRRI. The page lists the 3KRG morpho-agronomic Excel file and PLINK-formatted core SNP datasets, and links to the data usage license.

Source URL checked:

- https://42basepairs.com/download/s3/3kricegenome/3kRG_download.html

Citation anchors to use in the manuscript:

1. The 3,000 rice genomes project. GigaScience 3, 7 (2014). DOI: 10.1186/2047-217X-3-7. PMID: 24872877.
2. Li, J.-Y., Wang, J. & Zeigler, R. S. The 3,000 rice genomes project: new opportunities and challenges for future rice research. GigaScience 3, 8 (2014). DOI: 10.1186/2047-217X-3-8. PMID: 24872878.
3. Wang, W. et al. Genomic variation in 3,010 diverse accessions of Asian cultivated rice. Nature 557, 43-49 (2018). DOI: 10.1038/s41586-018-0063-9. PMID: 29695866.
4. Mansueto, L. et al. Rice SNP-seek database update: new SNPs, indels, and queries. Nucleic Acids Research 45, D1075-D1081 (2017). PMID: 27899667.
5. Alexandrov, N. et al. SNP-Seek database of SNPs derived from 3000 rice genomes. Nucleic Acids Research 43, D1023-D1027 (2015). PMID: 25429973.

### Manuscript wording

Recommended wording:

"Genotype and phenotype resources were obtained from the 3K Rice Genome/SNP-Seek data repository. We used the core SNP PLINK resource as the primary genotype source and the 3KRG morpho-agronomic phenotype spreadsheet for descriptor-code phenotypes."

Boundary:

- Do not claim that the exact local filtered 365,710-SNP resource is the full 29M or 32M SNP release.
- Cite the broader 3K Rice Genome papers and SNP-Seek database papers, and state the local SNP count as a processed benchmark artifact.

## 2. Rice gene annotation

The project uses Ensembl Plants Oryza sativa Japonica Group annotation on the IRGSP-1.0 assembly. The checked Ensembl Plants species page reports IRGSP-1.0 as the genome assembly and provides GFF3 downloads for gene annotation.

Source URL checked:

- https://plants.ensembl.org/Info/Index

Current web page status observed during audit:

- Ensembl Plants release 62 page was visible at the time of checking.
- The local pipeline history indicates release 61 GFF3 was downloaded earlier for this project. The manuscript should therefore cite Ensembl Plants generally and record the exact downloaded release in Methods from the local download manifest or script.

Recommended wording:

"Rice gene coordinates were taken from Ensembl Plants annotation for the IRGSP-1.0 assembly. SNP-to-gene maps were built from gene bodies with the main window set to ±5 kb, with body-only, ±2 kb, ±10 kb and nearest-gene variants used for ablation."

Boundary:

- Before submission, verify whether the local GFF3 is release 61 or another exact release, and cite the exact URL in the data availability statement.

## 3. STRING rice graph

The local project history records STRING v12.0 Oryza sativa Japonica taxon 39947 aliases and links files used to build the `string_v12_min700_top20` graph. Web search confirmed taxon 39947 corresponds to Oryza sativa Japonica Group, but did not retrieve a strong official STRING landing page snippet in this audit pass.

Local graph facts already verified in project logs:

- Graph name: `string_v12_min700_top20`.
- Taxon: 39947.
- Nodes: 34,139 mapped genes.
- Mapped STRING proteins: 33,979.
- Raw mapped undirected edges: 459,820.
- Pruned graph: 61,735 undirected edges / 123,470 directed edges.
- Score range: 700-999.
- Maximum degree after pruning: 20.

Recommended wording:

"Protein-association edges were derived from STRING v12.0 for Oryza sativa Japonica Group and mapped onto the project gene node set. We used a minimum combined score of 700 followed by top-20 per-gene pruning for the bounded STRING graph."

Boundary:

- Before submission, verify and cite the exact STRING v12 publication and download URL used by the script.

## 4. Current unresolved citation items

These items remain unresolved or should be finalized before submission:

1. Exact local Ensembl Plants release number and GFF3 file URL.
2. Exact STRING v12 file download URLs and STRING database publication citation.
3. Data usage license wording for the 3KRG/SNP-Seek resources.
4. Repository/DOI strategy for project code and lightweight processed summaries.

## 5. How this affects the manuscript checklist

The raw 3K Rice Genome and SNP-Seek source identity is now sufficiently verified for draft Methods wording. The remaining pre-submission gap is not the existence of the data source, but exact citation formatting, license text and local-release provenance for annotation/STRING resources.
