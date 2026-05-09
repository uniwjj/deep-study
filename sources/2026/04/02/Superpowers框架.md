---
ingested: 2026-05-09
wiki_pages: [ai-agent/superpowers-framework]
---

## 一个被严重低估的 Skills——Superpowers

**文章信息**：
- **原文标题**：一个被严重低估的 Skills——Superpowers
- **来源**：https://www.toutiao.com/article/7624175963751957001/
- **作者**：硅基观察室
- **发表时间**：2026-04-02 22:58

---

**一段话总结**：

Superpowers 是一套给 AI 编程 Agent 用的技能框架和软件开发方法论，不是单点功能插件。它的核心价值不是让 AI "会更多"，而是给 AI 灌输工程纪律——需求没清楚不准写代码，没写测试不准开始，没验证通过不准宣布完成。工作流分四步：先澄清需求再动手，拆任务到 bite-sized 粒度，子 Agent 驱动开发（每个任务独立执行+检查+review），强制 TDD（RED-GREEN-REFACTOR）。内置技能覆盖测试、调试、协作执行、Git 分支管理四大类。支持 Claude Code、Codex、OpenCode 安装。GitHub：github.com/obra/superpowers

---

**作者判断**：

作者明确肯定 Superpowers 的重要性，认为它"抓住的是一个更底层的问题：AI 写代码的真正瓶颈，很多时候不是不会写，而是写的过程太随意"。作者的核心观点是：Superpowers 不是"让 AI 更炫"，而是"让 AI 更像一个能长期合作的工程师"。同时作者也坦承它有局限：小任务可能显得流程太重，效果吃模型执行力，不是魔法而是"把成功率抬高的工作流框架"。

---

**核心内容**：

**问题定义**：Claude Code、Codex 等 AI 编程工具写代码快但"太急了"——不问需求、不做设计、不写测试就直接开写，像"手很快但不太稳的初级工程师"。Superpowers 解决的核心问题是**能力编排**，让 AI 知道什么时候该先问清楚、先出设计、拆任务、写测试、review，不能急着宣布搞定。

**四步工作流**：

1. **先聊清楚再动手**：AI 的第一反应不是写代码，而是问需求边界、约束条件、是否有更简单的做法，然后输出设计文档供确认
2. **拆任务到 bite-sized**：拆到具体改哪个文件、做什么修改、怎么验证、前后依赖是什么，控制在几分钟内完成的粒度
3. **子 Agent 驱动开发**：每个任务交给新子 Agent 执行，做完后检查、review，再决定是否继续下一个——不让 AI 一口气"自由发挥几个小时"
4. **强制 TDD**：RED-GREEN-REFACTOR 流程是工作流的一部分而非建议——先写失败测试，再写刚好让测试通过的代码，再做重构优化

**内置技能分类**：
- **测试**：test-driven-development、verification-before-completion
- **调试**：systematic-debugging
- **协作与执行**：brainstorming、writing-plans、executing-plans、dispatching-parallel-agents、subagent-driven-development、requesting-code-review、receiving-code-review
- **Git/分支管理**：using-git-worktrees、finishing-a-development-branch
- **元技能**：writing-skills、using-superpowers

覆盖的不是"怎么写一段代码"，而是从需求→设计→拆任务→实现→测试→Review→收尾的完整链路。

**核心哲学**：把优秀的软件工程习惯变成 AI 的默认行为——需求没清楚不准直接写、没做计划不准直接冲、没写测试不算真正开始、没验证通过不准宣布完成、没 review 不算真正交付。

**适合人群**：
1. 日常已用 AI 写代码但感觉"写得快但不够稳"的人
2. 想让 AI 更像工程同事的人
3. 对 Agent 工作流有兴趣的人

**三个局限**：
1. 强流程化对小任务显得重
2. 不同平台体验不一致（Claude Code 最完整）
3. 效果吃模型执行力

**安装方式**：
- Claude Code：`/plugin marketplace add obra/superpowers-marketplace` → `/plugin install superpowers@superpowers-marketplace`
- Codex / OpenCode：按仓库 INSTALL.md 接入

![Superpowers 工作流演示：展示 AI Agent 如何按纪律化流程执行开发任务](https://p26-sign.toutiaoimg.com/tos-cn-i-axegupay5k/1c3b067755054d97a08ea07820b97e29~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776654192&x-signature=zIXJYrNq07fJOngqN8iJ66tv3bc%3D)

![Superpowers 子 Agent 驱动开发流程示例](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/6668380723f84b56b97484b0e273f8ff~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776654192&x-signature=eAHEz2rcmVkxprZ2ciqIWHXvesc%3D)

![Superpowers TDD 强制执行界面](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/0e052154af1245de8c9561dd5fc3345e~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776654192&x-signature=Nh0%2FIoFRvu1dKYL7lE4109i%2BzY0%3D)
