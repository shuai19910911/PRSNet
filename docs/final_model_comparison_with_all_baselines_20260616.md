# Final model comparison with all baselines (random_seed42 test)

All metrics are computed on the same random_seed42 test split when available. Accuracy and MAE are pooled over observed test labels; macro-F1 is the mean across the 10 core ordinal traits.

| Rank | Category | Model | test macro-F1 | Accuracy | MAE | Observed labels |
|---:|---|---|---:|---:|---:|---:|
| 1 | Final / MoE（最终/专家混合） | RiceGATE-MoE final | 0.4235091944 | 0.6268417681 | 0.5829596413 | 3122 |
| 2 | Tree ensemble（树模型集成） | alpha3 ExtraTrees average base | 0.4181925505 | 0.6133888533 | 0.6002562460 | 3122 |
| 3 | Tree ensemble（树模型集成） | alpha3 ExtraTrees per-trait tuned | 0.4170108081 | 0.6137091608 | 0.5986547085 | 3122 |
| 4 | Tree ensemble（树模型集成） | alpha3 ExtraTrees global tuned | 0.4146637828 | 0.6079436259 | 0.6095451634 | 3122 |
| 5 | Tree baseline（树模型基线） | best single ExtraTrees top288 alpha1.0 | 0.4128983270 | 0.6060217809 | 0.6063420884 | 3122 |
| 6 | Tree ensemble（树模型集成） | ExtraTrees top288 5-seed ensemble | 0.4091922841 | 0.6079436259 | 0.6063420884 | 3122 |
| 7 | Tree baseline（树模型基线） | ExtraTrees top288 p-value order | 0.4081261641 | 0.6050608584 | 0.6076233184 | 3122 |
| 8 | Tree ensemble（树模型集成） | ExtraTrees top256 3-seed ensemble | 0.4066404522 | 0.5983344010 | 0.6143497758 | 3122 |
| 9 | Tree ensemble（树模型集成） | ExtraTrees top256/top288 10-model ensemble | 0.4057924795 | 0.6008968610 | 0.6121076233 | 3122 |
| 10 | Neural baseline（神经网络基线） | SNP-MLP seed ensemble | 0.3950466669 | 0.6165919283 | 0.6031390135 | 3122 |
| 11 | Tree baseline（树模型基线） | RandomForest top288 alpha1.0 | 0.3836440333 | 0.6185137732 | 0.6002562460 | 3122 |
| 12 | Neural baseline（神经网络基线） | best single SNP-MLP seed44 | 0.3801751159 | 0.5989750160 | 0.6111467008 | 3122 |
| 13 | Neural baseline（神经网络基线） | SNP-MLP seed43 | 0.3794283894 | 0.6162716208 | 0.5938500961 | 3122 |
| 14 | Tree baseline（树模型基线） | top2% ExtraTrees | 0.3793846085 | 0.6428571429 | 0.5313901345 | 3122 |
| 15 | Neural baseline（神经网络基线） | SNP-MLP seed44 macro-selected | 0.3772106227 | 0.5973734785 | 0.6153106983 | 3122 |
| 16 | Neural baseline（神经网络基线） | SNP-MLP seed42 | 0.3747689119 | 0.6063420884 | 0.6294042281 | 3122 |
| 17 | Tree baseline（树模型基线） | top0.5% ExtraTrees | 0.3744182004 | 0.6348494555 | 0.5679051890 | 3122 |
| 18 | Tree baseline（树模型基线） | top10% ExtraTrees | 0.3740561209 | 0.6386931454 | 0.5339525945 | 3122 |
| 19 | Tree baseline（树模型基线） | top1% ExtraTrees | 0.3730547175 | 0.6351697630 | 0.5522101217 | 3122 |
| 20 | Tree baseline（树模型基线） | top5% ExtraTrees | 0.3673612670 | 0.6303651505 | 0.5582959641 | 3122 |
| 21 | RiceGeneFormer baseline（基因图模型基线） | RiceGeneFormer p<0.05 cap20000 class-head | 0.3384759004 | 0.5807174888 | 0.6672005125 | 3122 |
| 22 | Boosting baseline（梯度提升树基线） | LightGBM top512 | 0.3379516154 | 0.6181934657 | 0.5845611787 | 3122 |
| 23 | RiceGeneFormer baseline（基因图模型基线） | RiceGeneFormer top2% merged SNP | 0.3233000267 | 0.5794362588 | 0.6806534273 | 3122 |
| 24 | RiceGeneFormer baseline（基因图模型基线） | RiceGeneFormer p<0.05 cap20000 expected-score | 0.3167930981 | 0.5054452274 | 0.6969891095 | 3122 |
| 25 | Boosting baseline（梯度提升树基线） | XGBoost top512 | 0.3161007836 | 0.6255605381 | 0.5717488789 | 3122 |
| 26 | RiceGeneFormer baseline（基因图模型基线） | RiceGeneFormer GWAS top2048 early pilot | 0.2989983753 | 0.5836002562 | 0.5877642537 | 3122 |
| 27 | Naive baseline（朴素基线） | train-median class baseline | 0.2103171176 | 0.5887251762 | 0.6140294683 | 3122 |
| 28 | Naive baseline（朴素基线） | train-majority class baseline | 0.2098019375 | 0.5848814862 | 0.7049967969 | 3122 |

Notes: train-majority/train-median baselines are computed from training labels only. LightGBM/XGBoost accuracy and MAE are recomputed as label-pooled values from per-trait metric TSV files. RiceGATE-MoE gates were selected on validation only; test labels were evaluated after gates froze.
