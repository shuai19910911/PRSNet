# RiceGeneFormer

RiceGeneFormer 是面向作物 genotype-to-phenotype prediction 的自研基因组大模型框架。第一阶段以 3K Rice Genome / SNP-Seek 为核心数据，构建水稻 SNP→gene→chromosome context→gene knowledge graph→multi-trait phenotype 的正式训练体系。

当前定位：不是测试 demo；按可发表研究标准推进，重点解决 3K Rice 中小样本、强群体结构、多表型缺失、表型为分类/有序 descriptor code 的现实约束。

## 模型主线

正式主模型命名为：`RiceGeneFormer-OMTL`

全称：Rice Gene-aware Ordinal Multi-Task Transformer。

核心思想：

```text
genotype dosage
  → VariantGate SNP encoder
  → GeneBag SNP-to-gene aggregator
  → chromosome segment context encoder
  → rice gene knowledge graph encoder
  → trait-query cross-attention decoder
  → ordinal / binary / nominal multitask heads
```

与传统单性状 SNP 回归不同，RiceGeneFormer 直接把 365,710 个 core SNP 映射为 gene-level token，并同时建模：

1. SNP 显著性先验：只使用 train fold 内 GWAS p-value，避免数据泄漏。
2. 基因局部窗口：gene body ±5 kb，后续做 body only / ±2 kb / ±10 kb 消融。
3. 染色体局部上下文：捕获 LD block 与 cis-regulatory neighborhood。
4. 水稻基因知识图谱：整合 PPI、pathway、co-expression 多关系边。
5. 表型条件解码：每个 trait 用独立 trait query cross-attend 到 gene token。
6. 有序等级建模：对 descriptor code 使用 cumulative-link ordinal head，而不是粗暴连续回归。

## 仓库内容

- `docs/PROJECT_PLAN_AND_PROGRESS.md`：总研究计划 + 阶段进展，后续每个小阶段只更新这个主进展文件。
- `docs/MODEL_ARCHITECTURE.md`：RiceGeneFormer-OMTL 自研模型结构、损失函数、消融设计和参数规模。
- `docs/TECHNICAL_ROUTE.md`：数据预处理、输入格式、训练、评估、SLURM 资源计划和执行路线。

## 当前数据状态

截至 2026-06-13 16:12:27 CST：

- 已下载 3K Rice phenotype：2,266 accessions × 53 columns，其中 47 个 phenotype fields。
- 已下载并验证 404k core v0.7 PLINK BED/BIM/FAM；因样本 ID 为 B001/B002，不作为第一阶段主训练数据。
- 已下载并验证 `3K_coreSNP-v2.1.plink.tar.gz`，包含 365,710 SNP × 3,000 samples，样本 ID 为 IRIS ID。
- v2.1 PED/MAP 与 phenotype 完整对接：2,266 / 2,266 phenotype rows。
- 当前 phenotype 主要是 IRRI rice descriptor code：多数为分类 / 有序等级 / 少数二分类。

## 第一阶段核心表型

优先使用高覆盖、育种相关、有明确等级含义的 traits：

```text
CULT_CODE_REPRO   culm length
LLT_CODE          leaf length
PLT_CODE_POST     panicle length
SDHT_CODE         seedling height
CUNO_CODE_REPRO   culm number
CUDI_CODE_REPRO   culm diameter
SPKF              spikelet fertility
CUST_REPRO        culm strength
PTH               panicle threshability
PSH               panicle shattering
CUAN_REPRO        culm angle
FLA_REPRO         flag leaf angle
PEX_REPRO         panicle exsertion
PTY               panicle type
ENDO              endosperm type
```

## 项目原则

1. GitHub 只保存计划、进展、项目介绍、模型结构、技术路线；不上传数据、结果、日志、模型权重。
2. 每一小阶段进展更新在 `docs/PROJECT_PLAN_AND_PROGRESS.md`，每条进展必须带具体时间。
3. 数据处理和 CPU 作业在 SLURM 登录节点提交到 q07/q08，资源不足时用 cu；单作业最多 30 CPU、150G RAM；每轮最多提交 6 个命令。
4. GPU 正式训练在 gpu10 上执行，使用 `CUDA_VISIBLE_DEVICES=` 指定 GPU，不使用 `--gpu` 参数。
5. 正式模型、配置字段、文档标题统一使用 RiceGeneFormer 命名体系。
