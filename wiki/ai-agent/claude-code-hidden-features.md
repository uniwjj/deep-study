---
title: Claude Code 隐藏功能
description: Claude Code 源码泄露的 87 个 feature flag 揭示的进化路线——从工具到常驻助手到多 Agent 协作到跨设备
aliases: [Claude Code feature flags, Claude Code未发布功能, Kairos, Daemon, Teleport]
tags: [ai-agent, tool, concept]
sources: [2026/04/01/Claude Code 87个未发布功能.md, 2026-05-10/Claude-Code-Source-Analysis.pdf]
created: 2026-05-09
updated: 2026-05-10
---

# Claude Code 隐藏功能（87 个 Feature Flags）

## 进化路线

```
工具(Tool) → 常驻助手(Daemon) → 团队协作(Coordinator) → 跨设备(Teleport) → 多感官(Voice)
```

> "从'你问它答'到'它自己找活干'，再到'它按时间表自动干'——这不是增量升级，是换了个物种。"

## 核心隐藏功能

### 一、Kairos + Daemon：去掉"你不问它不动"

| 能力 | 说明 |
|------|------|
| **Kairos 主动助手** | 24 小时在线，周期性 tick 评估有无可做之事，有就做、没就休眠 |
| **colleague 定位** | 提示词不再叫 "tool" 而是 "colleague"（同事） |
| **焦点感知** | 终端失焦=更大胆自主行动；聚焦=更多协商 |
| **专属工具** | SleepTool（休眠）、SendUserMessage（主动发消息）、PushNotification（手机推送）、SubscribePR（订阅 PR webhook）、CronCreate（定时任务） |
| **Daemon 底座** | 四种会话类型（interactive/bg/daemon/daemon-worker），headless 常驻进程，内置 cron 调度器 |
| **远程 scheduled agents** | 在 Anthropic 云端跑隔离的 Claude Code 实例，最小间隔 1 小时 |

### 二、Auto-Dream：去掉"下次要重新教"

- 后台自动整理记忆 → [[claude-code-auto-dream]]
- Kairos 模式下改用精细 cron 定时 dream

### 三、Coordinator + UDS + Ultrareview：去掉"只能一个人干"

| 能力 | 说明 |
|------|------|
| **Coordinator 模式** | 多 Agent 编排：Research → Synthesis → Implementation → Verification |
| **反偷懒设计** | 禁止写"based on your findings"，协调者必须消化信息给出精确指令 |
| **UDS Inbox** | 同机多个 Claude Code 实例通过 Unix Domain Socket 互相通信 |
| **Ultrareview** | 云端启动 5-20 个 Agent 多角度审查代码，≤22 分钟 |

### 四、Teleport + Ultraplan：去掉"绑定在一台机器"

| 能力 | 说明 |
|------|------|
| **Teleport** | `--remote` 上传到云端，`--teleport` 拉回；自动 clone/checkout/stash |
| **Ultraplan** | 云端启动远程 Claude Code（Opus 4.6），30 分钟探索项目、规划步骤 |

### 五、Voice + Chrome：去掉"只能打字"

- **Voice**：实时语音转文字（Deepgram Nova 3），20 种语言，16kHz/16bit/PCM，自动提取项目领域词汇提高识别
- **Chrome 控制**：在用户现有 Chrome 中操作（导航/点击/填表/截图/录 GIF/读 network 请求）

### 六、Buddy：彩蛋

- 虚拟宠物：18 种物种、五档稀有度（common 60% ~ legendary 1%）、5 项属性
- 反作弊：用户 ID + 盐值 FNV-1a 哈希，结果完全确定
- 发布窗：2026/4/1-7，24 小时滚动覆盖全球时区

## 关键信号

- 87 个 feature flag 代码已完成，非原型，随时可上线
- Anthropic 的路线是：去掉 Agent 身上的每一个限制，直到它不再像工具
- 提示词已不叫 "tool"，而叫 "colleague"

## 相关页面

- [[claude-code-architecture]] — Claude Code 源码架构
- [[claude-code-sourcemap-reverse]] — Sourcemap 逆向还原
- [[claude-code-auto-dream]] — Auto Dream 记忆机制
- [[agent-multi-agent-collaboration]] — 多 Agent 协作模式
- [[openclaw]] — OpenClaw 数字员工对比
- [[claude-code-internal-mechanisms]] — 内部机制（Undercover Mode/反蒸馏/情绪检测/对抗性验证）

## GrowthBook Feature Flag 基础设施

- **remoteEval**：flag 值由 GrowthBook 服务端预计算，非客户端规则评估
- **磁盘缓存**：flag 值写入 `~/.claude.json`——GrowthBook 不可达时仍能工作
- **多层覆写**：环境变量 > 本地配置覆写 > 远程值。内部员工可通过 `CLAUDE_INTERNAL_FC_OVERRIDES` 强制任何 flag
- **曝光追踪**：每次 flag 访问记录一次曝光事件，同 session 内自动去重
- 所有 flag 前缀为 `tengu_`（Tengu = Claude Code 内部项目代号）

## Dead Code Elimination 机制

`process.env.USER_TYPE === 'ant'` 是**编译时常量**（构建 `--define`）。打包器做常量折叠：`false ? require('./internal') : null` → 整个 `require` 模块被 tree-shake。外部构建**物理上不含**任何内部代码。

## BUDDY 实现细节

- **PRNG**：Mulberry32 算法
- **Seed**：`hash(userID) + salt 'friend-2026-401'`
- **18 物种**，五档稀有度（Common 60% → Legendary 1%）：duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, capybara, cactus, robot, rabbit, mushroom, chonk
- **5 属性**：DEBUGGING, PATIENCE, CHAOS, WISDOM, SNARK
- **反作弊**：骨骼数据（稀有度、物种）从不持久化——每次从 userID 重新计算。灵魂数据（名字、性格）可存储
- 物种名使用 `String.fromCharCode` 十六进制编码绕过构建时的敏感字符串扫描

## 44 个 tengu_ 前缀 Flag

| Flag | 功能 |
|------|------|
| `tengu_amber_flint` | Swarm/Team 能力开关 |
| `tengu_scratch` | 共享 scratch 目录（agent 间文件共享） |
| `tengu_hive_evidence` | Verification Agent（完成后对抗性验证） |
| `tengu_penguins_off` | Penguin Mode（动态速度质量权衡） |
| KAIROS（~150 次代码引用） | 常驻助手，只追加日志，15s 阻塞预算，定时任务 |
| ULTRAPLAN | 远程规划（CCR 中 Opus 4.6），30 分钟思考，浏览器审批 |
