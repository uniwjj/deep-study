---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-managed-agents]
---

# Anthropic亲自下场，又一批Agent创业公司死掉了

**来源**：钛媒体APP / 世界模型工场  
**时间**：2026-04-09  
**链接**：https://www.toutiao.com/article/7626717030502138403/

## 一段话总结

Anthropic发布Claude Managed Agents，直接砍掉Agent创业公司赖以生存的中间层。这是一套fully managed agent harness，开发者可通过API创建agents、配置容器、运行sessions，自建类似基础设施需6-12个月，而Anthropic按小时计费直接提供。微软Foundry Agent Service和Google Vertex AI Agent Engine也在同步推进平台化，三类创业公司将被降维打击：API中转商、通用Agent开发与编排平台（StackAI、E2B、Dify.ai）、无差异化编排框架（LangChain、CrewAI）。

## 作者判断

作者持悲观论调，认为这是AI行业的残酷循环——每次模型厂商升级能力，就有一批创业公司死掉。2023年GPT-4杀死文本生成公司，2024年GPTs杀死轻量Agent平台，2025年Operator杀死浏览器自动化，2026年Managed Agents杀死Agent运行层。作者认为真正能活下来的是做垂直闭环、拥有行业know-how和数据飞轮的公司，而非"把平台红利当自己产品能力"的中间层玩家。

## 核心内容

**Claude Managed Agents本质**：不是更好的API调用，而是把Agent直接扔给Anthropic运行。过去开发者需要自己扛起agent harness、受管运行环境、内置工具、沙箱、安全执行、长任务运行等整套基础设施，现在Claude Managed Agents全包。

**被冲击的三类公司**：

1. **API中转商**：靠解决"国内无法直接访问Claude"活着的公司，服务价值瞬间归零
2. **通用Agent编排平台**：StackAI（可视化拖拽）、E2B（安全沙箱）、Dify.ai（一站式编排）的核心卖点被官方完美覆盖
3. **无差异化编排框架**：LangChain、CrewAI从必需品沦为可选项

**能活下来的玩家**：金融风控Agent（自有风控数据）、医疗Agent（专有病例库）、工业Agent（产线IoT）——强场景、强数据、强结果交付的垂直闭环公司。


