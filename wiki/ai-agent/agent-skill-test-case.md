---
title: Agent Skill：QA 测试用例
description: 将 QA 测试用例规范写成 Agent Skill——三种模式（生成/补全/评审）、七项必填字段、书写规范，可随仓库提交的团队标准化方案
aliases: [test-case-skill, 测试用例Skill, QA测试用例Agent]
tags: [ai-agent, practice]
sources: [2026/04/05/Agent Skills测试用例Skill.md]
created: 2026-05-09
updated: 2026-05-09
---

# Agent Skill：QA 测试用例

## 问题

团队测试用例格式不统一：不写前置条件、步骤不可执行、预期结果含糊、无法追溯需求。

## 方案

将用例模板和书写规范写成一条 Agent Skill（`test-case/SKILL.md`），助手触发时按同一套结构输出。

## 七项必填字段

| 字段 | 要求 |
|------|------|
| 用例 ID | 唯一标识 |
| 标题 | 格式：模块-场景-预期 |
| 关联需求 | PRD/用户故事/接口 |
| 前置条件 | 无则明确写"无" |
| 测试步骤 | 按顺序编号，一步一动作 |
| 预期结果 | 可观察、可判断通过/失败 |
| 优先级 | P0/P1/P2 |

可选字段：测试类型、测试数据、备注、执行结果。

## 三种输出模式

1. **生成模式** — 从需求直接生成测试用例组
2. **补全模式** — 补充已有用例缺失字段、异常场景、边界 case
3. **评审模式** — 检查已有用例可执行/可验证/可追溯，输出必改/建议/可选清单

## 书写规范

- 标题避免笼统（"登录测试" → "登录-输入正确账号密码后成功进入首页"）
- 步骤必须可执行（一步一动作，避免"进行登录操作"）
- 预期结果必须可验证（"接口返回 200 且 body 含 token"）
- 每条用例至少关联一个需求 ID

## 目录结构

```
test-case/
├── SKILL.md          # 核心行为规范
├── reference.md      # 完整模板和命名规范
└── scripts/          # 可选：字段完整性检查脚本
```

## 与其他 Skill 协作

- **测试计划**：回答"测什么、怎么安排"
- **测试用例**：回答"每条怎么测、怎么判定通过"
- **测试报告**：总结"测得怎么样、能不能过"

## 团队协作

Skill 放在 `.cursor/skills/` 随仓库提交，新技能经 PR review。

## 相关页面

- [[agent-skills-system]] — Agent Skills 体系
- [[log-diagnosis-skill]] — 日志诊断 Skill（得物实战）
- [[superpowers-framework]] — Superpowers 框架
- [[agent-tdd-workflow]] — Agent TDD 流程
