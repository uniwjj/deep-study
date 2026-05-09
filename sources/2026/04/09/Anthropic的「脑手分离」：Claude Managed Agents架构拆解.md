---
ingested: 2026-05-09
wiki_pages: []
---

# Anthropic的「脑手分离」：Claude Managed Agents架构拆解

**文章信息**
- 来源：今日头条
- 作者：VibeCoder
- 发布时间：2026-04-09 13:22
- 原文链接：https://www.toutiao.com/article/7626625109545042495
- 声明：个人观点、仅供参考

## 一段话总结

Claude Managed Agents将Agent系统拆为Session、Harness、Sandbox三个可独立演进组件，p50 TTFT降60%、p95降超90%。这是从"模型供应商"向"Agent基础设施平台"转型的关键一步。

## 作者判断

VibeCoder是技术博主，本文是5篇文章中质量最高的一篇。立场冷静客观，既深入拆解架构细节，也明确指出限制（Beta阶段、只支持Claude模型、额外$0.08/session-hour费用）。结尾判断"脑手分离的架构方向是对的"，属于有观点的技术分析。

## 核心内容

### 四层API抽象

Claude Managed Agents提供四层可组合的API：
- **Agent**：声明式配置（模型、系统提示、工具集、MCP服务器）
- **Environment**：容器模板（预装包、网络规则）
- **Session**：运行实例（绑定Agent和Environment）
- **Events**：SSE流式通信通道（双向交互）

内置`agent_toolset_20260401`工具集，涵盖bash、文件读写、glob、grep、web fetch、web search。**8种语言官方SDK**（Python、TypeScript、Go、Java、C#、Ruby、PHP + CLI工具ant），覆盖面在Agent平台里最广。

![四层API架构](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/ea85261c67814379807d5bca6c6ba5b0~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669737&x-signature=dU%2BKR7D1ZPBXHZHW2o2Oe7Kbmjs%3D)

### 脑手分离架构

核心问题：**Harness会编码对模型能力的假设，但模型在快速进化**。Claude Sonnet 4.5有"上下文焦虑"，加了context reset；Opus 4.5上线后这成了纯开销。

三个可独立演进的组件：
- **Session**：append-only不可变事件日志，`emitEvent(id, event)`写入，`getEvents()`读取
- **Harness（大脑）**：无状态组件，崩溃后`wake(sessionId)`从Session日志重建上下文
- **Sandbox（双手）**：通过`execute(name, input) → string`统一接口通信，坏了一头换一头

![脑手分离架构](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/2fa5e76130414f6aadb2fbf035b275a6~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669737&x-signature=4Wx75HsZ34ZKEeTGW7yPwvYsYtI%3D)

### 性能收益

容器按需初始化，不需要沙箱的Session直接跳过配置。**p50 TTFT下降约60%，p95下降超90%**，任务成功率提升10%。

![性能数据与Pet到Cattle转变](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/6a7c7f74ae3c439e8bfec55756923b81~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669737&x-signature=W9VhmmffQajQYXBn9Ws204wbAw0%3D)

### 安全：结构性解决方案

凭证从架构层面进不了沙箱：
- **Git令牌**：初始化时clone用，之后push/pull走remote，Agent从未见过token
- **MCP OAuth令牌**：存在外部vault，代理负责取凭证执行调用
- 提示注入读不到任何敏感信息——**不依赖模型是否"听话"**

### 多Agent协调（Research Preview）

线程化委托模型：所有Agent共享容器和文件系统，协调者在主线程，子Agent在独立Session Thread。限制：只支持单层委托（协调者→子Agent，子Agent不能再调Agent）。

![多Agent协调架构](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/a6ed0743a97b4c4ca109138226934bf3~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669737&x-signature=IKeQBE%2BXDjMZKy%2FdD6ivD8H92iA%3D)

### 竞品对比

![竞品对比](https://p3-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/952f8283c1a44750ae44244d36f0c68c~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669737&x-signature=Xenw104F1ZhGhJaGfg8djY3wVD4%3D)

- **vs OpenAI Agents SDK**：OpenAI开源但无全托管运行时，沙箱/检查点/凭证要自建
- **vs Google Vertex AI Agent Builder**：全托管但SDK只有Python和TypeScript
- **vs 开源框架（LangChain/CrewAI/AutoGen）**：框架给最大控制力但运维全自建

Anthropic年化收入已突破300亿美元（超过OpenAI约240-250亿），在企业LLM API市场占32%份额。

### 客户案例

- **Rakuten**：5个部门部署企业Agent，每个仅用一周
- **Sentry**：调试+补丁Agent配对，Bug解决从数月缩短到数周
- **Asana**：AI Teammates在项目管理工作流中直接协作

![客户案例](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/1bd6ac079b064795af135846444ad61b~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669737&x-signature=%2F58pgYzggLRtvSCHzZgGBPdWLM8%3D)

### 定价

标准API token费率外，额外收**$0.08/session-hour**（按毫秒精确计量，空闲不计费），Web搜索$10/1000次。

### 总结

操作系统把硬件虚拟化成process、file这些抽象，通用到足以支撑当年想象不到的程序。Managed Agents把同样的思路用在Agent基础设施上——接口足够通用，底层实现随便换。

![OS级虚拟化思维类比](https://p26-sign.toutiaoimg.com/tos-cn-i-6w9my0ksvp/a5f3a3ac131a4582a7b4ef3989cd5878~tplv-tt-origin-web:gif.jpeg?_iz=58558&from=article.pc_detail&lk3s=953192f4&x-expires=1776669737&x-signature=IHk7d4MuGIFmTwtFTOHyFwTk0hw%3D)

当前限制：Beta阶段、多Agent在Research Preview、只支持Claude模型、额外费用。但"脑手分离"的架构方向是对的。

