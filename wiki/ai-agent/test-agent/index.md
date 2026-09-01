---
title: AI 测试自动化
description: AI 测试自动化子领域索引——Test Agent、测试 Harness、Mara/Etest 平台、配表驱动数据测试与业界实践
aliases: [test-agent, AI测试, Test Agent, 智能测试]
tags: [ai-agent, meta, summary]
sources: [2026/09/01/【AI测试自动化】跨部门交流会实录（文字稿）.html, 2026/09/01/【tm599】云音乐自动化测试阶段性同步.html, 2026/09/01/游戏数据年报类活动AI测试实践分享.html]
created: 2026-09-01
updated: 2026-09-01
---

# AI 测试自动化

网易内部 AI 测试自动化实践与业界对比的专题子领域。核心协作范式：**AI 负责概率（理解与生成），Harness 负责确定性（流程、状态、门禁、有限恢复），人负责风险和关键判断**。

## 页面索引

| 页面 | 说明 |
|------|------|
| [[ai-test-cross-team-review]] | 网易五团队（云音乐/效工/严选/雷火/有道）AI 测试跨部门交流会综述与焦点议题 |
| [[test-agent-harness-paradigm]] | 协作范式：共建复利运营、测试左移、L1~L4 分层、Test Agent 工程化四问题、Harness 职责拆分 |
| [[mara-test-platform]] | 云音乐 Mara 一站式 AI 自动化测试平台：QA WebUI 五阶段、文本用例即脚本、AI 断言 |
| [[config-driven-data-testing]] | 配表驱动的数据展示测试：年报五阶段流水线（算→验→定→跑→报+审），提效 33~66 倍 |
| [[cc-qa-skill-suite]] | 雷火 ccc-qa Skill 家族：用例/接口（模式A·B）/UI/年报/多语言五线串联与报告发布 |
| [[etest-quality-platform]] | 效率工程部 Etest 平台：五层架构、跨域全链路串联、规模化应用 |
| [[yanxuan-api-test-agent]] | 严选 AI 接口自动化：三阶段流水线、四级数据构造、双卡片打分、确定性原则 |
| [[youdao-codex-testing]] | 有道叭哥说×Codex：1 名测试+Computer Use 的跨端质量实践，17 天三端内测 |
| [[ai-test-industry-practices]] | 业界 AI 测试实践对比：Meta JiT、字节 Midscene、美团 KuiTest、快手、小红书、阿里、去哪儿 |

## 一句话结论

- 生码能力趋同后，**业务知识、环境数据、可信断言和工程闭环**才是壁垒
- **分层比全覆盖重要**：L1~L4 各层由最合适的角色负责，不追求全自动 E2E
- 范式迁移：从「人做事」到「人定规则、AI 做事、人审结果」

## 相关页面

- [[homepage]]
- [[agent-harness-overview]] — Agent Harness 通用模式
- [[meta-jit-testing]] — Meta 即时测试
- [[spec-kit]] — SDD 规范驱动开发
