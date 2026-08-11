---
title: AI Agent 改变 Builder 的工作流（ContextEcho 实证）
description: Cursor Ambassador 杨文在 DataFun Agentic AI 大会的演讲——用 3 天 MVP、76 天上架、468 commits 与 213 段 AI 对话的完整留档，实证 AI Agent 如何把 Builder 的工作流从「流水线」变成「反馈环」：AI 负责执行、人负责边界
aliases: [杨文, Mai Yang, ContextEcho, Builder 工作流, AI Agent 工作流, 从写代码到构建产品, Cursor Ambassador]
tags: [ai-agent, practice, summary]
sources: ["2026/08/11/前沿探索与超级Agent论坛/04-杨文-从写代码，到构建产品AlAgent 正在改变 Builder的工作流.pdf"]
created: 2026-08-11
updated: 2026-08-11
---

# AI Agent 改变 Builder 的工作流（ContextEcho 实证）

> 2026 Agent 大会（DataFun Agentic AI 大会）演讲《从写代码，到构建产品：AI Agent 正在改变 Builder 的工作流》。演讲者**杨文（Mai Yang）**，Cursor Ambassador，深圳，十余年后端工程师。
>
> 一句话开场：「一个用 AI 造出来的 App，它现在，没有任何 AI 功能。**AI 在工作流里，不在功能列表里。**」

## 产品实证：一个真实的产品

**ContextEcho**——iPhone-only 内容消费与轻量学习 App，3 天做出 MVP、76 天走到 App Store。

### 全部来自 Git 与对话记录的数字

| 项目 | 数据 |
|------|------|
| 从开工到上架 | 3 天 MVP / 76 天：04-14 写下第一行，04-16 交付 0.1.0（第 3 天已经是一个可以每天使用的 App），06-29 提交 App Store，正好 76 天 |
| 协作密度 | 468 commits / 213 段 AI 对话（每天 2.6 段对话） |
| 给 AI 的制度 | 11 条 rules（Cursor rules）+ 7 篇 spec，每条背后是一次翻车 |
| 最大一次砍功能 | -4,830 行：0.15.0 一个 PR、95 个文件——「做产品也要敢于让 AI 做减法」 |

真实构建（01）：04-14 开始、04-16 交付 0.1.0；完整留档（02）：468 commits、213 段 AI 对话、11 条 rules——「每次判断和翻车都有记录」；工程视角（03）：正因为会写代码，才更清楚这次变化改变的不是「写代码」本身。

## 起点：先写约束，再写代码（CONSTRAINTS FIRST）

- 2026-04-14 第一条对话给的是 **Plan Mode**，不是「帮我写代码」
- 先用 ChatGPT 聊清需求（需求梳理原文）：**先收敛需求再构建**——不直接给一份大 PRD，而是把想法收敛成可执行边界：一个「小而硬的 MVP Build Pack」（做什么、不做什么、页面、交互与数据模型），含 Problem Statement、Solution、User Stories、Implementation Decisions、Testing Decisions、Out of Scope、Further Notes，并补齐页面、交互、数据模型、技术边界——「先别让 Cursor 自己发散」
- 给 Cursor 的 Plan Mode 提示词：为 content consumption and lightweight learning app 规划 iPhone-only MVP——「This is not a traditional language learning app. It is an AI-enhanced layer for consuming high-quality content more smoothly, marking key moments, reconstructing memory after consumption, and only occasionally practicing one high-value sentence.」；技术边界：纯 Swift 本地方案（最小化后端依赖），AI 用 OpenAI 兼容的中转 API
- 伏笔——三次产品定位（后来被推翻了两次）：4 月「英语学习增强层（英文信息—中文理解，一个人沉淀）」；5 月「音频优先的『回声』（捕捉—回顾—重建记忆）」；6 月「克制的中文音频伴侣（每天都想打开的播放器＋一面镜子）」

## 工作流变成了循环：以前是流水线，现在是反馈环

- 以前 · 线性：Idea → 设计 → 写代码 → 调试 → 发布；每一步都有门槛（不会设计就卡在设计，不会调试就卡在调试）
- 现在 · 循环：Idea → AI Prototype → AI 产品 → 用户反馈 → AI；**卡住的地方，变成一次对话**
- 一个人做一款 iOS App 上架 ≈ 团队半年；**瓶颈从工程能力，转移到判断力**

## 五个真实案例（02 一 AI Agent 改变了什么）

### 案例 1/5 · UI：审美判断留给人

「我不喜欢紫色，很容易让人觉得low，AI 味道很重。」——附一张 Overcast 截图 + 色值 #FC7E11，主题色、分类色、图标一次重构（Before/After：紫色 AI 味 → Overcast 橙）。一次对话换掉全 App 主题。

### 案例 2/5 · 调试：不会 INSTRUMENTS 怎么办——逼 AI 给自己建可观测性

五步：01 真机现象（「点击播客单集要卡5秒」，反复描述修不好）→ 02 转折点（「你看不到我手机—那就在App里建一套诊断日志导出」）→ 03 现场取证（跑步遇到闪退 → 导出日志 → 微信传回电脑）→ 04 数据回流（日志扔进对话，从「我描述现象」升级为「我提供数据」）→ 05 定位根因（SwiftData 主线程查询 + SwiftUI 每帧重绘 → 采样时钟 + 节流）。

### 案例 3/5 · API：学习成本归零——每个源都是一个新方言

小宇宙、喜马拉雅、acast、Substack、Lex——每个内容源都是一套新方言。以前每个方言要读几天文档和逆向帖子，现在当天提问、当天上线。实例：Bilibili（WBI 签名 + 风控 -352、.dm_img 指纹对齐）；YouTube（embed error 152 / TLS 环境问题）；Podcast RSS（Podcast Index transcript 标准 + 各家时间轴格式）。

### 案例 4/5 · 重构：AI 的默认倾向是做加法

ContextEcho 代码量（按 commit，2026-04-16 至 2026-07-10）：42,453 当前代码行（cloc）、320 个 commits、+52,529 累计净增行。证据——播放一个领域堆出五个职责重叠的类：PlaybackBridge / PlaybackClock / PlaybackTimeBroker / PlayerSeekCoordinator / SmoothedClock。「增加的代码，总比减少的多」——增长很快，**收敛必须主动发生**（一天 29 轮收敛重构；AI 下线后出现谷底）。

### 案例 5/5 · Rules：给 AI 写「员工手册」——每条规矩背后，都是一次真实翻车

.cursor/rules/ 共 11 条 rules：

| rule | 翻车事件 → 产出的规矩 |
|------|----------------------|
| xcodegen-workflow.mdc | 新文件编译不进：Cannot find X in scope |
| product-principles.mdc | 「你不太能遵循 iOS 开发基本理念」——最高准则 |
| doc-sync-gate.mdc | 文档与代码脱节：代码是事实，文档是意图 |
| scope-boundary | 怕 AI 顺手加播放列表：「只支持单集播放」 |
| release-notes-sync.mdc | 发行说明漏写：每次 Minor 必须同步 |

> 小结：AI 负责执行，人负责边界。「以前我管代码，现在我管一个不知疲倦、但需要边界的合作者。」管好边界比盯住每一行代码更重要。

## 三个真实失败案例（03 不讲成功）

### 案例 A：被推翻四次的「回声」（产品名就来自这个功能）

| 版本 | 变化 |
|------|------|
| 04-22 · 0.7.0 | 音频优先大改版（「回声」成为核心叙事） |
| 04-23 · 0.8.0 | 播完自动回顾（回听补充体验闭环） |
| 04-24 · 0.9.0 | 标记收敛（五种标记收敛成两种） |
| 05-29 · 0.15.0 | 整个 Tab 下线（-4830 行，95 个文件） |

4 versions、37 天。

### 真实使用推翻原设计（2026-05-28 对话原文）

「我几乎不打开我们新设计出来的回声库……近期也没有再怎么用过回声功能了。我每天都用它来听播客、看bilibili视频。对听过、看过的数据有一种成就感。目前已经连续 42 天，最长连续 42 天。累计 97 小时 14 分钟。」

AI 的翻译与结论：真实在用的是——播放（真实在用）；统计/连续天数/累计时长/每个节目的明细（真实在用，而且是「给你愉悦感的主要来源」）；回声捕获（偶尔用，一周 3-5 次，打完不管）；回声库/回看/沉淀（几乎不用）。**vision 里「主用户」是那个会回看、会沉淀、会形成知识资产的人——不是你自己；vision 里降级到次要的「学习统计/连续天数/时长面板」反而是真实在用的核心。你 vision 的主次关系，被你的真实行为彻底翻转了。**

> 杀死「回声」的，不是 AI 的分析，是我自己。

（PDF 共 20 页，原 PPT 页码标注至 35 页；第 21 页起的「可迁移的方法」等内容不在 PDF 内。）

## 相关页面

- [[agent-design-paradigms]] — Agent 设计范式（Plan Mode 先规划后执行、反馈环与 ReAct 的关系）
- [[agent-skills-system]] — Agent 技能系统（.cursor/rules 员工手册与技能治理的实践对照）
- [[agent-tdd-workflow]] — Agent TDD 工作流（约束先行、边界管理）
- [[agent-architecture-patterns]] — Agent 架构模式（工具自描述、状态管理在真实产品中的体现）
- [[agent-hook-governance]] — Hook 护栏治理（规则即护栏：scope-boundary 防越权）
- [[next-gen-agent-form]] — 下一代 Agent 形态探索（同为 2026 Agent 大会论坛演讲）
