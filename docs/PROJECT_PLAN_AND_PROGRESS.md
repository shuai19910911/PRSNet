# RiceGeneFormer 水稻 3K Genome 正式研究计划与进展

最后更新：2026-06-13 16:12:27 CST

> 本文件是项目唯一主进展文件。后续每完成一个小阶段，只更新本文件中的“阶段进展记录”和必要计划状态，不新增零散进展文件。

## 1. 研究总目标

本项目目标是研发 RiceGeneFormer：面向作物 genotype-to-phenotype prediction 的 gene-aware ordinal multi-task foundation-style model。

第一阶段以 3K Rice Genome / SNP-Seek 为核心数据，构建：

```text
SNP dosage
  → train-fold significance-aware variant encoder
  → SNP-to-gene GeneBag aggregator
  → chromosome local-context encoder
  → rice gene knowledge graph encoder
  → trait-query decoder
  → ordinal / binary / nominal phenotype heads
```

目标不是做测试 demo，而是直接按可发表标准建立正式训练流程：真实 3K Rice genotype + phenotype；train-fold GWAS p-value 无泄漏先验；水稻 gene annotation 的 SNP-to-gene mapping；rice PPI/pathway/co-expression graph；多任务 ordinal/categorical prediction；严格 baseline/ablation；候选基因、通路和 trait-specific gene modules 解释。

## 2. 核心科学问题

1. 面向植物 descriptor code 的 ordinal multi-task learning 是否优于把等级表型当作普通连续回归或普通分类？
2. train-fold GWAS significance-aware variant gating 是否能在 2,266 样本规模下缓解过拟合并提高候选基因解释性？
3. 染色体局部上下文与水稻基因知识图谱的融合是否能提升跨亚群、跨地理区域泛化？
4. trait-query decoder 是否能为不同农艺性状学习不同 gene module，而不是共享一个全局 readout？
5. RiceGeneFormer 的设计能否后续迁移到 maize G2F 的 G×E 多环境预测？

## 3. 当前已完成数据准备状态

数据路径：本地项目目录下 `data/3krice`，不上传 GitHub。

已完成：

1. `phenotype.xlsx` 下载并验证可读。
2. `core_v0.7.bed.gz/bim.gz/fam.gz` 下载并验证 PLINK BED 可读；但 `.fam` 使用 B001/B002，不作为主训练数据。
3. `3K_list_sra_ids.txt` 下载，用于 metadata 对接。
4. `3K_coreSNP-v2.1.plink.tar.gz` 下载并验证，作为主 genotype 数据。

正式主数据：`raw/3K_coreSNP-v2.1.plink.tar.gz`。

选择原因：365,710 SNP；3,000 samples；PED 样本 ID 直接为 IRIS ID；与 metadata 完整对接 3,000/3,000；与 phenotype 完整对接 2,266/2,266。

phenotype：2,266 rows × 53 columns，其中 47 个 phenotype fields。当前 phenotype 主要是 IRRI rice descriptor code，类型包括 ordinal、nominal categorical、binary，不适合统一当连续回归处理。

## 4. 正式模型定位

主模型：`RiceGeneFormer-OMTL`

全称：Rice Gene-aware Ordinal Multi-Task Transformer。

关键模块：

1. `VariantGate`：SNP-level train-fold significance-aware gating。
2. `GeneBag`：基于 gene body ±5 kb 的 SNP-to-gene attention aggregation。
3. `ChrFormer`：染色体分段 local attention，捕获 LD block / cis-neighborhood。
4. `RiceGraphEncoder`：水稻 PPI、pathway、co-expression 多关系图编码。
5. `TraitQueryDecoder`：每个 trait 独立 query cross-attend 到 gene tokens。
6. `OrdinalMultiHead`：ordinal cumulative-link / binary BCE / nominal weighted CE 三类输出头。
7. `MaskedMultiTaskLoss`：支持 trait 缺失 mask、类别不平衡权重、attention sparse penalty、p-value-weighted L1。

## 5. 正式研究阶段计划

### Phase 0：项目文档与仓库同步

产物：`README.md`, `docs/PROJECT_PLAN_AND_PROGRESS.md`, `docs/MODEL_ARCHITECTURE.md`, `docs/TECHNICAL_ROUTE.md`。

资源：登录节点；预计时间 0.5–1 h；状态：已完成主文档重构，后续随阶段更新。

### Phase 1：正式输入数据构建

1. 解包 PED/MAP：1–2 CPU / 2–4G RAM / 4.5G disk / 10–30 min。
2. PED/MAP 转 `X_uint8.npy` 或 zarr：30 CPU / 80–150G RAM / 10–20G disk / 1–4 h。
3. phenotype 标准化与 trait type 标注：4 CPU / 8G RAM / <0.5 h。
4. 构建 random/subpopulation/region splits：1 CPU / 2G RAM / <10 min。

验收：`X`、`Y`、mask、sample_order 完全对齐；split 无泄漏；至少 10 个 high-coverage traits 入选。

### Phase 2：train-fold GWAS 与 p-value 先验

每个 split/trait 只用 train samples 生成 p-values，并加入 genotype PCs。

资源：每 trait 30 CPU / 100–150G RAM / 0.5–2 h；首批 10 traits 6–18 h。

验收：每个 trait 有 `pvalues.npy` 和 `gwas_summary.tsv.gz`；pvalue 维度与 SNP 顺序一致；严禁使用全数据 p-value。

### Phase 3：SNP-to-gene mapping

主窗口 gene body ±5kb。消融：body only、±2kb、±10kb、nearest-gene assignment。

资源：8 CPU / 16G RAM / 10–40 min。

验收：`gene2snps.pkl`、`snp2gene.tsv`、`gene_metadata.tsv` 与 genotype SNP index 对齐。

### Phase 4：rice gene knowledge graph 构建

图类型：STRING rice、Plant Reactome/KEGG pathway、公开 RNA-seq atlas co-expression、hybrid multi-relation、random degree-matched。

资源：16 CPU / 16–60G RAM / 1–4 h；下载图数据库时需要网络。

验收：graph gene IDs 与 `gene2snps` 交集明确，random graph negative control 完成。

### Phase 5：RiceGeneFormer-OMTL 正式训练

主配置：

```text
snp_kernels: 24
hidden_dim: 96
gene_projection_rank: 16
chromosome_attention_window: 64 or 128 genes
graph_layers: 2
trait_decoder_layers: 2
dropout: 0.15
edge_dropout: 0.10
optimizer: AdamW
lr: 2e-4
weight_decay: 1e-4
epochs: 300
early_stop_patience: 40
precision: AMP
```

资源：单模型单 split 1 GPU / 8–16 CPU / 60–120G RAM / 6–18 h；3 split 主模型 18–54 h。

### Phase 6：baseline 与消融

必须完成：rrBLUP/GBLUP、XGBoost/LightGBM、SNP-MLP、gene bag only、chromosome context only、graph only、chromosome+graph fusion、STRING graph、pathway graph、co-expression graph、hybrid graph、random graph、without significance gate、without p-value L1、without trait-query decoder、ordinary CE vs ordinal cumulative-link。

资源：baseline + ablation 2–5 天。

### Phase 7：解释性与下游任务

输出 gene attention、top candidate genes、trait gene module、pathway enrichment、known QTL overlap、cross-subpopulation generalization、cross-region generalization。后续迁移到 maize G2F 时加入 environment encoder 与 genotype × environment interaction block。

## 6. 总体时间预估

```text
文档与仓库：0.5–1 h
数据预处理：1–2 天
GWAS p-value：1 天内首批完成
SNP-to-gene + graph：0.5–1 天
主模型训练：1–3 天
baseline + ablation：2–5 天
解释性分析：1–2 天
第一轮完整正式结果：约 7–12 天
```

## 7. 当前阶段进展记录

- [2026-06-13 15:34:30 CST] 建立项目研究计划文档结构；明确 GitHub 只保存 README、计划进展、模型结构、技术路线，不上传数据/结果/日志/权重。
- [2026-06-13 15:00:00 CST] 下载 3K Rice phenotype 与 core v0.7 PLINK BED/BIM/FAM；验证 gzip/xlsx 完整，BED magic bytes 为 `6c 1b 01`，404,388 SNP × 3,024 samples 可解析。
- [2026-06-13 15:01:00 CST] 发现 v0.7 `.fam` 使用 B001/B002 样本 ID；phenotype 与 3K metadata 可通过 IRGC 编号 2,266/2,266 对接，但 v0.7 缺少 B-code 到 IRIS 映射，不作为主训练数据。
- [2026-06-13 15:19:00 CST] 下载 `3K_coreSNP-v2.1.plink.tar.gz`；验证 gzip 完整；解析得到 365,710 SNP × 3,000 samples，样本 ID 直接为 IRIS ID。
- [2026-06-13 15:25:00 CST] 完成 v2.1 PED 样本与 metadata / phenotype 对接；PED vs metadata：3,000/3,000；PED vs phenotype：2,266/2,266；确认 v2.1 作为第一阶段正式主 genotype 数据。
- [2026-06-13 15:53:45 CST] 根据 phenotype trait summary 确认当前 3K Rice 表型主要是分类/有序等级 descriptor code；将建模目标调整为 ordinal / binary / nominal masked multi-task learning。
- [2026-06-13 16:12:27 CST] 将正式模型体系重构为自研 RiceGeneFormer-OMTL：新增 VariantGate、GeneBag、ChrFormer、RiceGraphEncoder、TraitQueryDecoder、OrdinalMultiHead；正式文档不再使用旧模型命名字段。
- [2026-06-13 16:37:27 CST] 启动 Phase 1 正式输入构建；本地新增 PED/MAP 流式转 `X_uint8.npy` 脚本、phenotype 多任务矩阵脚本和 SLURM q07 预处理脚本；已完成 `py_compile`、`sh -n`、`git diff --check`、静态安全扫描和独立代码审核，等待在计算节点提交预处理作业。

## 8. 下一步执行优先级

1. 在 q07 提交 `scripts/slurm/preprocess_rice_geneformer.sh`，把 PED/MAP 转为 `X_uint8.npy` 并生成 phenotype multitask matrix。
2. 作业完成后检查 `processed/genotype/conversion_report.tsv`、`processed/phenotype/phenotype_report.tsv`、`trait_metadata.tsv` 和矩阵维度。
3. 构建 random/subpopulation/region splits。
4. 在 train fold 内生成 pvalues。
5. 下载 rice annotation，构建 gene2snps。
6. 下载或构建 rice gene knowledge graph。
