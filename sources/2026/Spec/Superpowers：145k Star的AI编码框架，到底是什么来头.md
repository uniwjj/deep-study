---
ingested: 2026-05-09
wiki_pages: [ai-agent/superpowers-framework]
---

# Superpowers：145k Star的AI编码框架，到底是什么来头？

**来源**：人人都是产品经理  
**时间**：2026-04-11  
**链接**：https://www.toutiao.com/article/7627381688884052518

## 一段话总结

Superpowers是一个145k Star的AI编码代理技能框架，核心思路是不直接让AI写代码，而是先教它做合格的软件工程师。采用7步工作流程（头脑风暴→隔离工作空间→任务分解→子代理驱动开发→TDD测试驱动→代码审查→完成分支），支持Claude Code、Cursor、Codex等多种AI编码工具。

## 作者判断

作者认为Superpowers的本质是"帮AI建立更好的开发习惯"，而非帮AI写更好的代码。它假设AI代理是"缺乏判断力的初级工程师"，需要人来把控方向。对个人开发者是"AI编程最佳实践"，对企业团队是把AI编程经验从个人上升到组织层面的工具。但提醒目前无商业化支持，企业使用需自行维护。

## 核心内容

**GitHub数据**：145k Stars、12.4k Forks、612 Watchers，最新版本v5.0.7（2026年3月31日），MIT协议免费开源。

![Superpowers GitHub仓库结构](https://p3-sign.toutiaoimg.com/tos-cn-i-axegupay5k/1c280618ee78446a81ff23b135a81d10~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669524&x-signature=NQGJ4cMe5Wphih0G7WNyjCjNRM8%3D)

**7步工作流程**：
1. **brainstorming**：先对话精炼模糊想法，把设计拆解给用户确认
2. **using-git-worktrees**：创建隔离Git worktree，实验代码不污染主分支
3. **writing-plans**：任务分解到2-5分钟，含确切文件路径和验证步骤
4. **subagent-driven-development**：每个小任务派发子代理，AI变"项目经理"
5. **test-driven-development**：强制RED-GREEN-REFACTOR，先写测试再写实现
6. **requesting-code-review**：持续审查，按严重程度分类报告
7. **finishing-a-development-branch**：验证所有测试，提供合并或PR选项

**与竞品差异**：
- Claude Code：superpowers是它的"方法论插件"
- Cursor/Copilot：superpowers提供完整工作流程框架
- SWE-agent/Devin：superpowers强调人机协作而非全自动化

**安装方式**：Claude Code用`/plugin install superpowers@claude-plugins-official`，Cursor搜索插件市场。
