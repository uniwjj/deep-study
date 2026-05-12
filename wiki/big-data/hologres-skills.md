---
title: Hologres Skills
description: Hologres 多模融合一站式 AI 数据检索分析平台——AI Function 一键调用百炼模型、CLI 与 Skills 体系、Mem0 长记忆方案，实现广告素材智能生成与投放分析
aliases: [Hologres Skills, Hologres AI Function]
tags: [big-data, ai-agent, platform]
sources: [2026/05/12/04. 基于 Hologres Skills 完成广告素材智能生成与投放效果分析.pdf, 2026/05/12/DataWorks DataAgent分享录音.md]
created: 2026-05-12
updated: 2026-05-12
---

# Hologres Skills

Hologres 定位为**多模融合的一站式 AI 数据检索分析平台**，核心理念：“一份数据、一份计算、多模分析”。演讲人：骆撷冬（阿里云智能集团产品专家）。

## 多模融合架构

```
统一 SQL 查询入口
├── 在线推荐 / BI 报表 / 检索分析 / RAG / 向量化 / 内容生成
├── 点查 / OLAP 分析 / 全文检索 / 向量检索 / 混合检索 / AI 推理
├── Dynamic Table 增量计算（ODS → DWD → DWS → ADS）
├── AI Function（Rerank / Embedding / chunking / parse_document / ...）
└── 百炼模型（Embedding / Qwen 系列 / DeepSeek / Wan 模型 / Qwen-VL）
```

### 多模数据存储

| 存储层 | 数据类型 |
|--------|---------|
| 内部存储 | 向量、文本、JSON、MaxCompute、Paimon、Iceberg |
| 数据湖 | 图片、PDF、PPT、视频 |
| 外部存储 | OSS（非结构化数据） |

## AI Function：一键调用百炼模型

在 Hologres 控制台通过 API_KEY 一键部署百炼模型，通过标准 SQL 调用：

```sql
-- 示例：AI Function 调用
SELECT ai_gen_image(prompt, style) FROM materials;
```

### 支持的百炼模型

| 类型 | 模型 |
|------|------|
| 文本推理 | Qwen 系列、Qwen3 系列、DeepSeek 系列、kimi |
| 多模态推理 | Qwen3-VL 系列、Qwen-VL 系列 |
| 向量化 | Text-embedding 系列、Tongyi-embedding 系列 |
| 内容生成 | Qwen-image 系列、Wan 模型 |
| 翻译 | Qwen-mt 系列 |

### 方案优势

- **零代码集成**：仅需 SQL 即可调用 AI 函数，分析师无需学 Python
- **实时高性能**：内核级函数调用，百亿级向量亚秒级响应
- **数据零搬迁**：直接在数仓中完成全流程，数据不出库
- **灵活低成本**：内置多种百炼模型一键部署，按 Token 调用按需使用

### 适用场景

智能搜索推荐、实时风控异常检测、智能内容生成（营销文案/广告视频/语音合成）、语音处理与交互。

## 智能营销内容生成场景

打通素材 → 生成 → 投放分析全链路：

1. **准备素材**：上传蒙版图片（如刘备/吕布人物蒙版）到 OSS
2. **Object Table 对接**：Hologres 读取 OSS 上的非结构化物料
3. **AI 图片生成**：AI Function 调用百炼模型，根据 Prompt + 风格参数生成营销图片（如"北京枭雄风格"）
4. **分镜脚本生成**：对生成的图片写分镜脚本
5. **AI 视频生成**：调用 Wan 模型根据分镜脚本生成视频
6. **投放效果分析**：投放数据写入 Hologres，进行效果闭环分析

整个过程可由 DataAgent/OpenClaw 触发，按 Skills 中步骤自动串联执行。

### 客户画像

游戏、电商、互联网、广告行业中：日常需大量生产营销素材，产能瓶颈明显，人工创作效率低、风格不统一，期望提高创作效率、实现自动化流水线。

## Hologres CLI & Skills

Agent Ready 的一站式实时数仓工具链：

### Hologres CLI

内置安全防护措施，支持结构化输出：
- 实例管理、计算组管理
- 数据库管理、用户授权
- 表/视图等元数据管理
- DQL 执行与结果导出
- 动态表管理
- 备份与安全

### Hologres Skills

| 类别 | 能力 |
|------|------|
| **智能运维** | 慢 Query 分析、SQL 优化 |
| **最佳实践** | Flink+Hologres 搭建实时数仓、专家权限模型授权、RoaringBitmap 长周期 UV 计算、Bit-sliced Index 行为标签计算 |
| **智能开发** | 自然语言转 DDL、智能化建表、AI Function 广告素材生成、多模态数据分析与检索 |

## Hologres Mem0：Agent 长记忆方案

解决 Agent “历史记忆遗忘、跨设备从零开始”的痛点：

```bash
openclaw plugins install @hologres/openclaw-mem0
```

基于 Hologres 实现 [[openclaw]] 的长记忆增强，让 Agent 记住用户偏好和历史上下文。

## 相关页面

- [[dataworks-data-agent]] — DataWorks Data Agent 产品
- [[maxcompute-skills]] — MaxCompute 同类 Skills 体系
- [[flink-skills]] — Flink Skills 联动方案
- [[emr-skills]] — EMR Skills
- [[maxcompute-data-ai]] — MaxCompute 多模态存储参考
- [[openclaw]] — OpenClaw Agent 框架
