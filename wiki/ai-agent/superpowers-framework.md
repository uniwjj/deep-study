---
title: Superpowers 框架
description: AI 编程 Agent 的工程纪律框架——四步工作流（需求澄清→任务拆分→子Agent驱动→强制TDD），覆盖从需求到交付的完整链路
aliases: [Superpowers, AI编程纪律框架, Superpowers Skills]
tags: [ai-agent, tool, practice]
sources: [2026/04/02/Superpowers框架.md]
created: 2026-05-09
updated: 2026-05-09
---

# Superpowers 框架

## 定位

Superpowers 不是单点功能插件，而是一套**给 AI 编程 Agent 灌输工程纪律的方法论**。核心价值不是让 AI "会更多"，而是让 AI "不乱来"。

问题根源：AI 编程 Agent 像"手很快但不太稳的初级工程师"——不问需求、不做设计、不写测试就直接开写。

## 四大工程纪律

- 需求没清楚 → **不准直接写**
- 没做计划 → **不准直接冲**
- 没写测试 → **不算真正开始**
- 没验证通过 → **不准宣布完成**
- 没 review → **不算真正交付**

## 四步工作流

```
需求澄清 → 任务拆分(Bite-Sized) → 子Agent驱动开发 → 强制TDD(Red-Green-Refactor)
```

### 1. 先聊清楚再动手
AI 第一反应不是写代码，而是问：需求边界、约束条件、是否有更简单的做法 → 输出设计文档供确认。

### 2. 拆任务到 Bite-Sized
拆到具体改哪个文件、做什么修改、怎么验证、前后依赖，控制在几分钟内完成的粒度。

### 3. 子 Agent 驱动开发
每个任务交给新子 Agent 执行 → 做完后检查、review → 再决定是否继续。不让 AI 一口气"自由发挥几个小时"。

### 4. 强制 TDD
RED-GREEN-REFACTOR 是工作流的一部分而非建议：
- RED：先写失败测试
- GREEN：写刚好让测试通过的代码
- REFACTOR：重构优化

## 内置技能分类

| 类别 | 技能 |
|------|------|
| **测试** | test-driven-development, verification-before-completion |
| **调试** | systematic-debugging |
| **协作执行** | brainstorming, writing-plans, executing-plans, dispatching-parallel-agents, subagent-driven-development |
| **Code Review** | requesting-code-review, receiving-code-review |
| **Git 管理** | using-git-worktrees, finishing-a-development-branch |
| **元技能** | writing-skills, using-superpowers |

## 安装

- Claude Code：`/plugin marketplace add obra/superpowers-marketplace` → `/plugin install superpowers@superpowers-marketplace`
- Codex / OpenCode：按 INSTALL.md 接入

## 局限

1. 强流程化对小任务显得重
2. 不同平台体验不一致（Claude Code 最完整）
3. 效果吃模型执行力

> "Superpowers 不是让 AI 更炫，而是让 AI 更像一个能长期合作的工程师。"

## 适用场景

Superpowers 强调**个体赋能**，适合快速原型、个人项目和探索性开发。在需要团队一致性的场景中，需搭配 [[openspec-sdd-practice]] 或参考 [[superpowers-vs-openspec]] 选择框架。

## 相关页面

- [[superpowers-vs-openspec]] — Superpowers vs OpenSpec 哲学对比
- [[agent-tdd-workflow]] — Agent TDD 开发流程
- [[sdd-openspec-superpowers]] — SDD 规范驱动开发
- [[agent-skills-system]] — Agent Skills 能力体系
- [[claude-code]] — Claude Code 开发工具
