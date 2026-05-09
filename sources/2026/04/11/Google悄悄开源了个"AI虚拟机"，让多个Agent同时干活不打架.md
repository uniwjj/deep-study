---
ingested: 2026-05-09
wiki_pages: [ai-agent/google-scion]
---

# Google悄悄开源了个"AI虚拟机"，让多个Agent同时干活不打架

**来源**：不1不2  
**时间**：2026-04-11  
**链接**：https://www.toutiao.com/w/1862140329945100

## 一段话总结

Google开源了内部项目Scion，核心能力是让每个AI助手跑在独立容器里互不干扰，支持多Agent并行执行代码审计、单元测试、文档生成等任务，将30分钟工作缩到10分钟。目前已适配Claude Code、Gemini CLI、Codex、OpenCode四个主流工具。

## 作者判断

作者认为趋势很清晰：AI正在从单兵作战变成团队协作。以前是你问一个问题等一个答案，未来是你定义一批任务，一群AI各自干活最后汇总结果。单Agent的天花板正在被多Agent的编排层打穿。但客观指出项目还在早期，本地运行基本稳定，Hub多机器协作约80%功能可用，K8s集群支持还有问题。

## 核心内容

**Scion核心逻辑**：一台电脑能开多个虚拟机，Scion让每个AI助手跑在独立容器里，互不干扰。

**已适配工具**：Claude Code、Gemini CLI、Codex、OpenCode，未来任何能跑在容器里的AI工具都能接进来。

**示例项目**：Google配了多Agent解谜游戏《Athenaeum遗迹》——一组AI分饰侦探、工程师、分析师，通过共享工作区和消息协作解谜，全程自动化。

![Athenaeum遗迹多Agent协作解谜游戏](https://p26-sign.toutiaoimg.com/tos-cn-i-qvj2lq49k0/ce75340a942c4a39a6c2d3b044423842~tplv-obj:1079:946.image?_iz=97245&bid=15&from=post&gid=1862140329945100&lk3s=06827d14&x-expires=1783814400&x-signature=6JB7yE%2BS%2FbXuofpYbbO7yyc5wjY%3D)

**当前局限**：还在早期，需要会Docker和命令行，有一定技术门槛；Hub多机器协作约80%功能可用；K8s集群支持还有问题。
