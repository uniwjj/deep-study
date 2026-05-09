---
title: Claude Code 隐藏功能
description: Claude Code 源码泄露的 87 个 feature flag 揭示的进化路线——从工具到常驻助手到多 Agent 协作到跨设备
aliases: [Claude Code feature flags, Claude Code未发布功能, Kairos, Daemon, Teleport]
tags: [ai-agent, tool, concept]
sources: [2026/04/01/Claude Code 87个未发布功能.md]
created: 2026-05-09
updated: 2026-05-09
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
