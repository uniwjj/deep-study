---
title: MaxCompute Data+AI
description: 阿里云 MaxCompute AI 化演进——存储层 BLOB+Object Table 多模态统一管理，计算层 SQL AI Function + MaxFrame(Pandas兼容分布式框架)，PB 级数据预处理性能提升 50%+
aliases: [MaxCompute AI, 大数据AI化]
tags: [big-data, ai-ml, platform, practice]
sources: [2026/04/27/MaxCompute Data+AI.md]
created: 2026-05-09
updated: 2026-05-09
---

# MaxCompute Data+AI

## 四层架构

```
数据层(多模态) → 模型层(千问/DeepSeek) → 计算层(CPU+GPU混合) → 引擎层(SQL+Python)
```

## 存储：多模态统一管理

- BLOB 类型存音频/视频/图片
- Object Table 打通 OSS/Hologres 外部存储
- Max Meta 统一元数据服务，无需移动数据
- 单表多模态一行多列混存（BLOB + JSON）

## 计算引擎

### SQL AI Function
SQL 直接调用大模型离线推理，分析师零门槛。

### MaxFrame
兼容 Pandas 的分布式 Python 框架：
- CPU/GPU 混合调度
- 分布式算子兼容 Pandas/XGBoost/LightGBM

## 实战数据

- 头部大模型公司 PB 级数据预处理，MinHash 性能 **+50%**
- 弹性资源达 16 万核
- 自动驾驶场景分布式效率 **+40%**

## 相关页面

- [[data-dev-skills-automation]] — 数仓开发 Skills
- spark-ai-integration — Spark AI 集成
- [[maxcompute-skills]] — MaxCompute MCMCP 与 Skills 体系
