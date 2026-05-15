---
title: SDD：OpenSpec + Superpowers
description: 规范驱动开发双框架对比——OpenSpec 聚焦变更追溯+知识积累，Superpowers 聚焦执行质量+自动化审查
aliases: [SDD, 双框架, OpenSpec+Superpowers]
tags: [ai-agent, practice, comparison]
sources: [2026/04/08/SDD 规范驱动，OpenSpec+SuperPowers 双框架.md]
created: 2026-05-09
updated: 2026-05-09
---

# SDD：OpenSpec + Superpowers

## SDD 核心逻辑

```
需求 → 规范文档 → AI 按规范编码 → 按规范验证
```

解决三个致命问题：需求传递偏差、缺乏可追溯性、技术债积累。

## 双框架对比

| | OpenSpec | Superpowers |
|---|---|---|
| **聚焦** | 变更管理、知识积累 | 执行质量、自动化审查 |
| **核心理念** | Artifact 依赖链 | Skills 组合+子代理驱动 |
| **独特设计** | 主 Spec + Delta Spec | 强制 TDD |
| **工具绑定** | 工具无关 | 多平台（CC/Codex/Cursor） |
| **适合** | 变更追溯、长期维护 | 执行质量、自动化 |

## OpenSpec 核心

Artifact 链：`proposal.md → design.md → specs/*.md → tasks.md → 代码 → 归档`

主 spec + delta spec 体系：变更级增量归档时自动合并。

常用命令：`/opsx:explore /opsx:ff /opsx:apply /opsx:verify /opsx:archive`

## Superpowers 核心

Skills 组合协作+子代理驱动+强制 TDD。只生成验证计划而非代码，先写失败测试再实现。

## 搭配建议

两框架可互补使用：OpenSpec 管理变更+知识，Superpowers 保障执行质量。

## 哲学层面

两者的路线分歧本质是 **个体效率优先 vs 团队一致性优先**。详见 [[superpowers-vs-openspec]]。

场景选择：创新平原（0→1）用 Superpowers，复杂城池（1→N）用 OpenSpec。最高效的开发者应成为能在两种模式间自如切换的"双语者"。

## 相关页面

- [[superpowers-vs-openspec]] — 个体效率 vs 团队一致性，哲学层面深度对比
- [[openspec-sdd-practice]] — OpenSpec 实战复盘
- [[superpowers-framework]] — Superpowers 框架
- [[progressive-spec-methodology]] — 渐进式 Spec
