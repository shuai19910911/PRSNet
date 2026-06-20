# SmartPRSOrdinalNet 架构设计说明

更新时间：2026-06-20

## 1. 设计目标

基于 PRSNet（polygenic risk score network，多基因评分网络）原始层级结构，借鉴 Huihui Li（李慧慧）团队/合作论文中的 DNNGP（deep neural network genomic prediction，深度神经网络基因组预测）、iGEP（integrated genomic-enviromic prediction，基因组-环境组整合预测）、AutoGS（automated genomic selection，自动基因组选择）、EXGEP（explainable genotype-by-environment prediction，可解释基因型环境互作预测）思路，设计一个适合水稻 ordinal traits（有序等级性状）的新模型候选：

SmartPRSOrdinalNet: PRSNet raw-SNP hierarchy + Huihui-Li-style modular multimodal fusion + conservative expert gate.

中文定位：SmartPRSOrdinalNet 是一个基于 PRSNet（多基因评分网络）层级结构、引入多模态拼接融合和专家门控的水稻多等级性状基因组预测模型。

## 2. 为什么不是简单复制 Huihui Li 的模型

Huihui Li 路线的核心不是“照搬一个 CNN（卷积神经网络，用滑动窗口提局部特征）”或“单纯堆深网络”，而是：

1. modular encoder（模块化编码器）：不同数据源先独立编码。
2. learnable fusion（可学习融合）：拼接后用 MLP（多层感知机）或 attention（注意力机制）学习交互。
3. strong baseline awareness（强基线意识）：保留 PRS/GBLUP/树模型等强对照，不把深度学习绝对化。
4. explainable decision support（可解释决策支持）：输出能解释哪个专家/模态贡献更大。
5. leakage-safe benchmark（防泄漏基准）：GWAS（全基因组关联分析）和 PRS（多基因评分）只来自 train split（训练集划分）。

我们的任务是 rice ordinal genomic prediction（水稻有序等级基因组预测），不是连续性状回归，所以保留 ordinal head（有序等级输出头）作为主头，类别头只做辅助。

## 3. 架构总览

输入：原始 SNP dosage（SNP剂量，0/1/2基因型编码）

流程：

1. PRSNet SNP2Gene branch（PRSNet SNP到基因分支）
   - raw SNP dosage（原始SNP剂量）
   - multi-kernel SNP-to-gene aggregation（多核SNP到基因聚合）
   - significance-guided dropout（显著性引导随机失活，低GWAS信号位点更容易被drop）
   - 输出 gene tokens（基因token，每个基因一个向量）

2. Gene graph branch（基因图分支）
   - gene tokens（基因token）进入 GIN-like message passing（类似GIN的图消息传递）
   - 使用 edge_index（基因图边索引）真实传播信号
   - manifest（训练记录）记录 graph_edges_used（实际使用图边数）

3. Trait-specific gene attention（性状特异基因注意力）
   - 每个 trait（性状）一个 trait query（性状查询向量）
   - 从 gene tokens（基因token）中读出 trait-specific representation（性状特异表示）

4. Global PRS branch（全局多基因评分分支）
   - 每个性状使用 train-fold GWAS beta（训练折GWAS效应值）选 top PRS SNPs（高权重SNP）
   - 计算 trait-wise PRS score（逐性状多基因评分）
   - 用小 MLP（多层感知机）编码成 PRS token（PRS向量）

5. Multimodal fusion（多模态融合）
   借鉴 Frontiers 2024 / G3 2023 多模态DL结构，不把信息简单相加，而是拼接后再学习：

   fusion input = [gene_trait_token, trait_query, prs_token, global_gene_token]

   - gene_trait_token（性状从基因图读出的向量）
   - trait_query（性状身份向量）
   - prs_token（该性状全局PRS向量）
   - global_gene_token（全局基因图平均向量）

   拼接后进入 fusion MLP（融合多层感知机）+ residual fusion blocks（残差融合块）+ trait mixer（性状间注意力混合）。

6. Three-expert output heads（三专家输出头）
   - ordinal expert（有序专家）：主输出，保持类别顺序。
   - categorical expert（类别专家）：辅助 macro-F1（宏平均F1，重视少数类别）。
   - PRS-only expert（PRS单独专家）：保留传统PRS信号，避免深度分支过拟合。

7. Conservative expert gate（保守专家门控）
   - gate（门控）输出三个专家权重。
   - 初始化偏向 ordinal expert（有序专家）和 PRS-only expert（PRS单独专家），压低 categorical expert（类别专家），避免类别头早期主导。
   - 输出 final_prob（最终类别概率）和 expert_weight（专家权重，可解释）。

## 4. 与当前 RiceGraphOrdinalNet v2 的区别

| 模块 | RiceGraphOrdinalNet v2 | SmartPRSOrdinalNet |
|---|---|---|
| SNP输入 | 预缓存 gene features（基因特征） | raw SNP dosage（原始SNP剂量）直接进 PRSNet SNP2Gene |
| PRSNet关系 | 更像 gene-feature 模型 | 明确保留 PRSNet raw SNP → gene → trait 层级 |
| Huihui Li 借鉴点 | 有 trait token（性状token）和类别辅助头 | 加入多模态分支编码、concat fusion（拼接融合）、专家门控、PRS-only专家 |
| 图使用 | 可选 graph message passing（图消息传递） | 默认 GNN（图神经网络）层消费 edge_index |
| 输出头 | ordinal + categorical 双头 | ordinal + categorical + PRS-only 三专家 |
| 可解释性 | 类别融合权重 | expert_weight（专家权重）+ 图边记录 + PRS分支 |

## 5. 防泄漏设计

1. GWAS p-values（全基因组关联p值）来自 active split（当前划分）的 train-fold GWAS（训练折GWAS）。
2. PRS beta（多基因评分效应值）来自同一 active train split（当前训练划分）。
3. selection_metric（模型选择指标）只用 validation split（验证集）。
4. test split（测试集）只在最终评估时用；导出的 test prediction npz（测试预测文件）不含 y/mask（真实标签/观测掩码）。
5. `--graph-root`（图目录参数）做原始字符串安全校验：拒绝绝对路径、反斜杠、空组件、`.` 和 `..`，防止路径穿越。

## 6. 已落地文件

脚本：

/home/user/zhangzhishuai/myhermes/PRSNet/scripts/model/rice_smart_prs_ordinal_net_train.py

说明：该脚本位于 `/scripts/` 下，当前仓库 `.gitignore`（Git忽略规则）会忽略整个 scripts 目录，因此它是本地研究脚本，不会自动出现在 `git status` 中。

## 7. 已完成验证

1. `py_compile`（Python语法编译检查）：通过。
2. 路径安全负测：`../bad`、`./bad`、`a/./b` 三种 `--graph-root`（图目录参数）均按预期失败。
3. CPU smoke（冒烟测试，小规模1轮训练）：通过。
4. manifest JSON（训练记录JSON）：`python3 -m json.tool` 可解析。
5. test predictions（测试预测文件）：不包含 y/mask（真实标签/观测掩码）。
6. static scan（静态安全扫描）：未发现硬编码密钥、shell注入、eval/exec、pickle反序列化或SQL注入。
7. independent code review（独立代码审核）：最终通过。

smoke 输出目录：

/home/user/zhangzhishuai/myhermes/PRSNet/data/3krice/processed/smoke_smart_prs_ordinal_net_20260620_v3

## 8. 推荐正式训练命令

功能：在 gpu10（12.12.12.210）上用2号卡训练 SmartPRSOrdinalNet（新PRSNet拼接架构）正式10性状候选模型。
资源估算：1×A100 40GB，CPU 8-16核，RAM 32-64GB，磁盘新增 1-3GB，预计 30-90 分钟；建议先用 GPU2，暂不使用 GPU0。

```bash
CUDA_VISIBLE_DEVICES=2 python scripts/model/rice_smart_prs_ordinal_net_train.py --out-dir data/3krice/processed/rice_smart_prs_ordinal_net_tentrait_top2048_h64_gpu2_seed42_20260620 --device cuda --max-genes 2048 --max-snps-per-gene 32 --global-prs-snps 512 --hidden-dim 64 --n-snp-kernels 4 --n-gnn-layers 1 --attention-heads 4 --batch-size 32 --eval-batch-size 128 --epochs 90 --max-steps-per-epoch 50 --early-stop-patience 12 --selection-metric val_macro_f1
```

功能：如果显存/速度允许，训练更大候选版本。
资源估算：1×A100 40GB，CPU 8-16核，RAM 64GB，磁盘新增 2-5GB，预计 1-3 小时。

```bash
CUDA_VISIBLE_DEVICES=2 python scripts/model/rice_smart_prs_ordinal_net_train.py --out-dir data/3krice/processed/rice_smart_prs_ordinal_net_tentrait_top4096_h96_gpu2_seed42_20260620 --device cuda --max-genes 4096 --max-snps-per-gene 32 --global-prs-snps 1024 --hidden-dim 96 --n-snp-kernels 4 --n-gnn-layers 1 --attention-heads 4 --batch-size 24 --eval-batch-size 96 --epochs 90 --max-steps-per-epoch 50 --early-stop-patience 12 --selection-metric val_macro_f1
```

## 9. 论文叙事建议

可把它写成：

SmartPRSOrdinalNet extends the PRSNet hierarchy into a leakage-aware modular framework for ordinal crop genomic prediction. It combines raw-SNP multi-kernel SNP-to-gene aggregation, graph-based gene message passing, global PRS scoring, trait-token attention, learnable multimodal fusion, and conservative expert gating between ordinal, categorical, and PRS-only predictors.

中文解释：SmartPRSOrdinalNet 把 PRSNet（多基因评分网络）的 SNP→gene→trait 层级扩展为一个防泄漏的作物有序等级预测框架，融合原始SNP多核聚合、基因图消息传递、全局PRS评分、性状token注意力、多模态拼接融合，以及有序/类别/PRS三专家门控。

## 10. 下一步实验建议

1. 先跑默认 top2048/h64/seed42（2048基因、隐藏维度64、随机种子42）正式候选。
2. 如果 macro-F1（宏平均F1，重视少数类别）提升，再做3 seeds（3个随机种子）重复。
3. 如果提升不明显，重点检查 expert_weight（专家权重）：
   - PRS-only权重过高：说明深度分支没学到额外信息。
   - categorical权重过高且MAE变差：说明类别头破坏有序结构。
   - ordinal权重稳定较高但macro-F1低：继续做 validation-only threshold calibration（只用验证集做阈值校准）。
4. 做 ablation（消融实验）：
   - `--gene-prior-mode none`（不用GWAS先验）
   - `--gene-prior-mode random`（随机打乱GWAS先验）
   - `--n-gnn-layers 0`（不用基因图）
   - 减小/增大 `global-prs-snps`（PRS分支SNP数量）
