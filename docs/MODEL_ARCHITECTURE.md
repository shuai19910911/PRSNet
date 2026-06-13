# PRSNet-2 与 CropPRSNet 模型结构解析

更新时间：2026-06-13 15:53:45 CST

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


## 8. 基于当前 3K Rice 表型特征的架构优化方案

更新时间：2026-06-13 15:53:45 CST

### 8.1 当前数据对模型设计的约束

当前 phenotype 表不是标准连续产量/株高原始测量值，而是 IRRI rice descriptor code：

```text
样本：2,266 phenotype-overlap accessions
表型：47 个 phenotype fields
类型：多数为分类 / 有序等级 / 少数二分类
SNP：365,710 core SNPs, 3,000 genotype samples, 2,266 可与 phenotype 对齐
```

因此不能把所有表型粗暴当作普通连续回归。最优架构应从“连续回归大模型”调整为：

```text
多任务 + 混合表型类型 + 有序等级感知 + 缺失 mask + trait-conditioned readout
```

### 8.2 表型类型分组

#### A. 强有序农艺等级 traits，主训练任务

这些是第一优先级，因为与育种相关，且数值等级有明确生物顺序：

```text
CULT_CODE_REPRO   culm length / 株高等级，7 类，n=2094
LLT_CODE          leaf length / 叶长等级，5 类，n=2095
PLT_CODE_POST     panicle length / 穗长等级，4 类，n=2090
SDHT_CODE         seedling height / 苗高等级，3 类，n=2093
CUNO_CODE_REPRO   culm number / 分蘖数量等级，3 类，n=2095
CUDI_CODE_REPRO   culm diameter / 茎秆直径等级，2 类，n=2111
SPKF              spikelet fertility / 小穗育性等级，5 类，n=2264
CUST_REPRO        culm strength / 茎秆强度抗倒伏等级，8 类，n=2264
PTH               panicle threshability / 脱粒性，3 类，n=2262
PSH               panicle shattering / 落粒性，3 类，n=1526
```

这些任务采用 ordinal-aware head，而不是普通 softmax head。

#### B. 姿态/结构类有序 traits，第二主任务

```text
CUAN_REPRO        culm angle，5 类，n=2264
FLA_REPRO         flag leaf angle，4 类，n=2262
LA                leaf angle，10 类，n=1528
PEX_REPRO         panicle exsertion，5 类，n=2265
PTY               panicle type，10 类，n=2263
SECOND_BR_REPRO   secondary branching，3 类，n=1522
```

这些保留 ordinal 或 semi-ordinal head，并在 multitask 中权重略低于 A 组。

#### C. 颜色/形态类别 traits，辅助任务

```text
AUCO_REV_VEG, BLCO_REV_VEG, AWCO_REV, INCO_REV_REPRO,
BLSCO_REV_VEG, LIGCO_REV_VEG, CCO_REV_VEG, LPCO_REV_POST,
APCO_REV_REPRO, SCCO_REV, SLCO_REV 等
```

这些类别通常不是严格线性顺序，适合 categorical head。它们可作为辅助任务提高 backbone 对亚群和形态背景的表征，但不作为核心性能结论。

#### D. 低覆盖 traits

低于 1,000 样本的 traits 不进入第一阶段主训练，只用于后续 few-shot / transfer 分析。

### 8.3 优化后的正式模型：CropPRSNet-Ordinal-MTL

针对当前数据，正式主模型从原计划的通用 CropPRSNet-MT-GPS 调整为：

```text
CropPRSNet-Ordinal-MTL
```

核心结构：

```text
Genotype X_uint8 / dosage
  → SNP2Gene encoder with train-fold GWAS SG-dropout
  → low-rank gene-specific projection
  → rice gene graph encoder
  → trait-conditioned attention readout
  → three head families:
       1) ordinal cumulative-link heads
       2) binary heads
       3) nominal categorical heads
  → masked multitask loss
```

### 8.4 SNP2Gene encoder 优化

当前样本量只有 2,266，不能使用过大的 gene-specific full projection 造成过拟合。采用低秩 gene projection：

```text
u_gene: n_kernels
A_gene: gene embedding, d_rank
B: shared kernel-to-hidden tensor, d_rank × n_kernels × d_hidden
W_gene = A_gene @ B
```

建议参数：

```text
n_snp_kernels = 16 或 24，第一正式主模型用 24
d_hidden = 96
d_rank = 8 或 16
gene_window = gene body ±5kb
sg_dropout_init = 0.9
sg_dropout_min = 0.15
sg_l1 = 100, 300, 1000 网格
```

理由：

- 365,710 SNP × 24 kernels 约 8.8M SNP filter 参数，可接受。
- 低秩 projection 避免 70M+ gene-specific projection 过拟合。
- d_hidden=96 比 128 稳，更适合 2,266 样本。

### 8.5 Gene graph encoder 优化

当前 phenotype 多为 descriptor code，强受群体结构和形态类别影响。graph encoder 不宜太深，避免 over-smoothing。

主配置：

```text
gnn_layers = 2
encoder = GIN + residual + layer norm
hidden_dim = 96
dropout = 0.15
edge_dropout = 0.10
```

增强配置用于第二轮：

```text
encoder = GPS-style local GIN + global Performer attention
gnn_layers = 2
```

第一轮不建议 3–4 层深 GNN，因为 2,266 样本 + descriptor labels 容易图过平滑和标签泄漏式拟合群体结构。

### 8.6 Trait-conditioned attention readout

不同表型依赖不同基因模块，因此 readout 必须 trait-conditioned：

```text
trait_embedding e_t
attention_score(g,t) = MLP([h_g, e_t, h_g * e_t])
h_trait = sum_g softmax_or_sparsemax(attention_score(g,t)) * V(h_g)
```

建议使用 sparsemax / entmax attention 做解释性增强：

```text
attention = entmax15(scores)
```

如果实现复杂，第一版用 softmax + entropy penalty。

### 8.7 输出头设计

#### Ordinal cumulative-link head

对有序 K 类 trait，不用普通 CrossEntropy，而用 K-1 个阈值：

```text
score_t = Linear(h_trait)
P(y > k) = sigmoid(score_t - threshold_k)
```

loss：ordinal BCE / cumulative link loss。

优点：

- 保留等级顺序。
- 把 “1 类错成 2 类” 与 “1 类错成 7 类” 区分开。
- 适合 CULT_CODE_REPRO、LLT_CODE、PLT_CODE_POST、SPKF 等。

#### Binary head

对二分类 trait：

```text
BCEWithLogitsLoss(pos_weight=class_balance)
```

#### Nominal categorical head

对颜色类无序 trait：

```text
CrossEntropyLoss(class_weight=effective_number_weight)
```

### 8.8 Loss 设计

```text
L_total = L_ordinal + L_binary + L_nominal
          + lambda_sg_l1 * L_pvalue_l1
          + lambda_attn * L_attention_entropy
          + lambda_pc_adv * L_population_adversarial_optional
```

任务权重：

```text
A组核心农艺等级 traits: weight 1.0
B组结构姿态 traits: weight 0.7
C组颜色辅助 traits: weight 0.3
低覆盖 traits: 第一阶段不用
```

缺失值处理：

```text
每个 trait 单独 mask；只对非缺失样本计算该 trait loss。
```

类别不平衡处理：

```text
effective number of samples weighting
或 inverse sqrt class frequency
```

### 8.9 群体结构控制

3K rice 亚群结构强，必须避免模型只学 indica/japonica 背景。

输入中加入 population covariates：

```text
metadata subgroup / region / top genotype PCs
```

但主模型不直接把 subgroup 当捷径特征。建议两种评估方式：

1. prediction model：允许 PC covariates 拼接到 readout，用于最大预测性能。
2. genetics-only model：不输入 subgroup，只在 split 和评估中控制，用于证明 genotype graph 能力。

可选 adversarial deconfounding：

```text
h_trait → phenotype head
h_trait → subgroup adversarial head with gradient reversal
```

第一阶段先做 PC-corrected GWAS pvalue + subgroup holdout，不立即加 adversarial，避免复杂化。

### 8.10 推荐第一版正式配置

```yaml
model: CropPRSNet-Ordinal-MTL
sample_count: 2266
snp_count: 365710 before gene mapping
trait_groups:
  core_ordinal: [CULT_CODE_REPRO, LLT_CODE, PLT_CODE_POST, SDHT_CODE, CUNO_CODE_REPRO, CUDI_CODE_REPRO, SPKF, CUST_REPRO, PTH, PSH]
  structure_ordinal: [CUAN_REPRO, FLA_REPRO, LA, PEX_REPRO, PTY, SECOND_BR_REPRO]
  auxiliary_nominal: [AUCO_REV_VEG, BLCO_REV_VEG, AWCO_REV, BLSCO_REV_VEG, APCO_REV_REPRO, SCCO_REV, ENDO]
encoder:
  snp_kernels: 24
  hidden_dim: 96
  gene_projection: low_rank
  rank: 16
  gene_window: ±5kb
graph:
  type: rice_string_first_then_hybrid
  layers: 2
  conv: GIN
  residual: true
  norm: layernorm
  dropout: 0.15
  edge_dropout: 0.10
readout:
  trait_conditioned_attention: true
  attention: softmax_with_entropy_penalty_first; entmax_second
heads:
  ordinal: cumulative_link
  binary: bce_logits
  nominal: weighted_cross_entropy
optimization:
  optimizer: AdamW
  lr: 2e-4
  weight_decay: 1e-4
  batch_size: 64 or full-batch gene graph with sample minibatch
  epochs: 300
  early_stop_patience: 40
  amp: true
```

### 8.11 预期相比原 PRSNet-2 的改进

1. 不再把 ordinal descriptor 当连续值硬回归，减少目标定义错误。
2. 多任务共享 backbone，缓解单 trait 样本不足。
3. trait-conditioned attention 允许不同表型有不同候选 gene。
4. 低秩 gene projection 显著降低过拟合风险。
5. subgroup/region holdout 可验证真实泛化，而不是随机划分虚高。
