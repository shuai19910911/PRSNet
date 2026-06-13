# RiceGeneFormer 技术路线与 SLURM 执行计划

更新时间：2026-06-13 16:12:27 CST

## 1. 当前计算环境约束

- 当前在 SLURM 登录节点；登录节点只做轻量检查、写脚本、提交 sbatch。
- CPU 任务提交到 `q07` 或 `q08`；如果 q07/q08 无资源，再用 `cu`。
- 每个 CPU 计算节点最多申请 30 cores、150G RAM；每轮最多提交 6 个命令。
- GPU 训练在 gpu10 上执行，命令必须用 `CUDA_VISIBLE_DEVICES=`，不使用 `--gpu`。
- GitHub 只提交 README 和 docs；不提交数据、结果、日志、权重或大文件。

## 2. 数据路径规划

```text
project_root/
  data/3krice/raw/          # 原始下载数据，不上传 GitHub
  data/3krice/checks/       # 校验报告，不上传 GitHub，必要信息写入 docs
  data/3krice/processed/    # 预处理输出，不上传 GitHub
  data/3krice/interim/      # 中间文件，不上传 GitHub
  src/                      # 后续代码
  scripts/                  # 后续 SLURM 脚本
  configs/                  # 训练配置
  docs/                     # GitHub 上传，仅计划/进展/模型结构/技术路线
```

## 3. 已确认数据

主 genotype：`raw/3K_coreSNP-v2.1.plink.tar.gz`，含 `3K_coreSNP-v2.1.map` 和 `3K_coreSNP-v2.1.ped`。

```text
SNPs: 365,710
samples: 3,000
sample IDs: IRIS_313-xxxxx / CXxxx
phenotype overlap: 2,266 / 2,266
```

phenotype：`raw/phenotype.xlsx`，2,266 rows × 53 columns，其中 47 个 phenotype fields。

## 4. 数据预处理正式流程

### Step 1：解包 PED/MAP 到 interim

输出：`interim/3K_coreSNP-v2.1.map`, `interim/3K_coreSNP-v2.1.ped`。

资源估计：1–2 CPU / 2–4G RAM / disk 4.5G / wall 10–30 min / network none。

### Step 2：PED/MAP 转 dosage 矩阵

输出：`processed/genotype/sample_order.tsv`, `snp_metadata.tsv`, `X_uint8.npy` 或 `X.zarr`。

编码：0=homozygous allele1, 1=heterozygous, 2=homozygous allele2, 3=missing。

正式策略：流式读取 PED，避免一次性加载 4.39G text。

资源估计：30 CPU / 80–150G RAM / disk 10–20G / wall 1–4 h / network none。

### Step 3：phenotype 解析与 trait 筛选

输出：`phenotype_matrix.tsv`, `trait_metadata.tsv`, `Y_multitask.npy`, `mask_multitask.npy`。

筛选规则：

1. 非缺失样本数 ≥1,500。
2. 类别型每类尽量 ≥50；极端稀有类别合并或排除。
3. 类别数不超过 20。
4. 记录 dictionary meaning。
5. 按 trait type 分配 loss：ordinal / binary / nominal。

资源估计：4 CPU / 8G RAM / wall 10–30 min / network none。

### Step 4：train/val/test 划分

三套 split：

```text
random stratified
subpopulation holdout
region holdout
```

资源估计：1 CPU / 2G RAM / wall <10 min / network none。

### Step 5：train-fold GWAS 生成 pvalues.npy

严禁全数据 GWAS 后再划分。每个 split、每个 trait 仅用 train samples 做 GWAS，并加入前 10 genotype PCs。

输出：`processed/gwas/{split}/{trait}/pvalues.npy` 和 `gwas_summary.tsv.gz`。

资源估计：每 trait 30 CPU / 100–150G RAM / wall 0.5–2 h；首批 10 traits 约 6–18 h，可并行但每轮最多 6 个 sbatch。

### Step 6：SNP-to-gene mapping

主窗口：gene body ±5kb。

输出：`snp2gene.tsv`, `gene2snps.pkl`, `gene_metadata.tsv`, `mapped_snp_indices.npy`。

资源估计：8 CPU / 16G RAM / wall 10–40 min / network none。

### Step 7：rice gene knowledge graph 构建

图类型：

```text
STRING rice score≥700/800
Plant Reactome / KEGG pathway co-membership
public rice RNA-seq atlas co-expression
hybrid multi-relation graph
random degree-matched graph
```

资源估计：16 CPU / 16–60G RAM / wall 1–4 h / network required for downloads。

## 5. RiceGeneFormer-OMTL 正式训练方案

主模型配置：

```yaml
model: RiceGeneFormer-OMTL
variant_gate:
  snp_kernels: 24
  significance_gate: train_fold_gwas_pvalue
  sg_dropout_init: 0.9
  sg_dropout_min: 0.15
gene_bag:
  gene_window: plus_minus_5kb
  hidden_dim: 96
  projection: low_rank
  rank: 16
chr_former:
  layers: 2
  local_attention_window_genes: 64
  heads: 4
rice_graph_encoder:
  graph_type: hybrid_multi_relation
  relation_types: [ppi, pathway, coexpression]
  layers: 2
fusion:
  type: trait_aware_gated_fusion
trait_decoder:
  layers: 2
  attention: softmax_with_entropy_penalty_first
heads:
  ordinal: cumulative_link
  binary: bce_logits
  nominal: weighted_cross_entropy
regularization:
  pvalue_weighted_l1: true
  attention_sparse_penalty: true
  graph_consistency: weak
optimization:
  optimizer: AdamW
  lr: 2e-4
  weight_decay: 1e-4
  scheduler: cosine_with_warmup
  batch_size: 64-256
  max_epochs: 300
  early_stopping: 40
  precision: amp
```

## 6. 训练资源与时间预估

CPU 阶段：1–2 天内完成首批正式输入数据构建。

GPU 阶段：

```text
single split main model: 1 GPU / 8–16 CPU / 60–120G RAM / 6–18 h
3 split main model: 18–54 h
baseline + ablation: 2–5 days
complete first-round result: 4–7 days after processed inputs are ready
```

## 7. SLURM 命令模板

CPU q07：

```bash
sbatch -p q07 -c 30 --mem=150G scripts/slurm/preprocess_ped_to_npy.sh
```

资源估计：30 CPU / 150G RAM / disk 10–20G / wall 1–4 h / network none。

CPU q08 fallback：

```bash
sbatch -p q08 -c 30 --mem=150G scripts/slurm/preprocess_ped_to_npy.sh
```

资源估计：30 CPU / 150G RAM / disk 10–20G / wall 1–4 h / network none。

CPU cu fallback：

```bash
sbatch -p cu -c 30 --mem=150G scripts/slurm/preprocess_ped_to_npy.sh
```

资源估计：30 CPU / 150G RAM / disk 10–20G / wall 1–4 h / network none。

GPU 训练：在 gpu10 上执行：

```bash
CUDA_VISIBLE_DEVICES=0 python scripts/train_rice_geneformer.py --config configs/3krice/rice_geneformer_omtl.yaml
```

资源估计：1×A100 40G / CPU 8–16 / RAM 60–120G / disk 20–50G / wall 6–18 h / network none。

## 8. 第一阶段正式验收标准

### 数据验收

1. genotype sample order 与 phenotype sample order 完全对齐。
2. `X_uint8.npy` / zarr 维度与 SNP metadata 一致。
3. trait metadata 明确记录 ordinal / binary / nominal 类型。
4. split 文件包含 random、subpopulation holdout、region holdout。
5. p-value 文件全部来自对应 train fold。

### 模型验收

1. RiceGeneFormer-OMTL 主模型可以完成至少 10 个 high-coverage traits 的 masked multi-task training。
2. 输出每个 trait 的 macro-F1、balanced accuracy、ordinal MAE / kappa。
3. 输出 random vs holdout 泛化差异。
4. 输出 trait-specific gene attention 排名。
5. 完成 random graph、no graph、no chromosome context、ordinary CE vs ordinal head 的关键消融。

### GitHub 验收

1. 只提交 README 和 docs。
2. 每个阶段更新 `docs/PROJECT_PLAN_AND_PROGRESS.md`。
3. 每条进展包含具体时间、产物、验证结果、下一步。
4. 不提交 data/results/logs/checkpoints/weights/binary genotype files。

## 9. 下一步执行顺序

1. 创建 PED/MAP 流式转换脚本。
2. 创建 phenotype trait metadata 构建脚本。
3. 创建 split 构建脚本。
4. 创建 train-fold GWAS 作业脚本。
5. 创建 SNP-to-gene mapping 脚本。
6. 创建 rice gene graph 构建脚本。
7. 实现 RiceGeneFormer-OMTL 训练代码。
8. 跑主模型、baseline、消融和解释性分析。
