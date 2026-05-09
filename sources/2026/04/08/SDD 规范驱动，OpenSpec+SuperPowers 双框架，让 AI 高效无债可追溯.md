---
ingested: 2026-05-09
wiki_pages: [ai-agent/sdd-openspec-superpowers]
---

## SDD 规范驱动，OpenSpec+SuperPowers 双框架

**文章信息**：
- **原文标题**：SDD 规范驱动，OpenSpec+SuperPowers 双框架，让 AI 高效无债可追溯
- **来源**：https://www.toutiao.com/article/7626258875386167859
- **作者**：AI码韵匠道
- **发表时间**：2026-04-08 13:44

---

**一段话总结**：

规范驱动开发（SDD）用"先想清楚再动手"的理念，解决 AI 编码不规范、技术债堆积的问题。SDD 形成需求→规范文档→AI 按规范编码→验证的闭环流程。文章拆解了两大落地框架：OpenSpec 侧重变更追溯与知识积累，通过 Artifact 链实现全流程可追溯；Superpowers 侧重执行质量与自动化审查，支持多平台、多代理协作。

![SDD 工作流概念图](https://p3-sign.toutiaoimg.com/tos-cn-i-axegupay5k/ae846e9f6ee64e618322114b8d2ecaf3~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776667679&x-signature=e5sTYXdNidPPnfy%2Fah1bIs1HThs%3D)

---

**作者判断**：

作者明确推荐 SDD 作为 AI 时代的核心开发范式，认为"先规范后编码"能从根本上解决 AI 编码的质量问题。作者立场清晰：AI 的高效必须用规范来约束，否则"AI 写代码"最终变成"AI 制造技术债"。作者同时认为 OpenSpec 和 Superpowers 各有所长——OpenSpec 适合注重变更追溯和长期维护的项目，Superpowers 适合追求执行质量和自动化的团队，建议根据项目特点灵活选用。

---

**核心内容**：

**传统 AI 辅助编码的三个致命问题**：
1. **需求传递偏差**：用户需求描述碎片化（如"写一个后台服务启动检查功能"），AI 只能根据字面猜测
2. **缺乏可追溯性**：代码出问题时无法追溯到设计意图
3. **技术债积累**：AI 生成的代码可能冗余、不规范，长期积累难以清理

**SDD 的核心逻辑**：需求→生成规范文档→AI 按规范实现代码→按规范验证代码。三个优势：需求传递更精准、开发过程可追溯、减少技术债。

**OpenSpec 框架详解**：
- 基于 Node.js，全局安装命令：`npm install -g @fission-ai/openspec@latest`
- 初始化：`cd your-project && openspec init`
- 核心是"Artifact 依赖链"：proposal.md → design.md → specs/*.md → tasks.md → 代码实现 → 归档
- 每个 Artifact 职责明确：proposal.md 阐述"为什么做"、design.md 记录"怎么做"、specs 定义"做成什么样"、tasks.md 拆解"分几步做"
- 常用命令：`/opsx:explore`（探索模式）、`/opsx:propose`（提案）、`/opsx:ff`（快速通道，简单需求一次生成全部文档）、`/opsx:apply`（按任务实现）、`/opsx:verify`（验证）、`/opsx:archive`（归档）
- 独特设计"主 spec + delta spec"体系：主 spec 是项目级知识库，delta spec 是变更级增量，归档时自动合并到主 spec

**Superpowers 框架详解**：
- 由 Jesse Vincent 开发，支持 Claude Code、Cursor、Codex、Gemini CLI 等多平台
- 核心是"Skills 组合"：Markdown 格式的指令文件，按功能分为协作（brainstorming）、规划（writing-plans）等类别
- 特色是"子代理驱动开发"和"强制 TDD"
- Superpowers 中一个 skill 定义了关键原则：① 只生成验证计划而非代码 ② 强制 TDD，先写失败测试 ③ 子代理必须深入源码验证行为

**两框架对比**：
- OpenSpec：聚焦变更管理、知识积累、工具无关
- Superpowers：聚焦执行质量、自动化审查、多代理协作
- 均为开源免费，可搭配使用
