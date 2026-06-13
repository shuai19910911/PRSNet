# PRSNet / CropPRSNet

项目目标：将 PRSNet-2 的 SNP→Gene→Graph→Phenotype 层级图神经网络迁移到植物基因组预测，第一阶段使用 3K Rice Genome / SNP-Seek 数据构建水稻 genotype-to-phenotype 正式训练体系。

当前定位：不是测试项目；按可发表的正式研究路线设计，优先完成可复现、可解释、可扩展的水稻 CropPRSNet 模型，然后扩展到玉米 G2F 多环境预测。

## 仓库内容

- `docs/PROJECT_PLAN_AND_PROGRESS.md`：总研究计划 + 阶段进展，后续每个小阶段只更新这个文件。
- `docs/MODEL_ARCHITECTURE.md`：PRSNet-2 原模型解析与 CropPRSNet 改造架构。
- `docs/TECHNICAL_ROUTE.md`：数据预处理、输入格式、训练、评估、SLURM 资源计划和执行路线。

## 当前数据状态

截至 2026-06-13 15:34:30 CST：

- 已下载 3K Rice phenotype：`3kRG_PhenotypeData_v20170411.xlsx`。
- 已下载 404k core v0.7 PLINK BED/BIM/FAM 并验证可读，但 `.fam` 为 B001/B002 编号，不作为第一训练主数据。
- 已下载 `3K_coreSNP-v2.1.plink.tar.gz`，包含 365,710 SNP × 3,000 samples，样本 ID 为 IRIS ID。
- v2.1 PED/MAP 与 phenotype 成功对接：2,266 / 2,266 phenotype rows。

## 原则

1. GitHub 只保存计划、进展、项目介绍、模型结构、技术路线；不上传数据、结果、日志、模型权重。
2. 每一小阶段进展更新在 `docs/PROJECT_PLAN_AND_PROGRESS.md`，每条进展必须带具体时间。
3. 数据处理和 CPU 作业在 SLURM 登录节点提交到 q07/q08，资源不足时用 cu；单作业最多 30 CPU、150G RAM；每轮最多提交 6 个命令。
4. GPU 正式训练在 gpu10 上执行，使用 `CUDA_VISIBLE_DEVICES=` 指定 GPU，不使用 `--gpu` 参数。
5. 使用 mamba 环境：`PRSNet`。
