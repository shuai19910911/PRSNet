# 面向 3K 水稻基因组基因型到表型预测的防泄漏有序标签基准：RiceGeneFormer

短标题：水稻表型预测的防泄漏基因感知基准

文章类型：面向 Briefings in Bioinformatics 的 Original Article / 计算生物信息学 benchmark 稿件中文版本

状态：接近投稿级完整中文稿。所有定量结论仅限于 2026-06-15 已验证项目产物；正式投稿前仍需完成参考文献格式、代码 DOI 和仓库 release。

## 摘要

植物基因型到表型预测越来越依赖高密度基因组标记，但许多作物群体规模表型并不是连续数值，而是具有等级顺序的 descriptor code（描述符代码）。这些表型通常类别不均衡，少数类别样本很少。因此，模型既需要尊重标签的有序结构，也需要避免 GWAS 信号、特征选择或模型选择过程把验证集/测试集信息泄漏进训练过程。我们构建了 RiceGeneFormer-OMTL，一个面向 3K Rice Genome 表型预测的基因感知有序多任务框架，并用它建立了十个核心水稻有序性状的防泄漏 benchmark。该框架整合 train-fold GWAS 先验、SNP-to-gene 聚合、图感知 gene token、trait-query 解码、gated top-SNP 融合和 cumulative-link 有序预测头。所有最终神经模型指标均用完整验证/测试角色进行确定性重算。

RiceGeneFormer 在无蒸馏设置下测试 macro-F1 为 0.3229 +/- 0.0062，在弱 expected-score 蒸馏下为 0.3292 +/- 0.0057。该结果接近使用 train-fold top SNP 的 LightGBM（macro-F1 0.3380），但低于 class-balanced SNP-MLP baseline（macro-F1 0.3644 +/- 0.0106 到 0.3781 +/- 0.0029）。Gated top-SNP fusion 和 mild class balancing 是最有用的设计；graph identity、SNP-to-gene window、threshold calibration 和 expected-score distillation 的收益有限或不稳定。Southeast Asia、South Asia 和 East Asia 的 cross-region benchmark 不支持 RiceGeneFormer 具有地理分布偏移鲁棒性优势。Gene-attention audit 进一步显示，受限 gene selection 会限制生物学解释；train-fold GWAS-top2048 gene panel 也未提高性能或恢复已知基因。

因此，RiceGeneFormer 当前应定位为可复现的 diagnostic benchmark framework，而不是 neural model superiority 声明。该 benchmark 说明了基因感知作物模型何时能接近 marker-level baseline、在哪里失败，以及在提出 robustness 或 biological discovery 声明前还需要哪些证据。

## Key points / 关键要点

- RiceGeneFormer-OMTL 是面向 3K 水稻基因型到表型预测的防泄漏、基因层级、有序多任务框架。
- Train-fold GWAS 先验、top-SNP selection、checkpoint selection 和 deterministic test evaluation 被明确分离，以降低验证/测试泄漏风险。
- 在当前 random split 下，RiceGeneFormer 接近 LightGBM，但 macro-F1 仍低于 balanced SNP-MLP baseline。
- Cross-region 和 gene-attention audit 被保留为主文结果，因为它们定义了 robustness 和 biological interpretation 的证据边界。

## 关键词

3K Rice Genome；基因型到表型预测；有序分类；GWAS；macro-F1；SNP-to-gene 聚合；benchmark；作物生物信息学

## 引言

高密度作物基因组资源使基因型到表型预测从小规模候选位点分析进入群体规模建模阶段。在水稻中，3K Rice Genome Project 和 SNP-Seek 提供了数千份 accession（材料/品种资源）的全基因组变异、metadata 和表型记录。这类资源不仅支持传统 GWAS，也支持更复杂的问题：模型能否联合利用许多标记？能否在多个性状之间共享信息？能否保留基因、通路或 trait-specific module 这样的生物学结构？对计算育种而言，一个模型如果只在单一随机切分中表现好但没有清楚的证据边界，其价值有限；相反，一个能够说明哪些信号稳定、哪些信号只是 top-SNP shortcut、哪些解释还缺证据的 benchmark 更适合投稿到 BIB 这类重视方法可复现性的期刊。

第一个核心难点是表型标签本身。许多公开植物表型不是连续测量值，而是 descriptor code。例如某个形态或农艺性状可能用 1、3、5、7 表示不同等级。它们是 ordinal（有序）的：类别之间有顺序，但相邻类别之间不一定等距。它们同时又是 sparse 和 imbalanced：有些类别样本很多，有些类别样本很少。如果只看 accuracy，模型可能只预测多数类别也能得到不错分数；如果过度 class balancing，macro-F1 可能提高，但 accuracy 和 ordinal MAE 可能变差。因此，水稻这类 descriptor phenotype benchmark 不能只用一个指标，而应该同时报告 macro-F1、accuracy 和 MAE。

第二个核心难点是 leakage（信息泄漏）。在 genomic prediction 里，模型经常使用 GWAS p-value 做特征选择或先验。如果先用全体样本计算 GWAS，再划分训练/验证/测试集，那么验证集和测试集表型已经影响了模型输入。这会夸大模型性能，尤其在样本量不大的作物数据中更危险。类似问题也出现在 top-SNP selection、threshold calibration 和 checkpoint selection 中。一个公平 benchmark 必须要求所有 GWAS prior 和 top-SNP feature 都只来自当前 active training fold，最终测试指标必须用完整 test role 重新确定性计算，而不是使用训练循环中的采样日志。

近年的 BIB benchmark 文章还反复强调另一个原则：benchmark competitiveness（基准测试中表现接近或优秀）不等于 generalization（真正跨数据/跨群体泛化）。Drug-response prediction 的 cross-dataset benchmark、AMR phenotype prediction、multi-population genomic prediction，以及 protein-ligand scoring 的 blind benchmark 都说明，一个方法可能在训练集和测试集共享相似结构时表现较好，但在外部数据、divergent lineages 或不同 population background 下明显下降。因此，本文把 random split benchmark 和 held-out geographic-region benchmark 分开写。Random split 衡量模型在严格防泄漏流程下能否学习有效信号；region split 则测试这种信号是否能经受 accession origin 改变带来的 harder distribution shift。

第三个现实问题是 marker-level baseline 很强。LightGBM、XGBoost 或 SNP-MLP 如果直接使用 train-fold top SNP，可以走一条很短的路径：从强关联 SNP 到表型标签。它们不一定有基因结构解释，但预测能力很强。Gene-aware neural model 的吸引力在于可以把 SNP 聚合成 gene token，利用 gene graph，学习 trait-specific gene attention，并产生解释性 hook。但是，模型包含 gene annotation 或 attention weight 并不等于完成了 biological validation；模型包含 graph 也不等于证明了 robustness。因此，本文的写法必须把强 baseline 放在中心位置，同时把 negative control 放在主文，而不是隐藏在 supplement。

与本文写法最接近的 BIB 文章之一是 “Should we really use graph neural networks for transcriptomic prediction?”。它并没有把 biological network 当成天然优势，而是把 network-aware model 当成需要被 benchmark 验证的假设。本文采用同样立场：gene graph、SNP-to-gene bag 和 attention layer 是有用的实验基础设施，但它们的价值必须通过与更简单 baseline 的公平比较来证明。这也是本文把 graph identity、mapping-window、cross-region 和 gene-attention audit 都放在主文的原因，即使部分结果是 negative finding。

本文提出 RiceGeneFormer-OMTL，定位为 3K 水稻基因型到表型预测的防泄漏有序标签 benchmark 和 diagnostic framework。模型整合 train-fold GWAS prior、SNP-to-gene aggregation、gene graph token、trait-query decoding、cumulative-link ordinal head 和 gated top-SNP fusion branch。我们将其与使用同一 train-fold top-SNP 信息的 LightGBM、XGBoost 和 balanced SNP-MLP 进行比较，并进一步用 ablation、cross-region benchmark 和 gene-attention audit 来限定结论边界。本文刻意采用保守叙事：RiceGeneFormer 不是 marker-level predictor 的通用替代品，而是一个可复现框架，用来衡量 gene-aware model 与强 top-SNP shortcut 之间的差距。

## 结果

### 从 3K 水稻基因组构建防泄漏有序表型 benchmark

我们首先构建了一个同时保留 3K 水稻任务结构和严格 leakage control 的 benchmark。主 genotype 输入是 3K core SNP PLINK 资源，处理后得到 3,000 个 accession 和 365,710 个 SNP 的 dosage matrix。Phenotype 来自 3KRG morpho-agronomic phenotype spreadsheet，并通过 metadata 和 stock identifier 与 genotype 顺序对齐。最终 phenotype matrix 包含 35 个 descriptor-code traits 及对应 observation mask；其中十个覆盖度较高的 ordinal traits 被选为核心建模目标。

主 random split 包含 1,586 个 training、340 个 validation 和 340 个 test accession。关键规则是：所有 GWAS p-value 只用当前 active split 的 training samples 计算。每个 core trait 得到一个长度为 365,710 的 p-value vector。这些 p-value 只在两个地方使用：作为 RiceGeneFormer 的 prior，以及作为 baseline top-SNP feature selection 的依据。这样可以防止 validation/test labels 影响 GWAS prior、top-SNP selection 或模型输入。Cross-region split 和后续 diagnostic pilot 也使用同样规则。

该 benchmark 还显式暴露了 class imbalance。多个 core traits 的 training set 被一个主要类别占据。因此，我们同时报告 macro-F1、accuracy 和 ordinal MAE。Macro-F1 是每个类别 F1 的非加权平均，更能反映 minority class；accuracy 反映总体预测正确率；MAE 用类别单位衡量 ordinal error。三者结合才能判断 class balancing 是否真的改善模型，还是只是在 minority class 和 majority class 之间移动 operating point。

### RiceGeneFormer-OMTL 整合 GWAS prior、gene token 和 ordinal multi-task decoding

RiceGeneFormer-OMTL 的目标是测试 gene-aware neural architecture 能否使用与强 baseline 相同的 train-fold marker 信息，同时保留 gene-level biological organization。每个 gene 由一个 GeneBag 表示，即从该 gene body 及其窗口中收集一组有上限的 SNP dosage。SNP-to-gene aggregation 把 marker dosage 转换成 gene token。Train-fold GWAS p-value prior 作为额外输入，使模型可以利用当前 split 内训练样本的关联信号，同时不暴露 validation/test labels。

Gene tokens 经过轻量 graph encoder。主图连接染色体邻近 genes；STRING protein association graph 和 fusion graph 用于 ablation。这里的 graph encoder 是 bounded lightweight message passing，不声称完整测试所有 biological network 的价值。Trait-specific prediction 由 trait-query attention 完成：每个核心 ordinal trait 有一个 query representation，关注 gene-token sequence，并接入 cumulative-link ordinal head。

Pilot 结果显示，直接 marker signal 仍然非常重要。因此最终模型包含 top-SNP branch，输入为多个 core traits 的 train-fold top SNP union。最强配置不是简单相加，而是 gated top-SNP fusion：模型为每个 trait token 学习注入多少 marker-level signal。最终训练 recipe 使用 gated fusion、mild class-balanced ordinal loss（alpha0.25）和 validation macro-F1 checkpoint selection。Expected-score distillation 来自 SNP-MLP teacher，但由于收益很小，只作为 exploratory component。

### 强 top-SNP baseline 定义当前预测上限

在解释 gene-aware model 之前，我们先在同样 leakage-control 规则下建立 marker-level baseline。LightGBM 和 XGBoost 按每个 core trait 单独训练，使用 train-fold top512 SNP。最终 test role 上，LightGBM 得到 macro-F1 0.3380、accuracy 0.6263、MAE 0.5630；XGBoost 得到 macro-F1 0.3161、accuracy 0.6332、MAE 0.5512。这说明 train-fold top-SNP features 本身就包含强预测信号，是 gene-aware model 必须面对的严格参照。

最强 baseline 是 multi-task SNP-MLP，输入为 core traits 的 train-fold top-SNP union。Class-balance interpolation 很关键。Alpha0.40 设置下，SNP-MLP 的 test macro-F1 为 0.3644 +/- 0.0106，accuracy 为 0.6317 +/- 0.0023，MAE 为 0.5639 +/- 0.0117。Alpha0.60 设置下，macro-F1 提高到 0.3781 +/- 0.0029，但 accuracy 降到 0.6148 +/- 0.0091，MAE 升到 0.5899 +/- 0.0176。这说明更强 minority-class weighting 可以提高 macro-F1，但会牺牲总体 accuracy 和 ordinal fidelity。

这组 baseline 定义了本文最重要的 benchmark 结论：gene-aware model 即使接近 tree-based top-SNP performance，也可能仍低于 balanced marker-level neural baseline。因此本文的问题不是 RiceGeneFormer 是否能宣称 superiority，而是它能否提供一个可复现框架，用来测量 gene-aware modelling 与强 marker shortcut 之间的差距。

### Gated top-SNP fusion 和 mild class balancing 产生最强 RiceGeneFormer 配置

RiceGeneFormer 最稳定的改进来自 learned gate 融合 top-SNP branch，以及 mild class-balanced ordinal loss。早期 bounded pilots 显示，trait-specific attention、top-SNP input 和 gated fusion 比 graph identity 或 SNP-to-gene mapping 改动更有用。更强 class balancing 在部分 pilot 中提高 macro-F1，但会降低 accuracy 和 MAE，因此最终使用 alpha0.25 作为保守 operating point。

在 deterministic full-role test evaluation 下，无蒸馏 RiceGeneFormer 的 macro-F1 为 0.3229 +/- 0.0062，accuracy 为 0.5895 +/- 0.0029。加入 weight 0.10 的 expected-score distillation 后，macro-F1 提高到 0.3292 +/- 0.0057，accuracy 为 0.5894 +/- 0.0049，MAE 为 0.5948 +/- 0.0067。该结果接近 LightGBM macro-F1，但低于最强 balanced SNP-MLP baselines。

Per-trait 结果解释了差距来源。RiceGeneFormer 在类别较少或不那么极端不均衡的 traits 上表现较好，例如 CUDI_CODE_REPRO 和 PTH；在强不均衡 multi-class traits 上表现较差，例如 CUST_REPRO 和 SPKF。这与 class distribution analysis 一致：gene-token compression 和 multi-task sharing 不能自动解决 minority-class scarcity。对 BIB benchmark 文章来说，这些 hard-trait 结果必须保留，因为它们指明了后续模型优化应重点评估的位置。

### Expected-score distillation 只有弱增益

我们测试 SNP-MLP teacher 是否能把 marker-level signal 转移到 gene-aware model。Teacher predictions 为 train、validation 和 test roles 导出，但 RiceGeneFormer 训练时只使用 train-role teacher outputs。Student cumulative-link logits 被转换为 expected ordinal scores，再用 masked MSE 与 teacher expected scores 对齐。

结果是弱正向但不稳定。Distillation weight 0.10 将 test macro-F1 从 0.3229 +/- 0.0062 提高到 0.3292 +/- 0.0057；weight 0.20 降到 0.3251 +/- 0.0054。收益不单调，也没有缩小到 balanced SNP-MLP 的主要差距。因此 expected-score distillation 只能写成 weak-positive component，不能写成核心突破。后续可以测试 probability-level 或 logit-level distillation，但当前证据不支持更强说法。

### Graph identity、SNP-to-gene mapping 和 threshold calibration 没有稳定增益

几个生物学动机模块提高了框架完整性，但没有在当前 bounded setting 下产生稳定预测增益。Chromosome-neighbour、edge-matched random、STRING 和 chromosome+STRING fusion graphs 的 pilot macro-F1 非常接近。这说明当前 lightweight graph encoder 没有提取出 graph-specific advantage，尽管 graph construction 和 STRING mapping 已经成功集成。

SNP-to-gene mapping variant 的影响也有限。Body-only、+/-2 kb、+/-5 kb、+/-10 kb 和 nearest-gene mappings 的 pilot macro-F1 相近，说明在当前模型容量和 top-SNP fusion 设置下，mapping-window choice 不是主要瓶颈。Post-hoc threshold calibration 也类似：train-calibrated thresholds 在部分短 pilot 中有帮助，但对 full-step checkpoints 会降低 macro-F1、accuracy 或 MAE。

这些 negative controls 必须作为主文结果，而不是隐藏起来。当前实现证明的是 leakage-aware gene-aware benchmark framework，而不是 stable biological-graph advantage 或 mapping-window-specific predictive effect。对 BIB 读者而言，这仍然有价值，因为它说明哪些 biologically motivated modules 需要更强架构或更大数据才能支撑强结论。

### Cross-region 和 gene-attention audit 定义生物学结论边界

我们进一步测试 RiceGeneFormer 是否在 geographic distribution shift 下具有优势。构建了 Southeast Asia、South Asia 和 East Asia 三个 held-out region splits。每个 split 中，baseline 和 RiceGeneFormer 都使用 split-specific train-fold top-SNP 和 GWAS 规则。RiceGeneFormer 使用最终 gated top-SNP fusion、alpha0.25 和 validation macro-F1 checkpoint selection，并做三 seed 训练。

结果不支持 cross-region robustness advantage。相对 SNP-MLP alpha0.60 baseline，RiceGeneFormer 的 macro-F1 gap 在 Southeast Asia 为 -0.0551，在 South Asia 为 -0.0506，在 East Asia 为 -0.0349。因此当前证据不能声称 gene-aware model 比强 marker-level neural baseline 更能泛化到 held-out geographic regions。

我们还审计了 gene-attention outputs。Scores 从 w=0.10 RiceGeneFormer seeds 42/43/44 的最佳 checkpoints 导出，并在 deterministic test-role samples 上求平均。审计发现一个关键解释限制：bounded runs 使用 graph-node order 的前 2,048 个 genes，相当于 chromosome 1 前缀，而不是 genome-wide candidate panel。因此，初步 known-gene panel 中许多经典水稻基因，包括 SD1、GS3、GW2、qSW5/GW5、DEP1、Hd1 和 Hd3a，并不在 attention layer 可见范围内。Gn1a 在可见 subset 中并在若干 traits 的 top200 中出现，但单个观察不足以支持 biological discovery claim。

为测试能否纠正 prefix limitation，我们构建了 split-specific GWAS-top2048 gene graph。Genes 根据 core traits 中最大 train-fold GWAS evidence 排名，选取 top 2,048 genes，再用 chromosome-neighbour edges 连接。Seed42 retraining pilot 的 deterministic test macro-F1 为 0.2990，accuracy 为 0.5836，MAE 为 0.5878，Spearman 为 0.2417，低于当前最佳 RiceGeneFormer family。其 top attention genes 也没有恢复 preliminary known-gene panel。因此，attention outputs 当前只能作为 diagnostic artifacts，不能作为 validated candidate-gene evidence。

### Benchmark 给出的实际使用建议

该 benchmark 给出的结论不是“唯一推荐某个模型”，而是一个有条件的 practical recommendation。如果当前目标只是最大化 3K rice random split 上的 macro-F1，balanced SNP-MLP baseline 仍是已评估模型中最强选择。Alpha0.60 的 macro-F1 最高，而 alpha0.40 在 accuracy 和 MAE 上更平衡。LightGBM 仍然是一个重要强 baseline，因为它简单、快速，并且 macro-F1 与 RiceGeneFormer 接近。因此，若只考虑当前 random-split prediction accuracy，RiceGeneFormer 不能写成默认推荐模型。

RiceGeneFormer 更适合用于严格 leakage control 下的 gene-aware diagnostic modelling。它把 train-fold GWAS prior、SNP-to-gene aggregation、graph token、trait-specific decoding、top-SNP shortcut、distillation、cross-region testing 和 attention export 放在同一个可复现框架里。换句话说，它的角色类似 BIB 中 drug-response、m6A、AMR 和 variant-calling benchmark 文章中的标准化评估平台：统一比较流程，暴露看似有吸引力的建模选择在哪里失败，并阻止没有证据的 biological 或 robustness 声明。当前稿件的主要贡献是 benchmark 和 diagnostic protocol，而不是宣称 RiceGeneFormer 应直接替代育种中的 marker-level baseline。

## 方法

### 数据来源和样本对齐

Genotype 和 phenotype 资源来自公开 3K Rice Genome / SNP-Seek。主 genotype 输入为 3K core SNP PLINK resource，处理成 3,000 accessions × 365,710 SNPs 的 dense dosage matrix。Phenotype labels 来自 3KRG morpho-agronomic phenotype spreadsheet。样本对齐使用 accession metadata 和 phenotype stock identifiers，将 phenotype rows 匹配到 genotype order。Benchmark 将 phenotype 存为 multi-task label matrix，并配套 observation mask。

### Phenotype encoding 和 core trait selection

处理后的 phenotype matrix 包含 35 个 descriptor-code traits。主实验选择十个 high-coverage ordinal traits。Labels 被视为 ordered categorical classes。缺失 phenotype entry 被 mask，不参与训练 loss 或指标计算。每个 observed trait 的 labels 被编码为有序 class indices，同时保留原始顺序用于 MAE 和 rank-based metrics。

### Split construction 和 leakage control

主 split 是 random_seed42，其中 supervised labels 分为 1,586 training、340 validation 和 340 test accessions。所有 GWAS priors、top-SNP feature sets、class weights、calibration decisions 和 checkpoint-selection decisions 都限制在当前实验允许的 training/validation roles 内。Test labels 只用于最终 deterministic evaluation。Cross-region experiments 对 Southeast Asia、South Asia 和 East Asia 使用 held-out roles，并应用同样 split-aware rule。

### Train-fold GWAS prior generation

对每个 split 和 core ordinal trait，GWAS p-value vectors 只用 training samples 生成。每个 vector 长度为 365,710，与 SNP matrix 对齐。P-values 用作 RiceGeneFormer gene-level priors，也用作 baseline top-SNP feature selection scores。Pipeline 检查 shape、finite values、valid p-value range 和 split-specific path identity，避免不同 split 或 trait 的 priors 混用。

### SNP-to-gene mapping 和 graph construction

Rice gene coordinates 来自 Ensembl Plants release 61 的 IRGSP-1.0 assembly annotation。主 SNP-to-gene mapping 使用 gene body +/-5 kb window。Body-only、+/-2 kb、+/-10 kb 和 nearest-gene mappings 用于 ablation。Gene graphs 建在 mapped gene node set 上。主图连接染色体相邻 genes。STRING v12.0 Oryza sativa Japonica Group protein associations 被映射到 project gene IDs，使用 combined score >=700，并进行 top-20 per-gene pruning。Fusion graph 是 chromosome-neighbour 和 STRING edges 的 union。

### RiceGeneFormer-OMTL architecture

对每个 selected gene，RiceGeneFormer 收集有上限的 mapped SNP dosages，并聚合成 gene token。Gene tokens 与 train-fold p-value priors 结合，投影到 shared hidden space。Lightweight graph encoder 使用 graph neighbourhood 更新 gene tokens。Trait-query attention 为每个 core ordinal phenotype 产生 trait representation。每个 trait representation 输入 cumulative-link ordinal head。Separate top-SNP branch 编码 train-fold top SNP union。最终配置通过 learned gate 将 top-SNP branch 融入 trait tokens。

### Training、model selection 和 deterministic evaluation

最终 full-step RiceGeneFormer runs 使用 AdamW，learning rate 2e-4，weight decay 1e-4，batch size 16，hidden dimension 64，2,048 genes，32 SNPs per gene，top512 SNP fusion，20 epochs。每个 epoch 最多 100 training steps。最终模型使用 alpha0.25 class-balanced ordinal loss。Checkpoints 由 validation macro-F1 选择。最终 validation/test metrics 由 deterministic evaluator 遍历完整 role samples 重新计算，避免把训练循环采样 summary 当成最终指标。

### Baseline models

LightGBM 和 XGBoost 按 trait 单独训练，输入为 train-fold top512 SNPs。Labels 必要时编码为 contiguous classes，并在计算指标前映射回 ordinal labels。SNP-MLP baseline 使用 ten core traits 的 train-fold top SNP union。其主架构为 two-layer feed-forward encoder，hidden dimension 256，dropout 0.4，多 trait classifier heads。Class-balanced loss 只从 training labels 计算。Alpha0.40 和 alpha0.60 用 seeds 42、43、44 评估。

### Distillation

Balanced SNP-MLP teacher predictions 被导出为 numeric arrays。RiceGeneFormer 训练时只使用 train-role teacher outputs。Student cumulative-link logits 被转换为 expected ordinal scores，distillation objective 用 masked MSE 对齐 student 和 teacher expected scores。最终 full-step 设置测试了 distillation weights 0.10 和 0.20。

### Cross-region evaluation

为 Southeast Asia、South Asia 和 East Asia 构建 held-out geographic splits。每个 split 中，top-SNP baselines 和 RiceGeneFormer 都使用 split-specific train-fold GWAS p-values 和 feature sets。RiceGeneFormer 使用 gated top-SNP fusion、class-balance alpha0.25、validation macro-F1 checkpoint selection 和 deterministic full-role test evaluation。每个 region split 运行三个 RiceGeneFormer seeds。

### Gene-attention export 和 GWAS-top2048 audit

Trait-gene attention weights 从 selected RiceGeneFormer checkpoints 导出。Scores 定义为 requested split role 中所有 samples 的 mean trait-to-gene attention。导出文件记录 checkpoint、split、graph、SNP-to-gene map、trait list、gene list 和 score definition。GWAS-top2048 audit 中，gene evidence 定义为该 gene 映射 SNPs 中跨 core traits 的最大 -log10(train-fold GWAS p-value)。Top 2,048 genes 用 chromosome-neighbour edges 连接，并用于 seed42 retraining pilot。

### Metrics

Macro-F1 是 observed labels 上每个类别 F1 的非加权平均。Accuracy 是 predicted class 等于 observed class 的比例。MAE 是 predicted 和 observed ordinal class index 的平均绝对差。Spearman correlation 在可用时作为 expected ordinal score 的 rank-based summary。Neural model summary 默认报告 seeds 的 mean +/- standard deviation。Single-seed tree baselines 在 summary tables 中 standard deviation 记为 0。

## 讨论

RiceGeneFormer 建立了一个防泄漏、基因感知、有序多任务的 3K 水稻 genotype-to-phenotype benchmark。核心信息是边界清楚的：模型接近 LightGBM top-SNP performance，并受益于 gated top-SNP fusion、mild class balancing 和弱 distillation；但在当前 random split 下没有超过强 balanced SNP-MLP baseline。这不是 benchmark 的失败，而是 benchmark 的主要发现。在样本量有限的植物 phenotype panel 中，直接 marker-level predictor 非常强，gene-aware neural architecture 必须在严格 leakage control 下与其比较。

Negative and bounded findings 同样有价值。Graph identity 没有稳定优势，SNP-to-gene mapping window 影响较小，threshold calibration 不稳定，expected-score distillation 不单调。这说明仅仅给模型加入生物结构还不够。Graph module 必须足够表达，gene panel 必须覆盖 relevant candidates，training objective 必须有效转移 marker-level information，同时不损害 ordinal performance。对于 BIB 读者，这些结果提供了设计 crop genomic prediction benchmark 的实践经验。

Cross-region 和 gene-attention audits 对限制 claims 尤其重要。RiceGeneFormer 在 held-out geographic regions 中没有超过 balanced SNP-MLP，所以不能声称 robustness advantage。Gene-attention audit 显示 bounded gene selection 会决定哪些 genes 能被解释。GWAS-top2048 replacement 在原则上纠正了 visibility 问题，但降低了 test macro-F1，也没有恢复 preliminary known-gene panel。因此，attention ranks 只有在 genome-wide coverage、curated QTL resources 和 independent validation 支持下才能作为 candidate-gene evidence。

本研究仍有几个限制。第一，supervised phenotype panel 对 deep multi-task learning 来说较小，多个 traits 存在严重 class imbalance。第二，当前 graph encoder 是 lightweight 且 non-relation-aware，不能代表更强 pathway/graph model 的上限。第三，最终 release DOI 和 repository accession 尚未生成，因此 Data and Code Availability 仍是 future tense。第四，known-gene interpretation 需要正式 identifier 和 citation verification。第五，当前 distillation 使用 expected score，而不是 probability distribution 或 logits。

后续工作应优先做能够改变证据边界的实验，包括 probability-level teacher distillation、relation-aware graph encoder、curated QTL/pathway overlay、外部 crop panel 和更大 multi-environment datasets。如果在更大监督信号或跨环境迁移中 gene-aware ordinal modelling 更有优势，RiceGeneFormer 可以作为可复用 scaffold。当前稿件的贡献则是一个可复现 benchmark 和 diagnostic framework，清楚界定 biological structure 与 marker-level shortcut 之间的当前边界。

## Data and code availability / 数据和代码可用性

Raw genotype、phenotype 和 accession metadata 来自公开 3K Rice Genome / SNP-Seek resources。主 genotype 输入为 3K core SNP PLINK resource，phenotype labels 来自 3KRG morpho-agronomic phenotype spreadsheet。Rice gene coordinates 来自 Ensembl Plants release 61 IRGSP-1.0 assembly，STRING graph inputs 来自 STRING v12.0 Oryza sativa Japonica Group。支持 manuscript figures、final performance tables、cross-region benchmark 和 gene-interpretation audit 的代码与 lightweight source data 将在投稿前通过 versioned public repository 发布。Large raw genotype/phenotype files、large processed matrices、GWAS p-value arrays、checkpoints 和 job logs 默认不纳入 lightweight manuscript repository，因为它们可由 public inputs 和 released code 再生成，且 redistribution policy 仍需最终 license review。DOI-backed release tag 仍需在投稿前生成。

## 图注

Figure 1. 3K 水稻有序 genotype-to-phenotype prediction 的防泄漏 benchmark 构建流程。该流程对齐 3,000 个 accessions、365,710 个 SNPs 和 35 个 descriptor-code phenotype traits，并选择十个核心有序 traits 建模。GWAS p-value priors 和 top-SNP feature sets 均在 active training fold 内生成。Trait imbalance 和 artifact count 说明为什么需要同时报告 macro-F1、accuracy 和 ordinal MAE。

Figure 2. RiceGeneFormer-OMTL architecture 和设计选择。SNP dosages 通过 SNP-to-gene bags 与 train-fold GWAS priors 聚合成 gene tokens。Gene tokens 经过 lightweight graph encoder，由 trait-query attention 解码，并通过 ordinal heads 预测。Gated top-SNP branch 注入直接 marker signal。Pilot evidence 显示 gated fusion 和 mild class balance 最有用，而 graph identity 没有稳定赢家。

Figure 3. 最终 test performance 和 hard-trait behaviour。带弱 distillation 的 RiceGeneFormer 接近 LightGBM macro-F1，但仍低于 balanced SNP-MLP baseline。Accuracy 和 MAE 展示 stronger class balancing 的 trade-off。Per-trait heatmap 和 imbalance scatter 说明强不均衡 traits 是 macro-F1 损失的主要来源。

Figure 4. Ablation boundary、cross-region check 和 gene-attention diagnostic。Expected-score distillation 弱且不单调。Graph identity 和 SNP-to-gene mapping windows 的影响较小。Held-out region benchmark 不支持 RiceGeneFormer 超过 balanced SNP-MLP。GWAS-top2048 replacement graph 纠正 prefix-gene visibility 问题，但降低性能，且不支持 positive known-gene recovery。

## 投稿前需格式化的参考文献锚点

1. The 3,000 rice genomes project. GigaScience 3, 7 (2014). DOI: 10.1186/2047-217X-3-7.
2. Li J-Y, Wang J, Zeigler RS. The 3,000 rice genomes project: new opportunities and challenges for future rice research. GigaScience 3, 8 (2014). DOI: 10.1186/2047-217X-3-8.
3. Wang W et al. Genomic variation in 3,010 diverse accessions of Asian cultivated rice. Nature 557, 43-49 (2018). DOI: 10.1038/s41586-018-0063-9.
4. Mansueto L et al. Rice SNP-seek database update: new SNPs, indels, and queries. Nucleic Acids Research 45, D1075-D1081 (2017).
5. Alexandrov N et al. SNP-Seek database of SNPs derived from 3000 rice genomes. Nucleic Acids Research 43, D1023-D1027 (2015).
6. Szklarczyk D et al. The STRING database in 2023: protein-protein association networks and functional enrichment analyses for any sequenced genome of interest. Nucleic Acids Research 51, D638-D646 (2023). DOI: 10.1093/nar/gkac1000.
