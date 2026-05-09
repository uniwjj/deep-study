---
title: OpenSpec SDD 实战
description: 真实企业项目中 OpenSpec 跑通 SDD 全流程——Brownfield-first 设计、七步工作流（Explore→Archive）、MCP 连接 Azure DevOps/Confluence/Figma
aliases: [OpenSpec, SDD实战, 规范驱动开发]
tags: [ai-agent, practice]
sources: [2026/04/08/OpenSpec 实战复盘.md]
created: 2026-05-09
updated: 2026-05-09
---

# OpenSpec SDD 实战

## 选型原因

1. **Brownfield-first**：增量描述变更，无需重写整个系统 Spec
2. **Fluid not rigid**：无死板流程，随时回溯修改
3. **工具无关**：不挑 IDE/模型，支持 20+ AI 编码助手

MCP 连接：Azure DevOps（拉 User Story）+ Confluence（业务背景）+ Figma（UI 设计）

## 七步工作流

```
Explore → New → Implement → Code Review → Verify → Testing → Archive
```

| 步骤 | 做什么 | 产出 |
|------|--------|------|
| Explore | 拉需求+澄清模糊点 | 事前对齐清单 |
| New | 逐件创建+人工 review | proposal/spec/design/tasks |
| Implement | AI 按 tasks 逐项实现 | 代码 |
| Code Review | AI 审 AI（Spec+团队规范） | Review 报告 |
| Verify | 完整性/正确性/一致性检查 | 验证结果 |
| Testing | 人工验收 | 确认 |
| Archive | Delta Spec 合并入主 Spec | 活的系统说明书 |

## Delta Spec 格式

```
GIVEN admin on workflow config page
WHEN admin defines a 3-step approval workflow
THEN workflow is saved AND audit log created
```

## 四个坑

1. 微服务下每服务独立 Spec + schema
2. Spec 漏细节时 Spec 永远是 source of truth
3. 团队抵触随代码质量提升而消退
4. 工具迭代快，定期 `openspec update`

## 2-3 迭代后变化

- 代码质量和一致性显著提升
- 需求前置对齐减少返工
- Spec 归档后成为活的系统说明书

SDD 本质是 **Harness Engineering**：让 AI 跑在正确的马路上，并由人牵着这匹马。

## 相关页面

- [[progressive-spec-methodology]] — 渐进式 Spec 方法论
- [[superpowers-framework]] — Superpowers 框架
- [[agent-tdd-workflow]] — TDD 工作流
