# RiceGeneFormer-OMTL 模型结构设计

更新时间：2026-06-13 16:12:27 CST

## 1. 模型定位

`RiceGeneFormer-OMTL` 是面向作物 genotype-to-phenotype prediction 的自研基因组表征模型。

全称：Rice Gene-aware Ordinal Multi-Task Transformer。

第一阶段针对 3K Rice Genome / SNP-Seek 数据的真实约束设计：

```text
samples with phenotype: 2,266
core SNPs: 365,710
genotype samples: 3,000
phenotype fields: 47
phenotype type: ordinal descriptor / nominal categorical / binary
```

核心目标不是单性状测试，而是构建正式的多任务、有序等级感知、可解释、可跨亚群评估的水稻基因组预测框架。

## 2. 总体架构

```text
X_uint8 genotype dosage
  │
  ├─ VariantGate
  │    SNP-level allele filters
  │    train-fold GWAS p-value gating
  │    MAF/missingness-aware dropout
  │
  ├─ GeneBag Aggregator
  │    gene body ±5kb SNP grouping
  │    distance / p-value / dosage aware attention pooling
  │    low-rank gene-specific projection
  │
  ├─ ChrFormer
  │    chromosome-wise gene ordering
  │    local attention over neighboring genes
  │    LD block and cis-context modeling
  │
  ├─ RiceGraphEncoder
  │    PPI / pathway / co-expression multi-relation graph
  │    relation-aware graph message passing
  │    graph residual and edge dropout
  │
  ├─ GatedFusion
  │    VariantGate-GeneBag token
  │    chromosome context token
  │    graph context token
  │
  ├─ TraitQueryDecoder
  │    per-trait query embedding
  │    cross-attention to fused gene tokens
  │    sparse / entropy-regularized gene attention
  │
  └─ Multi-head phenotype decoder
       ordinal cumulative-link heads
       binary BCE heads
       nominal weighted CE heads
```

核心改造点：不再只做 SNP→gene→graph→global readout，而是加入染色体局部上下文、trait query cross-attention、多关系知识图谱、ordinal-aware phenotype heads 和无泄漏统计遗传先验。

## 3. 输入数据定义

### 3.1 Genotype

```text
X: samples × SNPs
encoding:
  0 = homozygous allele1
  1 = heterozygous
  2 = homozygous allele2
  3 = missing
```

第一阶段主输入：3K coreSNP v2.1，365,710 SNP × 3,000 samples；与 phenotype 对齐后使用 2,266 samples。

### 3.2 SNP metadata

每个 SNP 保留：

```text
snp_id
chromosome
position
allele1
allele2
MAF
missing_rate
train_fold_pvalue_per_trait
```

p-value 只能来自对应 split 的 training samples，不能用全数据 GWAS。

### 3.3 Gene metadata

水稻 gene annotation 使用 IRGSP-1.0 / MSU7 坐标体系。

主窗口：

```text
gene window = gene body ± 5 kb
```

消融窗口：

```text
gene body only
±2 kb
±5 kb
±10 kb
nearest-gene assignment
```

### 3.4 Trait metadata

每个 trait 记录：

```text
trait_name
trait_group
trait_type: ordinal / binary / nominal
class_count
class_dictionary
n_nonmissing
class_frequency
loss_weight
metric_set
```

缺失 phenotype 通过 trait-level mask 处理。

## 4. VariantGate：SNP 显著性门控编码器

VariantGate 负责把原始 SNP dosage 转成若干全基因组 variant response channel。

对样本 i、SNP s、kernel k：

```text
z_i,s,k = dosage_i,s × w_s,k × g_s,t
```

其中：

```text
w_s,k: learnable SNP filter
g_s,t: trait-aware significance gate
```

significance gate 使用 train-fold GWAS p-value：

```text
score_s,t = -log10(p_s,t + eps)
g_s,t = clamp(normalize(score_s,t), g_min, g_max)
```

训练期 dropout：

```text
drop_prob_s,t = base_drop / (score_s,t + c)
drop_prob_s,t = clamp(drop_prob_s,t, min_drop, max_drop)
```

设计目的：

1. 显著 SNP 更稳定保留。
2. 非显著 SNP 更强正则，减少 2,266 样本下的过拟合。
3. 所有 p-value 都在 train fold 内生成，避免信息泄漏。
4. kernel 数量限制在 24，避免对 365,710 SNP 直接构建过大 dense projection。

推荐配置：

```yaml
variant_gate:
  n_kernels: 24
  sg_dropout_init: 0.9
  sg_dropout_min: 0.15
  sg_dropout_max: 0.95
  pvalue_eps: 1.0e-12
```

## 5. GeneBag：SNP-to-gene attention 聚合

GeneBag 将落入同一 gene window 的 SNP response 聚合为 gene token。

对 gene g 的 SNP 集合 S_g：

```text
a_i,s,g = Attention([z_i,s,* ; distance_s,g ; pvalue_s,t ; MAF_s ; missing_s])
u_i,g = sum_{s in S_g} softmax(a_i,s,g) × z_i,s,*
```

相比简单求和，GeneBag 显式利用：

1. SNP 到 gene body / TSS 的距离。
2. train-fold p-value。
3. MAF 与 missingness。
4. genotype dosage response。
5. 可扩展 functional annotation，例如 CDS/intron/promoter。

### 5.1 低秩 gene-specific projection

直接为每个 gene 学一个 full projection 容易过拟合。RiceGeneFormer 使用低秩 gene projection：

```text
A_g ∈ R^rank
B ∈ R^(rank × n_kernels × hidden_dim)
W_g = A_g @ B
h_i,g = u_i,g @ W_g + e_g
```

推荐配置：

```yaml
gene_bag:
  hidden_dim: 96
  rank: 16
  gene_window: plus_minus_5kb
  attention_hidden: 64
  dropout: 0.15
```

优势：

1. 既允许 gene-specific effect，又不会产生过大的每基因独立投影矩阵。
2. 对 2,266 phenotype-overlap samples 更稳。
3. 后续可以加入 gene family / pathway prior 初始化 A_g。

## 6. ChrFormer：染色体局部上下文编码器

植物基因组预测不能只依赖 gene graph，因为同染色体 LD block、相邻 cis-regulatory region 和 haplotype neighborhood 对表型有重要影响。

ChrFormer 按 chromosome 和 genomic position 排序 gene token，在每条染色体内做局部 attention：

```text
H_chr = LocalAttention(H_gene_ordered, window = 64 or 128 genes)
```

设计约束：

1. 不做全基因组 full attention，避免 O(n_genes^2) 成本。
2. 每条染色体独立编码，避免人为连接不同染色体。
3. 使用 relative genomic distance bias。
4. 使用 residual + layernorm。

推荐配置：

```yaml
chr_former:
  layers: 2
  attention_type: local
  window_genes: 64
  hidden_dim: 96
  num_heads: 4
  relative_position_bias: true
  dropout: 0.15
```

## 7. RiceGraphEncoder：水稻基因知识图谱编码器

RiceGraphEncoder 建模 gene-gene biological relationship。

候选关系：

```text
relation_1: STRING / PPI / cofunctional edge
relation_2: Plant Reactome or KEGG pathway co-membership
relation_3: public rice RNA-seq atlas co-expression
relation_4: optional orthology-supported functional edge
```

主模型采用多关系图编码：

```text
h_g^(l+1) = FFN(
  h_g^(l)
  + sum_r alpha_r × MessagePassing_r(h_g^(l), N_r(g))
)
```

实现可从 relation-aware GAT / R-GCN 起步；后续增强为 GraphGPS-style local graph + global attention。

推荐配置：

```yaml
rice_graph_encoder:
  graph_type: hybrid_multi_relation
  layers: 2
  hidden_dim: 96
  relation_types: [ppi, pathway, coexpression]
  edge_dropout: 0.10
  residual: true
  norm: layernorm
  activation: gelu
```

不建议第一版使用 4 层以上深图网络，因为 2,266 样本 + descriptor labels 容易过平滑。

## 8. GatedFusion：variant / chromosome / graph 三路融合

对每个 gene g，融合三种表征：

```text
h_gene_raw: GeneBag output
h_chr: ChrFormer output
h_graph: RiceGraphEncoder output
```

使用 trait-aware gate：

```text
α_t,g = softmax(MLP([h_gene_raw, h_chr, h_graph, trait_embedding_t]))
h_fused_t,g = α1*h_gene_raw + α2*h_chr + α3*h_graph
```

意义：

1. 有些 trait 更依赖局部 LD / haplotype context。
2. 有些 trait 更依赖 pathway / gene network。
3. 有些颜色或形态 descriptor 可能主要反映群体结构，gate 可辅助诊断。

## 9. TraitQueryDecoder：表型条件解码器

每个 trait t 维护一个 query embedding：

```text
q_t ∈ R^hidden_dim
```

通过 cross-attention 从所有 gene tokens 中读取 trait-specific representation：

```text
score_t,g = q_t^T W h_fused_t,g / sqrt(d)
a_t,g = sparse_attention(score_t,g)
h_t = sum_g a_t,g × V(h_fused_t,g)
```

第一版实现：softmax + entropy penalty。

第二版增强：entmax15 / sparsemax，提高解释性和候选基因稀疏度。

输出：

```text
h_t: trait-specific phenotype representation
a_t,g: trait-specific gene attention for interpretation
```

优势：

1. 不同 trait 学不同 candidate gene module。
2. 避免所有表型共用一个全局 readout。
3. attention 可用于 candidate gene ranking 与 pathway enrichment。

## 10. 输出头设计

### 10.1 Ordinal cumulative-link head

对有序 K 类 trait，不用普通 CrossEntropy，而使用 K-1 个阈值：

```text
score_t = Linear(h_t)
P(y > k) = sigmoid(score_t - threshold_t,k), k = 1..K-1
```

loss：ordinal BCE / cumulative link loss。

适用 trait：

```text
CULT_CODE_REPRO
LLT_CODE
PLT_CODE_POST
SDHT_CODE
CUNO_CODE_REPRO
CUDI_CODE_REPRO
SPKF
CUST_REPRO
PTH
PSH
CUAN_REPRO
FLA_REPRO
PEX_REPRO
PTY
```

优势：

1. 保留等级顺序。
2. 把相邻等级错误与远距离等级错误区分开。
3. 更符合 IRRI descriptor code 的数据本质。

### 10.2 Binary head

对二分类 trait：

```text
logit_t = Linear(h_t)
loss_t = BCEWithLogitsLoss(pos_weight = class_balance)
```

### 10.3 Nominal categorical head

对无严格顺序的颜色/形态类别：

```text
logits_t = Linear(h_t)
loss_t = CrossEntropyLoss(class_weight = effective_number_weight)
```

适用：颜色、形态状态、胚乳类型等 nominal traits。

## 11. Loss 设计

总 loss：

```text
L_total = Σ_t mask_t × w_t × L_trait_t
          + λ_pvalue × L_pvalue_weighted_l1
          + λ_attn × L_attention_sparse
          + λ_graph × L_graph_consistency
          + λ_calib × L_calibration_optional
```

### 11.1 Trait loss

```text
ordinal traits: cumulative-link ordinal loss
binary traits: balanced BCE
nominal traits: weighted CE
```

任务权重：

```text
core agronomic ordinal traits: 1.0
structure/posture ordinal traits: 0.7
auxiliary nominal traits: 0.3
low coverage traits: not used in first stage
```

### 11.2 p-value-weighted L1

```text
L_pvalue_weighted_l1 = Σ_s,k |w_s,k| × normalized_pvalue_weight_s
```

显著 SNP 惩罚较弱，非显著 SNP 惩罚较强。

### 11.3 Attention sparse penalty

第一版：entropy penalty。

第二版：entmax / sparsemax 后降低或移除 entropy penalty。

### 11.4 Graph consistency

轻量 graph consistency 约束相邻 gene 表征，但权重必须小，避免 graph over-smoothing。

## 12. 群体结构控制与评估 split

3K Rice 亚群结构强，随机划分可能产生虚高结果。因此必须至少使用三类 split：

```text
random stratified split
subpopulation holdout split
region holdout split
```

模型设置分两类报告：

1. `genetics_only`：不把 subgroup/region 作为输入，只在 split 与评估中控制。
2. `pc_corrected`：GWAS p-value 使用 train-fold top genotype PCs；可在 readout 末端加入 PCs 作为 covariate。

第一阶段暂不强制加入 adversarial subgroup head，避免模型过度复杂；第二阶段可加入 gradient reversal 做 deconfounding 消融。

## 13. 推荐第一版正式配置

```yaml
model:
  name: RiceGeneFormer-OMTL
  sample_count: 2266
  snp_count_before_mapping: 365710
  phenotype_fields: 47

trait_groups:
  core_ordinal:
    - CULT_CODE_REPRO
    - LLT_CODE
    - PLT_CODE_POST
    - SDHT_CODE
    - CUNO_CODE_REPRO
    - CUDI_CODE_REPRO
    - SPKF
    - CUST_REPRO
    - PTH
    - PSH
  structure_ordinal:
    - CUAN_REPRO
    - FLA_REPRO
    - LA
    - PEX_REPRO
    - PTY
    - SECOND_BR_REPRO
  auxiliary_nominal:
    - AUCO_REV_VEG
    - BLCO_REV_VEG
    - AWCO_REV
    - BLSCO_REV_VEG
    - APCO_REV_REPRO
    - SCCO_REV
    - ENDO

variant_gate:
  snp_kernels: 24
  significance_gate: train_fold_gwas_pvalue
  sg_dropout_init: 0.9
  sg_dropout_min: 0.15
  sg_l1_grid: [100, 300, 1000]

gene_bag:
  hidden_dim: 96
  projection: low_rank
  rank: 16
  gene_window: plus_minus_5kb
  attention_features:
    - dosage_response
    - minus_log10_pvalue
    - distance_to_gene
    - maf
    - missing_rate

chr_former:
  layers: 2
  attention: local
  window_genes: 64
  heads: 4
  relative_position_bias: true

rice_graph_encoder:
  type: hybrid_multi_relation
  relation_types: [ppi, pathway, coexpression]
  layers: 2
  residual: true
  norm: layernorm
  dropout: 0.15
  edge_dropout: 0.10

fusion:
  type: trait_aware_gated_fusion

trait_decoder:
  layers: 2
  attention: softmax_with_entropy_penalty_first
  sparse_attention_second_stage: entmax15

heads:
  ordinal: cumulative_link
  binary: bce_logits
  nominal: weighted_cross_entropy

optimization:
  optimizer: AdamW
  lr: 2.0e-4
  weight_decay: 1.0e-4
  batch_size: 64_to_256
  epochs: 300
  early_stop_patience: 40
  amp: true
```

## 14. 参数规模预估

```text
SNP filters: 365,710 × 24 ≈ 8.8M
GeneBag attention and low-rank projection: 10–20M
ChrFormer: 5–15M
RiceGraphEncoder: 5–10M
TraitQueryDecoder: 2–5M
Heads: <1M
Total: about 30–60M parameters
```

这个规模比直接构建巨大 gene-specific dense projection 更稳，更适合 2,266 phenotype-overlap samples。

## 15. 评估指标

### Ordinal traits

```text
macro-F1
balanced accuracy
ordinal MAE
quadratic weighted kappa
Spearman rho
calibration error
```

### Binary traits

```text
AUROC
AUPRC
balanced accuracy
macro-F1
```

### Nominal traits

```text
macro-F1
balanced accuracy
weighted CE
confusion matrix
```

### 泛化能力

```text
random split performance
subpopulation holdout performance
region holdout performance
performance drop from random to holdout
```

### 生物解释性

```text
trait-specific top attention genes
known QTL / candidate gene overlap
pathway enrichment
graph ablation: hybrid graph vs random graph vs no graph
attention stability across folds
```

## 16. Baseline 与消融实验

必须包含：

1. rrBLUP / GBLUP。
2. XGBoost / LightGBM SNP baseline。
3. SNP-MLP baseline。
4. GeneBag only。
5. GeneBag + ChrFormer。
6. GeneBag + RiceGraphEncoder。
7. GeneBag + ChrFormer + RiceGraphEncoder。
8. STRING/PPI graph only。
9. pathway graph only。
10. co-expression graph only。
11. hybrid graph。
12. random degree-matched graph negative control。
13. without significance gate。
14. without p-value weighted L1。
15. without trait-query decoder。
16. ordinary CrossEntropy vs ordinal cumulative-link。
17. gene window body / ±2kb / ±5kb / ±10kb。

## 17. 后续扩展

1. `RiceGeneFormer-GxE`：加入 environment encoder，用于 maize G2F 多环境预测。
2. `RiceGeneFormer-FS`：低覆盖 trait few-shot transfer。
3. `RiceGeneFormer-XAI`：candidate gene、pathway、trait module 稳定性分析。
4. `PanCropGeneFormer`：跨作物 orthology-aware 迁移。
