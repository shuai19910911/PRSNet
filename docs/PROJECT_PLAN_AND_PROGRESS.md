# RiceGeneFormer 水稻 3K Genome 正式研究计划与进展

最后更新：2026-06-14 10:33:14 CST

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
- [2026-06-13 17:41:37 CST] Phase 1 输入矩阵构建完成并验证：初始 SLURM 作业 `8562916` 在 phenotype sheet 选择处失败（误读 workbook 说明页，未找到 `GS_ACCNO`）；修复为自动选择含 `GS_ACCNO/STOCK_ID` 的 phenotype 数据页，并通过 `v2.1_ped_to_phenotype_overlap.tsv` 将 IRIS sample ID 映射到 phenotype `STOCK_ID`。补交作业 `8562917` 在 `cu` 分区 3 秒完成，跳过已存在 genotype 矩阵并重建 phenotype；验证 `X_uint8.npy` shape=`3000x365710`，`Y_multitask.npy`/`mask_multitask.npy` shape=`3000x35`，35 个 trait 入选。
- [2026-06-13 17:41:37 CST] Phase 1 split 构建完成：生成 `processed/splits/random_seed42.tsv` 和 region leave-out splits（Southeast_Asia、South_Asia、East_Asia）；监督可用样本 2,266，metadata 对接 3,000/3,000。当前 `3K_list_sra_ids.txt` 不含 subpopulation 字段，因此 subpopulation split 未生成并在 `split_report.tsv` 中显式记录。
- [2026-06-13 18:20:50 CST] Phase 2 train-fold GWAS 作业 `8562918` 在 `cu` 分区完成（`COMPLETED`, exit `0:0`, elapsed `00:07:31`, batch MaxRSS `2151896K`）。验证 `core_ordinal/gwas_report.tsv` 为 40 行（10 个核心 trait × 4 个 split），40 组 `pvalues.npy`/`betas.npy` shape 均为 `(365710,)`，无缺失输出；所有代表性数组 finite p-value 数为 365,710。
- [2026-06-13 18:20:50 CST] Phase 3 启动并完成首版 SNP-to-gene mapping：下载 Ensembl Plants IRGSP-1.0 release 61 GFF3（gzip 校验通过），新增 `download_rice_annotation.sh`、`build_snp_gene_map.py`、`build_snp_gene_map.sh`，通过 `sh -n`、`py_compile`、`git diff --check`、静态安全扫描和独立代码审核。SLURM 作业 `8562919` 在 `cu` 分区完成（`COMPLETED`, exit `0:0`, elapsed `00:00:07`），窗口为 gene body ±5 kb；结果：365,710 SNP、35,806 genes、265,028 unique SNPs mapped、486,452 SNP-gene edges、34,139 genes with SNPs。
- [2026-06-13 18:35:56 CST] Phase 4 完成首版 gene graph baseline：新增 `build_gene_graph.py` 和 `build_gene_graph_baseline.sh`，通过 `py_compile`、`sh -n`、`git diff --check`、静态安全扫描和独立代码审核。SLURM 作业 `8562920` 在 `cu` 分区完成（`COMPLETED`, exit `0:0`, elapsed `00:00:10`）；基于 34,139 个 mapped genes 构建 chromosome k-neighbor graph（k=5）和 degree-matched random negative-control graph，二者均为 170,515 undirected edges / 341,030 directed edges。
- [2026-06-13 18:57:26 CST] Phase 4 输出复核通过：确认 `graph_report.tsv`、`chr_neighbor_k5` 与 `random_degree_matched_k5` 的 `gene_nodes.tsv`/`graph_edges.tsv`/`edge_index.npy`/`edge_weight.npy`/`edge_relation.npy` 均存在且非空；`edge_index` shape 均为 `(2, 341030)`，directed edge count 均等于 `2 × 170,515`，random graph 与 chromosome-neighbor graph 边数一致。随后 Phase 5 dataloader/model-input smoke 作业 `8562921` 在 `cu` 分区完成（`COMPLETED`, exit `0:0`, elapsed `00:00:08`）：验证 `X_uint8.npy=(3000,365710)`、`Y/mask=(3000,35)`、random split train/val/test/unused=`1586/340/340/734`、10 个 core ordinal trait 的 GWAS pvalue shape 均为 365,710、baseline graph 34,139 nodes / 341,030 directed edges。
- [2026-06-13 19:51:59 CST] Phase 5 model-input smoke 终验通过：`sacct` 确认作业 `8562921` 为 `COMPLETED|0:0|00:00:08`，`model_input_smoke_manifest.json` 与 `model_input_smoke_report.tsv` 均记录 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、`core_traits=10`、`graph_nodes=34139`、`graph_directed_edges=341030`、random split train/val/test=`1586/340/340`。随后准备最小 RiceGeneFormer-OMTL PyTorch smoke：本地 CPU 小批次前向/反向验证通过（`torch=2.6.0+cpu`、`logit_shape=2x35`、loss finite、grad_norm positive），代码通过 `py_compile`、SLURM `sh -n`、静态安全扫描和独立审核；当前 PRSNet 环境为 CPU-only PyTorch，GPU smoke 需先安装/切换 CUDA 版 PyTorch 后再提交。
- [2026-06-13 20:18:46 CST] Phase 5 最小 RiceGeneFormer-OMTL smoke 复核：修正 masked loss 只在 observed labels 上计算，避免缺失表型 `NaN` 被 `0*NaN` 传播；复跑 CPU smoke 通过（`torch=2.6.0+cpu`、`status=ok`、`x_shape=3000x365710`、`y_shape=3000x35`、`logit_shape=2x35`、`loss=6.324990272521973`、`grad_norm=23.806309651940786`），并通过 `py_compile`、SLURM `sh -n`、`git diff --check`、静态安全扫描和独立代码复审。`PRSNet` 环境仍为 CPU-only PyTorch（`cuda_available=False`、`torch.version.cuda=None`），因此未提交 GPU smoke；待切换 CUDA 版 PyTorch 后运行 `scripts/slurm/rice_geneformer_omtl_gpu_smoke.sh`。
- [2026-06-13 20:41:45 CST] Phase 5 GPU smoke 启动条件复查：`PRSNet` 环境已可加载 CUDA 版 PyTorch（`torch=2.6.0+cu124`、`torch.version.cuda=12.4`），登录节点无 GPU 所以 `torch.cuda.is_available=False`；复跑最小 CPU smoke 通过（`status=ok`、`x_shape=3000x365710`、`y_shape=3000x35`、`logit_shape=2x35`、`loss=6.424133777618408`、`grad_norm=24.175355683459742`）。尝试提交 `sbatch -p gpu10 -c 4 --mem=24G --gres=gpu:1 scripts/slurm/rice_geneformer_omtl_gpu_smoke.sh` 失败：SLURM 返回 `invalid partition specified: gpu10`；`sinfo` 当前仅显示 `cu/fat/q03/q04/q05/q07/q08` 且 GRES 为 `(null)`，未暴露 GPU 分区/节点。因此本轮未提交 GPU 作业，需用户在可访问的 `gpu10` shell 或管理员提供的 GPU 分区上手动启动。
- [2026-06-13 21:06:38 CST] Phase 5/6 最小训练闭环完成：新增本地 `scripts/model/rice_geneformer_omtl_train.py`，实现 10 个 `core_ordinal` traits 的 ordinal cumulative-link masked multi-task loss、train/val loop、accuracy/MAE/macro-F1/Spearman 指标、manifest/metrics 输出和 checkpoint。脚本通过 `py_compile`、`git diff --check`、CPU 1 epoch smoke 以及独立代码审核（无 security_concerns / logic_errors）。在 `gpu10` 通过 SSH 仅使用 GPU 0 运行 3 epoch 小训练（batch=8, genes=2048, hidden_dim=64, max_steps_per_epoch=20, torch `2.6.0+cu124`）：train loss `0.6585 → 0.5768 → 0.5579`，val loss `0.5965 → 0.5547 → 0.5223`，最终 val accuracy `0.3873`、MAE `0.8111`、macro-F1 `0.1427`、Spearman `0.0931`，输出目录为本地数据区 `data/3krice/processed/rice_geneformer_omtl_train_gpu0_e3_g2048_b8/`（不上传 GitHub）。
- [2026-06-13 21:23:57 CST] Phase 5/6 bounded pilot 继续推进：确认 `sacct` 中 Phase 5 model-input smoke 作业 `8562921` 为 `COMPLETED|0:0|00:00:08`，manifest/report 终验仍为 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。随后在 `gpu10` 物理 GPU 0 上运行 10 epoch bounded pilot（batch=8, genes=2048, hidden_dim=64, max_steps_per_epoch=30, torch `2.6.0+cu124`）：train loss `0.6362 → 0.4560`，val loss `0.5651 → 0.4500`，best epoch `10`，最终 val accuracy `0.5894`、MAE `0.5935`、macro-F1 `0.2236`、Spearman `-0.0292`，输出目录为本地数据区 `data/3krice/processed/rice_geneformer_omtl_train_gpu0_e10_g2048_b8_s30/`（含 `training_manifest.json`、`training_metrics.tsv`、`checkpoint_last.pt`，约 927K；不上传 GitHub）。
- [2026-06-13 21:27:40 CST] Phase 6 bounded pilot 与 baseline smoke 完成：在 `gpu10` 仅 GPU 0 运行 15 epoch bounded pilot（batch=16, genes=2048, hidden_dim=64, max_steps_per_epoch=50），train loss `0.6040 → 0.4145`，val loss 最优 epoch 11 为 `0.4080`，最终 val accuracy `0.5936`、MAE `0.6133`、macro-F1 `0.2125`；说明最小模型可稳定下降但后期出现轻微波动。随后新增本地 `scripts/model/baseline_lightgbm_core_smoke.py`，用每 trait train-fold GWAS top-512 SNP 训练 LightGBM baseline（10 traits, 40 estimators），脚本通过 `py_compile`、`git diff --check`、静态扫描、CPU smoke 和独立代码审核。LightGBM baseline 平均 accuracy `0.6271`、MAE `0.5547`、macro-F1 `0.3354`，majority baseline 平均 accuracy `0.6028`、MAE `0.6736`；当前 LightGBM 仍强于最小神经模型，下一步应优先做模型容量/训练策略与消融，而不是直接宣称优于传统 baseline。
- [2026-06-13 21:42:45 CST] Phase 6 模型容量/学习率 pilot 完成：在 `gpu10` 仅 GPU 0 追加两组训练。容量提升组（genes=4096, hidden_dim=96, batch=8, 10 epoch, lr=2e-4）best/final val loss `0.4426`、accuracy `0.5867`、MAE `0.6125`、macro-F1 `0.2183`，未超过 2048-gene 15 epoch 组；低学习率组（genes=2048, hidden_dim=64, batch=16, 15 epoch, lr=1e-4）best/final val loss `0.4378`、accuracy `0.5936`、MAE `0.6140`、macro-F1 `0.2174`，较 lr=2e-4 的 best val loss `0.4080` 更差。结论：单纯增大 gene 数/hidden_dim 或降低 lr 未缩小与 LightGBM top-SNP baseline 的差距，下一步应优先做结构性改造（trait-specific gene pooling、best checkpoint/early stopping、no-prior/random-prior 消融、加入 top-GWAS SNP dense side branch）。
- [2026-06-13 21:45:54 CST] Phase 6 追加容量安全复核：确认 `gpu10` 物理 GPU 0 可用（A100-40G，PyTorch `2.6.0+cu124`，CUDA available），运行更保守的 4096 genes / hidden_dim 96 / batch 4 / 8 epoch / 30 step pilot；输出 `status=ok`，best epoch 7，best val loss `0.4623`，final val loss `0.4788`、accuracy `0.3890`、MAE `0.8247`、macro-F1 `0.1525`，输出目录 `data/3krice/processed/rice_geneformer_omtl_train_gpu0_e8_g4096_h96_b4_s30/`（约 1.9M，不上传 GitHub）。该复核进一步支持：当前瓶颈不是单纯扩大 gene/token 容量，而是模型结构与训练目标需要改造。
- [2026-06-13 22:08:40 CST] Phase 6 结构性改造第一步完成：训练脚本新增 `trait_attention` / `mean` pooling 选项，默认 `trait_attention`，用每个 trait query 对 gene tokens 做 multi-head attention，实现 trait-specific gene attention pooling；同时新增 best checkpoint 保存、`checkpoint_best.pt` / `checkpoint_last.pt` manifest 记录、`early_stop_patience` 早停、非有限 validation loss 显式失败、标准 JSON 输出（NaN/Inf 指标转为 `null`）。验证：`py_compile` 通过；CPU smoke（32 genes、hidden_dim 32、batch 2、2 epoch、1 step/epoch）输出 `status=ok`、pooling=`trait_attention`、best epoch `2`、best val loss `0.6908`、manifest 可被 `python -m json.tool` 解析；独立代码复审通过，无 security_concerns / logic_errors。该步只验证结构与产物正确性，不用于性能宣称；下一步需要在 `gpu10` 物理 GPU 0 上跑 trait_attention bounded pilot，并与原 mean/global pooling、LightGBM baseline 对比。
- [2026-06-13 22:29:25 CST] Phase 6 trait_attention bounded pilot 完成：在 `gpu10` 物理 GPU 0 上运行与上一轮对齐的 15 epoch pilot（batch=16, genes=2048, hidden_dim=64, max_steps_per_epoch=50, lr=2e-4, pooling=`trait_attention`, PyTorch `2.6.0+cu124`）。输出 `status=ok`，15/15 epoch 完成，best epoch `11`，best val loss `0.3927`，final val loss `0.4097`、accuracy `0.5943`、MAE `0.6126`、macro-F1 `0.2171`、Spearman `-0.0043`；产物位于本地数据区 `data/3krice/processed/rice_geneformer_omtl_train_traitattn_gpu0_e15_g2048_b16_s50/`，`training_manifest.json` 可被 `json.tool` 解析，`checkpoint_best.pt`/`checkpoint_last.pt` 分别约 990K/992K（不上传 GitHub）。结论：trait-specific attention pooling 的 best val loss 优于原 2048-gene mean/global pilot（约 `0.4080`），但 final macro-F1 仍低于 LightGBM top-SNP baseline（`0.3354`），下一步应做 top-GWAS SNP dense side branch 和 pooling/prior 消融。
- [2026-06-13 22:41:32 CST] Phase 6 top-GWAS SNP dense side branch 完成 CPU 级结构验证：训练脚本新增 `--top-snp-count`，默认 `0` 关闭；启用时仅从当前 split 的 train-fold GWAS p-values 选择 top SNP union，并强制 p-value 路径位于 `gwas/core_ordinal/<split>/<trait>/pvalues.npy`，检查 shape、finite、范围 `[0,1]`、去重和边界，防止使用全数据 GWAS 造成泄漏。模型新增 top-SNP dense encoder，将 top SNP dosage 编码为 side token 并加到 trait tokens；manifest 记录 `top_snp_count`、`top_snp_count_used`、`top_snp_traits_used`。验证：TDD red 阶段确认旧脚本不接受 `--top-snp-count`；`py_compile` 通过；CPU smoke（top_snp_count=16、trait_attention、16 genes、hidden_dim 32、1 epoch）输出 `status=ok`、`top_snp_count_used=16`、best val loss `0.7368`，manifest 可由 `json.tool` 解析；默认 `top_snp_count=0` + mean pooling 兼容 smoke 通过；独立代码复审通过，无 security_concerns / logic_errors。该步只验证结构和无泄漏数据通路，不作性能宣称；下一步需在 `gpu10` 物理 GPU 0 上做 bounded pilot。
- [2026-06-13 22:56:23 CST] Phase 6 top-GWAS SNP dense side branch bounded pilot 完成：先按独立代码复审意见补强 p-value shape/finite/range 校验、smoke 脚本 canonical train-fold p-value 路径约束、split sample_index 边界/重复检查，并复跑 `py_compile`、`sh -n`、`git diff --check`、CPU top_snp_count=16 smoke 与独立复审，均通过。随后在 `gpu10` 物理 GPU 0 上运行 top_snp_count=512 + trait_attention pilot（genes=2048、hidden_dim=64、batch=16、15 epoch、50 steps/epoch、lr=2e-4、early_stop_patience=5，PyTorch `2.6.0+cu124`），输出 `status=ok`、`top_snp_count_used=512`、15/15 epoch 完成，best epoch `11`，best val loss `0.3821`，final val loss `0.3945`、accuracy `0.5624`、MAE `0.6106`、macro-F1 `0.2707`、Spearman `0.2205`。产物位于本地数据区 `data/3krice/processed/rice_geneformer_omtl_train_top512_traitattn_gpu0_e15_g2048_b16_s50/`，manifest 可由 `json.tool` 解析，`checkpoint_best.pt`/`checkpoint_last.pt` 分别约 1.15 MB（不上传 GitHub）。结论：top-GWAS side branch 改善 best val loss 和 macro-F1，但 accuracy 仍低于 LightGBM top-SNP baseline，下一步继续做 no-prior/random-prior/mean pooling 消融和 SNP-MLP/XGBoost baseline。
- [2026-06-13 23:16:05 CST] Phase 6 pooling 消融新增 mean-pooling 对照：在 `gpu10` 物理 GPU 0 上复用 top_snp_count=512、genes=2048、hidden_dim=64、batch=16、15 epoch、50 steps/epoch、lr=2e-4 的配置，仅将 pooling 改为 `mean`。运行完成 `status=ok`、`top_snp_count_used=512`、15/15 epoch、best epoch `13`、best val loss `0.4052`，final val loss `0.4052`、accuracy `0.5624`、MAE `0.6072`、macro-F1 `0.2583`、Spearman `0.2152`；`training_manifest.json` 通过 `json.tool`，`checkpoint_best.pt`/`checkpoint_last.pt` 分别约 1.15 MB（不上传 GitHub）。对比 trait_attention + top512 的 best val loss `0.3821` / macro-F1 `0.2707`，当前对照支持 trait-specific attention pooling 优于 mean pooling。
- [2026-06-13 23:45:10 CST] Phase 6 gene p-value prior 消融开关完成 CPU 级结构验证：训练脚本新增 `--gene-prior-mode observed|none|random`，默认 `observed` 保持原 train-fold p-value gene prior；`none` 将 gene prior 置零；`random` 用固定 CPU seed 打乱 gene prior，保持分布但破坏 gene 对应关系。manifest 新增 `gene_prior_mode`，random 模式额外记录 `gene_prior_random_seed`。验证：TDD red 阶段确认旧脚本不接受 `--gene-prior-mode`；`py_compile` 通过；CPU smoke 分别验证 `none` 和 `random` 均输出 `status=ok`，random v2 manifest 记录 `gene_prior_mode=random`、`gene_prior_random_seed=59` 且可由 `json.tool` 解析；静态扫描仅有 `model.eval()` 假阳性；独立代码复审通过，无 security_concerns / logic_errors。该步只验证消融开关与数据通路正确性，不作性能宣称；下一步应在 `gpu10` 物理 GPU 0 上对齐 top512 + trait_attention 当前最佳配置运行 `none` 与 `random` bounded pilot。
- [2026-06-14 00:05:28 CST] Phase 6 gene p-value prior GPU bounded pilot 完成：先复核 Phase 5 model-input smoke 作业 `8562921` 已 `COMPLETED|0:0|00:00:08`，manifest/report 终验仍为 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。随后在 `gpu10` 物理 GPU 0 上对齐 top_snp_count=512 + trait_attention 最佳配置运行两个 15 epoch bounded pilot：`gene_prior_mode=none` 输出 `status=ok`、best epoch `11`、best val loss `0.3824`、final val loss `0.3951`、accuracy `0.5631`、MAE `0.6038`、macro-F1 `0.2720`、Spearman `0.2223`；`gene_prior_mode=random` 输出 `status=ok`、random seed `59`、best epoch `11`、best val loss `0.3822`、final val loss `0.3944`、accuracy `0.5624`、MAE `0.6058`、macro-F1 `0.2698`、Spearman `0.2223`。两组 `training_manifest.json` 均通过 `json.tool`，`checkpoint_best.pt`/`checkpoint_last.pt` 均存在且约 1.15 MB（不上传 GitHub）。结论：在当前 top512 side branch 强先验配置下，observed/none/random gene prior 指标非常接近，gene p-value prior 的独立增益不明显；下一步应转向 random graph、`max_snps_per_gene`、以及 SNP-MLP/XGBoost baseline。
- [2026-06-14 00:09:41 CST] Phase 6 第二传统模型 baseline 完成：新增本地 `scripts/model/baseline_xgboost_core_smoke.py`，一 trait 一个 XGBoost classifier，仅用当前 split 的 train-fold GWAS top SNP 作为输入特征；脚本检查 split/role 列、sample_index 边界和重复、trait_index 边界、label 范围、GWAS duplicate rows、p-value canonical 路径、shape、finite、范围 `[0,1]` 与超参边界，并输出 majority baseline 对照。TDD red 阶段确认新增前脚本不存在；初始 smoke 暴露 XGBoost 训练标签必须连续 `0..K-1`，已修复为训练时编码、预测后映射回原始 ordinal label 计算指标；`py_compile`、CPU smoke、`json.tool`、静态扫描、`git diff --check` 和独立代码复审均通过。正式 smoke（top_snps=512, n_estimators=40, 10 core traits）完成：mean accuracy `0.6274`、mean MAE `0.5662`、mean macro-F1 `0.3115`、majority accuracy `0.6028`；输出目录 `data/3krice/processed/xgboost_core_top512_e40/`（本地，不上传 GitHub）。对比：XGBoost accuracy 与 LightGBM `0.6271` 接近、macro-F1 略低于 LightGBM `0.3354`，但仍高于当前最佳神经模型 macro-F1 `0.2707`，说明传统 top-SNP tree baseline 仍是主要性能门槛。
- [2026-06-14 01:25:03 CST] Cron 复核并推进 random graph 消融：`squeue -j 8562921` 已无活动记录，`sacct` 确认 Phase 5 model-input smoke 作业 `8562921` 为 `COMPLETED|0:0|00:00:08`；manifest/report 再次验证 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。随后在 `gpu10` 物理 GPU 0 上运行 random degree-matched graph 的短程消融（top_snp_count=512、trait_attention、genes=2048、hidden_dim=64、batch=16、5 epoch、30 steps/epoch、lr=2e-4，PyTorch `2.6.0+cu124`）：输出 `status=ok`、best epoch `5`、best/final val loss `0.4379`、accuracy `0.5456`、MAE `0.6334`、macro-F1 `0.2488`、Spearman `0.1957`；`training_manifest.json` 通过 `json.tool`，`checkpoint_best.pt`/`checkpoint_last.pt` 分别约 1.15 MB（本地数据区，不上传 GitHub）。短程结果低于真实 chromosome-neighbor graph + top512 trait_attention 的 15 epoch pilot（best val loss `0.3821`、macro-F1 `0.2707`），但 epoch 数不同，需后续对齐 15 epoch 后再作正式结论。
- [2026-06-14 01:46:55 CST] Random graph 15 epoch 对齐复跑完成并发现关键方法学问题：在 `gpu10` 物理 GPU 0 上运行 random degree-matched graph + top512 + trait_attention（genes=2048、hidden_dim=64、batch=16、15 epoch、50 steps/epoch）输出 `status=ok`、best epoch `11`、best val loss `0.3820795923`、final val loss `0.3944686621`、accuracy `0.5624`、MAE `0.6106`、macro-F1 `0.2707`、Spearman `0.2205`，manifest/checkpoint 均验证通过。然而代码复核显示当前 `rice_geneformer_omtl_train.py` 只读取 `gene_nodes.tsv` 决定 gene 顺序，并未读取 `edge_index.npy`/`graph_edges.tsv` 参与 message passing；因此 random graph 与 chromosome-neighbor graph 在当前模型中不是有效图结构消融，指标近乎相同不能作为 graph 增益结论。下一步优先修复/新增真正使用 graph edges 的轻量 GraphEncoder 后再重做 random graph 对照。
- [2026-06-14 02:04:36 CST] GraphEncoder 修复完成第一轮 smoke：在本地训练脚本中新增 `edge_index.npy` 读取与轻量 mean-neighbor GraphEncoder，模型前向现在在 gene transformer 后执行 message passing，并在 manifest 记录 `graph_edges_used`。验证：`py_compile`、`git diff --check`、静态扫描、独立代码审核均通过；CPU smoke 分别跑通 `chr_neighbor_k5`（32 genes, `graph_edges_used=290`）和 `random_degree_matched_k5`（32 genes, `graph_edges_used=2`）；随后在 `gpu10` 物理 GPU 0 上运行 chr graph GPU smoke（128 genes、top_snp_count=32、batch=4、1 epoch、2 step/epoch、PyTorch `2.6.0+cu124`），输出 `status=ok`、`cuda_available=true`、`graph_edges_used=1250`、best/final val loss `0.7391`、accuracy `0.4085`、macro-F1 `0.2173`，`training_manifest.json` 可解析且 `checkpoint_best.pt`/`checkpoint_last.pt` 分别约 126K/128K（本地数据区，不上传 GitHub）。注意：当前 bounded random induced subgraph 在小 `max_genes` 下边数很少，下一步重跑正式 random graph 对照前需先设计等节点/等边数的局部 random negative-control 或扩大可承受 gene 范围。
- [2026-06-14 02:42:26 CST] GraphEncoder 公平 random negative-control 首轮完成：为避免全图 degree-matched random 在 `max_genes=2048` 前缀诱导子图过稀疏，新增本地数据图 `prefix2048_random_k5`，固定使用与 `chr_neighbor_k5` 相同的前 2,048 个 gene nodes，并生成 10,225 条无向随机边 / 20,450 条有向边，恰好匹配 `chr_neighbor_k5` 在前 2,048 genes 的 `graph_edges_used=20450`。CPU smoke 通过（32 genes，`status=ok`）；随后在 `gpu10` 物理 GPU 0 上运行对齐 top512 + trait_attention + GraphEncoder 15 epoch bounded pilot（batch=16、genes=2048、hidden_dim=64、50 steps/epoch、PyTorch `2.6.0+cu124`），输出 `status=ok`、`graph_edges_used=20450`、best epoch `11`、best val loss `0.3765`、final val loss `0.3921`、accuracy `0.5719`、MAE `0.5916`、macro-F1 `0.2844`、Spearman `0.1995`，`training_manifest.json` 可解析且 `checkpoint_best.pt`/`checkpoint_last.pt` 分别约 1.51 MB（本地数据区，不上传 GitHub）。当前等边随机图并未低于 chr-neighbor 对照（chr top512 trait_attention 15 epoch best val loss `0.3821`、macro-F1 `0.2707`），说明现有 GraphEncoder 的收益暂不能归因于染色体邻接结构，下一步应做多 seed 与更强外部 biological graph，而非宣称 baseline graph 有稳定增益。
- [2026-06-14 04:03:58 CST] Cron 继续推进 GraphEncoder 多 seed 对照：复核 Phase 5 model-input smoke 作业 `8562921` 仍为 `COMPLETED|0:0|00:00:08`，manifest/report 仍为 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。随后在 `gpu10` 物理 GPU 0 上追加 seed=43 的对齐 15 epoch bounded pilot（top512 + trait_attention + GraphEncoder、genes=2048、hidden_dim=64、batch=16、50 steps/epoch、PyTorch `2.6.0+cu124`）：`chr_neighbor_k5` 输出 `status=ok`、`graph_edges_used=20450`、best epoch `10`、best val loss `0.3796`、final val loss `0.3832`、accuracy `0.5865`、MAE `0.5893`、macro-F1 `0.2755`、Spearman `0.2545`；`prefix2048_random_k5` 输出 `status=ok`、`graph_edges_used=20450`、best epoch `10`、best val loss `0.3797`、final val loss `0.3835`、accuracy `0.5810`、MAE `0.5859`、macro-F1 `0.2699`、Spearman `0.2532`。两组 manifest 均通过 `json.tool`，`checkpoint_best.pt`/`checkpoint_last.pt` 均非空（本地数据区，不上传 GitHub）。seed=43 结果显示 chr 与等边随机图仍非常接近，继续支持“当前轻量 GraphEncoder 未证明染色体邻接结构特异增益”的保守结论。
- [2026-06-14 04:29:36 CST] Cron 继续推进 GraphEncoder 多 seed 对照 seed=44：复核 SLURM 作业 `8562921` 已完成且输入 smoke 仍通过后，在 `gpu10` 物理 GPU 0 上顺序运行两组对齐 bounded pilot（top512 + trait_attention + GraphEncoder、genes=2048、hidden_dim=64、batch=16、最多 15 epoch、50 steps/epoch、early_stop_patience=5、PyTorch `2.6.0+cu124`）。`chr_neighbor_k5` 输出 `status=ok`、`graph_edges_used=20450`、early stop 于 14 epoch、best epoch `9`、best val loss `0.3757`、final val loss `0.3943`、accuracy `0.5788`、MAE `0.5989`、macro-F1 `0.2908`、Spearman `0.2087`；`prefix2048_random_k5` 输出 `status=ok`、`graph_edges_used=20450`、early stop 于 14 epoch、best epoch `9`、best val loss `0.3767`、final val loss `0.3947`、accuracy `0.5815`、MAE `0.6009`、macro-F1 `0.2969`、Spearman `0.2054`。两组 `training_manifest.json` 均通过 `json.tool`，`checkpoint_best.pt`/`checkpoint_last.pt` 均非空（约 1.51 MB，本地数据区，不上传 GitHub）。seed=44 与 seed=43 一致：chr-neighbor 与等边随机图差距极小，当前轻量 GraphEncoder 不能支持染色体邻接结构有特异增益的结论。
- [2026-06-14 04:51:59 CST] Cron 启动 `max_snps_per_gene` 消融第一组：复核 Phase 5 model-input smoke 作业 `8562921` 已 `COMPLETED|0:0|00:00:08`，manifest/report 仍为 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。随后在 `gpu10` 物理 GPU 0 上运行 chr graph + top512 + trait_attention + GraphEncoder 的 `max_snps_per_gene=64` bounded pilot（genes=2048、hidden_dim=64、batch=16、15 epoch、50 steps/epoch、early_stop_patience=5、PyTorch `2.6.0+cu124`），输出 `status=ok`、`graph_edges_used=20450`、15/15 epoch、best epoch `11`、best val loss `0.3768`、final val loss `0.3920`、accuracy `0.5733`、MAE `0.5909`、macro-F1 `0.2875`、Spearman `0.2024`；`training_manifest.json` 通过 `json.tool`，`checkpoint_best.pt`/`checkpoint_last.pt` 均非空（本地数据区 `data/3krice/processed/rice_geneformer_omtl_train_graph_chr_maxsnp64_seed42_e15_g2048_b16_s50/`，不上传 GitHub）。与 seed42 默认 `max_snps_per_gene=32` 近似持平，暂未看到显著收益，仍需补 `16/128` 或多 seed 后再定论。
- [2026-06-14 05:37:59 CST] Cron 完成 `max_snps_per_gene` seed42 补充消融：先复核 SLURM 输入 smoke 作业 `8562921` 仍为 `COMPLETED|0:0|00:00:08`，manifest/report 继续满足 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。随后在 `gpu10` 物理 GPU 0 上对齐运行 `max_snps_per_gene=16` 与 `128` 两组 chr graph + top512 + trait_attention + GraphEncoder pilot（genes=2048、hidden_dim=64、batch=16、15 epoch、50 steps/epoch、early_stop_patience=5、PyTorch `2.6.0+cu124`），两组均 `status=ok`、`graph_edges_used=20450`、15/15 epoch、best epoch `11`，manifest 可由 `json.tool` 解析且 `checkpoint_best.pt`/`checkpoint_last.pt` 均非空。`max_snps_per_gene=16`：best val loss `0.3766`、final val loss `0.3920`、accuracy `0.5760`、MAE `0.5875`、macro-F1 `0.2884`、Spearman `0.2043`；`max_snps_per_gene=128`：best val loss `0.3768`、final val loss `0.3920`、accuracy `0.5733`、MAE `0.5909`、macro-F1 `0.2875`、Spearman `0.2025`。结合 32/64 结果，当前 `max_snps_per_gene` 在 16–128 范围内影响很小，短期不应作为主要性能瓶颈；下一步转向 SNP-MLP baseline 与外部 biological graph。
- [2026-06-14 06:11:35 CST] Cron 完成 SNP-MLP baseline smoke：复核 Phase 5 model-input smoke 作业 `8562921` 仍为 `COMPLETED|0:0|00:00:08`，manifest/report 仍满足 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。新增本地 `scripts/model/baseline_snp_mlp_core_smoke.py` 与 `scripts/slurm/baseline_snp_mlp_core_smoke.sh`，仅用 active split 的 train-fold GWAS top SNP union，检查 p-value canonical path、shape、finite、范围 `[0,1]`、split disjointness、observed label 与 JSON `allow_nan=False`；通过 `py_compile`、`sh -n`、`git diff --check`、CPU tiny smoke、manifest `json.tool` 和独立代码复审。SLURM 作业 `8562943` 在 `cu` 分区完成（`COMPLETED`, exit `0:0`, elapsed `00:01:14`, batch MaxRSS `1518152K`）；SNP-MLP top512 per trait union 共 `4218` SNP，10 epoch 输出 `status=ok`、best epoch `1`、best val loss `0.8913`、final accuracy `0.5916`、MAE `0.6231`、macro-F1 `0.3466`、Spearman `0.2728`。结论：SNP-MLP macro-F1 略高于 LightGBM/XGBoost baseline，但 accuracy 低于 tree baseline/majority，且 val loss 从 epoch 2 后上升，说明 top-SNP MLP 快速过拟合；后续应加入 early stopping/正则网格，并继续外部 biological graph。
- [2026-06-14 07:07:57 CST] Cron 推进外部 biological graph：复核 Phase 5 model-input smoke 作业 `8562921` 已完成且输入终验仍通过（`status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`）。随后下载 STRING v12.0 `Oryza sativa Japonica` taxon `39947` aliases 与 links 原始文件到本地数据区（aliases `2,248,066` bytes，links `358,782,402` bytes，均通过 `gzip -t`；不上传 GitHub），新增本地 STRING graph 构建脚本并通过 `py_compile`、SLURM `sh -n`、`git diff --check`、fixture smoke、静态扫描和独立代码复审。SLURM 作业 `8562944` 在 `cu` 分区完成（`COMPLETED`, exit `0:0`, elapsed `00:01:03`, batch MaxRSS `210640K`），生成 `string_v12_min700_top20` graph：34,139 nodes、mapped STRING proteins `33,979`、raw mapped undirected edges `459,820`、按每 gene 最高 20 条边剪枝后 undirected edges `61,735` / directed edges `123,470`、score range `700–999`、max degree `20`、`edge_index=(2,123470)`。CPU 级训练读取 smoke 通过：`graph=string_v12_min700_top20`、`graph_edges_used=416`（前 2,048 genes）、`status=ok`、best val loss `0.6928`。当前无法通过批处理访问 GPU 分区，`ssh gpu10` 本轮超时，因此未启动 STRING graph GPU pilot；下一步需在可用 gpu10 shell 上运行对齐 15 epoch bounded pilot 后与 chr/random graph 对比。
- [2026-06-14 07:42:43 CST] Cron 复核 Phase 5 model-input smoke：`squeue -j 8562921` 已无活动记录，`sacct` 返回 `8562921|rgf_input_smoke|cu|COMPLETED|0:0|00:00:08`；`model_input_smoke_manifest.json` 与 `model_input_smoke_report.tsv` 均仍为 `status=ok`，并确认 `X=3000x365710`、`Y/mask=3000x35`、`core_traits=10`、`graph_nodes=34139`、`graph_directed_edges=341030`、random split train/val/test=`1586/340/340`。PRSNet 环境 PyTorch 已是 CUDA 版（`torch=2.6.0+cu124`、`torch.version.cuda=12.4`），登录节点无 GPU 所以 `cuda_available=False`；尝试 `ssh gpu10` 可连到主机名但 PyTorch CUDA 检查命令在 30 秒内超时，因此本 tick 未新提交 GPU 训练。继续保持 GitHub 只同步 docs 轻量进展，不上传数据/日志/权重。
- [2026-06-14 08:05:10 CST] Cron 再次复核 Phase 5 model-input smoke 与 GPU 可用性：`squeue -j 8562921` 返回 `Invalid job id specified`（队列中已无活动作业），`sacct` 仍确认为 `8562921|rgf_input_smoke|cu|COMPLETED|0:0|00:00:08`，输出 manifest/report 继续满足 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。登录节点 `PRSNet` 环境可导入 CUDA 版 PyTorch（`torch=2.6.0+cu124`、CUDA build `12.4`），但本机无 GPU；再次尝试 `ssh gpu10` 执行 `hostname/nvidia-smi/torch.cuda` 检查在 45 秒内超时并被终止，因此本 tick 仍未启动 STRING graph GPU pilot。仅更新 docs 轻量进展并准备推送到 GitHub；数据、日志、脚本、模型权重继续保留本地不上传。
- [2026-06-14 08:24:12 CST] Cron 复核 Phase 5 model-input smoke 并完成最小 CUDA smoke：`squeue -j 8562921` 返回队列中无活动作业，`sacct` 仍确认为 `8562921|rgf_input_smoke|cu|COMPLETED|0:0|00:00:08`；`model_input_smoke_manifest.json` 与 report 再次验证 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。`PRSNet` 环境为 CUDA 版 PyTorch（`torch=2.6.0+cu124`、CUDA build `12.4`）；本轮 `ssh gpu10` 成功，确认物理 GPU0 为 A100-40GB 且空闲约 3 MiB。随后在 `gpu10` 物理 GPU0 上运行最小 RiceGeneFormer-OMTL CUDA forward/backward smoke（batch=4、genes=256、max_snps_per_gene=32、hidden_dim=64），输出 `status=ok`、`cuda_available=true`、`logit_shape=4x35`、loss `4.5322`、grad_norm `22.9060`；产物在本地数据区 `data/3krice/processed/rice_geneformer_omtl_smoke_gpu0_cron_20260614_0824/`，不上传 GitHub。代码侧复核 `py_compile`、SLURM `sh -n`、`git diff --check` 均通过；本 tick 仅推送 docs 轻量进展。
- [2026-06-14 08:35:03 CST] 用户询问“接下来怎么做”后按自动执行模式直接推进 STRING rice graph GPU pilot：先确认 `gpu10` 可达且物理 GPU0 为 A100-40GB、空闲约 3 MiB；随后在 `gpu10` 物理 GPU0 上运行 `string_v12_min700_top20` + top512 + trait_attention + GraphEncoder 对齐 15 epoch bounded pilot（genes=2048、max_snps_per_gene=32、hidden_dim=64、batch=16、50 steps/epoch、early_stop_patience=5、lr=2e-4、seed=42，PyTorch `2.6.0+cu124`）。运行完成 `status=ok`、`graph_edges_used=416`、15/15 epoch、best epoch `11`、best val loss `0.3782`、final val loss `0.3914`、accuracy `0.5685`、MAE `0.5916`、macro-F1 `0.2826`、Spearman `0.2066`；`training_manifest.json` 通过 `json.tool`，`checkpoint_best.pt`/`checkpoint_last.pt` 分别约 1.19 MB（本地数据区 `data/3krice/processed/rice_geneformer_omtl_train_graph_string_top512_seed42_e15_g2048_b16_s50/`，不上传 GitHub）。初步对比：STRING graph seed42 优于无 GraphEncoder 原 top512 trait_attention（best val loss `0.3821`、macro-F1 `0.2707`），但略低于等边 prefix random seed42（best val loss `0.3765`、macro-F1 `0.2844`），且在前 2,048 genes 仅使用 416 条边；下一步应补 seed43/44 或构建前缀边数更充分的 STRING/STRING+chr 融合图，再判断外部 biological graph 是否有稳定增益。
- [2026-06-14 08:40:51 CST] 继续自动补齐 STRING graph seed43/44 对齐复跑：在 `gpu10` 物理 GPU0 上顺序运行 `string_v12_min700_top20` + top512 + trait_attention + GraphEncoder（genes=2048、max_snps_per_gene=32、hidden_dim=64、batch=16、15 epoch、50 steps/epoch、early_stop_patience=5）。seed43：`status=ok`、`graph_edges_used=416`、best epoch `12`、best val loss `0.3770`、final val loss `0.3827`、accuracy `0.5845`、MAE `0.5776`、macro-F1 `0.2653`、Spearman `0.2532`；seed44：`status=ok`、`graph_edges_used=416`、early stop 于 14 epoch、best epoch `9`、best val loss `0.3767`、final val loss `0.3926`、accuracy `0.5734`、MAE `0.6009`、macro-F1 `0.2769`、Spearman `0.2041`。三组 STRING manifest 均通过 `json.tool` 且 best/last checkpoint 非空。综合 seed42/43/44：STRING best val loss 接近 chr/random GraphEncoder，但 macro-F1 不稳定且前缀子图边数仅 416，暂不能宣称 STRING 外部生物图谱有稳定优势；下一步应构建 STRING+chr 融合图或放宽 STRING 阈值/拓扑选择以提升 bounded subset 边覆盖，再重做对照。
- [2026-06-14 08:54:36 CST] Cron 构建并验证 `chr_string_v12_min700_top20_fusion` 融合图：新增本地 `build_fusion_graph.py`，将 `chr_neighbor_k5` 与 `string_v12_min700_top20` 按相同 gene node 顺序做无向边 union，生成 34,139 nodes、232,171 undirected edges / 464,342 directed edges，其中 STRING 与 chr overlap 79 条；前 2,048 genes 中 `graph_edges_used=20850`，比纯 STRING 的 416 条显著提高。脚本通过 `py_compile`、静态安全扫描、独立代码复审（初审发现 tmp-dir 碰撞风险，已修复为 pid 临时目录并复审通过）和 CPU 训练读取 smoke（64 genes、top_snp_count=16、`status=ok`、`graph_edges_used=610`）。随后在 `gpu10` 物理 GPU0 上运行 fusion graph + top512 + trait_attention + GraphEncoder seed42 对齐 15 epoch bounded pilot（genes=2048、batch=16、hidden_dim=64、50 steps/epoch、PyTorch `2.6.0+cu124`）：`status=ok`、15/15 epoch、best epoch `11`、best val loss `0.3767`、final val loss `0.3920`、accuracy `0.5739`、MAE `0.5923`、macro-F1 `0.2858`、Spearman `0.2029`；manifest 可由 `json.tool` 解析，`checkpoint_best.pt`/`checkpoint_last.pt` 非空（约 1.52 MB，本地数据区，不上传 GitHub）。初步结果：fusion seed42 与 chr/random/STRING GraphEncoder best val loss 接近，macro-F1 略高于纯 STRING seed42 但仍未形成明确优势；下一步补 seed43/44 后再做结论。
- [2026-06-14 08:58:48 CST] Cron 补齐 `chr_string_v12_min700_top20_fusion` seed43/44 对齐复跑：继续在 `gpu10` 物理 GPU0 上运行同一配置（top512 + trait_attention + GraphEncoder、genes=2048、hidden_dim=64、最多 15 epoch、50 steps/epoch、early_stop_patience=5）。seed43：`status=ok`、`graph_edges_used=20850`、early stop 后记录 best epoch `10`、best val loss `0.3796`、final val loss `0.3832`、accuracy `0.5865`、MAE `0.5893`、macro-F1 `0.2755`、Spearman `0.2545`；seed44：`status=ok`、`graph_edges_used=20850`、early stop 于 14 epoch、best epoch `9`、best val loss `0.3757`、final val loss `0.3943`、accuracy `0.5788`、MAE `0.5983`、macro-F1 `0.2908`、Spearman `0.2087`。两组 manifest 均通过 `json.tool`，`checkpoint_best.pt`/`checkpoint_last.pt` 均非空（约 1.52 MB，本地数据区，不上传 GitHub）。综合 seed42/43/44：fusion 提升了 STRING 在 bounded subset 的边覆盖，但指标仍与 chr-neighbor / prefix random 非常接近；当前不能声称 STRING+chr 融合有稳定结构特异增益。
- [2026-06-14 09:07:21 CST] Cron 汇总 GraphEncoder 多图多 seed 并更新结论：复核 chr-neighbor、prefix random、STRING、fusion 的 seed42/43/44 结果后，当前轻量 GraphEncoder 的各图差异很小。三 seed best val loss 大致为 chr `0.376–0.380`、prefix random `0.376–0.380`、STRING `0.377–0.378`、fusion `0.376–0.380`；macro-F1 波动范围大致 `0.265–0.297`，没有任何图在多 seed 上稳定胜出。STRING 与 fusion 是有价值的外部图谱接入验证，但当前实现不能支撑“外部 biological graph 带来结构特异增益”的论文结论。短期主线应转向更强 neural baseline / 正则策略和 SNP-to-gene mapping 消融；若继续图路线，应升级到 relation-aware graph encoder 或更高覆盖的 pathway/co-expression 图，而不是继续堆当前轻量 mean-neighbor 消融。
- [2026-06-14 09:30:02 CST] 用户再次要求自动推进后，转向 SNP-MLP 正则/early stopping 网格：本地 `baseline_snp_mlp_core_smoke.py` 新增 best-state restore 与 `--early-stop-patience`，训练按 validation loss 保存 CPU `state_dict`，结束后恢复 best epoch 再重算 `final_val_*`，manifest 记录 `epochs_completed`、`early_stop_patience`、`stopped_early`；SLURM wrapper 改为 `epochs=30`、`early_stop_patience=3` 且输出目录命名同步。验证：`py_compile`、`sh -n`、CPU tiny smoke、三组正式 CPU run、manifest `json.tool`、`git diff --check`、静态扫描和独立代码复审均通过。top512 SNP-MLP 正则网格（均 seed42、hidden_dim=256、batch=64、50 steps/epoch、patience=3）结果：dropout `0.2`/wd `1e-4`/lr `1e-3`，early stop 4 epoch，best loss `0.8913`、accuracy `0.6244`、MAE `0.5673`、macro-F1 `0.3167`、Spearman `0.2765`；dropout `0.4`/wd `1e-3`/lr `1e-3`，early stop 5 epoch，best loss `0.8840`、accuracy `0.6193`、MAE `0.5862`、macro-F1 `0.3234`、Spearman `0.3075`；dropout `0.4`/wd `1e-3`/lr `5e-4`，early stop 5 epoch，best loss `0.8723`、accuracy `0.6278`、MAE `0.5634`、macro-F1 `0.3148`、Spearman `0.3064`。结论：early stopping 后 validation-loss 最优的 SNP-MLP 可达到 tree baseline accuracy（约 `0.627`），但 macro-F1 低于未恢复 best-loss checkpoint 时的后期过拟合高点，也低于 LightGBM macro-F1 `0.3354`；当前更适合作为强 neural SNP baseline，而不是已解决类别不平衡的主模型。
- [2026-06-14 09:41:07 CST] 继续自动推进 SNP-MLP class-balanced loss：本地 `baseline_snp_mlp_core_smoke.py` 新增 `--loss-weighting none|balanced`，默认 `none` 保持兼容；`balanced` 仅用 train split 的 observed labels 为每个 trait 计算 inverse-frequency class weights，训练 loss 使用 weighted cross-entropy，validation loss/early stopping 仍保持 unweighted 以便和前序结果可比，manifest 记录 `loss_weighting`。验证：`py_compile`、`sh -n`、balanced CPU tiny smoke、manifest `json.tool` 和独立代码复审均通过。正式 balanced run（top512 SNP union=4218、hidden_dim=256、dropout=0.4、weight_decay=1e-3、lr=5e-4、epochs=30、early_stop_patience=3、seed42）early stop 于 7 epoch，best epoch `4`，best/unweighted val loss `0.9774`、accuracy `0.5572`、MAE `0.6450`、macro-F1 `0.3606`、Spearman `0.3025`。结论：class-balanced training 显著提高 macro-F1，超过 LightGBM `0.3354`、XGBoost `0.3115` 和未加权 SNP-MLP best-loss checkpoint `0.3148`，但 accuracy/MAE 明显下降；这说明类别不平衡确实是 macro-F1 瓶颈，下一步应在 balanced/ordinal/multi-objective 间折中，而不是单看 validation loss。
- [2026-06-14 09:53:07 CST] 自动补齐 SNP-to-gene mapping 消融输入：扩展 `build_snp_gene_map.py` 支持 `--mode window|nearest`，并新增 `build_snp_gene_map_ablation.sh` 批量生成 body-only、±2 kb、±10 kb、nearest-gene 四套 mapping。代码通过 `py_compile`、SLURM `sh -n`、fixture smoke、静态扫描和独立代码复审；初审发现 nearest-gene 对嵌套/长基因会漏判以及输出发布非原子，已修复为 sweep-line nearest assignment 和 tmp/bak 安全发布后复审通过。SLURM 作业 `8562949` 在 `cu` 分区完成（`COMPLETED`, exit `0:0`, elapsed `00:00:24`, batch MaxRSS `1148K`）。验证结果：body-only mapped SNPs/edges/genes-with-SNPs=`90,886/99,313/24,565`；±2 kb=`194,406/262,227/32,960`；±10 kb=`313,987/847,820/34,724`；nearest-gene=`365,710/365,710/29,460`。这些 mapping 仅写入本地数据区 `data/3krice/processed/snp_gene_map/`，不上传 GitHub；下一步可用其中 body-only/±2kb/±10kb/nearest 重建 gene graph 与训练输入，做正式 SNP-to-gene mapping 消融。
- [2026-06-14 10:15:08 CST] Cron 复核 Phase 5 model-input smoke 并继续 mapping 消融图构建：`sacct` 确认作业 `8562921` 仍为 `COMPLETED|0:0|00:00:08`，manifest/report 再次验证 `status=ok`、`X=3000x365710`、`Y/mask=3000x35`、10 core traits、graph `34139` nodes / `341030` directed edges、random split train/val/test=`1586/340/340`。新增本地 SLURM 脚本 `build_gene_graph_mapping_ablation.sh`（仅本地，不上传 GitHub），通过 `sh -n`、`git diff --check`、静态安全扫描和独立代码复审。SLURM 作业 `8562950` 在 `cu` 分区完成（`COMPLETED`, exit `0:0`, elapsed `00:00:38`, batch MaxRSS `18116K`），为 body-only、±2 kb、±10 kb、nearest-gene 四套 mapping 生成 chr-neighbor/random-degree 图：body-only genes/directed edges=`24,565/245,290`；±2 kb=`32,960/329,240`；±10 kb=`34,724/346,880`；nearest-gene=`29,460/294,240`。输出位于本地数据区 `data/3krice/processed/gene_graph/mapping_ablation/`，不上传 GitHub；下一步需要让训练脚本参数化 `snp_gene_map` 路径，才能用这些 mapping 做真正的 bounded pilot。
- [2026-06-14 10:19:09 CST] 继续自动推进 SNP-MLP balanced 强度退火：在 `baseline_snp_mlp_core_smoke.py` 中新增 `--balance-alpha`，默认 `1.0`，用于把 class weight 从 unweighted 平滑插值到 full balanced：`(1-alpha)+alpha*balanced_weight`；`alpha=0` 近似未加权，`alpha=1` 等同 full balanced，manifest 记录 `balance_alpha`。验证：`py_compile`、`sh -n`、alpha=0.5 CPU tiny smoke、manifest `json.tool`、静态扫描和独立代码复审均通过。正式 top512 SNP-MLP alpha 网格（hidden_dim=256、dropout=0.4、wd=1e-3、lr=5e-4、patience=3、seed42）结果：alpha `0.25` early stop 5 epoch，best loss `0.8883`、accuracy `0.6264`、MAE `0.5586`、macro-F1 `0.3319`、Spearman `0.3092`；alpha `0.50` early stop 5 epoch，best loss `0.9155`、accuracy `0.6177`、MAE `0.5752`、macro-F1 `0.3563`、Spearman `0.3075`；alpha `0.75` early stop 7 epoch，best loss `0.9449`、accuracy `0.5961`、MAE `0.5986`、macro-F1 `0.3503`、Spearman `0.3036`。结论：alpha `0.50` 是当前最佳折中，macro-F1 接近 full balanced `0.3606`，但 accuracy/MAE 明显好于 full balanced；alpha `0.25` 保留 tree-baseline accuracy/MAE，但 macro-F1 未超过 LightGBM `0.3354`。后续可围绕 alpha `0.4–0.6` 做多 seed 或引入 ordinal loss。
- [2026-06-14 10:33:14 CST] 继续自动推进 SNP-to-gene mapping bounded pilot：训练脚本 `rice_geneformer_omtl_train.py` 新增 `--graph-root` 与 `--snp-gene-map`，默认仍为 `baseline` / `window_5kb` 以保持兼容；路径按 component 校验并用 `resolve_under` 限制在 `processed/gene_graph` 与 `processed/snp_gene_map` 下，manifest 记录 `graph_root` 和 `snp_gene_map`。验证：`py_compile`、默认 `window_5kb` CPU smoke、`body_only` mapping CPU smoke、manifest `json.tool`、静态扫描和独立代码复审均通过。随后在 `gpu10` 物理 GPU0 上运行两组差异最大的 mapping bounded pilot（top512 + trait_attention + GraphEncoder、genes=2048、hidden_dim=64、batch=16、15 epoch、50 steps/epoch、seed42）：`body_only` 输出 `status=ok`、`graph_edges_used=20450`、best epoch `11`、best val loss `0.3779`、final accuracy `0.5767`、MAE `0.5855`、macro-F1 `0.2857`、Spearman `0.2020`；`nearest_gene` 输出 `status=ok`、`graph_edges_used=20450`、best epoch `11`、best val loss `0.3759`、final accuracy `0.5719`、MAE `0.5936`、macro-F1 `0.2863`、Spearman `0.2006`。两组 manifest 均通过 `json.tool`，checkpoint_best/last 均非空约 1.51 MB（本地数据区，不上传 GitHub）。初步结论：body-only 与 nearest-gene 在当前模型中与原 window_5kb/GraphEncoder 指标非常接近，mapping 方式不是当前主瓶颈；仍需补 ±2 kb / ±10 kb 或多 seed 后再最终冻结结论。

## 8. 下一步执行优先级

1. 不再把当前 chr / prefix random / STRING / fusion 的轻量 GraphEncoder 结果解释为图结构特异增益；若继续图方向，应转向 Plant Reactome/KEGG pathway、co-expression atlas，或重新设计 relation-aware graph encoder。
2. SNP-MLP class-balanced alpha 网格显示 alpha `0.50` 是当前较好折中（macro-F1 `0.3563`、accuracy `0.6177`）；下一步可围绕 alpha `0.4–0.6` 做多 seed 或引入 ordinal/class-balanced 混合目标。
3. 已完成 `max_snps_per_gene=16/32/64/128` 的 seed42 对齐消融；指标近似持平，暂不作为主瓶颈继续深挖。
4. body-only 与 nearest-gene mapping pilot 已完成且接近原 window_5kb；下一步补 ±2kb / ±10kb 或多 seed，若仍接近即可冻结 mapping 不是主瓶颈。
