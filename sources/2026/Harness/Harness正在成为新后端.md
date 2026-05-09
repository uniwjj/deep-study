---
ingested: 2026-05-09
wiki_pages: [ai-agent/harness-as-backend]
---
## Harness 正成为新后端

**文章信息**：
- **原文标题**：Harness 正在成为新后端（原标题较长，此为提炼）
- **来源**：https://mp.weixin.qq.com/s/6bKuYLV1E5LGqUEKV0tbtA
- **作者**：架构师（公众号 JiaGouX，主理人若飞）
- **发表时间**：约 2026 年 5 月初

---

**一段话总结**：

文章从"30 分钟手搓 Agent"的最小 loop 出发，继续往后端方向推一层：当 Agent 进入真实业务系统，Harness 会开始承担后端职责。基于 iii.dev 开源运行时和 Mike Piccolo 的 Heavybit 访谈，将 Agent 后端的核心概念翻译为三个工程词——参与者（Worker，可以是 Agent/浏览器/沙箱/Python/Rust 服务）、能力（Function，有稳定 ID 的工作单元）、入口（Trigger，HTTP/队列/cron/状态变化）。核心论点是：工具目录应从静态"函数数组"升级为运行时能力目录，后端要开始给 Agent 提供可发现的能力、可恢复的状态、可审计的权限、可追踪的链路。文章提出四项落地动作：能力目录化、worker 上下线纳入正常路径、从主动调用扩到事件触发、trace 从协议层设计，同时明确了六个工程边界。

---

**核心内容**：

**一、从 Demo Loop 到生产后端的差距**

最小 Agent loop（模型→选工具→执行→写回上下文→下一轮）让 Agent 动起来，但真实长任务中 Agent 可能先读仓库、再开浏览器查文档、进沙箱跑测试、调 Python 分析服务、写结果到状态存储。这些动作如果各走各的 SDK、各写各的日志、各自管重试，排查会非常困难。

公式警示：agents² × services —— 1 个 Agent 接 5 个后端系统 = 5 条路径；4 个 Agent 接 5 个系统 = 80 条组合路径。复杂性多了一层运行时组合。

**二、三个核心概念重新翻译**

| 原文词 | 工程翻译 | Agent 系统含义 |
|--------|---------|--------------|
| Worker | 参与者 | 能连接到运行时、注册能力的进程 |
| Function | 能力 | 有稳定 ID 的工作单元 |
| Trigger | 入口 | 什么事触发这个能力运行 |

TypeScript API、Python 脚本、Rust 微服务、浏览器 tab、沙箱、Agent 本身都可以是"参与者"，只要能连到运行时并注册能力，就能进同一套系统。

核心原则：An agent is a worker. Its tools are functions. Its memory is state. Its orchestration is triggers.

**三、运行时能力目录 vs 静态工具列表**

- 服务发现告诉程序"服务在哪里"
- 运行时能力目录告诉 Agent"此刻系统实际能做什么"——工人是不是在线、版本对不对、权限允不允许、调用能不能进同一条链路
- Worker 连接时 engine 把 functions 和 triggers 加入 live registry，断开或注销时立刻变化
- 支持两类操作：查询当前能力、订阅能力变化

MCP 解决了 Agent 连接外部工具的成本问题；运行时目录需要继续看工具背后的进程状态和追踪。

**四、四个落地动作**

1. **工具从函数数组升级为能力目录**：每个工具要有 ID、描述、输入输出 schema、owner、版本、权限和观测信息。不一定是复杂系统，一张数据库表或配置文件就可以起步。

2. **Worker 上下线当成正常路径**：SDK 断线重连后重新注册 function 和 trigger；engine 维护 function_owners 处理旧 worker 清理和新注册的竞态。

3. **从 Agent 主动调用扩到事件触发**：Agent 可被 HTTP/cron/队列/状态变化触发——线上告警触发 incident_triage，每晚 cron 触发 release_notes，用户上传文件后队列触发 document_reader，审批通过触发 sandbox::execute_plan。

4. **Trace 从协议层设计**：engine 的 InvokeFunction 和 InvocationResult 都携带 traceparent 和 baggage。Agent 执行跨语言、跨进程、跨队列、跨工具，没有统一 trace 就只能 grep 所有日志猜测。

**五、六个需要提前想的工程边界**

1. **中心 engine 可用性**：routing/registry/trigger/observability 都收拢到同一运行时，它自己就是承重点
2. **统一原语不等于统一语义**：HTTP 关心延迟和响应，队列关心重复消费和死信，cron 关心锁和错过执行窗口
3. **能力目录会膨胀**：命名规范、版本策略、owner、权限、废弃流程必须跟上
4. **动态创建 worker 很危险**：Agent 拉起 sandbox/浏览器/临时服务需要 RBAC、沙箱隔离、资源配额和审计
5. **许可证边界**：engine 是 Elastic License 2.0，SDK 是 Apache 2.0
6. **上下文工程不会自动消失**：能力目录能告诉 Agent 有哪些能力，不能替它决定当前任务该看哪段上下文

**六、三层问题框架**

作者将 Agent 系统建设分为三层：第一层（30 分钟 loop）让 Agent 跑起来；第二层（Harness/长任务/上下文/Subagent）让 loop 跑稳；第三层（本文）让 loop 进入真实后端系统，与浏览器、沙箱、状态、队列、不同语言服务一起工作。

核心自问："我们的后端，离一个 Agent 能理解、能参与、能恢复、能追踪的系统，还有哪些缺口？"
