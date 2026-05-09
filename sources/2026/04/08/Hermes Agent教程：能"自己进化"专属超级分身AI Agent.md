---
ingested: 2026-05-09
wiki_pages: [ai-agent/hermes-agent]
---

# Hermes Agent教程：能"自己进化"专属超级分身AI Agent

**文章信息**
- 来源：今日头条
- 作者：数创共和
- 发布时间：2026-04-08 14:17
- 原文链接：https://www.toutiao.com/article/7626262813001990702

## 一段话总结

Hermes Agent是开源AI Agent框架中最会"自己进化"的选择。Nous Research于2026年2月发布的Hermes Agent采用内置自我改进学习循环，每次完成任务后自动提炼可复用Skill存入持久记忆，用得越久越像你的数字分身。

## 作者判断

作者是技术自媒体"数创共和"，立场明显倾向Hermes Agent，认为它在自我进化和记忆深度上碾压OpenClaw。文章带有推广性质（多次插入"完整版请站内私信获取"），但技术对比内容本身相对客观。

## 核心内容

### 什么是Hermes Agent

Hermes Agent是Nous Research开源的自改进AI Agent框架，核心创新是**内置学习闭环（closed learning loop）**：

- **持久多层记忆**：SQLite + FTS5全文搜索 + LLM自动总结，跨会话永久记住你的偏好和历史
- **自动技能进化**：任务完成后自动生成Markdown Skill文件，下次直接调用并自我迭代优化
- **自主执行力**：支持终端命令、浏览器、文件操作、代码生成、Web搜索等40+工具
- **模型超灵活**：支持OpenRouter（200+模型）、OpenAI、Anthropic、Nous Portal、本地Ollama等
- **完全开源免费**：MIT协议，可在$5 VPS、本地、Docker、Modal等环境运行

![Hermes Agent对比图](https://p26-sign.toutiaoimg.com/tos-cn-i-axegupay5k/8c036c047ec04610ac7fa9e9b3206840~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669295&x-signature=4VrlWmJDQrPh5FLFQmAZgjRxYf4%3D)

### Hermes vs OpenClaw

两者都是本地自托管、持久记忆的Agent，但设计哲学完全不同：

- **Hermes最大杀手锏**：自我进化——OpenClaw更像"万能工具箱"靠你喂Skill，Hermes自动成长
- **记忆深度碾压**：跨会话更懂你，不会重复问同样问题
- **更轻更专注**：部署快、资源占用低，适合研究型/长期个性化用户
- **零痛迁移**：内置`hermes claw migrate`，可直接导入OpenClaw的记忆、Skill、配置

![Hermes vs OpenClaw 全方位对比](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/de14d5ca294b44569f72fd3fbe1b698d~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669295&x-signature=5aLOAIZ%2B4BLiyrfaGHwDEsFH10o%3D)

![深度对比详情](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/31e1543f23f94335a110f105e122db8b~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669295&x-signature=0juU0OyFmyRgfldgniw74quOd3I%3D)

### 安装方法

Mac/Linux终端执行：
```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

Windows PowerShell执行：
```bash
irm https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1 | iex
```

![安装脚本运行界面](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/711c2fedf5da4ac48e1b3a31c0717658~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669295&x-signature=PDqbrINUORUIgAQfj4Jjj9SLdG8%3D)

脚本自动检测并安装Python、Node.js、Git、ripgrep等依赖。安装后选Quick setup，配置OpenRouter API Key、默认模型、消息平台（Telegram/Discord等），最后安装为系统服务可开机自启。

### 核心命令一览

![核心命令列表](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/ac14007305d0476eb8c58cfa74606bd7~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669295&x-signature=1aEFty1%2BLLrhMqCQ3jS0VIXkpS0%3D)

### Token消耗对比

Hermes采用分层召回模式（将经验转化为技能、仅提取相关片段），相比OpenClaw全量上下文模式可节省30%-60%的Token消耗。

![Token消耗对比图](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/ace4d3129c2244a2bf3a9a93f6781d4a~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669295&x-signature=88k4l4L4inhwa5BUUtGslDgnbu4%3D)

### 总结

Hermes Agent真正做到了"越用越聪明"，是目前开源Agent中自我进化能力最强的选择。很多用户现在两者并用（OpenClaw做执行，Hermes做大脑）。

