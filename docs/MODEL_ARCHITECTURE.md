# PRSNet-2 与 CropPRSNet 模型结构解析

更新时间：2026-06-13 15:34:30 CST

## 1. PRSNet-2 原始模型结构

代码来源：`lihan97/PRSNet-2`，本地分析路径：`/home/user/zhangzhishuai/myhermes/PRSNet/PRSNet-2/PRSNet-2/src/model.py`。

原始 PRSNet-2 的主链路为：

```text
sample-level SNP dosage matrix X
  → SNP2Gene multi-kernel aggregator
  → gene-level hidden states H_gene
  → gene-gene interaction graph GIN message passing
  → attentive graph readout
  → phenotype prediction MLP
```

### 1.1 输入

原始代码要求：

```text
X.npy              samples × SNPs genotype dosage
Y.npy              samples labels；原代码默认二分类
pvalues.npy        SNP-level GWAS p-value
gene2snps.pkl      gene_id -> SNP index list
ggi_graph_800.bin  DGL gene-gene graph
```

原始任务是人类疾病二分类：

- loss：`BCEWithLogitsLoss`
- metric：AUROC / AUPRC
- graph：人类 gene-gene interaction，类似 STRING 高置信网络
- regularization：GWAS significance-guided dropout + p-value-weighted L1

### 1.2 SNP2Gene 多核聚合

核心类：`SNP2Gene`。

每个 SNP 输入为 dosage。模型维护 `n_snp_kernels` 个全基因组 SNP filter，每个 filter 对全部 SNP 有一组可训练标量权重：

```text
filter_k: R^(n_snps)
```

对每个样本、每个 SNP、每个 kernel：

```text
z(sample,snp,k) = dosage(sample,snp) × filter(k,snp)
```

随后按 `gene2snps` 聚合：

```text
u(sample,gene,k) = sum over snp in gene window of z(sample,snp,k)
```

再通过 gene-specific projection 将 kernel 维度映射到 hidden dimension：

```text
h_gene = u_gene × W_gene + e_gene
```

其中：

- `W_gene`: gene-specific projection，形状约为 `n_genes × n_snp_kernels × d_hidden`
- `e_gene`: gene embedding，形状为 `n_genes × d_hidden`

### 1.3 significance-guided dropout

训练时根据 SNP p-value 自适应 dropout：

```text
p_drop_snp = sg_dropout_init / (-log10(p_value_snp))
p_drop_snp = clamp(p_drop_snp, sg_dropout_min, 0.99)
```

意义：

- GWAS 越显著，`p_value` 越小，`-log10(p)` 越大，dropout 越低。
- 非显著 SNP 更容易被 dropout。
- 这将统计遗传先验注入端到端深度模型，降低小样本植物数据上的过拟合风险。

### 1.4 p-value-weighted L1 正则

原始 trainer 中 L1：

```text
L_sg_l1 = mean_k sum_snp |filter(k,snp)| × p_value_snp / sum_snp p_value_snp
```

显著 SNP p-value 小，惩罚弱；不显著 SNP 惩罚强。

总 loss：

```text
L = L_pred + lambda_sg_l1 × L_sg_l1
```

### 1.5 Gene-Gene GNN

原模型使用 DGL `GINConv`：

```text
h_g(l+1) = MLP_l((1+eps)h_g(l) + sum over u in N(g) h_u(l))
```

实现细节：

- aggregator：sum
- learn_eps：False
- 每层后：BatchNorm + GELU
- 默认层数：1，可扩展到 2–4 层

### 1.6 attention readout

`AttentiveReadout` 对每个 gene node 计算权重：

```text
a_g = sigmoid(w^T Linear(h_g))
v_g = Linear(h_g)
h_graph = sum_g a_g × v_g
```

输出：

- phenotype prediction
- gene attention weights，可用于候选基因解释。

## 2. 植物迁移关键变化

### 2.1 表型从疾病二分类改为植物多任务预测

水稻第一阶段正式目标不做“玩具测试”，直接做高覆盖 phenotype 的正式预测：

- ordinal/categorical traits：CrossEntropy / ordinal regression / macro-F1 / balanced accuracy
- numeric-coded agronomic traits：Huber/MSE / Pearson r / R² / RMSE
- 多任务版本：共享 SNP2Gene + GNN backbone，多 trait heads

正式训练不只训练单一性状；主模型采用多任务学习，单性状结果作为消融。

### 2.2 SNP-to-gene mapping

水稻使用 IRGSP-1.0/MSU7 坐标体系。正式定义：

```text
gene window = gene body ± 5 kb
```

消融：

- gene body only
- ±2 kb
- ±5 kb 主模型
- ±10 kb
- LD-pruned SNP subset vs all available v2.1 core SNP

如果 SNP 不落入任何 gene window：

- 主模型丢弃 intergenic orphan SNP，保证生物解释性；
- 扩展模型可加入 nearest-gene assignment。

### 2.3 plant gene graph

水稻主模型 graph 优先级：

1. STRING Oryza sativa high-confidence PPI/cofunctional graph，score ≥ 700 或 ≥ 800。
2. Rice pathway graph：Plant Reactome / KEGG pathway co-membership。
3. Rice co-expression graph：公开 RNA-seq atlas 构建 gene-gene Pearson/Spearman top-k graph。
4. Hybrid graph：PPI + pathway + co-expression 多关系图。

主模型建议采用多关系图神经网络：

```text
relation types = PPI, pathway, coexpression
```

如果先用 DGL 实现，正式主模型可采用：

- R-GCN / HGT-style relation-specific message passing；或
- GraphGPS-style local GNN + global attention；或
- GIN/GATv2 多图 ensemble。

### 2.4 CropPRSNet 正式架构

正式推荐架构：CropPRSNet-MT-GPS。

```text
X_genotype: samples × SNPs
p_snp: SNP p-values from train-fold GWAS
snp2gene: SNP→gene window mapping
G_gene: multi-relation rice gene graph
Trait metadata: trait type, dictionary class count

SNP2Gene encoder:
  multi-kernel SNP filters + significance-guided dropout
  gene-specific projection
  gene embedding

Gene graph encoder:
  2–3 GraphGPS/GIN/GATv2 layers
  residual connections
  layer norm / batch norm
  dropout 0.1–0.3

Readout:
  trait-conditioned attention readout
  optional Set Transformer / gated attention pooling

Heads:
  regression head for continuous/ordinal numeric traits
  classification head for categorical descriptor traits
  multitask uncertainty weighting or GradNorm loss balancing

Outputs:
  predictions
  gene attention per trait
  SNP kernel importance
  candidate gene ranking
```

## 3. 参数规模预估

以水稻 v2.1：

```text
SNPs: 365,710
samples: 3,000 genotype; 2,266 phenotype-overlap
mapped genes: 预计 25,000–40,000
n_snp_kernels: 16–32
d_hidden: 64–128
gnn_layers: 2–3
```

核心参数估算：

- SNP filters：`365,710 × 32 ≈ 11.7M`
- gene projection：`35,000 × 32 × 64 ≈ 71.7M`，若 d=64,k=32
- gene embedding：`35,000 × 64 ≈ 2.2M`
- GNN + heads：< 5M

主模型参数约：

```text
80M–110M parameters
```

如显存不足，采用低秩 gene projection：

```text
W_gene = A_gene × B_kernel
```

将 projection 参数从 ~72M 降到 ~10M–20M。

## 4. 训练目标

多任务总 loss：

```text
L_total = sum_t w_t L_t + lambda_sg_l1 L_sg_l1 + lambda_graph L_graph_smooth + lambda_attn L_attention_sparse
```

其中：

- `L_t`：每个 trait 的分类/回归 loss。
- `w_t`：uncertainty weighting 或 GradNorm 自适应任务权重。
- `L_sg_l1`：p-value-weighted SNP filter L1。
- `L_graph_smooth`：相邻 gene 表征平滑，但权重较小，避免过平滑。
- `L_attention_sparse`：促进解释性 gene attention 稀疏。

## 5. 评估指标

### 回归/ordinal numeric traits

- Pearson r
- Spearman rho
- R²
- RMSE
- MAE

### categorical traits

- balanced accuracy
- macro-F1
- AUROC/AUPRC for binary traits

### 生物解释性

- attention top genes 与已知 QTL/gene 富集
- GWAS significant region recovery
- pathway enrichment
- graph ablation: real graph > random graph > no graph

## 6. 消融实验

正式实验必须包含：

1. rrBLUP / GBLUP baseline。
2. XGBoost / LightGBM SNP baseline。
3. SNP-MLP baseline。
4. PRSNet-style no-graph。
5. CropPRSNet with PPI graph。
6. CropPRSNet with pathway graph。
7. CropPRSNet with hybrid graph。
8. without significance-guided dropout。
9. without p-value L1。
10. different gene windows：gene body / ±2kb / ±5kb / ±10kb。
11. random graph negative control。

## 7. 下游任务设计

1. 多性状 phenotype prediction。
2. candidate gene prioritization。
3. trait-specific gene network discovery。
4. pathway-level interpretation。
5. cross-subpopulation generalization：indica↔japonica↔aus。
6. 后续迁移到 maize G2F：加入 G×E environment encoder。
