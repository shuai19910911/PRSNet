# Huihui Li（李慧慧）智能育种/基因组预测方向论文学习笔记

更新时间：2026-06-20

## 0. 完成状态

已完成一版“可用于后续模型设计和文章写作”的学习笔记。范围限定为 Huihui Li（李慧慧）在 smart breeding（智能育种，用数据和算法加速育种）、genomic prediction/GP（基因组预测，用基因型预测表型或育种值）、genomic selection/GS（基因组选择，用预测结果提前筛选材料）、deep learning/DL（深度学习，多层神经网络模型）、multi-modal learning（多模态学习，融合基因型/环境/表型图像/多组学等多种数据）、genotype-by-environment/G×E（基因型与环境互作，同一基因型在不同环境表现不同）方向的直接相关论文和平台工作；没有把其早期 QTL mapping（数量性状位点定位，用遗传标记找影响性状的区域）、association mapping（关联分析，用群体自然变异找性状相关位点）等不直接服务当前模型设计的论文逐篇展开。

## 1. 作者与方向定位

Huihui Li（李慧慧）是中国农业科学院作物科学研究所/Center of Crop Genes and Molecular Design（作物基因与分子设计中心）研究员。官方主页给出的研究兴趣包括 gene mining algorithms（基因挖掘算法，从基因组数据中寻找功能基因）、intelligent design breeding（智能设计育种，用模型指导亲本/材料选择）、statistical genomics（统计基因组学）、QTL mapping（数量性状位点定位）和 GWAS（全基因组关联分析，用全基因组标记找性状相关位点）。

对我们当前 PRSNet/RiceGraphOrdinalNet（水稻多等级性状预测模型）最相关的是：

1. 把 breeding problem（育种问题）从单纯“预测一个性状”提升到 genetic gain（遗传增益，单位时间育种进步）、decision support（决策支持，帮助育种家选择材料）和 platform（平台化工具）的层面。
2. 把输入从 SNP（单核苷酸多态性，基因组标记）扩展到 multi-omics（多组学，如基因组/转录组/蛋白组/代谢组）、phenomics（表型组，高通量表型数据）、enviromics（环境组，气象/土壤/管理数据）和 G×E（基因型与环境互作）。
3. 模型上强调 modular（模块化，不同数据源先单独建模再融合）、attention（注意力机制，让模型自动聚焦关键信息）、AutoML（自动机器学习，自动选择模型/参数/特征）、explainability（可解释性，输出重要特征/互作网络），而不是只堆一个黑盒深度网络。
4. 文章上常用“挑战—技术框架—数据整合—模型验证—育种应用”的结构，突出 practical breeding utility（育种实用价值），不是单纯刷分。

## 2. 用户给定 Frontiers 文章：它在这条路线中的位置

文章：Montesinos-López A. et al. 2024. Deep learning methods improve genomic prediction of wheat breeding. Frontiers in Plant Science（植物科学前沿期刊）15:1324090. DOI（数字对象唯一标识符）: 10.3389/fpls.2024.1324090。

Huihui Li（李慧慧）是该文作者之一，单位为 State Key Laboratory of Crop Gene Resources and Breeding, Institute of Crop Sciences and CIMMYT China Office, CAAS（中国农科院作科所和 CIMMYT 中国办公室）。通讯作者是 Osval A. Montesinos-López 和 Jose Crossa；因此这篇更像是 Huihui Li 参与的国际合作论文，不是其团队第一/通讯主导论文，但它与其 DNNGP（深度神经网络基因组预测）和 multi-modal DL（多模态深度学习）路线高度一致。

### 2.1 文章核心问题

问题不是“深度学习一定比 GBLUP（genomic best linear unbiased prediction，基因组最佳线性无偏预测，经典线性混合模型）强”，而是：在 moderate-sized dataset（中等规模数据集）下，multi-modal DL（多模态深度学习）是否能在 wheat breeding（小麦育种）GP（基因组预测）中达到或超过 GBLUP。

### 2.2 数据与任务

- 4,464 wheat lines（小麦材料）。
- 18,239 SNP markers（SNP 标记）。
- 5个性状：Yield（产量）、Germination（发芽）、Heading（抽穗）、Height（株高）、Maturity（成熟期）。
- 4个环境：B5IR（5次灌溉）、B2IR（2次灌溉）、BDRT（滴灌干旱胁迫）、BLHT（晚播热胁迫）。
- 验证方式包括 5-fold cross-validation（五折交叉验证，把样本分5份轮流验证）和 LOEO（leave-one-environment-out，留一环境外推，整环境作为测试）。

### 2.3 模型结构

Frontiers 2024 的模型延续 Montesinos-López 团队此前 multi-modal DL（多模态深度学习）结构：

1. 每个 modality（模态，输入数据类型）先有一个独立 neural network（神经网络）分支。
2. 各分支输出 concat（拼接）成联合表示。
3. 2024 Frontiers 文章的关键增量是：不是直接合并最终输出，而是在 concat（拼接）后再接一个 MLP（多层感知机，普通全连接神经网络），让不同模态之间有一次额外交互学习。
4. 与 GBLUP（经典线性混合模型）比较，DL（深度学习）在五折验证中对2/5性状更优，其他性状相近；在 LOEO（留一环境外推）中有竞争力但不稳定压倒 GBLUP。

### 2.4 对我们模型的直接启发

1. 我们不能只写“深度学习比传统模型强”，应写“深度学习在部分性状/部分验证设定下有优势，但 GBLUP/树模型仍是强基线”。这有利于 BIB（Briefings in Bioinformatics，生物信息学综述/方法期刊）审稿可信度。
2. 对当前 RiceGraphOrdinalNet（我们的水稻图序数模型）最有用的是“modality-specific encoder（每种数据源单独编码）+ fusion MLP/attention（融合层/注意力层）”这个思路。我们现在只有 SNP→gene（SNP到基因聚合）和 PRS（polygenic risk score，多基因评分）分支，后续可以扩展 environment token（环境 token，把环境当成模型输入单元）、trait token（性状 token，把性状当成查询向量）、gene prior token（基因先验 token，把GWAS和注释信号作为输入）。
3. 文章用了 LOEO（留一环境外推）证明泛化；我们水稻如果没有环境，可类比做 LOA（leave-one-accession-like group，按材料群体外推）、LOT（leave-one-trait，留一性状）或 subpopulation holdout（亚群体外推）。

## 3. Huihui Li 方向代表论文与“可学习点”

| 年份 | 论文/平台 | 论文类型 | 核心思路 | 模型/框架 | 对我们可学的点 |
|---|---|---|---|---|---|
| 2018 | Fast-Forwarding Genetic Gain. Trends in Plant Science（植物科学趋势） | Spotlight（短观点） | genetic gain（遗传增益）由 selection accuracy（选择准确度）、selection intensity（选择强度）、additive variance（加性遗传方差）和 breeding cycle（育种周期）共同决定；用 speed breeding（加速世代育种）+ GS（基因组选择）+ gene bank（种质库）+ genome editing（基因编辑）加速遗传进步 | 不是具体模型，是育种系统路线图 | 我们论文引言可把模型贡献放到“提高选择准确度、缩短育种周期”的遗传增益框架里 |
| 2022 | Smart breeding driven by big data, AI, and iGEP（integrated genomic-enviromic prediction，基因组-环境组整合预测）. Molecular Plant（分子植物） | Perspective（观点/框架文） | P=G+E+GEI（表型=基因型+环境+基因型环境互作）；未来预测应整合 G-P-E（基因型-表型-环境）三维数据 | iGEP（整合基因组-环境组预测） | 我们文章讨论部分要强调当前只用基因型/性状，下一步可接入环境组/管理变量，形成 G×E（基因型环境互作）扩展 |
| 2023 | DNNGP, a deep neural network-based method for genomic prediction using multi-omics data in plants. Molecular Plant（分子植物） | Resource article（资源/方法文章） | 传统线性模型难捕捉非线性和多组学关系；DNNGP（深度神经网络基因组预测）可处理 SNP、SV（结构变异）、InDel（插入缺失变异）、gene expression（基因表达）等 | 3个 CNN（卷积神经网络，用局部窗口提特征）层 + BN（batch normalization，批归一化，稳定训练）+ dropout（随机失活，防过拟合）+ early stopping（早停，验证集不再提升即停止） | 我们要学习其“多数据集、多作物、多基线、多消融”的结构；模型上可以把 SNP→gene 局部聚合类比为 CNN 的局部特征抽取，但更有生物解释性 |
| 2023 | Multimodal deep learning methods enhance genomic prediction of wheat breeding. G3（遗传学期刊） | 方法比较 | 把 genomic（基因组）、year/environment（年份/环境）和 image/NDVI（图像/植被指数）作为多模态输入，用 DL（深度学习）建模跨年/跨环境预测 | 每个 modality（模态）一个分支，拼接后融合；含 ResNet（残差网络，缓解梯度消失）和 Bayesian optimization（贝叶斯优化，自动调参） | 我们后续如果加入环境/遥感/组织表达，应该采用“分支编码+融合”的模块化结构，而不是把所有特征粗暴拼一起 |
| 2024 | Deep learning methods improve genomic prediction of wheat breeding. Frontiers in Plant Science（用户给的文章） | 方法比较 | 中等规模数据下验证 DL（深度学习）是否比 GBLUP（经典基因组预测模型）更好 | 多模态 DL（多模态深度学习）+ concat（拼接）后额外 MLP（全连接融合网络） | 文章写作上应诚实展示 DL 对部分性状优势、部分性状持平；模型上可借鉴“额外融合层” |
| 2024 | Smart Breeding Platform. Molecular Plant（分子植物） | Platform correspondence（平台通信短文） | 把 germplasm（种质）、field test（田间试验）、phenotype（表型）、genotype（基因型）、GWAS（全基因组关联分析）、GS（基因组选择）放进统一网页平台 | 4大模块：germplasm data management（种质数据管理）、test management（试验管理）、genomic data management（基因组数据管理）、data analysis（数据分析） | 我们论文可把模型定位为“可嵌入智能育种平台的预测模块”，不仅是脚本 |
| 2024 | Artificial intelligence in plant breeding. Trends in Genetics（遗传学趋势） | Review（综述） | AI（人工智能）贯穿数据采集、种质库挖掘、基因型-表型桥接、基因编辑和精准育种 | 综述 AI breeding pipeline（人工智能育种流程） | 我们引言/讨论可借鉴“data deluge（数据洪流）→ AI-enabled breeding（AI赋能育种）→ genotype-to-phenotype gap（基因型到表型鸿沟）”这条叙事 |
| 2025 | AutoGS / Leveraging automated machine learning for environmental data-driven genetic analysis and genomic prediction in maize hybrids. Advanced Science（先进科学） | 方法+软件 | 环境维度数据驱动 GP（基因组预测），用 dimension-reduced environmental parameters（降维环境因子）解释性状-环境关系和 G×E（基因型环境互作） | AutoML（自动机器学习）集成多种回归器，如 ExtraTrees（极端随机树）、LightGBM（轻量梯度提升树）、XGBoost（梯度提升树）等；含 SHAP（解释特征贡献的方法） | 我们可学“强传统机器学习+解释性+自动调参”路线；当前水稻 ordinal（有序等级）任务不要只依赖深度模型，树模型和解释性要保留 |
| 2025 | Accurate genomic prediction for grain yield and grain moisture content of maize hybrids using multi-environment data. JIPB（植物学报英文版） | 应用研究 | 2,126个玉米杂交种、34环境、19气候因子；把 GE（基因型环境互作）加入 GBLUP | GBLUP-GE 19CF（含19个气候因子的基因型环境互作模型）、GBLUP-GE 9CF（筛9个气候因子）、GBLUP-GE PCA（主成分降维环境因子） | 即使不用深度学习，严谨 G×E（基因型环境互作）建模也能发好文章；我们的Rice模型如果加入环境，先做 GBLUP-GE/树模型强基线 |
| 2025 | Fast-forwarding plant breeding with deep learning-based genomic prediction. JIPB（植物学报英文版） | Mini review（小综述） | DL-based GP（深度学习基因组预测）有潜力，但受限于大规模高质量数据、benchmark（基准评测）不一致、环境因素整合不足 | 提出 modular approaches（模块化方法）、data augmentation（数据增强）、advanced attention mechanisms（高级注意力机制） | 这正好支持我们设计“模块化、注意力、严格benchmark”的 BIB 论文定位 |
| 2025 | EXGEP: a framework for predicting genotype-by-environment interactions using ensembles of explainable machine-learning models. Briefings in Bioinformatics（生物信息学简报） | 方法+软件 | 用 ensemble（集成，把多个模型组合）和 explainable ML（可解释机器学习）建模 G×E（基因型环境互作），更适合先验不足场景 | EXGEP（可解释集成G×E预测框架），自动整合 genotype/weather/soil（基因型/气象/土壤）并解释关键特征互作 | 对我们目标期刊 BIB 很关键：BIB 认可“框架+可解释+软件+多基线+生物问题”的组合，不一定必须最复杂深度网络 |

## 4. Huihui Li 论文的共同“文章架构”

这些文章的写法有明显模板：

### 4.1 引言结构

1. 全球需求：climate change（气候变化）、food security（粮食安全）、population growth（人口增长）。
2. 育种瓶颈：field trial cost（田间试验成本高）、long breeding cycle（育种周期长）、G×E（基因型环境互作）导致泛化难。
3. 方法缺口：传统 GP（基因组预测）模型强但偏线性，DL（深度学习）有潜力但数据量/基准/环境整合不足。
4. 本文贡献：提出可复用框架、软件或系统，不只是一个模型。

### 4.2 方法结构

1. 数据：清楚列出作物、材料数量、环境数量、性状、标记数量、表型/环境/组学来源。
2. 预处理：缺失值、编码、QC（quality control，质量控制）、环境特征降维、训练/验证/测试设置。
3. 模型：传统强基线 + 新模型 + 自动调参/消融。
4. 验证：CV（cross-validation，交叉验证）、LOEO（留一环境外推）、跨年/跨区域/跨环境。
5. 解释：feature importance（特征重要性）、SHAP（解释模型特征贡献）、环境因子贡献、marker（标记）或 gene（基因）解释。
6. 平台/代码：GitHub（代码仓库）或 Web platform（网页平台）。

### 4.3 结果结构

1. 总体预测精度：PCC（皮尔逊相关，预测分数和真实值线性一致性）、MAE（平均绝对误差）、RMSE（均方根误差）、accuracy（准确率）等。
2. 分性状/分环境结果：不只给平均值。
3. 与经典方法比较：GBLUP、BayesB（贝叶斯基因组预测模型）、LightGBM、SVR（支持向量回归）、DeepGS（深度学习基因组选择）等。
4. 消融：去掉某个模态、去掉环境因子、减少环境因子、不同模型/参数/数据量。
5. 实用价值：筛选 top hybrids（优良杂交种）、减少 field trials（田间试验）、提高 genetic gain（遗传增益）。

### 4.4 讨论结构

1. 为什么模型有效：非线性、互作、多源融合、环境因素。
2. 为什么模型不总是赢：数据量不足、噪声、性状遗传结构不同、基线强。
3. 育种可用性：如何嵌入平台或决策流程。
4. 局限与未来：多环境、多种子、多作物、可解释性、标准化 benchmark（基准评测）。

## 5. 模型架构规律：应该学什么，不该照搬什么

### 5.1 应该学

1. 模块化编码：genotype encoder（基因型编码器）、environment encoder（环境编码器）、phenomics encoder（表型图像编码器）、omics encoder（组学编码器）分开，再融合。
2. 融合层要可学习：concat（拼接）后接 MLP（全连接网络）或 attention（注意力机制），不能只做简单平均。
3. 传统模型必须强：GBLUP、BayesB、LightGBM、XGBoost、ExtraTrees（极端随机树）要作为强基线。
4. 解释性是卖点：SHAP、特征重要性、环境因子、基因/通路解释都要服务生物叙事。
5. 评估要跨场景：random CV（随机交叉验证）不够，要加 LOEO（留一环境外推）、跨年、跨区域、亚群体 holdout（保留亚群体做测试）。
6. 软件/平台意识：方法要有可运行脚本、manifest（训练记录文件）、数据格式说明、可复现实验表。

### 5.2 不该照搬

1. 不应直接照搬 DNNGP 的 CNN（卷积神经网络）到水稻多等级分类；DNNGP主要处理连续性状回归，我们是 ordinal classification（有序等级分类）。
2. 不应只追求 DL（深度学习）超过 GBLUP；Huihui Li 路线本身也承认 GBLUP/机器学习很强，关键是“合适任务+合适数据+严谨验证”。
3. 不应把 test（测试集）用于调参；他们强调交叉验证和外推，我们也要保持 train/val/test（训练/验证/测试）边界。
4. 不应只报平均指标；要按 trait（性状）、class（类别）、environment（环境）拆开。

## 6. 对 RiceGraphOrdinalNet / PRSNet 当前项目的具体设计建议

### 6.1 模型命名与定位

建议把我们的模型定位为：

RiceGraphOrdinalNet: a leakage-aware gene-informed ordinal genomic prediction framework for rice breeding.

中文含义：RiceGraphOrdinalNet 是一个 leakage-aware（防数据泄漏）、gene-informed（基因先验引导）、ordinal genomic prediction（有序等级基因组预测）的水稻育种预测框架。

这个定位比“改进版PRSNet”更像 Huihui Li 团队的写法：不是单模型，而是 framework（框架）。

### 6.2 架构可向 Huihui Li 路线靠拢

当前结构可以解释为：

1. SNP-to-gene pooling（SNP到基因池化）：把 SNP 标记聚合到基因层面，类似 DNNGP 的局部特征抽取，但更有生物可解释性。
2. GWAS priors（全基因组关联先验）：只用 train-fold GWAS（训练折GWAS）提供先验，防泄漏。
3. gene graph/message passing（基因图消息传递）：用基因邻接关系或注释图传播信号，但要有 matched random graph（匹配随机图）作为消融。
4. trait token attention（性状token注意力）：每个性状一个查询向量，学习性状特异的基因组合。
5. ordinal head（序数头）：保留等级顺序信息，适合我们的水稻等级性状。
6. categorical auxiliary head（类别辅助头）：辅助 macro-F1（宏平均F1，重视少数类别），但不能让类别头破坏有序结构。

### 6.3 论文主线建议

BIB（Briefings in Bioinformatics，生物信息学简报）论文可按下面结构写：

1. Background（背景）：智能育种需要可靠的 genotype-to-phenotype prediction（基因型到表型预测），但水稻等级性状存在 class imbalance（类别不平衡）、multi-trait heterogeneity（多性状异质性）、small-to-moderate sample size（中小样本）和 leakage risk（数据泄漏风险）。
2. Gap（缺口）：现有 GP（基因组预测）多关注连续性状回归；PRSNet-2 等来自人类复杂性状，不直接适配作物多等级性状；DL（深度学习）在作物中表现不稳定，需要强基线和防泄漏评估。
3. Method（方法）：提出 RiceGraphOrdinalNet，整合 train-fold GWAS priors（训练折关联先验）、SNP-to-gene mapping（SNP到基因映射）、gene graph（基因图）、trait-aware attention（性状感知注意力）和 ordinal prediction（有序等级预测）。
4. Benchmark（基准评测）：比较 majority（多数类基线）、GBLUP/PRS（基因组线性预测/多基因评分）、LightGBM/XGBoost/ExtraTrees（树模型）、MLP（多层感知机）、PRSNet-2-style（PRSNet-2风格模型）和我们模型。
5. Results（结果）：总体指标 + 逐性状表 + 少数类表现 + 校准前后 + 消融。
6. Interpretation（解释）：基因层面/图层面/性状层面解释，不夸大。
7. Discussion（讨论）：与 Huihui Li 路线一致，强调下一步接入 environment/enviromics（环境组）和 platform（平台化）。

## 7. 推荐后续模型迭代方向

按优先级：

1. 先稳住 ordinal calibration（有序等级校准）：我们已经看到 expected score（期望分数）和 PCC（皮尔逊相关）好时，validation-only threshold calibration（只用验证集阈值校准）能提升 macro-F1（宏平均F1）。这与 Frontiers 文章“DL连续预测有潜力但解码要稳”的经验一致。
2. 做 trait-wise expert（性状专家）：每个性状一个小 head（输出头）或 gate（门控），类似多模态分支思想，避免10个性状互相干扰。
3. 做 gene prior ablation（基因先验消融）：无GWAS先验、GWAS先验、随机先验、不同窗口 SNP-to-gene mapping（SNP到基因映射）对照。
4. 做 graph ablation（图消融）：真实基因图、染色体邻近图、匹配随机图、无图；只有真实图稳定胜出，才能讲 biological graph advantage（生物图优势）。
5. 做 distribution-shift split（分布迁移划分）：如果有亚群体/籼粳/地理来源，必须做 holdout（留出外推），学习 Huihui Li 多环境/跨区域思路。
6. 如果后续拿到环境数据，再升级到 RiceGxEOrdinalNet（基因型×环境有序预测网络）：genotype encoder（基因型编码器）+ environment encoder（环境编码器）+ trait token（性状token）+ ordinal head（序数头）。

## 8. 可直接放进论文的参考叙事

可改写成论文英文逻辑：

Recent advances in smart breeding（智能育种）have shifted genomic prediction（基因组预测）from marker-only linear models toward integrative frameworks that combine genomic, phenomic, enviromic, and multi-omics information. Huihui Li and collaborators have proposed iGEP（integrated genomic-enviromic prediction，基因组-环境组整合预测）, DNNGP（deep neural network genomic prediction，深度神经网络基因组预测）, Smart Breeding Platform（智能育种平台）, AutoGS（自动机器学习基因组选择）and EXGEP（可解释基因型环境互作预测框架）, forming a coherent research program centered on data integration, modular modeling, explainability, and practical breeding deployment. However, existing work mainly focuses on continuous-trait regression（连续性状回归）or multi-environment prediction（多环境预测）, whereas ordinal crop traits（有序等级作物性状）remain underexplored. This motivates RiceGraphOrdinalNet, a leakage-aware（防数据泄漏）gene-informed（基因先验引导）framework for multi-trait ordinal genomic prediction（多性状有序等级基因组预测）in rice.

## 9. 参考文献清单（本轮已核对到的核心文献）

1. Li, H., Rasheed, A., Hickey, L. T. & He, Z. Fast-Forwarding Genetic Gain. Trends in Plant Science 23, 184-186 (2018). DOI: 10.1016/j.tplants.2018.01.007.
2. Xu, Y. et al. Smart breeding driven by big data, artificial intelligence, and integrated genomic-enviromic prediction. Molecular Plant 15, 1664-1695 (2022). DOI: 10.1016/j.molp.2022.09.001.
3. Wang, K. et al. DNNGP, a deep neural network-based method for genomic prediction using multi-omics data in plants. Molecular Plant 16, 279-293 (2023). DOI: 10.1016/j.molp.2022.11.004.
4. Montesinos-López, A. et al. Multimodal deep learning methods enhance genomic prediction of wheat breeding. G3: Genes | Genomes | Genetics 13, jkad045 (2023). DOI: 10.1093/g3journal/jkad045.
5. Montesinos-López, A. et al. Deep learning methods improve genomic prediction of wheat breeding. Frontiers in Plant Science 15, 1324090 (2024). DOI: 10.3389/fpls.2024.1324090.
6. Li, H. et al. Smart Breeding Platform: A web-based tool for high-throughput population genetics, phenomics, and genomic selection. Molecular Plant 17, 677-681 (2024). DOI: 10.1016/j.molp.2024.03.002.
7. Farooq, M. A. et al. Artificial intelligence in plant breeding. Trends in Genetics 40, 891-908 (2024). DOI: 10.1016/j.tig.2024.07.001.
8. Montesinos-López, O. A. et al. A review of multimodal deep learning methods for genomic-enabled prediction in plant breeding. Genetics 228, iyae161 (2024). DOI: 10.1093/genetics/iyae161.
9. He, K. et al. Leveraging automated machine learning for environmental data-driven genetic analysis and genomic prediction in maize hybrids. Advanced Science 12, e2412423 (2025). DOI: 10.1002/advs.202412423.
10. Wang, J. et al. Accurate genomic prediction for grain yield and grain moisture content of maize hybrids using multi-environment data. Journal of Integrative Plant Biology 67, 1379-1394 (2025). DOI: 10.1111/jipb.13857.
11. Gao, S. et al. Fast-forwarding plant breeding with deep learning-based genomic prediction. Journal of Integrative Plant Biology 67, 1700-1705 (2025). DOI: 10.1111/jipb.13914.
12. Yu, T. et al. EXGEP: a framework for predicting genotype-by-environment interactions using ensembles of explainable machine-learning models. Briefings in Bioinformatics (2025). DOI: 10.1093/bib/bbaf414.

## 10. 结论：我们应该学到的“一句话”

Huihui Li（李慧慧）这个方向不是单纯追求一个更深的网络，而是把 genomic prediction（基因组预测）包装成 smart breeding（智能育种）可用的 framework（框架）：强基线、防泄漏、多源数据、模块化模型、跨环境验证、可解释输出、平台化部署。我们后续做 RiceGraphOrdinalNet/RiceGSFormer（水稻基因组预测模型）时，应沿着这条路线写：模型贡献只是核心，真正的文章价值是“面向作物多等级性状的严谨基准框架和可解释预测模块”。
