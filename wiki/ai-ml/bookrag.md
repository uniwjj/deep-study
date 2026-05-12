---
title: BookRAG
description: 港中深团队结构感知 RAG——BookIndex（层级树+知识图谱+GT-Link）+ 代理动态检索，基于信息觅食理论，三大基准 SOTA
aliases: [BookRAG, BookIndex, 结构感知RAG]
tags: [ai-ml, concept, paper]
sources: [2026/04/06/BookRAG.md]
created: 2026-05-09
updated: 2026-05-09
---

# BookRAG

## 现有 RAG 两大缺陷

1. 文本方法丢失文档布局，布局方法难捕捉跨章节关联
2. 所有问题用同一种检索策略，效率低下

## BookIndex：混合索引

```
层级树(Tree) + 知识图谱(Graph) + GT-Link
```

| 组件 | 作用 |
|------|------|
| 层级树 | 解析标题/章节/表格/图片，充当"目录" |
| 知识图谱 | 从树节点提取实体和关系 |
| GT-Link | 每个实体映射回来源树节点 |

**梯度式实体消歧**：通过分析相似度分数的"陡降点"识别并合并别名，避免昂贵全局比较。

## 代理检索（信息觅食理论）

三个阶段：
1. **代理规划**：分类问题（单跳/多跳/全局聚合），四类操作符（规划/选择/推理/合成）组合执行计划
2. **结构化执行**：气味过滤→推理精炼（天际线排名保留帕累托前沿）→合并生成
3. **生成答案**

## 实验结果

| 基准 | BookRAG | Vanilla RAG |
|------|---------|-------------|
| M3DocVQA | **61.0** | 36.5 |
| Qasper F1 | **61.1** | 44.4 |

全 SOTA，大幅领先所有基线。

## 三大创新

1. 文档原生混合索引（层级树+知识图谱）
2. 动态智能检索代理（信息觅食理论）
3. 验证结构感知 RAG 巨大潜力

论文: arxiv.org/pdf/2512.03413 | 代码: github.com/sam234990/BookRAG

## 相关页面

- [[llm-wiki-concept]] — LLM Wiki 编译式知识库
- [[rag-vs-llm-wiki]] — RAG vs Wiki 对比
- [[llm-wiki-implementations]] — 开源实现
