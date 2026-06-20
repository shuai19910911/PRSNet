# RiceGraphOrdinalNet ten-trait per-trait metrics / 十性状逐性状指标

Run（运行）: `data/3krice/processed/rice_graph_ordinal_feature_net_tentrait_top2048_h64_gpu5_seed42_20260620`

Device（设备）: 12.12.12.210 / gpu10 / CUDA_VISIBLE_DEVICES=5 / NVIDIA A100-SXM4-40GB

说明 / Note: PCC（皮尔逊相关）基于 expected ordinal score（期望序数分数）。单个性状没有跨性状混合，因此 `pooled PCC single trait（单性状混合PCC）` 与该性状 PCC 相同；真正的 overall pooled PCC（十性状混合标签PCC）只在最后一行有意义。

| 性状 / Trait | n（测试有效标签数） | 类别数 | macro-F1（宏平均F1） | accuracy（准确率） | MAE（平均绝对误差） | mean PCC（单性状PCC） | pooled PCC（单性状=同左；整体行为混合PCC） |
|---|---:|---:|---:|---:|---:|---:|---:|
| CUDI_CODE_REPRO | 315 | 2 | 0.6676 | 0.7111 | 0.2889 | 0.3777 | 0.3777 |
| CULT_CODE_REPRO | 312 | 7 | 0.2964 | 0.3269 | 0.9968 | 0.6329 | 0.6329 |
| CUNO_CODE_REPRO | 313 | 3 | 0.3878 | 0.5112 | 0.4952 | 0.2957 | 0.2957 |
| LLT_CODE | 313 | 5 | 0.2951 | 0.4952 | 0.5495 | 0.5113 | 0.5113 |
| PLT_CODE_POST | 311 | 4 | 0.4184 | 0.7781 | 0.2251 | 0.4623 | 0.4623 |
| SDHT_CODE | 313 | 3 | 0.3522 | 0.7125 | 0.2875 | 0.2224 | 0.2224 |
| PTH | 339 | 3 | 0.4824 | 0.4897 | 0.5870 | 0.3398 | 0.3398 |
| SPKF | 340 | 5 | 0.1827 | 0.4559 | 0.6735 | 0.1042 | 0.1042 |
| CUST_REPRO | 340 | 8 | 0.0522 | 0.1029 | 2.1088 | 0.3204 | 0.3204 |
| PSH | 226 | 3 | 0.3289 | 0.7035 | 0.3584 | 0.0389 | 0.0389 |
| ALL_10_TRAITS | 3122 |  | 0.3464 | 0.5192 | 0.6775 | 0.3306 | 0.7263 |

## Training summary / 训练摘要

- best epoch（最佳轮次）: 10
- epochs completed（完成轮次）: 20
- elapsed seconds（耗时秒）: 49.965
- test macro-F1（测试宏平均F1）: 0.3464
- test accuracy（测试准确率）: 0.5192
- test MAE（测试平均绝对误差）: 0.6775
- test mean PCC（测试各性状平均皮尔逊相关）: 0.3306
- test pooled PCC（测试混合标签皮尔逊相关，辅助）: 0.7263
