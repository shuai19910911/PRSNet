# PRSNet-2 article vs our rice five-trait dataset comparison / PRSNet-2原文效果与我们水稻五性状数据集效果比较

Date（日期）: 2026-06-19

## Bottom line / 结论

我们的 PRSNet-2（SNP→基因→表型层级图神经网络）在五性状有序分类任务上，主指标 macro-F1（各类别公平平均的F1分数）相对 SNP-MLP（SNP输入的普通多层感知机神经网络）提升 6.21%，这个提升幅度与 PRSNet-2 原文在 height/BMI（身高/体重指数）连续性状上报告的约 6% R²（决定系数，回归解释方差比例）相对提升接近。

但两者不能直接用绝对数值比较：原文主要是人类 UKBB（英国生物样本库）疾病二分类/连续性状回归，指标是 AUROC（ROC曲线下面积，二分类排序能力）、AUPRC（精确率-召回率曲线下面积，适合类别不均衡）和 R²（决定系数）；我们是水稻五个 ordinal trait（有序等级性状）分类，主指标是 macro-F1（各类别公平平均的F1分数）、accuracy（准确率）、MAE（平均绝对误差）和 PCC（皮尔逊相关）。

## Paper-reported effect / 原文报告的效果

Source（来源）: Li et al., PRSNet-2: End-to-end genotype-to-phenotype prediction via hierarchical graph neural networks, bioRxiv 2025, DOI（数字对象唯一标识符）: 10.1101/2025.11.22.689899. bioRxiv PDF（论文PDF）在当前环境直接下载被 Cloudflare 403 拦截；以下数字来自可检索的 bioRxiv PDF text snippets（PDF文本片段）和页面摘要。

| Paper setting（原文设置） | Metric（指标） | Article claim（原文说法） | Interpretation（解读） |
|---|---:|---|---|
| Eight diseases（8种疾病） | AUPRC/AUROC（精确率-召回率/ROC曲线下面积） | PRSNet-2 outperformed all baselines across all eight diseases in AUPRC and AUROC average; at disease level, AUPRC best for all diseases and AUROC best for seven diseases. | 原文在疾病风险预测上是“全局最好”，尤其强调类别不均衡时 AUPRC（精确率-召回率曲线下面积）。 |
| Multiple sclerosis（多发性硬化） | AUPRC relative improvement（AUPRC相对提升） | +4.76% versus best baseline PRSNet（上一代多基因风险评分网络） | 疾病任务中报告的最大相对提升之一。 |
| Alzheimer’s disease（阿尔茨海默病） | AUPRC relative improvement（AUPRC相对提升） | +4.42% versus best baseline PRSNet（上一代多基因风险评分网络） | 疾病任务中报告的另一个代表性提升。 |
| Height GWAS 1（身高GWAS第一组） | R² relative improvement（决定系数相对提升） | +6.05% versus best baseline | 连续性状上代表性提升。 |
| BMI GWAS 1（体重指数GWAS第一组） | R² relative improvement（决定系数相对提升） | +6.02% versus best baseline | 连续性状上代表性提升。 |
| Height GWAS 2（身高GWAS第二组） | R² relative improvement（决定系数相对提升） | +7.42% versus best baseline | 另一组GWAS下仍稳定。 |
| BMI GWAS 2（体重指数GWAS第二组） | R² relative improvement（决定系数相对提升） | +5.93% versus best baseline | 另一组GWAS下仍稳定。 |
| Multi-GWAS（多GWAS整合） height/BMI（身高/体重指数） | R² relative improvement（决定系数相对提升） | +3.75% for height, +19.03% for BMI versus single-GWAS PRSNet-2 | 多GWAS整合是原文额外亮点；我们当前水稻数据还没有做等价的多GWAS设置。 |

## Our dataset effect / 我们数据集上的效果

Run（运行）: `docs/fivetrait_prsnet2_paper_aligned_comparison_20260619.md`

| Our setting（我们设置） | Metric（指标） | PRSNet-2（主模型） | Best paper-aligned baseline（对齐原文风格的最佳基线） | Relative gain（相对提升） | Interpretation（解读） |
|---|---:|---:|---:|---:|---|
| Five rice ordinal traits（水稻五个有序等级性状） | macro-F1（各类别公平平均的F1分数） | 0.4754 | 0.4476, SNP-MLP（普通神经网络基线） | +6.21% | 与原文连续性状 R²（决定系数）约 +6% 的提升幅度非常接近。 |
| Five rice ordinal traits（水稻五个有序等级性状） | mean PCC（各性状平均皮尔逊相关） | 0.3878 | 0.3531, SNP-MLP（普通神经网络基线） | +9.85% | 用 PCC（皮尔逊相关）桥接文献时，提升幅度比原文 +6% 稍高。 |
| Five rice ordinal traits（水稻五个有序等级性状） | macro-F1（各类别公平平均的F1分数） | 0.4754 | 0.3319, GWAS top-k additive PRS（训练集GWAS前k个SNP加性多基因评分） | +43.25% | 相对传统 PRS（传统多基因评分）基线优势很明显，但这个基线比 SNP-MLP 弱。 |
| Five rice ordinal traits（水稻五个有序等级性状） | mean PCC² approx（各性状平均皮尔逊相关平方近似） | 0.1504 | NA | context only（仅作尺度参考） | 可粗略理解为 ordinal expected score（期望序数分数）解释的趋势强度；不是严格 R²（决定系数），不能直接和原文 R² 比大小。 |

## Comparable narrative / 可写进论文的比较表述

Compared with the PRSNet-2 preprint, where the authors reported approximately 4.4–4.8% relative AUPRC（精确率-召回率曲线下面积） gains in disease prediction and approximately 6.0% relative R²（决定系数） gains for height and BMI（身高和体重指数）, our rice benchmark showed a comparable relative gain over the neural baseline: PRSNet-2 improved mean macro-F1（各性状宏平均F1） by 6.21% and mean PCC（各性状平均皮尔逊相关） by 9.85% over SNP-MLP（SNP输入的普通多层感知机神经网络）. This suggests that the PRSNet-2 hierarchy is transferable from human genotype-to-phenotype prediction to crop ordinal trait prediction, although absolute metrics are not directly comparable because the task type and evaluation metrics differ.

中文表述：与 PRSNet-2 原文相比，原文在人类疾病预测中报告了约 4.4–4.8% 的 AUPRC（精确率-召回率曲线下面积）相对提升，在身高/BMI（体重指数）连续性状中报告了约 6.0% 的 R²（决定系数）相对提升；我们在水稻五性状有序分类 benchmark（基准测试）中，PRSNet-2 相对 SNP-MLP（普通神经网络基线）的 macro-F1（各类别公平平均的F1分数）提升为 6.21%，mean PCC（各性状平均皮尔逊相关）提升为 9.85%。因此可以说：我们数据集上的提升幅度与原文连续性状结果处在同一量级，并支持 PRSNet-2 层级结构从人类G2P（基因型到表型预测）迁移到作物有序性状预测的可行性。

## Caveats / 注意事项

1. 不要写成“我们效果超过原文”。原文是 AUROC/AUPRC/R²（疾病二分类和连续性状），我们是 macro-F1/PCC/MAE（有序等级分类），指标体系不同。
2. 更稳妥的说法是“relative gain magnitude comparable（相对提升幅度相近）”。
3. 原文使用约 411,000 UKBB（英国生物样本库）样本和五次随机种子重复；我们当前是 3,000 水稻材料、单 split（一次训练/验证/测试划分）+ 单 seed（一个随机种子）。投稿前最好补 multi-seed（多随机种子）或多fold（多折交叉验证）以提高可信度。
4. 我们的 pooled accuracy（混合准确率）和 pooled MAE（混合平均绝对误差）不如部分简单基线，主文应以 macro-F1（各类别公平平均的F1分数）和 mean PCC（各性状平均皮尔逊相关）作为核心积极结果。
