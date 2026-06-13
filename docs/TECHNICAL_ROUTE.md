# CropPRSNet 技术路线与 SLURM 执行计划

更新时间：2026-06-13 15:53:45 CST

## 1. 当前计算环境约束

- 当前在 SLURM 登录节点；登录节点只做轻量检查、写脚本、提交 sbatch。
- CPU 任务提交到 `q07` 或 `q08`；如果 q07/q08 无资源，再用 `cu`。
- 每个 CPU 计算节点最多申请 30 cores、150G RAM；每轮最多提交 6 个命令。
- 使用 mamba 环境：`PRSNet`。
- GPU 训练在 gpu10 上执行，命令必须用 `CUDA_VISIBLE_DEVICES=`，不使用 `--gpu`。

## 2. 数据路径规划

```text
/home/user/zhangzhishuai/myhermes/PRSNet/
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

phenotype：`raw/phenotype.xlsx`，2,266 rows × 53 columns。

## 4. 数据预处理正式流程

### Step 1：解包 PED/MAP 到 interim

输出：`interim/3K_coreSNP-v2.1.map`, `interim/3K_coreSNP-v2.1.ped`。

资源估计：1–2 CPU / 2–4G RAM / disk 4.5G / wall 10–30 min / network none。

### Step 2：PED/MAP 转 dosage 矩阵

输出：`processed/genotype/sample_order.tsv`, `snp_metadata.tsv`, `X_uint8.npy` 或 `X.zarr`。

编码：0=homozygous allele1, 1=heterozygous, 2=homozygous allele2, 3=missing。

资源估计：30 CPU / 80–150G RAM / disk 10–20G / wall 1–4 h / network none。正式策略是流式读取 PED，避免一次性加载 4.39G text。

### Step 3：phenotype 解析与 trait 筛选

输出：`phenotype_matrix.tsv`, `trait_metadata.tsv`, `Y_multitask.npy`, `mask_multitask.npy`。

筛选规则：非缺失样本数 ≥1,500；类别型每类 ≥50；类别数不超过 20；记录 dictionary meaning；按 trait type 分配 loss。

资源估计：4 CPU / 8G RAM / wall 10–30 min。

### Step 4：train/val/test 划分

三套 split：random stratified、subpopulation holdout、region holdout。

资源估计：1 CPU / 2G RAM / wall <10 min。

### Step 5：train-fold GWAS 生成 pvalues.npy

严禁全数据 GWAS 后再划分。每个 split、每个 trait 仅用 train samples 做 GWAS，并加入前 10 genotype PCs。

输出：`processed/gwas/{split}/{trait}/pvalues.npy` 和 `gwas_summary.tsv.gz`。

资源估计：每 trait 30 CPU / 100–150G RAM / wall 0.5–2 h；首批 10 traits 约 6–18 h，可并行但每轮最多 6 个 sbatch。

### Step 6：SNP-to-gene mapping

主窗口：gene body ±5kb。输出：`snp2gene.tsv`, `gene2snps.pkl`, `gene_metadata.tsv`, `mapped_snp_indices.npy`。

资源估计：8 CPU / 16G RAM / wall 10–40 min。

### Step 7：gene graph 构建

图类型：STRING rice score≥700/800、pathway graph、co-expression graph、hybrid multi-relation graph、random degree-matched graph。

资源估计：16 CPU / 16–60G RAM / wall 1–4 h / network required for downloads。

## 5. 模型训练正式方案

主模型：CropPRSNet-MT-GPS。

```yaml
snp_kernels: 32
hidden_dim: 96 or 128
gnn_layers: 3
message_passing: GraphGPS / GIN+GATv2 hybrid
readout: trait-conditioned attentive pooling
heads: multitask regression/classification heads
regularization: sg_dropout + pvalue_l1 + graph_smooth + attention_sparse
optimizer: AdamW
lr: 1e-4 to 3e-4
scheduler: cosine with warmup
batch_size: 64-256
max_epochs: 200-500
early_stopping: patience 30-50
precision: mixed precision fp16/bf16 when stable
```

## 6. 训练资源与时间预估

CPU 阶段：1–2 天内完成首批正式输入数据构建。

GPU 阶段：单 split 主模型 6–18 h；3 splits 主模型 18–54 h；baseline + ablation 2–5 天；完整第一轮正式结果约 4–7 天。

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

CPU cu fallback：

```bash
sbatch -p cu -c 30 --mem=150G scripts/slurm/preprocess_ped_to_npy.sh
```

GPU 训练：在 gpu10 上执行：

```bash
CUDA_VISIBLE_DEVICES=0 python scripts/train_crop_prsnet.py --config configs/3krice/crop_prsnet_mt_gps.yaml
```

资源估计：1×A100 40G / CPU 8–16 / RAM 60–120G / disk 20–50G / wall 6–18 h / network none。

## 8. GitHub 更新规则

每个小阶段只更新 `docs/PROJECT_PLAN_AND_PROGRESS.md`。每条进展格式：`[YYYY-MM-DD HH:MM:SS TZ] 做了什么；产物路径；验证结果；下一步。`

GitHub 只提交 README 和 docs；不提交 data/results/logs/checkpoints/*.npy/*.bed/*.ped/*.tar.gz/*.xlsx。
