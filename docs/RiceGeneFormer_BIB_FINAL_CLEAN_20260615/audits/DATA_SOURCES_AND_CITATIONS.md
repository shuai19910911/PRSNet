# Data sources and citation audit for RiceGeneFormer

Status: lightweight citation-control note updated on 2026-06-14 22:55 CST. This file records source URLs and citation anchors for manuscript drafting. It does not include raw data, logs, model weights or credentials.

## 1. 3K Rice Genome genotype and phenotype sources

### Primary dataset used in this project

The project uses the 3K Rice Genome / SNP-Seek download resources hosted by IRRI/AWS mirrors. The local working inputs are:

- `raw/3K_coreSNP-v2.1.plink.tar.gz`: main genotype source used for the current 365,710-SNP matrix.
- `raw/phenotype.xlsx`: 3K Rice Genome morpho-agronomic phenotype spreadsheet.
- `raw/3K_list_sra_ids.txt`: metadata table used for sample accession matching.

The 42basepairs mirror of the SNP-Seek download page identifies itself as the data repository for the 3K Rice Genomes Project and Oryza SNP Project hosted by IRRI. The page lists the 3KRG morpho-agronomic Excel file and PLINK-formatted core SNP datasets, and links to the data usage license.

Source URLs checked:

- https://42basepairs.com/download/s3/3kricegenome/3kRG_download.html
- https://3kricegenome.s3.amazonaws.com/3kRG_download.html
- https://s3.amazonaws.com/3kricegenome/README-3kRG-SNPs-Permissive-License.txt
- https://42basepairs.com/browse/s3/3kricegenome/snpseek-dl?file=3K_coreSNP-v2.1.plink.tar.gz

Citation anchors to use in the manuscript:

1. The 3,000 rice genomes project. GigaScience 3, 7 (2014). DOI: 10.1186/2047-217X-3-7. PMID: 24872877.
2. Li, J.-Y., Wang, J. & Zeigler, R. S. The 3,000 rice genomes project: new opportunities and challenges for future rice research. GigaScience 3, 8 (2014). DOI: 10.1186/2047-217X-3-8. PMID: 24872878.
3. Wang, W. et al. Genomic variation in 3,010 diverse accessions of Asian cultivated rice. Nature 557, 43-49 (2018). DOI: 10.1038/s41586-018-0063-9. PMID: 29695866.
4. Mansueto, L. et al. Rice SNP-seek database update: new SNPs, indels, and queries. Nucleic Acids Research 45, D1075-D1081 (2017). PMID: 27899667.
5. Alexandrov, N. et al. SNP-Seek database of SNPs derived from 3000 rice genomes. Nucleic Acids Research 43, D1023-D1027 (2015). PMID: 25429973.
6. Toronto International Data Release Workshop Authors. Prepublication data sharing. Nature 461, 168-170 (2009). DOI: 10.1038/461168a.

License wording checked:

- `README-3kRG-SNPs-Permissive-License.txt` states that the SNP, indel and large-SV datasets are released under the permissive license stated in the Toronto Statement, and that downloading or using the dataset means agreeing to abide by the spirit of the Toronto Statement.

### Manuscript wording

Recommended wording:

"Genotype and phenotype resources were obtained from the 3K Rice Genome/SNP-Seek data repository. We used the core SNP PLINK resource as the primary genotype source and the 3KRG morpho-agronomic phenotype spreadsheet for descriptor-code phenotypes."

Boundary:

- Do not claim that the exact local filtered 365,710-SNP resource is the full 29M or 32M SNP release.
- Cite the broader 3K Rice Genome papers and SNP-Seek database papers, and state the local SNP count as a processed benchmark artifact.

## 2. Rice gene annotation

The project uses Ensembl Plants Oryza sativa Japonica Group annotation on the IRGSP-1.0 assembly. The checked Ensembl Plants species page reports IRGSP-1.0 as the genome assembly and provides GFF3 downloads for gene annotation.

Source URLs checked:

- https://plants.ensembl.org/Info/Index
- https://plants.ensembl.org/Oryza_sativa/Info/Index
- https://ftp.ensemblgenomes.ebi.ac.uk/pub/plants/release-61/gff3/oryza_sativa/
- https://ftp.ensemblgenomes.ebi.ac.uk/pub/plants/release-61/gff3/oryza_sativa/Oryza_sativa.IRGSP-1.0.61.gff3.gz

Current web page and local-script status observed during audit:

- The live Ensembl Plants species page currently shows release 63 and IRGSP-1.0 as the genome assembly.
- The local download script uses Ensembl Plants release 61 GFF3: `Oryza_sativa.IRGSP-1.0.61.gff3.gz`.
- The release-61 FTP index lists `Oryza_sativa.IRGSP-1.0.61.gff3.gz`, last modified 2025-04-09, size 7.0M.

Recommended wording:

"Rice gene coordinates were taken from Ensembl Plants release 61 annotation for the IRGSP-1.0 assembly (`Oryza_sativa.IRGSP-1.0.61.gff3.gz`). SNP-to-gene maps were built from gene bodies with the main window set to ±5 kb, with body-only, ±2 kb, ±10 kb and nearest-gene variants used for ablation."

Boundary:

- Before submission, decide whether to cite Ensembl Plants generally, the release-61 FTP URL, or both; the local file identity is now release 61.

## 3. STRING rice graph

The local project history records STRING v12.0 Oryza sativa Japonica Group taxon 39947 aliases and links files used to build the `string_v12_min700_top20` graph. The Ensembl Plants Oryza sativa species page also lists taxonomy ID 39947 for Oryza sativa Japonica Group.

Local graph facts already verified in project logs:

- Graph name: `string_v12_min700_top20`.
- Taxon: 39947.
- Nodes: 34,139 mapped genes.
- Mapped STRING proteins: 33,979.
- Raw mapped undirected edges: 459,820.
- Pruned graph: 61,735 undirected edges / 123,470 directed edges.
- Score range: 700-999.
- Maximum degree after pruning: 20.

Source URLs and citation anchors:

- STRING database: https://string-db.org/
- STRING v12.0 publication: Szklarczyk, D. et al. The STRING database in 2023: protein-protein association networks and functional enrichment analyses for any sequenced genome of interest. Nucleic Acids Research 51, D638-D646 (2023). DOI: 10.1093/nar/gkac1000.
- Direct download URLs used by STRING v12.0 bulk resources and verified by HTTP HEAD: `https://stringdb-downloads.org/download/protein.aliases.v12.0/39947.protein.aliases.v12.0.txt.gz` and `https://stringdb-downloads.org/download/protein.links.v12.0/39947.protein.links.v12.0.txt.gz`.

Recommended wording:

"Protein-association edges were derived from STRING v12.0 for Oryza sativa Japonica Group (taxonomy ID 39947) and mapped onto the project gene node set. We used a minimum combined score of 700 followed by top-20 per-gene pruning for the bounded STRING graph."

Boundary:

- The local file names, parser header check and HTTP size check match the compact `protein.links.v12.0` three-column format rather than `protein.links.full.v12.0`.

## 4. Current unresolved citation items

These items remain unresolved or should be finalized before submission:

1. Final DOI or accession for the code/source-data release. The planning document is now `docs/DATA_AND_CODE_AVAILABILITY_PLAN.md`, but no public release DOI has been minted yet.
2. Final journal-specific formatting of the data, annotation and STRING references.
3. Source-data README/data dictionary for any lightweight TSV/CSV/JSON summaries copied out of local `data/3krice/processed/` into a public release directory.

## 5. How this affects the manuscript checklist

The raw 3K Rice Genome/SNP-Seek source identity, data-usage license wording, Ensembl Plants release-61 annotation identity and STRING v12 compact-link download identity are now sufficiently verified for draft Methods wording. The remaining pre-submission gap is not the existence of the data sources, but final journal-specific reference formatting, DOI-backed code/source-data release, and source-data dictionary preparation.
