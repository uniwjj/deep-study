---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-managed-agents]
---

# Anthropic新发布的工程博客，重新思考Agent的新架构

**文章信息**
- 来源：今日头条
- 作者：王鹏LLM
- 发布时间：2026-04-09 18:41
- 原文链接：https://www.toutiao.com/article/7626707383988847156

## 一段话总结

Anthropic工程博客提出Agent新架构：把"脑子"和"手"拆开。将Agent拆为Session、Harness、Sandbox三个独立模块，解耦后的p50 TTFT降60%、p95降90%。

## 作者判断

作者王鹏以第一人称深度解读Anthropic博客，立场务实且有独立思考。既认可拆分架构的价值，也指出耦合设计有其道理（简单直接），最后抛出开放问题引人深思，属于高质量技术解读。

## 核心内容

### 踩坑：一个容器干所有事

Anthropic最开始的设计把session、harness、sandbox全塞到一个容器里。这导致容器变成了运维圈所说的**"宠物"**（不能丢、要精心照料），而非**"cattle"**（死了一个换一个）。容器挂了session丢了，卡住了又不能随便进去看（涉及用户数据），形成死循环。

### 拆法：脑子和手，分开

Anthropic把Agent拆成三个接口：

- **Session**：只负责记事，append-only log，所有事件写进去，谁都能读
- **Harness（脑）**：调用Claude、路由tool calls、管理上下文
- **Sandbox（手）**：执行代码、编辑文件、跑外部工具

![Harness架构图](https://p26-sign.toutiaoimg.com/tos-cn-i-axegupay5k/8b4aefa011254145a8cd9cd0b722eb4d~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669490&x-signature=E9%2BfoLcNOJBx%2BdjNvAYQANzyaR4%3D)

关键在于交互方式：Harness把Sandbox当成一个tool（`execute(name, input) → string`），不在乎Sandbox是容器、手机还是Pokémon模拟器。Sandbox变成cattle——死了一个，换新的用标准recipe初始化就行。

![Harness与Sandbox交互](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/4152f5da2ba94486a868fd1e293f0f80~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669490&x-signature=VBgpkgA0IjrPOAvCNFMjeOhhNPQ%3D)

### 安全边界重塑

耦合设计中有个隐形炸弹：Sandbox里跑着Claude生成的代码，同一容器又存着凭证。Prompt injection只需让Claude读环境变量就拿到token。

解法：**凭证永远不进Sandbox**。Git的access token只在初始化时clone用，MCP工具的OAuth token存在vault里走proxy代理调用。Harness完全不知道凭证的存在。

### Session与上下文窗口分离

Session设为context window外面的独立对象，`getEvents()`可按位置切片读事件流，支持回退和重读。Session只管存，Harness只管怎么用。

![Session与Harness事件流](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/b506246c591c40bc84a8a751ef397f02~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669490&x-signature=t654kygKBXTdJ7Jh7iURhhgqMaU%3D)

### 性能收益

拆开后容器只在需要时才provision，Harness变成cattle。Anthropic报告**p50 TTFT降了60%，p95降了90%**。

### 核心洞察

文章本质在说一件事：**别做假设**。Harness假设所有资源都在身边、假设Claude会焦虑context limit、假设一个脑绑一个手——这些假设在模型快速迭代的今天随时会变成废代码。

> Operating systems have lasted decades by virtualizing the hardware into abstractions general enough for programs that didn't exist yet.

Managed Agents也想这样：接口设计得足够通用，背后的实现可以随便换。这是一个**meta-harness**的思路——不规定Claude需要什么harness，而是提供一个能装任何harness的系统。
