---
ingested: 2026-05-09
wiki_pages: [ai-agent/claude-code-architecture]
---

## Claude Code 源码架构解析：从启动、Prompt 到权限管道

**文章信息**：
- **原文标题**：Claude Code 源码架构解析：从启动、Prompt 到权限管道
- **来源**：https://mp.weixin.qq.com/s/ibU8rAPPkcWrBKw3wArUFw
- **作者**：架构师（JiaGouX）
- **发表时间**：2026-04-01 23:23

---

**一段话总结**：

Claude Code 源码泄露后，作者沿着"启动→上下文装配→主循环→权限管道"这条主线深入解析了其工程实现。src/main.tsx 说明它从一开始按长期交互系统优化，而非一次性 CLI。Prompt 在这里不是静态文案，而是一组可缓存的装配层。工具调用遵循"先读再改""默认保守"的工程纪律。权限系统更像一条决策管道而非界面逻辑。长任务续航将历史压缩与状态续写分开处理，工程完整性比较扎实。作者认为泄露的主要是工程实现层，最值得看的主线是这几层怎么连起来。

---

**作者判断**：

作者明确表示"我不确定每个看法都对，但至少压在了具体代码上，比纯靠二手材料踏实一点。" 作者认为如果只看架构和代码，最值得看的主线不是"功能大全"，而是启动、上下文装配、主循环、工具契约、编辑约束、权限管道、长任务续航这几层怎么连起来。作者同时指出这些内容与此前 Harness 认知框架文章的观察是一致的，这次有了具体代码支撑，比纯靠二手材料更踏实。

---

**核心内容**：

**核心主线**：Claude Code 本地 Agent Runtime 从启动到上下文装配到主循环到权限管道，层层搭接。泄露的主要是工程实现层，不是模型权重。

**太长不看版要点**：
- 泄露的是 Claude Code 工程实现层，非模型权重
- 最值得看的主线：启动、上下文装配、主循环、工具契约、编辑约束、权限管道、长任务续航
- src/main.tsx 从一开始按长期交互系统优化
- Prompt 更像装配层，不是静态文案
- src/QueryEngine.ts + src/utils/processUserInput/processUserInput.ts + src/query.ts 共同构成运行主链路
- src/Tool.ts + src/tools.ts + src/tools/FileEditTool/* 体现"先读再改""默认保守"的工程纪律
- src/utils/permissions/permissions.ts 更像权限决策管道

Claude Code 运行主链路图：

![Claude Code 运行主链路](https://mmbiz.qpic.cn/mmbiz_png/Fnx2G2wYdELd52lglicmVKKPmSOvchsBhn0lI3CKzC8mLr0IibV4Mn7Zx0NicJicEXPKeOMCqFAOPomck56G365NsIPSw9MfHxqibORye35b0Ouw/640)

**启动与入口（src/main.tsx）**：从一开始就按长期交互系统优化，而非一次性 CLI。这解释了为什么 Claude Code 看起来不只是个终端聊天工具。

**Prompt 装配（src/constants/prompts.ts + src/utils/queryContext.ts）**：Prompt 在 Claude Code 中更像装配层，不是一段静态文案。这比许多产品更早把"上下文装配"当成了一层正式工程对象。

Prompt 装配架构——上下文作为可缓存对象：

![Prompt 装配架构图](https://mmbiz.qpic.cn/mmbiz_png/Fnx2G2wYdELZhp0vnFSCfGojGR0XRmouPdfEWXzJbRZwT80MAv6DHWV4kUCRldZfmLWDSKdT9WmYP9cDwFwBr56aJBwiaVbRngCqZfqDibQ5s/640)

**工具契约（src/Tool.ts + src/tools.ts + src/tools/FileEditTool/*）**：最能看出 Anthropic 如何把"先读再改""默认保守"写成机制。靠的是工程纪律，不是某个神奇技巧。工具调用前先把边界写清楚。

工具调用边界约束机制：

![工具调用边界约束](https://mmbiz.qpic.cn/sz_mmbiz_png/Fnx2G2wYdEJskiaTYMyibNFzIn94yfd2GdbZlnFuibwrhiaNbcrZvaPbEHDwBZJCsP3vbAkZ1djBdWtx0cNFThVe1z5MjEkejSoj8Zx4WmlBsC0/640)

**权限管道（src/utils/permissions/permissions.ts）**：更像权限决策管道，不只是一层"最后再补"的界面逻辑。本地环境持续运行的产品，权限不太可能继续作为界面逻辑存在。

权限决策管道流程：

![权限决策管道](https://mmbiz.qpic.cn/mmbiz_png/Fnx2G2wYdEJvnTFTQ97CjAXbhial2icGJMblib3rprUP7VV3g8mc0pZ9RMwPRYLzQKEFaxIdWibYMVBEvvbESIQCS9oFhibwyXUrTdSiazCkg7Y0c/640)

**长任务续航**：把"历史压缩"和"状态续写"分开处理，工程完整性比较扎实，做到了最小侵入的处理。

长任务历史压缩与状态续写架构：

![长任务续航架构](https://mmbiz.qpic.cn/mmbiz_png/Fnx2G2wYdEJwPp37xe0FWOBJMBAYXsqEdW8ds7cRLIlhgLXCCMZAYiaHDiaFI6DF1sFFjr7rkn9yHYiaUXyZXnIqGGicn8Otia39w1CC0MasoiaQs/640)

**前序系列文章**：作者此前已发表两篇相关文章——《刚刚，Claude Code"代码泄露"背后：如何重新看 Agent Harness》从事件外层看行业影响，《大家都在讲 Harness，但它到底该怎么理解》补认知框架。本篇是有了本地源码仓库后将观察往具体文件上落一层。

