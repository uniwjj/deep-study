---
title: 渐进式 Spec 方法论
description: 2026 年 AI 编程新范式——根据任务复杂度灵活调整规范粒度，三条铁律 + 四步闭环（Propose→Apply→Review→Archive），知识飞轮效应
aliases: [Progressive Spec, Spec Coding, 渐进式规格]
tags: [ai-agent, practice, concept]
sources: [2026/04/05/渐进式Spec新范式.md]
created: 2026-05-09
updated: 2026-05-09
---

# 渐进式 Spec 方法论

## 核心问题

Vibe Coding（随口描述让 AI 写）= 口述传话；Spec Coding = 书面合同。

AI 编码翻车四大原因：
- 接口命名不匹配项目规范
- 边界情况全漏
- 改三遍还不对
- 协作方式不对（非模型不够聪明）

## 核心思想

根据任务复杂度灵活调整：简单需求直接走、中等加规格文档、复杂走完整流程。

## 三条铁律

1. **先 Spec 后编码** — 没写清楚不准写
2. **Spec 覆盖所有场景** — WHEN/THEN 句式穷举
3. **Reverse Sync** — 发现偏差先更新 Spec 再改代码，保证 Spec 是唯一真相源

## 四步闭环

```
Propose（人主导）→ Apply（AI主导）→ Review（双Agent质检）→ Archive（知识沉淀）
```

### Step 1: Propose
- 人主导，AI 辅助
- AI 自动 Research 代码现状
- 分段提问收敛模糊需求
- 产出：spec.md + tasks.md + log.md

### Step 2: Apply
- AI 主导，人审查
- 按 tasks.md 逐步执行
- 每个 task 提供可验证证据
- 自动 commit（每个 task 一个 commit）

### Step 3: Review
- 两阶段 Sub Agent 质检
- **Spec Reviewer**：检查实现是否符合 spec
- **Code Quality Reviewer**：检查代码质量/安全/性能
- 发现问题进入 Fix 循环

### Step 4: Archive
- 经验教训写入 knowledge/ 知识库
- 变更记录归档
- Delta Specs 合并到主规格文档

## 项目结构推荐

```
rules/      — 工程记忆（技术栈、规范、安全红线）
knowledge/  — 团队集体智慧（踩坑记录、架构决策）
changes/templates/ — 标准化工件模板
agents/     — 不同角色 Prompt 配置
```

## 知识飞轮效应

> "今天踩的坑明天变规则，本周的方案下月变模板。时间越长越值钱，别人抄不走。"

## 模型梯队策略

编排层用 T0/T1 强模型做决策和 Spec 生成；执行层用 T1/T1.5 模型写代码。

## 五大致命坑

1. 没想清楚就让 AI 写
2. 调研阶段就要求代码产出
3. 踩过的坑不记录
4. 迷信特定工具（方法论不变）
5. 忽视 Git 规范

## 相关页面

- [[superpowers-framework]] — Superpowers 工程纪律框架
- [[sdd-openspec-superpowers]] — SDD 规范驱动开发
- [[agent-tdd-workflow]] — Agent TDD 流程
- [[agent-skills-system]] — Agent Skills 体系
