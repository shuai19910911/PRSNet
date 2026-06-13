# CropPRSNet 水稻 3K Genome 正式研究计划与进展

最后更新：2026-06-13 15:53:45 CST

> 本文件是项目唯一主进展文件。后续每完成一个小阶段，只更新本文件中的“阶段进展记录”和必要的计划状态，不新增零散进展文件。

## 1. 研究总目标

本项目目标是将 PRSNet-2 从人类 polygenic risk prediction 扩展为植物 genotype-to-phenotype 的正式大模型框架 CropPRSNet。

第一阶段以 3K Rice Genome / SNP-Seek 为核心数据，构建：

```text
SNP dosage → SNP-to-gene encoder → rice gene graph encoder → multitask phenotype predictor
```

目标不是做测试 demo，而是直接按可发表标准建立正式训练流程：真实 3K Rice genotype + phenotype；train-fold GWAS p-value 无泄漏先验；水稻 gene annotation 的 SNP-to-gene mapping；rice PPI/pathway/co-expression graph；多任务预测；严格 baseline/ablation；候选基因和通路解释。

## 2. 核心科学问题

1. PRSNet-2 的层级 SNP→gene 编码是否能提升植物复杂表型预测，尤其跨亚群/跨地理区域泛化？
2. GWAS significance-guided regularization 是否能缓解植物小样本过拟合？
3. gene attention 是否能恢复已知候选基因/QTL/pathway？
4. 框架能否后续迁移到 maize G2F G×E 多环境预测？

## 3. 当前已完成数据准备状态

数据路径：`/home/user/zhangzhishuai/myhermes/PRSNet/data/3krice`。

已完成：

1. `phenotype.xlsx` 下载并验证可读。
2. `core_v0.7.bed.gz/bim.gz/fam.gz` 下载并验证 PLINK BED 可读；但 `.fam` 使用 B001/B002，不作为主训练数据。
3. `3K_list_sra_ids.txt` 下载，用于 metadata 对接。
4. `3K_coreSNP-v2.1.plink.tar.gz` 下载并验证，作为主 genotype 数据。

正式主数据：`raw/3K_coreSNP-v2.1.plink.tar.gz`。

原因：365,710 SNP；3,000 samples；PED 样本 ID 直接为 IRIS ID；与 metadata 完整对接 3,000/3,000；与 phenotype 完整对接 2,266/2,266。

phenotype：2,266 rows × 53 columns。高覆盖 trait 初筛包括 AWPR_REPRO、CUDI_CODE_REPRO、CULT_CODE_REPRO、CUNO_CODE_REPRO、LLT_CODE、PLT_CODE_POST、SDHT_CODE、APCO_REV_REPRO、AWCO_REV、BLPUB_VEG、BLSCO_REV_VEG、CCO_REV_VEG、CUAN_REPRO、ENDO、FLA_REPRO。

## 4. 正式研究阶段计划

### Phase 0：项目文档与仓库同步

产物：`README.md`, `docs/PROJECT_PLAN_AND_PROGRESS.md`, `docs/MODEL_ARCHITECTURE.md`, `docs/TECHNICAL_ROUTE.md`。

资源：登录节点。预计时间：0.5–1 h。状态：进行中。

### Phase 1：正式输入数据构建

1. 解包 PED/MAP：1–2 CPU / 2–4G RAM / 4.5G disk / 10–30 min。
2. PED/MAP 转 `X_uint8.npy` 或 zarr：30 CPU / 80–150G RAM / 10–20G disk / 1–4 h。
3. phenotype 标准化：4 CPU / 8G RAM / <0.5 h。
4. 构建 random/subpopulation/region splits：1 CPU / 2G RAM / <10 min。

验收：`X`、`Y`、mask、sample_order 完全对齐；split 无泄漏；至少 10 个 high-coverage traits 入选。

### Phase 2：train-fold GWAS 与 p-value 先验

每个 split/trait 只用 train samples 生成 p-values，并加入 genotype PCs。

资源：每 trait 30 CPU / 100–150G RAM / 0.5–2 h；首批 10 traits 6–18 h。

验收：每个 trait 有 `pvalues.npy` 和 `gwas_summary.tsv.gz`；pvalue 维度与 SNP 顺序一致。

### Phase 3：SNP-to-gene mapping

主窗口 gene body ±5kb。消融：body only、±2kb、±10kb。

资源：8 CPU / 16G RAM / 10–40 min。

验收：`gene2snps.pkl`、`snp2gene.tsv`、`gene_metadata.tsv` 与 genotype SNP index 对齐。

### Phase 4：rice gene graph 构建

图类型：STRING、pathway、co-expression、hybrid multi-relation、random degree-matched。

资源：16 CPU / 16–60G RAM / 1–4 h。

验收：graph gene IDs 与 `gene2snps` 交集明确，random graph control 完成。

### Phase 5：正式模型训练

主模型：CropPRSNet-MT-GPS。

配置：snp_kernels 32；hidden_dim 96/128；gnn_layers 3；batch_size 64–256；AdamW；lr 1e-4–3e-4；cosine warmup；AMP；200–500 epochs；early stopping 30–50。

资源：单模型单 split 1 GPU / 8–16 CPU / 60–120G RAM / 6–18 h；3 split 主模型 18–54 h。

### Phase 6：baseline 与消融

必须完成 rrBLUP/GBLUP、XGBoost/LightGBM、SNP-MLP、SNP2Gene no graph、STRING graph、pathway graph、hybrid graph、random graph、without SG-dropout、without p-value L1。

资源：baseline + ablation 2–5 天。

### Phase 7：解释性与下游任务

输出 gene attention、top candidate genes、trait gene network、pathway enrichment、known QTL overlap。下游任务包括 trait-specific candidate gene discovery、pathway interpretation、cross-subpopulation generalization、maize G2F G×E 迁移。

## 5. 总体时间预估

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

## 6. 当前阶段进展记录

- [2026-06-13 15:34:30 CST] 建立项目研究计划文档结构；明确 GitHub 只保存 README、计划进展、模型结构、技术路线，不上传数据/结果/日志/权重。
- [2026-06-13 15:00:00 CST] 下载 3K Rice phenotype 与 core v0.7 PLINK BED/BIM/FAM；验证 gzip/xlsx 完整，BED magic bytes 为 `6c 1b 01`，404,388 SNP × 3,024 samples 可解析。
- [2026-06-13 15:01:00 CST] 发现 v0.7 `.fam` 使用 B001/B002 样本 ID；phenotype 与 3K metadata 可通过 IRGC 编号 2,266/2,266 对接，但 v0.7 缺少 B-code→IRIS 映射，不作为主训练数据。
- [2026-06-13 15:19:00 CST] 下载 `3K_coreSNP-v2.1.plink.tar.gz`；验证 gzip 完整；解析得到 365,710 SNP × 3,000 samples，样本 ID 直接为 IRIS ID。
- [2026-06-13 15:25:00 CST] 完成 v2.1 PED 样本与 metadata / phenotype 对接；PED vs metadata：3,000/3,000；PED vs phenotype：2,266/2,266；确认 v2.1 作为第一阶段正式主 genotype 数据。

- [2026-06-13 15:53:45 CST] 根据 phenotype_trait_summary.tsv 确认当前 3K Rice 表型主要是分类/有序等级 descriptor code；将主模型从通用 CropPRSNet-MT-GPS 优化为 CropPRSNet-Ordinal-MTL：低秩 SNP2Gene projection、2 层 residual GIN、trait-conditioned attention readout、ordinal cumulative-link heads、binary heads 和 nominal categorical heads；详见 docs/MODEL_ARCHITECTURE.md 第 8 节。

## 7. 下一步执行优先级

1. 写 `scripts/slurm/preprocess_ped_to_npy.sh` 和 Python 流式 PED 转换脚本。
2. 提交 CPU 作业到 q07/q08，把 PED/MAP 转为 `X_uint8.npy` 或 zarr。
3. 解析 phenotype dictionary，确定首批 10–20 个正式 traits。
4. 构建 random/subpopulation/region splits。
5. 在 train fold 内生成 pvalues。
6. 下载 rice annotation，构建 gene2snps。
