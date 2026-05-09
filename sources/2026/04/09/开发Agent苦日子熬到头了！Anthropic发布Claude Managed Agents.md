---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-managed-agents]
---

# 开发Agent苦日子熬到头了！Anthropic发布Claude Managed Agents

**文章信息**
- 来源：今日头条
- 作者：51CTO
- 发布时间：2026-04-09 15:08
- 原文链接：https://www.toutiao.com/article/7626652547272491547

## 一段话总结

Claude Managed Agents将Agent开发中80%的基础设施工作全托管，用户只需定义任务、工具和边界。拆分架构后p50 TTFT降60%、p95降超90%。

## 作者判断

51CTO作为技术媒体，以科普口吻全面拆解Managed Agents的技术细节。立场积极正面（标题用"苦日子熬到头了"），引用X上网友评论"20%写业务逻辑，80%构建基础设施→Anthropic负责这80%"来强化价值主张。评论区有开发者反驳"这是Agent开发完蛋了"。

## 核心内容

### 为什么需要Managed Agents

旧的Agent Harness（控制逻辑）会随模型迭代不断"过时"。经典例子：Claude Sonnet 4.5有"情境焦虑"（感知到上下文极限会提前结束任务），工程师加了context reset。但Claude Opus 4.5上线后这些就不需要了，成了"累赘"。每次模型升级都要大改控制逻辑。

### 架构拆解：不要养宠物，要养牛

早期把Session、Harness、Sandbox全塞一个容器，导致容器成了"宠物"（挂了就丢记忆、调试要暴露用户数据）。

解法：拆成三个独立部分：
- **Brain（大脑）**：Claude模型 + Harness控制逻辑
- **Hands（双手）**：沙箱、工具、外部服务
- **Session（记忆）**：持久化事件日志

关键操作：Harness离开容器，通过`execute(name, input) → string`调用沙箱。沙箱坏了？Claude决定重试，用`provision({resources})`初始化新的，不用人工抢救。

![架构接口表](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/85af37e6690b45368e046a0892509d9a~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669668&x-signature=o8mo6PSxoWd9PnBbQQ%2BgMfXYyaI%3D)

### 安全性提升

旧架构凭证和用户代码同容器，prompt injection可直接读环境变量拿token。新架构：
- **Bundled Auth**：Git token在初始化时写入remote，Agent从没见过token
- **Vault + Proxy**：MCP工具OAuth token存在vault，代理负责取凭证

### 上下文管理

Session成为上下文窗口外的独立对象，`getEvents()`可灵活查询事件流任意切片。Session只管持久化，Harness负责上下文工程。

![Session与Harness交互](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/581cd1f3185146b3ae1e966a6a0a9f7d~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669668&x-signature=Thz%2BT1VSPir8c%2BP7vakF%2FaJUPQo%3D)

### 多脑多手：性能收益

大脑与手分离后，容器只在需要时才provision。**p50 TTFT下降约60%，p95下降超90%**。

![Harness与Sandbox交互](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/cbf2874cc04b438d8e5f961e88f71f97~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669668&x-signature=8o8D3Noig2SVzD1whmRsqbOp3gc%3D)

### 范式转变

Managed Agents不是具体的Agent框架，而是**meta-harness**——接口设计足够通用，底层实现随便换。正如网友所言："2025年：人人都在聊Agent；2026年：Agent变成了云服务。"

![Claude Managed Agents推文](https://p3-sign.toutiaoimg.com/tos-cn-i-axegupay5k/12af3f9259a24e8faf7aa4f95d394658~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669668&x-signature=1FI0u67M5gLHhnYegAdGeTe5CII%3D)

